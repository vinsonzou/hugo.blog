+++
description = ""
date = "2021-09-28T09:08:08+08:00"
title = "CFSSL使用"
tags = ["安全"]

+++

## 简介

在工作中，我们经常会遇到各种证书问题，如k8s,etcd等服务时，我们往往都要使用证书，本文将使用CFSSL工具快速简单的配置证书。这里将生成三种证书包含客户端证书、服务端证书、双向证书。
- client certificate: 用于服务端认证客户端，例如etcdctl、etcd proxy、fleetctl、docker客户端
- server certificate: 服务端使用，客户端以此验证服务端身份，例如docker服务端、kube-apiserver
- peer certificate: 双向证书，用于etcd集群成员间通信

## 1. cfssl安装

项目地址: https://github.com/cloudflare/cfssl

```sh
wget https://github.com/cloudflare/cfssl/releases/download/v1.6.1/cfssl_1.6.1_linux_amd64 -O /usr/local/bin/cfssl
wget https://github.com/cloudflare/cfssl/releases/download/v1.6.1/cfssljson_1.6.1_linux_amd64 -O /usr/local/bin/cfssljson
chmod +x /usr/local/bin/{cfssl,cfssljson}
```

## 2. cfssl常用命令

```sh
# 初始化CA
cfssl gencert -initca ca-csr.json | cfssljson -bare ca

# 查看cert(证书信息)
cfssl certinfo -cert ca.pem

# 查看CSR(证书签名请求)信息
cfssl certinfo -csr ca.csr
```

## 3. 创建CA证书

### 3.1 CA证书配置

默认证书配置模板

```sh
cfssl print-defaults config > ca-config.json
cat ca-config.json
{
    "signing": {
        "default": {
            "expiry": "168h"
        },
        "profiles": {
            "www": {
                "expiry": "8760h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth"
                ]
            },
            "client": {
                "expiry": "8760h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "client auth"
                ]
            }
        }
    }
}
```

修改后的ca-config.json
```json
{
    "signing": {
        "default": {
            "expiry": "8760h"
        },
        "profiles": {
            "server": {
                "expiry": "8760h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth"
                ]
            },
            "client": {
                "expiry": "8760h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "client auth"
                ]
            },
            "peer": {
                "expiry": "8760h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
```

知识点:

- CA证书有效期为5年
- ca-config.json：可以定义多个 profiles，如server,client,peer，分别定义不同的过期时间、使用场景等参数；后续在签名证书时使用某个 profile。
- default默认策略，指定了证书的默认有效期是1年(8760h)
- signing：表示该证书可用于签名其它证书；生成的 ca.pem 证书中 CA=TRUE；
- server auth：表示client可以用该CA对server提供的证书进行验证；
- client auth：表示server可以用该CA对client提供的证书进行验证；
- expiry：也表示过期时间，如果不写以default中的为准;
- 注意标点符号，标点符号只支持英文，最后一个字段一般是没有逗号（,）的;

### 3.2 CA CSR请求

默认csr请求模板，后面的server、client、peer参照此模板修改。
```sh
cfssl print-defaults csr > ca-csr.json
cat ca-csr.json
{
    "CN": "example.net",
    "hosts": [
        "example.net",
        "www.example.net"
    ],
    "key": {
        "algo": "ecdsa",
        "size": 256
    },
    "names": [
        {
            "C": "US",
            "L": "CA",
            "ST": "San Francisco"
        }
    ]
}
```

修改后的CA CSR
```json
{
  "CN": "Self Signed CA",
  "key": {
    "algo": "ecdsa",
    "size": 256
  },
  "names": [
    {
      "C": "CN",
      "ST": "Shanghai",
      "L": "Shanghai",
      "O": "ops",
      "OU": "System"
    }
  ],
  "ca": {
      "expiry": "87600h"
  }
}
```

知识点：
- key: 生成证书的算法
- CN: Common Name，kube-apiserver 从证书中提取该字段作为请求的用户名 (User Name)，浏览器使用该字段验证网站是否合法，一般写的是域名。非常重要。浏览器使用该字段验证网站是否合法
- names：一些其它的属性
    - C: Country，国家
    - ST: State，州，省
    - L: Locality，地区，城市
    - O: Organization Name，组织名称，公司名称(在k8s中常用于指定Group，进行RBAC绑定)
    - OU: Organization Unit Name，组织单位名称，公司部门
- ca: ca属性
    - expiry: CA证书有效期调整为10年（默认为5年）

### 3.3 生成CA证书和私钥

该命令会生成运行CA所必需的文件ca-key.pem（私钥）和ca.pem（证书），还会生成ca.csr（证书签名请求），用于交叉签名或重新签名。

```sh
cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
```

### 3.4 其他

如果CA证书过期，则可以使用下面方法重新生成CA证书

```sh
# 使用现有的CA私钥，重新生成
cfssl gencert -initca -ca-key ca-key.pem ca-csr.json | cfssljson -bare ca

# 使用现有的CA私钥和CA证书，重新生成
cfssl gencert -renewca -ca-key ca-key.pem -ca ca.pem  | cfssljson -bare ca
```

## 4. 签发Server证书

修改后server-csr.json
```json
{
    "CN": "Server",
    "hosts": [
        "127.0.0.1","192.168.0.1"
    ],
    "key": {
        "algo": "ecdsa",
        "size": 256
    },
    "names": [
        {
            "C": "CN",
            "L": "Shanghai",
            "ST": "Shanghai"
        }
    ]
}
```

知识点:
- hosts: 表示哪些主机名(域名)或者IP可以使用此csr申请的证书，为空或者""表示所有的都可以使用。

生成服务端证书和私钥
```sh
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=server server-csr.json | cfssljson -bare server
```

知识点:

- config: 引用的是模板中的默认配置文件;
- profile: 是指定特定的使用场景，比如ca-config.json中的server区域;

## 5. 签发Client证书

修改后的client-csr.json
```json
{
    "CN": "Client",
    "hosts": [
        ""
    ],
    "key": {
        "algo": "ecdsa",
        "size": 256
    },
    "names": [
        {
            "C": "CN",
            "L": "Shanghai",
            "ST": "Shanghai"
        }
    ]
}
```

生成客户端证书和私钥
```sh
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=client client-csr.json | cfssljson -bare client
```

## 6. 签发peer证书

修改后的peer-csr.json
```json
{
    "CN": "Peer",
    "hosts": [
        "127.0.0.1","192.168.0.1"
    ],
    "key": {
        "algo": "ecdsa",
        "size": 256
    },
    "names": [
        {
            "C": "CN",
            "L": "Shanghai",
            "ST": "Shanghai"
        }
    ]
}
```

生成peer证书和私钥
```sh
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=peer peer-csr.json | cfssljson -bare peer
```

## 7. 校验证书

校验生成的证书是否和配置相符
```sh
openssl x509 -in ca.pem -text -noout
openssl x509 -in server.pem -text -noout
openssl x509 -in client.pem -text -noout
```
