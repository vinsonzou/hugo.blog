+++
title = "Python版本号比较"
description = ""
topics = []
tags = ["Python"]
date = "2017-07-23T16:13:10+08:00"

+++

## 第一种比较方法(StrictVersion)

StrictVersion是由.将一串带有预发布标签的数字分隔为两个或三个部分的格式，预发布标签的字母只能是a或者b加数字版本号，而且只能在最末尾。预发布a版本低于b版本，并且预发布版本永远小于正式发布版本。

**合法格式:**

```py
0.4       0.4.0  (相同版本)
0.4.1
0.5a1     (预发布版本a1，小于0.5，即0.5版本更新)
0.5b3
0.5
0.9.6
1.0
1.0.4a3
1.0.4b1
1.0.4
```

**非法格式:**

```py
1           没有.分隔，需要分隔为2-3部分
2.7.2.2     被分隔成了4个部分
1.3.a4      预发布版本号应该在数字后面
1.3pl1      预发布版本号字母标签只能是a或者b
1.3B1       预发布版本号字母标签只能是a或者b
1.3c        预发布版本号字母标签后必须加数字版本号
```

**版本比较**

```py
In [1]: from distutils.version import StrictVersion

In [2]: StrictVersion('1.2a3') < StrictVersion('1.2b1')
Out[2]: True

In [3]: StrictVersion('1.2b1') < StrictVersion('1.2')
Out[3]: True

In [4]: StrictVersion('1.2') < StrictVersion('1.2.1')
Out[4]: True

In [5]: StrictVersion('1.2') == StrictVersion('1.2.0')
Out[5]: True

In [6]: StrictVersion('1.2.11') < StrictVersion('1.11')
Out[6]: True
```

## 第二种比较方法(LooseVersion)

LooseVersion格式要求和StrictVersion不同，或者说它并没有任何规定的格式。由一系列数字,相隔时间或字母的字符串组成，并没有一个严格的格式。在进行比较的时候按照数字大小，字符串按字典顺序比较。

**合法格式**

```py
1.5.1
1.5.2b2
161
3.10a
8.02
3.4j
1996.07.12
3.2.pl0
3.1.1.6
2g6
11g
0.960923
2.2beta29
1.13++
5.5.kw
2.0b1pl0
```

**非法格式**

    并没有哟

**格式比较**

```py
In [1]: from distutils.version import StrictVersion

In [2]: LooseVersion('1.6.0x') < LooseVersion('1.20.0x')
Out[2]: True

In [3]: LooseVersion('2.6.0x') < LooseVersion('1.20.0x')
Out[3]: False

In [4]: LooseVersion('1.20.0x') < LooseVersion('1.20.0z')
Out[4]: True

In [5]: LooseVersion('1') < LooseVersion('a')
Out[5]: True 
```
