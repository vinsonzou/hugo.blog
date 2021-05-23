+++
date = "2019-01-23T17:46:06+08:00"
title = "Python的while 1跟while True到底有什么区别?"
description = ""
topics = ["Python"]
tags = ["Python"]

+++

定义两个方法,分别使用while循环

```py
def t1():
    while 1:
        pass

def t2():
    while True:
        pass
```

单从功能上说,两种无任何区别,那么,来看看字节码上的区别:

**For Python 2.x**

```py
import dis  #载入反编译模块,Python内置的

dis.dis(t1) #对应的是while 1,下面是输出
  2           0 SETUP_LOOP               3 (to 6)

  3     >>    3 JUMP_ABSOLUTE            3
        >>    6 LOAD_CONST               0 (None)
              9 RETURN_VALUE

dis.dis(t2) #对应的是while True,下面是输出
  2           0 SETUP_LOOP              10 (to 13)
        >>    3 LOAD_GLOBAL              0 (True)
              6 POP_JUMP_IF_FALSE       12

  3           9 JUMP_ABSOLUTE            3
        >>   12 POP_BLOCK
        >>   13 LOAD_CONST               0 (None)
             16 RETURN_VALUE
```

很明显, while 1的字节码只有while True的一半. 为什么呢? 因为Python2.x中True不是关键字,只是一个全局变量而已


**For Python 3.x**

```py
>>> dis.dis(t1)
  2           0 SETUP_LOOP               4 (to 6)

  3     >>    2 JUMP_ABSOLUTE            2
              4 POP_BLOCK
        >>    6 LOAD_CONST               0 (None)
              8 RETURN_VALUE
>>> dis.dis(t2)
  2           0 SETUP_LOOP               4 (to 6)

  3     >>    2 JUMP_ABSOLUTE            2
              4 POP_BLOCK
        >>    6 LOAD_CONST               0 (None)
              8 RETURN_VALUE
```

在python 3.x中，while 1和while True无任何区别。

**总结:**

- 在Python 2.x中，True不是关键字，而是在bool类型中定义为1的[built-in global constant](http://docs.python.org/library/constants.html#True)。因此，解释器仍然必须加载True的内容。换句话说，True是可重新分配的。

```py
Python 2.7.15 (default, Jan 12 2019, 21:07:57)
[GCC 4.2.1 Compatible Apple LLVM 10.0.0 (clang-1000.11.45.5)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> True = 4
>>> True
4
```

- 在Python 3.x中，True也是[关键字](http://docs.python.org/py3k/reference/lexical_analysis.html#keywords)了。

```py
Python 3.7.0 (default, Jun 29 2018, 20:13:13)
[Clang 9.1.0 (clang-902.0.39.2)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> True = 4
  File "<stdin>", line 1
SyntaxError: can't assign to keyword
```

**参考:**

- https://stackoverflow.com/questions/3815359/while-1-vs-for-whiletrue-why-is-there-a-difference
