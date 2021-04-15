---
title: git仓库用户权限问题
tags:
  - git
categories: git
keywords: git
description: 在仓库地址上加上用户名，来更改对该仓库的授权认证
abbrlink: f5f9
date: 2021-04-15 20:44:59
---

> 通常我们使用一个github账号进行代码的拉去和推送，但假如现在有一个私有仓库，而且需要使用另外一个github账号，可能会出现authentication error

如果我们使用`git config user.email`是没有办法更改这个仓库的认证信息的

这是因为，我们通过https操作仓库时，会使用缓存在本机的授权认证信息，也就是会使用之前账户的信息。

更改方法比较简单，给仓库地址上加上你所要使用的用户名(username2)即可

```bash
cd repo
git remote set-url origin https://username2@github.com/<username>/repo
```



## 参考

[1] [How to change user for git repository](https://stackoverflow.com/questions/65932601/how-to-change-user-for-git-repository)