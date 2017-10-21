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
1. docker方式
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
    **坑3** 你以为启动了就ok了,简直是葫芦娃救爷爷，坑一个一个来。发现访问不了，于是乎怀疑是防火墙问题
    打开端口后，终于正常了，唉,不得不说docker还是强大
    上图
    ![](http://icepear.oss-cn-shenzhen.aliyuncs.com/jenkins/jenkins-start.png)
## 插件

docker run -p 8008:8080 -p 50000:50000 --name jenkins \
    -v /home/wulm/jenkins:/var/jenkins_home \
    -v /home/wulm/jdk1.8.0_131:/usr/java/jdk1.8.0_131 \
    -v /var/maven:/var/maven \
    -v $(which docker):/usr/bin/docker:ro \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /var/lib64:/var/lib64 \
-d jenkins