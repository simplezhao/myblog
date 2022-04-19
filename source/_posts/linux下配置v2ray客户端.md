---
title: linux下配置v2ray客户端
tags:
  - v2ray
date: 2022-04-19 09:37:48
categories: 工具
keywords: v2ray
description: 如何在linux 下使用v2ray 客户端
---



> 腾讯云服务器访问github出奇的慢，因此搭个梯子来加速访问github

**本篇是客户端教程，不是服务端教程**

**服务器系统：Ubuntu 20.4**

访问[官网下载页面](https://github.com/v2fly/v2ray-core/releases)，选择适配版本的安装包，我的系统是linux 64为系统，选择了v2ray-linux-64.zip

![image-20220419094326600](https://oss.smart-lifestyle.cn/file/115zq.png)

* 下载安装包

```bash
wget https://github.com/v2fly/v2ray-core/releases/download/v4.44.0/v2ray-linux-64.zip
```

服务商本身访问github有问题，可以先下载到本地，然后上传到服务器上



* 解压目录

```bash
uzip v2ray-linux-64.zip -d v2ray-linux
```

* 获取配置文件

可以使用你已经有的config.json 将其上传到服务器上，下面的命令可以测试文件是否正确

```
./v2ray -test config.json
```



我这边在etc目录下创建了一个v2ray-linux文件夹，并将所有相关文件都放到这个文件夹里

![image-20220419095826482](https://oss.smart-lifestyle.cn/file/g05rn.png)



* 配置service文件

创建v2ray.service文件，放到/usr/lib/systemd/system/v2ray.service

```bash
[Unit]
Description=V2Ray Service
Documentation=https://www.v2fly.org/
After=network.target nss-lookup.target

[Service]
ExecStart=/etc/v2ray-linux/v2ray -config /etc/v2ray-linux/config.json
Restart=on-failure
RestartPreventExitStatus=23

[Install]
WantedBy=multi-user.target
```

* 启动service

```bash
service v2ray start
```



* 查看状态

```bash
service v2ray status
● v2ray.service - V2Ray Service
     Loaded: loaded (/lib/systemd/system/v2ray.service; disabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-04-18 21:49:55 CST; 12h ago
       Docs: https://www.v2fly.org/
   Main PID: 3047292 (v2ray)
      Tasks: 10 (limit: 8819)
     Memory: 6.2M
     CGroup: /system.slice/v2ray.service
             └─3047292 /etc/v2ray-linux/v2ray -config /etc/v2ray-linux/config.json

```

* 终端启用代理

```bash
export http_proxy=http://127.0.0.1:1087;export https_proxy=http://127.0.0.1:1087;export ALL_PROXY=socks5://127.0.0.1:1080
```



### 参考

[1] [记录：Linux下 V2Ray 原生客户端配置 - 简书 (jianshu.com)](https://www.jianshu.com/p/dae87e4d7691)

[2] [Releases · v2fly/v2ray-core (github.com)](https://github.com/v2fly/v2ray-core/releases)

