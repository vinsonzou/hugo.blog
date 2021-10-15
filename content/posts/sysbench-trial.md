+++
description = ""
date = "2021-06-08T10:10:08+08:00"
title = "sysbench试用"
tags = ["MySQL", "sysbench"]

+++

## 目录

- [背景](#背景)
- [环境](#环境)
- [安装](#安装)
- [测试](#测试)

## 背景

主要测试开启审计插件OLTP性能损失多大，安装和测试方法如下

## 环境

- CentOS 7.4
- MySQL: 5.7.23
    - innodb_buffer_pool_size: 8G
    - innodb_buffer_pool_instances: 8【默认】
    - innodb_buffer_pool_chunk_size: 134217728【即128M，默认值】

## 安装

```sh
yum install gcc gcc-c++ autoconf automake make libtool bzr mysql-devel mysql

wget https://github.com/akopytov/sysbench/archive/refs/tags/1.0.20.tar.gz
tar zxf 1.0.20.tar.gz
cd sysbench-1.0.20
./autogen.sh
./configure --prefix=/usr/local/sysbench
make
make install
```

## 测试

- 创建测试库

```sh
mysql> create database sbtest;
```

- 测试只读性能，整个过程将持续10分钟

```sh
#准备数据
sysbench --db-driver=mysql --mysql-host=XXX --mysql-port=XXX --mysql-user=XXX --mysql-password=XXX --mysql-db=sbtest --table_size=25000 --tables=250 --events=0 --time=600 oltp_read_only prepare

#运行workload
sysbench --db-driver=mysql --mysql-host=XXX --mysql-port=XXX --mysql-user=XXX --mysql-password=XXX --mysql-db=sbtest --table_size=25000 --tables=250 --events=0 --time=600 --threads=XXX --percentile=95 --range_selects=0 --skip-trx=1 --report-interval=1 oltp_read_only run

#清理
sysbench --db-driver=mysql --mysql-host=XXX --mysql-port=XXX --mysql-user=XXX --mysql-password=XXX --mysql-db=sbtest --table_size=25000 --tables=250 --events=0 --time=600 --threads=XXX --percentile=95 --range_selects=0 oltp_read_only cleanup
```

- 测试写入性能，整个过程将持续10分钟

```sh
#准备数据
sysbench --db-driver=mysql --mysql-host=XXX --mysql-port=XXX --mysql-user=XXX --mysql-password=XXX --mysql-db=sbtest --table_size=25000 --tables=250 --events=0 --time=600 oltp_write_only prepare

#运行workload
sysbench --db-driver=mysql --mysql-host=XXX --mysql-port=XXX --mysql-user=XXX --mysql-password=XXX --mysql-db=sbtest --mysql-ignore-errors=1062 --table_size=25000 --tables=250 --events=0 --time=600 --threads=XXX --percentile=95 --report-interval=1 oltp_write_only run

#清理
sysbench --db-driver=mysql --mysql-host=XXX --mysql-port=XXX --mysql-user=XXX --mysql-password=XXX --mysql-db=sbtest --table_size=25000 --tables=250 --events=0 --time=600 --threads=XXX --percentile=95 oltp_write_only cleanup
```

- 测试混合读写性能，整个过程将持续10分钟

```sh
#准备数据
sysbench --db-driver=mysql --mysql-host=XXX --mysql-port=XXX --mysql-user=XXX --mysql-password=XXX --mysql-db=sbtest --table_size=25000 --tables=250 --events=0 --time=600 oltp_read_write prepare

#运行workload
sysbench --db-driver=mysql --mysql-host=XXX --mysql-port=XXX --mysql-user=XXX --mysql-password=XXX --mysql-db=sbtest --mysql-ignore-errors=1062 --table_size=25000 --tables=250 --events=0 --time=600 --threads=XXX --percentile=95 --report-interval=1 oltp_read_write run

#清理
sysbench --db-driver=mysql --mysql-host=XXX --mysql-port=XXX --mysql-user=XXX --mysql-password=XXX --mysql-db=sbtest --table_size=25000 --tables=250 --events=0 --time=600 --threads=XXX --percentile=95 oltp_read_write cleanup
```

## FAQ

**Q: 128线程只读测试时，报错如下:**

```sh
FATAL: mysql_stmt_prepare() failed
(last message repeated 1 times)
FATAL: MySQL error: 1461 "Can't create more than max_prepared_stmt_count statements (current value: 16382)"
```

> 解决方法: 

> mysql> set global max_prepared_stmt_count=1048576;

**Q: sysbench FATAL: mysql_stmt_execute() returned error 1062**

> 解决方法: 忽略1062错误

> 写入测试添加此参数 --mysql-ignore-errors=1062

## 参考
- https://help.aliyun.com/document_detail/146103.html
- https://www.unixso.com/MySQL/max_prepared_stmt_count.html
- https://blog.csdn.net/vkingnew/article/details/81025882
