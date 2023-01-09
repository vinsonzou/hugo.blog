---
title: "APISIX配置http重定向到https
"
subtitle: ""
date: 2022-10-18T13:21:00+08:00
lastmod: 2022-10-18T13:21:00+08:00
description: ""

tags: ["APISIX"]
categories: ["APISIX"]
---

## 背景

一般网站在使用https协议之后，都会在访问 http 站点的时候，自动重定向到https，那么使用 APISIX 该怎么实现呢？

答案是通过内置变量 `http_x_forwarded_proto` 判断请求协议实现跳转，实现如下

## 实现
```shell
curl -i http://127.0.0.1:9080/apisix/admin/routes/1  -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "uri": "/",
    "host": "test.m114.org",
    "vars": [
        [
            "http_x_forwarded_proto",
            "==",
            "http"
        ]
    ],
    "plugins": {
        "redirect": {
            "uri": "https://$host$request_uri",
            "ret_code": 307
        }
    }
}'
```
