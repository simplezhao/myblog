---
title: kombu.exceptions.EncodeError XXXX is not JSON serializable
tags:
  - celery
  - python
categories: python
description: celery序列化问题
abbrlink: '3277'
date: 2021-03-31 14:02:05
keywords:
---

本文记录在使用celery时，任务函数参数序列化问题

```python
@celery_app.task(name="create_subscription")
def create_subscription_file(req: SubscriptionRequest):

    req = SubscriptionRequest(**kwargs)
    subscription_count = req.count
    run_times = math.ceil(subscription_count / 200)
    results = {
        "count": 0,
        "validDateTime": None
    }
    ...
    ...
    
    
class SubscriptionRequest(BaseModel):
    bearer_token: str = None
    validDateTime: datetime = None
    validPeriodTime: int = 12
    count: int = 200
    deviceTypeId: int = 2
    subscriptionLevelId: int = 2

    def data_json(self):
        return self.json(exclude={'bearer_token'})

    @validator('validDateTime')
    def create_default_date(cls, v):
        if v is None:
            v = get_default_valid_datetime()
        return v

    def default(self):
        return self.json()
    

```



在实际项目中，准备使用pydantic作为参数校验，因此在传递给celery task时，传递了一个class Instance；接着在任务调度时报错：

`kombu.exceptions.EncodeError: SubscriptionRequest is not JSON serializable`

默认情况下，celery 使用JSON进行序列化数据，因此根本原因在于默认情况下class实例无法被json序列化

```python
> json.dumps(SubscriptionRequest)
TypeError: Object of type SubscriptionRequest is not JSON serializable
 
```

json.dumps有一个参数default，在python docs文档中描述如下

> 当 *default* 被指定时，其应该是一个函数，每当某个对象无法被序列化时它会被调用。它应该返回该对象的一个可以被 JSON 编码的版本或者引发一个 [`TypeError`](https://docs.python.org/zh-cn/3/library/exceptions.html#TypeError)。如果没有被指定，则会直接引发 [`TypeError`](https://docs.python.org/zh-cn/3/library/exceptions.html#TypeError)。

我们可以借助default函数来为class实例创建一个方法实现对其json序列化。

这里没有对类进行改造，而是优化函数的入参

最后改成，先传递参数到task，然后在task内对所有参数进行pydantic 校验

```python
@celery_app.task(name="create_subscription")
def create_subscription_file(*args, **kwargs):
		# 对kv参数使用pydantic校验
    req = SubscriptionRequest(**kwargs)
    subscription_count = req.count
    run_times = math.ceil(subscription_count / 200)
    results = {
        "count": 0,
        "validDateTime": None
    }
    result_queue = Queue()
		...
    ...
```



## 参考

[1] [class serialize](https://stackoverflow.com/questions/10252010/serializing-class-instance-to-json)

[2] [github issue](https://github.com/celery/celery/issues/5922)

