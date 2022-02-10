---
title: "修复rpmdb损坏无法使用yum问题"
subtitle: ""
date: 2022-02-10T15:30:00+08:00
lastmod: 2022-02-10T15:30:00+08:00
description: ""

categories: ["Troubleshooting"]
---

**故障现象**

修复Linux Polkit本地权限提升漏洞（CVE-2021-4034）时，出现如下报错

```
error: rpmdb: BDB0113 Thread/process 6161/140008053192768 failed: BDB1507 Thread died in Berkeley DB library
error: db5 error(-30973) from dbenv->failchk: BDB0087 DB_RUNRECOVERY: Fatal error, run database recovery
error: cannot open Packages index using db5 -  (-30973)
error: cannot open Packages database in /var/lib/rpm
CRITICAL:yum.main:

Error: rpmdb open failed
```

**解决方法**

```sh
mkdir /var/lib/rpm/backup
cp -a /var/lib/rpm/__db* /var/lib/rpm/backup/
rm -f /var/lib/rpm/__db.[0-9][0-9]*
rpm --quiet -qa
rpm --rebuilddb
yum clean all
yum makecache
```
