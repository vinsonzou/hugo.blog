---
title: "[Go] 踩坑系列之channel阻塞"
subtitle: ""
date: 2023-05-20T13:00:03+08:00
lastmod: 2023-05-20T13:00:03+08:00
description: ""

tags: ["Golang"]
categories: ["Golang"]
---

最近为了优化执行效率，使用`goroutine`并发处理，同时使用`channel`来采集 error 信息，结果踩坑了。示例代码如下：

```go
package main

import (
	"errors"
	"fmt"
	"sync"
)

func demo() error {
	errCh := make(chan error)

	var wg sync.WaitGroup
	wg.Add(1)

	go func() {
		defer wg.Done()

		for i := 0; i < 3; i++ {
			if i == 3 {
				errCh <- errors.New("错误测试")
				return
			}
		}
	}()

	go func() {
		wg.Wait()
	}()

	for err := range errCh {
		return err
	}

	return nil
}

func main() {
	err := demo()
	if err != nil {
		fmt.Println(err)
	}

	fmt.Println("ok")
}

// fatal error: all goroutines are asleep - deadlock!
```

> - i = 3，无 error 输出时，出现 deadlock。
> - i < 3，有 error 输出，运行正常。

## 原因说明

[RangeClause](https://go.dev/ref/spec#RangeClause)

> For channels, the iteration values produced are the successive values sent on the channel `until the channel is closed`. If the channel is nil, the range expression blocks forever.
> 使用 for range channel 时，只有 channel 被关掉才会结束。

## 解决
chan写结束后，调用`close(ch)`解决。
```go
package main

import (
	"errors"
	"fmt"
	"sync"
)

/**
个人理解：
    前提：无缓冲通道，没有容量，读写是阻塞的。意思就是写一个，必须读一个，否则写就会一直阻塞。
    for 为啥不会出错，因为手动的的控制了读取的次数。读写次数不一致一样死锁。
    for range 为啥报错，是因为没有close(ch)，已经没有值了。还读取就会一直阻塞，程序就会报死锁。
        手动的 close(ch) 掉后，range 在读完chan里的值后会自动结束循环。
**/

func demo() error {
	errCh := make(chan error)

	var wg sync.WaitGroup
	wg.Add(1)

	go func() {
		defer wg.Done()

		for i := 0; i < 3; i++ {
			if i == 3 {
				errCh <- errors.New("错误测试")
				return
			}
		}
	}()

	go func() {
		wg.Wait()
		// 这里生产者不再生产数据了就把ch关闭，这样来通知消费者，没数据了不用再等待了。
		close(errCh)
	}()

	for err := range errCh {
		return err
	}

	return nil
}

func main() {
	err := demo()
	if err != nil {
		fmt.Println(err)
	}

	fmt.Println("ok")
}
```

## channel 特性

- 给一个 nil channel 发送数据，造成永远阻塞
- 从一个 nil channel 接收数据，造成永远阻塞
- 给一个已经关闭的 channel 发送数据，引起 panic
- 从一个已经关闭的 channel 接收数据，如果缓冲区中为空，则返回一个零值
- 无缓冲的 channel 是同步的，而有缓冲的 channel 是非同步的

channel 状态与操作之间关系
|状态/操作|写操作|读操作|关闭|
|:-|:-|:-|:-|
|nil 状态|写阻塞|读阻塞|产生 panic(close of nil channel)|
|同步写阻塞|写阻塞|成功读取数据|进入关闭状态，产生 panic|
|同步读阻塞|成功写入数据|读阻塞|进入关闭状态|
|关闭状态|产生 panic|立即返回(nil, false)|产生 panic|
|队列写阻塞|写阻塞|成功读取队列中数据|进入关闭状态，成功写入队列的数据可读|
|队列读阻塞|成功写入数据|读阻塞|进入关闭状态|
|队列可读写|成功写入数据|成功读取数据|进入关闭状态，成功写入队列的数据可读|

## channel 延伸说明

golang 中的 channel 思路就是生产者消费者，无论生产者写入数据还是消费者读取数据都是阻塞的,理解这个的思路要基于阻塞这个前提。

fori 这种形式是自己判断从 channel 中读取多少次
for range 这种就是 runtime 帮我们来判断了，他的判断标准是 `close(ch)`

你的 fori 改成多一次循环同样会被 go 判定为 `deadlock`
因为最后一次的读取会一直阻塞在那里，原因是生产者不再生产了，消费者还阻塞在那里等待。go 判断到这个会一直阻塞在这里的场景就直接抛出错误退出了，否则这个进程就一直 hang 在这里还不易被发现。

导致这种错误的情况有两种

**生产者**

- 没有消费者消费 channel 中的数据，channel 中的数据已经填充满了，但是还在往里写入，此刻是要阻塞等待的，由于没有消费者，这个阻塞会一直阻塞下去

**消费者**

- 生产者不再生产数据了，也就是是 channel 中会一直为空了，但是消费者还在读取 channel 中的数据，这个读取也是阻塞等待的，channel 中不会再有数据，这个等待也是会一直等待下去

## 参考

- [go 通道 channel 使用 for-range 造成死锁，而使用 for 计数器迭代却不会？](https://segmentfault.com/q/1010000021869998)