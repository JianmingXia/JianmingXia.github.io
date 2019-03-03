---
title: HTTP header 介绍
date: 2017/07/29 15:30:00
tags:
  - HTTP
categories: 技术杂集
---

HTTP 协议的请求和响应报文中必定包含HTTP header，作为附加信息，客户端或服务器就能了解到报文具有哪些属性，报文发送端的一些要求等。

<a name="61c7af5c"></a>
## HTTP 报文header

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550631030100-079073df-6c17-4afa-8d94-14829f00a3dc.png#align=left&display=inline&height=361&name=image.png&originHeight=361&originWidth=980&size=206156&status=done&width=980)


<a name="464f14e2"></a>
### HTTP 请求报文
由方法、URI、HTTP版本、HTTP首部字段等部分构成
<!-- more -->

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550631161174-f2486027-a150-4ee3-9271-6d9d7c432282.png#align=left&display=inline&height=305&name=image.png&originHeight=305&originWidth=665&size=136865&status=done&width=665)

以访问曲奇为例：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550631314378-b5c35b0d-90e8-426b-9f10-1d20823438fc.png#align=left&display=inline&height=345&name=image.png&originHeight=345&originWidth=1063&size=34613&status=done&width=1063)

<a name="5b3dafab"></a>
### HTTP 响应报文
由HTTP版本、状态码（数字和原因短语）、HTTP首部字段构成

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550631391014-712f3c63-9140-41d1-b0c3-cee72fbe9ddf.png#align=left&display=inline&height=298&name=image.png&originHeight=298&originWidth=679&size=133936&status=done&width=679)

之前请求的响应报文header：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550631465429-885cc913-a38d-4a02-8ccf-ca41f1ed9320.png#align=left&display=inline&height=326&name=image.png&originHeight=326&originWidth=845&size=32313&status=done&width=845)

在报文众多的字段中，HTTP header包含的信息最为丰富，header会同时存在与请求/响应报文中。

<a name="8d5720d6"></a>
## HTTP header
<a name="0c2c344d"></a>
### header 传递重要信息
使用 header 是为了给浏览器和服务器提供报文主体大小、使用语言、认证信息等内容。

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550631945624-ca0e8ca7-5a4e-43c9-83c4-9e7b2143a5a4.png#align=left&display=inline&height=464&name=image.png&originHeight=464&originWidth=1183&size=337347&status=done&width=1183)

<a name="6f085979"></a>
### HTTP header结构
由字段名和字段值组成，以":"分隔：

```
首部字段名: 字段值
```

<a name="334fe161"></a>
#### 简单的例子
以Content-Type为例：

```
Content-Type: application/x-www-form-urlencoded
```

<a name="b5f5adab"></a>
#### 复杂的例子
字段值对应单个HTTP header可以有多个值：

```
Content-Type: text/html; charset=utf8
```

<a name="d06c4750"></a>
#### HTTP header重复会怎么办
在**规范内尚不明确**：
* 有的浏览器会优先处理第一个
* 有的浏览器会优先处理最后一个

<a name="d39df2d8"></a>
### header 类型
<a name="a456e143"></a>
#### 通用首部字段（General Header Fields）
请求/响应报文都会使用

<a name="0dcbbe4a"></a>
#### 请求首部字段（Request Header Fields）
客户端发送请求时附加的header——补充请求的附加内容、客户端信息、响应内容相关优先级等信息

<a name="cb47be69"></a>
#### 响应首部字段（Response Header Fields）
服务端返回响应报文时使用的header——补充了响应的附加内容

<a name="b7df60cd"></a>
#### 实体首部字段（Entity Header Fields）
针对请求/响应报文中的实体部分使用的首部——补充了资源内容更新时间等与实体有关的信息

<a name="628cc77d"></a>
### HTTP/1.1 首部字段
<a name="ad595596"></a>
#### 通用首部字段

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550634240986-14b855aa-b95b-4d32-b7ee-9f8dc355936c.png#align=left&display=inline&height=569&name=image.png&originHeight=569&originWidth=844&size=126902&status=done&width=844)

<a name="77a3e006"></a>
#### 请求首部字段

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550634291994-9f649ab2-2277-4b5e-aef5-f370997cde0c.png#align=left&display=inline&height=833&name=image.png&originHeight=833&originWidth=719&size=224982&status=done&width=719)

<a name="b7f3c1e8"></a>
#### 响应首部字段

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550634320428-4656becd-1b1d-4d7e-8a35-9a9a894d5fcc.png#align=left&display=inline&height=445&name=image.png&originHeight=445&originWidth=696&size=112791&status=done&width=696)

<a name="56e4abe9"></a>
#### 实体首部字段

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550634333321-26468890-a42e-479d-aead-8a3d231d88e0.png#align=left&display=inline&height=480&name=image.png&originHeight=480&originWidth=648&size=112772&status=done&width=648)
<a name="a2be9933"></a>
### 非 HTTP/1.1 首部字段
除了上述定义的首部字段（见[RFC2616](https://tools.ietf.org/html/rfc2616#section-14)），还有cookie等在其他RFC中定义的首部字段，见[RFC4229](https://tools.ietf.org/html/rfc4229)

<a name="1af8d3aa"></a>
### End-to-end & Hop-by-hop
HTTP 首部字段将定义成缓存代理和非缓存代理的行为：

<a name="4c0f7165"></a>
#### 端到端首部（End-to-end Header）
此类首部会转发给请求/响应对应的最终接受目标，则必须保存在由缓存生成的响应中，另外，规定它必须被转发。

<a name="8c530cb7"></a>
#### 逐跳首部（Hop-by-hop header）
此类首部只对单次转发有效，会因通过缓存/代理而不再转发。在HTTP/1.1及之后版本，如果要使用此首部，需要提供Connection首部。

以下8个都是逐跳首部，其它字段都是端到端首部：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550643769996-53d51772-0e36-470a-abe0-78f440bf687a.png#align=left&display=inline&height=523&name=image.png&originHeight=523&originWidth=387&size=35157&status=done&width=387)

<a name="433531fd"></a>
## 结语
因HTTP版本或扩展规范的变化，可支持的字段可能略有不同，本文主要是HTTP/1.1常用的header。

<a name="447564e4"></a>
## 资料
* [https://tools.ietf.org/html/rfc2616#section-14](https://tools.ietf.org/html/rfc2616#section-14)
* [https://tools.ietf.org/html/rfc4229](https://tools.ietf.org/html/rfc4229)
* 《图解HTTP》
