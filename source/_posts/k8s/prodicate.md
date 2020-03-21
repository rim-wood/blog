---
title: k8s 集群调度
date: 2019-09-15 20:30:02
toc: true
tags: 
- k8s
- nodeAffinity
- Taint
- Toleration
categories:
- k8s
---
因为集群节点性能的不同，所以应用分布也需要针对节点特性进行部署，比如，一些节点cpu性能特别强，可以针对大型计算的应用部署，而一些节点存储是ssd的存储，存储速度很快，可以部署一些索引排序的应用，这就是集群调度的作用
<!--more-->
# 集群调度
因为集群节点性能的不同，所以应用分布也需要针对节点特性进行部署，比如，一些节点cpu性能特别强，可以针对大型计算的应用部署，而一些节点存储是ssd的存储，存储速度很快，可以部署一些索引排序的应用，这就是集群调度的作用。
## 简介

Scheduler 是k8s的调度器，要保证四个特性

- 公平
- 资源高效利用
- 效率
- 灵活

Scheduler是单独的程序运行的，启动之后会一直监听API Server，获取PodSpec.NodeName 为空的对每个pod都会创建一个binding，表明该pod应该放到那个节点上

## 调度过程
调度分为几个部分： 首先过滤不满足条件的节点，这个过程称为 predicate 然后通过节点按照优先级排序，这个是  priority  ；最后从中选择优先级最高的节点。如果中间任何一步有错误，直接返回错误

predicate 有一系列的算法可以使用
- PodFitsResources 节点上剩余的资源是否大于pod请求的资源
- PodFitsHost 如果pod制定了NodeName，检查节点名称是否匹配
- PodFitsHostPorts 节点上已经使用的port是否和pod申请的port冲突
- PodSelectorMatches 过滤掉和pod指定的label不匹配的节点
- NoDiskConflict 已经mount的volume 和pod指定的volume不冲突，除非他们都是只读

如果在predicate过程中没有合适的节点，pod会一直在pending状态，不断重试调度，如果满足就会继续priorities过程：按照优先级大小对节点排序

优先级由一系列的键值对组成，见识优先级项的名称，值是它的权重，这些优先级选项包括：
- LeastRequestedPriority 通过计算CPU和Memory的使用率来决定权重，使用率越低权重越高。换句话说就是，优先放到资源使用率低的节点上
- BalancedResourceAllocation 节点上CPU和Memory使用率越接近，权重越高。这个一般跟上一个一起使用，不应该单独使用
- ImageLocalityPriority 倾向于已经有要使用镜像的节点，镜像总大小值越大，权重越高

## 亲和性
分为节点亲和性和pod的亲和性，而亲和性策略也分为软策略和硬策略，软策略为倾向性，硬策略则为必须满足才行。
### 节点亲和性
```
nodeAffinity:
  preferredDuringSchedulingIgnoredDuringExecution 软策略
  requiredDuringSchedulingIgnoredDuringExecution 硬策略
```
1. 下面看一个requiredDuringSchedulingIgnoredDuringExecution例子：
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: affinity
    labels:
        app: node-affinity-pod
spec:
    containers:
        - name: with-node-affinity
          image: myapp:v1
    affinity:
        nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
               nodeSelectorTerms:
                   - matchExpressions: #匹配规则
                       - key: kubernetes.io/hostname #规则为主机名
                         operator: NotIn             #条件为NotIn
                         values:                     #值为下面名称    
                             - k8s-node02
```
**注意nodeSelectorTerms下面有多个选项的话，满足任何一个条件即可，但是如果matchExpressions 有多个选项的话，则必须同时满足这些条件才能正常调度**

2. preferredDuringSchedulingIgnoredDuringExecution例子：
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: affinity
    labels:
        app: node-affinity-pod
spec:
    containers:
        - name: with-node-affinity
          image: myapp:v1
    affinity:
        nodeAffinity:
            preferredDuringSchedulingIgnoredDuringExecution:
               - weight: 1                   # 权重，权重越大优先级越大
               preference:
                   matchExpressions: #匹配规则
                       - key: source              #规则为一个source的标签
                         operator: In             #条件为In
                         values:                  #值为下面名称    
                             - saddas
```
键值运算关系
- In label的值在某个列表中
- NotIn label的值不在某个列表中
- Gt label的值大于某个值
- Lt label的值小于某个值
- Exists 某个label存在
- DoesNotExist 某个label不存在

### pod亲和性
```
podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution 软策略
    preferredDuringSchedulingIgnoredDuringExecution 硬策略
```

例子：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity:    # pod与指定的pod在同一拓扑域
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: failure-domain.beta.kubernetes.io/zone
    podAntiAffinity: # pod与指定的pod不在同一拓扑域
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: security
              operator: In
              values:
              - S2
          topologyKey: kubernetes.io/hostname
  containers:
  - name: with-pod-affinity
    image: gcr.io/google_containers/pause:2.0
```

原则上，topologyKey可以是任意标签，为了性能考虑只允许一些有限的topologyKey，默认情况下，有以下几个：
- kubernetes.io/hostname
- failure-domain.beta.kubernetes.io/zone
- failure-domain.beta.kubernetes.io/region


## 污点 (Taint)和容忍(Toleration)

taint和toleration相互配合使用，可以避免pod被分配到不合适的节点上，每个节点上都可以应用一个或多个taint，这表示那些不能容忍这些taint的pod，不会被该节点运行，如果toleration应用于pod，则表示这些pod可以被调度到具有匹配taint的节点上

### 污点

使用kubectl taint 命令可以给某个Node节点设置污点，node被设置上污点之后就合pod之间存在了一种互斥的关系，可以让node拒绝pod的调度执行，甚至将node已经存在的pod驱逐

污点的形式如下
```bash
key=value:effect
```
而effect有三个选项
- NoSchedule  不会调度到具有该污点的node上
- PreferNoSchedule 尽量避免调度到具有该污点的node上
- NoExecute 不会调度到具有该污点的node上，并且会将node上已经存在的pod驱逐

污点的设置、查看和移除
```bash
#设置污点
$ kubectl taint nodes node1 key1=value1:NoSchedule

#节点说明中，查找Taints字段
$ kubectl describe pod pod-name

# 去除污点
$ kubectl taint nodes node1 key1:NoSchedule-
```

### 容忍
既然污点是针对node，那么容忍则是针对pod。pod如果能容忍污点，那么pod就可以运行

通过 pod.spec.tolerations 配置

例如
```yaml
toleration:
    - key:"key1"
      oprator: "Equal"
      value: "value1"
      effect: "NoSchedule"
      tolerationSeconds: 8000
    - key:"key1"
      oprator: "Equal"
      value: "value1"
      effect: "NoExecute"
```

- 其中key，value,effect 要与node上设置的taint保持一致
- operator 的值为Exists将会忽略value的值
- tolerationSeconds 用于描述当pod需要呗驱逐时可以在pod上继续保留运行的时间

**当不指定key时，表示容忍所有的污点key，当不指定effect时，表示容忍所有的污点作用**