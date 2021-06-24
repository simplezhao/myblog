---
title: rancher多主机部署
tags:
  - k8s
  - rancher
abbrlink: ee95
date: 2021-05-11 23:17:39
categories: 微服务
keywords:
description: 使用rancher搭建k8s集群
---



> 整理如何使用rancher搭建k8s集群

## 配置说明

在部署中使用的服务器配置如下：

![image-20210623144304343](https://oss.smart-lifestyle.cn/file/svwr6.png)

### 端口要求

参考：[端口要求 | Rancher文档](https://docs.rancher.cn/docs/rancher2/installation/requirements/ports/_index)

其中

| master | 2核4G | 安装rancher server， 集群etcd、control角色 |
| ------ | ----- | ------------------------------------------ |
| node   | 4核8G | 安装k8s 节点，集群worker角色               |

## 安装rancher server

远程连接到master节点

```bash
docker pull rancher/rancher
sudo docker run -d --privileged --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher
```

## 添加集群

登录到Rancher 管理页面

![image-20210511144233895](https://oss.smart-lifestyle.cn/file/orl7g.png)

点击添加集群，选择自定义

![image-20210511144255559](https://oss.smart-lifestyle.cn/file/skrdl.png)



### 配置master节点

1. 选择Etcd、Control角色
2. 填写内网地址
3. 填写节点名称

![image-20210511152659512](https://oss.smart-lifestyle.cn/file/uebzb.png)
