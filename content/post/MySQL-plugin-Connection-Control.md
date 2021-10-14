+++
description = ""
date = "2021-06-08T13:10:08+08:00"
title = "MySQL插件之Connection-Control"
tags = ["MySQL", "等保"]

+++

## 目录

- [等保需求](#等保需求)
- [插件介绍](#插件介绍)
- [插件安装](#插件安装)
- [插件配置](#插件配置)


## 等保需求

- 登录失败策略：登录失败5次锁定10分钟，使用 Connection-Control 插件实现。

## 插件介绍

MySQL 5.7.17 以后提供了 Connection-Control 插件用来控制客户端在登录操作连续失败一定次数后的响应的延迟。该插件可有效的防止客户端暴力登录的风险(攻击)。该插件包含以下2个组件:

- CONNECTION_CONTROL：用来控制登录失败的次数及延迟响应时间
- CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS：该表将登录失败的操作记录至IS库中

## 插件安装

```sh
-- 配置文件增加以下配置
[mysqld]
plugin-load-add									= connection_control.so
connection-control                             	= FORCE_PLUS_PERMANENT    // 防止运行时卸载
connection-control-failed-login-attempts       	= FORCE_PLUS_PERMANENT    // 防止运行时卸载 
connection_control_min_connection_delay			= 600000
connection_control_max_connection_delay			= 2147483647
connection_control_failed_connections_threshold	= 5

-- 插件动态安装启用
mysql> INSTALL PLUGIN CONNECTION_CONTROL SONAME 'connection_control.so';
mysql> INSTALL PLUGIN CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS SONAME 'connection_control.so';

-- 验证是否正常安装
mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS
FROM INFORMATION_SCHEMA.PLUGINS
WHERE PLUGIN_NAME LIKE 'connection%';

mysql> SHOW PLUGINS;
```

## 插件配置

```sh
-- 查看默认相关变量
mysql> show variables like 'connection_control%';
+-------------------------------------------------+------------+
| Variable_name                                   | Value      |
+-------------------------------------------------+------------+
| connection_control_failed_connections_threshold | 3          |
| connection_control_max_connection_delay         | 2147483647 |
| connection_control_min_connection_delay         | 1000       |
+-------------------------------------------------+------------+

-- 定制化配置
mysql> SET GLOBAL connection_control_failed_connections_threshold = 5;
mysql> SET GLOBAL connection_control_min_connection_delay = 600000;

-- 查看修改后的配置
mysql> show variables like 'connection_control%';
+-------------------------------------------------+------------+
| Variable_name                                   | Value      |
+-------------------------------------------------+------------+
| connection_control_failed_connections_threshold | 5          |
| connection_control_max_connection_delay         | 2147483647 |
| connection_control_min_connection_delay         | 600000     |
+-------------------------------------------------+------------+
```

- connection_control_failed_connections_threshold
    - 失败尝试的次数，默认为3，表示当连接失败3次后启用连接控制，0表示不开启
- connection_control_max_connection_delay
    - 响应延迟的最大时间，默认约25天
- connection_control_min_connection_delay
    - 响应延迟的最小时间，默认1秒(1000ms)

```sh
-- 该表记录登录失败的用户及失败次数，当用户登录成功后，登录失败的记录则会被删除。
-- 重新配置connection_control_failed_connections_threshold变量，该表记录会被删除(重置)
-- 如果使用不存在的用户登录，则该表记录用户名为空，但会记录具体登录的IP

mysql> use information_schema;
mysql> select * from connection_control_failed_login_attempts;

-- 连接控制的使用次数(可用户判断是否存在暴力登录尝试)
-- 重新配置connection_control_failed_connections_threshold变量，该表记录会被删除(重置)
mysql> show global status like 'Connection_control_delay_generated';
+------------------------------------+-------+
| Variable_name                      | Value |
+------------------------------------+-------+
| Connection_control_delay_generated | 5     |
+------------------------------------+-------+
```
