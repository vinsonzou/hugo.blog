---
title: "[Go] 函数参数传递的sync.Mutex不是指针会怎么样"
subtitle: ""
date: 2022-09-15T09:39:03+08:00
lastmod: 2022-09-15T09:39:03+08:00
description: ""

tags: ["Golang"]
categories: ["Golang"]
---

## 函数传值

```go
package main

import (
	"fmt"
	"sync"
)

var a = 1

func t(lock sync.Mutex, wg *sync.WaitGroup) {
	defer wg.Done()
	lock.Lock()
	defer lock.Unlock()
	for i := 0; i < 10000000; i++ {
		a++
	}
}

func main() {
	lock := sync.Mutex{}
	wg := &sync.WaitGroup{}
	wg.Add(2)
	go t(lock, wg)
	go t(lock, wg)
	wg.Wait()
	fmt.Println(a)
}
```

输出

```go
// 每次输出值都不一样
10149117
```

## 函数传指针

```go
package main

import (
	"fmt"
	"sync"
)

var a = 1

func t(lock *sync.Mutex, wg *sync.WaitGroup) { // 改成指针
	defer wg.Done()
	lock.Lock()
	defer lock.Unlock()
	for i := 0; i < 10000000; i++ {
		a++
	}
}

func main() {
	lock := &sync.Mutex{} // 改成指针
	wg := &sync.WaitGroup{}
	wg.Add(2)
	go t(lock, wg)
	go t(lock, wg)
	wg.Wait()
	fmt.Println(a)
}
```

输出

```go
// 每次输出值都固定
20000001
```

这才是正确的结果。

## 结论

函数传参会发生值拷贝(也就是复制了锁的状态)，所以一定注意在不同函数内操作`同一个锁`时一定要使用`指针`进行传递。
