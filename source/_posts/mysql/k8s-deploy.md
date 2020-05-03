---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/other/mysql-conf/mysql.png
title: mysql 部署在k8s时出现问题
date: 2020-05-02 14:30:02
toc: true
tags: 
- Mysql
- k8s
categories:
- mysql
---
最近对mysql部署到k8s时，碰到一个问题，报错说，数据库初始化时，目录文件中存在文件。辗转issue、StackOverflow都没有解决，最后采取蠢办法解决，在此记录一下过程
<!--more-->

# story故事

因为要将应用使用helm快捷部署至k8s，所以必不可少的需要mysql，就找到helm官方的mysql charts.[链接](https://hub.helm.sh/charts/stable/mysql)
但是在官方charts中并未发现有storageclass存在，但是看说明又是支持storageclass的，所以就自己定义了一个storageclass，而pv的提供者是自己用nfs-client-provisioner搭建的共享存储。
然后错误就出现了

# 重现

## 下载charts

找到官方的charts源码，并[下载](https://github.com/helm/charts/tree/master/stable/mysql)，然后在templates文件夹下建立storageclass.yaml

```yaml
{{- if and .Values.persistence.enabled (not .Values.persistence.existingClaim) -}}
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: {{ .Values.persistence.storageClass.storageClassName }}
provisioner: 
{{ toYaml .Values.persistence.storageClass.provisioner | indent 2 }}
parameters:
{{ toYaml .Values.persistence.storageClass.classParameters | indent 2 }}
reclaimPolicy: Retain
allowVolumeExpansion: true
{{- end }}
```

## 启动

1. 启动

```bash
$ helm install --namespace mysql-dev --name mysql-01 ./
```

2. 查看pod

```yaml
$ kubectl get pods -n mysql-dev
```
发现pod一直处于CrashLoopBackOff的状态，

3. 查看日志

```yaml
$ kubectl logs pod名称
```

发现提示错误

```bash
[ERROR] --initialize specified but the data directory has files in it. Aborting.
[ERROR] Aborting
```

## 搜索

然后就开始了一些列的搜索，首先到官方github上提[issue](https://github.com/helm/charts/issues/21293)，然后Stack Overflow上找找找····，结果还是没有解决办法。


# 解决

万念俱灰的时候，想到是否可以尝试手动创建pv的方式，是否真的是nfs-client-provisioner创建的pv有问题，跟mysql不兼容。

然后我就手动创建了hostpath方式的一个pv

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  hostpath:
    path: "/data/k8s/mysql"
```

然后把pvc改成pv的方式，不用storageclass，发现部署成功了。难道真的是nfs方式不支持吗？小伙伴不知道有没有同样的情况，欢迎评论区交流。

**最后，是不是说像mysql这类存储型的中间件，采取hostpath的方式对于I/O来说其实是最优的选择？**
