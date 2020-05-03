---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/k8s/k8s.png
title: k8s storageclass 自动创建pv
date: 2020-04-30 20:30:02
toc: true
tags: 
- storageclass
categories:
- k8s
---
在k8s运维的过程中，针对于各种应用都会产生数据，而数据挂载需要pv来存储，我们必须一个一个手动来创建pv，这对运维来说是不够合理，storageclass正好就是干这事的，就是根据pvc的要求，结合第三方应用自动创建合适的pv
<!--more-->

# StorageClass

 前面讲的 PV 都是静态的，意思就是我要使用的一个 PVC 的话就必须手动去创建一个 PV，我们也说过这种方式在很大程度上并不能满足我们的需求，比如我们有一个应用需要对存储的并发度要求比较高，而另外一个应用对读写速度又要求比较高，特别是对于 StatefulSet 类型的应用简单的来使用静态的 PV 就很不合适了，这种情况下我们就需要用到动态 PV，也就是StorageClass。

## 安装

要使用StorageClass自动创建pv，就得安装对应的自动配置程序。这个程序叫做 **nfs-client-provisioner**，需要结合我们上面讲的nfs来使用，创建pv的时候有两个特性
- 自动创建的 PV 以${namespace}-${pvcName}-${pvName}这样的命名格式创建在 NFS 服务器上的共享数据目录中
- 而当这个 PV 被回收后会以archieved-${namespace}-${pvcName}-${pvName}这样的命名格式存在 NFS 服务器上。

下面我们分三步来创建这个自动配置程序：

### 1.创建 ServiceAccount

现在的 Kubernetes 集群大部分是基于 RBAC 的权限控制，所以创建一个一定权限的 ServiceAccount 与后面要创建的 “NFS Provisioner” 绑定，赋予一定的权限。

nfs-rbac.yaml
```yaml
kind: ServiceAccount
apiVersion: v1
metadata:
  name: nfs-client-provisioner
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: default      #替换成你要部署NFS Provisioner的 Namespace
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: default      #替换成你要部署NFS Provisioner的 Namespace
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
```

创建 RBAC
```bash
$ kubectl apply -f nfs-rbac.yaml
```

### 2.部署nfs-client-provisioner的Deployment

nfs-provisioner-deploy.yaml
```yaml
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: nfs-client-provisioner
spec:
  replicas: 1
  strategy:
    type: Recreate   #---设置升级策略为删除再创建(默认为滚动更新)
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: quay.io/external_storage/nfs-client-provisioner:latest
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: icepear.cn/ifs #--- nfs-provisioner的名称，以后设置的storageclass要和这个保持一致
            - name: NFS_SERVER
              value: 192.168.110.15  #---NFS服务器地址，和 valumes 保持一致
            - name: NFS_PATH
              value: /data/nfs-share #---NFS服务器目录，和 valumes 保持一致
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.110.15  #---NFS服务器地址
            path: /data/nfs-share #---NFS服务器目录
```
创建 NFS Provisioner
```bash
$ kubectl apply -f nfs-provisioner-deploy.yaml -n default
```

### 3.创建SotageClass

nfs-storage.yaml

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-storage
  annotations:
    storageclass.kubernetes.io/is-default-class: "true" #---设置为默认的storageclass
provisioner: icepear.cn/ifs #---动态卷分配者名称，必须和上面创建的"provisioner"变量中设置的Name一致
parameters:
  archiveOnDelete: "true"  #---设置为"false"时删除PVC不会保留数据,"true"则保留数据
```

### 4.测试storegeclass

1. 创建测试 PVC

test-pvc.yaml
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-pvc
spec:
  storageClassName: nfs-storage #---需要与上面创建的storageclass的名称一致
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Mi
```

创建 PVC

```bash
$ kubectl apply -f test-pvc.yaml -n default
```

查看 PVC 状态是否与 PV 绑定

利用 Kubectl 命令获取 pvc 资源，查看 STATUS 状态是否为 “Bound”。

```bash
$ kubectl get pvc test-pvc -n default

NAME       STATUS   VOLUME                 CAPACITY   ACCESS MODES   STORAGECLASS
test-pvc   Bound    pvc-be0808c2-9957-11e9 1Mi        RWO            nfs-storage
```

2. 创建测试 Pod 并绑定 PVC

创建一个测试用的 Pod，指定存储为上面创建的 PVC，然后创建一个文件在挂载的 PVC 目录中，然后进入 NFS 服务器下查看该文件是否存入其中。

test-pod.yaml
```
kind: Pod
apiVersion: v1
metadata:
  name: test-pod
spec:
  containers:
  - name: test-pod
    image: busybox:latest
    command:
      - "/bin/sh"
    args:
      - "-c"
      - "touch /mnt/SUCCESS && exit 0 || exit 1"  #创建一个名称为"SUCCESS"的文件
    volumeMounts:
      - name: nfs-pvc
        mountPath: "/mnt"
  restartPolicy: "Never"
  volumes:
    - name: nfs-pvc
      persistentVolumeClaim:
        claimName: test-pvc
```
创建 Pod
```bash
$ kubectl apply -f test-pod.yaml -n default
```

3. 查看nfs服务器上是否创建的了文件

```bash
$ cd /data/nfs-share
$ ls

```

