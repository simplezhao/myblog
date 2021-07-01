---
title: 函数计算中python依赖包打包
tags:
  - function
  - serverless
date: 2021-06-30 10:09:58
categories: serverless
keywords:
description: 如何在函数计算中为python打包依赖包
---



以下操作使用于华为云的[functiongraph](https://support.huaweicloud.com/functiongraph/)和腾讯云的[serverless](https://cloud.tencent.com/document/product/583/9199)



*！有些包需要编译，因此建议在centos7中进行操作*

### 准备内容

1. Centos7 + python3.6环境 + zip
2. requirements.txt

### 操作步骤

* 安装依赖到本地文件夹

  ```bash
  # 新建文件夹
  mkdir new_dir
  # python依赖包安装到指定目录, -t将所有依赖安装到new_dir目录
  pip install -r requirements-prod.txt -t  new_dir/
  
  ```

  

* 打包成zip包

  ```bash
  # 进入到安装目录文件夹里
  cd new_dir
  # 压缩为zip包(打包的为里面的文件，不是new_dir这个目录)
  zip -rq package_name.1.1.zip *
  ```

* 上传

  根据各个云的要求，将zip包上传，如果zip包太大，一般都要求先上传到对象存储中

  

## 参考

[1] [云函数 部署函数 - 操作指南 - 文档中心 - 腾讯云 (tencent.com)](https://cloud.tencent.com/document/product/583/9702)

[2] [如何制作函数依赖包？_函数工作流 FunctionGraph_常见问题_通用问题_华为云 (huaweicloud.com)](https://support.huaweicloud.com/functiongraph_faq/functiongraph_03_0343.html)

