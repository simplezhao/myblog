---
title: QPS和TPS
tags:
  - 测试
  - 性能指标
date: 2021-06-29 16:38:03
categories: 测试
keywords: 
description: 理解QPS和TPS的区别
---

* QPS

  英文全称：Queries Per Second，即每秒的响应的请求数，也代表最大吞吐能力，QPS的**一次请求**代表一个接口的一次请求到服务器返回结果

* TPS

  英文全称：Transactions Per Second，即每秒处理的事务数目，一个事务指的是**一个客户端**向服务器发送请求并返回结果的过程。
  
  例如，某次客户访问一个页面会请求4次服务器，其中1次html，1次css，1次js，1次API，那么访问这个页面会产生一个“T”，而会产生四个“Q”。
  
* PV

  英文全称 Page View，即页面浏览量，用户每一次对网站中每个页面访问，就会被记录一次，多次刷新一个页面会累计PV

* UV

  英文全称：Unique Visitor，即独立访客访问数，统计1天内访问某站点的用户数（以cookie为依据）

* IP

  即独立IP数，指的是一天内多少个独立IP浏览页面，如果多个用户在同一个路由器内上网，可能会被记录为1个独立IP访问者
