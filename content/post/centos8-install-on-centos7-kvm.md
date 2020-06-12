+++
date = "2020-06-12T13:12:06+08:00"
title = "CentOS7上如何支持安装CentOS8 VM"
description = ""
topics = ["libvirt"]
tags = ["libvirt","kvm"]

+++

## 环境

- 宿主机：CentOS 7.5
- VM: CentOS 8.1

## 查看支持os

osinfo-query os

当前环境默认不支持CentOS8/RHEL8，需做如下操作:

```
# osinfo-db 需更新至osinfo-db-20190805-2
yum install osinfo-db-tools osinfo-db
```

## 报错处理

**ERROR 'virConnect' object has no attribute 'baselineHypervisorCPU'**

将 `libvirt-python-3.9.0-1` 升级至 `libvirt-python-4.5.0-1` 即可解决

参考链接: https://github.com/virt-manager/virt-manager/issues/57
