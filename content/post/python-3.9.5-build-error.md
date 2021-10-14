+++
description = ""
date = "2021-06-05T13:15:08+08:00"
title = "python 3.9.5 编译报错 Could not import runpy module 问题"
tags = ["Python"]

+++

## 环境

- CentOS 7.3.1611
- gcc-4.8.5-28.el7_5.1
- Python 3.9.5

## 相关报错

```sh
./python -E -S -m sysconfig --generate-posix-vars ;\
if test $? -ne 0 ; then \
	echo "generate-posix-vars failed" ; \
	rm -f ./pybuilddir.txt ; \
	exit 1 ; \
fi
Could not import runpy module
Traceback (most recent call last):
  File "/tmp/Python-3.9.5/Lib/runpy.py", line 15, in <module>
    import importlib.util
  File "/tmp/Python-3.9.5/Lib/importlib/util.py", line 2, in <module>
    from . import abc
  File "/tmp/Python-3.9.5/Lib/importlib/abc.py", line 17, in <module>
    from typing import Protocol, runtime_checkable
  File "/tmp/Python-3.9.5/Lib/typing.py", line 21, in <module>
    import collections
SystemError: <built-in function compile> returned NULL without setting an error
generate-posix-vars failed
make[1]: *** [pybuilddir.txt] Error 1
make[1]: Leaving directory `/tmp/Python-3.9.5'
make: *** [profile-opt] Error 2
```

## 导致原因

- 在低版本的gcc版本中带有 `--enable-optimizations` 参数时会出现上面问题
- gcc 8.1.0 修复此问题

## 解决方法

- 1、升级gcc至8.1.0【不推荐】
- 2、./configure参数中去掉--enable-optimizations

## 参考链接

- https://bugs.python.org/issue34112
