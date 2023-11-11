---
title: "luajit 64位位运算"
subtitle: ""
date: 2023-11-10T10:40:03+08:00
lastmod: 2023-11-10T10:40:03+08:00
description: ""

tags: ["OpenResty"]
categories: ["OpenResty"]
---


春哥说的，luajit 支持64位位运算。通过 `int64_t` 或者 `uint64_t` 类型的 FFI cdata 类型。比如

```lua
$ luajit -e 'local bit = require "bit" print(bit.lshift(0xffffffffffLL, 1))'
2199023255550LL
```

- 参考
    - https://forum.openresty.us/d/2827-718049530f7d8de817ffe41defbfd6b5
