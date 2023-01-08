---
title: "APISIX在k8s的使用01: 离线安装及初步使用"
subtitle: ""
date: 2022-10-12T14:08:00+08:00
lastmod: 2022-10-12T14:08:00+08:00
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
  - APISIX: 0.11.1
    - etcd: 3.5.4
  - APISIX ingress-controller: 0.10.1
  - APISIX Dashboard: 0.6.0
  - 关联镜像
    - apache/apisix:2.15.0-alpine
    - bitnami/etcd:3.5.4-debian-11-r14
    - apache/apisix-ingress-controller:1.5.0
    - apache/apisix-dashboard:2.13-alpine
    - busybox:1.28

## 2、安装

### 2.1 APISIX安装

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
	apisix-0.11.1.tgz
```

> Ps:
>
> - 未开启stream_route功能: 需手动修改apisix的configmap，添加端口监听
>
> - 当前版本暂不支持同时安装`ingress-controller`和`dashboard`，即如下参数
>
>   ```
>   --set apisix-ingress-controller.image.repository=dockerhub.kubekey.local/3rd/apisix-ingress-controller \
>   --set apisix-ingress-controller.initContainer.image=dockerhub.kubekey.local/library/busybox \
>   --set apisix-dashboard.image.repository=dockerhub.kubekey.local/3rd/apisix-dashboard:2.13-alpine \
>   --set dashboard.enabled=true \
>   --set ingress-controller.enabled=true \
>   ```

返回如下，表示安装成功。

```
NAME: apisix
LAST DEPLOYED: Tue Oct 11 19:29:58 2022
NAMESPACE: ingress-apisix
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. Get the application URL by running these commands:
     NOTE: It may take a few minutes for the LoadBalancer IP to be available.
           You can watch the status of by running 'kubectl get --namespace ingress-apisix svc -w apisix-gateway'
  export SERVICE_IP=$(kubectl get svc --namespace ingress-apisix apisix-gateway --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
  echo http://$SERVICE_IP:80
```

### 2.2 ingress-controller安装

```shell
helm install apisix-ingress-controller \
	--namespace ingress-apisix \
	--create-namespace \
	--set image.repository=dockerhub.kubekey.local/3rd/apisix-ingress-controller \
	--set initContainer.image=dockerhub.kubekey.local/library/busybox \
	apisix-ingress-controller-0.10.1.tgz
```

返回如下，表示安装成功。

```
NAME: apisix-ingress-controller
LAST DEPLOYED: Tue Oct 11 19:32:28 2022
NAMESPACE: ingress-apisix
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Get the application URL by running these commands:
export POD_NAME=$(kubectl get pods --namespace ingress-apisix -l "app.kubernetes.io/name=apisix-ingress-controller,app.kubernetes.io/instance=apisix-ingress-controller" -o jsonpath="{.items[0].metadata.name}")
export CONTAINER_PORT=$(kubectl get pod --namespace ingress-apisix $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
echo "Visit http://127.0.0.1:8080 to use your application"
kubectl --namespace ingress-apisix port-forward $POD_NAME 8080:$CONTAINER_PORT
```

### 2.3 dashboard安装

```shell
helm install apisix-dashboard \
	--namespace ingress-apisix \
	--create-namespace \
	--set image.repository=dockerhub.kubekey.local/3rd/apisix-dashboard \
	--set config.conf.etcd.endpoints.1=apisix-etcd.ingress-apisix.svc.cluster.local:2379 \
	apisix-dashboard-0.6.0.tgz
```

返回如下，表示安装成功。

```
NAME: apisix-dashboard
LAST DEPLOYED: Tue Oct 11 20:51:13 2022
NAMESPACE: ingress-apisix
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace ingress-apisix -l "app.kubernetes.io/name=apisix-dashboard,app.kubernetes.io/instance=apisix-dashboard" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace ingress-apisix $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace ingress-apisix port-forward $POD_NAME 8080:$CONTAINER_PORT
```

## 3、APISIX配置

- 开启stream_proxy功能，并监听端口
  - 同时配置`k8s Service`之`apisix-gateway`的端口监听
- 开启插件server-info

### 3.1 http转发配置

- dashboard转发配置

  http_dashboard.yaml，即通过ingress-controller转换为apisix配置

  ```yaml
  apiVersion: apisix.apache.org/v2
  kind: ApisixRoute
  metadata:
    name: dashboard
    namespace: ingress-apisix
  spec:
    http:
      - name: root 
        match:
          hosts:
            - apisix.test.com
          paths:
            - "/*"
        backends:
        - serviceName: apisix-dashboard
          servicePort: 80
  ```

### 3.2 tcp转发配置

- 动态注册：使用init-container调用apisix admin api实现

  ```shell
  # 注册stream_routes接口
  curl http://apisix-admin.ingress-apisix.svc.cluster.local:9180/apisix/admin/stream_routes/testgateway0 -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X PUT -d '
  {
  	"server_port": 8201,
  	"upstream": {
  		"nodes": {
  			"10.233.107.69:8100": 1
  		},
  		"type": "roundrobin",
  		"scheme": "tcp",
  	}
  }'
  
  # 查看stream_routes接口
  curl http://127.0.0.1:9180/apisix/admin/stream_routes -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1"
  ```

- 静态配置：使用`kubectl apply -f tcp-route.yaml`，即通过ingress-controller转换为apisix配置

  ```yaml
  apiVersion: apisix.apache.org/v2
  kind: ApisixRoute
  metadata:
    name: tcp-route
  spec:
    stream:
      - name: tcp-route-gateway-1
        protocol: TCP
        match:
          ingressPort: 8201
        backend:
          serviceName: gateway   # 对应k8s的serverName
          servicePort: 8100
  ```

## 4、日常维护

- 卸载

  ```shell
  helm uninstall apisix -n ingress-apisix
  ```

  > ps:
  >
  > - namespace不会删除
  > - etcd持久卷不会删除