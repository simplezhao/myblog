---
title: docker部署记录
tags:
  - docker
categories: docker
abbrlink: c5ac
date: 2020-12-26 10:09:58
description: docker 部署操作
---
- [x]() 磁盘初始化
- [x]() 修改系统时区为UTC+8
- [x]() 部署docker环境，修改docker安装位置
- [x]() 优化docker环境，适配china
- [x]() 修改代码配置文件，适配测试环境和生产环境
- [x]() 代码merge到master分支
- [x]() 安装docker镜像
- [x]() 从git获取代码，部署
- [ ]() 配置https
- [ ]() 测试

> 本文记录在Azure 上部署docker应用的过程

## 磁盘初始化
Azure提供的磁盘为系统盘+数据盘，数据盘的大小在新建虚拟机的时候可以选，默认数据盘是未挂载到系统上的，需要手动挂载，挂载数据盘可以参考：
https://docs.azure.cn/zh-cn/virtual-machines/linux/add-disk
```bash
# 通过dh -h命令查看当前系统磁盘的大小和使用情况，目前没有数据盘
Filesystem      Size  Used Avail Use% Mounted on
udev            3.9G     0  3.9G   0% /dev
tmpfs           796M  676K  795M   1% /run
/dev/sda1        29G  1.5G   28G   5% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/sda15      105M  3.6M  101M   4% /boot/efi
/dev/sdb1        16G   45M   15G   1% /mnt
tmpfs           796M     0  796M   0% /run/user/1000
........
# 操作完之后， df -h，最后一行已经挂在到/datadrive上
Filesystem      Size  Used Avail Use% Mounted on
udev            3.9G     0  3.9G   0% /dev
tmpfs           796M  680K  795M   1% /run
/dev/sda1        29G  1.5G   28G   5% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/sda15      105M  3.6M  101M   4% /boot/efi
/dev/sdb1        16G   45M   15G   1% /mnt
tmpfs           796M     0  796M   0% /run/user/1000
/dev/sdc1       196G   61M  186G   1% /datadrive



```

**之后的所有文件操作都是在/datadrive下**
默认datadrive的权限为root权限，修改所有者为当前用户

```bash
sudo chown -R cniabservice:cniabservice /datadrive/
```

## 修改系统时区
Azure提供的linux默认时区为UTC，需要改为中国UTC+8时区
简单操作如下：
```bash
sudo cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime 
```
**!docker 内时区默认也为UTC，跟主机并不同步，需要修改为UTC+8，下面会说明，不在这里介绍**

## 安装docker
### 1. 使用Ubuntu 源进行安装
参考 https://docs.docker.com/install/linux/docker-ce/ubuntu/ 
```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
sudo apt-get update
sudo apt-get install \
apt-transport-https \
ca-certificates \
curl \
gnupg-agent \
software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io
sudo apt-get install docker-compose

```
**apt-get是从国外网站下载，可能速度会非常慢**，可以下载deb离线包：
[https://download.docker.com/linux/ubuntu/dists/bionic/pool/stable/amd64/][11]

> Go to https://download.docker.com/linux/ubuntu/dists/, choose your Ubuntu version, browse to pool/stable/, choose amd64, armhf, arm64, ppc64el, or s390x, and download the .deb file for the Docker Engine - Community version you want to install.
使用dpkg -i \*.deb 安装顺序为：
1. containerd.io
2. docker-ce
3. docker-ce-cli
4. 最后 apt-get install docker-compose


docker默认root权限操作，将当前用户加入root组，免每次操作输入密码

```bash
# If you would like to use Docker as a non-root user, you should now consider adding your user to the “docker” group with something like:
sudo usermod -aG docker your-user
```

### 2. 修改docker镜像存储等存储位置
默认存储位置在系统盘，要将其迁移至数据盘
在控制台输入docker info，查看当前docker 存储位置
![][image-1]
在/etc/docker/下新建daemon.json文件, graph为存储位置

```json
{
  "registry-mirrors": ["https://1x2ypu6h.mirror.aliyuncs.com"],
  "hosts":[
    "tcp://0.0.0.0:2375",
    "unix:///var/run/docker.sock"
  ],
  "graph": "/datadrive/dockerdata"
}
```

重启docker，sudo service docker restart，再次docker info查看配置
![][image-2]
**修改成功！**

## 从代码仓库获取代码
### 1. pull request to Master
遇到问题，没有权限进行合并操作，只有审批功能，修改如下地方
![][image-3]
### 2. 在远程服务器使用git clone获取代码
```bash
# 使用如下用户名密码
# 用户名：jenkins 密码：***************
git clone https://dev.azure.com/ds-rnd/*************
```

## 环境配置
```bash
mkdir redisdb
mkdir mysqldb
mkdir ssl_key
mkdir conf
# 修改nginx.conf文件
mkdir -p static/.well-known
echo "1.0.0" \> version
```

### 1. 构建docker
在工程根目录下执行 docker-compose up 进行首次编译镜像和启动容器

### 2. 初始化数据库
```bash
# 创建abbdrive数据库
python manage.py makemigrations abbdrivemodel
python manage.py migrate
执行
http://g*************.cloudapp.chinacloudapi.cn/**********
```





[11]:	https://download.docker.com/linux/ubuntu/dists/bionic/pool/stable/amd64/ "针对于ubuntu 18.04"

[image-1]:	https://oss.smart-lifestyle.cn/blog/qx062.jpg
[image-2]:	https://oss.smart-lifestyle.cn/blog/7rvdj.jpg
[image-3]:	https://oss.smart-lifestyle.cn/blog/98qaq.jpg