---
title: "[Go] json库之sonic"
subtitle: ""
date: 2023-02-25T11:10:00+08:00
lastmod: 2023-02-25T11:10:00+08:00
description: ""

tags: ["Golang"]
categories: ["Golang"]
---

前几日发布的gin v1.9.0版本json库多了个选择，就是字节开源的[sonic](https://github.com/bytedance/sonic)，支持JIT(just-in-time compiling)和SIMD(single-instruction-multiple-data)加速。

本文目的就是测试[go-json](https://github.com/goccy/go-json)替换为sonic，需要哪些调整。

## 环境
- go 1.20

## 依赖

- Go 1.15~1.20
- Linux/MacOS/Windows
- Amd64 ARCH

## 已有用法切换

### html转义
```go
content := "https://www.baidu.com?id=12&test=23"

// go-json
gojsonByte, _ := gojson.MarshalWithOption(content, gojson.DisableHTMLEscape())

// sonic(默认不转义)
sonicByte, _ := sonic.Marshal()

// sonic转义
ret, _ := encoder.Encode(content, encoder.EscapeHTML)
```
> 看需求是否开启Escape HTML，开启后有15%性能损失。

### key排序
```go
m := map[int]string{1: "1", 11: "11", 10: "10<", 2: "c"}

// go-json
gojson_v, err := gojson.Marshal(m)
fmt.Println(string(gojson_v))

// sonic
v0, err := sonic.Marshal(m) // 输出key顺序随机、html不转义
fmt.Println(string(v0))

v1, err := sonic.ConfigDefault.Marshal(m) // 按map已有顺序输出(以效率和安全为目标)
fmt.Println(string(v1))

v2, err := sonic.ConfigFastest.Marshal(m) // 按map已有顺序输出(以速度为目标)
fmt.Println(string(v2))

v3, err := sonic.ConfigStd.Marshal(m)   // 与标准库输出一致(兼容json标准库)
fmt.Println(string(v3))

v4, err := encoder.Encode(m, encoder.SortMapKeys) // 按key排序
fmt.Println(string(v4))
```

> 看需求是否开启key排序(按字符串排序)，开启后有10%性能损失。

### 缩进

```go
m := map[int]string{1: "1", 11: "11", 10: "10<", 2: "c"}

// go-json
gojsonByte, _ := gojson.MarshalIndentWithOption(m, "", "\t", json.DisableHTMLEscape())

// sonic
sonicByte, _ := encoder.EncodeIndented(m, "", "\t", encoder.SortMapKeys)
```

