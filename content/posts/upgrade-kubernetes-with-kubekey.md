---
title: "KubeSphere-4: 使用 KubeKey 离线升级 Kubernetes"
subtitle: ""
date: 2022-11-20T13:30:00+08:00
lastmod: 2022-11-20T13:30:00+08:00
description: ""

tags: ["Kubernetes","KubeSphere"]
categories: ["Kubernetes"]
---

为了修复最近Kubernetes 敏感信息泄露漏洞[CVE-2022-3162](https://github.com/kubernetes/kubernetes/issues/113756)。需升级 Kubernetes 到 1.22.16 或 1.23.14 或 1.24.8 或 1.25.4。

本文就是使用 KubeKey 离线升级 Kubernetes至1.25.4的全过程。

## 升级准备

### 1. 环境信息

- OS: AlmaLinux 9.0
- Kubernetes: 1.24.3 ==> 1.25.4
  - containerd: 1.6.4
- KubeSphere: 3.3.1
- Kubekey: 3.0.1
- Harbor: 2.5.3

### 2. 离线升级所需artifact清单

`k8s-v1.25.4-upgrade-manifest.yaml`

```yaml
apiVersion: kubekey.kubesphere.io/v1alpha2
kind: Manifest
metadata:
  name: sample
spec:
  arches:
  - amd64
  operatingSystems:
  - arch: amd64
    type: linux
    id: almalinux
    version: "9.0"
    osImage: AlmaLinux 9.0
    repository:
      iso:
        localPath: /app/k8s/kk-3.0.1/almalinux-9.0-rpms-amd64.iso
        url:
  kubernetesDistributions:
  - type: kubernetes
    version: v1.25.4
  components:
    helm:
      version: v3.9.0
    cni:
      version: v0.9.1
    etcd:
      version: v3.4.13
    containerRuntimes:
    - type: containerd
      version: 1.6.4
    crictl:
      version: v1.24.0
    docker-registry:
      version: "2"
    harbor:
      version: v2.5.3
    docker-compose:
      version: v2.2.2
  images:
  # k8s-images
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-apiserver:v1.25.4
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-controller-manager:v1.25.4
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-proxy:v1.25.4
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-scheduler:v1.25.4
  - registry.cn-beijing.aliyuncs.com/kubesphereio/pause:3.8
  - registry.cn-beijing.aliyuncs.com/kubesphereio/coredns:v1.9.3
```

### 3. 导出制品 artifact

```shell
export KKZONE=cn
./kk artifact export -m k8s-v1.25.4-upgrade-manifest.yaml -o k8s-v1.25.4-upgrade-artifact.tar.gz
```

> 青云国内未及时同步最新kubelet，导致kk命令无法下载，此处手动下载kubelet 1.24
> - curl -L -o /app/k8s/kk-3.0.1/kubekey/artifact/kube/v1.25.4/amd64/kubelet https://dl.k8s.io/v1.25.4/bin/linux/amd64/kubelet

## 离线升级

### 1. 拷贝安装文件至离线环境

将下载的 KubeKey 和制品 artifact 通过 U 盘等介质拷贝至离线环境安装节点。

修改kubernetes升级版本

```yaml
spec:
  kubernetes:
    version: v1.25.4    # 升级的版本
    clusterName: cluster.local
    autoRenewCerts: true
    containerManager: containerd
```

### 2. 推送离线镜像至 Harbor 仓库

```shell
./kk artifact image push -f config-sample.yaml -a k8s-v1.25.4-upgrade-artifact.tar.gz
```

### 3. 升级 Kubernetes

```shell
./kk upgrade -f config-sample.yaml -a k8s-v1.25.4-upgrade-artifact.tar.gz
```
