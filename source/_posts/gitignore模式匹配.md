---
title: .gitignore模式匹配
tags: 
  - git 
  - gitignore
categories: git
abbrlink: 4da0
date: 2020-10-26 10:09:58
description: gitignore模式匹配
---
* 匹配模式前使用 `/` 表示根目录
		/filename 表示匹配根目录下的文件filename
* 匹配模式后使用 `/` 代表是目录（不是文件）
		dirname/ 表示匹配的是dirname文件夹
* 匹配模式前加 `！` 表示取反
* `*` 代表任意个字符
		db*.json 表示匹配以db开头的json文件
* `?` 匹配任意一个字符
		db?.json 表示以db开头并且文件名为三个字符的json文件
*  `**` 匹配任意级目录