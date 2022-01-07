---
title: "[Go]实时监控日志文件的包tail"
subtitle: ""
date: 2022-01-07T09:48:03+08:00
lastmod: 2022-01-07T09:48:03+08:00
description: ""

tags: ["golang"]
categories: ["golang"]
---

在linux中有一个tail命令，`tail -f` 可以实时的监控文件新增加的内容，如果用代码实现这个逻辑，可以使用这个包
```bash
go get github.com/hpcloud/tail
```

示例代码
```go
package main

import (
    "fmt"

    "github.com/hpcloud/tail"
)

func main() {
    t, _ := tail.TailFile("/tmp/log.txt", tail.Config{Follow: true})
    for line := range t.Lines {
        fmt.Println(line.Text)
    }
}
```
