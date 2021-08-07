---
title: linux shell参数传递
tags:
  - linux
  - shell
categories: linux
description: 在shell脚本中获取执行时，传递的参数
abbrlink: fab
date: 2021-07-01 21:10:52
keywords:
---

| $#   | 参数数量                              |
| ---- | ------------------------------------- |
|      |                                       |
| $n   | 表示第n个参数，n为0时表示运行的脚本名 |
| $@   | "$1" "$2" "$3".... 传递的多个参数     |



```shell
#!/bin/bash

echo "total params: $#"
echo "filename: $0"
echo "param1: $1"
echo "param2: $2"
echo "param3: $3"
echo "=============="
for i in "$@"; do
    echo "param: $i"
done

```

```bash
total params: 2
filename: ./test.sh
param1: x1
param2: x2
param3:
==============
param: x1
param: x2
```



## 参考

[1] [Shell 传递参数 | 菜鸟教程 (runoob.com)](https://www.runoob.com/linux/linux-shell-passing-arguments.html)

