---
title: 在Ubuntu中创建sudo user
tags:
  - ubuntu
categories: linux
abbrlink: c222
date: 2020-04-27 10:09:58
description: 创建超级用户
---
* 创建用户
```sh
# 创建用户为username的用户，并指定主目录home
sudo user add [username] --home [home]
# 为创建的用户设定密码
passwd [username]
```
* 通过usermod命令将用户添加到sudo group（部署root组）
```sh
sudo usermode -aG sudo [username]
# 之后就可以使用sudo 将当前用户权限提升到管理员权限
su [username]
# 输入密码
# sudo + command 执行命令
```
参考：
1. https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart

2. https://ohmyz.sh/