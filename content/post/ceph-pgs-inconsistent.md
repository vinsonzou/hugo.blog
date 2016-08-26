+++
date = "2016-08-17T23:44:59+08:00"
description = ""
tags = []
title = "ceph pgs inconsistent"
topics = ["Ceph"]

+++

OSD扩容后出现如下错误

`HEALTH_ERR 2 pgs inconsistent; 320 scrub errors`

```
[root@ceph01 ~]# ceph health detail
HEALTH_ERR 2 pgs inconsistent; 320 scrub errors
pg 10.9 is active+clean+inconsistent, acting [6,0,9]
pg 10.10 is active+clean+scrubbing+deep+inconsistent, acting [2,6,4]
320 scrub errors
```

修复处于不一致状态的pgs
```
[root@ceph01 ~]# ceph pg repair 10.9
instructing pg 10.9 on osd.6 to repair

[root@ceph01 ~]# ceph pg repair 10.10
instructing pg 10.10 on osd.2 to repair
```

经过一段时间的修复后，ceph恢复正常。
