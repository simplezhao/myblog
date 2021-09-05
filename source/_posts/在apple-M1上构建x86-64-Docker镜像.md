---
title: 在apple M1上构建x86_64 Docker镜像
tags:
  - docker
  - Apple M1
categories: Devops
description: 在apple M1设备上使用docker buildx build构建x86_64 的镜像
abbrlink: de6
date: 2021-09-05 13:40:26
keywords:
---



默认通过`docker build`方式构建的镜像时arm64的版本，可以通过`docker inspect IMAGE_ID`查看到

```json
# docker image inspect 0382b9b17bdb
{
  ...
  "Architecture": "arm64",
        "Variant": "v8",
        "Os": "linux",
        "Size": 223036168,
        "VirtualSize": 223036168,
  ...
}
```



这样的镜像时没有办法在Intel x86/64的容器服务中运行的，我们可以选择基于arm版本服务器的容器服务，更好的选择是在M1上编译 x86架构的容器镜像

Docker Desktop for Mac M1中集成了一个buildx的工具，可以方便我们编译各种跨平台的容器镜像

```bash
docker buildx ls

desktop-linux desktop-linux   running linux/arm64, linux/amd64, linux/riscv64, linux/ppc64le, linux/s390x, linux/386, linux/arm/v7, linux/arm/v6
default *       docker
  default       default         running linux/arm64, linux/amd64, linux/riscv64, linux/ppc64le, linux/s390x, linux/386, linux/arm/v7, linux/arm/v6
```

使用docker buildx build来构建X86/64 docker镜像

```bash
docker buildx build --platform=linux/amd64 . -t xxx
```

之后就可按照正常的docker tag、docker push进行操作

更多的命令可以参考官方文档





## 参考

[1] [How to build x86 (and others!) Docker images on an M1 Mac – Jaimyn's Blog](https://blog.jaimyn.dev/how-to-build-multi-architecture-docker-images-on-an-m1-mac/)

[2] [buildx/buildx_build.md at master · docker/buildx (github.com)](https://github.com/docker/buildx/blob/master/docs/reference/buildx_build.md)

[3] [Docker Buildx | Docker Documentation](https://docs.docker.com/buildx/working-with-buildx/)

