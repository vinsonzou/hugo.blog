---
title: "Kubernetes容器化 - Zookeeper"
subtitle: ""
date: 2022-10-04T13:21:00+08:00
lastmod: 2022-10-04T13:21:00+08:00
description: ""

tags: ["Kubernetes","zookeeper","容器化"]
categories: ["Kubernetes"]
---

Kubernetes官方有一个[ZooKeeper教程](https://kubernetes.io/zh/docs/tutorials/stateful-application/zookeeper/)。此文使用的是Docker官方镜像，主要是两个环境变量的设置

- `ZOO_MY_ID`：用来指定myid文件中的编号
- `ZOO_SERVERS`：用来指定集群列表

在Kubernetes中通过StatefulSet来部署，但`ZOO_MY_ID`环境变量无法通过Yaml文件注入到容器，网上找了一圈后，通过MySQL主从配置`server-id`跟我们配置zk集群一样，可通过pod ordinal index获得`ZOO_MY_ID`。

> StatefulSet 是用来管理有状态应用的工作负载 API 对象。
>
> StatefulSet 中的 Pod 拥有一个具有黏性的、独一无二的身份标识。这个标识基于 StatefulSet 控制器分配给每个 Pod 的唯一顺序索引。Pod 的名称的形式为$(statefulset name)-$(ordinal index) 。例如：zk的StatefulSet 拥有三个副本，所以它创建了三个 Pod：zk-0、zk-1、zk-2。
>
> 和 Deployment 相同的是，StatefulSet 管理了基于相同容器定义的一组 Pod。但和 Deployment 不同的是，StatefulSet 为它们的每个 Pod 维护了一个固定的 ID。这些 Pod 是基于相同的声明来创建的，但是不能相互替换：无论怎么调度，每个 Pod 都有一个永久不变的 ID。
>
> 【使用场景】StatefulSets 对于需要满足以下一个或多个需求的应用程序很有价值：
>
> - 稳定的、唯一的网络标识符，即Pod重新调度后其PodName和HostName不变【当然IP是会变的】
> - 稳定的、持久的存储，即Pod重新调度后还是能访问到相同的持久化数据，基于PVC实现
> - 有序的、优雅的部署和缩放
> - 有序的、自动的滚动更新
>
> 如上面，稳定意味着 Pod 调度或重调度的整个过程是有持久性的。
>
> 如果应用程序不需要任何稳定的标识符或有序的部署、删除或伸缩，则应该使用由一组无状态的副本控制器提供的工作负载来部署应用程序，比如使用 Deployment 或者 ReplicaSet 可能更适用于无状态应用部署需要。

## 配置说明

- env设置
  - ZOO_STANDALONE_ENABLED：`false`
  - ZOO_ADMINSERVER_ENABLED：`false`
  - ZOO_SERVERS：`server.1=zk-0.zk-hs.test.svc.cluster.local:2888:3888;2181 server.2=zk-1.zk-hs.test.svc.cluster.local:2888:3888;2181 server.3=zk-2.zk-hs.test.svc.cluster.local:2888:3888;2181`
    - zk-0.zk-hs.test.svc.cluster.local格式说明: `$(statefulset name)-$(pod ordinal index).$(serviceName).$(namespace).svc.cluster.local`
- 动态PVC自动创建关联PV
- ZOO_MY_ID通过`initContainers`写入至PVC中，即`/data/myid`

## 部署

这里以一个3个节点的ZooKeeper集群为例，`zookeeper-headless.yaml`如下

```yaml
apiVersion: v1
kind: Service
metadata:
  name: zk-hs
  labels:
    app: zk
spec:
  ports:
  - port: 2888
    name: server
  - port: 3888
    name: leader-election
  clusterIP: None
  selector:
    app: zk
---
apiVersion: v1
kind: Service
metadata:
  name: zk-cs
  labels:
    app: zk
spec:
  ports:
  - port: 2181
    name: client
  selector:
    app: zk
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: zk-pdb
spec:
  selector:
    matchLabels:
      app: zk
  maxUnavailable: 1
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: zk
spec:
  selector:
    matchLabels:
      app: zk
  serviceName: zk-hs
  replicas: 3
  updateStrategy:
    type: RollingUpdate
  podManagementPolicy: OrderedReady
  template:
    metadata:
      labels:
        app: zk
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: "app"
                    operator: In
                    values:
                    - zk
              topologyKey: "kubernetes.io/hostname"
      initContainers:
      - name: init-zk
        image: "zookeeper:3.6.3"
        imagePullPolicy: IfNotPresent
        command:
        - bash
        - "-c"
        - |
          set -ex
          # Generate Zookeeper ZOO_MY_ID from pod ordinal index.
          [[ `hostname` =~ -([0-9]+)$ ]] || exit 1
          ordinal=${BASH_REMATCH[1]}
          # Add an offset to avoid reserved ZOO_MY_ID=0 value.
          echo $((1 + $ordinal)) > /data/myid
        volumeMounts:
        - name: zk-pvc
          mountPath: /data
      containers:
      - name: zk
        image: "zookeeper:3.6.3"
        imagePullPolicy: IfNotPresent
        resources:
          requests:
            memory: "1Gi"
            cpu: "0.5"
        ports:
        - containerPort: 2181
          name: client
        - containerPort: 2888
          name: server
        - containerPort: 3888
          name: leader-election
        readinessProbe:
          tcpSocket:
            port: 2181
          initialDelaySeconds: 10
          timeoutSeconds: 15
          periodSeconds: 5
        livenessProbe:
          tcpSocket:
            port: 2181
          initialDelaySeconds: 60
          timeoutSeconds: 15
          periodSeconds: 15
        env:
        - name: ZOO_STANDALONE_ENABLED
          value: "false"
        - name: ZOO_ADMINSERVER_ENABLED
          value: "false"
        - name: ZOO_SERVERS
          value: "server.1=zk-0.zk-hs.test.svc.cluster.local:2888:3888;2181 server.2=zk-1.zk-hs.test.svc.cluster.local:2888:3888;2181 server.3=zk-2.zk-hs.test.svc.cluster.local:2888:3888;2181"
        volumeMounts:
        - name: zk-pvc
          mountPath: /data
      securityContext:
        runAsUser: 1000
        fsGroup: 1000
  volumeClaimTemplates:
  - metadata:
      name: zk-pvc
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: local
      resources:
        requests:
          storage: 1Gi
      #selector: # 静态pvc需要绑定
      #  matchLabels:
      #    pv: zk-pv

```

```shell
# 部署
$ kubectl -n test apply -f zookeeper-headless.yaml

# 查看svc
$ kubectl -n test get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
zk-cs        ClusterIP   10.233.2.187    <none>        2181/TCP            29s
zk-hs        ClusterIP   None            <none>        2888/TCP,3888/TCP   29s

# 查看pod
$ kubectl -n test get pod
NAME   READY   STATUS    RESTARTS   AGE
zk-0   1/1     Running   0          102s
zk-1   1/1     Running   0          62s
zk-2   1/1     Running   0          38s
```

## 查看zookeeper配置

```shell
$ kubectl -n test exec zk-0 -c zk -- cat /conf/zoo.cfg

dataDir=/data
dataLogDir=/datalog
tickTime=2000
initLimit=5
syncLimit=2
autopurge.snapRetainCount=3
autopurge.purgeInterval=0
maxClientCnxns=60
standaloneEnabled=false
admin.enableServer=false
server.1=zk-0.zk-hs.test.svc.cluster.local:2888:3888;2181
server.2=zk-1.zk-hs.test.svc.cluster.local:2888:3888;2181
server.3=zk-2.zk-hs.test.svc.cluster.local:2888:3888;2181
```

## 查看集群状态

```shell
$ kubectl -n test exec zk-0 -c zk -- zkServer.sh status

ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: follower

$ kubectl -n test exec zk-1 -c zk -- zkServer.sh status

ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: leader

$ kubectl -n test exec zk-2 -c zk -- zkServer.sh status

ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: follower
```

可以看到：zk-1 是集群leader，zk-0 和 zk-2 是集群follower。

```shell
$ kubectl -n test exec zk-0 -c zk -- cat /data/myid
1

$ kubectl -n test exec zk-1 -c zk -- cat /data/myid
2

$ kubectl -n test exec zk-2 -c zk -- cat /data/myid
3
```

## 集群测试

```shell
$ kubectl -n test exec zk-0 -c zk -- zkCli.sh create /hello world

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
Created /hello

$ kubectl -n test exec zk-1 -c zk -- zkCli.sh get /hello
$ kubectl -n test exec zk-2 -c zk -- zkCli.sh get /hello
```

在 zk-0 创建的数据在集群中所有的服务上都是可用的。

## FAQ

### PodDisruptionBudget
k8s可以为每个应用程序创建 PodDisruptionBudget 对象（PDB）。PDB 将限制在同一时间因资源干扰导致的复制应用程序中宕机的 pod 数量。

可以通过两个参数来配置PodDisruptionBudget

- MinAvailable：表示最小可用POD数，表示应用POD集群处于运行状态的最小POD数量，或者是运行状态的POD数同总POD数的最小百分比
- MaxUnavailable：表示最大不可用PO数，表示应用POD集群处于不可用状态的最大POD数，或者是不可用状态的POD数同总POD数的最大百分比

需要注意的是，MinAvailable参数和MaxUnavailable参数只能同时配置一个。
