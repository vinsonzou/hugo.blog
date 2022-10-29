---
title: "[Go] 1.18新增库之net/netip"
subtitle: ""
date: 2022-10-16T14:01:00+08:00
lastmod: 2022-10-16T14:01:00+08:00
description: ""

tags: ["Golang"]
categories: ["Golang"]
---

go1.18标准库都支持了，还直接支持ipv6判断包含之类的。

示例如下

```go
package main

import (
    "fmt"
    "net/netip"
)

func main() {
    p, err := netip.ParsePrefix(`10.10.10.0/24`)
    if err != nil {
        panic(err)
    }
    a, err := netip.ParseAddr(`10.10.10.6`)
    if err != nil {
        panic(err)
    }
    fmt.Println(p.Contains(a))
}
```
