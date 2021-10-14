+++
description = ""
date = "2021-09-11T11:08:08+08:00"
title = "使用OpenResty实现简易CC防护"
tags = ["OpenResty"]

+++

## 环境

- CentOS 7
- OpenResty 1.19.9.1
    - [lua-resty-ipmatcher](https://github.com/api7/lua-resty-ipmatcher)
    - [lua-var-nginx-module](https://github.com/api7/lua-var-nginx-module)

## 功能

- 支持IP白名单
- 支持IP黑名单
- 计数key为：ip + "." + md5(host + `request_uri` + useragent)，可自定义
- 60秒内请求超过60次就封禁3600秒

## 配置

**对应nginx配置**

```sh
lua_shared_dict cc_counter 100m;

server {
    listen 80;
    server_name demo.test.com;

    location / {
        access_by_lua_file lua/anticc.lua;
    }
}
```

**anticc.lua**

{{< gist vinsonzou f63571c8cdd25fcf67c75bd4fdfc624d >}}
