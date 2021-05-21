---
title: python递归创建目录(mkdir -p)
tags:
  - python
  - python小技巧
categories: python
description: 使用python递归创建目录（类似于mkdir -p)
abbrlink: '7376'
date: 2021-05-22 16:15:50
keywords:
---

在为日志文件设置存放位置时，需要在程序里判断文件夹（位置）是否存在，通常使用`os.mkdir`来创建目录，但是如果目录为多层次目录，而且某一层目录存在，在使用`os.mkdir`时会报错：FileNotFoundError*

```python
>>> os.mkdir('/tmp/logpath/path')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
FileNotFoundError: [Errno 2] No such file or directory: '/tmp/logpath/path'
```

这是因为os.mkdir不支持多层目录递归创建。

在linux mkdir命令中，它支持如下操作，使用-p 可以实现递归创建目录

```bash
tldr mkdir
Cache is out of date. You should run "tldr --update"

  mkdir

  Creates a directory.
  More information: https://www.gnu.org/software/coreutils/mkdir.

  - Create a directory in current directory or given path:
    mkdir directory

  - Create directories recursively (useful for creating nested dirs):
    mkdir -p path/to/directory
```

而在python中我们可以使用pathlib.Path.mkdir来实现，需要让`exist_ok=True` 以及`parents=True`

```python
Path.mkdir(mode=0o777, parents=False, exist_ok=False)
If exist_ok is true, FileExistsError exceptions will be ignored (same behavior as the POSIX mkdir -p command), but only if the last path component is not an existing non-directory file.
```

```python
>>> Path('/tmp/logtmp/XMKDRL').mkdir(exist_ok=True, parents=True)
>>> os.path.exists('/tmp/logtmp/XMKDRL')
True
```



## 参考

[1] [mkdir-p-functionality-in-python]([mkdir -p functionality in Python - Stack Overflow](https://stackoverflow.com/questions/600268/mkdir-p-functionality-in-python))

[2] [Path.mkdir]([pathlib — Object-oriented filesystem paths — Python 3.9.5 documentation](https://docs.python.org/3/library/pathlib.html#pathlib.Path.mkdir))
