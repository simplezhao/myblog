---
title: git abc tutorial
tags:
  - git
categories: git
description: 最常用且最基本的git用法
abbrlink: de34
date: 2021-03-31 22:19:56
keywords:
---


# git-abc

下面介绍经常用到且最基本的git 命令

## 仓库管理

### 创建本地仓库

* `git init`

```bash
mkdir git-abc
cd git-abc
git init
```

![image-20210331222402099](https://oss.smart-lifestyle.cn/file/qm2vb.png)

### 添加远程分支

* `git remote add <shortname> <url>`

```bash
git remote add origin https://github.com/simplezhao/git-abc.git
```

### 抓取远程代码

* `git fetch <remote>`

```bash
git fetch origin
```

![image-20210331223237496](https://oss.smart-lifestyle.cn/file/48gpy.png)



### 查看某个远程仓库

* `git remote show <remote>`

```bash
git remote show origin
```

![image-20210331223628444](https://oss.smart-lifestyle.cn/file/50dkt.png)



### 同步远程分支

* `git checkout -b <branch> <remote>`

```bash
git checkout -b main origin/main
```

![image-20210331224921389](https://oss.smart-lifestyle.cn/file/vkntn.png)



## 代码管理

### 查看当前分支状态

* `git status`

1. 修改readme.md文件

```bash
git status
```

提示文件被修改

![image-20210331225856222](https://oss.smart-lifestyle.cn/file/jjthz.png)



2. 新增一个readme_en.md文件

```bash
git status
```

提示有未跟踪的文件

![image-20210331230055526](https://oss.smart-lifestyle.cn/file/ttydm.png)



### 添加文件

* `git add .` 添加所有文件

* `git add <file>`添加指定文件

```bash
# 添加所有文件
git add .
```

添加成功后，显示准备提交的文件

![image-20210331230400289](https://oss.smart-lifestyle.cn/file/2m0ge.png)



### 提交文件

* `git commit -a` 在默认编辑器内增加本次提交内容
* `git commit -m <message>`简短提交

```python
git commit -m "update readme file and add new readme for english version"
```

![image-20210331230900239](https://oss.smart-lifestyle.cn/file/xkx4s.png)



### 推送到远程分支

* `git push <shortname> <branch>`

```bash
git push origin main
```

![image-20210331231029318](https://oss.smart-lifestyle.cn/file/136q4.png)



### 查看提交记录

* `git log`

```bash
git log
```

![image-20210331231130554](https://oss.smart-lifestyle.cn/file/wvarp.png)



## 代码合并

多人开发同一个项目，需要共同维护同一个代码库，假如每个人负责独立的模块，不会涉及到代码冲突

某次提交时，提示如下问题

![image-20210331232246543](https://oss.smart-lifestyle.cn/file/d62zw.png)



这是因为远程分支要优先本地的分支，需要先执行git pull，然后在执行git push命令

```bash
git pull origin main
git push origin main
```

执行pull时会提示Merge信息

![image-20210331232418971](https://oss.smart-lifestyle.cn/file/xq0ww.png)



完成并退出编辑后，提示pull信息

![image-20210331232514825](https://oss.smart-lifestyle.cn/file/ct9wp.png)



再次执行git push origin main，显示提交成功

![image-20210331232632195](https://oss.smart-lifestyle.cn/file/lwto3.png)



## 分支管理

### 创建新的分支

* `git branch <branch>`

```bash
git branch hotfix
git checkout hotfix
```

### 分支合并

在上一步操作，因为系统出现bug，紧急创建一个分支，然后在这个分支上进行修复，修改测试验证完毕后，从hotfix分支合并到main分支

```bash
git checkout main
git merge hotfix
```

![image-20210331235235214](https://oss.smart-lifestyle.cn/file/8q60h.png)



### 分支删除

* `git branch -d <branch>`

```bash
git branch -d hotfix
```

### 打标签

* `git tag`

* `git tag -a <version> -m <message>`

通过`git tag`查看仓库内已经存在的标签列表



#### 创建标签

```bash
git tag -a '1.0' -m "first version"
```

也可以使用轻量级标签

`git tag 1.0-a`



#### 共享标签

默认情况下 git push命令不会把标签传送到远程仓库，必须显式地推送标签到服务器，类似于推送代码

`git push <shortname> <tagname>`

如果想一次推送多个标签，可以使用如下命令

`git push <shortname> --tags`

```bash
git push origin 1.0
git push origin --tags
```

推送指定标签

![image-20210401001342160](https://oss.smart-lifestyle.cn/file/o6cju.png)



推送所有标签

![image-20210401001405265](https://oss.smart-lifestyle.cn/file/fyqxl.png)



## git 配置

使用`git config -l `查看当前仓库配置

### 配置全局信息

```bash
git config --global user.name "yourname"
git config --global user.email "youremail"
```



### 配置当前仓库信息

和全局区别在于去掉--global标记

```bash
git config user.name "yourname"
git config user.email "youremail"
```

## 参考

[1] [Book: Pro Git](https://git-scm.com/book/en/v2)