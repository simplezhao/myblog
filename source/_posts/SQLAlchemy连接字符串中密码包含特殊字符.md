---
title: SQL连接字符串中密码包含特殊字符
tags:
  - SQL
  - SQLAlchemy
date: 2021-07-01 16:59:35
categories: SQLAlchemy
keywords:
description: SQLAlchemy连接字符串中密码包含特殊字符，可以进行url编码
---

生产环境中，数据库的密码会包含特殊字符，如果包含了`@`等符号，会被create_engine识别错误。

可以对密码进行url编码，使用`urllib.parse.quote_plus`

> As the URL is like any other URL, **special characters such as those that may be used in the password need to be URL encoded to be parsed correctly.**. Below is an example of a URL that includes the password `"kx%jj5/g"`, where the percent sign and slash characters are represented as `%25` and `%2F`, respectively:

```python
from sqlalchemy import create_engine
import urllib.parse
username = 'scoot'
password = '@#@%/g'

engine = create_engine(f'postgresql://{username}:{urllib.parse.quote_plus(password)}@localhost:5432/mydatabase')
```



## 参考

[1] [Engine Configuration — SQLAlchemy 1.4 Documentation](https://docs.sqlalchemy.org/en/14/core/engines.html)

