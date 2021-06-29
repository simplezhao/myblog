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



> 整理在腾讯云上如何使用rancher搭建k8s集群，以下操作不适用于生产环境，用于个人测试

## 配置说明

使用到的服务：[CFS](https://cloud.tencent.com/document/product/582/9127#null)，[CVM](https://cloud.tencent.com/document/product/213/16918#null) 等

在部署中使用的CVM服务器配置如下：

![image-20210623144304343](https://oss.smart-lifestyle.cn/file/svwr6.png)

使用的文件存储配置如下：

![image-20210624143319616](https://oss.smart-lifestyle.cn/file/pgg6q.png)



### 端口要求

参考：[端口要求 | Rancher文档](https://docs.rancher.cn/docs/rancher2/installation/requirements/ports/_index)；需要在安全组内开放以上端口

其中

| master | 2核4G | 安装rancher server， 集群etcd、control角色 |
| ------ | ----- | ------------------------------------------ |
| node   | 4核8G | 安装k8s 节点，集群worker角色               |

## 安装rancher server

Ssh远程连接到master节点

```bash
docker pull rancher/rancher
sudo docker run -d --privileged --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher
```

以上执行时，使用自签名证书，如果使用CA签名证书，可以参考如下[命令](https://docs.rancher.cn/docs/rancher2.5/installation/other-installation-methods/single-node-docker/_index/)

```bash
sudo docker run -d  --restart=unless-stopped \
	-p 80:80 -p 443:443 \
	-v <rancher/data path>:/var/lib/rancher \
	-v <full_chain.pem path>:/etc/rancher/ssl/cert.pem \
	-v <private.pem path>:/etc/rancher/ssl/key.pem \
	-v /var/log/rancher/auditlog:/var/log/auditlog \
	--privileged --name rancher-server  rancher/rancher --no-cacerts
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

然后会生成命令，在Master节点主机上执行

![image-20210624131205714](https://oss.smart-lifestyle.cn/file/9aknb.png)

### 配置worker节点

1. 选择Worker角色
2. 填写内网地址
3. 填写节点名称

然后会生成命令，在Node节点主机上执行

![image-20210624131205714](https://oss.smart-lifestyle.cn/file/9aknb.png)



Rancher会监测角色是否齐全，如果不齐全，它会一直等待，只有我们创建的节点包含所有角色，才会去创建K8s服务

### 最后的结果

![image-20210624131709858](https://oss.smart-lifestyle.cn/file/vt5r4.png)



## 部署应用测试

选择已经安装的集群，选择命名空间Default

![image-20210624133826105](https://oss.smart-lifestyle.cn/file/pr139.png)

里面有工作负载、负载均衡等项

![image-20210624133946097](https://oss.smart-lifestyle.cn/file/y9tpe.png)

点击部署服务，填写名称、docker镜像、端口映射等，点击启动

![image-20210624134301378](https://oss.smart-lifestyle.cn/file/5akkx.png)

完成后，显示当前处于Active，对外暴露端口为31621

![image-20210624134430442](https://oss.smart-lifestyle.cn/file/hhjuh.png)

如果想通过公网访问，需要手动在安全组开放端口

![image-20210624134713237](https://oss.smart-lifestyle.cn/file/sqw39.png)

打开浏览器，输入IP/域名+端口号进行访问

![image-20210624135020120](https://oss.smart-lifestyle.cn/file/bpcop.png)

### 通过Ingress访问

在负载均衡页面添加转发规则，点击保存，等待生效

![image-20210624140420462](https://oss.smart-lifestyle.cn/file/dcl00.png)

这样就可以通过id/域名+path的方式访问

![image-20210624140608911](https://oss.smart-lifestyle.cn/file/aqhg5.png)



## 增加持久卷

以下使用腾讯云产品cfs NFS方式挂载PV；首先需要购买腾讯云的文件存储

### 购买文件存储

1. 新建文件系统

2. 购买资源包(个人测试建议选择跟云主机同一区域的)

   ![image-20210624141158272](https://oss.smart-lifestyle.cn/file/hp09i.png)

3. 查看挂点信息中的IP

   ![image-20210624141316684](https://oss.smart-lifestyle.cn/file/4jo9q.png)

### 在Rancher中添加持久卷PV

选择集群，然后选择存储--持久卷，点击添加PV

* 卷插件选择NFS Share
* 服务器填写上一步生成的地址
* 路径填写/；新建的NFS系统中没有其他文件夹，**如果想挂载二级目录，需要先手动创建目录**
* 访问模式选择多主机读写

![image-20210624141553553](https://oss.smart-lifestyle.cn/file/gujzs.png)

### 添加PVC

点击添加PVC，选择上一步新建的持久卷（一个PV只能在一个PVC下面），删除PVC时，会将PV也“删除"😱

![image-20210624141855083](https://oss.smart-lifestyle.cn/file/8iwlx.png)

### 挂载数据卷

新建工作负载或者升级已有负载

选择数据卷--添加卷--使用现有PVC

* 添加正确的容器路径
* 子路径填写相对路径（相对于根路径），这里面写的路径，会自动在NFS文件系统中创建😄

![image-20210624142438484](https://oss.smart-lifestyle.cn/file/2wyug.png)

通过df -h 能看到挂载成功

![image-20210624142958655](https://oss.smart-lifestyle.cn/file/s4f4n.png)

## 证书申请

免费证书可以在这个[网站](https://freessl.cn/)生成，使用比较简单，具体使用参考网上说明

![image-20210624143604995](https://oss.smart-lifestyle.cn/file/p0hg5.png)

## 参考

[1] [安装指南 | Rancher文档](https://docs.rancher.cn/docs/rancher2.5/installation/other-installation-methods/single-node-docker/_index/)

[2] [具体要求 | Rancher文档](https://docs.rancher.cn/docs/rancher2/installation/requirements/_index/)

[3] [手动快速部署 | Rancher文档](https://docs.rancher.cn/docs/rancher2/quick-start-guide/deployment/quickstart-manual-setup/_index)

[4] [如何在国内使用 Rancher | Rancher文档](https://docs.rancher.cn/docs/rancher2/best-practices/use-in-china/_index/)

[5] [证书相关的问题排查 | Rancher文档](https://docs.rancher.cn/docs/rancher2/installation/other-installation-methods/single-node-docker/troubleshooting/_index/)

[6] [单节点安装的高级选项 | Rancher文档](https://docs.rancher.cn/docs/rancher2.5/installation/other-installation-methods/single-node-docker/advanced/_index/)

[7] [变更 Rancher Server IP 或域名 | Rancher文档](https://docs.rancher.cn/docs/rancher2.5/admin-settings/replace-ip-domain/_index/)

[8] [rancher-helpers/cleanup_rancher.sh at master · theAkito/rancher-helpers (github.com)](https://github.com/theAkito/rancher-helpers/blob/master/scripts/cleanup_rancher.sh)

[9] [Rancher 2.x 负载均衡配置及使用_哎_小羊的博客-CSDN博客_rancher 负载均衡](https://blog.csdn.net/aixiaoyang168/article/details/88664263)

[10] [Rancher 2.x 搭建及管理 Kubernetes 集群_哎_小羊的博客-CSDN博客_rancher](https://blog.csdn.net/aixiaoyang168/article/details/88600530#t12)

[11] [文件存储 在容器上使用 CFS - 最佳实践 - 文档中心 - 腾讯云 (tencent.com)](https://cloud.tencent.com/document/product/582/36929)









