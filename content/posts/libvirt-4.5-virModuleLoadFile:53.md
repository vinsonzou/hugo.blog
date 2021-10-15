+++
description = ""
date = "2019-01-15T17:27:22+08:00"
title = "libvirt 4.5 virModuleLoadFile:53"
tags = []

+++

CentOS 7.5.1804的libvirt从3.9升级至4.5时，无法启动，报错如下：

> error : virModuleLoadFile:53 : internal error: Failed to load module '/usr/lib64/libvirt/storage-backend/libvirt_storage_backend_rbd.so': /usr/lib64/libvirt/storage-backend/libvirt_storage_backend_rbd.so: undefined symbol: rbd_diff_iterate2

**详细报错如下:**

```sh
[root@localhost ~]# libvirtd -v
2019-01-15 08:56:53.433+0000: 34181: info : libvirt version: 4.5.0, package: 10.el7_6.3 (CentOS BuildSystem <http://bugs.centos.org>, 2018-11-28-20:51:39, x86-01.bsys.centos.org)
2019-01-15 08:56:53.433+0000: 34181: info : hostname: localhost.localdomain
2019-01-15 08:56:53.433+0000: 34181: info : virObjectNew:248 : OBJECT_NEW: obj=0x56166f5da690 classname=virAccessManager
2019-01-15 08:56:53.434+0000: 34181: info : virObjectNew:248 : OBJECT_NEW: obj=0x56166f5cbfe0 classname=virAccessManager
2019-01-15 08:56:53.434+0000: 34181: info : virObjectRef:382 : OBJECT_REF: obj=0x56166f5da690
2019-01-15 08:56:53.434+0000: 34181: info : virObjectUnref:344 : OBJECT_UNREF: obj=0x56166f5da690
2019-01-15 08:56:53.434+0000: 34181: info : virObjectNew:248 : OBJECT_NEW: obj=0x56166f5cc470 classname=virNetDaemon
2019-01-15 08:56:53.434+0000: 34181: info : virEventPollAddHandle:140 : EVENT_POLL_ADD_HANDLE: watch=1 fd=5 events=1 cb=0x7f5e248272d0 opaque=(nil) ff=(nil)
2019-01-15 08:56:53.434+0000: 34181: info : virObjectNew:248 : OBJECT_NEW: obj=0x56166f5cc7d0 classname=virNetServer
2019-01-15 08:56:53.434+0000: 34181: info : virObjectRef:382 : OBJECT_REF: obj=0x56166f5cc7d0
2019-01-15 08:56:53.462+0000: 34181: error : virModuleLoadFile:53 : internal error: Failed to load module '/usr/lib64/libvirt/storage-backend/libvirt_storage_backend_rbd.so': /usr/lib64/libvirt/storage-backend/libvirt_storage_backend_rbd.so: undefined symbol: rbd_diff_iterate2
```

**解决方法:**

`libvirt_storage_backend_rbd.so`应该是连接ceph块设备（rbd）的一个模块，这里暂时不用连接ceph，最简单粗暴的做法就是移除该模块，重启libvirtd。

```sh
[root@localhost ~]# cd /usr/lib64/libvirt/storage-backend
[root@localhost storage-backend]# mv libvirt_storage_backend_rbd.so libvirt_storage_backend_rbd.so.bak
[root@localhost ~]# systemctl restart libvirtd
```
