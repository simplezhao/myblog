---
title: python文件名操作
tags:
  - python
  - file
date: 2021-06-30 13:41:58
categories: python
keywords:
description: 获取文件路径、文件名、文件后缀
---

在进行文件上传或者文件转换时，需要进行文件上传（读取完整的本地文件路径），文件保存（仅文件名），格式转换（不包含后缀的文件名以及后缀名）



使用到python库有：

* os.path.splitext()

  分离文件名和文件后缀，以最后一个'.'来获取文件后缀

* os.path.split()

  分离文件路径和文件名



1. os.path.splitext()

```python
>>> from os.path import splitext
>>> splitext('/Users/simple/workspace/tmp/traefik/docker-compose.yml')
('/Users/simple/workspace/tmp/traefik/docker-compose', '.yml')

# 如果输入是一个不包含路径的隐藏文件格式，后缀返回为空
>>> splitext('.bashrc')
('.bashrc', '')
```

2. os.path.split()

```python
>>> from os.path import split
>>> split('/Users/simple/workspace/tmp/traefik/docker-compose.yml')
('/Users/simple/workspace/tmp/traefik', 'docker-compose.yml')

# 如果输入为一个不含路径的以.开头的文件，路径返回为空
>>> split('.bascrc')
('', '.bascrc')
```



## 参考

[1] [os.path — Common pathname manipulations — Python 3.9.6 documentation](https://docs.python.org/3/library/os.path.html)

[2] [Python获取文件路径、文件名和扩展名_小龙在线-CSDN博客_python 获取路径文件名](https://blog.csdn.net/lilongsy/article/details/99853925)



