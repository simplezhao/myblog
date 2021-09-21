---
title: 通过ftplib获取ftp指定日期间的文件
tags:
  - ftplib
  - python
categories: python
abbrlink: e530
date: 2020-05-20 10:09:58
description: ftplib的日期使用
---
## 使用python ftplib 库获取 某一天的文件列表，并下载到本地


```python
# ftp 设置
ftp_server = "******"
ftp_user = "*****"
ftp_password = "******"
```


```python
from ftplib import FTP

ftp = FTP(host=ftp_server, user=ftp_user, passwd=ftp_password)
# 通过nlst获取文件列表
file_list = ftp.nlst()
# 通过voidcmd 获取文件更新（上传）时间
date = ftp.voidcmd(f"MDTM 202002190502.csv")
print(date)
```

```
# 213 20200219170403
```

```python
# 获取文件
filter_date = '20200520'

# 遍历所有文件，生成一个迭代器
items = filter(lambda x: ftp.voidcmd(f"MDTM {x}")[4:12] == filter_date, file_list)
# 遍历迭代器, 通过retrbinary下载文件
for item in items:
    print(f"file name: {item}")
    with open(f'temp/{item}', 'wb') as fp:
        ftp.retrbinary(f'RETR {item}', fp.write)
# 关闭ftp
ftp.quit()
```

```
file name: HMI123456789_20200508.csv
file name: HMI123456789_20200509.csv
file name: HMI123456789_20200514.csv
file name: HMI123456789_20200520.csv

'221 Goodbye.'
```

```python
# 当ftp 服务器关闭是，再遍历过滤后的迭代器将为空
for item in items:
    print(itme)
    
```

```python

```
另外还可以通过ftp.dir获取，具体实现参考文档：https://www.semicolonworld.com/question/57790/how-to-get-ftp-file-39-s-modify-time-using-python-ftplib


参考：
1. [https://docs.python.org/3/library/ftplib.html][1]
2. [https://stackoverflow.com/questions/8990598/python-ftp-get-the-most-recent-file-by-date?answertab=votes#tab-top][2]
3. [https://stackoverflow.com/questions/29026709/how-to-get-ftp-files-modify-time-using-python-ftplib][3]
4. [https://tools.ietf.org/html/rfc3659#section-3][4]
5. [ https://www.semicolonworld.com/question/57790/how-to-get-ftp-file-39-s-modify-time-using-python-ftplib ][5]


[1]:	https://docs.python.org/3/library/ftplib.html "ftplib"
[2]:	https://stackoverflow.com/questions/8990598/python-ftp-get-the-most-recent-file-by-date?answertab=votes#tab-top "Python FTP get the most recent file by date"
[3]:	https://stackoverflow.com/questions/29026709/how-to-get-ftp-files-modify-time-using-python-ftplib "How to get FTP file's modify time using Python ftplib"
[4]:	https://tools.ietf.org/html/rfc3659#section-3 "rfc3659"
[5]:	https://www.semicolonworld.com/question/57790/how-to-get-ftp-file-39-s-modify-time-using-python-ftplib