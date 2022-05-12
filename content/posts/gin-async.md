---
title: "[Go] Gin异步协程"
subtitle: ""
date: 2022-01-07T09:30:03+08:00
lastmod: 2022-01-07T09:30:03+08:00
description: ""

tags: ["Golang"]
categories: ["Golang"]
---

golang的高并发一大利器就是协程。gin里可以借助协程实现异步任务。


```go
package main
import (
    "log"
    "time"

    "github.com/gin-gonic/gin"
)


func main(){
    router := gin.Default()

    router.GET("/sync", func(c *gin.Context) {
        time.Sleep(5 * time.Second)
        log.Println("Done! in path" + c.Request.URL.Path)
    })

    router.GET("/async", func(c *gin.Context) {
        // 因为涉及异步过程，请求的上下文需要copy到异步的上下文，并且这个上下文是只读的。
        cCp := c.Copy()
        go func() {
            time.Sleep(5 * time.Second)
            log.Println("Done! in path" + cCp.Request.URL.Path)
        }()
    })

    router.Run(":8080")
}
```
