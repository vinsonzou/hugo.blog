---
title: "[Node] error:0308010C:digital envelope routines::unsupported"
subtitle: ""
date: 2023-12-03T11:00:00+08:00
lastmod: 2023-12-03T11:00:00+08:00
description: ""

tags: []
categories: ["Troubleshooting"]
---

## 环境
- Node: v16.20.2

## 错误日志
```sh
Error: error:0308010C:digital envelope routines::unsupported
```

## 解决方法
On Unix-like (Linux, macOS, Git bash, etc.)
```sh
export NODE_OPTIONS=--openssl-legacy-provider
```

## 参考
- https://stackoverflow.com/questions/69692842/error-message-error0308010cdigital-envelope-routinesunsupported