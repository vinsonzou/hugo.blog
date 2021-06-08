+++
date = "2020-06-18T13:41:15+08:00"
title = "CentOS8编译安装MySQL 5.7.30"
description = ""
topics = ["MySQL"]
tags = ["MySQL","centos8"]

+++

## 环境

- OS: CentOS 8.2
- MySQL: 5.7.30

## 报错信息

### 1. 编译报错
```c
CMake Error at plugin/group_replication/libmysqlgcs/rpcgen.cmake:100 (MESSAGE):
  Could not find rpcgen
Call Stack (most recent call first):
  plugin/group_replication/libmysqlgcs/CMakeLists.txt:38 (INCLUDE)
```

**解决方法**

```sh
yum --enablerepo=PowerTools install rpcgen
```

### 2. systemctl start mysqld.service报错无法启动问题

**报错信息如下**

```sh
[ERROR] Can't start server: can't check PID filepath: No such file or directory
```

/usr/lib/systemd/system/mysqld.service 默认配置如下

```sh
# Copyright (c) 2015, 2016, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License, version 2.0,
# as published by the Free Software Foundation.
#
# This program is also distributed with certain software (including
# but not limited to OpenSSL) that is licensed under separate terms,
# as designated in a particular file or component or in included license
# documentation.  The authors of MySQL hereby grant you an additional
# permission to link the program and your derivative works with the
# separately licensed software that they have included with MySQL.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License, version 2.0, for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA
#
# systemd service file for MySQL forking server
#

[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target

[Install]
WantedBy=multi-user.target

[Service]
User=mysql
Group=mysql

Type=forking

PIDFile=/var/run/mysqld/mysqld.pid

# Disable service start and stop timeout logic of systemd for mysqld service.
TimeoutSec=0

# Execute pre and post scripts as root
PermissionsStartOnly=true

# Needed to create system tables
ExecStartPre=/usr/local/mysql/bin/mysqld_pre_systemd

# Start main service
ExecStart=/usr/local/mysql/bin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS

# Use this to switch malloc implementation
EnvironmentFile=-/etc/sysconfig/mysql

# Sets open_files_limit
LimitNOFILE = 5000

Restart=on-failure

RestartPreventExitStatus=1

PrivateTmp=false
```

**解决方法**

- 默认mysqld.service中的pid存在/var/run/mysqld，此目录必须存在且mysql用户有权限

```sh
echo -e 'd /run/mysqld 0755 mysql mysql -' > /usr/lib/tmpfiles.d/mysqld
```

### 3. 启动警告信息

```sh
[Warning] Could not increase number of max_open_files to more than 5000 (request: 10000)
```

**解决方法**

```sh
# 调整systemd mysqld.service配置(即/usr/lib/systemd/system/mysqld.service)
LimitNOFILE = 5000 调整为 LimitNOFILE = 10000
systemctl daemon-reload
systemctl restart mysqld
```

