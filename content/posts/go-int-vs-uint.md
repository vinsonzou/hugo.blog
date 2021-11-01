---
title: "Golang中uint、int, int8, int16, int32, int64区别"
subtitle: ""
date: 2021-10-20T16:47:09+08:00
lastmod: 2021-11-01T10:10:00+08:00
description: ""

tags: ["golang"]
categories: ["golang"]
---

Golang各种数值占据的大小

```
int   类型大小为 8 字节
int8  类型大小为 1 字节
int16 类型大小为 2 字节
int32 类型大小为 4 字节
int64 类型大小为 8 字节
```
> go语言中的int的大小是和操作系统位数相关的，如果是32位操作系统，int类型的大小就是4字节; 如果是64位操作系统，int类型的大小就是8个字节

数据类型占据的范围

```
int8:   -128 ~ 127
int16:  -32768 ~ 32767
int32:  -2147483648 ~ 2147483647
int64:  -9223372036854775808 ~ 9223372036854775807
uint8:  0 ~ 255
uint16: 0 ~ 65535
uint32: 0 ~ 4294967295
uint64: 0 ~ 18446744073709551615
```
> 由于Go语言中各int类型的取值范围不同，各int类型间进行数据转换时，会存在数据截断的问题，在使用过程中要引起注意。

拓展:

**字符串和各种int类型之间的相互转换方式**

- string转成int
    - int, err := strconv.Atoi(string)

- string转成int64
    - int64, err := strconv.ParseInt(string, 10, 64)

- string转成uint64
    - uint64, err := strconv.ParseUint(string, 10, 64)

- int转成string
    - string := strconv.Itoa(int)

- int64转成string
    - string := strconv.FormatInt(int64, 10)

- uint64转成string(10进制)
    - string := strconv.FormatUint(uint64, 10)

- uint64转成string(16进制)
    - string := strconv.FormatUint(uint64, 16)
