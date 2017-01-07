+++
title = "不重启解决Too Many Connections"
description = ""
topics = ["MySQL"]
tags = ["MySQL"]
date = "2017-01-07T19:57:32+08:00"

+++

当发生Too many connections时，即使是DBA也无法登录到数据库，一般的做法是修改配置文件的max_connections参数，然后重启数据库，这样业务就有几秒钟的中断，对于线上不能中断的数据库就只能采用另外一种极客的方法了，用gdb直接修改mysqld内存中max_connections的值，具体做法如下：

```
gdb -p $(cat /data/mysql/mysql-server.pid) -ex "set max_connections=3000" -batch
```

**改进方法如下**

通常有两个参数控制控制最大连接数：
    
    max_connections：该实例允许最大的连接数
    max_user_connections：该实例允许每个用户的最大连接数

每个人要根据自己业务量，设置合适的值，不要盲目设置过大，但也不可设置过小，因为MySQL在连接数上升的情况下性能下降非常厉害，如果需要大量连接，这时可以引入thread_pool，所以我们需要保持一个原则：系统创建的用户（给应用使用用户）数 * max_user_connections < max_connections。
