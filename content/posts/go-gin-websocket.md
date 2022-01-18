---
title: "[Go] 使用gin+websocket实现日志实时输出"
subtitle: ""
date: 2022-01-07T15:48:03+08:00
lastmod: 2022-01-07T15:48:03+08:00
description: ""

tags: ["golang"]
categories: ["golang"]
---

## 背景

由于工作需要，在web端执行相关的部署操作，能够在页面实时展示部署任务的实时日志信息，使用到websocket来实现。

websocket通信特点

- 全双工通信协议
- 允许服务端主动向客户端推送数据
- 浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。

示例

```go
package main

import (
	"log"
	"net/http"
	"time"

	"github.com/gin-gonic/gin"
	"github.com/gorilla/websocket"
)

var upgrader = websocket.Upgrader{
	// 这个是校验请求来源
	// 在这里我们不做校验，直接return true
	CheckOrigin: func(r *http.Request) bool {
		return true
	},
}

func main() {
	route := gin.Default()

	route.GET("/helloWebSocket", func(context *gin.Context) {
		// 将普通的http GET请求升级为websocket请求
		ws, _ := upgrader.Upgrade(context.Writer, context.Request, nil)
		for {
			// 每隔两秒给前端推送一句消息“hello, WebSocket”
			err := ws.WriteMessage(websocket.TextMessage, []byte("hello, WebSocket"))
			if err != nil {
				log.Println(err)
			}
			time.Sleep(time.Second * 2)
		}
	})

	err := route.Run()
	if err != nil {
		log.Fatalln(err)
	}
}
```

写完以后你可以用websocket在线测试工具测试你的代码: http://coolaf.com/tool/chattest

![](/img/gin-websocket-demo.png)


参考:
- https://huangzhongde.cn/post/Golang/%E4%BD%BF%E7%94%A8gin+websocket%E5%AE%9E%E7%8E%B0%E4%BB%BB%E5%8A%A1%E7%9A%84%E5%AE%9E%E6%97%B6%E6%97%A5%E5%BF%97/
- https://github.com/plholx/web-tail
- http://45.63.114.236/2021/04/go-vue%E5%AE%9E%E7%8E%B0web%E5%AE%9E%E6%97%B6%E6%89%93%E5%8D%B0%E4%B8%BB%E6%9C%BA%E6%97%A5%E5%BF%97%E4%B8%80/
- http://45.63.114.236/2021/07/go-vue%E5%AE%9E%E7%8E%B0web%E5%AE%9E%E6%97%B6%E6%89%93%E5%8D%B0%E4%B8%BB%E6%9C%BA%E6%97%A5%E5%BF%97%E4%BA%8C/
- https://blog.csdn.net/qq_42887507/article/details/120230212
