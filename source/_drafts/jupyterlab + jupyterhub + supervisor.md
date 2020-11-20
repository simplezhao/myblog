---
title: jupyterlab + jupyterhub + supervisor
tags:
  - jupyterlab
categories: python
abbrlink: 93be
---
> 记录在jupyterhub 中使用jupyterlab，并且使用supervisor管理进程运行
> supervisor 在非root用户下运行和管理进程

### 安装jupyterhub
> 前提条件
> 1. python 3.5+
> 2. nodejs/npm
```sh

# 安装 http-proxy，并使用淘宝镜像
npm install -g configurable-http-proxy --registry=https://registry.npm.taobao.org
# 安装 jupyterhub，并使用豆瓣镜像
python3 -m pip install jupyterhub -i https://pypi.douban.com/simple/
# 在终端中输入jupyterhub 看是否运行
jupyterhub
# Visit https://localhost:8000 in your browser, and sign in with your unix PAM credentials.
```
### 安装 jupyterlab，并配置到 jupyterhub中
```sh
# 安装 jupyterlab
python3 -m pip install jupyterlab -i https://pypi.douban.com/simple/
# 通过 jupyterhub生成配置文件，并存放在 $HOME/jupyterhub中
jupyterhub --generate-config
# 编辑 jupyterhub_config.py
# 设定你的启动打开目录
c.Spawner.notebook_dir = '/home/simple/develop/jupyter_home'
# 设定以 jupyterlab运行
c.Spawner.default_url = '/lab'
```
### 安装supervisor，并将 jupyterhub管理加入其中
```sh
# 安装supervisor
python -m pip install supervisor
# 生成配置文件，放在/etc/ 中
echo_supervisord_conf > /etc/supervisord.conf 
# 创建配置文件夹
sudo mkdir -p /etc/supervisor/conf.d/
# 编辑 supervisord.conf ，在最后取消注释，加入f iles = /etc/supervisor/conf.d/*.conf
[include]
;files = relative/directory/*.ini
files = /etc/supervisor/conf.d/*.conf
# 修改启动用户为当前用户，如果是root，请忽略
[supervisord]
user=simple
```

```vim
# 在 /etc/supervisor/conf.d/ 新建 jupyterhub.conf， 内容如下
[program:jupyterhub]
command=jupyterhub -f /home/simple/jupyterhub/jupyterhub_config.py
directory=/home/simple/jupyterhub
autostart=true
autorestart=true
startretries=3
exitcodes=0,2
stopsignal=TERM
redirect_stderr=true
stdout_logfile=/home/simple/jupyterhub/log/jupyterhub.log
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=10
user=simple
```
### 运行
```sh
# 在任意位置运行 supervisord，启动 supervisor 主进程
supervisord
# 通过supervisorctl status 查看状态
supervisorctl status
$ jupyterhub                       RUNNING   pid 16161, uptime 0:24:15
```
在浏览器中访问ip:8000，输入系统的设置的用户名密码进行登录
### 配置https
```sh
# 编辑 jupyterhub_config.py， 配置ssl_cert 和 ssl_key
c.JupyterHub.ssl_cert = '/****/*****/ssl_file/full_chain.pem'
c.JupyterHub.ssl_key = '/****/****/ssl_file/private.key'
# 保存后重启supervisor
supervisorctl restart jupyterhub
```
*ssl 证书申请可以参考：https://freessl.cn/*
最终效果：
![效果][image-1]
### Tips
* 启用jupyterlab 插件管理
- Settings --\> Enable Extension Manager 启用
- 在 Extension Manager中搜索manager，选择@jupyter-widgets/jupyterlab-manager，点击安装


### 参考
1. [如何安装supervisor][1]
2. [npm淘宝镜像][2]
3. [豆瓣python源][3]
4. [jupyterhub官网 supervisor参考][4]
5. [配置jupyterhub 使用jupyterlab][5]
6. [jupyterlab安装][6]
7. [jupyterhub安装][7]
8. [ssl 加密 https化][8]

[1]:	https://juejin.im/post/5d80da83e51d45620c1c5471
[2]:	https://developer.aliyun.com/mirror/NPM
[3]:	https://www.jianshu.com/p/c5b7c619dd0b
[4]:	https://github.com/jupyterhub/jupyterhub-tutorial/tree/master/supervisor
[5]:	https://jupyterlab.readthedocs.io/en/latest/user/jupyterhub.html
[6]:	https://jupyter.org/install
[7]:	https://github.com/jupyterhub/jupyterhub
[8]:	https://jupyterhub.readthedocs.io/en/stable/getting-started/security-basics.html

[image-1]:	https://oss.smart-lifestyle.cn/blog/mrwzk.png