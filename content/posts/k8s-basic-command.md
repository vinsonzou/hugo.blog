---
title: "k8s查看集群信息及基本命令"
subtitle: ""
date: 2022-09-29T15:11:03+08:00
lastmod: 2022-09-29T15:11:03+08:00
description: ""

tags: ["Kubernetes"]
categories: ["Kubernetes"]
---

集群信息的各种查看基本上是在Master节点操作

- 查看 Node状态
```
kubectl get nodes
kubectl get node IP //节点IP可以用空格隔开写多个
```

- 查看 Service 信息
```
kubectl get service
```

- 查看所有名称空间内资源
```
kubectl get pods --all-namespaces
或者
kubectl get pods -A
```

- 同时查看多种资源信息
```
kubectl get pod,svc -n kube-system
```

- 查看 API 对象细节
使用 kubectl describe 命令，查看一个 API 对象的细节：
```
kubectl describe node IP
kubectl delete node IP
```

- 查看集群信息
```
kubectl cluster-info
Kubernetes master is running at http://localhost:8080
```

- 查看各组件信息
```
使用安全连接：
kubectl -s https://URL get componentstatuses
未使用安全连接
kubectl -s http://URL get componentstatuses
```

- 查看资源类型所对应的Apiversion
```
kubectl explain pod
```

- 查看帮助
```
kubectl explain deployment
kubectl explain deployment.spec
kubectl explain deployment.spec.replicas
```
