---
title: "[Go] mTLS client示例"
subtitle: ""
date: 2023-02-25T13:40:00+08:00
lastmod: 2023-02-25T13:40:00+08:00
description: ""

tags: ["Golang"]
categories: ["Golang"]
---

## 环境
- go 1.20

## GET请求示例

```go
package main

import (
	"crypto/tls"
	"crypto/x509"
	"io"
	"log"
	"net/http"
	"os"
)

func main() {
	// 加载证书和密钥
	cert, err := tls.LoadX509KeyPair("client.crt", "client.key")
	if err != nil {
		log.Fatal(err)
	}

	// 加载服务器的根证书
	caCert, err := os.ReadFile("ca.crt")
	if err != nil {
		log.Fatal(err)
	}
	caCertPool := x509.NewCertPool()
	caCertPool.AppendCertsFromPEM(caCert)

	// 配置TLS客户端
	tlsConfig := &tls.Config{
		Certificates: []tls.Certificate{cert},
		RootCAs:      caCertPool,
	}

	// 创建HTTP客户端
	client := &http.Client{
		Transport: &http.Transport{
			TLSClientConfig: tlsConfig,
		},
	}

	// 发送HTTP请求
	resp, err := client.Get("https://example.com")
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()

	// 处理响应
	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}
	log.Println(string(body))
}
```

> - 在这个示例代码中，首先使用`tls.LoadX509KeyPair`函数加载客户端的证书和密钥，然后使用`os.ReadFile`函数加载服务器的根证书，并将其添加到一个`x509.CertPool`对象中。然后，创建一个`tls.Config`对象，其中包含客户端的证书和服务器的根证书，并将其传递给http.Transport对象的TLSClientConfig字段。最后，创建一个http.Client对象，并使用它来发送HTTP请求。
> - 需要注意的是，示例代码中使用的是http.Get函数发送HTTP请求。如果需要发送其他类型的请求，可以使用http.NewRequest函数创建一个新的请求对象，然后调用client.Do方法来发送请求。

## POST请求示例
```go
package main

import (
	"bytes"
	"crypto/tls"
	"crypto/x509"
	"io"
	"log"
	"net/http"
	"os"
)

func main() {
	// 加载证书和密钥
	cert, err := tls.LoadX509KeyPair("client.crt", "client.key")
	if err != nil {
		log.Fatal(err)
	}

	// 加载服务器的根证书
	caCert, err := os.ReadFile("ca.crt")
	if err != nil {
		log.Fatal(err)
	}
	caCertPool := x509.NewCertPool()
	caCertPool.AppendCertsFromPEM(caCert)

	// 配置TLS客户端
	tlsConfig := &tls.Config{
		Certificates: []tls.Certificate{cert},
		RootCAs:      caCertPool,
	}

	// 创建HTTP客户端
	client := &http.Client{
		Transport: &http.Transport{
			TLSClientConfig: tlsConfig,
		},
	}

	// 准备POST请求数据(与GET请求不同的部分)
	data := []byte(`{"name":"John Doe","age":30}`)
	req, err := http.NewRequest("POST", "https://example.com/api", bytes.NewBuffer(data))
	if err != nil {
		log.Fatal(err)
	}
	req.Header.Set("Content-Type", "application/json")

	// 发送HTTP请求
	resp, err := client.Do(req)
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()

	// 处理响应
	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}
	log.Println(string(body))
}
```

> - 需要注意的是，在创建请求对象时，需要将请求数据作为`bytes.Buffer`对象传递给`http.NewRequest`函数。在设置请求头时，我们设置了Content-Type为application/json，这表明我们要发送的是一个JSON格式的数据。
