---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/other/firewalld/banner.png
title: linux 防火墙配置
date: 2017-3-21 18:30:02
toc: true
tags: 
- linux
- 防火墙
- network
categories:
- network
---
目前大多情况都使用CentOs7，但Centos升级到7之后，内置的防火墙已经从iptables变成了firewalld。所以两个都记录一下。
<!--more-->
# linux 防火墙配置
## firewalld
Centos7默认安装了firewalld，如果没有安装的话，可以使用 yum install firewalld firewalld-config进行安装。
### 常用命令
```bash
启动防火墙 systemctl start firewalld 

禁用防火墙 systemctl stop firewalld

设置开机启动 systemctl enable firewalld

停止并禁用开机启动 sytemctl disable firewalld

重启防火墙 firewall-cmd --reload

查看状态 systemctl status firewalld
     或者 firewall-cmd --state
```
### 其他命令
```bash
查看版本 firewall-cmd --version

查看帮助 firewall-cmd --help

查看区域信息 firewall-cmd --get-active-zones

查看指定接口所属区域信息 firewall-cmd --get-zone-of-interface=eth0

拒绝所有包 firewall-cmd --panic-on

取消拒绝状态 firewall-cmd --panic-off

查看是否拒绝 firewall-cmd --query-panic

将接口添加到区域(默认接口都在public)
    firewall-cmd --zone=public --add-interface=eth0(永久生效再加上 --permanent 然后reload防火墙)

设置默认接口区域
    firewall-cmd --set-default-zone=public(立即生效，无需重启)

更新防火墙规则
    firewall-cmd --reload 无需断开连接
  或firewall-cmd --complete-reload 需要断开连接，类似重启服务
```
### 打开端口
```bash
查看指定区域所有打开的端口
    firewall-cmd --zone=public --list-ports

在指定区域打开端口
    firewall-cmd --zone=public --add-port=80/tcp --permanent 需要重启防火墙
```
说明：
–zone 作用域
–add-port=8080/tcp 添加端口，格式为：端口/通讯协议
–permanent #永久生效，没有此参数重启后失效
## iptables
### 常用命令
```bash
    开启防火墙(即时生效，重启后失效)：service iptables start

    关闭防火墙(即时生效，重启后失效)：service iptables stop

    开启防火墙(重启后永久生效)：chkconfig iptables on

    关闭防火墙(重启后永久生效)：chkconfig iptables off

    重启防火墙:service iptables restartd
```
### 打开、查看端口
```bash
    /etc/init.d/iptables status
```
打开某个端口(以8080为例)
```bash
    iptables -A INPUT -p tcp --dport 8080 -j ACCEPT 打开

    /etc/rc.d/init.d/iptables save 保存

    /etc/init.d/iptables restart 重启
```

### 其他方式

可以通过修改/etc/sysconfig/iptables文件的方式开启端口，运行
```bash
vi /etc/sysconfig/iptables
```
增加一行
```txt
-A RH-Firewall-1-INPUT -m state –state NEW -m tcp -p tcp –dport 8080 -j ACCEPT
```
参数说明:

–A 参数就看成是添加一条规则
–p 指定是什么协议，我们常用的tcp 协议，当然也有udp，例如53端口的DNS
–dport 就是目标端口，当数据从外部进入服务器为目标端口
–sport 数据从服务器出去，则为数据源端口使用
–j 就是指定是 ACCEPT -接收 或者 DROP 不接收