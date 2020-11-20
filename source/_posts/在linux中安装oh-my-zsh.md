---
title: 在linux中安装oh-my-zsh
tags:
  - oh-my-zsh
categories: zsh
abbrlink: ea0a
---

> oh-my-zsh 是zsh的管理配置工具，因此在使用oh-my-zsh之前安装zsh

![zsh][image-1]
1. 判断当前系统使用的shell
```sh
# 打印当前系统使用的shell
echo $SHELL
# 可能会输出
# /bin/bash
# /bin/sh
# /bin/zsh

# 查看当前系统支持的shell
cat /etc/shells
```
2. 安装zsh
```sh
# 如果未安装zsh
sudo apt-get install zsh
```
3. 切换shell为zsh
```sh

# 需要重启
chsh -s /bin/zsh
```
4. 安装 oh my zsh
```sh
# via curl
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
# via wget
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
参考：
1. https://www.jianshu.com/p/d194d29e488c

[image-1]:	https://oss.smart-lifestyle.cn/blog/w5ahz.jpg