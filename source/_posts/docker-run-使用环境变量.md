---
title: docker run -- 使用环境变量
tags:
  - docker
date: 2021-06-30 09:42:48
categories: docker
keywords:
description: 运行docker时，通过多种方式使用环境变量
---



* docker run

  ```bash
  # 使用 -e, --env
  docker run -e ENV=prod --env IP=1.2.3.4 .....
  # 如果只声明变量，没有值，表示从系统环境变量中获取改值
  export secret=%$#$FG
  docker run -e secret --env IP=1.2.3.4 .....
  # 从文件中获取
  cat .env
  # ENV=prod
  # IP=1.2.3.4
  
  docker run --env-file .env .....
  
  ```

  
