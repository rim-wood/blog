---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/jenkins/jenkins.png
title: jenkins 持续集成 
date: 2017-4-10 18:30:02
toc: true
tags: 
- jenkins
- 持续集成
categories:
- jenkins
---
持续集成是DevOps不可或缺的一部分，可以减少大量的部署操作。之所以选择jenkins，其重要原因可能在于其开源免费，接下来进行详细介绍
<!--more-->
# Jenkins
## What is Jenkins?
1. **Jenkins is a self-contained, open source automation server which can be used to automate all sorts of tasks related to building, testing, and deploying software.**
2. **Jenkins can be installed through native system packages, Docker, or even run standalone by any machine with a Java Runtime Environment (JRE) installed.**

官方给出了两点定义,大概的意思就是:

1. jenkins 是一个独立的开源自动化服务器，可用于自动化与构建，测试和部署软件相关的各种任务。
2. Jenkins可以通过本机系统软件包安装，Docker，甚至可以通过安装Java Runtime Environment（JRE）的任何机器独立运行。

## 构建
现在主要的这类软件，我都习惯使用docker进行部署，当然官方也说了可以直接使用jre，根据个人习惯选择。
介绍三种方法:
1. docker方式(推荐)
    - 下载镜像
        ```bash
        docker pull jenkins
        ```
    - 运行实例
    ```bash
    docker run -p 8080:8080 -p 50000:50000 jenkins
    ```
    - 如果想挂载数据到宿主机，就先创建一个目录
    ```bash
    docker run -p 8080:8080 -p 50000:50000 -v /your/home:/var/jenkins_home jenkins
    ```
    这只是简单的运行，并不能用于实际，后面会细说
2. jre运行
[下载](https://jenkins.io/download/),然后运行
```bash
java -jar jenkins.war
```
3. 官方方式
    Ubuntu
    ```bash
    wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
    sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
    sudo apt-get update
    sudo apt-get install jenkins
    ```
    centos
    ```bash
    sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
    sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
    yum install jenkins
    ```
    你也可以在*/etc/sysconfig/jenkins*中修改参数
    ```xml
    JENKINS_USER="root"
    JENKINS_PORT="8008"
    ```
    设置完成之后，运行
    ```bash
    service jenkins start
    ```
    如果能成功启动,那么恭喜你。像我就没这么轻松了，一脚下去几个坑，md，只能默默地填上。
    **坑1:** 本以为可以成功,查看*/etc/log/jenkins/jenkins.log*发现端口占用导致错误,我X,docker那边没停占用了8008
    顺带记录一下查看端口占用命令，老是忘
    查看端口占用：
    ```bash
             netstat -anp | grep 8008 
        或者 lsof -i:8008
    ```
    查看应用占用：
    ```bash
        ps -aux | grep docker
    ```
    **坑2:** 端口解决了，结果尼玛还是失败,发现启动脚本里面的java路径没设置
    ```bash
        vi /etc/init.d/jenkins
    ```
    打开脚本发现
    ```bash
        candidates="
        /etc/alternatives/java
        /usr/lib/jvm/java-1.8.0/bin/java
        /usr/lib/jvm/jre-1.8.0/bin/java
        /usr/lib/jvm/java-1.7.0/bin/java
        /usr/lib/jvm/jre-1.7.0/bin/java
        /usr/bin/java
        "
    ```
    尼玛，还要配java路径,加上之后是可以启动了。
    **坑3** 你以为启动了就ok了,简直是葫芦娃救爷爷，一个一个来。发现访问不了，于是乎怀疑是防火墙问题
    打开端口后，终于正常了
    上图
    ![](http://icepear.oss-cn-shenzhen.aliyuncs.com/jenkins/jenkins-start.png)
## 细说docker方式运行
以docker方式运行，主要是快速方便。搭建docker形式需要注意几点
1. 需要自己编写dockerfile，安装所需的一些工具
2. 如果利用jenkins构建docker镜像，就要考虑是使用dockerIndocker还是dockerOutdocker
3. 如果项目使用docker，可以考虑镜像私服，搭建方法可以![参考](http://www.icepear.cn/2017/04/11/other/docker-registry/)
下面详细针对这几点说明。
- 编写dockerfile是必要的，为什么这么说呢，因为避免不了要使用docker，jenkins使用docker有两种方式，一种是在dockerfile中再安装一个docker就是
所谓的dockerIndocker，但是很多都不太推荐这种用法，大部分还是选择将宿主机的docker挂载至jenkins容器内运行。那既然能挂载那为什么还要写dockerfile
呢，因为你宿主机跟jenkins镜像引入的基础镜像里自带的一些包都有些差异，毕竟镜像不可能像宿主机一样完整。
像我用的centos，直接将docker挂进容器里面用，jenkins运行docker命令就会提示包不存在
```bash
docker: error while loading shared libraries: libltdl.so.7: cannot open shared object file: No such file or directory
```
后面找了一下，其实就是少包
![参考链接](https://stackoverflow.com/questions/45121945/jenkins-in-docker-container-run-docker-pipeline)
所以最终还是编写dockerfile，然后安装缺少的包
假如宿主机docker 的权限是root，直接挂进jenkins容器还是会存在权限问题
运行docker权限错误
```bash
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock:
Get http://%2Fvar%2Frun%2Fdocker.sock/v1.27/containers/json: dial unix /var/run/docker.sock: connect: permission denied
```
![参考链接](https://stackoverflow.com/questions/42164653/docker-in-docker-permissions-error)
办法就是看宿主机docker用户的GID是多少，然后镜像中对应添加一致的，就可以愉快的在jenkins容器中使用docker了，我的dockerfile如下：
```Dockerfile
FROM jenkins:latest

USER root

ARG DOCKER_GID=991

RUN groupadd -g ${DOCKER_GID} docker

RUN apt-get update \
      && apt-get install -y sudo libltdl-dev expect \
      && rm -rf /var/lib/apt/lists/*

RUN usermod -aG docker jenkins

RUN echo "jenkins ALL=NOPASSWD: ALL" >> /etc/sudoers

USER jenkins

```
jenkins自动部署也少不了maven，jdk一系列插件，建议是直接挂在宿主机使用的，运行命令如下：
docker run -p 8008:8080 -p 50000:50000 --add-host stpass-15.com:192.168.110.15 --name jenkins \
    -v /home/wulm/jenkins:/var/jenkins_home \
    -v /home/wulm/jdk1.8.0_131:/usr/java/jdk1.8.0_131 \
    -v /var/maven:/var/maven \
    -v /root/.ssh:/jenkins/.ssh \
    -v /usr/bin/docker:/usr/bin/docker:ro \
    -v /var/run/docker.sock:/var/run/docker.sock \
 -d myjenkins:1.0
（注：--add-host 是为了让容器dns能识别到内网私服库的域名）
