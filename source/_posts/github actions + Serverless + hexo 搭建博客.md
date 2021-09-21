---
title: github actions + serverless + hexo 搭建博客
tags:
  - hexo
  - serverless
categories: serverless
description: 通过github actions将hexo博客编译后生成静态文件，然后serverless自动获取分支代码部署网站
abbrlink: 186f
date: 2021-09-10 23:26:08
keywords:
---

> 2021年9月21日：新增404页面的展示
>
> 20201年9月10日：基本功能

## 背景

从最开始的ECS（阿里云云服务器）到后来的轻量应用服务器，都是在伴随着新用户打折的优惠上，续费下去，但是最近轻量应用服务器器到期了，在没有大优惠的情况下一年的费用需要600多软妹币，于是开始动手将博客部署到serverless服务上，基本上对于我这种访问量极低的博主来说，费用基本接近于0

![image-20210911093216962](https://oss.smart-lifestyle.cn/file/2z2j0.png)



## serverless的选择

经过对比，选择了腾讯云的[Serverless应用](https://console.cloud.tencent.com/sls)，本身博客的内容都是静态文件，而且腾讯也是将静态网站托管到它的对象存储服务上。虽然也可以直接在对象存储直接托管，但是腾讯云的Serverless应用在部署上提供了代码托管方式，检测到代码分支上的变动，会自动触发构建

![image-20210911095143015](https://oss.smart-lifestyle.cn/file/by7fr.png)

下面开干！

## 搭建过程

### 开通serverless应用，建站

新建应用

![image-20210911100249575](https://oss.smart-lifestyle.cn/file/bifuy.png)

选择快速部署一个Website静态网站

![image-20210911100406421](https://oss.smart-lifestyle.cn/file/1i7ek.png)



开启跨域访问配置（如果你的网站访问第三方资源）

![image-20210911100923927](https://oss.smart-lifestyle.cn/file/4q9e7.png)

其他按照流程点击下一步执行就可以



完成之后，点击访问地址可以看到一个简单的Demo页面。同时也在对象储存中创建了一个跟网址前缀一样的存储桶（这个地址就是对象存储静态网站功能提供的）

![image-20210911101411755](https://oss.smart-lifestyle.cn/file/xsnza.png)

###  绑定域名和证书

这里先不着急去部署我们的网站内容，先去绑定我们自己的域名以及https证书，要不然你的博客只能通过腾讯对象存储提供的一个四级域名去访问

#### 绑定域名

绑定域名之前第一是你必须有自己的域名，各个云厂商基本都提供域名注册服务，这里注册过程不再介绍，第二是域名需要备案，否则也无法绑定成功

![image-20210911102552375](https://oss.smart-lifestyle.cn/file/7tuxq.png)



在对象存储服务中，找到你博客对应的桶

![image-20210911102232485](https://oss.smart-lifestyle.cn/file/5a2pl.png)



点击进去之后，选择域名与传输管理中的自定义源站域名

![image-20210911102349968](https://oss.smart-lifestyle.cn/file/66q6f.png)



在自定义源站域名中添加域名

![image-20210911102725447](https://oss.smart-lifestyle.cn/file/p9c2k.png)



同时，你也需要在你域名服务商那里增加一项CNAME解析

![image-20210911103016315](https://oss.smart-lifestyle.cn/file/fbz1o.png)



这是你通过http://自定义域名可以访问到你的网站

#### 绑定证书

如果你不需要让用户通过https访问你的博客，那可以忽略本节，但是强烈建议开启https，毕竟安全又免费，可以在这个[网站](https://freessl.cn/)上申请免费的证书。拿到证书（公钥和私钥）之后， 点击绑定证书，如果你不是在腾讯云申请的证书，选择自有证书，将公钥、私钥粘贴进去就可以。腾讯云证书直接在证书列表选择就可以

![image-20210911103552585](https://oss.smart-lifestyle.cn/file/yasz4.png)



以上操作完成之后，如果你通过https://你的博客域名，访问还是显示的没有证书，因为默认情况下，还是将https的访问重新定向回了http，我们需要修改储存桶的配置才可以使用https

在静态网站配置中，使能强制HTTPS，默认没有使能

![image-20210911104131084](https://oss.smart-lifestyle.cn/file/xg2x7.png)



此时再通过https://你的博客域名就可以访问到你的博客（可能需要手动清除一下缓存，或者开启一个浏览器隐私窗口）

## 部署hexo博客

大致流程为：使用github管理源码（md文件），并通过github actions将hexo内容编译为静态文件，并推送到新的分支gh_pages，在腾讯云Serverless应用那里，会监测gh_pages，自动构建

![流程](https://oss.smart-lifestyle.cn/file/2vses.png)



### github actions实现



新建一个workflow

![image-20210911120703454](https://oss.smart-lifestyle.cn/file/zb5z8.png)



具体实现如下

```yaml
# This is a basic workflow to help you get started with Actions

name: build and publish gh-pages

# Controls when the workflow will run
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
      - name: setup node version
        uses: actions/setup-node@v2
        with:
          node-version: '12.x'

      # Caching dependencies to speed up workflows. (GitHub will remove any cache entries that have not been accessed in over 7 days.)
      - name: install hexo-cli
        run: npm install -g hexo-cli
      - name: Cache node modules
        uses: actions/cache@v1
        id: cache
        with:
          path: node_modules
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-
      - name: Install Dependencies
        if: steps.cache.outputs.cache-hit != 'true'
        run: |
          npm ci

      - name: generate public tatic files
        run: |
          hexo clean
          hexo g

      - name: copy robots.txt to public directory
        run: cp robots.txt ./public


      - name: push public to gh-pages branch
        env:
          REMOTE_REPO: https://${{ secrets.GIT_TOKEN }}@github.com/${{ secrets.GIT_REPOSITORY }}.git
          REMOTE_BRANCH: gh-pages
        run: |
          cd ./public && git init && git add .
          git config user.name "${{ secrets.GIT_USERNAME }}"
          git config user.email "${{ secrets.GIT_USEREMAIL }}"
          echo -n 'Files to Commit:' && ls -l | wc -l
          git commit -m "github actions build at $(TZ=ANY-8 date +'%Y-%m-%d %H:%M:%S')" > /dev/null 2>&1
          git push --force $REMOTE_REPO master:$REMOTE_BRANCH > /dev/null 2>&1

      - name: deploy
        run: |
          echo build successfully, now you can redeploy your website

      
          

```

steps中包括如下actions

1. actions/checkout@v2 

   获取master分支代码

   ```yaml
   - uses: actions/checkout@v2
   ```

   

2. actions/setup-node@v2

   安装node，版本为12.x

   ```yaml
   - name: setup node version
           uses: actions/setup-node@v2
           with:
             node-version: '12.x'
   ```

   

3. npm install -g hexo-cli

   安装hexo cli

   ```yaml
   - name: install hexo-cli
           run: npm install -g hexo-cli
   ```

   

4. actions/cache@v1 | npm ci

   安装npm依赖包，并使能缓存

   ```yaml
   - name: Cache node modules
           uses: actions/cache@v1
           id: cache
           with:
             path: node_modules
             key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
             restore-keys: |
               ${{ runner.os }}-node-
         - name: Install Dependencies
           if: steps.cache.outputs.cache-hit != 'true'
           run: |
             npm ci
   ```

   

5. hexo clean | hexo g

   清空并部署

   ```yaml
    - name: generate public tatic files
           run: |
             hexo clean
             hexo g
   ```

   

6. cp robots.txt ./public

   将自定义限制爬虫文件复制到public即生成的静态文件的文件夹中

   ```yaml
    - name: copy robots.txt to public directory
           run: cp robots.txt ./public
   ```

   

7. 将代码推送到gh-pages分支

   ```yaml
   - name: push public to gh-pages branch
           env:
             REMOTE_REPO: https://${{ secrets.GIT_TOKEN }}@github.com/${{ secrets.GIT_REPOSITORY }}.git
             REMOTE_BRANCH: gh-pages
           run: |
             cd ./public && git init && git add .
             git config user.name "${{ secrets.GIT_USERNAME }}"
             git config user.email "${{ secrets.GIT_USEREMAIL }}"
             echo -n 'Files to Commit:' && ls -l | wc -l
             git commit -m "github actions build at $(TZ=ANY-8 date +'%Y-%m-%d %H:%M:%S')" > /dev/null 2>&1
             git push --force $REMOTE_REPO master:$REMOTE_BRANCH > /dev/null 2>&1
   
         - name: deploy
           run: |
             echo build successfully, now you can redeploy your website
   ```

   

   这里面涉及到的secret变量可以在，代码仓库secrets中添加

   ![image-20210911121933711](https://oss.smart-lifestyle.cn/file/buzmw.png)

   ​	

   补充：

   `$(TZ=ANY-8 date +'%Y-%m-%d %H:%M:%S')` github action默认时区为UTC时间，这是TZ=ANY-8将时区变为UCT8时间

   ​	



### 在Serverless中配置代码托管

在开发部署中选择代码托管，选择github（需要授权代码源），然后选择blog仓库的gh-pages分支，选择自动触发构建

![image-20210911122509404](https://oss.smart-lifestyle.cn/file/x1n8d.png)



这样我们的工作流就配置完了



## 404页面

1. 在hexo中增加404.html，qq404.html文件，放在source文件夹下面

   ![image-20210922000046337](https://oss.smart-lifestyle.cn/file/j193w.png)

   为什么是两个404文件？qq404里面放的是腾讯公益404页面，但是我想在这个页面上增加一个返回我的主页的按钮，需要通过iframe嵌套这个腾讯404页面，代码如下。

   

   qq404.html

   ```html
   <!DOCTYPE html>
   <html lang="zh-CN">
   <head>
     <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=2">
   </head>
   
   <body>
    
    <script type="text/javascript" src="https://qzonestyle.gtimg.cn/qzone/hybrid/app/404/search_children.js" charset="utf-8" homePageUrl="https://blog.smart-lifestyle.cn/" homePageName="回到我的主页"></script>
   </body>
   </html>
   ```

   404.html

   ```html
   <!DOCTYPE html>
   <html lang="zh-CN">
   <head>
     <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=2">
     <meta name="theme-color" content="#222">
     <meta name="generator" content="Hexo 5.4.0">
     <link rel="apple-touch-icon" sizes="180x180" href="/images/favicon.ico">
     <link rel="icon" type="image/png" sizes="32x32" href="/images/favicon.ico">
     <link rel="icon" type="image/png" sizes="16x16" href="/images/favicon-16x16.ico">
     <link rel="mask-icon" href="/images/favicon.ico" color="#222">
     <style>
       body {margin:0; padding:0;}
       .button {margin: 10px 10px; border-radius: 4px;}
       .button:hover {background-color: #222; color: #eee; }
     </style>
     <title>simplezhao的博客 - 专注于物联网架构和产品</title>
     
   </head>
   
   <body>
    <div>
     <button class="button" onclick="location.href='https://blog.smart-lifestyle.cn'">回到我的主页</button>
    </div>
   <iframe src="./qq404.html" width="100%" height="700" frameborder="0" ></iframe>
   </body>
   </html>
   ```

2. 在hexo _config.yml中修改skip_render，自动从source文件夹下查找，所有不要加source

   ```yaml
   skip_render: [404.html, qq404.html, '*.html', robots.txt]
   ```

3. 在腾讯对象存储，保存网页的桶中找到静态网站的配置，更改错误文档为404.html

   ![image-20210922001015308](https://oss.smart-lifestyle.cn/file/uv767.png)



## 总结

整个流程配置完之后，我们在本地写完博客，提交到github之后，就可以按照流水线自动更新在线博客网站。

![image-20210911124903269](https://oss.smart-lifestyle.cn/file/0jhjn.png)



一点感悟：

腾讯云的Serverless应用整合了自家的各种服务，比如对象存储的网站托管、而代码托管使用的coding来获取授权

![image-20210911003110757](https://oss.smart-lifestyle.cn/file/mub2b.png)



## 参考

[1] [Hexo Action · Actions · GitHub Marketplace](https://github.com/marketplace/actions/hexo-action)

[2] [jekyll-deploy-gh-pages/entrypoint.sh at master · BryanSchuetz/jekyll-deploy-gh-pages (github.com)](https://github.com/BryanSchuetz/jekyll-deploy-gh-pages/blob/master/deploy/entrypoint.sh)

[3] [deploy_jamstack_action/deploy.sh at main · NickSchimek/deploy_jamstack_action (github.com)](https://github.com/NickSchimek/deploy_jamstack_action/blob/main/deploy.sh)

[4] [使用 GitHub Actions 自动部署 Hexo 博客到 GitHub Pages - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/161969042)

[5] [timezone - Why does TZ=UTC-8 produce dates that are UTC+8? - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/104088/why-does-tz-utc-8-produce-dates-that-are-utc8)

[6] [FreeSSL首页 - FreeSSL.cn一个提供免费HTTPS证书申请的网站](https://freessl.cn/)

[7] [Configuration | Hexo](https://hexo.io/docs/configuration#Directory)

[8] [Hexo跳过指定文件的渲染 | Hello Memo (iitii.github.io)](https://iitii.github.io/2019/02/15/1/)

