+++
date = "2016-08-03T10:52:24+08:00"
description = ""
tags = ["MySQL"]
title = "MySQL Unable to lock ibdata1 error 11 fix"
topics = ["MySQL"]

+++

A bad shutdown can cause such erros on MySQL.

```
InnoDB: Unable to lock ./ibdata1, error: 11
InnoDB: Check that you do not already have another mysqld process
InnoDB: using the same InnoDB data or log files.
InnoDB: Error in opening ./ibdata1
```

For solution
------------
```
mv ibdata1 ibdata1.bak
cp -a ibdata1.bak ibdata1
service mysqld restart
```
