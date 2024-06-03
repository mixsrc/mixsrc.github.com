---
title: RESTful API 实践建议
categories: 开发
toc: false
permalink: /restful-suggest.html
---

## REST

REST (Representational State Transfer，表述性状态转移）是一种架构风格，源于 [Roy Thomas Fielding](http://en.wikipedia.org/wiki/Roy_Fielding) 2000 年在加州大学欧文分校就读博士期间发表的一篇学术论文[Architectural Styles and the Design of Network-based Software Architectures](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)，该论文提出了 REST 的 6 个特点，分别是：

1. 客户端-服务器架构（Client-Server）

   客户端和服务器分离。

2. 无状态通信（Stateless）

   客户端的每个请求都要包含服务器需要的所有信息。

3. 可缓存（Cache）

   响应内容可以在通信链的某处被缓存，以改善网络效率，服务器返回是否可以缓存的标记。

4. 统一接口（Uniform Interface）

    通信链的组件之间通过统一的接口相互通信。

5. 分层系统（Layered System）

   通过限制组件的行为，将架构分解为若干层，每个组件只与相邻的层交互。

6. 支持按需编码（Code-On-Demand）

   支持通过下载并执行一些代码（例如Java Applet、Flash 或 JavaScript）对客户端的功能进行扩展。

## RESTful API 设计

符合 REST 原则的架构称为 RESTful 架构，RESTful 实际上是一种 ROA（Resource-Oriented Architecture，面向资源的架构），其核心是资源，并通过不同的 HTTP 请求方法以及 Header 实现资源状态的改变。

设计 RESTful API 可遵循以下建议：

1. **避免在 URL 中使用动词，用合适的 HTTP 请求方法代替**

   ```text
   - GET：读取（Read）
   - POST：新建（Create）
   - PUT：更新（Update）
   - PATCH：更新（Update），通常是部分更新
   - DELETE：删除（Delete）
   ```

2. **使用名词的复数表示资源**

    使用复数形式因为复数形式可以满足所有类型的需求，并统一 URL 的形式。

3. **请求加上 request_id，对外请求加上调用方标志**

    request_id 可用于定位日志和请求判重，通过调用方标志可以知道哪些客户端调用了接口。

4. **不要返回纯文本，应该始终返回一致的错误信息**

    ```json
    HTTP/1.1 401 Unauthorized

    {
        "status": "Unauthorized",
        "message": "No access token provided.",
        "request_id": "594600f4-7eec-47ca-8012-02e7b89859ce"
    }
    ```

5. **使用适当的 HTTP 状态码**

    HTTP 状态码是一个三位数，分成五个类别，共 100 多个，覆盖了绝大部分可能遇到的情况，客户端只需查看状态码，就可以判断出发生了什么情况，所以服务器应该返回尽可能精确的状态码。

   ```text
    - 1xx: 相关信息
    - 2xx: 操作成功
    - 3xx: 重定向
    - 4xx: 客户端错误
    - 5xx: 服务器错误
   ```

    对于成功的返回，建议使用如下状态码

    ```text
    - GET: 200 OK
    - POST: 201 Created
    - PUT: 200 OK
    - PATCH: 200 OK
    - DELETE: 204 No Content
    ```

6. **避免多级 URL**

   资源需要分页或者多级分类时，比如获取某个作者的某一类文章，不要设计类似`GET /authors/12/categories/2`的 URL，更好的做法是，除了第一级，其他级别都用查询字符串，比如`/authors/12?categories=2`。分页参数可以作为查询字符串，也可以设置在 Header 中。

7. **不要在 URI 中加入版本号**

   版本号可以在HTTP请求头信息的 Accept 字段中进行区分。

8. **提供链接**

   可以在响应中给出 API 的相关文档链接，便于下一步操作。

## Richardson 成熟度模型

[Richardson 成熟度模型](https://martinfowler.com/articles/richardsonMaturityModel.html)用于衡量 RESTful 接口的成熟度，有四个级别。

1. **LEVEL 0**

   使用HTTP作为远程交互的传输系统

2. **LEVEL 1**

    引入资源的概念

3. **LEVEL 2**

    合理使用HTTP动词，并配合响应码来帮助交流

4. **LEVEL 3**

    做到 HATEOAS（Hypertext As The Engine Of Application State），响应信息中包括一系列uri，指出后续的操作。

实际应用中，应该根据具体情况和场景选择合适的成熟度。
