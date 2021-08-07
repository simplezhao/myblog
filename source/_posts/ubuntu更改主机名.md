---
title: ubuntu更改主机名
tags:
  - ubuntu
  - 运维
categories: 运维
description: 修改公有云上linux VM主机名
abbrlink: '62e8'
date: 2021-07-01 22:17:51
keywords:
---

* 增加或者修改`/etc/hostname`，添加新的主机名
* 修改`/etc/hosts`，在127.0.0.1中添加新的主机名
* 如果是在云上运行vm实例，需要修改上面的`/etc/cloud/cloud.cfg`，否则重启机器后，hostname会变为默认值😭；找到`preserve_hostname`，将其值改为`true`



## 参考

[1] [如何在Ubuntu 20.04上更改主机名 (myfreax.com)](https://www.myfreax.com/how-to-change-hostname-on-ubuntu-20-04/)

