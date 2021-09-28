---
title: 在 Apple M1 python3.9上安装psycopg2
tags:
  - python
  - psycopg2
date: 2021-09-28 16:36:04
categories: python
keywords:
description: 在Apple M1做python开发，需要重新编译psycopg2
---



在MacBook M1上使用 pip install psycopg2时，报如下错误

`Error: pg_config executable not found`

![BA27CAD4-A8E8-4216-A7C6-BEB0B0DEADBC](https://oss.smart-lifestyle.cn/file/dcn67.png)



是因为目前pip源里目标没有ARM版本的psycopg2，需要进行手动编译才可以用

![D0DACCD0-12AD-4CA6-B93B-1182C36F0D9A](https://oss.smart-lifestyle.cn/file/itzxs.png)



* 安装postgresql

  ```bash
  brew install postgresql
  ```

* 再次执行pip install即可

  ```bash
  pip install psycopg2
  ```

  

## 参考

[1] [psycopg2-binary support for M1 chipset · Issue #1286 · psycopg/psycopg2 (github.com)](https://github.com/psycopg/psycopg2/issues/1286)
