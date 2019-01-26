---
title: POST请求常用的 Content-Type
date: 2017/07/26 22:30:00
tags:
  - HTTP
categories: 技术杂集
---

我们都知道HTTP协议和TCP/IP协议族内的其他协议相同——用于客户端与服务端之间的通信。而客户端发出请求至服务端时，一般都会有请求方法、请求URI、协议版本等；当然，还有一个我们不能忽视的是——内容实体，这也是今天的主角。<br />使用POST请求时，待提交的数据是在**请求首部**内容实体**Content-Type**用于表示实体主体内对象的媒体类型；同样的，在服务端返回数据时，同样也有Content-Type这个字段。这里主要说一下常用的：

## application/x-www-form-urlencoded
这应该是使用最广泛的格式了，相信现在很多旧的项目都是这个格式。看一下曲奇的请求：
<!-- more -->

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1548510298490-c48bbaa6-c1ec-4f58-8ae6-1169f631ef8f.png#align=left&display=inline&height=248&linkTarget=_blank&name=image.png&originHeight=248&originWidth=357&size=16391&width=357)

**form表单**（<form enctype="value">application/x-www-form-urlencoded**。<br />在发起请求时，数据会被编码成以 '&' 分隔的键-值对，同时以 '=' 分隔键和值。

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1548510662139-7de6c7d3-ffe7-4123-8d77-a4602450e9a5.png#align=left&display=inline&height=53&linkTarget=_blank&name=image.png&originHeight=53&originWidth=599&size=5539&width=599)

Note：有一点需要注意的是，请求中如果有特殊字符，是会被[重新编码](https://developer.mozilla.org/en-US/docs/Glossary/percent-encoding)的

## multipart/form-data
这个类型在上传文件时是经常遇到的：**multipart/form-data**不对字符编码；当文件上传表单时，必须设置为此值

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1548511396529-a190cbb6-f9ba-4bee-a785-eb055e582827.png#align=left&display=inline&height=239&linkTarget=_blank&name=image.png&originHeight=239&originWidth=583&size=20236&width=583)

在上图可以发现，Content-Type中除了multipart/form-data外，还有一个boundary，义如其名，它的作用就是作为一个分界线（最终也需要以boundary结尾）：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1548511427803-05b38d59-a390-4adb-bb33-7e2fde16821d.png#align=left&display=inline&height=573&linkTarget=_blank&name=image.png&originHeight=573&originWidth=482&size=36192&width=482)

## application/json
考虑一个发数据的场景，比如代发的数据如下：

```
{"a":1,"b":2,"c":{"d":3,"e":4}}
```

此时如果使用application/x-www-form-urlencoded的方式就需要对数据进行额外处理。<br />这个时候使用application/json格式就很方便了，目前这个格式用得越来越多了，用起来也很方便：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1548511785977-13228b65-a906-4020-9dbf-91d57da9f2c1.png#align=left&display=inline&height=89&linkTarget=_blank&name=image.png&originHeight=89&originWidth=351&size=6527&width=351)

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1548512122743-afe99195-0e62-4dbb-8dec-6a27116e1709.png#align=left&display=inline&height=50&linkTarget=_blank&name=image.png&originHeight=50&originWidth=403&size=4455&width=403)

## 资料
* [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type)
* [https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types)
* [https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST)
* [https://developer.mozilla.org/en-US/docs/Glossary/percent-encoding](https://developer.mozilla.org/en-US/docs/Glossary/percent-encoding)
