---
title: RESTFul API设计规则
tags:
  - api
  - web
categories: web开发
keywords: RESTFul
description: 你需要了解的RESTFul API设计规则
abbrlink: 81a9
date: 2021-11-30 15:21:05
---

> 总结一下restful api在设计时的一些规则

首先应该满足，无论API内部如何实现，任何客户端（python、c#、C++等）都应该能够代用API，

引入版本管理，对于已发布的API，保持已使用的客户端不受影响



## 设计原则

RESTful API设计的主要原则

* REST API围绕资源设计，资源是可由客户端访问的任何类型的对象、数据或服务
* 每一个资源都有一个标识符，即唯一标识该资源的URI
* 通过JSON作为数据交换格式
* REST API使用同一接口（POST新增、GET获取、DELETE删除、PUT修改等）
* REST API使用无状态模型（最好）



### URI基于（复数）名词，且简洁

资源的URI基于名词而不是动词（对资源执行的操作），资源是一个集合，最好用名词的复数

* 用名词不用动词

```http
# 好
https://d-wz.cc/orders
# 不好
https://d-wz.cc/create-order  
```

* 避免设计过长的资源URI

避免请求复杂度避免超过**集合/项目/集合**的资源

比如查询某个客户某个订单的产品有哪些

```http
# 请过过于复杂
https://d-wz.cc/customers/2/orders/100/products
```

拆分为2步

1. 先获取客户订单号列表
2. 再获取订单号下面的产品列表
3. 如果查询某个产品详细信息，可以再加一步

```http
# 第一步 
https://d-wz.cc/customers/2/orders
# 第二步
https://d-wz.cc/orders/100/products
```

### 按照HTTP定义方法实现

1. 根据HTTP方法定义API操作

- **GET** 检索位于指定 URI 处的资源。 响应消息的正文包含所请求资源的详细信息。
- **POST** 在指定的 URI 处创建新资源。 请求消息的正文将提供新资源的详细信息。 请注意，POST 还用于触发不实际创建资源的操作。
- **PUT** 在指定的 URI 处创建或替换资源。 请求消息的正文指定要创建或更新的资源。
- **PATCH** 对资源执行部分更新。 请求正文包含要应用到资源的一组更改。
- **DELETE** 删除位于指定 URI 处的资源



特定请求的影响应取决于资源是集合还是单个子项。 下表总结了大多数 RESTful 实现使用电子商务示例采用的共同约定。 并非所有这些请求都可能会实现 — ，这取决于特定方案。

| **资源**            | **POST**            | **GET**               | **PUT**                           | **DELETE**            |
| :------------------ | :------------------ | :-------------------- | :-------------------------------- | :-------------------- |
| /customers          | 创建新客户          | 检索所有客户          | 批量更新客户                      | 删除所有客户          |
| /customers/1        | 错误                | 检索客户 1 的详细信息 | 如果客户 1 存在，则更新其详细信息 | 删除客户 1            |
| /customers/1/orders | 创建客户 1 的新订单 | 检索客户 1 的所有订单 | 批量更新客户 1 的订单             | 删除客户 1 的所有订单 |

2. 使用HTPP标准规范实现返回码



客户端发送给服务端的content-type，如果服务端不支持，返回415错误

客户端请求包含accept，如果服务端无法匹配，返回406错误

* get方法
  1. 成功返回200， 找不到返回404
* post方法
    	 1. 创建新资源返回201，新资源的URI包含在响应的Location标头中
      2. 如果处理但未创建新资源，返回200
      3. 客户端将无效数据放入到请求，服务端返回400
* delete方法
    	1. 删除成功返回204
     2. 资源不存在返回404



3. 异步操作中的规范

异步操作，是请求发送到服务端后，服务端需要一段时间才能处理完成的，如果需要等待该操作完成后才向客户端发送响应，延迟太大，这时候需要考虑异步操作，首先返回状态码202，表示请求接受处理，但尚未完成，然后在返回内容中，包含一个链接，用户可以去查询请求的当前状态



第一步，先返回202状态法，并同时将后续查询状态URI放到Location中

```http
HTTP/1.1 202 Accepted
Location: /api/status/12345
```

客户端向状态终结点发送GET请求，响应中返回之前请求的状态

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "status":"In progress",
    "link": { "rel":"cancel", "method":"delete", "href":"/api/status/12345" }
}
```

如果异步操作创建了新资源，状态终结点返回状态码303，并包含一个location标头用于提供新资源的URI

```http
HTTP/1.1 303 See Other
Location: /api/orders/12345
```

4. 对于终结点带不带'/'都应该支持

   用一个URI以'/'结尾或者不带'/'服务端都应该支持

   ```bash
   下面应该返回同样内容
   curl https://d-wz.cc/api/cats/
   curl https://d-wz.cc/api/cats
   ```

5. 401和403区别

   * 401 表示用户名或者密码错误，或者无法识别用户身份

   * 403 表示能够识别用户身份，但是他没有权限去操作

### 返回数据需要筛选和分页

前端展示的数据往往是所有数据的一小部分，通过筛选和分页减少对服务端和客户端的压力，在GET请求中使用query方式，比如增加page表示第几页，page_size表示每页的数量，bool值表示状态等等

```http
GET: /books?published=true&page=2&page_size=10
```



### API设计时要考虑版本



尽可能保证释放出去的API，客户在使用后不用修改接口

目前考虑两种方式做版本控制

1. 在URI中增加版本号，对于URI一致的，以前存在的URI按照以前一样继续运行，如果新版增加了新的返回字段，则通过增加版本号的方式管理

```bash
https://adventure-works.com/v2/customers/3
```



2. 查询字符串方式版本控制

```bash
https://adventure-works.com/customers/3?version=2
```

注意：某些较旧的 Web 浏览器和 Web 代理不会缓存在 URI 中包含查询字符串的请求的响应。 这可能会降低使用 web API 的 web 应用程序的性能，并从这类 web 浏览器中运行。



URI 版本控制和查询字符串版本控制方案都是缓存友好的，因为同一 URI/查询字符串组合每次都指向相同的数据。

### API发布要有配套接口文档支持

比如使用swagger管理API文档，在API发布时，同样要发布接口文档，来方便用户了解如何调用你的API



## 参考

[1] [Web API 设计最佳做法 - Azure Architecture Center | Microsoft Docs](https://docs.microsoft.com/zh-cn/azure/architecture/best-practices/api-design)

[2] [API Architecture — Design Best Practices for REST APIs | by Abdul Wahab | Oct, 2021 | Medium](https://abdulrwahab.medium.com/api-architecture-best-practices-for-designing-rest-apis-bf907025f5f)

[3] [异步Request-Reply模式 - Azure Architecture Center | Microsoft Docs](https://docs.microsoft.com/zh-cn/azure/architecture/patterns/async-request-reply)

[4] [The Web API Checklist -- 43 Things To Think About When Designing, Testing, and Releasing your API (fenniak.net)](https://mathieu.fenniak.net/the-api-checklist/)
