---
title: python基础 -- time与datetime常见用法
tags:
  - python
categories: python
description: str、time、datetime之间的转换
abbrlink: 3a95
keywords:
date: 2021-04-30 10:09:58
description: python datetime
---

在HTTP API接口，以及跟数据库交互时经常用到时间戳（时间），这里整理python中常见的time、datetime用法，以及之间的相互转化方法



```python
import time
from datetime import datetime
from datetime import timedelta
from datetime import timezone
```



## time基本用法

```python
# 获取当前unix时间戳，返回值为float
> time.time()
1615178606.6729627

# 获取本地时间（struct time）
> time.localtime()
time.struct_time(tm_year=2021, tm_mon=3, tm_mday=8, tm_hour=12, tm_min=47, tm_sec=12, tm_wday=0, tm_yday=67, tm_isdst=0)

# 获取UTC时间（struct time）
> time.gmtime()
time.struct_time(tm_year=2021, tm_mon=3, tm_mday=8, tm_hour=4, tm_min=47, tm_sec=25, tm_wday=0, tm_yday=67, tm_isdst=0)

# 获取毫秒级Unix 时间戳 int
> int(time.time()*1000)
1615178896227

# 获取秒级Unix 时间戳 int
> int(time.time())
1615178924

# Unix 时间戳（int/float）转化为 struct time
> time.localtime(time.time())
time.struct_time(tm_year=2021, tm_mon=3, tm_mday=25, tm_hour=21, tm_min=44, tm_sec=6, tm_wday=3, tm_yday=84, tm_isdst=0)

# strcut time 转化为 Unix时间戳（float）
# 主要mktime的参数应该是localtime，不能是UTCtime
> time.mktime(time.localtime())
1616680137.0
```



## datetime基本用法

* datetime(year, month, day[, hour[, minute[, second[, microsecond[,tzinfo]]]]])

*注意：秒后面是微秒，不是毫秒*



```python
# 获取当前时刻的datetime
> datetime.now()
datetime.datetime(2021, 3, 25, 21, 59, 29, 534163)

# 获取当前时刻的UTC datetime
> datetime.utcnow()
datetime.datetime(2021, 3, 25, 13, 59, 39, 803165)

# 获取当前时刻的Unix 时间戳, 参数为 datetime
> datatime.timestamp(datetime(2021, 3, 25, 13, 59, 39, 803165))
1616651979.803165

# 更新datetime,replace(year, month, day, hour, minute, second, microsecond...)
> datetime.now().replace(year=2022)
datetime.datetime(2022, 3, 25, 21, 59, 29, 534163)


```

### iosformat用法

* isoformat(sep='T', timespec='auto')

默认情况下，如果微秒存在，则输出微秒，如果微秒为0，则微秒会被忽略，而不是输出0

```python
# 不修改默认参数时，返回格式化字符串
> datetime.isoformat(datetime(2021, 3, 25, 13, 59, 39, 803165))
'2021-03-25T13:59:39.803165'

# 支持两个参数，sep， timespec
# sep 默认为'T'，即用T做分隔符
# timespec默认为'auto', 可以为'hours', 'seconds', 'milliseconds', 'microseconds'
# 输出带微秒
> datetime.isoformat(datetime(2021, 3, 25, 13, 59, 39, 1))
'2021-03-25T13:59:39.000001'

# 微秒被忽略
> datetime.isoformat(datetime(2021, 3, 25, 13, 59, 39, 0))
'2021-03-25T13:59:39'

# 如果timespec为milliseconds，小于1ms会输出000
> datetime.isoformat(datetime(2021, 3, 25, 13, 59, 39, 2), timespec='milliseconds')
'2021-03-25T13:59:39.000'

# 等于0时，也会输出000
> datetime.isoformat(datetime(2021, 3, 25, 13, 59, 39), timespec='milliseconds')
'2021-03-25T13:59:39.000'

# 以空格作为分隔符
> datetime.isoformat(datetime(2021, 3, 25, 13, 59, 39, 1000), timespec='milliseconds', sep=' ')
'2021-03-25 13:59:39.001'
```

### timedelta

* datetime.timedelta(days, seconds, microseconds, minutes, hours, weeks)

两个datetime直接的时间间隔

> 只有 *days*,*seconds* 和 *microseconds* 会存储在内部，即python内部以 *days*,*seconds* 和 *microseconds* 三个单位作为存储的基本单位。参数单位转换规则如下：
>
> - 1毫秒会转换成1000微秒。
> - 1分钟会转换成60秒。
> - 1小时会转换成3600秒。
> - 1星期会转换成7天。

```python
# 比当前时刻少1天5分钟
> datetime.now() + timedelta(days=-1, minutes=-5)
> datetime.now() - timedelta(days=1, minutes=5)
datetime.datetime(2021, 3, 24, 23, 41, 10, 557550)
```

### timezone/astimezone

将UTC时间转化为本地时间

> 假如数据库存储的时间戳都是UTC时间，如果客户端获取到后，转化为本地时间，可以借助timezone和astimezone
>
> 简单可以使用timedelta(hours=±8)进行北京时间和UTC时间转换

具体如下：

```python
# 获取一个UTC时间
> date = datetime.utcnow()
> date
datetime.datetime(2021, 3, 29, 7, 22, 21, 240199)

# 有趣的时，如果此时获取tzinfo，不是utc，而是None
> date.tzinfo
None

# 因此先给它赋值一个UTC timezone
> new = date.replace(tzinfo=timezone.utc)
> new
datetime.datetime(2021, 3, 29, 7, 22, 21, 240199, tzinfo=datetime.timezone.utc)

# astimezone 接受一个tz参数，默认为None，为None时，转化为本地时间
# 28800 = 8h * 3600
> new.astimezone()
datetime.datetime(2021, 3, 29, 15, 22, 21, 240199, tzinfo=datetime.timezone(datetime.timedelta(seconds=28800), 'CST'))


```

