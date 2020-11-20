---
title: ZSH下使用Anaconda
tags:
  - Anaconda
  - ZSH
categories: python
abbrlink: e37f
---

> 在zsh下面找不到conda或者查看python，并不是用的anaconda版本的
> anaconda的安装参考：https://docs.anaconda.com/anaconda/install/
```sh
# 编辑zshrc文件，将下面这句加到zshrc中；anaconda_home为anaconda的安装目录
export PATH="[anaconda_home]/bin:$PATH"
```
参考：
1. https://www.jianshu.com/p/74b1c60148e8