---
title: node查看process架构
tags:
  - node
  - 前端
categories: node
description: 在MacBook M1架构下使用了多版本的node，通过node -p "process.arch"查看当前node的编译架构
abbrlink: 163c
date: 2021-09-22 08:59:52
keywords:
---

现在node的新版本已经支持MacBook M1，即ARM版本的node，但是有时候需要老版本的node，目前我使用nvm可以切换node的版本， 查看当前node是x64还是ARM版本，可以通过下面的命令查看

```bash
node -p "process.arch"
```

![image-20210922090704217](https://oss.smart-lifestyle.cn/file/uw2au.png)
