+++
description = ""
date = "2021-06-02T10:18:08+08:00"
title = "使用Nginx做SNI反向代理"
tags = ["OpenResty", "SNI"]

+++

从 Nginx 1.11.5 版本开始支持用做 SNI 反代，使用 Nginx 做 SNI 反代比用 SNI Proxy 配置起来更简单、更稳定。Nginx stream流程处理见[官方文档](http://nginx.org/en/docs/stream/stream_processing.html)，此功能可用来加速国外https业务，如苹果订单确认接口。

### 安装

编译时添加如下参数

> --with-stream --with-stream_ssl_preread_module --with-stream_ssl_module

### 配置

**基础版**
```sh
stream {
    server {
        listen 443;
        ssl_preread on;
        resolver local=on ipv6=off valid=60s;  # local=on需要OpenResty补丁
        proxy_pass $ssl_preread_server_name:$server_port;
    }
}
```

**优化版**

- 解决原有无法过滤域名的风险

```sh
log_format proxy '$remote_addr [$time_local] $sniproxy_upstream '
                 '$protocol $status $bytes_sent $bytes_received '
                 '$session_time "$upstream_addr" '
                 '"$upstream_bytes_sent" "$upstream_bytes_received" "$upstream_connect_time"';

init_by_lua_block {
    local sni = require("resty.sniproxy")
    sni.rules = {
        {"buy.itunes.apple.com"},
        {".", "unix:/var/run/nginx-default.sock"}
    }
}

# for OpenResty >= 1.13.6.1, native Nginx proxying
lua_add_variable $sniproxy_upstream;

server {
    listen 443;
    resolver local=on ipv6=off valid=60s;
    resolver_timeout 5s;
    preread_by_lua_block {
        local sni = require("resty.sniproxy")
        local sp = sni:new()
        sp:preread_by()
    }
    proxy_pass $sniproxy_upstream;
    access_log logs/tcp_ssl.access.log proxy buffer=32k flush=10s;
}
```

### 参考

- http://nginx.org/en/docs/stream/ngx_stream_ssl_preread_module.html
- https://zhuanlan.zhihu.com/p/46014862
- https://github.com/fffonion/lua-resty-sniproxy
