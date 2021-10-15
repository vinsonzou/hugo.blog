+++
date = "2021-04-19T17:01:15+08:00"
title = "4层TCP转发后获得真实IP之proxy_protocol"
description = ""
tags = ["golang","proxy_protocol","cloudflare","go-mmproxy"]
categories = ["golang"]

+++

## 需求

后端TCP Server在经过TCP代理(nginx stream模块)后，程序不做任何调整获得用户真实IP。
了解到cloudflare的spectrum产品与我们这个需求一致，刚好有细节和源码分享，参考如下:

- https://blog.cloudflare.com/mmproxy-creative-way-of-preserving-client-ips-in-spectrum/
- https://github.com/cloudflare/mmproxy

有公司参考cloudflare的mmproxy，用golang实现了性能更优的版本。

- https://github.com/path-network/go-mmproxy

## 部署

拓扑与cloudflare spectrum产品一样，示意图如下：

![](https://blog.cloudflare.com/content/images/2018/04/Screen-Shot-2018-04-15-at-12.26.28-PM-1.png)

- go > 1.11
- as root or with `CAP_NET_RAW` capability to be able to set `IP_TRANSPARENT` socket opt.

```sh
go get github.com/path-network/go-mmproxy
sudo setcap cap_net_raw=+ep $(readlink -f $(which go-mmproxy))
```

## 测试

- 启动

```sh
# go-mmproxy监听端口为25577，TCP后端端口为25578
ip rule add from 127.0.0.1/8 iif lo table 123
ip route add local 0.0.0.0/0 dev lo table 123
go-mmproxy -l 0.0.0.0:25577 -4 127.0.0.1:25578 -6 "[::1]:25578" -allowed-subnets ./path-prefixes.txt
```

- 停止

```sh
# 清理路由
ip rule del from 127.0.0.1/8 iif lo table 123
ip route del local 0.0.0.0/0 dev lo table 123
```

## FAQ

- **目前无法做到多端口映射，一个端口就得启动一个进程**

