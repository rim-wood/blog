---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/k8s/k8s.png
title: k8s service访问控制
date: 2019-08-09 22:30:08
toc: true
tags: 
- k8s
categories:
- k8s
---
kubernetes Service 定义了一个抽象层：用来管理Pod的逻辑分组，外部访问service即可以访问Pod的策略，pod和service通过Label Selector连接
<!--more-->
# service

kubernetes Service 定义了一个抽象层：用来管理Pod的逻辑分组，外部访问service即可以访问Pod的策略，pod和service通过Label Selector连接

**Service提供了负载均衡的能力，但只能提供4层的负载能力，不支持7层，可以借助ingress实现**
![http://icepear.oss-cn-](http://icepear.oss-cn-shenzhen.aliyuncs.com/k8s/youdao/service.png)

## service 类型

- ClusterIp： 默认类型，自动分配一个可供集群内部访问的ip
- NodePort：在ClusterIp的基础上为service在每台机器上绑定一个端口，供外部访问。
- LoadBalance：在Nodeport的基础上，借助cloud provider创建一个外部负载均衡器，并将请求转发到节点端口上，这个是第三方提供的收费方案，像阿里云、AWS等
- ExternalName：把集群外部的服务引入到集群内部来，在集群内部即可使用外部的服务。假如外部服务地址发生变化，也只需要更新externalName的service，而不要更新集群内部的pod。

service实现原理

![image](http://icepear.oss-cn-shenzhen.aliyuncs.com/k8s/youdao/service%E5%8E%9F%E7%90%86.png)

## service代理模式分类

userspace ----> iptables ----> ipvs

![image](http://icepear.oss-cn-shenzhen.aliyuncs.com/k8s/youdao/userspace.png)

![image](http://icepear.oss-cn-shenzhen.aliyuncs.com/k8s/youdao/iptables.png)

**III** ipvs 代理模式

与iptables；类似，ipvs基于netfilter的hook功能，但使用哈希表作为底层数据接口并在内核中工作，这意味着ipvs可以更快的重定向流量，此外ipvs为负载均衡算法提供了更多选项
- rr  轮询调度
- lc  最小连接数
- dh  目标哈希
- sh  源哈希
- sed  最短期望延迟
- nq  不排队调度

**需要注意的是节点上必须安装了 IPVS 内核模块，如果节点未安装，kube-proxy默认会退级采用iptables的方式**

![image](http://icepear.oss-cn-shenzhen.aliyuncs.com/k8s/youdao/ipvs.png)

## service 样例

1. 首先创建一个deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myNginx
  labels:
    app: myNginx
spec:
  replicas: 3
  template:
    metadata:
      name: myNginx
      labels:
        app: myNginx
        version: 1.7.9
    spec:
      containers:
        - name: myNginx
          image: nginx:1.7.9
          imagePullPolicy: IfNotPresent
      restartPolicy: Always
  selector:
    matchLabels:
      app: myNginx
```
2. 创建NodePort service映射到deployment的pod上,**根据selector对应**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: ngService
spec:
  selector:
    app: myNginx
    version: 1.7.9
  ports:
    - port: 80
  type: NodePort
```

3. 创建ExternalName service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myService
spec:
  type: ExternalName
  externalName: hub.icepear.cn
```
这个svc创建之后就会有一个myService.defalut.svc.cluster.local的名称出现，内部只需要访问myService.defalut.svc.cluster.local，到时候就会转发到对应的ExternalName的域名上。

**service跟pod的关系是多对多**

**ipvs可以通过 ipvsadm -Ln 查看路由规则，iptables则通过 iptables -t nat -nvL**

## service ingress

Ingress-Nginx [官网](https://kubernetes.github.io/ingress-nginx/)

![image](http://icepear.oss-cn-shenzhen.aliyuncs.com/k8s/youdao/ingress-Nginx.png)

### HTTP代理样例:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myApp
  labels:
    app: myApp
spec:
  replicas: 3
  template:
    metadata:
      name: myApp
      labels:
        app: myApp
    spec:
      containers:
        - name: myApp
          image: icepear/myApp:v1
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
      restartPolicy: Always
  selector:
    matchLabels:
      app: myApp
---
apiVersion: v1
kind: Service
metadata:
  name: ngService
spec:
  selector:
    app: myApp
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
  type: NodePort
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: my-Ingress
spec:
  rules:
    - host: a.ice.com
      http:
        paths: /
        - backend:
            serviceName: ngService
            servicePort: 80
```

### HTTPS 代理样例

1. 创建证书，以及cert存储方式
```bash
openssl req -x509 -sha256 -nodes -day 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=nginxsvc/O=nginxsvc"

kubectl create secret tls tls-secret --key tls.key --cert tls.crt
```
2. 样例文件
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: my-Ingress
spec:
  tls:
    - hosts:
      - a.ice.com
      secretName: tls-secret
  rules:
    - host: a.ice.com
      http:
        paths: /
        - backend:
            serviceName: ngService
            servicePort: 80
```