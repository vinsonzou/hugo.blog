---
title: "KubeSphere系列二: 使用 KubeKey 离线安装 Cilium 替代 kube-proxy"
subtitle: ""
date: 2022-10-15T14:24:48+08:00
lastmod: 2022-10-15T14:24:48+08:00
description: ""

tags: ["Kubernetes","KubeSphere"]
categories: ["Kubernetes", "Cilium"]
---

详细安装部分见[前文](/post/install-kubesphere-with-kubekey/)，本文仅记录 Cilium 替换 kube-proxy 差异部分。

## 1. 系统限制
- Linux kernel >= 4.9.17(推荐>=5.10)

245 以上的 systemd 版本，会导致Pod到节点之外IPV4流量异常，官方文档提供了以下解决方案：

```shell
echo 'net.ipv4.conf.lxc*.rp_filter = 0' > /etc/sysctl.d/99-override_cilium_rp_filter.conf
sysctl --system
```

> 参考Issues：https://github.com/cilium/cilium/issues/10645

## 2. 环境信息

- OS: AlmaLinux 9.0
- Kubernetes: 1.24.3
  - containerd: 1.6.4
- KubeSphere: 3.3.1-rc.2
- Kubekey: 2.3.0-rc.2
- Harbor: 2.5.3
- Cilium 1.11.6  # 当前KubeKey代码写死，无法配置

## 3. 离线集群配置文件

> 与前文新增及差异部分
> - 新增部分: disableKubeProxy: true
> - 修改部分: network plugin改为cilium

`config-sample.yaml`内容修改后如下

