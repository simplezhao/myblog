---
title: Apple M1 node多版本切换
tags:
  - nvm
  - node
categories: node
description: Apple M1下，如果管理多个版本的node
abbrlink: aa65
date: 2021-10-11 16:35:53
keywords:
---

本文介绍如何使用nvm在Macbook M1下管理多个版本（架构）的node

目前Node V16已经原生支持Macbook M1，但是如果选择V16以下的版本，需要安装X86的版本

![image-20211011164226288](https://oss.smart-lifestyle.cn/file/brnx9.png)



## 安装nvm

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
# 或者
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
```

## 安装node

### 最新ARM版本V16+

```bash
nvm install node
```

```bash
node -v
v16.6.0
node -p process.arch
arm64
```

### 安装X86版本

#### 切换到x86环境

```bash
arch -x86_64 zsh
```

#### 查看当前环境

```bash
arch
# i386

```

#### 安装node

```bash
nvm install v14
```

```bash
node -p process.arch
x64
```





## 参考

[1] [nvm-sh/nvm: Node Version Manager - POSIX-compliant bash script to manage multiple active node.js versions (github.com)](https://github.com/nvm-sh/nvm#installing-and-updating)

[2] [macOS Big Sur: How to setup Node.js on Apple M1 Machine – Jurnal Anas](https://www.jurnalanas.com/node-js-mac-m1/)



