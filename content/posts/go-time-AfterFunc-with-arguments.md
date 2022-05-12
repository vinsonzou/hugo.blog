---
title: "[Go] 将带参数的函数传递给time.AfterFunc"
subtitle: ""
date: 2022-05-08T19:30:03+08:00
lastmod: 2022-01-08T19:30:03+08:00
description: ""

tags: ["Golang"]
categories: ["Golang"]
---

`time.AfterFunc()` 接受持续时间和要在该持续时间到期时执行的函数。但函数不能是接受参数的函数。

例如：无法传递以下函数：
```go
func Foo (b *Bar) {}
```

但是，可以初始化调用上述函数的新函数，然后传递它：
```go
f := func() {
    Foo(somebar)
}
timer := time.AfterFunc(1*time.Second, f)
```

真的应该这样做吗？
为什么time.afterfunc不接受接受任何接受参数的函数？
是否存在其他/更好的方法？

最佳答案：

从参数创建一个函数，返回它。

```go
package main

import (
    "fmt"
    "time"
)

func foo(bar string) {
    fmt.Printf("in foo('%s')\n", bar)
}

func newFunc(bar string) func() {
    fmt.Printf("creating func with '%s'\n", bar)
    return func() {
        foo(bar)
    }
}

func main() {
    somebar := "Here we go!"
    f := newFunc(somebar)
    _ = time.AfterFunc(1*time.Second, f)

    time.Sleep(2 * time.Second)
}
```
