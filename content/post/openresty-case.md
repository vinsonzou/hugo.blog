+++
description = ""
date = "2021-06-01T10:08:08+08:00"
title = "本人OpenResty实践集合"
tags = ["OpenResty", "Nginx"]

+++

- **优化**
    - 支持Brotli
    - Upstream主动健康检查
    - 动态证书
    - OCSP缓存
    - ssl分布式session
    - [mTLS支持](/post/openresty-mtls/)
- **命令行执行**
    - [Nginx支持WebP](/post/nginx-support-webp/)
    - [gogs/gitea webhook](/post/gitea-webhook/)
- **灰度发布**
    - 基于IP（动态IP）
    - 自定义变量
- [微信机器人](/post/nginx-wechat-ops/)
- **安全相关**
    - [授权下载](/post/nginx-authorized-download/)
    - [简易CC防护](/post/openresty-anticc)
    - [Gitea通过飞书SSO认证](https://gist.github.com/vinsonzou/6ce186914171736becb5e35ebdf806e3)
    - 安全token
    - [防止SNI信息泄露](/post/use-ssl_reject_handshake/)
- **API服务**
    - [IP查询服务](/post/nginx-ipip-service/)
    - [MQ解耦，对接阿里云RocketMQ](/post/nginx-use-mq/)
    - 对接Kafka
    - 对接Pulsar
- **缓存服务**
    - 主动缓存
    - SDK缓存
- **特权进程实践**
    - [ipsec操作](/post/openresty-privileged-agent/)
- **代理相关**
    - [http代理](/post/dynamic-http-proxy/)
    - [SNI代理](/post/nginx-sniproxy/)
