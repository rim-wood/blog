---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/docker/registry/registry.png
title: docker 私服库(V2)
date: 2017-4-9 18:30:02
toc: true
tags: 
- docker
- 持续集成
categories:
- docker
---
搭建docker 私服库是持续集成的一部分，当然你可以使用公共库，但我更倾向于私服库，对团队内部来讲还是相当有益的，接下来详细记录如何搭建以及注意环节
<!--more-->
# docker registry 搭建
## 介绍
registry是一种开源的,无状态，高度可扩展的服务器端应用程序，可存储和分发Docker映像。其实这些都是屁话，哈哈，只要了解私服库字面意思就知道是干嘛的了。
V1就直接pass了，官方都抛弃了，直接用的是V2
## 搭建
**前提：先安装了docker，而且版本1.6以上**
### 下载
下载比较简单，通过docker下载镜像就行了
```bash
docker pull registry:2
```
### 简单运行
下载之后，运行
```bash
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```
这样一个私服库就跑起来了，但事实上这样基本上是没卵用的。官方文档说了
*A production-ready registry must be protected by TLS and should ideally use an access-control mechanism*
生产必须被TLS保护，合理的使用访问控制
所以说了这么多其实没卵用，官方文档这么带我的，我也就这么写咯，所以还是看下面吧
### 安全运行
1. 做TLS就要签名证书，所以第一步就是创建证书用openssl
创建文件夹用于存放证书,也为了方便后续挂载至registry容器,官方也有![说明](https://docs.docker.com/registry/insecure/#use-self-signed-certificates)
```bash
$ mkdir -p certs
$ openssl req \
  -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key \
  -x509 -days 365 -out certs/domain.crt
```
注意一定要输入CN（common name）之后的要用到(例如：myregistrydomain.com)
创建之后，文件夹下就有两个文件：domain.crt domain.key
2. 将证书添加至系统信任
使用身份验证时，Docker的某些版本还要求在操作系统级别信任证书。
centos:
```bash
$ cat certs/domain.crt » /etc/pki/tls/certs/ca-bundle.crt
$ update-ca-trust
```
ubuntu:
```bash
$ cp certs/domain.crt /usr/local/share/ca-certificates/myregistrydomain.com.crt
$ update-ca-certificates
```
3. 生成用户和密码
光有TLS是不够的，还必须做到访问控制。实现访问限制的最简单方法是通过Native basic auth（这与其他Web服务器的基本身份验证机制非常相似）。
```bash
$ mkdir auth
$ docker run \
  --entrypoint htpasswd \
  registry:2 -Bbn testuser testpassword > auth/htpasswd
```
4. 启动registry容器
万事具备，只欠东风。现在只需要把容器启动起来，该挂载的挂载，改配的环境变量配上
```bash
docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  -v /home/docker-registry:/var/lib/registry \
  -v /home/wulm/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  -v /home/wulm/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  registry:2
```
至此registry算是搭建成功了，但是真的成没成功还得测试，最简单的是直接访问![https://myregistrydomain.com/v2/_catalog](https://myregistrydomain.com/v2/_catalog)
前提是这个域名写到了你的hosts文件
5. 客户端docker测试
    在另外一台主机上测试私服库是否可用，才能真正测试出正确性。
    - 首先将myregistrydomain.com域名（也就是你配置的域名）对应的IP一起写到你的hosts文件中，以便系统能够根据域名找到私服
    ```bash
    vi /etc/hosts 
    ```
    在文件末尾添加一行
    - 然后不管以什么方法，将私服库上的证书搞到测试机上来,并且写到docker认证的目录下（/etc/docker/certs.d/）
    ```bash
    mkdir -p /etc/docker/certs.d/myregistrydomain.com:5000 建一个文件夹
    cp /certs/domain.crt /etc/docker/certs.d/myregistrydomain.com:5000 然后将证书放到这个目录下（假设证书已经从私服库上拷下来放到/certs/domain.crt中）
    ```
    建议是重启一下docker
    ```bash
    systemctl restart docker
    ```
    这会假如你测试机上跑了其他容器，那这些肯定都是挂了的，一个一个起有太麻烦，所以可以用下面的命令批量重启一下
    ```bash
      docker start $(docker ps -a | awk '{ print $1}' | tail -n +2)
    ```
    - 就可以测试登录了
    ```bash
    docker login myregistrydomain.com:5000
    ```
    不出意外输入账号密码，会提示登录成功。
**good luck**
