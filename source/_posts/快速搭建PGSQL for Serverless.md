---
title: 快速搭建PGSQL for Serverless
tags:
  - serverless
  - scf
date: 2021-06-29 15:12:13
categories: serverless
keywords:
description: 使用腾讯云serverless快速搭建postgresql数据库
---

>  想使用一个独立的数据库服务，使用量和规模不用太大

单独购买数据库实例，太贵了

![image-20210629151750968](https://oss.smart-lifestyle.cn/file/tmasb.png)



发现在数据库服务列表里有一个Serverless版本，看了介绍，需要通过API或者serverless组件方式创建，目前还处于免费的公测阶段，嘿嘿😋，搞起

![image-20210629152124434](https://oss.smart-lifestyle.cn/file/2sg3e.png)



## 操作步骤

操作步骤按照[官网说明](https://cloud.tencent.com/document/product/1154/43004)，很快就能完成

### 安装 serverless cli

```bash
npm install -g serverless
```

### 配置

创建目录，并新建一个serverless.yml文件

```yaml
# serverless.yml
component: postgresql #(必填) 引用 component 的名称，当前用到的是 postgresql 组件
name: serverlessDB # (必填) 该 postgresql 组件创建的实例名称
org: smart-lifestyle # (可选) 用于记录组织信息，默认值为您的腾讯云账户 appid
app: serverlessDB # (可选) 该 sql 应用名称
stage: prod # (可选) 用于区分环境信息，默认值是 dev
inputs:
  region: ap-beijing  # 可选 ap-guangzhou, ap-shanghai, ap-beijing
  zone: ap-beijing-3  # 可选 ap-guangzhou-2, ap-shanghai-2, ap-beijing-3
  dBInstanceName: serverlessdb
  vpcConfig:
    vpcId: 根据实际填写
    subnetId: 根据实际填写
  extranetAccess: false
```

### 部署

执行 `sls deploy`自动完成部署（会显示二维码进行授权），成功之后，会打印显示数据库的连接信息

```yaml
private:
  connectionString: postgresql://xxxx:xxxx@10.0.0.9:5432/tencentdb_hy7vs5lu
  host:             10.0.0.9
  port:             5432
  user:             xxxx
  password:         xxxx
  dbname:           tencentdb_hy7vs5lu
```

且在数据库实例serverless版中可以查看到数据库


![image-20210629151932596](https://oss.smart-lifestyle.cn/file/jenl2.png)

## 连接

创建的数据库没有开启外网连接，而且在实际生产中，也禁止改操作，因此如果想在其他VPC中使用，需要使用到对等连接

![img](https://oss.smart-lifestyle.cn/file/7lno9.png)

具体操作步骤参考[官网说明](https://cloud.tencent.com/document/product/553/18836)

1. 新建有个对等连接

![image-20210629154210941](https://oss.smart-lifestyle.cn/file/9tase.png)

2. 在两端路由表中增加策略，选择对等连接

![image-20210629154322540](https://oss.smart-lifestyle.cn/file/tzmch.png)



完成以上步骤之后可以进行测试

无法ping通，但是可以通过telnet 连接到数据库服务，完成。

![image-20210629154533240](https://oss.smart-lifestyle.cn/file/92fpw.png)

## 参考

[1] [Serverless 应用中心 数据库 PostgreSQL 组件 - Serverless 组件 - 文档中心 - 腾讯云 (tencent.com)](https://cloud.tencent.com/document/product/1154/43004)

[2] [对等连接 同账号创建对等连接通信 - 快速入门 - 文档中心 - 腾讯云 (tencent.com)](https://cloud.tencent.com/document/product/553/18836)
