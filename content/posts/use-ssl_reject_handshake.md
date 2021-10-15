+++
description = ""
date = "2021-07-04T10:08:08+08:00"
title = "防止SNI信息泄露"
tags = ["Nginx"]
categories = ["Nginx"]

+++

有了 `ssl_reject_handshake` (Nginx ≥ 1.19.4)这个参数，再也不需要strict-sni.patch了。

本质需求就是为了当机器人或者奇怪的人类通过HTTPS访问你的IP时不暴露证书，也就不会暴露域名。

官方文档给了个例子如下，在以下配置中，除example.com以外，其他域名的SSL握手将被拒绝。（返回UNRECOGNIZED NAME，Chrome提示ERR_SSL_UNRECOGNIZED_NAME_ALERT）

```sh
server {
    listen 443 ssl default_server;
    ssl_reject_handshake on;
}

server {
    listen              443 ssl;
    server_name         example.com;
    ssl_certificate     example.com.crt;
    ssl_certificate_key example.com.key;
}
```

**注意事项**

- nginx无法启用tls1.3的握手失败解决办法，openssl 1.1.1h bug，详细见[传送门](https://trac.nginx.org/nginx/ticket/2071)。
    - 升级至openssl ≥ 1.1.1j 修复此问题
