---
title: "[Go] json的html转义问题"
subtitle: ""
date: 2022-05-13T10:11:03+08:00
lastmod: 2022-05-13T10:11:03+08:00
description: ""

tags: ["Golang"]
categories: ["Golang"]
---

## 环境

- json库：github.com/goccy/go-json

## 问题

go语言提供了json的编解码包，json字符串作为参数值传输时发现，json.Marshal生成json特殊字符<、>、&会被转义。

```go
package main

import (
	"fmt"
	json "github.com/goccy/go-json"
)

type Test struct {
	Content string
}

func main() {
	t := new(Test)
	t.Content = "https://www.baidu.com?id=12&test=23"
	jsonByte, _ := json.Marshal(t)
	fmt.Println(string(jsonByte))
}
```

输出

```
{"Content":"https://www.baidu.com?id=12\u0026test=23"}
```

## 解决方法

```go
package main

import (
	"fmt"
	json "github.com/goccy/go-json"
)

type Test struct {
	Content string
}

func main() {
	t := new(Test)
	t.Content = "https://www.baidu.com?id=12&test=23"
	jsonByte, _ := json.MarshalWithOption(t, json.DisableHTMLEscape())
	fmt.Println(string(jsonByte))
}
```

输出

```
{"Content":"https://www.baidu.com?id=12&test=23"}
```

