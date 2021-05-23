+++
date = "2017-08-16T23:28:33+08:00"
title = "Golang之command line flag笔记"
description = ""
topics = []
tags = ["golang"]

+++

**示例代码，仅供参考**

```go
package main

import (
    "fmt"
    "flag"
)

func main() {
    // golang的flag包的一些基本使用方法

    // 待使用的变量
    var id int
    var name string
    var male bool

    // 是否已经解析
    fmt.Println("parsed? = ", flag.Parsed())

    // 设置flag参数 (变量指针，参数名，默认值，帮助信息)
    // 也可以用以下带返回值的方法代替，不过他们返回的是指针，比较麻烦点
    // Int(name string, value int, usage string) *int
    // String(name string, value string, usage string) *string
    // Bool(name string, value bool, usage string) *bool
    flag.IntVar(&id, "id", 123, "help msg for id")
    flag.StringVar(&name, "name", "default name", "help msg for name")
    flag.BoolVar(&male, "male", false, "help msg for male")

    // 解析
    flag.Parse()

    // 是否已经解析
    fmt.Println("parsed? = ", flag.Parsed())

    // 获取非flag参数
    fmt.Println("------ Args start ------")
    for i, v := range flag.Args() {
        fmt.Printf("arg[%d] = (%s).\n", i, v)
    }
    fmt.Println("------ Args end ------")

    // visit只包含已经设置了的flag
    fmt.Println("------ visit flag start ------")
    flag.Visit(func(f *flag.Flag) {
        fmt.Println(f.Name, f.Value, f.Usage, f.DefValue)
    })
    fmt.Println("------ visit flag end ------")

    // visitAll只包含所有的flag(包括未设置的)
    fmt.Println("------ visitAll flag start ------")
    flag.VisitAll(func(f *flag.Flag) {
        fmt.Println(f.Name, f.Value, f.Usage, f.DefValue)

    })
    fmt.Println("------ visitAll flag end ------")

    // flag参数
    fmt.Printf("id = %d\n", id)
    fmt.Printf("name = %s\n", name)
    fmt.Printf("male = %t\n", male)

    // flag参数默认值
    fmt.Println("------ PrintDefaults start ------")
    flag.PrintDefaults()
    fmt.Println("------ PrintDefaults end ------")

    // 非flag参数个数
    fmt.Printf("NArg = %d\n", flag.NArg())
    // 已设置的flag参数个数
    fmt.Printf("NFlag = %d\n", flag.NFlag())
}
```

**传参数go run**

```go
go run flag_demo.go -id=123 -name="zhangsan" -male=true p1 p2 p3 p4
```

**示例输出结果**

```go
parsed? =  false
parsed? =  true
------ Args start ------
arg[0] = (p1).
arg[1] = (p2).
arg[2] = (p3).
arg[3] = (p4).
------ Args end ------
------ visit flag start ------
id 123 help msg for id 123
male true help msg for male false
name zhangsan help msg for name default name
------ visit flag end ------
------ visitAll flag start ------
id 123 help msg for id 123
male true help msg for male false
name zhangsan help msg for name default name
------ visitAll flag end ------
id = 123
name = zhangsan
male = true
------ PrintDefaults start ------
  -id int
    	help msg for id (default 123)
  -male
    	help msg for male
  -name string
    	help msg for name (default "default name")
------ PrintDefaults end ------
NArg = 4
NFlag = 3
```

**参考**

* https://gobyexample.com/command-line-flags
