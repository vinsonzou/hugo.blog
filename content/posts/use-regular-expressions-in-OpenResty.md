+++
description = ""
date = "2016-10-30T10:48:19+08:00"
title = "在 OpenResty 中使用正则表达式"
tags = ["转载"]
categories = ["OpenResty"]

+++

在 OpenResty 中使用正则表达式，社区中推荐的做法是使用ngx.re api。比如匹配一个字符串是否为 http(s) 的链接，可以这么写：

```lua
local function is_http_url(s)
    return ngx.re.find(s, [[^https?://[\w-_?.:/+=&#%]+$]])
end
```

压测一下:

```lua
local t = os.clock()
for _ = 1, max do
    is_http_url("http://blog.stackoverflow.com/2016/10/Stack-Overflow-92-Podcast-The-Guerilla-Guide-to-Interviewing/?cb=1")
end
print("Time cost: ", os.clock() - t, " s")
```

结果：Time cost: 2.663408 s

另一种做法是使用 lua 的正则语法：

```lua
local function is_http_url(s)
    return s:find("^https?://[%w-_%.%?:/%+=&#%%]+$")
end
```

结果：Time cost: 0.652221 s

呃，怎么前者耗时是后者的四倍？lua 内置的小小状态机实现，居然打败了大名鼎鼎的 PCRE 库！说好的社区推荐呢！

仔细一瞧，前者的确漏了点东西。ngx.re默认不会缓存正则表达式编译后的结果。一般在其它编程平台上，我们都会先把字符串编译成正则表达式，再用到正则函数中。比如在
Python 里使用 re.compile。所以赶紧补上：

```lua
return ngx.re.find(s, [[^https?://[\w-_?.:/+=&#%]+$]], "o")
```

好，这次性能有了明显提升：Time cost: 0.646518 s

不错不错，虽然还是跟 lua 的实现不分上下，考虑到 lua 本身的正则支持非常弱（比如连 (foo|bar)
这种形式都不行），而且语法离经叛道，改用 ngx.re 还是挺有必要的。毕竟 PCRE 可是 Perl Compatibility Regex
Expression库，我最喜欢它支持的(?name:pattern)形式的命名捕获功能。

其实 ngx.re 实现尚未用尽全力呢。开启了 JIT 之后，PCRE 库的性能会更上一层楼：

```lua
return ngx.re.find(s, [[^https?://[\w-_?.:/+=&#%]+$]], "jo")
```

结果：Time cost: 0.421824 s

此时，lua 正则已经被甩到后面去了。

还能更快吗？

当然，OpenResty 军火库里还有另外一个武器：[lua-resty-core](https://github.com/openresty/lua-resty-core)

```lua
require 'resty.core.regex'

local function is_http_url(s)
    return ngx.re.find(s, [[^https?://[\w-_?.:/+=&#%]+$]], "jo")
end
```

结果：Time cost: 0.175346 s

Boom！最终用时是 lua 正则的四分之一。lua 正则已经望尘莫及了。有趣的是，这个结果正好是第一次比较的结果倒过来。
实话说，这个结果在我的意料之外。resty.core.regex 版本的 ngx.re api，跟默认版本的区别在于对入参和出参的处理是在 lua
代码里完成的，另外调用 C 函数部分采用的是 ffi 而非传统的 C binding。但为什么会这么快？luajit 是否对 ffi 有 jit 优化？

需要注明一下，resty.core.regex并非银弹。在我们自己的应用上，我尝试引入resty.core.regex，发现对性能无可见的提升。当然，我们的应用的功能不是匹配
url，正则处理亦非瓶颈。不过 resty.core.regex 对自己的项目是否有效，还需要诸君自己测试一番。

总结

* 如果在 OpenResty 项目中需要使用正则表达式，请使用 ngx.re api，并开启 jo 选项。
* resty.core.regex 值得一试。

原文：https://segmentfault.com/a/1190000007298100
