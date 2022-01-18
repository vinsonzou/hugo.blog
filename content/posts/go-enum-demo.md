---
title: "[Go] 怎么正确实现枚举？答案藏着官方的源码里"
subtitle: ""
date: 2022-01-18T10:34:03+08:00
lastmod: 2022-01-18T10:34:03+08:00
description: ""

tags: ["golang"]
categories: ["golang"]
---

后来这两年主要在用Go做项目，我发现相似的问题 Go 里也存在，但是 Go 并没有提供枚举类型，那怎么做到进行状态值的正确限制呢？如果还是用 int 型的常量肯定不行。比如：

```go
const (
    Draft int = 1
    Published = 2
    Deleted   = 3
)

const (
    Summer int = 1
    Autumn     = 2
    Winter     = 3
    Spring     = 4
)

func main() {
    // 输出 true, 不会有任何编译错误
    fmt.Println(Autumn == Draft)
}
```

比如上面定义了两组 int 类型的常量，一类代表文章状态，一类代表季节的四季。这种方式拿文章状态与季节进行比较不会有任何编译上的错误。

答案在 Go 内置库或者一些咱们都知道的开源库的代码里就能找到。比如看看 google.golang.org/grpc/codes 里的gRPC 的错误码是怎么定义的，我们马上就能明白该怎么正确的实现枚举。

下面不多卖关子直接上答案了，不想去源码里看的，就看我这里写的也行，都是这么做的。

我们可以用 int 作为基础类型创建一个别名类型，Go 里边是支持这个的

```go
type Season int

const (
 Summer Season = 1
 Autumn        = 2
 Winter        = 3
 Spring        = 4
)
```

当然定义连续的常量值的时候 Go 里边经常使用 iota，所以上面的定义还能进一步简化。

```go
type Season int

const (
 Summer Season = iota + 1
 Autumn
 Winter
 Spring
)

type ArticleState int

const (
  Draft ArticleState = iota + 1
  Published
  Deleted
)

func checkArticleState(ArticleState state) {
 // ...
}

 func main() {
   // 两个操作数类型不匹配，编译错误
   fmt.Println(Autumn == Draft)
   // 参数类型不匹配，编译错误
   checkArticleState(100)
 }
```

虽然这些状态值的底层的类型都是 int 值，但是现在不论是进行两个不相干类型的枚举值比较，还是用整型值作为参数调用 checkArticleState 方法检查文章状态，都会造成编译错误，因为现在我们使用状态值的地方都有了类型限制。

这就是为什么针对错误码、状态机这种涉及有限数量状态值的场景下不能用整型常量而是要用枚举的原因。虽然 Go 语言里没有像 Java 一样单独提供一个 enum 表示枚举的类型，但是我们仍然能通过创建类型别名来实现枚举。

原文: https://mp.weixin.qq.com/s/CB1TBcxMf2XSvTqsslM_-A
