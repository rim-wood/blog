---
title: k8s 存储
date: 2019-09-12 19:30:02
toc: true
tags: 
- k8s
- configmap
- nfs
- volumes
categories:
- k8s
---
k8s 存储是一个对于pod的一个外置存储对象，通过pod--->pvc--->pv--->物理存储的一个过程
<!--more-->

# 存储

## ConfigMap
应用程序会从配置文件、命令行参数或者环境变量中读取配置信息。ConfigMap 提供了一个可以向容器注入配置信息的机制。

**可以保存单个属性，也可以通过文件创建**
```bash
[node1@localhost ~]$ cat application.yaml
server:
    port: 8080
    undertow:
        accesslog:
          dir: 
          enabled: false  
          pattern: common   
          prefix: access_log  
          rotate: true 
          suffix: log   

[node1@localhost ~]$ kubectl create configmap app-config --from-file=application.yaml
```
--from-file 这个参数可以多次使用，可以使用两次分别指定两个不同的文件

```bash
[node1@localhost ~]$ kubectl create configmap app-config --from-literal=application.server=aaa --from-literal=application.port=80
[node1@localhost ~]$ kubectl get configmaps app-config -o yaml
```
--from-literal 可以通过key-value参数配置，使用多次

### pod中使用
1. 使用configmap代替**环境变量**,可以通过env标签变量一一对应，也可以通过envFrom直接引用configmap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  application.server: aaa
  application.port: 80
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: env-config
  namespace: default
data:
  log_level: INFO
---
apiVersion: v1
kind: Pod
metadata:
  name: app1
spec:
  containers:
    - name: test-container
      image: ice.com/app1:v1
      command: ["bin/sh","-c","env"]
      env:
        - name: APPLICATION_SERVER
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: application.server
        - name: APPLICATION_PORT
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: application.port
      envFrom:
        - configMapRef:
          name: env-config
  restartPolicy: Never
```
可以通过以下命令查看
```bash
[node1@localhost ~]$ kubectl describe cm app-config
```
2. 用configmap设置命令行参数,可以使用
```
$(环境变量) 例如 $(APPLICATION_SERVER)
```

3. 通过数据卷插件
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app1
spec:
  containers:
    - name: test-container
      image: ice.com/app1:v1
      command: ["bin/sh","-c","cat /etc/config/application.server"]
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: app-config
  restartPolicy: Never
```
### configmap 热更新

假设现在有一个deployment引用了configmap，可以通过命令
```bash
[node1@localhost ~]$ kubectl edit configmap app-config
```
来对configmap进行修改。
**需要注意的是deployment中的pod并不会触发更新，重新加载configmap**
要达到滚动更新可以使用
```bash
[node1@localhost ~]$ kubectl patch deployment app1 --patch '{"spec":{"template":{"metadata":{"annotations":{"version/config":"20191101"}}}}}'
```
这个例子主要是通过 spec.template.metadata.annotations 中添加version/config,每次通过修改version/config 来触发deployment的滚动更新

**注意使用configmap挂载的volume中的数据需要大概10秒的时间才能同步更新**

## Secret
Secret解决了密码、token、秘钥等敏感数据的配置问题，使用方式跟configmap类似

Secret有三种类型：
- **Service Account:** 由k8s自动创建，并挂在/run/secrets/kuberbetes.io/serviceaccount目录中
- **Opaque:** base64编码格式的Secret，用来存储密码、秘钥等，使用之前要先对明文进行base64加密 如：echo -n "icepear" | base64
- **kubernetes.io/dockerconfigjson:** 用来存储私有docker registry的认证信息,可以通过如下命令创建
```bash
kubectl create secret docker-registry ice-harbor --docker-server=reg.icepear.cn --docker-username=admin --docker-password=Harbor123456 --docker-email=a@icepear.cn
```
然后yaml文件通过imagePullSecrets: - name: ice-harbor 添加认证

## Volume
**Volume的生命周期跟pod一致**

### emptyDir
- 临时空间，创建时为空的目录
- 容器崩溃时不会删除

样例
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app1
spec:
  containers:
    - name: test-container
      image: ice.com/app1:v1
      volumeMounts:
        - name: cache-volume
          mountPath: /etc/config
  volumes:
    - name: cache-volume
      emptyDir:{}
