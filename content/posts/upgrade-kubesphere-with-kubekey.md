---
title: "KubeSphere-3: 使用 KubeKey 离线升级 KubeSphere"
subtitle: ""
date: 2022-10-28T12:16:00+08:00
lastmod: 2022-10-28T12:16:00+08:00
description: ""

tags: ["Kubernetes","KubeSphere"]
categories: ["Kubernetes"]
---

## 升级准备

### 1. 环境信息

- OS: AlmaLinux 9.0
- Kubernetes: 1.24.3
  - containerd: 1.6.4
- KubeSphere: 3.3.1
- Kubekey: 2.3.0
- Harbor: 2.5.3

### 2. kubekey离线所需artifact清单

`ks-v3.3.1-manifest.yaml`

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
        localPath: /app/k8s/3.3.1/almalinux-9.0-rpms-amd64.iso
        url:
  kubernetesDistributions:
  - type: kubernetes
    version: v1.24.3
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
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-apiserver:v1.24.3
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-controller-manager:v1.24.3
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-proxy:v1.24.3
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-scheduler:v1.24.3
  - registry.cn-beijing.aliyuncs.com/kubesphereio/pause:3.7
  - registry.cn-beijing.aliyuncs.com/kubesphereio/coredns:1.8.6
  - registry.cn-beijing.aliyuncs.com/kubesphereio/k8s-dns-node-cache:1.15.12
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-controllers:v3.23.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/cni:v3.23.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/node:v3.23.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/pod2daemon-flexvol:v3.23.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/typha:v3.23.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/provisioner-localpv:3.3.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/flannel:v0.12.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/linux-utils:3.3.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/haproxy:2.3
  - registry.cn-beijing.aliyuncs.com/kubesphereio/nfs-subdir-external-provisioner:v4.0.2
  # cilium
  - registry.cn-beijing.aliyuncs.com/kubesphereio/cilium:v1.11.6
  - registry.cn-beijing.aliyuncs.com/kubesphereio/operator-generic:v1.11.6
  # kubesphere-images
  - registry.cn-beijing.aliyuncs.com/kubesphereio/ks-installer:v3.3.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/ks-apiserver:v3.3.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/ks-console:v3.3.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/ks-controller-manager:v3.3.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kubectl:v1.22.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kubefed:v0.8.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/tower:v0.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/snapshot-controller:v4.0.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/nginx-ingress-controller:v1.1.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/defaultbackend-amd64:1.4
  - registry.cn-beijing.aliyuncs.com/kubesphereio/metrics-server:v0.4.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/redis:6.2.6-alpine
  - registry.cn-beijing.aliyuncs.com/kubesphereio/haproxy:2.0.25-alpine
  - registry.cn-beijing.aliyuncs.com/kubesphereio/alpine:3.14
  - registry.cn-beijing.aliyuncs.com/kubesphereio/openldap:1.3.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/netshoot:v1.0
  # kubesphere插件-应用商店
  - registry.cn-beijing.aliyuncs.com/kubesphereio/openpitrix-jobs:v3.3.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/minio:RELEASE.2019-08-07T01-59-21Z
  - registry.cn-beijing.aliyuncs.com/kubesphereio/mc:RELEASE.2019-08-07T23-14-43Z
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-rbac-proxy:v0.11.0
  # 其他
  - registry.cn-beijing.aliyuncs.com/kubesphereio/cloudcore:v1.9.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/iptables-manager:v1.9.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/edgeservice:v0.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/gatekeeper:v3.5.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/devops-apiserver:v3.3.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/devops-controller:v3.3.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/devops-tools:v3.3.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/ks-jenkins:v3.3.0-2.319.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/inbound-agent:4.10-2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/builder-base:v3.2.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/builder-maven:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/s2ioperator:v3.2.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/s2irun:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/s2i-binary:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/java-8-centos7:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/java-8-runtime:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/argocd:v2.3.3
  - registry.cn-beijing.aliyuncs.com/kubesphereio/argocd-applicationset:v0.4.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/dex:v2.30.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/configmap-reload:v0.5.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/prometheus:v2.34.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/prometheus-config-reloader:v0.55.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/prometheus-operator:v0.55.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-state-metrics:v2.5.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/node-exporter:v1.3.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/alertmanager:v0.23.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/thanos:v0.25.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/grafana:8.3.3
  - registry.cn-beijing.aliyuncs.com/kubesphereio/notification-manager-operator:v1.4.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/notification-manager:v1.4.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/notification-tenant-sidecar:v3.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/elasticsearch-curator:v5.7.6
  - registry.cn-beijing.aliyuncs.com/kubesphereio/elasticsearch-oss:6.8.22
  - registry.cn-beijing.aliyuncs.com/kubesphereio/fluentbit-operator:v0.13.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/docker:19.03
  - registry.cn-beijing.aliyuncs.com/kubesphereio/fluent-bit:v1.8.11
  - registry.cn-beijing.aliyuncs.com/kubesphereio/log-sidecar-injector:1.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/filebeat:6.7.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-events-operator:v0.4.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-events-exporter:v0.4.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-events-ruler:v0.4.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-auditing-operator:v0.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kube-auditing-webhook:v0.2.0
  - registry.cn-beijing.aliyuncs.com/kubesphereio/pilot:1.11.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/proxyv2:1.11.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/jaeger-operator:1.27
  - registry.cn-beijing.aliyuncs.com/kubesphereio/jaeger-agent:1.27
  - registry.cn-beijing.aliyuncs.com/kubesphereio/jaeger-collector:1.27
  - registry.cn-beijing.aliyuncs.com/kubesphereio/jaeger-query:1.27
  - registry.cn-beijing.aliyuncs.com/kubesphereio/jaeger-es-index-cleaner:1.27
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kiali-operator:v1.38.1
  - registry.cn-beijing.aliyuncs.com/kubesphereio/kiali:v1.38
  - registry.cn-beijing.aliyuncs.com/kubesphereio/scope:1.13.0
