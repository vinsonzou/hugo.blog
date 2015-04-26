+++
date = "2015-04-26T16:59:30+08:00"
menu = "main"
tags = ["java"]
title = "俄罗斯永久冬令时的坑"

+++

### 系统环境

OS: CentOS 6.X

JDK: 6

### 背景

因游戏在俄罗斯运营，采用了莫斯科时间，俄罗斯宣布2014年10月26日开始永久冬令时。

### 过程

更新了系统时区文件，保证系统时间不会切换至夏令时，执行`yum update tzdata`即可。

但在莫斯科时间的2015年3月29日凌晨2点，java游戏进程的日志时间跳到了凌晨3点(比系统时间快了1个小时)

### 解决方法

更新JDK的时区文件即可

	ln -sf /usr/share/zoneinfo/Europe/Moscow /usr/local/jdk/jre/lib/zi/Europe/Moscow