```

### hostPath
**hostPath 可以理解为跟docker 的volume挂载方式一样**

- 运行需要访问docker内部的容器，使用 /var/lib/docker 的hostPath
- 在容器中运行cAdvisor，使用 /dev/cgroups 的hostPath

可以指定type标签


值 | 行为
---|---
空字符串 | 表示不检查
DirectoryDrCreate | 如果在给定的路径上没有任何东西存在，那么将创建一个755权限的空目录
Directory | 给定路径下必须存在目录
FileOrCreate | 如果在给定的路径上没有任何东西存在，那么将创建一个644权限的空文件
File | 给定的路径下必须存在文件
Socket | 给定的路径下必须存在UNIX套接字
CharDevice | 给定路径下必须存在字符设备
BlockDevice | 给定的路径下必须存在块设备

- 由于每个节点上的文件都不同，具有相同配置的pod在不同节点上的行为可能会有所不同
- 在底层主机上创建的文件或目录只能由root写入，您需要在特权容器中以root身份运行进程，或修改主机文件或目录的权限供k8s访问写入

样例
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app1
spec:
  containers:
    - name: test-container
      image: ice.com/app1:v1
      volumeMounts:
        - name: path-volume
          mountPath: /etc/config
  volumes:
    - name: path-volume
      hostPath:
        path: /config
        type: Directory
```
## PV-PVC

PV也是以插件的形式实现，支持第三方的云商提供的各类插件，也支持本地的存储插件如nfs、hostpath等

PVC则是一种资源规定，定义好之后会按需求自动寻找合适的PV进行存储，也就是说存在这样一种关系：
```
graph TD
Pod-->PVC
PVC-->PV
```

### PV 访问模式

- ReadWriteOnce（RWO）单节点读/写模式挂载
- ReadOnlyMany（ROX）多节点只读模式挂载
- ReadWriteMany（RWX）多节点读/写模式挂载

### 回收策略

- Retain（保留）手动回收
- Recycle（回收）基本擦除
- Delete（删除）关联的存储资产将被删除
- 只有NFS和HostPath支持回收策略，AWS EBS、GCE PD、Azure Disk支持删除策略

### 状态

- Available(可用) 一块空闲资源还没有被任何声明绑定
- Bound(已绑定) 卷已被绑定
- Released(已释放) 声明被删除，但是资源还没有被集群重新声明
- Failed(失败) 自动回收失效

### nfs服务器创建

1. 服务端安装nfs，启动服务
```bash
#安装服务
yum install -y nfs-common nfs-utils rpcbind 

#启动服务
systemctl start rpcbind
systemctl start nfs

#开机自启
systemctl enable rpcbind
systemctl enable nfs
```
2. 服务端配置
```bash
#创建文件
mkdir /data
chmod 755 /data
chown -R nfsnobody:nfsnobody /data

#配置/etc/exports
vi /etc/exports
#写入配置
/data *(rw,sync,all_squash,no_root_squash)
#执行后 使配置生效
exportfs -r 
```
nfs配置文件的格式：

NFS共享的目录 NFS客户端地址（参1，参2，……） NFS客户端地址2（参1，参2，……）

NFS共享的目录 NFS客户端地址（参1，参2，……）

3. 客服端安装启动服务
```bash
#安装服务
yum install -y nfs-utils rpcbind 
```

4. 客户端简单使用
```bash
#查看nfs服务器共享目录 showmount -e nfs服务器IP
showmount -e 192.168.110.11
#挂载至本机目录 mount -t nfs 服务器IP:/目录 本机目录
mount -t nfs 192.168.110.11::/data /test
```

### PV、PVC 样例

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-pv
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow #存储类名称，划分存储类指标
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /temp
    server: 192.168.110.15
```

### pv 扩容

在kubernetes 1.11版本中开始支持pvc创建后的扩容

storageclass可以根据pvc自动创建pv，所以在创建storageclass时，需要添加参数

例如
```yaml

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: glusterfs-sc
parameters:
  clusterid: 075e35a0ce70274b3ba7f158e77edb2c
  resturl: http://172.16.9.201:8087
  volumetype: replicate:3
provisioner: kubernetes.io/glusterfs
reclaimPolicy: Delete
volumeBindingMode: Immediate

allowVolumeExpansion: true  # 重要

```
