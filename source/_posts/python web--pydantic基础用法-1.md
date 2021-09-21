---
title: python web -- pydantic基础用法-1
tags:
  - python
  - pydantic
categories: python
keywords: pydantic
description: 使用pydantic对入参校验
abbrlink: e075
date: 2021-03-29 10:09:58
description: pydantic基础用法
---

> 使用pydantic校验输入参数，可以省去后续函数体内的校验，并可以作为一个通用的校验器方便其他方法调用，做的入参校验与入参调用的解耦
>
> 著名的FastAPI框架，也是使用的pydantic作为http请求中参数的校验

### 安装

```bash
# need python3.6+
pip install pydantic -i https://pypi.douban.com/simple/
```

### 基本模型用法

```python
# model.py
from pydantic import BaseModel, validator
# 创建一个模型
class SubscriptionRequest(BaseModel):
    bearer_token: str = None
    validDateTime: str = None
    validPeriodTime: int = 12
    count: int = 200
    deviceTypeId: int = 2
    subscriptionLevelId: int = 2

    # 获得json格式内容，不返还bearer_token
    # def data_json(self):
    #    return self.json(exclude={'bearer_token'})

    # 校验参数
    # @validator('validDateTime')
    # def create_default_date(cls, v):
    #    if v is None:
    #        v = get_default_valid_datetime().strftime("%Y-%m-%dT23:59:59.000Z")
    #    return v
```

因为给模型中每个字段都设置了默认值，在没有传入参数时也不会报错

```python
>> from model import SubscriptionRequest
>> m = SubscriptionRequest()
>> print(m)
bearer_token=None validDateTime=None validPeriodTime=12 count=200 deviceTypeId=2 subscriptionLevelId=2

```

现在将模型文件中一些参数默认值去掉

```python
# model.py
from pydantic import BaseModel, validator
# 创建一个模型
class SubscriptionRequest(BaseModel):
    bearer_token: str
    validDateTime: str
    validPeriodTime: int = 12
    count: int = 200
    deviceTypeId: int = 2
    subscriptionLevelId: int = 2
		...
```

继续执行，将会提示参数缺失

```python
>> from model import SubscriptionRequest
>> m = SubscriptionRequest()
>> print(m)
Traceback (most recent call last):
  File "/Users/simple/workspace/Columbus/admin_tools/serializer.py", line 60, in <module>
    m = SubscriptionRequest()
  File "pydantic/main.py", line 391, in pydantic.main.BaseModel.__init__
pydantic.error_wrappers.ValidationError: 2 validation errors for SubscriptionRequest
bearer_token
  field required (type=value_error.missing)
validDateTime
  field required (type=value_error.missing)
```

正常用法

```python
>> data= {
...        'bearer_token': '1234567==',
...        'validDateTime': None,
...        'validPeriodTime': 12,
...        'count': 12,
...        'deviceTypeId': 2,
...        'subscriptionLevelId': 2
...    }
>> m = SubscriptionRequest(**data)
>> print(m)
bearer_token='1234567==' validDateTime='2022-09-01T23:59:59.000Z' validPeriodTime=12 count=12 deviceTypeId=2 subscriptionLevelId=2
# 输出json格式
>> m.json()
{"bearer_token": "1234567==", "validDateTime": "2022-09-01T23:59:59.000Z", "validPeriodTime": 12, "count": 12, "deviceTypeId": 2, "subscriptionLevelId": 2}
```

在上面返回json结果中，除了bearer_token，其他都是要赋值给body json，因此想在json输出中去除

`bearer_token`

在模型类中增加如下方法

```python
获得json格式内容，不返还bearer_token
     def data_json(self):
        return self.json(exclude={'bearer_token'})
```

同样如果想对输入的参数做具体校验，validDateTime如果为None，就为它赋值一个时间戳

在模型类中增加一个validator

```python
#校验参数
     @validator('validDateTime')
     def create_default_date(cls, v):
        if v is None:
            v = get_default_valid_datetime().strftime("%Y-%m-%dT23:59:59.000Z")
        return v
```

```python
def get_default_valid_datetime():
    """
    默认有效期时间为当前时间+1年半
    :return: 时间戳字符串
    """
    diff = 365 + int(365/2)
    valid_datetime = datetime.utcnow() + timedelta(days=diff)
    # return valid_datetime.strftime("%Y-%m-%dT23:59:59.000Z")
    return valid_datetime
```

需要注意的是validator装饰的为类函数，函数的第一个参数为cls，不是self，另外这个函数还有其他参数

* v 为当前参数的值

* values为当前传递到模型所有参数的字典集合，比如通过values['count'] 来获取count的值

```python
@validator('validDateTime')
     def create_default_date(cls, v, values, **kwargs):
    		pass

```

**另外如果校验的参数没有传递，而是有一个默认值，不会调用校验方法**



### 参考

[1] [pydantic validators](https://pydantic-docs.helpmanual.io/usage/validators/)

[2] [export json](https://pydantic-docs.helpmanual.io/usage/exporting_models/#json_encoders)

[3] [install](https://pydantic-docs.helpmanual.io/install/)

[4] [basic models](https://pydantic-docs.helpmanual.io/usage/models/)

