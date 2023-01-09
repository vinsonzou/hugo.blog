---
title: "[Go] 使用execCommand时，如何关闭子进程"
subtitle: ""
date: 2022-12-26T13:30:00+08:00
lastmod: 2022-12-26T13:30:00+08:00
description: ""

tags: ["Golang"]
categories: ["Golang"]
---

## 需求

Unity打包调度程序，正在打包的任务，可以发送指令取消，每个任务都有唯一的任务Id。

## 实现

本文实现仅支持 Linux/Mac，暂不支持Windows。

```go
package main

import (
	"context"
	"log"
	"os/exec"
	"syscall"
	"time"

	"github.com/gin-gonic/gin"
)

var taskContext map[string]context.Context
var taskContextCancel map[string]context.CancelFunc

func main() {
	taskContext = make(map[string]context.Context)
	taskContextCancel = make(map[string]context.CancelFunc)

	r := gin.Default()

	r.GET("/hello", func(c *gin.Context) {
		taskId := c.Query("taskId")

		ctx, cancel := context.WithCancel(context.Background())
		taskContext[taskId] = ctx
		taskContextCancel[taskId] = cancel

		timer := time.NewTimer(1 * time.Second)
		go func(t *time.Timer) {
			timeout := time.After(40 * time.Second)
			doneCh := make(chan struct{})
			errStopCh := make(chan struct{})

			cmd := exec.CommandContext(ctx, "/bin/bash", "-c", "ping www.baidu.com >/tmp/top.log")
			// 将PGID设置成与PID相同的值(不适用 su - <user> command)
			cmd.SysProcAttr = &syscall.SysProcAttr{Setpgid: true}

			for {
				select {
				case <-t.C:
					// 业务逻辑
					if err := cmd.Start(); err != nil {
						return
					}

					go func() {
						if err := cmd.Wait(); err != nil {
							errStopCh <- struct{}{}
							return
						}

						doneCh <- struct{}{}
					}()
				case <-doneCh:
					// 业务逻辑结束后逻辑
					log.Println("task Done --- " + taskId)
					return
				case <-errStopCh:
					// 业务逻辑报错退出
					log.Println("task ErrDone --- " + taskId)
					return
				case <-taskContext[taskId].Done():
					// 取消逻辑
					syscall.Kill(-cmd.Process.Pid, syscall.SIGKILL)
					log.Println("task Exit --- " + taskId)
					return
				case <-timeout:
					// timeout
					syscall.Kill(-cmd.Process.Pid, syscall.SIGKILL)
					log.Println("task Timeout --- " + taskId)
					return
				}
			}
		}(timer)

		c.JSON(200, gin.H{
			"message": "Hello world!",
		})
	})

	r.GET("/cancel", func(c *gin.Context) {
		taskId := c.Query("taskId")
		taskContextCancel[taskId]()

		c.JSON(200, gin.H{
			"message": "cancel work!",
		})
	})

	r.Run("127.0.0.1:8080")
}
```

## 参考
- [如何避免 Go 命令行执行产生“孤儿”进程？](https://developer.aliyun.com/article/786841)
