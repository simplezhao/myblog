---
title: jupyter notebook使用环境变量
tags:
  - jupyter
  - env
categories: jupyter
description: 在jupyter notebook中使用环境变量
abbrlink: 9e2a
date: 2021-06-18 15:44:57
keywords:
---

> 使用python-dotenv库，在jupyter notebook中加载环境变量



* 安装python-dotenv

```bash
pip install python-dotenv
```

* 使用

```python
from dotenv import load_dotenv
import os
load_dotenv(dotenv_path='.env')

db_user = os.getenv('DB_USER')
db_password = os.getenv('DB_PASSWORD')
db_host = os.getenv('DB_HOST')
db_name = os.getenv('DB_NAME')
```



## 参考

[1] [Read Environment Variables in a Python Notebook · Nono Martínez Alonso](https://nono.ma/environment-variable-python-notebook-os-environ-get)

