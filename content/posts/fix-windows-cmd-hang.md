---
title: "Windows下cmd运行go，出现假死现象，已解决"
subtitle: ""
date: 2021-12-14T13:50:03+08:00
lastmod: 2021-12-14T13:50:03+08:00
description: ""

tags: ["golang"]
categories: ["golang", "Troubleshooting"]
---

最近遇到一个很尴尬的现象，现象如下:

在windows上部署了一个go web应用，运行一段时间项目就假死一样，telnet端口是通的，但调用接口就一直卡在那。

## 问题根本

`cmd`默认开启了“快速编辑模式”，只要当鼠标点击`cmd`任何区域时，就自动进入了编辑模式，之后的程序向控制台输入内容甚至后台的程序都会被阻塞。

我们在控制台里面回车或者右键鼠标后，自动退出了编辑模式。因此，控制台又恢复输出内容，服务端又正常了。

## 解决方法

知道了什么原因出现的问题，解决起来就十分简单了

`windows cmd` -> 属性 -> 选项 -> 编辑选项
![](/img/fix-windows-cmd-hang.png)
