---
title: mac下git status中文乱码
tags:
  - git
date: 2021-09-22 09:18:47
categories: 工具
keywords:
description: 在mac命令行下中文乱码的问题
---

文件名字是中文时，mac下执行git status命令查看状态时，会显示一堆unicode编码的字符，不便于查看，因为默认情况下对于大于0x80的文件路径字符会进行编码，可以将`core.quotePath`设置为false，来取消强制编码

![image-20210922093322918](https://oss.smart-lifestyle.cn/file/ck0gv.png)

```bash
git config --global core.quotepath false
```

![image-20210922093348556](https://oss.smart-lifestyle.cn/file/zym74.png)



## 参考

[1] [Git - git-config Documentation (git-scm.com)](https://git-scm.com/docs/git-config)

[2] [mac下git中文乱码 - 破男孩 - 博客园 (cnblogs.com)](https://www.cnblogs.com/ayseeing/p/4268655.html)

