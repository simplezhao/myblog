---
title: docker运行linux终端显示bash-4.2#
tags:
  - linux
  - docker
date: 2021-05-23 15:39:12
categories:
keywords:
description: docker运行linux终端显示bash-4.2#
---

一次在docker中运行centos，为了省事，将数据卷挂载了整个用户工作目录下，执行`docker run -it -v /Users/simple/workspace/tmp/data:/root python36:centos7 bash`

![image-20210523154159576](https://oss.smart-lifestyle.cn/file/k5nrn.png)

理论上应该按如下显示：

![image-20210523155114587](https://oss.smart-lifestyle.cn/file/0vks3.png)

发现此时终端显示的bash-4.2，而不是显示用户名@主机，另外终端文件夹文件的颜色也没有显示，最开始以为没有设置term颜色，但经过尝试不是这个问题（`TERM=xterm-256color`）。

看到自己运行的docker命令，-v将data目录挂载了用户目录下，这时候想起里啊，docker volume第一次挂载是单项的，即及时容器内目录有内容，第一次挂载时，也会被主机目录覆盖掉，也就是现在主机data目录是空的，会把docker内用户目录内文件全部清空

​	进行挂载后：

![image-20210523154824106](https://oss.smart-lifestyle.cn/file/io22a.png)

​	挂载前：

![image-20210523155224464](https://oss.smart-lifestyle.cn/file/gq6l8.png)

在用户目录下，存放着bash以及用户的各种配置文件，而进行挂载后，将所有配置文件全部清除了。

正确做法，不要直接挂载到用户目录下，而应该挂载用户目录下的二级目录

```shell
#!/bin/bash
volume="/Users/simple/workspace/function_graph/data"
docker run -it --rm --init  --name centos -v $volume:/root/data -w /root python36:centos7 bash
```



## 参考

[1] ([linux 命令终端提示符显示-bash-4.2#解决方法__kairui的博客-CSDN博客_-bash-4.2#](https://blog.csdn.net/liulihui1988/article/details/52796395))

