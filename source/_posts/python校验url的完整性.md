---
title: python校验url的完整性
tags:
  - python
  - urlparse
  - 正则表达
categories: python
description: '校验url是否完整，比如http(s)://www.baidu.com'
abbrlink: 2c0e
date: 2021-09-28 10:55:30
keywords:
---

在抓取链接之后，我们需要对链接进行清洗，把格式不正确的链接去除，以便后面数据抓取

* https://www.baidu.com ✅
* httpz://www.baidu.com ❌
* \# ❌
* ftp://a.com/file/1111 ✅ 

以下总结两种方式来判断url的完整性

### 通过python urlparse库来实现

```python
# 这种没有协议头的，无法返回协议头，以及网络位置，把实际的网络位置定义为了路径部分
>>> urlparse('www.baidu.com')
ParseResult(scheme='', netloc='', path='www.baidu.com', params='', query='', fragment='')
# 正确识别到url协议，以及网络位置
>>> urlparse('https://www.baidu.com')
ParseResult(scheme='https', netloc='www.baidu.com', path='', params='', query='', fragment='')
# 一个 # 返回一个‘空’对象
>>> urlparse('#')
ParseResult(scheme='', netloc='', path='', params='', query='', fragment='')

# 但是向这种格式正确，但是url协议头不对的，虽然能解析，但是不会提示错误
>>> urlparse('ssp://www.baidu.com')
ParseResult(scheme='ssp', netloc='www.baidu.com', path='', params='', query='', fragment='')




```

urlparse的功能强大，支持多种协议格式的解析

下面的例子将用户名、密码、服务器地址、端口等全部解析出来

```bash
>>> data = urlparse('wss://username:password@192.168.1.1:90/s/data#video1')
>>> data.username
'username'
>>> data.password
'password'
>>> data.port
90
>>> data.hostname
'192.168.1.1'
# netloc解析范围应该是//.../中间的内容
>>> data.netloc
'username:password@192.168.1.1:90'
```



如果是用来解析http(s)，可以参考下面的代码

```python
from urllib.parse import urlparse

myurl = 'https://smart-lifestyle.cn/404.html'
ret = urlparse(myurl)
if ret.scheme in ('https', 'http') and ret.netloc:
  fetch_data(url)
 else:
  raise Exception('URL is not vaild')
```



### 通过正则表达匹配来实现

也可以使用正则表达式实现自己的解析功能，比如我们只判断http和https的url格式，可以这么做

这里参考了[Django](https://github.com/django/django/blob/main/django/core/validators.py#L74)的代码

```python
import re
ipv4_re = r'(?:0|25[0-5]|2[0-4]\d|1\d?\d?|[1-9]\d?)(?:\.(?:0|25[0-5]|2[0-4]\d|1\d?\d?|[1-9]\d?)){3}'
ipv6_re = r'\[[0-9a-f:.]+\]'
ul = '\u00a1-\uffff'
domain_re = r'(?:\.(?!-)[a-z' + ul + r'0-9-]{1,63}(?<!-))*'
tld_re = (
        r'\.'                                # dot
        r'(?!-)'                             # can't start with a dash
        r'(?:[a-z' + ul + '-]{2,63}'         # domain label
        r'|xn--[a-z0-9]{1,59})'              # or punycode label
        r'(?<!-)'                            # can't end with a dash
        r'\.?'                               # may have a trailing dot
    )
hostname_re = r'[a-z' + ul + r'0-9](?:[a-z' + ul + r'0-9-]{0,61}[a-z' + ul + r'0-9])?'
host_re = '(' + hostname_re + domain_re + tld_re + '|localhost)'
regex = re.compile(
        r'^(?:[a-z0-9.+-]*)://'  # scheme is validated separately
        r'(?:[^\s:@/]+(?::[^\s:@/]*)?@)?'  # user:pass authentication
        r'(?:' + ipv4_re + '|' + ipv6_re + '|' + host_re + ')'
        r'(?::\d{1,5})?'  # port
        r'(?:[/?#][^\s]*)?'  # resource path
        r'\Z', re.IGNORECASE)

if __name__ == '__main__':
    url1 = 'https://www.baidu.com'
    url2 = 'http://www.baidu.com'
    url3 = 'ht:/www.baidu.com'
    url4 = 'www.baidu.com'
    print(regex.match(url1))
    print(regex.match(url2))
    print(regex.match(url3))
    print(regex.match(url4))
```

返回结果如下：

```bash
<re.Match object; span=(0, 21), match='https://www.baidu.com'>
<re.Match object; span=(0, 20), match='http://www.baidu.com'>
None
None
[Finished in 78ms]
```



在Django的代码里，对于url协议类型的判断它没有集成到这个正则匹配中，而是先判断url协议，再解析格式，这样他可以灵活定义支持哪些url协议

```python
schemes = ['http', 'https', 'ftp', 'ftps']
# Check if the scheme is valid.
scheme = value.split('://')[0].lower()
if scheme not in self.schemes:
    raise ValidationError(self.message, code=self.code, params={'value': value})
```



### 参考

[1] [urllib.parse — Parse URLs into components — Python v3.0.1 documentation](https://docs.python.org/3.0/library/urllib.parse.html)

[2] [django/validators.py at main · django/django (github.com)](https://github.com/django/django/blob/main/django/core/validators.py#L74)
