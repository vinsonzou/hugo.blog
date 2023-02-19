---
title: "[Go] slice删除元素的性能对比"
subtitle: ""
date: 2023-02-19T20:10:00+08:00
lastmod: 2023-02-19T20:10:00+08:00
description: ""

tags: ["Golang"]
categories: ["Golang"]
---

## 环境
- MacBook Pro (Retina, 13-inch, Mid 2014)
	- Intel(R) Core(TM) i5-4278U CPU @ 2.60GHz
- go 1.20

## 代码如下

```go
package bechmark

import (
    "testing"
)

var (
    // 原始slice
    origin = []string{"a", "b", "c", "d", "e", "f", "g", "h"}
    // 需要删除的元素
    targetEle = "e"
)

// 第一种
func BenchmarkMake(t *testing.B) {
    t.ResetTimer()

    for i := 0; i < t.N; i++ {
        target := make([]string, 0, len(origin))
        for _, item := range origin {
            if item != targetEle {
                target = append(target, item)
            }
        }
    }
}

// 第二种
func BenchmarkReuse(t *testing.B) {
    t.ResetTimer()

    for i := 0; i < t.N; i++ {
        target := origin[:0]
        for _, item := range origin {
            if item != targetEle {
                target = append(target, item)
            }
        }
    }
}

// 第三种
func BenchmarkEditOne(t *testing.B) {
    t.ResetTimer()

    for i := 0; i < t.N; i++ {
        for i := 0; i < len(origin); i++ {
            if origin[i] == targetEle {
                origin = append(origin[:i], origin[i+1:]...)
                i-- // 维护正确的索引
            }
        }
    }
}
```

### Benchmark结果

```
 ❯❯❯ go test -v -bench=. -benchtime=3s -benchmem
goos: darwin
goarch: amd64
pkg: benchmark
cpu: Intel(R) Core(TM) i5-4278U CPU @ 2.60GHz
BenchmarkMake
BenchmarkMake-4      	25386793	       132.6 ns/op	     128 B/op	       1 allocs/op
BenchmarkReuse
BenchmarkReuse-4     	60299337	        56.07 ns/op	       0 B/op	       0 allocs/op
BenchmarkEditOne
BenchmarkEditOne-4   	83621989	        42.50 ns/op	       0 B/op	       0 allocs/op
PASS
ok  	benchmark	11.050s
```

**解释**
- 除了第一种方法外，其他方法都对原数据进行了修改。
- 第一种方法适合不污染原slice数据的情况下使用，这种方式也比较简单，大部分学习golang的人也都能想到，不过性能稍差一些，还存在内存分配情况，不过也要看业务需要。
- 第二种方法比较巧妙，也是看到一个大神写的，创建了一个slice，但是共用原始slice的底层数组；这样就不需要额外分配内存空间，直接在原数据上进行修改。
- 第三种方法也会对底层数组进行修改，思路和前两种正好相反，如果找到需要移除的元素的时候，将其之后的元素前移，覆盖该元素的位置。
