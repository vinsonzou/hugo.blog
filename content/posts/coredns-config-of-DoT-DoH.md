---
title: "DoT/DoH 之 CoreDNS配置"
subtitle: ""
date: 2022-11-12T15:00:03+08:00
lastmod: 2022-11-12T15:00:03+08:00
description: ""

tags: ["CoreDNS"]
categories: []
---

DoT/DoH/DoQ 的好处就不用多说了，那么如何让你的网络用上DoT/DoH/DoQ呢？

> - DoT：DNS over TLS
> - DoH：DNS over HTTPS
> - DoQ：DNS over QUIC(CoreDNS暂未支持)
> - https://adguard.com/en/blog/dns-over-quic.html
> - https://www.cloudflare.com/zh-cn/learning/dns/dns-over-tls/

本文以CoreDNS为例，CoreDNS是一个Go语言实现的DNS server，具有跨平台、插件化、可拓展等诸多优点。

## 0x01: 配置Corefile

CoreDNS服务端当前是支持四种协议（DNS、DoT、DoH、gRPC），forward插件只支持DNS和DoT。

```
# DNS配置
.:53 {
    hosts m114.hosts . {
        fallthrough
    }

    forward . 223.5.5.5 119.29.29.29 # 将非m114.hosts文件的域名的 DNS 请求，转发给公网 DNS 服务器。
    log     # 启动日志
    cache   # 启用缓存，缓存的 TTL 设为 30
    loop    # 检测并停止死循环解析
    reload  # 支持动态更新 Corefile

    # 随机化 A/AAAA/MX 记录的顺序以实现负载均衡。
    # 因为 DNS resolver 通常使用第一条记录，而第一条记录是随机的。这样客户端的请求就能被随机分配到多个后端。
    loadbalance
}

# DoT配置
tls://.:853 {
    #「tls」后为证书和密钥文件路径，可自定义
    tls m114.crt m114.key
    
	#「m114.hosts」为hosts文件路径，可自定义
    hosts m114.hosts . {
        fallthrough 
    }

    #「forward .」后为上游DNS地址，可自定义
    forward . tls://223.5.5.5:853 {
        tls_servername dns.alidns.com
    }

    cache
    log
    errors
}

# 独立DoH配置
https://.:443 {
    tls m114.crt m114.key
    forward . 223.5.5.5 119.29.29.29
    cache
    log
    errors
}

# Nginx + DoH配置
https://.:8053 {
    bind 127.0.0.1
    forward . 223.5.5.5 119.29.29.29
    cache
    log
    errors
}
```

## 0x02: 启动测试

```sh
./coredns
# 或coredns -conf Corefile路径
```

### DoT测试
> Mac若提示找不到kdig命令，使用brew安装`knot`
```sh
kdig -p 853 @127.0.0.1 +tls test.m114.org
```

## 0x03: 服务部署
- `coredns.service`

  ```sh
  [Unit]
  Description=CoreDNS
  
  [Service]
  Type=simple
  ExecStart=/usr/local/bin/coredns -conf /etc/coredns/Corefile -pidfile /var/run/coredns.pid
  
  [Install]
  WantedBy=multi-user.target
  ```

- 设定开机启动并启动

  ```sh
  systemctl enable coredns --now
  ```

## 0x04: 系统/浏览器配置
- [Chrome浏览器使用安全DNS(即DoH)](https://help.aliyun.com/document_detail/176821.html#h2-7fz-y7e-9sm)
- [Android系统配置加密DNS(即DoT)](https://help.aliyun.com/document_detail/176821.html#h2-5dz-2gm-y47)
- [iOS14和macOS11配置](https://help.aliyun.com/document_detail/176821.html#h2-bmo-jy0-zby)
