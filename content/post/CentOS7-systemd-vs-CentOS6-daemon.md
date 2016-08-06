+++
date = "2016-08-06T13:54:53+08:00"
description = ""
tags = ["systemd"]
title = "Centos 7.x systemd对比Centos 6.x daemon"
topics = ["CentOS7"]

+++

从CentOS 7.x开始，CentOS开始使用systemd服务来代替daemon，原来管理系统启动和管理系统服务的相关命令全部由systemctl命令来代替。

1、原来的 service 命令与 systemctl 命令对比
==========================================

| daemon命令 | systemctl命令 | 说明 |
| :-: | :-: | :-: |
| service [服务] start | systemctl start [unit type] | 启动服务 |
| service [服务] stop | systemctl stop [unit type] | 停止服务 |
| service [服务] restart | systemctl restart [unit type] | 重启服务 |

此外还有二个systemctl参数没有与service命令参数对应

```
status: 参数来查看服务运行情况
reload: 重新加载服务，加载更新后的配置文件（并不是所有服务都支持这个参数，比如network.service）
```

应用举例:
```
#启动网络服务
systemctl start network.service
#停止网络服务
systemctl stop network.service
#重启网络服务
systemctl restart network.service
#查看网络服务状态
systemctl status network.serivce
```

2、原来的chkconfig 命令与 systemctl 命令对比
========================================

2.1、设置开机启动/不启动
---------------------

| daemon命令|systemctl命令|说明|
| :-: | :-: | :-: |
| chkconfig [服务] on | systemctl enable [unit type] | 设置服务开机启动 |
| chkconfig [服务] off | systemctl disable [unit type] | 设备服务禁止开机启动 |

应用举例：
```
#停止cup电源管理服务
systemctl stop cups.service
#禁止cups服务开机启动
systemctl disable cups.service
#查看cups服务状态
systemctl status cups.service
#重新设置cups服务开机启动
systemctl enable cups.service
```

2.2、查看系统上所有的服务
-----------------------

命令格式：
```
systemctl [command] [--type=TYPE] [--all]
```
参数详解：

command
```
list-units: 依据unit列出所有启动的unit。加上 --all 才会列出没启动的unit;
list-unit-files: 依据 /usr/lib/systemd/system/ 内的启动文件，列出启动文件列表
```
`--type=TYPE`
```
为unit type, 主要有service, socket, target, timer
```

应用举例：
```
systemctl                  #列出所有的系统服务
systemctl list-units       #列出所有启动unit
systemctl list-unit-files  #列出所有启动文件
systemctl list-units --type=service --all  #列出所有service类型的unit
systemctl list-units --type=socket --all   #列出所有socket类型的unit
systemctl list-units --type=timer --all    #列出所有timer类型的unit
systemctl list-units --type=target --all   #列出所有target
```

3、systemctl特殊的用法
====================
```
#查看网络服务是否启动
systemctl is-active network.service
#检查网络服务是否设置为开机启动
systemctl is-enable network.service
#停止cups服务
systemctl stop cups.service
#注销cups服务
systemctl mask cups.service
#查看cups服务状态
systemctl status cups.service
#取消注销cups服务
systemctl unmask cups.service
```

4、init 命令与systemctl命令对比
============================

| init命令 | systemctl 命令 | 说明 |
| :-: | :-: | :-: |
| init 0 | systemctl poweroff | 系统关机 |
| init 6 | systemctl reboot | 重新启动 |

与开关机相关的其他命令：

| systemctl命令 | 说明 |
| :-: | :-: |
| systemctl suspend |	进入睡眠模式 |
| systemctl hibernate | 进入休眠模式 |
| systemctl rescue | 强制进入救援模式 |
| systemctl emergency | 强制进入紧急救援模式 |

5、设置系统运行级别
================

5.1、运行级别对应表
----------------

| init级别 | systemctl target |
| :-: | :-: |
| 0 | shutdown.target |
| 1	| emergency.target |
| 2	| rescure.target |
| 3	| multi-user.target |
| 4 | 无 |
| 5	| graphical.target |
| 6	| 无 |
此外还有一个getty.target用来设置tty的数量。

5.2、设置运行级别
--------------
```
systemctl get-default  #获得当前的运行级别
systemctl set-default multi-user.target  #设置默认的运行级别为mulit-user
systemctl isolate multi-user.target      #在不重启的情况下，切换到运行级别mulit-user下
systemctl isolate graphical.target       #在不重启的情况下，切换到图形界面下
```

6、关闭网络服务
============

在使用systemctl关闭网络服务时有一些特殊，需要同时关闭unit.servce和unit.socket

使用systemctl查看开启的sshd服务
```
[root@localhost system]# systemctl list-units --all | grep sshd
sshd-keygen.service loaded inactive dead        OpenSSH Server Key Generation
sshd.service        loaded active   running     OpenSSH server daemon
sshd.socket         loaded inactive dead        OpenSSH Server Socket
```
可以看到系统同时开启了sshd.service和sshd.socket , 如果只闭关了sshd.service那么sshd.socket还在监听网络，在网络上有要求连接sshd时就会启动sshd.service。因此如果要完全关闭sshd服务，需要同时停用sshd.service和sshd.socket。
```
systemctl stop sshd.service
systemctl stop sshd.socket
systemctl disable sshd.service sshd.socket
```
