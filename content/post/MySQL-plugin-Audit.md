+++
description = ""
date = "2021-06-08T14:10:08+08:00"
title = "MySQL插件之审计"
tags = ["MySQL", "等保"]
topics = ["MySQL"]

+++

## 目录

- [等保需求](#等保需求)
- [背景](#背景)
- [插件安装](#插件安装)
- [插件配置](#插件配置)


## 等保需求

- 审计功能：记录时间、来源IP、用户、行为、数据库、SQL。

## 背景

Oracle 的 MySQL 社区版不带审计插件（Audit Plugin），要想使用审计功能，你可以用企业版，不过这需要银子。业界还有一些 GPL 协议的审计插件，这里我们选择 MariaDB 的审计插件。

- 系统版本: CentOS 7.4
- MySQL版本: 5.7.23 社区版
- MariaDB审计插件版本: 1.4.7

备注:
- MariaDB审计插件一直在更新，不同版本的审计插件功能也不同，每个版本的功能见：https://mariadb.com/kb/en/mariadb-audit-plugin-options-and-system-variables/#server_audit_events
- 我们在给MySQL数据库安装审计插件时，需要从MariaDB里面拷贝审计插件。MariaDB版本与审计插件版本关系如下：https://mariadb.com/kb/en/mariadb-audit-plugin-versions/

## 插件安装

MariaDB 的 10.1 版本对应与 Oracle 的 MySQL 5.7，我们到它的官网上下载 Linux 的通用版本。

找到我们需要的审计插件：

```sh
./mariadb-10.1.48-linux-x86_64/lib/plugin/server_audit.so
```

把这个 so 结尾的文件拷贝到 MySQL 的插件目录，例如：/usr/local/mysql/lib/plugin/ 

```sh
-- 配置文件增加以下配置
[mysqld]
plugin_load_add                = server_audit
server_audit                   = FORCE_PLUS_PERMANENT  # 防止审计插件被卸载
server_audit_logging           = on
server_audit_events            = CONNECT,TABLE,QUERY_DDL,QUERY_DCL,QUERY_DML_NO_SELECT
server_audit_file_rotate_size  = 268435456
server_audit_file_rotations    = 64

-- 插件动态安装启用
mysql> install plugin server_audit SONAME 'server_audit.so';

-- 验证是否正常安装
mysql> SELECT PLUGIN_NAME, PLUGIN_LIBRARY, PLUGIN_STATUS, LOAD_OPTION 
FROM INFORMATION_SCHEMA.PLUGINS 
WHERE PLUGIN_LIBRARY = 'server_audit.so';

mysql> SHOW PLUGINS;
```

## 插件配置

```sh
-- 查看默认相关变量
mysql> show variables like '%server_audit%';
+-------------------------------+-----------------------+
| Variable_name                 | Value                 |
+-------------------------------+-----------------------+
| server_audit_events           |                       |
| server_audit_excl_users       |                       |
| server_audit_file_path        | server_audit.log      |
| server_audit_file_rotate_now  | OFF                   |
| server_audit_file_rotate_size | 1000000               |
| server_audit_file_rotations   | 9                     |
| server_audit_incl_users       |                       |
| server_audit_loc_info         |                       |
| server_audit_logging          | OFF                   |
| server_audit_mode             | 1                     |
| server_audit_output_type      | file                  |
| server_audit_query_log_limit  | 1024                  |
| server_audit_syslog_facility  | LOG_USER              |
| server_audit_syslog_ident     | mysql-server_auditing |
| server_audit_syslog_info      |                       |
| server_audit_syslog_priority  | LOG_INFO              |
+-------------------------------+-----------------------+

-- 定制化配置
mysql> set global server_audit_events='CONNECT,TABLE,QUERY_DDL,QUERY_DCL,QUERY_DML_NO_SELECT';
mysql> set global server_audit_file_rotations=64;
mysql> set global server_audit_file_rotate_size=268435456;
mysql> set global server_audit_logging=ON;

-- 查看修改后的配置
mysql> show variables like '%server_audit%';
+-------------------------------+-------------------------------------------------------+
| Variable_name                 | Value                                                 |
+-------------------------------+-------------------------------------------------------+
| server_audit_events           | CONNECT,TABLE,QUERY_DDL,QUERY_DCL,QUERY_DML_NO_SELECT |
| server_audit_excl_users       |                                                       |
| server_audit_file_path        | server_audit.log                                      |
| server_audit_file_rotate_now  | OFF                                                   |
| server_audit_file_rotate_size | 67108864                                              |
| server_audit_file_rotations   | 128                                                   |
| server_audit_incl_users       |                                                       |
| server_audit_loc_info         |                                                       |
| server_audit_logging          | ON                                                    |
| server_audit_mode             | 1                                                     |
| server_audit_output_type      | file                                                  |
| server_audit_query_log_limit  | 1024                                                  |
| server_audit_syslog_facility  | LOG_USER                                              |
| server_audit_syslog_ident     | mysql-server_auditing                                 |
| server_audit_syslog_info      |                                                       |
| server_audit_syslog_priority  | LOG_INFO                                              |
+-------------------------------+-------------------------------------------------------+
```

审计参数说明:
- server_audit_logging 这参数默认为 OFF，把这个参数设置为 ON 才能启动审计功能。
- server_audit_events 决定记录的事件，这里我们记录 CONNECT,TABLE,QUERY_DDL,QUERY_DCL,QUERY_DML_NO_SELECT，默认为空代表审计所有事件。
- server_audit_file_rotate_size 日志文件大小，单位为byte。
- server_audit_file_rotations 日志文件的数量，到这个阀值时会覆盖第一个审计记录文件，默认为 9。
- server_audit_output_type 设置为 file 时，记录成文件，默认目录是 MySQL 的 datadir 目录，默认文件名是 server_audit.log。当设置为 syslog 时，审计记录会通过标准 <syslog.h> API 发送给本地的 syslogd daemon。

审计日志格式如下：

```sh
[timestamp],[serverhost],[username],[host],[connectionid],[queryid],[operation],[database],[object],[retcode]
```

审计日志示例如下:

```sh
20210608 09:47:13,localhost,dev,127.0.0.1,429930,0,CONNECT,,,0
20210608 09:47:13,localhost,dev,127.0.0.1,429930,0,DISCONNECT,,,0
```
