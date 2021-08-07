---
title: pandas与SQL整理-1
tags:
  - pandas
  - sqlalchemy
categories: pandas
keywords: pandas
description: 使用pandas读取数据表，并导出excel
abbrlink: eac6
date: 2021-06-18 14:08:16
---



## 读取数据库内容

```python
import os
from sqlalchemy import create_engine
import pandas as pd

# excel header无格式
pd.io.formats.excel.ExcelFormatter.header_style = None

db_user = os.getenv('DB_USER')
db_password = os.getenv('DB_PASSWORD')
db_host = os.getenv('DB_HOST')
db_name = os.getenv('DB_NAME')


engine = create_engine(f'postgresql://{db_user}:{db_password}@{db_host}:5432/{db_name}')

# 读取表
df = pd.read_sql_table('AssetWeeklyReportView', engine)

# 保存到excel中
df.to_excel('asset_table.xlsx', index=False)

# 通过是否为空进行过滤
df1 = df[df['Type'].notnull()]
df2 = df[df['Type'].isnull()]

# 获取行数
size = len(df1)

# 遍历每一行
for index, row in df1.iterrows():
  print(row['Type'].....)
```





## 参考

[1] [pandas dataframe len() 和 count() 的区别_weixin_45616551的博客-CSDN博客](https://blog.csdn.net/weixin_45616551/article/details/103469386)

[2] [IO tools (text, CSV, HDF5, …) — pandas 1.2.4 documentation (pydata.org)](https://pandas.pydata.org/pandas-docs/stable/user_guide/io.html#sql-queries)

