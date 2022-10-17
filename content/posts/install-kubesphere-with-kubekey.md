---
title: "KubeSphere系列一: 使用 KubeKey 离线部署 Kubernetes 与 KubeSphere"
subtitle: ""
date: 2022-09-17T12:36:03+08:00
lastmod: 2022-09-17T12:36:03+08:00
description: ""

tags: ["Kubernetes","KubeSphere"]
categories: ["Kubernetes"]
---

## 部署准备

### 1. 资源清单

| 名称   | 数量 | 用途             |
| ------ | ---- | ---------------- |
| 服务器 | 2    | 离线部署环境使用 |

### 2. 环境信息

- OS: AlmaLinux 9.0
- Kubernetes: 1.24.3
  - containerd: 1.6.4
- KubeSphere: 3.3.1-rc.2
- Kubekey: 2.3.0-rc.2
- Harbor: 2.5.3

### 3. kubekey离线所需artifact清单

`ks-v3.3.1-rc.2-manifest.yaml`

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
        localPath: /app/k8s/3.3.1-rc.2/almalinux-9.0-rpms-amd64.iso
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
  - registry.cn-beijing.aliyuncs.com/kubesphereio/ks-installer:v3.3.1-rc.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/ks-apiserver:v3.3.1-rc.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/ks-console:v3.3.1-rc.2
  - registry.cn-beijing.aliyuncs.com/kubesphereio/ks-controller-manager:v3.3.1-rc.2
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
  - registry.cn-beijing.aliyuncs.com/kubesphereio/openpitrix-jobs:v3.3.1-rc.0
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

### 4. 导出制品 artifact

```shell
export KKZONE=cn
./kk artifact export -m ks-v3.3.1-rc.2-manifest.yaml -o kubesphere-v3.3.1-rc.2-artifact.tar.gz
```

> 制品（artifact）是一个根据指定的 manifest 文件内容导出的包含镜像 tar 包和相关二进制文件的 tgz 包。在 KubeKey 初始化镜像仓库、创建集群、添加节点和升级集群的命令中均可指定一个 artifact，KubeKey 将自动解包该 artifact 并在执行命令时直接使用解包出来的文件。
>
> - 导出时请确保网络连接正常。
> - KubeKey 会解析镜像列表中的镜像名，若镜像名中的镜像仓库需要鉴权信息，可在 manifest 文件中的 **.registry.auths** 字段中进行配置。

## 离线安装

### 1. 拷贝安装文件至离线环境

将下载的 KubeKey 和制品 artifact 通过 U 盘等介质拷贝至离线环境安装节点。

### 2. 创建离线集群配置文件

```shell
./kk create config --with-kubesphere v3.3.1-rc2 --with-kubernetes v1.24.3 -f config-sample.yaml
```

> 说明:
>
> 1. 按照实际离线环境配置修改节点信息和角色信息
> 2. 必须指定 registry 仓库部署节点（因为 KK 部署自建 harbor 仓库需要使用）
> 3. registry 里必须指定 type 类型为 harbor，不配 harbor 的话默认是装的 docker registry

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
  etcd:
    type: kubekey
  network:
    plugin: calico
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

### 3. 安装镜像仓库

```shell
./kk init registry -f config-sample.yaml -a kubesphere-v3.3.1-rc.2-artifact.tar.gz
```

> 命令中的参数解释如下
>
> - **config-sample.yaml** 指离线环境集群的配置文件。
> - **kubesphere-v3.3.1-rc.2-artifact.tar.gz** 指制品 artifact打包文件。

### 4. 创建 Harbor 项目

- 方法一、执行脚本创建 Harbor 项目

  - 脚本[参考](https://raw.githubusercontent.com/kubesphere/ks-installer/master/scripts/create_project_harbor.sh)

  - 修改后脚本`create_project_harbor.sh`

    ```bash
    #!/usr/bin/env bash
    
    # Harbor 仓库地址
    url="https://dockerhub.kubekey.local"
    
    # 访问 Harbor 仓库用户
    user="admin"
    
    # 访问 Harbor 仓库用户密码
    passwd="Harbor12345"
    
    # 需要创建的项目名列表，正常只需要创建一个**kubesphereio**即可，这里为了保留变量可扩展性多写了两个。
    harbor_projects=(library
        kubesphereio
        kubesphere
    )
    
    for project in "${harbor_projects[@]}"; do
        echo "creating $project"
        curl -u "${user}:${passwd}" -X POST -H "Content-Type: application/json" "${url}/api/v2.0/projects" -d "{ \"project_name\": \"${project}\", \"public\": true}" -k
    done
    ```

  - 执行以下命令创建 Harbor 项目

    ```shell
    sh create_project_harbor.sh
    ```

- 方法二、登录 Harbor 仓库创建项目

  将项目设置为**公开**以便所有用户都能够拉取镜像。关于如何创建项目，请参阅[创建项目](https://goharbor.io/docs/2.5.0/working-with-projects/create-projects/)。

### 5. 安装 KubeSphere 集群

```shell
./kk create cluster -f config-sample.yaml -a kubesphere-v3.3.1-rc.2-artifact.tar.gz --with-packages
```

参数解释如下

- **config-sample.yaml**：离线环境的配置文件。
- **kubesphere-v3.3.1-rc.2-artifact.tar.gz**：指制品 artifact打包文件。
- **--with-packages**：若需要安装操作系统依赖，需指定该选项。

### 6. 查看集群状态

```
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l 'app in (ks-install, ks-installer)' -o jsonpath='{.items[0].metadata.name}') -f
```

安装完成后，您会看到以下内容：

```
**************************************************
#####################################################
###              Welcome to KubeSphere!           ###
#####################################################
   
Console: http://10.10.1.1:30880
Account: admin
Password: P@88w0rd
   
NOTES：
1. After you log into the console, please check the
monitoring status of service components in
the "Cluster Management". If any service is not
ready, please wait patiently until all components
are up and running.
1. Please change the default password after login.
   
#####################################################
https://kubesphere.io             2022-09-17 10:40:08
#####################################################
```

### 7. 访问 KubeSphere 控制台

通过 `http://{IP}:30880` 使用默认帐户和密码 `admin/P@88w0rd` 

## 参考

- https://kubesphere.io/zh/docs/v3.3/installing-on-linux/introduction/air-gapped-installation/
