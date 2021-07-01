---
title: 修改SQL Server连接字符串
tags:
  - SQL
  - sqlserverException
date: 2021-06-30 11:13:50
categories: SQL
keywords:
description: 在DataGrip中解决SQL Server连接报“驱动程序无法使用安全套接字层（SSL）加密建立到SQL Server的安全连接”
---

> com.microsoft.sql server.jdbc.sqlserverException:驱动程序无法使用安全套接字层（SSL）加密建立到SQL Server的安全连接

在高级设置中trustServerCertificate改为true即可

![image-20210630111813541](https://oss.smart-lifestyle.cn/file/cs6v8.png)



## 参考

[1] [修改SQL Server连接字符串-FME社区 - 亚搏在线 (baooytra.com)](https://www.baooytra.com/knowledge/questions/56246/modifying-sql-server-connection-string.html?smartspace=fme-desktop-getting-started_2)
