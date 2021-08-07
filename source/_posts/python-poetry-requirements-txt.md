---
title: python poetry requirements.txt
tags:
  - python
  - poetry
categories: python
description: 使用poetry来管理python包，并介绍如何导出和导入requirements.txt
abbrlink: 143a
date: 2021-06-29 23:33:21
keywords:
---

Poetry 是一个python虚拟环境和依赖管理工具，也可以用来构建打包python包，只需要一个标准化[pyproject.toml](https://www.python.org/dev/peps/pep-0518/)文件，其他依赖管理工具如：virtualenv、pipenv等

requirements.txt是用来描述用了哪些python包，以及版本是什么

这里记录一下这两者之间如何转换



* poetry export requirements.txt

  导出requirements.txt用于容器等环境构建

  ```bahs
  poetry export -f requirements.txt --output requirements-prod.txt --without-hashes
  ```

  

* poerty add from requirements.txt 

  如果现在有requirements.txt，而后续想通过poetry管理包

  ```python
  cat requirements.txt|xargs poetry add
  ```

  

  ### 已知的问题

poetry 导出的requirements.txt里包含了依赖的python最低版本，如果poetry使用的python版本高于requirements.txt使用的python版本，执行安装时将会忽略版本不满足的包

```
markupsafe==2.0.1; python_version >= "3.6"
numpy==1.20.3; python_version >= "3.7" and python_full_version >= "3.7.1"
```

比如上面两个包，如果你的python版本≤3.6，numpy将不会安装，因为它要求python版本大于3.7，但是也不会报错，这个地方很容易忽视。



## 参考

[1] [python - How to import requirements.txt from an existing project using Poetry - Stack Overflow](https://stackoverflow.com/questions/62764148/how-to-import-requirements-txt-from-an-existing-project-using-poetry)

[2] [poetry/cli.md at master · python-poetry/poetry (github.com)](https://github.com/python-poetry/poetry/blob/master/docs/cli.md)



