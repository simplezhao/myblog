---
title: python json.dumps unicode
tags:
  - python
  - json
categories: python
description: 处理json dumps时中文显示unicode编码
abbrlink: b1
date: 2021-03-31 15:54:04
keywords:
---

> 在dumps/dump中使用ensure_ascii=False，来手动编码为UTF8格式



```python
import json
# 输出为Unicode编码，不便于可视化
> json.dumps("我爱China🇨🇳")
'"\\u6211\\u7231China\\ud83c\\udde8\\ud83c\\uddf3"'

> json.dumps("我爱China🇨🇳", ensure_ascii=False)
'"我爱China🇨🇳"'
```



## 参考

[1] [json dumps unicode](https://stackoverflow.com/questions/18337407/saving-utf-8-texts-with-json-dumps-as-utf8-not-as-u-escape-sequence)

