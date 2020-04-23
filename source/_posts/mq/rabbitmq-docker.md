---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/mq/rabbitmq.png
title: docker中运行RabbitMQ 
date: 2020-04-19 19:30:02
toc: true
tags: 
- RabbitMQ
- docker
categories:
- mq
---
RabbitMQ是实现高级消息队列协议（AMQP）的开源消息代理软件（有时称为面向消息的中间件）。
RabbitMQ服务器使用Erlang编程语言编写，并基于Open Telecom Platform框架构建，用于集群和故障转移。
主要用于各个业务场景的拆分，降低系统耦合性。本文主要介绍在docker中如何运行rabbitmq中间件
<!--more-->

# 下载及简单运行

rabbitmq官方已经编写了docker镜像，![镜像地址](https://hub.docker.com/_/rabbitmq?tab=description)。

### 1. 下载

```bash
docker pull rabbitmq # rabbitmq最小安装
#或者采用（建议）
docker pull rabbitmq:management #带有web控制台的镜像
```

### 2. 运行

```bash
docker run -d --hostname my-rabbit --name some-rabbit -p 15672:15672 -p 5672:5672 rabbitmq:management
```

访问[http://localhost:15672](http://localhost:15672) 即可访问rabbitmq的管理界面。**默认的用户名/密码为 guest/guest**

# 带参数运行

## 通过cookie连接rabbitmq

cookie 对于rabbitmq集群来说相当重要，集群间应该配置相同的cookie使节点互通。[参考官网](https://www.rabbitmq.com/clustering.html#erlang-cookie)
Cookie只是一串字母数字字符，最大长度为255个字符。它通常存储在本地文件中。

1. 在docker 中 可以使用参数**RABBITMQ_ERLANG_COOKIE**在运行命令中开启cookie，

```bash
docker run -d --hostname some-rabbit --name some-rabbit --network some-network -e RABBITMQ_ERLANG_COOKIE='secret cookie here' rabbitmq:3
```

2. 也可以使用挂在cookie文件的方式，cookie文件容器内部的地址在 /var/lib/rabbitmq/.erlang.cookie通过docker 创建secrets文件

```bash
docker service create --name rabbitmq-cookie --secret source=my-erlang-cookie,target=/var/lib/rabbitmq/.erlang.cookie,uid=1001,gid=1001,mode=0600 rabbitmq
```

*需要指定uid = XXX，gid = XXX，mode = 0600，以便容器中的Erlang能够正确读取cookie文件。*

## 其他参数

1. SSL 配置

```bash
# 对于使用管理插件的SSL配置
RABBITMQ_MANAGEMENT_SSL_CACERTFILE
RABBITMQ_MANAGEMENT_SSL_CERTFILE
RABBITMQ_MANAGEMENT_SSL_DEPTH
RABBITMQ_MANAGEMENT_SSL_FAIL_IF_NO_PEER_CERT
RABBITMQ_MANAGEMENT_SSL_KEYFILE
RABBITMQ_MANAGEMENT_SSL_VERIFY
```

2. 设置默认用户和密码

如果要更改**guest / guest**的默认用户名和密码，可以使用**RABBITMQ_DEFAULT_USER**和**RABBITMQ_DEFAULT_PASS**环境变量来进行更改.

```bash
 docker run -d --hostname my-rabbit --name some-rabbit -e RABBITMQ_DEFAULT_USER=user -e RABBITMQ_DEFAULT_PASS=password rabbitmq:3-management
```

3. 设置默认vhost

可以使用**RABBITMQ_DEFAULT_VHOST**来改变vhost

```bash
docker run -d --hostname my-rabbit --name some-rabbit -e RABBITMQ_DEFAULT_VHOST=my_vhost rabbitmq:3-management
```

4. 开启插件

- 修改dockerfile的方法

```bash
FROM rabbitmq:3.7-management
RUN rabbitmq-plugins enable --offline rabbitmq_mqtt rabbitmq_federation_management rabbitmq_stomp
```

- 您还可以在/etc/rabbitmq/enabled_plugins上挂载一个文件，该文件的内容是以英文句号结尾的插件列表名称数组。

例如

```bash
[rabbitmq_federation_management,rabbitmq_management,rabbitmq_mqtt,rabbitmq_stomp].
```

5. 其他配置

其他配置我们可以通过配置文件配置，容器内配置文件地址在/etc/rabbitmq/rabbitmq.conf，文件内容可[参考官网](https://www.rabbitmq.com/configure.html#configuration-files)
在运行的时候，讲写好的配置文件改在到容器内的地址

# 插件

上面我们讲到了插件的开启方式，但是对于插件我们怎么安装呢，在rabbitmq中，插件都放在了/plugins文件夹下，所以我们在运行rabbitmq容器时，
可以把/plugins文件夹挂在到宿主机上,以及开启插件的文件

```bash
docker run -d --hostname my-rabbit --name some-rabbit -v /home/rabbitmq/plugins:/plugins -v /home/rabbitmq/enabled_plugins:/etc/rabbitmq/enabled_plugins rabbitmq:3-management
```

1. 例如下载延迟队列 wget  https://dl.bintray.com/rabbitmq/community-plugins/3.7.x/rabbitmq_delayed_message_exchange/rabbitmq_delayed_message_exchange-20171201-3.7.x.zip

2. 解压得到文件 rabbitmq_delayed_message_exchange-20171201-3.7.x.ez

3. 然后将下载的插件放入宿主机plugins文件夹下，在enabled_plugins文件中添加插件名称。

4. rabbitmq_delayed_message_exchange