```

### 3. 导出制品 artifact

```shell
export KKZONE=cn
./kk artifact export -m ks-v3.3.1-manifest.yaml -o kubesphere-v3.3.1-artifact.tar.gz
```

> 制品（artifact）是一个根据指定的 manifest 文件内容导出的包含镜像 tar 包和相关二进制文件的 tgz 包。在 KubeKey 初始化镜像仓库、创建集群、添加节点和升级集群的命令中均可指定一个 artifact，KubeKey 将自动解包该 artifact 并在执行命令时直接使用解包出来的文件。
>
> - 导出时请确保网络连接正常。
> - KubeKey 会解析镜像列表中的镜像名，若镜像名中的镜像仓库需要鉴权信息，可在 manifest 文件中的 **.registry.auths** 字段中进行配置。

## 离线升级

### 1. 拷贝安装文件至离线环境

将下载的 KubeKey 和制品 artifact 通过 U 盘等介质拷贝至离线环境安装节点。

修改kubesphere升级版本

```yaml
---
apiVersion: installer.kubesphere.io/v1alpha1
kind: ClusterConfiguration
metadata:
  name: ks-installer
  namespace: kubesphere-system
  labels:
    version: v3.3.1     # 升级的版本
```

### 2. 推送离线镜像至 Harbor 仓库

```shell
./kk artifact image push -f config-sample.yaml -a kubesphere-v3.3.1-artifact.tar.gz
```

### 3. 升级 KubeSphere

```shell
./kk upgrade -f config-sample.yaml -a kubesphere-v3.3.1-artifact.tar.gz
```

发现无法从 `3.3.1-rc.2` 升级至 `3.3.1`，使用如下方式处理

- 查看当前kubesphere image版本
```shell
[root@master01 ~]# kubectl -n kubesphere-system get deploy -o wide
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS              IMAGES                                                                    SELECTOR
ks-apiserver            1/1     1            1           15d   ks-apiserver            dockerhub.kubekey.local/kubesphereio/ks-apiserver:v3.3.1-rc.2             app=ks-apiserver,tier=backend
ks-console              1/1     1            1           15d   ks-console              dockerhub.kubekey.local/kubesphereio/ks-console:v3.3.1-rc.2               app=ks-console,tier=frontend
ks-controller-manager   1/1     1            1           15d   ks-controller-manager   dockerhub.kubekey.local/kubesphereio/ks-controller-manager:v3.3.1-rc.2    app=ks-controller-manager,tier=backend
ks-installer            1/1     1            1           15d   installer               dockerhub.kubekey.local/kubesphereio/ks-installer:v3.3.1-rc.2             app=ks-installer
```

- 更新镜像
```shell
kubectl -n kubesphere-system set image deploy ks-installer installer=dockerhub.kubekey.local/kubesphereio/ks-installer:v3.3.1
```

- 升级后kubesphere image版本
```shell
[root@master01 ~]# kubectl -n kubesphere-system get deploy -o wide
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS              IMAGES                                                                    SELECTOR
ks-apiserver            1/1     1            1           15d   ks-apiserver            dockerhub.kubekey.local/kubesphereio/ks-apiserver:v3.3.1             app=ks-apiserver,tier=backend
ks-console              1/1     1            1           15d   ks-console              dockerhub.kubekey.local/kubesphereio/ks-console:v3.3.1               app=ks-console,tier=frontend
ks-controller-manager   1/1     1            1           15d   ks-controller-manager   dockerhub.kubekey.local/kubesphereio/ks-controller-manager:v3.3.1    app=ks-controller-manager,tier=backend
ks-installer            1/1     1            1           15d   installer               dockerhub.kubekey.local/kubesphereio/ks-installer:v3.3.1             app=ks-installer
```

- 如未生效，可重启 `ks-installer` 的 deployment
```shell
kubectl -n kubesphere-system rollout restart deploy ks-install
```
