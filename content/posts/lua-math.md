---
title: "Lua小数四舍五入"
subtitle: ""
date: 2022-03-25T10:40:03+08:00
lastmod: 2022-03-25T10:40:03+08:00
description: ""

tags: ["OpenResty"]
categories: ["OpenResty"]
---

- 保留整数

```lua
math.floor(x + 0.5)  // 4.4 --> 4
math.floor(x)        // 会直接舍弃小数点后的部分
```

- 保留几位小数进行四舍五入

    公式为: `math.floor(x * num + 0.5) / num`

    保留2位 num为100，3位为1000，…

    比如0.03688要变成 0.037 (保留3位小数)
