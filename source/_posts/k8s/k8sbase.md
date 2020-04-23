---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/k8s/k8s.png
title: k8s 基础定义
date: 2019-08-01 21:30:00
toc: true
tags: 
- k8s
categories:
- k8s
---
kubernetes是一个开源的，用于管理云平台中多个主机上的容器化的应用，Kubernetes的目标是让部署容器化的应用简单并且高效（powerful）,Kubernetes提供了应用部署，规划，更新，维护的一种机制。
Kubernetes一个核心的特点就是能够自主的管理容器来保证云平台中的容器按照用户的期望状态运行着（比如用户想让apache一直运行，用户不需要关心怎么去做，Kubernetes会自动去监控，然后去重启，新建，总之，让apache一直提供服务），管理员可以加载一个微型服务，让规划器来找到合适的位置，同时，Kubernetes也系统提升工具以及人性化方面，让用户能够方便的部署自己的应用（就像canary deployments）。
<!--more-->

# 集群资源分类

## 名称空间级别

- 工作负载型资源：Pod、ReplicaSet、Deployment、StatefulSet、DaemonSet、Job、CronJob

- 服务发现及负载均衡型资源：Service、Ingress

- 配置与存储型资源：Volume、CSI(容器存储接口，可以扩展各种第三方存储卷)

- 特殊类型的存储卷：ConfigMap(当配置中心来使用的资源类型)、Secret(保存敏感数据)、DownwardAPI(把外部环境中的信息输出给容器)

## 集群级别资源

- Namespace
- Node
- Role
- ClusterRole
- RoleBinding
- ClusterRolebinding

## 原数据型资源

- HPA
- PodTemplate(pod模板)
- LimitRange(资源限制)

# 资源清单YAML定义，常用字段

```yaml
version: v1 #代表k8s api接口的版本，通常是执行各种命令调用接口的资源路径，可以用kubectl api-versions 查询
apiVerion: 1.0 #版本号
kind: deployment #资源类型 pod/deployment/service
metadata: #object	元数据对象
  name: mydeployment #string	元数据对象名称 如Pod的名字等
  namespace: test #string	元数据的命名空间，指的是资源所处的命名空间,默认default下
spec:     #object	详细的定义资源对象
  replicas: 5 #pods的副本数量
  strategy: #object  #将现有pod替换为新pod的部署策略
    rollingUpdate: #Object 滚动更新配置参数，仅当类型为RollingUpdate
      maxSurge: 3 #滚动更新过程产生的最大pod数量，可以是个数，也可以是百分比
      maxUnavailable: 2 #滚动更新过程减少的最大pod数量，可以是个数，也可以是百分比
    type: RollingUpdate #部署类型，Recreate，RollingUpdate
  revisionHistoryLimit: 5 #设置保留的历史版本个数，默认是10
  rollbackTo:
    revision: 0 #设置回滚的版本，设置为0则回滚到上一个版本
  selector: #pod标签选择器，匹配pod标签，默认使用pods的标签
    matchLabels: 
      key1: value1
      key2: value2
    matchExpressions: 
      operator: In #设定标签键与一组值的关系，In, NotIn, Exists and DoesNotExist
      key: key1
      values: va
  template: #pod模板
    metadata:
    spec:
      containers: # 容器配置
        - name: # 名称
          image: # 镜像
          imagePullPolicy: IfNotPresent #镜像拉取策略 Always、Never、IfNotPresent
          command: #容器的启动命令列表，如不指定，使用打包时使用的启动命令
          args: #容器的启动命令参数列表
          workingDir: #容器的工作目录
          ports:
            - name: #定义端口名
              containerPort: #容器暴露的端口
              hostPort:  #容器所在主机需要监听的端口号，默认与Container相同
              protocol: #TCP UDP 协议
          volumeMounts:
            - name: #设置卷名称
              mountPath: #设置需要挂在容器内的路径
              readOnly: #设置是否只读
          env:
            - name: # 环境变量名称
              value: # 环境变量的值
          readnessProde: #就绪检测
            exec:
              command:
          livenessProbe: #对Pod内个容器健康检查的设置，当探测无响应几次后将自动重启该容器，检查方法有exec、httpGet和tcpSocket，对一个容器只需设置其中一种方法即可
            exec: # 对 pod 容器内检查方式设置为exec方式
              command: # 执行命令或者脚本
            httpGet: # http检测，200-400之间为成功
              path: #请求路径
              port: #端口
              host: #ip
              scheme: #
              HttpHeaders: #请求头
                - name: #键
                  value: #值
            tcpSocket: #tcp方式检测,检测端口是否通的
              port: #端口
            initialDelaySeconds: 0 #容器启动完成后首次探测的时间，单位为秒
            timeoutSeconds: 10 #对容器健康检查探测等待响应的超时时间，单位秒，默认1秒
            periodSeconds: 3600 #对容器监控检查的定期探测时间设置，单位秒，默认10秒一次
            successThreshold: 1 # 连续成功几次认定为成功，默认一次
            failureThreshold: 3 # 连续失败几次认定为失败，默认三次
        restartPolicy: Always #Pod的重启策略，Always表示一旦不管以何种方式终止运行，kubelet都将重启，OnFailure表示只有Pod以非0退出码退出才重启，Nerver表示不再重启该Pod
        nodeSelector:  <map[string]string>  #设置NodeSelector表示将该Pod调度到包含这个label的node上，以key：value的格式指定 
        hostNetwork:false      #是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络
        volumes: #数据卷配置
          - name: #设置卷名称,与volumeMounts名称对应
            hostPath:  #设置挂载宿主机路径
              path: #路径
              type: #类型：DirectoryOrCreate、Directory、FileOrCreate、File、Socket、CharDevice、BlockDevice
          - name: nfs
            nfs: #设置NFS服务器
              server: #设置NFS服务器地址
              path:  #设置NFS服务器路径
              readOnly: false #设置是否只读
          - name: configmap
            configMap: 
              name: #configmap名称
              defaultMode: 664 #权限设置0~0777，默认0664
              optional: false #指定是否必须定义configmap或其keys
              items: 
                - key: asc
                  path: /home/sad
```

## pod生命周期
![image](http://icepear.oss-cn-shenzhen.aliyuncs.com/k8s/youdao/pod%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)