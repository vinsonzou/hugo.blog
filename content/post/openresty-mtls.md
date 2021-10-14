+++
description = ""
date = "2021-09-29T13:08:08+08:00"
title = "使用OpenResty mTLS认证"
tags = ["OpenResty"]

+++

## 环境

- CentOS 7
- OpenResty 1.19.9.1
    - cosocket-mtls patch
		- [ngx_lua-0.10.20_01-cosocket-mtls.patch](https://github.com/Kong/kong-build-tools/blob/master/openresty-patches/patches/1.19.9.1/ngx_lua-0.10.20_01-cosocket-mtls.patch)
		- [lua-resty-core-0.1.22_01-cosocket-mtls.patch](https://github.com/Kong/kong-build-tools/blob/master/openresty-patches/patches/1.19.9.1/lua-resty-core-0.1.22_01-cosocket-mtls.patch)
    - [lua-resty-http](https://github.com/vinsonzou/lua-resty-http)

为了支持mTLS功能，折腾的够呛，OpenResty官方又没支持，基于OpenResty的APISIX和Kong都有补丁，但两家公司的补丁又有点细微的差别。APISIX的定制OpenResty版本目前仅支持到1.19.3，而Kong的定制OpenResty跟官方版本是同步的。此处的方案就是用Kong的patch和APISIX的lua-resty-http(我稍微改了一丢丢兼容Kong的patch)。

## 功能

- OpenResty作为客户端调用外部HTTP接口, 使用mTLS认证

## Server配置

**对应nginx配置**

```sh
server {
　　listen 443 ssl;
    server_name ssl.test.com;
　　ssl_certificate ssl/mtls_server.crt;      #server公钥
　　ssl_certificate_key ssl/mtls_server.key;  #server私钥
　　ssl_client_certificate ssl/mtls_ca.crt;   #根级证书公钥，用于验证各个二级client
　　ssl_verify_client on;
}
```

## 请求验证

- curl命令行验证

```sh
curl --resolve ssl.test.com:443:127.0.0.1 --cacert ssl/mtls_ca.crt --cert ssl/mtls_client.crt --key mtls_client.key https://ssl.test.com
```

- lua-resty-http验证

```lua
location /t {
    resolver local=on ipv6=off;
    resolver_timeout 5s;
    lua_ssl_verify_depth 1;
    lua_ssl_trusted_certificate /data/nginx/mtls/ca.pem;

    content_by_lua_block {
        local http = require "resty.http"
        local httpc = http.new()

        local res, err = httpc:request_uri("https://ssl.test.com/", {
            ssl_cert_path = "/data/nginx/mtls/client.pem",
            ssl_key_path = "/data/nginx/mtls/client.key",
        })
        if not res then
            ngx.log(ngx.ERR, err)
        else
            ngx.say(res.body)
            ngx.exit(res.status)
        end
    }
}
```

## FAQ

**1. 多CA证书如何使用**
> 一个location下同时使用系统CA和自签名CA时，将CA合并成1个文件即可，参考系统的 `/etc/pki/tls/certs/ca-bundle.crt`

**2. client证书兼容性测试结果**

CA证书和Server证书都是ecdsa 256类型

{{< pure_table
"系统版本|客户端|版本|client证书类型| 能否使用"
"CentOS 7 | curl           | 7.29.0 | ecdsa 256      | ❌        "
"CentOS 8 | curl           | 7.61.1 | ecdsa 256      | ✅        "
"CentOS 7 | curl           | 7.29.0 | rsa 2048       | ✅        "
"CentOS 8 | curl           | 7.61.1 | rsa 2048       | ✅        "
"-        | lua-resty-http | 0.2.0  | ecdsa 256      | ✅        "
"-        | lua-resty-http | 0.2.0  | rsa 2048       | ✅        "
>}}
