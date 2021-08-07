---
title: python url decode
tags:
  - python
  - urlenocde
categories: python
description: 解码url编码格式的参数
abbrlink: '6645'
date: 2021-06-29 22:42:02
keywords:
---

在http请求中，如果url query中是Unicode，将会议url encode方式发送到服务端，可能需要我们去解析，这里用到python的库`urllib.parse.unquote`

```http
GET /?w=%E6%B1%89%E5%AD%97%EC%A4%91%EA%B5%AD HTTP/1.1
Content-Type: application/x-www-form-urlencoded; charset=utf-8
Host: echo.paw.cloud
Connection: close
User-Agent: Paw/3.2.2 (Macintosh; OS X/11.4.0) GCDHTTPRequest
```



```python
>> from urllib.parse import unquote
>> unquote('/?w=%E6%B1%89%E5%AD%97%EC%A4%91%EA%B5%AD')
'/?w=汉字중국'
```





## 参考

[1] [urllib.parse — Parse URLs into components — Python 3.9.6 documentation](https://docs.python.org/3/library/urllib.parse.html#urllib.parse.unquote)

[2] [encoding - Url decode UTF-8 in Python - Stack Overflow](https://stackoverflow.com/questions/16566069/url-decode-utf-8-in-python)
