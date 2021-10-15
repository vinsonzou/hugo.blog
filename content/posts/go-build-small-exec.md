+++
title = "go如何编译出更小的执行文件?"
description = ""
date = "2017-08-23T23:38:09+08:00"
tags = ["golang"]
categories = ["golang"]

+++

前言
===

本地默认编译出的文件总与官方提供的二进制文件大很多，Google之后得知通过编译参数控制还能编译出更小的可执行文件。

加-ldflags参数
==============

在程序编译的时候可以加上`-ldflags "-s -w"` 参数来优化编译程序, 其实通过去除部分连接和调试等信息来使得编译之后的执行程序更小,具体参数如下:

* -a 强制编译所有依赖包
* -s 去掉符号表信息, panic时候的stack trace就没有任何文件名/行号信息了
* -w 去掉DWARF调试信息，得到的程序就不能用gdb调试了

测试代码如下
```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, 世界")
}
```

编译方式及文件大小对比结果如下

|编译参数                      | 大小   |
|-|-|
|go build(默认)              | 1.6M |
|go build -ldflags -s      | 1.6M |
|go build -ldflags "-s -w" | 1.1M |
|go build -ldflags -w      | 1.1M |

> 测试环境: go 1.8.3 on macOS 10.12.6
> 不建议s和w同时使用。

使用upx
=======

上面go build 时加上-ldflags参数得到了比较小的可执行程序，但是还可以通过upx这个开源，绿色，好用的压缩工具进行进一步压缩。

Mac用户直接brew安装

```sh
brew install upx
```

upx使用效果如下

```sh
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2017
UPX 3.94        Markus Oberhumer, Laszlo Molnar & John Reiser   May 12th 2017

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
   1181728 ->    438928   37.14%   macho/amd64   test

Packed 1 file.
```

可以看到通过upx进一步压缩之后得到的程序只有429K了，压缩比率达到了37.14%.
