---
title: Paw文件操作
tags:
  - paw
date: 2021-06-29 17:02:28
categories: 工具
keywords:
description: 在使用paw中，如何使用文件请求和下载文件
---

> 整理paw中文件的请求和下载

> Paw is a full-featured HTTP client that lets you test and describe the APIs you build or consume. It has a beautiful native macOS interface to compose requests, inspect server responses, generate client code and export API definitions.



## 上传文件请求

### 通过表单上传

* body类型选择Multipart
* Content-Type: multipart/form-data或者application/x-www-form-urlencoded

* value选择 file content

![image-20210629170831026](https://oss.smart-lifestyle.cn/file/mshyx.png)

然后上传文件

![image-20210629170927905](https://oss.smart-lifestyle.cn/file/vj768.png)



### 通过body

## 下载响应返回文件

在file -- 导出响应---响应body即可

![image-20210629170706183](https://oss.smart-lifestyle.cn/file/8majz.png)



## 参考

[1] [paw app - How do download response binary? - Stack Overflow](https://stackoverflow.com/questions/40680954/how-do-download-response-binary)

[2] [Documentation | Paw](https://paw.cloud/docs/getting-started/set-request-body#Set_Multipart_body)
