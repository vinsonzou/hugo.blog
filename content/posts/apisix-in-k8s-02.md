---
title: "APISIX在k8s的使用02: 升级至3.1.0"
subtitle: ""
date: 2023-01-01T13:06:00+08:00
lastmod: 2023-01-01T13:06:00+08:00
description: ""

tags: ["Kubernetes","APISIX"]
categories: ["Kubernetes","APISIX"]
---

## 1、环境

- k8s环境信息
  - 私有镜像地址：`dockerhub.kubekey.local`
  - k8s: 1.24.3
  - LoadBalancer组件: openelb
- [APISIX Helm](https://github.com/apache/apisix-helm-chart)组件信息
  - APISIX: 0.12.3
    - etcd: 3.5.4
  - APISIX ingress-controller: 0.11.3
  - APISIX Dashboard: 0.7.0
  - 需更新关联镜像
    - apache/apisix:3.1.0-debian
    - apache/apisix-ingress-controller:1.6.0
    - apache/apisix-dashboard:2.15-alpine

## 2、更新

### 2.1 APISIX及ingress-controller更新

```shell
helm install apisix \
  --namespace ingress-apisix \
  --create-namespace \
  --set global.imageRegistry=dockerhub.kubekey.local \
  --set etcd.image.repository=3rd/etcd \
  --set apisix.image.repository=dockerhub.kubekey.local/3rd/apisix \
  --set initContainer.image=dockerhub.kubekey.local/library/busybox \
  --set gateway.type=LoadBalancer \
  --set gateway.annotations."eip\.openelb\.kubesphere\.io/v1alpha2"=eip-layer2-pool \
  --set gateway.annotations."lb\.kubesphere\.io/v1alpha1"=openelb \
  --set gateway.annotations."protocol\.openelb\.kubesphere\.io/v1alpha1"=layer2 \
  --set gateway.stream.enabled=true \
  --set ingressPublishService="ingress-apisix/apisix-gateway" \
  --set admin.allow.ipList.1="127.0.0.1/24" \
  --set admin.allow.ipList.2="10.233.64.0/18" \
  --set ingress-controller.enabled=true \
  --set ingress-controller.image.repository=dockerhub.kubekey.local/3rd/apisix-ingress-controller \
  --set ingress-controller.initContainer.image=dockerhub.kubekey.local/library/busybox \
  --set ingress-controller.config.apisix.adminAPIVersion="v3" \  # apisix > 3.0, 此选项必须为v3
  --set ingress-controller.config.apisix.serviceNamespace=ingress-apisix \
  --set ingress-controller.config.apisix.kubernetes.enableGatewayAPI=true \ # 此选项还需额外导入GatewayClass CRD才能生效
  apisix-0.12.3.tgz
```

> 升级后变化: apisix configMap被重置了

### 2.2 dashboard更新

```shell
helm upgrade apisix-dashboard \
  --namespace ingress-apisix \
  --set image.repository=dockerhub.kubekey.local/3rd/apisix-dashboard \
  --set config.conf.etcd.endpoints.1=apisix-etcd.ingress-apisix.svc.cluster.local:2379 \
  apisix-dashboard-0.7.0.tgz
```

## 3、APISIX配置

apisix自身配置都保存在configMap中。

- 修改`stream_proxy`监听端口
  ```
  apisix:
    stream_proxy:       # TCP/UDP proxy
      only: false
      tcp:              # TCP proxy port list
        - 8201
        - 8202
      udp:              # UDP proxy port list
        - 8201
        - 8202
  ```
  - 同时配置`k8s Service`之`apisix-gateway`的端口监听
    - 发布tcp/udp 8201,8202对外
- 开启基于 gRPC 的 etcd 配置同步
  ```
  deployment:
    etcd:
      use_grpc: true
  ```
- 开启插件server-info
  ```
  plugin:
    - server-info
  ```

> ps: 修改ConfigMap后，记得更新apisix deployment，否则配置不会生效。
