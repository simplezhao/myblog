---
title: 在code server中使用docker
tags:
  - docker
  - code server
date: 2022-04-27 15:07:42
categories: docker
keywords: docker
description: 配置docker运行模式为rootless，这样non-root用户也可以访问
---

> 上一篇部署了code server，[搞了一个浏览器版VS Code Server - simplezhao的博客 (smart-lifestyle.cn)](https://blog.smart-lifestyle.cn/posts/a6e3/)
>
> 在code server部署完之后，安装docker插件，却无法连接docker，改怎么解决呢

docker默认是以root用户运行的，如果你也是通过root身份登录的主机，应该不会有该问题，下面介绍的非root用户如何解决

**以下操作命令都是在Ubuntu系统下执行**

**以下操作是在Ubuntu系统下执行**

## 安装docker插件

> 第一步是先安装docker，(*￣︶￣)

首先你得在code server中安装docker插件，直接在插件中搜索即可

![image-20220427153653431](https://oss.smart-lifestyle.cn/file/okqw8.png)





点击docker图标，各种提示Failed to connect

<img src="https://oss.smart-lifestyle.cn/file/bckgw.png" alt="image-20220426150451206" style="zoom:67%;" />



## 以rootless模式运行docker

在docker插件上也介绍了，如何在vscode中使用docker

![image-20220427154201414](https://oss.smart-lifestyle.cn/file/vb0bh.png)

打开[链接](https://docs.docker.com/engine/security/rootless/)按照步骤执行

1. 安装`uidmap`

   ```bash
   sudo apt-get install uidmap
   ```

2. 安装`dbus-user-session`

   ```bash
   sudo apt-get install -y dbus-user-session
   ```

3. 将之前docker停止

   ```bash
   sudo systemctl disable --now docker.service docker.socket
   
   sudo rm /var/run/docker.sock
   ```

4. 使用当前non-root用户安装docker

   ```bash
   dockerd-rootless-setuptool.sh install
   
   systemctl --user start docker
   ```

5. 开机启动

   ```bash
   sudo loginctl enable-linger $(whoami)
   ```

6. 将docker host加入到环境变量中

   ```bash
   # 获取当前用户XDG_RUNTIME_DIR
   echo $XDG_RUNTIME_DIR
   
   # 将下面两行加入到.bashrc或者.zshrc中，具体看用的哪种shell
   # run/user/1000 为echo $XDG_RUNTIME_DIR的结果
   export PATH=/usr/bin:$PATH
   export DOCKER_HOST=unix:///run/user/1000/docker.sock
   ```

   ```bash
   source ~/.zshrc
   # 或者
   source ~/.bashrc
   ```

   

7. 运行docker 命令

   ```bash
   # 此时就可以运行docker 命令
   docker run -d -p 6379:6379 redis:latest
   
   docker ps
   
   CONTAINER ID   IMAGE          COMMAND                  CREATED       STATUS       PORTS                                       NAMES
   5e416fb0fa5d   redis:latest   "docker-entrypoint.s…"   7 hours ago   Up 7 hours   0.0.0.0:6379->6379/tcp, :::6379->6379/tcp   gifted_diffie
   ```

   

## 在code server中生效docker操作

前面一路很顺利，但是发现在code server中还是不能用😓

![image-20220427213459661](https://oss.smart-lifestyle.cn/file/etact.png)



而我们在之前的环境变量中已经配置了DOCKER_HOST = `export DOCKER_HOST=unix:///run/user/1000/docker.sock`



经过测试发现，在vscode terminal和 自己电脑上使用iterm通过ssh连接服务器之后的结果竟然不一样

* echo  $XDG_RUNTIME_DIR

​	在iterm上

![image-20220427213948405](https://oss.smart-lifestyle.cn/file/7h9xk.png)

​	

​	在vscode terminal上

![image-20220427214104155](https://oss.smart-lifestyle.cn/file/eyur9.png)



**在vscode terminal ` $XDG_RUNTIME_DIR`为空**，这里没有再深入去看什么原因，而我已在之前的设置中将`$XDG_RUNTIME_DIR`替换为实际的结果`run/user/1000`



* DOCKER_HOST没有生效

在vscode docker的上下文配置中，有两个contexts，一个default（使用当前DOCKER_HOST），另外一个是rootless

![image-20220427214524692](https://oss.smart-lifestyle.cn/file/q3thf.png)

在code server中`$DOCKER_HOST`也是返回为空

![image-20220427214721962](https://oss.smart-lifestyle.cn/file/3ehj3.png)



因为没有连接到正确的docker.sock上，所以一直出错



### 在code server上切换contexts为rootless

在command palette中找到`Docker Contexts: Use`

![image-20220427215039799](https://oss.smart-lifestyle.cn/file/4gjvw.png)

将其调整为rootless

![image-20220426220112259](https://oss.smart-lifestyle.cn/file/6hfz0.png)

然后就可以在code server中使用docker 了

![image-20220427215239438](https://oss.smart-lifestyle.cn/file/uqlgn.png)



## 问题

除了上面提到的问题

$DOCKER_HOST、$XDG_RUNTIME_DIR在 code server terminal中为空

还有一个问题：

在code server terminal中为空无法使用`systemctl --user restart|start|status  docker.service`命令

![image-20220427215515635](https://oss.smart-lifestyle.cn/file/fz5dv.png)

这个问题在[troubleshooting](https://docs.docker.com/engine/security/rootless/#troubleshooting)中也提到了，但我没去解决，如果重启就在iterm中操作

![image-20220427215701871](https://oss.smart-lifestyle.cn/file/8gpp2.png)

这两个问题等后面有时间再去解决吧



## 参考

[1] [Post-installation steps for Linux | Docker Documentation](https://docs.docker.com/engine/install/linux-postinstall/)

[2] [Troubleshooting · microsoft/vscode-docker Wiki (github.com)](https://github.com/microsoft/vscode-docker/wiki/Troubleshooting)

[3] [Run the Docker daemon as a non-root user (Rootless mode) | Docker Documentation](https://docs.docker.com/engine/security/rootless/#troubleshooting)

