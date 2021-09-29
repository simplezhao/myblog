---
title: 浅谈python生成短随机数
tags:
  - python
  - 随机数
date: 2021-09-29 12:53:44
categories: python
keywords:
description: 介绍几种生成短随机数的方法
---



有时候我们需要较短的随机数（比如8位），比如在生成url短链接，需要生成随机且唯一的8位字符作为新的url

![image-20210929130158602](https://oss.smart-lifestyle.cn/file/9qfwc.png)

以下介绍几种生成short unique随机数的方法

### random.choices

```python
import string
import random
alphabet = string.ascii_letters + string.digits

"".join(random.choices(alphabet, k=8))
```



### uuid4

```python
from uuid import uuid4
str(uuid4())[:8]
```



### secrets.choice

```python
import secrets
secrets.token_urlsafe(6)
```



### shortuuid

需要先安装第三方库

```bash
pip install shortuuid
```

```python
import shortuuid
su = shortuuid.ShortUUID(alphabet)
su.random(length=8)
```



### NamedTemporaryFile

```python
from tempfile import NamedTemporaryFile

temp = NamedTemporaryFile('r')
temp.name[-8:]
```

### 测评

无论哪种方法，优先考虑这种方法的碰撞概率是多少，以下是对这几种方法在碰撞次数、效率上的测评

#### random.choices

```python
import time
counts = 10000000
alphabet = string.ascii_letters + string.digits
store = set()
collisions = 0
start = time.perf_counter()
for _ in range(counts):
    code = "".join(random.choices(alphabet, k=8))
    if code in store:
        collisions += 1
    else:
        store.add(code)
end = time.perf_counter()

print(f"run counts: {counts}, total time: {end-start}s, each time: {(end-start)/1.0/counts}s, collisions={collisions}")
```

```bash
run counts: 10000000, total time: 12.96840283399979s, each time: 1.296840283399979e-06s, collisions=0
```



#### uuid4

```python
import time
from uuid import uuid4
counts = 10000000
store = set()
collisions = 0
start = time.perf_counter()
for _ in range(counts):
    code = str(uuid4())[:8]
    if code in store:
        collisions += 1
    else:
        store.add(code)
end = time.perf_counter()

print(f"run counts: {counts}, total time: {end-start}s, each time: {(end-start)/1.0/counts}s, collisions={collisions}")
```

```bash
run counts: 10000000, total time: 20.264591582999856s, each time: 2.0264591582999855e-06s, collisions=11668
```



#### secrets.choice

```python
import time
import secrets
counts = 10000000
store = set()
collisions = 0
start = time.perf_counter()
for _ in range(counts):
    code = secrets.token_urlsafe(6)
    if code in store:
        collisions += 1
    else:
        store.add(code)
end = time.perf_counter()

print(f"run counts: {counts}, total time: {end-start}s, each time: {(end-start)/1.0/counts}s, collisions={collisions}")
```

```bash
run counts: 10000000, total time: 9.400313791999906s, each time: 9.400313791999906e-07s, collisions=0
```



#### shortuuid

```python
import time
import shortuuid
import string
counts = 10000000
store = set()
collisions = 0
alphabet = string.ascii_letters + string.digits
start = time.perf_counter()
su = shortuuid.ShortUUID(alphabet)
for _ in range(counts):
    code = su.random(length=8)
    if code in store:
        collisions += 1
    else:
        store.add(code)
end = time.perf_counter()

print(f"run counts: {counts}, total time: {end-start}s, each time: {(end-start)/1.0/counts}s, collisions={collisions}")
```

```bash
run counts: 10000000, total time: 20.793132207999406s, each time: 2.0793132207999407e-06s, collisions=1
```



#### NamedTemporaryFile

```python
import time
from tempfile import NamedTemporaryFile
counts = 10000000
store = set()
collisions = 0
start = time.perf_counter()
for _ in range(counts):
    temp = NamedTemporaryFile('r')
    code = temp.name[-8:]
    if code in store:
        collisions += 1
    else:
        store.add(code)
end = time.perf_counter()

print(f"run counts: {counts}, total time: {end-start}s, each time: {(end-start)/1.0/counts}s, collisions={collisions}")
```

返回时间太长，手动终止

`NamedTemporaryFile`内部方法用的`random.choices`

```python
class _RandomNameSequence:
    """An instance of _RandomNameSequence generates an endless
    sequence of unpredictable strings which can safely be incorporated
    into file names.  Each string is eight characters long.  Multiple
    threads can safely use the same instance at the same time.
    _RandomNameSequence is an iterator."""

    characters = "abcdefghijklmnopqrstuvwxyz0123456789_"

    @property
    def rng(self):
        cur_pid = _os.getpid()
        if cur_pid != getattr(self, '_rng_pid', None):
            self._rng = _Random()
            self._rng_pid = cur_pid
        return self._rng

    def __iter__(self):
        return self

    def __next__(self):
        return ''.join(self.rng.choices(self.characters, k=8))
```

稍微修改如下：

```python
class RandomNameSequence:
    characters = string.ascii_letters + string.digits

    @property
    def rng(self):
        cur_pid = os.getpid()
        print(cur_pid)
        print(getattr(self, '_rng_pid', None))
        if cur_pid != getattr(self, '_rng_pid', None):
            self._rng = Random()
            self._rng_pid = cur_pid
        return self._rng
    
    def __iter__(self):
        return self

    def __next__(self):
        return ''.join(self.rng.choices(self.characters, k=8))
```

测试内容(这里将次数缩小了10倍，由于耗时太长)

```python
import time
counts = 1000000
store = set()
collisions = 0

start = time.perf_counter()

for _ in range(counts):
    code = next(RandomNameSequence())
    if code in store:
        collisions += 1
    else:
        store.add(code)
end = time.perf_counter()

print(f"run counts: {counts}, total time: {end-start}s, each time: {(end-start)/1.0/counts}s, collisions={collisions}")
```

```bash
run counts: 1000000, total time: 21.044718625s, each time: 2.1044718625000002e-05s, collisions=0
```

### 结论

* UUID只包含小写字母和数字，因此它的碰撞次数变高

* NamedTemporaryFile 虽然也用的random.choices，但是使用了迭代器的形式，速度变慢（有待进一步测试）
* 优选random.choices 和 secrets.choice的方式



### 参考

[1] [python - safe enough 8-character short unique random string - Stack Overflow](https://stackoverflow.com/questions/13484726/safe-enough-8-character-short-unique-random-string/55890039#55890039)

[2] [cpython/tempfile.py at b3ab4344d11edd890916be3526b07362182fb172 · python/cpython (github.com)](https://github.com/python/cpython/blob/b3ab4344d11edd890916be3526b07362182fb172/Lib/tempfile.py#L128)

