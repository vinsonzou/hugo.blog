+++
date = "2018-10-16T16:27:12+08:00"
title = "Python数据库连接池实例--PooledDB"
description = ""
tags = ["Python","MySQL"]

+++

**不用连接池的MySQL连接方法**

```py
import MySQLdb
conn= MySQLdb.connect(host='127.0.0.1',user='root',passwd='password',db='DB_test',port=3306)
cur=conn.cursor()
SQL="select * from table_test"
cur.execute(SQL)
r=cur.fetchall()
cur.close()
conn.close()
```

**用连接池后的连接方法**
```py
import MySQLdb
from DBUtils.PooledDB import PooledDB
pool = PooledDB(MySQLdb,5,host='127.0.0.1',user='root',passwd='password',db='DB_test',port=3306)    #5为连接池里的最少连接数

conn = pool.connection()  #以后每次需要数据库连接就是用connection()函数获取连接就好了
cur=conn.cursor()
SQL="select * from table_test"
cur.execute(SQL)
r=cur.fetchall()
cur.close()
conn.close()
```

**PooledDB的参数**

- mincached: 最少的空闲连接数，如果空闲连接数小于这个数，pool会创建一个新的连接
- maxcached: 最大的空闲连接数，如果空闲连接数大于这个数，pool会关闭空闲连接
- maxconnections: 最大的连接数，
- blocking: 当连接数达到最大的连接数时，在请求连接的时候，如果这个值是True，请求连接的程序会一直等待，直到当前连接数小于最大连接数，如果这个值是False，会报错，
- maxshared: 当连接数达到这个数，新请求的连接会分享已经分配出去的连接

**连接池对性能的提升表现在**

- 在程序创建连接的时候，可以从一个空闲的连接中获取，不需要重新初始化连接，提升获取连接的速度
- 关闭连接的时候，把连接放回连接池，而不是真正的关闭，所以可以减少频繁地打开和关闭连接
- 避免mysql连接数耗尽

DBUtils下载地址：https://pypi.python.org/pypi/DBUtils/
