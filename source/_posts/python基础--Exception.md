---
title: python基础篇--Exception
tags:
  - python
  - exception
abbrlink: f1e8
date: 2021-03-29 23:30:00
categories: python
keywords:
description: 如何使用python的异常处理
---

以下总结一下python的异常处理

## 基类

### BaseException

> 所有内置异常的基类。 它不应该被用户自定义类直接继承 (这种情况请使用 [`Exception`](https://docs.python.org/zh-cn/3/library/exceptions.html#Exception))。 如果在此类的实例上调用 [`str()`](https://docs.python.org/zh-cn/3/library/stdtypes.html#str)，则会返回实例的参数表示，或者当没有参数时返回空字符串

### Exception

> 所有内置的非系统退出类异常都派生自此类。 所有用户自定义异常也应当派生自此类

### ArithmeticError

> 此基类用于派生针对各种算术类错误而引发的内置异常: [`OverflowError`](https://docs.python.org/zh-cn/3/library/exceptions.html#OverflowError), [`ZeroDivisionError`](https://docs.python.org/zh-cn/3/library/exceptions.html#ZeroDivisionError), [`FloatingPointError`](https://docs.python.org/zh-cn/3/library/exceptions.html#FloatingPointError)

### BufferError

> 当与 [缓冲区](https://docs.python.org/zh-cn/3/c-api/buffer.html#bufferobjects) 相关的操作无法执行时将被引发

### LookupError

> 此基类用于派生当映射或序列所使用的键或索引无效时引发的异常: [`IndexError`](https://docs.python.org/zh-cn/3/library/exceptions.html#IndexError), [`KeyError`](https://docs.python.org/zh-cn/3/library/exceptions.html#KeyError)。 这可以通过 [`codecs.lookup()`](https://docs.python.org/zh-cn/3/library/codecs.html#codecs.lookup) 来直接引发



### 具体异常

```bash
BaseException
 +-- SystemExit
 +-- KeyboardInterrupt
 +-- GeneratorExit
 +-- Exception
      +-- StopIteration
      +-- StopAsyncIteration
      +-- ArithmeticError
      |    +-- FloatingPointError
      |    +-- OverflowError
      |    +-- ZeroDivisionError
      +-- AssertionError
      +-- AttributeError
      +-- BufferError
      +-- EOFError
      +-- ImportError
      |    +-- ModuleNotFoundError
      +-- LookupError
      |    +-- IndexError
      |    +-- KeyError
      +-- MemoryError
      +-- NameError
      |    +-- UnboundLocalError
      +-- OSError
      |    +-- BlockingIOError
      |    +-- ChildProcessError
      |    +-- ConnectionError
      |    |    +-- BrokenPipeError
      |    |    +-- ConnectionAbortedError
      |    |    +-- ConnectionRefusedError
      |    |    +-- ConnectionResetError
      |    +-- FileExistsError
      |    +-- FileNotFoundError
      |    +-- InterruptedError
      |    +-- IsADirectoryError
      |    +-- NotADirectoryError
      |    +-- PermissionError
      |    +-- ProcessLookupError
      |    +-- TimeoutError
      +-- ReferenceError
      +-- RuntimeError
      |    +-- NotImplementedError
      |    +-- RecursionError
      +-- SyntaxError
      |    +-- IndentationError
      |         +-- TabError
      +-- SystemError
      +-- TypeError
      +-- ValueError
      |    +-- UnicodeError
      |         +-- UnicodeDecodeError
      |         +-- UnicodeEncodeError
      |         +-- UnicodeTranslateError
      +-- Warning
           +-- DeprecationWarning
           +-- PendingDeprecationWarning
           +-- RuntimeWarning
           +-- SyntaxWarning
           +-- UserWarning
           +-- FutureWarning
           +-- ImportWarning
           +-- UnicodeWarning
           +-- BytesWarning
           +-- ResourceWarning
```



## 具体用法

典型的异常处理结构

```python
try:
  (your code)
except YourException as e:
  (your code)
else:
  (your code)
finally:
  (your code)

```

* finally 语句 无论是否有异常都会执行，另外如果finally中有return语句，那么始终返回finally中的return语句
* else  语句在没有异常时执行
* except 可以同时填写多个Exception

### 处理多个异常

```python
from math import pow
def test(n1, n2):
    try:
        return pow(n1, n2) / n2
    except (TypeError,ZeroDivisionError) as e:
        print(e)

> test(1, 1)
1.0

# 捕获到 ZeroDivisionError
> test(1, 0)
float division by zero

# 捕获到TypeError
> test('0', 0)
must be real number, not str

# 没有捕获到，程序异常触发ValueError
> test(-1, 0.1)
ValueError: math domain error       
```



### 捕获所有异常

使用`except Exception as e:`来捕获其他所有异常（注：这个将会捕获除了 `SystemExit` 、 `KeyboardInterrupt` 和 `GeneratorExit` 之外的所有异常）

```python
from math import pow
def test(n1, n2):
    try:
        return pow(n1, n2) / n2
    except (TypeError,ZeroDivisionError) as e:
        print(e)
    except Exception as e:
      	print('Exception: ', e)
        
# 如果不确定异常类型，可以写Exception，捕获通用Exception   
> test(-1, 0.1)
Exception:  math domain error
```



### 创建自定义异常

自定义异常，继承于`Exception` 或者其他任何一个已存在的异常类型，假如在处理流，需要涉及到网络流、文件流、内存流，那么可以涉及到如下异常

```python
class StreamError(Exception):
  pass

class NetWorkStreamError(StreamError):
  pass

class FileStreamError(StreamError):
  pass

class MemoryStreamError(StreamError):
  pass


```

需要用到时，可以按照Exception方式，传递参数即可，Exception将所有传递的参数以元组的形式，存在`args`里

```python
try:
    raise ValueError('name error', 'IO error', 'EOF error')
except Exception as e:
    print(e.args)
    
# 输出结果
('name error', 'IO error', 'EOF error')
```

如果要重写`__init__()`方法，需要确保所有参数都给赋值到父类`Exception.__init__()`

```python
class CustomError(Exception):
  def __init__(self, message, status):
    super().__init__(message, status)
    self.msg = message
    self.sta = status
```



### 捕获异常后抛出另外异常

使用`raise Error from e`来形成异常链，可以看到下面的信息：

The above exception was the direct cause of the following exception

```python
try:
    raise ValueError('EOF')
except Exception as e:
    raise TypeError('need str') from e
```

```bash
# 错误信息
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-80-e2122a02c1f3> in <module>
      1 try:
----> 2     raise ValueError('EOF')
      3 except Exception as e:

ValueError: EOF

The above exception was the direct cause of the following exception:

TypeError                                 Traceback (most recent call last)
<ipython-input-80-e2122a02c1f3> in <module>
      2     raise ValueError('EOF')
      3 except Exception as e:
----> 4     raise TypeError('need str') from e
      5     print(e)

TypeError: need str
```

如果没有使用`from e`，认为同时发生了两个Exception

During handling of the above exception, another exception occurred

```python
try:
     x = 3 / 0
except Exception as e:
    raise TypeError('need str')
```

```bash
---------------------------------------------------------------------------
ZeroDivisionError                         Traceback (most recent call last)
<ipython-input-93-7ae0db540006> in <module>
      1 try:
----> 2      x = 3 / 0
      3 except Exception as e:

ZeroDivisionError: division by zero

During handling of the above exception, another exception occurred:

TypeError                                 Traceback (most recent call last)
<ipython-input-93-7ae0db540006> in <module>
      2      x = 3 / 0
      3 except Exception as e:
----> 4     raise TypeError('need str')

TypeError: need str
```

如果要忽略掉异常链，可以使用`raise from None`, 只最后的异常抛出

```python
try:
     x = 3 / 0
except Exception as e:
    raise TypeError('need str') from None
```

```bash
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-94-3858a6e7a744> in <module>
      2      x = 3 / 0
      3 except Exception as e:
----> 4     raise TypeError('need str') from None

TypeError: need str
```



### 重新抛出被捕获的异常

如果想将异常重新被上一级捕获，可以在except中单独加一个`raise`语句，这样可以在异常中记录日志等操作后将异常传播出去

```python
try:
     x = 3 / 0
except Exception as e:
    # other code
    raise 
```

```bash
---------------------------------------------------------------------------
ZeroDivisionError                         Traceback (most recent call last)
<ipython-input-95-1519a0755ccc> in <module>
      1 try:
----> 2      x = 3 / 0
      3 except Exception as e:
      4     raise

ZeroDivisionError: division by zero
```

## 性能

引入异常处理，会带来一定的性能损耗，对于对性能有要求的程序，应该对可预测的结果做逻辑处理，而不是全部用异常处理



```python
import time
def timer(func):
    def wrapper(*args, **kwargs):
       st = time.perf_counter()
       func(*args, **kwargs)
       end = time.perf_counter()
       print(f'using {end - st}s')
        
    return wrapper

@timer
def div1(num):
    try:
        return 3 / num
    except Exception as e:
        return None
   
@timer
def div2(num):
    if num == 0:
        return None
    else:
        return 3 / num
      
> div1(0)
using 4.4405460357666016e-06s

> div2(0)
using 1.341104507446289e-06s
```

也许会有人说，既然用python了，还在乎那点性能，我想说python性能调优也是一种追求

## 参考

[1] [python内置异常](https://docs.python.org/zh-cn/3/library/exceptions.html#bltin-exceptions)

[2] [python cookbook](https://python3-cookbook.readthedocs.io/zh_CN/latest/)



