+++
date = "2015-06-03T23:04:55+08:00"
tags = ["igb"]
title = "kernel igb 00000500 0 eth0 reset adapter"

+++

系统环境

* CentOS 6.4
* igb driver version 4.0.1-k

报错信息如下

```sh
Jun  3 13:20:05 localhost kernel: igb 0000:05:00.1: eth1: Reset adapter
Jun  3 13:20:06 localhost kernel: igb 0000:05:00.0: eth0: Reset adapter
Jun  3 13:20:11 localhost kernel: igb: eth1 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: RX
Jun  3 13:20:12 localhost kernel: igb: eth0 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: RX
Jun  3 13:55:26 localhost kernel: igb 0000:05:00.1: eth1: Reset adapter
Jun  3 13:55:27 localhost kernel: igb 0000:05:00.0: eth0: Reset adapter
Jun  3 13:55:30 localhost kernel: igb: eth1 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: RX
Jun  3 13:55:30 localhost kernel: igb: eth0 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: RX
```

解决办法,参考[CentOS Bug Tracker](https://bugs.centos.org/view.php?id=7034)，升级igb driver至5.1.2以上，我这里直接升级至最新版本。

```sh
wget http://sourceforge.net/projects/e1000/files/igb%20stable/5.2.18/igb-5.2.18.tar.gz/download -O igb-5.2.18.tar.gz
tar zxf igb-5.2.18.tar.gz
cd igb-5.2.18/src/igb-5.1.2/src
yum install kernel-devel
make install
rmmod igb; modprobe igb
```