本地时间转化为utc时间

> 写入到数据库时，需要将实际转化为UTC时间
>
> 方法一：通过astimezone转换，此时tz=timezone.info
>
> 方法二：借助于fromtimestamp/utcfromtimestamp转换

```python
# 获取一个本地时间
> localdate = datetime.now()
> localdate
datetime.datetime(2021, 3, 29, 16, 16, 33, 974434)

# 方法1，给datetime加上时区信息，然后通过astimezone转换
> new_localdate = localdate.replace(tzinfo=timezone(timedelta(seconds=28800)))
> new_localdate
datetime.datetime(2021, 3, 29, 16, 16, 33, 974434, tzinfo=datetime.timezone(datetime.timedelta(seconds=28800)))

# 转化时区
> utc_date = new_localdate.astimezone(tz=timezone.utc)
> utc_date
datetime.datetime(2021, 3, 29, 8, 16, 33, 974434, tzinfo=datetime.timezone.utc)

# 方法2，借助于timestamp转换
> timestamp = localdate.timestamp()
> utc_date = datetime.utcfromtimestamp(timestamp)
> utc_date
datetime.datetime(2021, 3, 29, 8, 16, 33, 974434)
# 或者使用fromtimestamp 加上时区参数
> utc_date = datetime.fromtimestamp(timestamp, tz=timezone.utc)
> utc_date
datetime.datetime(2021, 3, 29, 8, 16, 33, 974434, tzinfo=datetime.timezone.utc)
```



## str与datetime互转

* datetime.strptime(string, format) 将字符串时间转为datetime

* datetime.strftime(format) 将datetime转化为字符串时间

```python
# str --> datetime
# '2021-03-25T13:59:39.001'
> datetime_str = datetime.isoformat(datetime(2021, 3, 25, 13, 59, 39, 1000), timespec='milliseconds')

# %f 用于格式化微秒
> datetime.strptime(datetime_str, "%Y-%m-%dT%H:%M:%S.%f")
datetime.datetime(2021, 3, 25, 13, 59, 39, 1000)

# datetime --> str
# 推荐第一种用上面提到的isoformat
# 第二种使用strftime
> datetime(2021, 3, 25, 13, 59, 39, 1000).strftime(format="%Y-%m-%dT%H:%M:%S.%f")
'2021-03-25T13:59:39.001000'

# 将微秒输出为000
> datetime(2021, 3, 25, 13, 59, 39, 1000).strftime(format="%Y-%m-%dT%H:%M:%S.000")
'2021-03-25T13:59:39.000'

```



## str与time互转

* time.strftime(format[,t]) 将time换位为字符串时间，如果t(time)未提供，则使用locatime()获取当前时间

* time.strptime(string[,format]) 将字符串时间转换为 struct time

```python
# local time
# time.struct_time(tm_year=2021, tm_mon=3, tm_mday=25, tm_hour=23, tm_min=12, tm_sec=10, tm_wday=3, tm_yday=84, tm_isdst=0)
> now = time.localtime()

# time --> str
# 转化为字符串时间, 不支持微秒
> time.strftime( "%Y-%m-%dT%H:%M:%S", now)
'2021-03-25T23:12:10'

# str --> time
# 将字符串时间转换为 struct time
> time.strptime(time.strftime( "%Y-%m-%dT%H:%M:%S", now), "%Y-%m-%dT%H:%M:%S")

time.struct_time(tm_year=2021, tm_mon=3, tm_mday=25, tm_hour=23, tm_min=12, tm_sec=10, tm_wday=3, tm_yday=84, tm_isdst=-1)
```



## time与datetime互转

* datetime.timetuple(datetime) 将datetime转化为 struct time

* datetime.fromtimestamp(unix时间戳) 将unix时间戳转化为datetime

* datetime.timestamp() 将datetime转化为unix 时间戳

```python
# time --> datetime
# 第一种 unix 时间戳 转化为 datetime
> datetime.fromtimestamp(time.time())
datetime.datetime(2021, 3, 25, 23, 28, 33, 548042)

# 第二种 struct time转化为 datetime
> datetime.fromtimestamp(time.mktime(time.localtime()))
datetime.datetime(2021, 3, 25, 23, 30, 23)

# datetime --> time
# 第一种使用timetuple
> datetime.timetuple(datetime.now())
time.struct_time(tm_year=2021, tm_mon=3, tm_mday=25, tm_hour=23, tm_min=33, tm_sec=9, tm_wday=3, tm_yday=84, tm_isdst=-1)

# 第二种使用Unix时间戳
> time.localtime(datetime.timestamp(datetime.now()))
time.struct_time(tm_year=2021, tm_mon=3, tm_mday=25, tm_hour=23, tm_min=34, tm_sec=6, tm_wday=3, tm_yday=84, tm_isdst=0)

```



# 参考

[1] [Isoformat() Method Of Datetime Class In Python](https://pythontic.com/datetime/datetime/isoformat)

[2] [datetime --- 基本的日期和时间类型](https://docs.python.org/zh-cn/3.7/library/datetime.html)

[3] [time --- 时间的访问和转换](https://docs.python.org/zh-cn/3/library/time.html)

[4] [strftime() 和 strptime()的行为](https://docs.python.org/zh-cn/3.7/library/datetime.html#strftime-and-strptime-behavior)

[5] [Convert a python UTC datetime to a local datetime](https://stackoverflow.com/questions/4563272/convert-a-python-utc-datetime-to-a-local-datetime-using-only-python-standard-lib/13287083#13287083)



