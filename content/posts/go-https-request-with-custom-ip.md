---
title: "[Go] 使用指定IP请求HTTPS"
subtitle: ""
date: 2023-03-11T15:01:00+08:00
lastmod: 2023-03-11T15:01:00+08:00
description: ""

tags: ["Golang"]
categories: ["Golang"]
---

## 环境
- go 1.20

## 需求

比如要请求一个域名`test.com`, 希望DNS解析到指定IP`1.1.1.1`，如何设置http.Client呢？
- 因权限限制，不能改HOSTS文件

## 实现

curl实现
```sh
curl --resolve test.com:443:1.1.1.1 https://test.com
```

go实现
```go
package main

import (
	"context"
	"fmt"
	"io"
	"net"
	"net/http"
	"strings"
)

func main() {
	client := &http.Client{
		Transport: &http.Transport{
			DialContext: func(ctx context.Context, network, addr string) (net.Conn, error) {
				// addr返回: test.com:443
				addrArr := strings.Split(addr, ":")
				ServerName := addrArr[0]
				ServerPort := addrArr[1]
				if ServerName == "test.com" {
					ServerName = "1.1.1.1" // 自定义当前http.Client的连接
				}
				addr = ServerName + ":" + ServerPort
				var d net.Dialer
				return d.DialContext(ctx, network, addr)
			},
		},
	}

	resp, err := client.Get("https://test.com")
	if err != nil {
		fmt.Println("Error sending request:", err)
		return
	}
	defer resp.Body.Close()

	// 读取响应体中的数据
	body, err := io.ReadAll(resp.Body)
	if err != nil {
		fmt.Println("Error reading response:", err)
		return
	}

	fmt.Println("Response:", string(body))
}
```
