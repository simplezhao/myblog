---
title: github actions初试牛刀
tags:
  - github actions
  - hexo
categories: 部署
description: 通过github actions部署hexo博客
abbrlink: 9f77
date: 2021-03-26 10:09:58
description: github action初识
---

> 最近将hex博客部署由手动执行，改为通过github actions自动部署，这里做下记录

### 自动部署原理

使用github actions功能，将hexo生成的静态文件在每次提交代码时，通过scp将文件上传到服务器



### 使用步骤

![image-20210326140634778](https://oss.smart-lifestyle.cn/file/jl6ye.png)



在git仓库新增actions，内容如下：

```yaml
# This is a basic workflow to help you get started with Actions

name: CI

# Controls when the action will run. 
on:
  # Triggers the workflow on push or pull request events but only for the master branch
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2

      # Runs a single command using the runners shell
      - name: Run a one-line script
        run: echo Hello, world!

      # Runs a set of commands using the runners shell
      - name: Run a multi-line script
        run: |
          echo Add other actions to build,
          echo test, and deploy your project.
          ls
          pwd
      - name: copy file via ssh password
        uses: appleboy/scp-action@master
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          password: ${{ secrets.PASSWORD }}
          port: ${{ secrets.PORT }}
          source: "public/*,!public/robots.txt"
          target: ${{ secrets.TARGET_DIR }}

```

说明

* on 触发条件
  * push 推送目标仓库
  * pull_request 合并分支仓库

* steps 步骤
  * uses: actions/checkout@v2 获取代码
  * uses: appleboy/scp-action@master 使用第三方库，通过scp 传输文件
    * host: ${{ secrets.HOST }}
    * username: ${{ secrets.USERNAME }}
    *  password: ${{ secrets.PASSWORD }}
    * port: ${{ secrets.PORT }}
    *  source: "public/*,!public/robots.txt"
    *  target: ${{ secrets.TARGET_DIR }}
    
    

上面的secrets信息在github上进行配置



![image-20210326141554880](https://oss.smart-lifestyle.cn/file/oydi5.png)



每当push代码到仓库master分支时，会进行执行actions，进行博客部署

![image-20210326141938875](https://oss.smart-lifestyle.cn/file/jbae5.png)



### 不足

1. 目前没有通过github actions生成静态文件，也就是推送之前需要手动执行hexo g，这个在进一步研究之后再尝试
2. github 本身可以添加Deploy keys，为了安全，scp应该避免使用密码进行操作，也在研究明白之后再尝试
3. 由于问题1，尤为hexo g会重新生成静态文件，每次push的时候会有大量带提交文件。。。

### 参考

[1] [GitHub Actions 入门教程](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)

[2] [🚀 SCP for GitHub Actions](https://github.com/appleboy/scp-action)

[3] [How to automate a deploy in a VPS with GitHub actions via SSH](https://dev.to/miangame/how-to-automate-a-deploy-in-a-vps-with-github-actions-via-ssh-101e)

