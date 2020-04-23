---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/k8s/k8s.png
title: helm 总结
date: 2019-09-20 20:30:02
toc: true
tags: 
- k8s
- helm
categories:
- k8s
---
helm分为helm客户端和tiller服务器。k8s的部署如果用kubectl一个个创建显得太繁琐了；为了让k8s像docker compose一样快速启动一组pod，就有了helm。
helm客户端负责chart和release的创建和管理以及和tiller的交互，而tiller服务器则运行在k8s集群中，他负责处理helm客户端的请求，然后转化为 KubeApiServer 的请求跟k8s交互
<!--more-->
# helm
helm分为helm客户端和tiller服务器。k8s的部署如果用kubectl一个个创建显得太繁琐了；为了让k8s像docker compose一样快速启动一组pod，就有了helm。

helm客户端负责chart和release的创建和管理以及和tiller的交互，而tiller服务器则运行在k8s集群中，他负责处理helm客户端的请求，然后转化为 KubeApiServer 的请求跟k8s交互

- **chart** 是创建一个应用的信息集合，包含各种k8s的对象的配置模板、参数定义、依赖关系、文档说明等
- **release** 是chart 的运行实例，代表了一个正在运行的应用。当chart被安装到k8s集群，就生成了一个release，chart能多次安装到同一个集群，每次安装都是一个release

## helm 部署

### 客户端

[下载客户端](https://www.rancher.cn/docs/rancher/v2.x/cn/install-prepare/download/helm/)

下载了之后解压

    sudo tar -zxvf helm-v2.16.1-linux-amd64.tar.gz

解压之后放到linux执行目录下，修改权限

    sudo cp linux-amd64/helm /usr/local/bin/
    sudo chmod a+x /usr/local/bin/helm

输入 helm 显示提示信息则安装成功

### 安装tiller服务端
安装tiller服务器，还需要在机器上配置好kubectl工具和kubeconfig文件，确保kubectl工具可以在这台机器上访问apiserver切正常使用

因为k8s apiserver开启了rbac的访问控制，所以需要创建tiller使用的service account：tiller并分配合适的角色给他，可以查看helm的文档 [link](https://helm.sh/docs/topics/chart_best_practices/rbac/)

创建clusterRoleBinding
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```
创建
```bash
kubectl create -f rbac-config.yaml
```
初始化tiller
```bash
helm init --service-acount tiller --skip-refresh
```

这个过程拉取镜像可能有问题，会被墙掉，可以自行去dockerhub中找

**gcr.io.kubernetes-helm.tiller:v2.16.1** 版本可以通过kubectl describe 看下是什么，要下载对应的

然后通过改tag让容器运行起来
```bash
docker tag fishead/gcr.io.kubernetes-helm.tiller:v2.16.1 gcr.io/kubernetes-helm/tiller:v2.16.1
```

至此helm就安装好了

## helm 使用

helm [仓库地址](https://hub.helm.sh/)

### 创建chart包

当前目录创建一个 myapp chart包
```bash
$ helm create myapp
```

创建完之后，目录结构如下

```
myapp            - chart 包目录名
├── charts       - 依赖的子包目录，里面可以包含多个依赖的chart包
├── Chart.yaml   - chart定义，可以定义chart的名字，版本号信息。
├── templates    - k8s配置模版目录， 我们编写的k8s配置都在这个目录， 除了NOTES.txt和下划线开头命名的文件，其他文件可以随意命名。
│   ├── deployment.yaml
│   ├── _helpers.tpl    -下划线开头的文件，helm视为公共库定义文件，主要用于定义通用的子模版、函数等，helm不会将这些公共库文件的渲染结果提交给k8s处理。
│   ├── ingress.yaml
│   ├── NOTES.txt       -chart包的帮助信息文件，执行helm install命令安装成功后会输出这个文件的内容。
│   └── service.yaml
└── values.yaml         -chart包的参数配置文件，模版可以引用这里参数。
```

### 部署应用
通过命令 **helm install app文件夹路径**
```bash
$ helm install /myapp
```

### 更新应用
通过命令 **helm upgrade app名称 app文件夹路径**
```bash
$ helm upgrade myapp /myapp
```

### 删除应用
```bash
$ helm delete myapp 
```
但是helm还是会保留已经删除的chart的历史版本，当你重新创建相同名称的chart时，会报错
```
Error: a release named nacos already exists.
Run: helm ls --all nacos; to check the status of the release
Or run: helm del --purge nacos; to delete it
```
如果想彻底删除镜像可以使用
```bash
$ helm delete --purge myapp 
```