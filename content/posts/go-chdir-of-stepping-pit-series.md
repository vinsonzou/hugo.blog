---
title: "[Go] 踩坑系列之os.Chdir"
subtitle: ""
date: 2023-06-20T09:20:00+08:00
lastmod: 2023-06-20T09:20:00+08:00
description: ""

tags: ["Golang"]
categories: ["Golang"]
---

## 场景

当前的打包调度程序，同时出ipa/apk时(2个goroutine)，总是失败，单独执行时则没有影响。

因运维习惯，把ipa/apk打包的脚本转为go代码执行，基本脚本如下

```shell
cd gitDir && git clean -xdf && git restore . && git pull && fastlane build
```

转换为go实现如下

```go
package main

import (
	"fmt"
	"os"
	"os/exec"
)

func main() {
	gitDir := "/path/to/gitDir"
  
  os.Chdir(gitdir)

	cmd1 := exec.Command("git", "clean", "-xdf")
	cmd2 := exec.Command("git", "restore", ".")
	cmd3 := exec.Command("git", "pull")
	cmd4 := exec.Command("fastlane", "build")

	// 使用管道将命令串联起来
	cmd2.Stdin, _ = cmd1.StdoutPipe()
	cmd3.Stdin, _ = cmd2.StdoutPipe()
	cmd4.Stdin, _ = cmd3.StdoutPipe()

	// 设置标准输出和标准错误输出
	cmd4.Stdout = os.Stdout
	cmd4.Stderr = os.Stderr

	// 启动命令
	if err := cmd4.Start(); err != nil {
		fmt.Println("Error starting command:", err)
		return
	}

	// 等待命令结束
	if err := cmd4.Wait(); err != nil {
		fmt.Println("Command failed:", err)
		return
	}
}
```

## 说明

在 Go 中，`os.Chdir` 函数可以用于改变当前工作目录。如果多个 goroutine 并发地执行 `os.Chdir` 函数，会出现竞态条件，并且可能会导致程序出现异常行为。

具体来说，当多个 goroutine 同时执行 `os.Chdir` 函数时，由于该函数会改变全局状态（即当前工作目录），因此这些 goroutine 可能会产生互相干扰的情况。例如，一个 goroutine 执行了 `os.Chdir` 函数后，会改变全局的当前工作目录，而其他 goroutine 也可能会依赖这个目录进行操作。此时，如果其他 goroutine 没有及时更新自己的工作目录，就有可能导致程序出现异常行为。

为了避免这种情况，在使用`exec.Command`函数时，可以采取以下措施：

- 指定工作目录，如`cmd.Dir = gitDir`
- 对应命令自带指定工作目录功能，如git，`git -C gitDir pull`

## 解决
使用`exec.Command`函数指定工作目录解决。
```go
package main

import (
	"fmt"
	"os"
	"os/exec"
)

func main() {
	gitDir := "/path/to/gitDir"

	cmd1 := exec.Command("git", "clean", "-xdf")
	cmd1.Dir = gitDir

	cmd2 := exec.Command("git", "restore", ".")
	cmd2.Dir = gitDir

	cmd3 := exec.Command("git", "pull")
	cmd3.Dir = gitDir

	cmd4 := exec.Command("fastlane", "build")
	cmd4.Dir = gitDir

	// 使用管道将命令串联起来
	cmd2.Stdin, _ = cmd1.StdoutPipe()
	cmd3.Stdin, _ = cmd2.StdoutPipe()
	cmd4.Stdin, _ = cmd3.StdoutPipe()

	// 设置标准输出和标准错误输出
	cmd4.Stdout = os.Stdout
	cmd4.Stderr = os.Stderr

	// 启动命令
	if err := cmd4.Start(); err != nil {
		fmt.Println("Error starting command:", err)
		return
	}

	// 等待命令结束
	if err := cmd4.Wait(); err != nil {
		fmt.Println("Command failed:", err)
		return
	}
}
```

