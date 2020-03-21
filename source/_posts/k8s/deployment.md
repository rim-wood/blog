---
title: k8s 控制器
date: 2019-08-05 19:30:02
toc: true
tags: 
- k8s
- deployment
- StateFulSet
categories:
- k8s
---
k8s中内置的控制器相当于一个状态机，用来控制Pod的具体状态和行为
<!--more-->

# 控制器定义
k8s中内置的控制器相当于一个状态机，用来控制Pod的具体状态和行为

# 控制器类型

- ReplicationController 和 ReplicaSet
- Deployment
- DaemonSet
- StateFulSet
- Job/CronJob
- Horizontal Pod Autoscaling

## RC和RS

**RC目前不太采用，都采用RS方式。**

RC用来控制pod维持一个正常稳定的数量

RS支持集合式的selector，可以根据标签匹配

样例：
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myRS
spec:
  selector:
    matchLabels:
      auth: myAuth
  replicas: 3
  template:
    metadata:
      labels:
        auth: myAuth
    spec:
      containers:
        - name: myAuth
          image: icepear/dendalion-auth:2.0.0
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
```

## Deployment

**提供声明式的定义方法，用来替代RC**

- 定义deployment来创建pod和replicaSet
- 提供滚动升级和回滚应用
- 扩容和缩容
- 暂停和继续 deployment

deployment跟replicaSet以及pod的关系
![image](http://icepear.oss-cn-shenzhen.aliyuncs.com/k8s/youdao/dpAndRs.png)
![image](http://icepear.oss-cn-shenzhen.aliyuncs.com/k8s/youdao/dpAndRs1.png)

**声明式的创建建议要用kubectl apply····,不要使用kubectl create···**
**--record参数可以记录命令，可以方便查看每次reversion的变化**
### 1. 部署简单的应用

样例：
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
### 2. 扩容
样例：
```yaml
kubectl scale deployment myNginx --replicas=10
```

### 3.集群如果支持HPA，还可以
```bash
kubecl autoscale deployment myNginx --min=10 --max=15 --cpu-percent=80
```
### 4.更新镜像
```bash
kubectl set image deployment/myNginx nginx:1.8.0
```
- 25%-25%的策略，首先会在新的replicas中新建25%，旧的replicas中删除25%。按照这种规律更新
- rollover 策略，当还在创建的时候就更新新的版本，deployment会直接干掉之前创建的rs，直接生成新的

### 5.版本回滚
```bash
kubectl rollout undo deployment/myNginx --to-version=2 #回滚操作，设置回退的版本号

kubectl rollout status deployment myNginx #查看回滚状态

kubectl rollout history deployment/myNginx #查看历史版本信息

kubectl rollout pause deployment/myNginx #暂停回滚更新

```

## DaemonSet

**DaemonSet确保全部node上运行一个pod的副本,新增或删除node时,node上对应的pod也会被新增或删除,删除DaemonSet将删除它创建的pod**

- 运行集群存储daemon,例如在每个node上运行glusterd、ceph
- 在每个node上运行日志收集daemon、例如logstash、fluentd
- 在每个node上运行监控daemon、例如Promethenus node exporter

**创建使用kubectl create**

## Job

**Job负责批处理任务，可以理解为就是用来运行脚本的控制器**

- spec.template格式同pod
- RestartPolicy仅支持Never或Onfailure
- 单个Pod时，默认Pod成功运行后Job即结束
- spec.completions 标志job结束需要成功运行的Pod个数，默认为1
- spec.parallelism 标志并行运行的Pod的个数，默认为1
- spec.activeDeadlineSeconds 标志失败Pod的重试最大时间，超过不再重试

例如:创建一个使用perl语言，计算圆周率打印2000位
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    metadata:
      name: pi
    spec:
      containers:
        - name: pi
          image: perl
          command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```

## CronJob

**在Job的基础上，提供了定时执行，周期执行的方案**

- spec.schedule 运行周期，格式采用Cron
- spec.jobTemplate Job模板，格式采用Job
- spec.startingDeadlineSeconds 启动Job的期限，秒
- spec.concurrencyPolicy 并发策略,因为有可能在第一个job没完成的时候，第二个job又被创建了，就会形成并发
   - Allow 允许
   - Firbid 禁止
   - Replace 取消当前的，用新的替换
- sper.suspend 挂起
- spec.successfulJobHistoryLimit和failedJobsHistoryLimit 设置job成功或失败的保留的pod数，默认成功是3个，失败是1个

例子：
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: myCronjob
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: myCronjob
              image: busybox
              args:
                - /bin/sh
                - -c
                - date; echo Hello k8s
          restartPolicy: OnFailure
```
**CronJob 运行的结果应该是幂等的，就是每次运行的结果应该一样**
## StateFulSet

**StateFulSet解决了有状态的服务运行的问题**

- 稳定的持久化存储，即pod重新调度之后依然能访问到相同的持久化数据，基于PVC实现
- 稳定的网络标志，即pod重新调度之后，podName和HostName不变，基于Headless service实现
- 有序部署，有序扩展，pod启动是有序的，依据顺序依次进行，只有前一个pod启动成功之后，下面才能继续进行
- 有序收缩，有序删除，从后往前删除

## HPA

**用于pod的自动扩展，在高峰时扩容，低谷时删除一些资源，提高系统稳定性**