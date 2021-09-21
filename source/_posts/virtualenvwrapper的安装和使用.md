---
title: virtualenvwrapper的安装和使用
tags:
  - virtualenv
categories: python
abbrlink: 9fc5
date: 2020-04-26 10:09:58
description: virtualenvwrapper的使用
---


> virtualenvwrapper 是python虚拟运行环境的管理工具，在多个项目时，防止包版本不同造成冲突等麻烦
> 也可以在共享项目时，不引入不必要的python包
​
### 安装
```sh
python -m pip install virtualenvwrapper
# 执行
source /usr/local/bin/virtualenvwrapper.sh
​
```
### 配置
安装完 virtualenvwrapper，默认没有生效，需要在终端中执行 `source /usr/local/bin/virtualenvwrapper.sh` 才能生效，而且每次运行前都需要执行该命令，因此需要将其配置到 终端的.\* shrc 文件中
```sh
# 根据终端类型，修改 对应的.zshrc .bashrc文件，加入下面这句话
source /usr/local/bin/virtualenvwrapper.sh
# 应用新的配置文件，以zshrc为例
source ~/.zshrc
```

### 使用
```sh
# 查看当前系统虚拟环境列表
lsvirtualenv
# 创建新的虚拟环境, [name] 为虚拟环境名字
mkvirtualenv [name]
# 删除虚拟环境
rmvirtualenv [name]
# 应用某个虚拟环境
workon [name]
```

### 参考
1. [virtualenvwrapper.readthedocs.io][1]
	 ​

[1]:	https://virtualenvwrapper.readthedocs.io/en/latest/install.html