```yaml
apiVersion: kubekey.kubesphere.io/v1alpha2
kind: Cluster
metadata:
  name: sample
spec:
  hosts:
  - {name: master, address: 10.10.1.1, internalAddress: 10.10.1.1, user: root, password: "cloudSky"}
  - {name: node01, address: 10.10.1.2, internalAddress: 10.10.1.2, user: root, password: "cloudSky"}
  roleGroups:
    etcd:
    - master
    control-plane: 
    - master
    worker:
    - master
    - node01
    # 如需使用 kk 自动部署镜像仓库，请设置该主机组 （建议仓库与集群分离部署，减少相互影响）
    registry:
    - master
  controlPlaneEndpoint:
    ## Internal loadbalancer for apiservers 
    # internalLoadbalancer: haproxy

    domain: lb.kubesphere.local
    address: ""
    port: 6443
  kubernetes:
    version: v1.24.3
    clusterName: cluster.local
    autoRenewCerts: true
    containerManager: containerd
    disableKubeProxy: true          # 禁用 kube-proxy 组件
  etcd:
    type: kubekey
  network:
    plugin: cilium                  # CNI使用cilium
    kubePodsCIDR: 10.233.64.0/18
    kubeServiceCIDR: 10.233.0.0/18
    ## multus support. https://github.com/k8snetworkplumbingwg/multus-cni
    multusCNI:
      enabled: false
  registry:
  	# 如需使用 kk 部署 harbor, 可将该参数设置为 harbor，不设置该参数且需使用 kk 创建容器镜像仓库，将默认使用docker registry。
    type: "harbor"
    # 如使用 kk 部署的 harbor 或其他需要登录的仓库，可设置对应仓库的auths，如使用 kk 创建的 docker registry 仓库，则无需配置该参数。
    auths:
      "dockerhub.kubekey.local":
         username: admin
         password: Harbor12345
         skipTLSVerify: true # 忽略自签名证书认证
         plainHTTP: false # Allow contacting registries over HTTP.
         #certsPath: "/etc/docker/certs.d/dockerhub.kubekey.local" # Use certificates at path (*.crt, *.cert, *.key) to connect to the registry.
    # 设置集群部署时使用的私有仓库
    privateRegistry: "dockerhub.kubekey.local"
    namespaceOverride: "kubesphereio"
    registryMirrors: []
    insecureRegistries: []
  addons: []



---
apiVersion: installer.kubesphere.io/v1alpha1
kind: ClusterConfiguration
metadata:
  name: ks-installer
  namespace: kubesphere-system
  labels:
    version: v3.3.1-rc.2
spec:
  persistence:
    storageClass: ""
  authentication:
    jwtSecret: ""
  zone: ""
  local_registry: ""
  namespace_override: ""
  # dev_tag: ""
  etcd:
    monitoring: false
    endpointIps: localhost
    port: 2379
    tlsEnable: true
  common:
    core:
      console:
        enableMultiLogin: true
        port: 30880
        type: NodePort
    # apiserver:
    #  resources: {}
    # controllerManager:
    #  resources: {}
    redis:
      enabled: false
      volumeSize: 2Gi
    openldap:
      enabled: false
      volumeSize: 2Gi
    minio:
      volumeSize: 20Gi
    monitoring:
      # type: external
      endpoint: http://prometheus-operated.kubesphere-monitoring-system.svc:9090
      GPUMonitoring:
        enabled: false
    gpu:
      kinds:
      - resourceName: "nvidia.com/gpu"
        resourceType: "GPU"
        default: true
    es:
      # master:
      #   volumeSize: 4Gi
      #   replicas: 1
      #   resources: {}
      # data:
      #   volumeSize: 20Gi
      #   replicas: 1
      #   resources: {}
      logMaxAge: 7
      elkPrefix: logstash
      basicAuth:
        enabled: false
        username: ""
        password: ""
      externalElasticsearchHost: ""
      externalElasticsearchPort: ""
  alerting:
    enabled: false
    # thanosruler:
    #   replicas: 1
    #   resources: {}
  auditing:
    enabled: false
    # operator:
    #   resources: {}
    # webhook:
    #   resources: {}
  devops:
    enabled: false
    # resources: {}
    jenkinsMemoryLim: 8Gi
    jenkinsMemoryReq: 4Gi
    jenkinsVolumeSize: 8Gi
  events:
    enabled: false
    # operator:
    #   resources: {}
    # exporter:
    #   resources: {}
    # ruler:
    #   enabled: true
    #   replicas: 2
    #   resources: {}
  logging:
    enabled: false
    logsidecar:
      enabled: true
      replicas: 2
      # resources: {}
  metrics_server:
    enabled: false
  monitoring:
    storageClass: ""
    node_exporter:
      port: 9100
      # resources: {}
    # kube_rbac_proxy:
    #   resources: {}
    # kube_state_metrics:
    #   resources: {}
    # prometheus:
    #   replicas: 1
    #   volumeSize: 20Gi
    #   resources: {}
    #   operator:
    #     resources: {}
    # alertmanager:
    #   replicas: 1
    #   resources: {}
    # notification_manager:
    #   resources: {}
    #   operator:
    #     resources: {}
    #   proxy:
    #     resources: {}
    gpu:
      nvidia_dcgm_exporter:
        enabled: false
        # resources: {}
  multicluster:
    clusterRole: none
  network:
    networkpolicy:
      enabled: false
    ippool:
      type: none
    topology:
      type: none
  openpitrix:
    store:
      enabled: false
  servicemesh:
    enabled: false
    istio:
      components:
        ingressGateways:
        - name: istio-ingressgateway
          enabled: false
        cni:
          enabled: false
  edgeruntime:
    enabled: false
    kubeedge:
      enabled: false
      cloudCore:
        cloudHub:
          advertiseAddress:
            - ""
        service:
          cloudhubNodePort: "30000"
          cloudhubQuicNodePort: "30001"
          cloudhubHttpsNodePort: "30002"
          cloudstreamNodePort: "30003"
          tunnelNodePort: "30004"
        # resources: {}
        # hostNetWork: false
      iptables-manager:
        enabled: true
        mode: "external"
        # resources: {}
      # edgeService:
      #   resources: {}
  terminal:
    timeout: 600
```
## 4. 安装 KubeSphere 集群

```shell
./kk create cluster -f config-sample.yaml -a kubesphere-v3.3.1-rc.2-artifact.tar.gz --with-packages
```

参数解释如下

- **config-sample.yaml**：离线环境的配置文件。
- **kubesphere-v3.3.1-rc.2-artifact.tar.gz**：指制品 artifact打包文件。
- **--with-packages**：若需要安装操作系统依赖，需指定该选项。
