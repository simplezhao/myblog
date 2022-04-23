---
title: 搞了一个浏览器版VS Code Server
tags:
  - code server
  - oauth
date: 2022-04-22 14:54:12
categories: 工具
keywords: code server
description: 在腾讯轻量服务器上搭建vscode在线版code server，并使用github账号授权访问
---

> 工作用的Mac电脑是ARM芯片的，而目前部署的服务大部分还都是amd64架构的，手头上新购了几台轻量服务器，因此搞了一个在线版的vscode来方便开发，实际用过之后确实挺香(*￣︶￣)

**本方案所有实现都是在腾讯云上，如果是其他云厂商，请参考对标服务**

下面介绍如何在轻量服务器上搭建一个基于浏览器的[VS Code Server](https://coder.com/docs/code-server/latest/guide)，其实只是整理了官方文档



## 整体介绍

本着能省就省同时兼顾安全的角度，部署使用了如下资源：

* 腾讯云服务
  * [轻量应用服务器(Ubuntu 20.04)](https://cloud.tencent.com/product/lighthouse)
  * [内网互联](https://cloud.tencent.com/document/product/877/18675)
  * [函数服务](https://cloud.tencent.com/document/product/1154/39271)
  * [API网关](https://cloud.tencent.com/document/product/628)
  * [容器镜像服务（个人版）](https://cloud.tencent.com/document/product/1141/50310)

* 软件部署
  * [https证书](https://freessl.cn/)，提供https访问
  * [Nginx](https://nginx.org/en/)，对Code Server进行反向代理
  * [Code Server](https://github.com/coder/code-server)， 在线版VS Code
  * [OAuth Proxy](https://oauth2-proxy.github.io/oauth2-proxy/)，提供github OAuth认证



整体架构如下图所示

1. 将code server部署在轻量服务器中，使用nginx做反向代理，并启用https加密访问
2. 在云函数中部署OAuth Proxy，并通过API网关对内网开放authorization接口
3. 轻量服务器和云函数在不同的VPC内，为了实现内网访问，通过云联网打通两个VPC
4. 最终用户在浏览器里输入url，然后通过github认证登录到Code Server上

![codeserver架构](https://oss.smart-lifestyle.cn/file/4uyym.png)



最终效果，通过浏览器访问，跟本地VS Code几乎没有任何区别

![image-20220422214335880](https://oss.smart-lifestyle.cn/file/18nzr.png)

## 部署

下面分步骤介绍完整的部署过程，包括下面的步骤

1. 在轻量服务器中部署Code Server和Nginx
2. 在云函数中部署OAuth2 Proxy，并在API网关中配置触发器
3. 配置云联网，打通VPC
4. 调试



### 在轻量服务器中部署Code Server和Nginx

#### 部署Code Server

参考官方通过命令脚本的安装方式，[Install - code-server v4.3.0 docs (coder.com)](https://coder.com/docs/code-server/latest/install#installsh)

```bash
curl -fsSL https://code-server.dev/install.sh | sh
```

> ### Detection reference
>
> - For Debian and Ubuntu, code-server will install the latest deb package.(如果是Ubuntu系统，code server会使用最新的deb包安装)
> - For Fedora, CentOS, RHEL and openSUSE, code-server will install the latest RPM package.

安装完成后，code server的配置文件在`$HOME/.config/code-server/config.yaml`，而插件等都会安装到`$HOME/.local/share/code-server`中



因为后面使用OAuth2 Proxy来做认证，所以在配置文件中将auth改为None，并且修改code server默认端口，且只能本机访问（后面会使用nginx代理转发）

```yaml
bind-addr: 127.0.0.1:32000
auth: none
cert: false
```



启动code server，并设为开机自启动

```bash
sudo systemctl enable --now code-server@$USER

```



#### 部署Nginx

使用apt-get直接安装

```bash
sudo apt-get install nginx
```

##### 增加https配置文件

此处参考：[Usage - code-server v4.3.0 docs (coder.com)](https://coder.com/docs/code-server/latest/guide#using-lets-encrypt-with-nginx)



新增` /etc/nginx/sites-available/code-server.https.conf`文件

使用nginx反向代理code server，此处不是最终完整配置文件

```bash
server {
    listen 443 ssl http2 default_server;
    listen [::]:443;
    server_name code.smart-lifestyle.cn;
    ssl_certificate /etc/nginx/ssl/full_chain.pem;
    ssl_certificate_key /etc/nginx/ssl/private.key;
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout 10m;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;


    location / {
        proxy_pass http://127.0.0.1:32000/;
      	proxy_set_header Host $host;
      	proxy_set_header Upgrade $http_upgrade;
      	proxy_set_header Connection upgrade;
      	proxy_set_header Accept-Encoding gzip;
    }
}
```

其中，

* ssl_certificate /etc/nginx/ssl/full_chain.pem;

* ssl_certificate_key /etc/nginx/ssl/private.key;

分别为网站的公钥和私钥，可以在[FreeSSL首页 - FreeSSL.cn一个提供免费HTTPS证书申请的网站](https://freessl.cn/)申请

然后执行

```bash
# 此处才是加code-server配置加入到nginx配置中
sudo ln -s /etc/nginx/sites-available/code-server.https.conf /etc/nginx/sites-enabled/code-server

# 重新加载配置文件
sudo nginx -s reload
```



此时可以通过https://你的域名，来访问code server，但是因为是直接暴露在公网中，需要增加认证手段，来避免其他人可以直接访问



### 在云函数中部署OAuth2 Proxy，并在API网关中配置触发器

#### 部署OAuth2 Proxy

OAuth2 Proxy提供了多种OAuth2源，这里我选择了GitHub，OAuth2 Proxy使用go 语言开发，虽然云函数提供了go语言的支持，将OAuth2 Proxy最终部署应该还是需要一点点适配工作

![image-20220423142804041](https://oss.smart-lifestyle.cn/file/kaawc.png)

云函数支持以容器镜像的形式部署服务，因此可以直接拉去OAuth2 Proxy的镜像来部署，需要注意的是，云函数默认监听端口为9000，且不能修改，因此需要将OAuth2 Proxy的监听端口也调整为9000

![image-20220423145027200](https://oss.smart-lifestyle.cn/file/hkzvv.png)



##### 流程

如果使用容器镜像部署云函数，镜像来源只能选择腾讯云个人版或者企业版的镜像服务，因此需要将OAuth2 Proxy的镜像先拉到本地，然后推送到腾讯云的镜像服务中（个人版免费）

1. 从OAuth2 Proxy[官方镜像仓库](https://quay.io/repository/oauth2-proxy/oauth2-proxy?tab=tags&tag=latest)拉去镜像

   ```bash
   docker pull quay.io/oauth2-proxy/oauth2-proxy:latest
   ```

2. 在本地打上tag，tag名为在腾讯云镜像服务创建的镜像的镜像地址，然后上传镜像，具体如何使用腾讯云镜像服务，可以参考官方文档：[容器镜像服务 个人版操作指南 - 操作指南 - 文档中心 - 腾讯云 (tencent.com)](https://cloud.tencent.com/document/product/1141/50310)

3. 在github上创建一个application

   在github Developer settings中新建一个OAuth APP

   ![image-20220423204710201](https://oss.smart-lifestyle.cn/file/wme6e.png)

   

   回调地址修改为你实际的回调地址：https://your-code-server-domain/oauth2/callback

   /oauth2/callback 需要在之前的nginx配置中增加一条代理记录，后面会介绍

   ![image-20220423204935842](https://oss.smart-lifestyle.cn/file/mwadc.png)

   创建完应用后，记录你的Client ID、Client Secret后面会使用

4. 云函数创建一个函数服务

   * 选择使用容器镜像
   * 基础配置选择Web函数
   * 函数代码选择上一步选择的镜像

   ![image-20220423202037576](https://oss.smart-lifestyle.cn/file/swtfp.png)

   

   在高级配置中，增加环境变量，来配置github 认证，需要配置的字段有：

   * OAUTH2_PROXY_CLIENT_ID=<Your github app client id>

   * OAUTH2_PROXY_CLIENT_SECRET=<Your github app client secret>

   * OAUTH2_PROXY_COOKIE_SECRET=<随机生成>

   * OAUTH2_PROXY_EMAIL_DOMAINS=*

   * OAUTH2_PROXY_GITHUB_USERS=<your github name> 保证认证后只有你可以访问

   * OAUTH2_PROXY_HTTP_ADDRESS=0.0.0.0:9000 # 这里是修改OAuth2 Proxy监听地址，需要改为9000

   * OAUTH2_PROXY_PROVIDER=github

   * OAUTH2_PROXY_REDIRECT_URL=<your redirect url in app settings>

   详细配置请参考官方：[OAuth Provider Configuration | OAuth2 Proxy (oauth2-proxy.github.io)](https://oauth2-proxy.github.io/oauth2-proxy/docs/configuration/oauth_provider#github-auth-provider)

   下文介绍，在使用环境变量配置OAuth2 Proxy时，在文档的配置字段前加上OAUTH2_PROXY，同时连字符(-)改为下划线（_)

   ![image-20220423210548845](https://oss.smart-lifestyle.cn/file/ldqul.png)

   

   ![image-20220423205906590](https://oss.smart-lifestyle.cn/file/ywaip.png)

   触发器先选择默认，后面需要删除重新建，默认生成的API网关触发配置，支持公网访问，而我只想让这个服务只能内网访问，既保证安全，**有避免公网资源的浪费**

   熟悉腾讯云API网关的话，可以先在API网关创建一个服务，然后再此处选择自定义触发器

   ![image-20220423202303118](https://oss.smart-lifestyle.cn/file/cjgr1.png)

   ![image-20220423203119102](https://oss.smart-lifestyle.cn/file/y49cb.png)

   

#### 部署内网API服务



1. 为云函数创建一个内网访问的API网关触发器

在API网关中，创建一个API网关服务

![image-20220423203527650](https://oss.smart-lifestyle.cn/file/2x3rd.png)



* 访问方式选择内网VPC
* 所属VPC选择跟OAuth2 Proxy服务同一个VPC

![image-20220423203658410](https://oss.smart-lifestyle.cn/file/i84hp.png)

2. 在云函数中，重新配置触发器

   删除之前的触发器，新建触发器

   ![image-20220322230121860](https://oss.smart-lifestyle.cn/file/399ju.png)

   选择自定义触发器，并选择之前创建的API网关服务

   ![image-20220322230157775](https://oss.smart-lifestyle.cn/file/t0m13.png)

   

   配置完之后，可以查看到内网访问路径，在没有配置云联网之前，轻量服务器是无法访问到这个云函数服务

   ![image-20220423213305703](https://oss.smart-lifestyle.cn/file/opgzl.png)

   

### 配置云联网打通VPC

在轻量服务器的管理界面--内网互联中，选择服务器所在的区域，新建个内网互联

![image-20220423212226912](https://oss.smart-lifestyle.cn/file/2h5s8.png)



在内网互联配置中，将轻量服务器VPC和云函数所在VPC关联进去

![image-20220423212342714](https://oss.smart-lifestyle.cn/file/04jmq.png)



由于我的轻量服务器和云函数都在北京区域，因此两个VPC之间是内网访问，因此互相访问是免费的

![image-20220423213011059](https://oss.smart-lifestyle.cn/file/8awk2.png)



登录轻量服务器，访问OAuth2 Proxy url，如果能返回html页面表明，服务成功运行，且可以内网访问

```bash
curl https://service-oll9qkj1-1258647687-in.bj.apigw.tencentcs.com:9003/release/
```



![image-20220423213458827](https://oss.smart-lifestyle.cn/file/2xoco.png)



### 配置nginx，增加auth认证

最后我们在重新修改code-server.https.conf，新增auth部分功能，官方参考：[Configuring for use with the Nginx `auth_request` directive](https://oauth2-proxy.github.io/oauth2-proxy/docs/configuration/overview/#configuring-for-use-with-the-nginx-auth_request-directive)

完整code-server.https.conf文件代码如下

```bash
server {
    listen 443 ssl http2 default_server;
    listen [::]:443;
    server_name _;
    ssl_certificate /etc/nginx/ssl/full_chain.pem;
    ssl_certificate_key /etc/nginx/ssl/private.key;
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout 10m;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    location /oauth2/ {
      	proxy_pass       http://service-oll9qkj1-1258647687-in.bj.apigw.tencentcs.com:8003;
      	proxy_set_header Host                    $proxy_host;
      	proxy_set_header X-Real-IP               $remote_addr;
      	proxy_set_header X-Scheme                $scheme;
      	proxy_set_header X-Auth-Request-Redirect $request_uri;
      	# or, if you are handling multiple domains:
      	# proxy_set_header X-Auth-Request-Redirect $scheme://$host$request_uri;
    }
    location = /oauth2/auth {
      	proxy_pass       http://service-oll9qkj1-1258647687-in.bj.apigw.tencentcs.com:8003;
      	proxy_set_header Host             $proxy_host;
      	proxy_set_header X-Real-IP        $remote_addr;
      	proxy_set_header X-Scheme         $scheme;
      	# nginx auth_request includes headers but not body
      	proxy_set_header Content-Length   "";
     	proxy_pass_request_body           off;
    }

    location / {
     	auth_request /oauth2/auth;
    	error_page 401 = /oauth2/sign_in;

    	# pass information via X-User and X-Email headers to backend,
    	# requires running with --set-xauthrequest flag
    	auth_request_set $user   $upstream_http_x_auth_request_user;
    	auth_request_set $email  $upstream_http_x_auth_request_email;
    	proxy_set_header X-User  $user;
    	proxy_set_header X-Email $email;

    	# if you enabled --pass-access-token, this will pass the token to the backend
    	auth_request_set $token  $upstream_http_x_auth_request_access_token;
    	proxy_set_header X-Access-Token $token;
      	# if you enabled --cookie-refresh, this is needed for it to work with auth_request
    	auth_request_set $auth_cookie $upstream_http_set_cookie;
    	add_header Set-Cookie $auth_cookie;

    	# When using the --set-authorization-header flag, some provider's cookies can exceed the 4kb
    	# limit and so the OAuth2 Proxy splits these into multiple parts.
    	# Nginx normally only copies the first `Set-Cookie` header from the auth_request to the response,
    	# so if your cookies are larger than 4kb, you will need to extract additional cookies manually.
    	auth_request_set $auth_cookie_name_upstream_1 $upstream_cookie_auth_cookie_name_1;

    	# Extract the Cookie attributes from the first Set-Cookie header and append them
    	# to the second part ($upstream_cookie_* variables only contain the raw cookie content)
    	if ($auth_cookie ~* "(; .*)") {
            set $auth_cookie_name_0 $auth_cookie;
            set $auth_cookie_name_1 "auth_cookie_name_1=$auth_cookie_name_upstream_1$1";
    	}

	# Send both Set-Cookie headers now if there was a second part
    	if ($auth_cookie_name_upstream_1) {
    	    add_header Set-Cookie $auth_cookie_name_0;
    	    add_header Set-Cookie $auth_cookie_name_1;
   	}

        proxy_pass http://127.0.0.1:32000/;
      	proxy_set_header Host $host;
      	proxy_set_header Upgrade $http_upgrade;
      	proxy_set_header Connection upgrade;
      	proxy_set_header Accept-Encoding gzip;
    }
}
```

配置完成之后，重启nginx



##  效果

![image-20220423215409071](https://oss.smart-lifestyle.cn/file/uv1hl.png)

![image-20220423215456692](https://oss.smart-lifestyle.cn/file/utszp.png)



![image-20220423215521287](https://oss.smart-lifestyle.cn/file/a6nh5.png)

![image-20220423215614112](https://oss.smart-lifestyle.cn/file/der2y.png)



## 后记

随着自己逐渐向云原生方向发展，腾讯云在这方面做的还是不错目前正在充分的撸厂商的各种免费云产品，而不是像以前只撸虚机，然后搭建各种服务

## 参考

[1] [排查授权请求错误 - GitHub Docs](https://docs.github.com/cn/developers/apps/managing-oauth-apps/troubleshooting-authorization-request-errors)

[2] [Overview | OAuth2 Proxy (oauth2-proxy.github.io)](https://oauth2-proxy.github.io/oauth2-proxy/docs/configuration/overview/#environment-variables)

[3] [Usage - code-server v4.3.0 docs (coder.com)](https://coder.com/docs/code-server/latest/guide)

[4] [Overview | OAuth2 Proxy (oauth2-proxy.github.io)](https://oauth2-proxy.github.io/oauth2-proxy/docs/configuration/overview/#configuring-for-use-with-the-nginx-auth_request-directive)













