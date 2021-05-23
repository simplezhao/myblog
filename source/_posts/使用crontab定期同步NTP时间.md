---
title: 使用crontab定期同步NTP时间
tags:
  - crontab
  - linux
categories: linux
description: 使用crontab定期同步NTP时间
abbrlink: c57a
date: 2021-05-21 14:33:40
keywords:
---

购买的阿里云香港虚机，不知道怎么回事，过一段时间，它的时间就跟实际时间偏离越来越远，导致HTTPS无法正常工作，

最开始是手动的登录的服务器，使用`rdate -s time.nist.gov`进行手动更新时间，但后来发现自己越来越懒，而且最近这个命令一直出错: `rdate: Could not read data: Cannot assign requested address`, 可能是后面的时间同步服务器挂了。

因此改为使用`ntpdate`来同步时间，使用阿里云的ntp服务器ntp.cloud.aliyuncs.com

```bash
ntpdate ntp.cloud.aliyuncs.com
23 May 14:43:07 ntpdate[9801]: step time server 100.100.61.88 offset 3.848040 sec
```

然后将改名了加入crontab任务中

```bash
crontab -e
# 将下面命令添加到最后
* * * * * ntpdate ntp.cloud.aliyuncs.com
```

保存退出，但是实际运行一段时间，发现服务的实际还是距离真实时间越来越远，感觉是ntpdate命令没有工作，于是将定时任务改为：

```bash
* * * * * ntpdate ntp.cloud.aliyuncs.com >> /var/log/myntp.log 2>&1
```

保存运行退出后，查看ntpdate运行日志，果然存在问题`/bin/sh: 1: ntpdate: not found`

```ba
tail -f myntp.log
/bin/sh: 1: ntpdate: not found
/bin/sh: 1: ntpdate: not found
/bin/sh: 1: ntpdate: not found
```

cron在运行时，找不到ntpdate命令，继续排查

```bash
# which ntpdate
/usr/sbin/ntpdate
```

在crontab任务中增加一行`* * * * * env >> /var/log/env.log 2>&1`

并查看日志

```bash
# tail -f env.log
PATH=/usr/bin:/bin
LANG=en_US.UTF-8
SHELL=/bin/sh
PWD=/root
```

终于发现问题了，crontab运行时查找命令的路径为`/usr/bin` 和`/bin`目录，而不包含`/usr/sbin/`

我们可以在crontab中增加`PATH=$PATH:/usr/sbin/`,也可以使用ntpdate的绝对路径，将之前ntpdate任务改为如下：

```she
* * * * * /usr/sbin/ntpdate ntp.cloud.aliyuncs.com
```

保存退出后，观察服务器时间，发现已经变为正常时间了。

## crontab设置

最前面5段用于设置周期时间

![image-20210521150550791](https://oss.smart-lifestyle.cn/file/rdivm.png)

这里举一些常用的例子，可以使用这个[网站](https://crontab.guru/)进行测试

* 每一分钟执行一次

  `* * * * *`

  ![image-20210521145726370](https://oss.smart-lifestyle.cn/file/sgflk.png)

* 每隔十分钟运行一次(📢这里的每个10分钟，不是从现在或者任务开始算起，而是每小时的第十分钟，10，20， 30....)

  `*/10 * * * *`

  ![image-20210521145941538](https://oss.smart-lifestyle.cn/file/qpet2.png)

* 每天晚上23：55执行一次

  `55 23 * * *`

  ![image-20210521150051826](https://oss.smart-lifestyle.cn/file/udg1l.png)

* 每隔两个月的第一天晚上23：55执行一次（比如用于ssl证书的更新）

  `55 23 1 */2 *`

  ![image-20210521150255631](https://oss.smart-lifestyle.cn/file/w278h.png)

* 每周五晚上23：55发送报表

  `55 23 * * 5`

  ![image-20210521150430138](https://oss.smart-lifestyle.cn/file/xlh36.png)



## 参考

[1] [crontab tool](https://crontab.guru/#*_*_*_*_*)

[2] [log cron jobs](https://stackoverflow.com/questions/4811738/how-to-log-cron-jobs)

[3] [ntpdate: command not found](https://www.cnblogs.com/centos2017/p/12963610.html)

