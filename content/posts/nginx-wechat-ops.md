+++
description = ""
date = "2021-06-11T10:08:08+08:00"
title = "lua实现微信机器人"
tags = ["OpenResty"]
categories = ["OpenResty"]

+++

使用企业微信调整值班人员信息，示例为联系人tag调整，可以自定义任何想实现功能。

## 环境

- CentOS 7
- Nginx
    - lua-nginx-module
    - [lua-resty-http](https://github.com/ledgetech/lua-resty-http)
    - [lua实现微信加密库](https://github.com/vinsonzou/WXBizMsgCrypt)
    - [xml2lua ≤ v1.4-5](https://github.com/manoelcampos/xml2lua)
- 依赖服务
    - 企业微信

## 配置

**对应nginx配置**

```sh
server {
    listen 443 ssl http2;
    ssl_certificate     test.crt;
    ssl_certificate_key test.key;
    ssl_protocols             TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5;
    ssl_session_timeout       1d;
    ssl_session_cache         shared:SSL:20m;
    add_header Strict-Transport-Security max-age=31536000;
    server_name wx-ops.test.com;
    access_log logs/wechat-ops.access.log main;
    error_log logs/wechat-ops.info;

    location /api/ {
        resolver local=on ipv6=off;
        resolver_timeout 2s;
        lua_ssl_verify_depth 1;
        lua_ssl_trusted_certificate /etc/pki/tls/certs/ca-bundle.crt;
        content_by_lua_file lua/wechat-ops.lua;
    }
}
```

**wechat_ops.lua**

{{< gist vinsonzou fa6ea86de40dd36bce5eaa4c4e936942 >}}
