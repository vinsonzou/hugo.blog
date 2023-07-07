---
title: "[Go] 使用函数选项模式"
subtitle: ""
date: 2023-06-21T12:42:00+08:00
lastmod: 2023-06-21T14:42:00+08:00
description: ""

tags: ["Golang"]
categories: ["Golang"]
---

Go 不是完全面向对象语言，有一些面向对象模式不太适合它。但经过这些年的发展，Go 有自己的一些模式。今天介绍一个常见的模式：函数选项模式（Functional Options Pattern）。

## 0x01 什么是函数选项模式

Go 语言没有构造函数，一般通过定义 New 函数来充当构造函数。然而，如果结构有较多字段，要初始化这些字段，有很多种方式，但有一种方式认为是最好的，这就是函数选项模式（Functional Options Pattern）。

函数选项模式是一种在 Go 中构造结构体的模式，它通过设计一组非常有表现力和灵活的 API 来帮助配置和初始化结构体。

## 0x02 一个示例

为了更好的理解该模式，我们通过一个例子来讲解。

定义一个 Server 结构体：

```go
package main

type Server struct {
  host string
  port int
}

func New(host string, port int) *Server {
  return &Server{host, port}
}

func (s *Server) Start() error {
}
```

如何使用呢？

```go
package main

import (
  "log"
  "server"
)

func main() {
  svr := New("localhost", 1234)
  if err := svr.Start(); err != nil {
    log.Fatal(err)
  }
}
```
但如果要扩展 Server 的配置选项，如何做？通常有三种做法：

- 为每个不同的配置选项声明一个新的构造函数
- 定义一个新的 Config 结构体来保存配置信息
- 使用 Functional Option Pattern

来，我们直接切入主题，使用Functional Option Pattern

### 使用 Functional Option Pattern
在这个模式中，我们定义一个 Option 函数类型：
```go
type Option func(*Server)
```

Option 类型是一个函数类型，它接收一个参数：*Server。然后，Server 的构造函数接收一个 Option 类型的不定参数：
```go
func New(options ...Option) *Server {
  svr := &Server{}
  for _, f := range options {
    f(svr)
  }
  return svr
}
```
那选项如何起作用？需要定义一系列相关返回 Option 的函数：
```go
func WithHost(host string) Option {
  return func(s *Server) {
    s.host = host
  }
}

func WithPort(port int) Option {
  return func(s *Server) {
    s.port = port
  }
}

func WithTimeout(timeout time.Duration) Option {
  return func(s *Server) {
    s.timeout = timeout
  }
}

func WithMaxConn(maxConn int) Option {
  return func(s *Server) {
    s.maxConn = maxConn
  }
}
```

针对这种模式，客户端类似这么使用：
```go
package main

import (
  "log"
  
  "server"
)

func main() {
  svr := New(
    WithHost("localhost"),
    WithPort(8080),
    WithTimeout(time.Minute),
    WithMaxConn(120),
  )
  if err := svr.Start(); err != nil {
    log.Fatal(err)
  }
}
```
将来增加选项，只需要增加对应的 WithXXX 函数即可。

## 0x03 参考
- https://golang.cafe/blog/golang-functional-options-pattern.html
- https://github.com/uber-go/guide/blob/master/style.md#functional-options
