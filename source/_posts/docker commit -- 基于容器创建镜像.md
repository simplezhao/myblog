---
title: docker commit -- 基于容器创建镜像
tags:
  - docker
date: 2021-06-30 09:33:50
categories: docker
keywords:
description: 使用docker commit创建一个新的镜像
---

* docker commit 命令
```bash
# 基本用法
docker commit container_id image_name:tag

# 如果想修改某些内容，可以使用--change
docker commit --change "ENV DEBUG=true" container_id image_name:tag
docker commit --change='CMD ["python", "app.py"]' container_id image_name:tag
docker commit --change "EXPOSE 80" container_id image_name:tag
```



## 参考

[1] [docker commit | Docker Documentation](https://docs.docker.com/engine/reference/commandline/commit/)

