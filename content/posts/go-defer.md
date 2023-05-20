---
title: "[Go] defer学习"
subtitle: ""
date: 2023-05-20T14:02:03+08:00
lastmod: 2023-05-20T14:02:03+08:00
description: ""

tags: ["Golang"]
categories: ["Golang"]
---

在 Go 语言中使用 `defer` 关键字可以将代码延迟到函数结束之前执行。在开发中，我们经常使用 defer 关键字完成善后工作，如关闭打开的文件描述符、关闭连接以及释放资源等。

```go
func demo0() {
	fileName := "./demo.txt"
	f, _ := os.OpenFile(fileName, os.O_RDONLY, 0)
	defer f.Close()

	contents, _ := io.ReadAll(f)
	fmt.Println(string(contents))
}
```

`defer` 关键字一般紧跟在打开资源代码的后面，防止后续忘记释放资源，defer 声明的代码实际上要等到函数结束之前才会被执行。defer 虽然简单易用，但如果忽略了它的特性，就会在开发中面临困惑。针对实际使用中，defer 出现的各种场景，直接上 demo，便于理解。

## 0x01：先进后出

使用多个 defer 关键字时，先被声明的 defer 语句后被调用。类似于 “栈” 先进后出的特性，defer 的这一特性也很好理解，先被打开的资源，可能会被后续代码依赖，所以要后释放才安全。

```go
func demo1() {
	for i := 0; i < 5; i++ {
		defer fmt.Println("defer:", i)
	}
}

// defer: 4
// defer: 3
// defer: 2
// defer: 1
// defer: 0
```

## 0x02：作用域仅为当前函数

运行 demo2 ，从结果中可以看出，第一个匿名函数和第二个匿名函数的 defer 执行顺序没有关系。
defer 作用域仅为当前函数，在当前函数最后执行，所以不同函数下拥有不同的 defer 栈。

```go
func demo2() {
	func() {
		defer fmt.Println(1)
		defer fmt.Println(2)
	}()

	fmt.Println("=== 测试 ===")

	func() {
		defer fmt.Println("a")
		defer fmt.Println("b")
	}()

}

// 2
// 1
// === 测试 ===
// b
// a
```

## 0x03：defer 后的函数形参在声明时确认（预计算参数）

运行 demo3_1 ，根据结果，我们可以得出：defer 在声明时，就已经确认了形参 n 的值，而不是在执行时确认的；所以，后续变量 num 无论如何改变都不影响 defer 的输出结果。

```go
func demo3_1() {
	num := 0
	defer func(n int) {
		fmt.Println("defer:", n)
	}(num)
	// 等同 defer fmt.Println("defer:", num)

	for i := 0; i < 10; i++ {
		num++
	}

	fmt.Println(num)

}

//10
//defer: 0
```

运行 demo3_2，为什么这里 defer 的最终输出的结果会和变量 num 相同？因为这里使用的是指针。
defer 声明时，已经确认了形参 p 指针的指向地址，指向变量 num；后续变量 num 发生改变。所以在 defer 执行时，输出的是 p 指针指向的变量 num 的当前值。

```go
func demo3_2() {
	num := 0
	p := &num
	defer func(p *int) {
		fmt.Println("defer:", *p)
	}(p)

	for i := 0; i < 10; i++ {
		num++
	}

	fmt.Println(*p)
}

//10
//defer: 10
```

再看一下 demo3_3，defer 打印的变量并没有通过函数参数传入，在 defer 执行时，才获取的”全局变量”num，所以 defer 输出结果与变量 num 一致。

```go
func demo3_3() {
	num := 0
	defer func() {
		fmt.Println("defer:", num)
	}()

	for i := 0; i < 10; i++ {
		num++
	}

	fmt.Println(num)
}

// 10
// defer: 10
```

## 0x04：return 先 defer 后

运行 demo4_1，可以发现 defer、return 都是在函数最后执行，但 return 先于 defer 执行；

```go
func demo4_1() (int, error) {
	defer fmt.Println("defer")
	return fmt.Println("return")
}

// return
// defer
```

这一点从输出结果上显而易见，但当 return、defer 的执行顺序和**函数返回值**“相遇” 时，又将会产生许多复杂的场景。
在 demo4_2 中，函数使用命名返回值，最终输出结果为 7。其中经历了这几个过程：

（首先）变量 num 作为返回值，初始值为 0；

（其次）随后变量 num 被赋值为 10；

（然后）return 时，变量 num 作为返回值被重新赋值为 2；

（接着）defer 在 return 后执行，拿到变量 num 进行修改，值为 7；

（最后）变量 num 作为返回值，最终函数返回结果为 7；

```go
func demo4_2() (num int) {
	num = 10
	defer func() {
		num += 5
	}()

	return 2
}

// 7
```

再来看一个例子。
在 demo4_3 中，函数使用匿名返回值，最终结果输出为 2。其中经历的过程是这样的：

进入函数，此时返回值变量并未创建；

创建变量 num，赋值为 10；

return 时创建函数返回值变量，并赋值为 2；这个返回值变量你可以把它看成匿名变量，或者是变量 a、b、c、d……，但它就不是变量 num；

defer 时，无论怎样修改变量 num，都与函数返回值无关；

所以，最终的函数返回结果为 2；

```go
func demo4_3() int {
	num := 10
	defer func() {
		num += 5
	}()

	return 2
}

// 2
```

## 0x05：panic 捕获，防止奔溃

运行 demo5_1，可以看到当出现 panic 时，会触发已经声明的 defer 出栈执行，随后在再 panic，而在 panic 之后声明的 defer 将得不到执行。

```go
func demo5_1() {
	defer fmt.Println(1)
	defer fmt.Println(2)
	defer fmt.Println(3)

	panic("异常触发") // 触发 defer 出栈执行

	defer fmt.Println(4) // 得不到执行
}
```

正是利用这个特性，在 defer 中可以通过 recover 捕获 panic，防止程序崩溃。

```go
func demo5_2() {
	defer func() {
		if err := recover(); err != nil {
			fmt.Println(err, "问题不大")
		}
	}()

	panic("异常触发") // 触发 defer 出栈执行

	// ...
}
```