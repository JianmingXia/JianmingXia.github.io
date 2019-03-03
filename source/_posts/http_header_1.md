---
title: HTTP header 详解
date: 2017/07/29 17:30:00
tags:
  - HTTP
categories: 技术杂集
---

在上文中介绍了 HTTP header 的作用、规则定义及类别。在本文针对每个 header 做更详细介绍：

<a name="ad595596"></a>
## 通用首部字段
<a name="Cache-Control"></a>
### Cache-Control
> 操作缓存的工作机制


Cache-Control 的指令可用于请求及响应时：
<a name="69c70c1a"></a>
#### 缓存请求指令

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550644033993-b52297d5-d768-4afb-8b28-32507cb5587e.png#align=left&display=inline&height=402&name=image.png&originHeight=402&originWidth=704&size=106755&width=704#align=left&display=inline&height=402&originHeight=402&originWidth=704&status=done&width=704)
<!-- more -->

<a name="75215614"></a>
#### 缓存响应指令

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550644060699-2d0c765a-2dea-4cfa-86ac-6a0a476c7b0b.png#align=left&display=inline&height=504&name=image.png&originHeight=504&originWidth=938&size=153469&width=938#align=left&display=inline&height=401&originHeight=504&originWidth=938&status=done&width=746)

<a name="dc502bb1"></a>
#### 是否能缓存的指令
* public：明确表示其他用户也可利用缓存
* private：缓存服务器会对该特定用户提供资源缓存的服务，对于其他用户发送的请求，代理服务器则不会缓存
* no-cache：使用no-cache指令的目的是为了防止从缓存中返回过期的资源
  * 客户端：表示客户端不会接受缓存过的响应，缓存服务器必须将客户端请求转发至源服务器
  * 服务端：缓存服务器不能对资源进行缓存

<a name="b26f1266"></a>
#### 控制可执行缓存的对象的指令
* no-store：当使用no-store时，暗示请求/响应中包含机密信息，规定缓存不能在本地存储请求/响应的任一部分。


<a name="5b043e59"></a>
#### 指定缓存期限和认证的指令
* s-maxage=604800（秒）：功能上与max-age相同；
  * 不同之处是s-maxage只适用于供多位用户使用的公共缓存服务器。对于向同一用户重复返回响应的服务器来说，指令没有作用。
  * 当使用s-maxage时，会**直接忽略Expires 和max-age指令**
* max-age=604800（秒）：
  * 客户端：如果缓存资源的缓存时间比指定时间的数值更小，客户端则使用缓存的资源；如果max-age=0，则缓存服务器转发请求
  * 服务端：缓存服务器不对资源的有效性进行确认，max-age数值代表资源保存为缓存的最长时间
  * Expires 及 max-age 同时设置时的处理：
    * HTTP/1.1：优先处理max-age，Expires会被忽略
    * HTTP/1.0：max-age会被忽略
* min-fresh=60（秒）：要求缓存服务器返回至少还未过指定时间的缓存资源
* max-stale=3600（秒）：指示缓存资源，即使过期也照常接收
  * 如果指令未指定值，客户端始终接收响应
  * 指定具体数值：只要处于max-stale指定的时间内，客户端依然可接收
* only-if-cached：该指令要求缓存服务器不重新加载，也不会确认资源有效性。若发生缓存无响应，则返回504
* must-revalidate：代理会向源服务器再次验证即将返回的缓存目前是否依然有效
  * 若代理无法再次获取有效资源，返回504
  * 会忽略max-stale
* proxy-revalidate：要求所有的缓存在接收客户端的请求返回响应之前，必须再次验证缓存
* no-transform：缓存不能改变实体主体的媒体类型——防止缓存/代理压缩图片等操作

<a name="d04ceaf0"></a>
#### Cache-Control 扩展

* cache-extension token

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550646353168-d3b6e8da-6e4e-45e8-9a47-b8c4cb868969.png#align=left&display=inline&height=345&name=image.png&originHeight=345&originWidth=947&size=133655&width=947#align=left&display=inline&height=272&originHeight=345&originWidth=947&status=done&width=746)

<a name="Connection"></a>
### Connection
Connection 字段有两个作用：

* 控制不再转发给代理的首部字段
* 管理持久连接

<a name="c08ce813"></a>
#### 控制不再转发给代理的首部字段
以Upgrade 为例：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550646534117-11009c64-0be3-4a99-b46a-cfa158091d8c.png#align=left&display=inline&height=377&name=image.png&originHeight=377&originWidth=864&size=243528&width=864#align=left&display=inline&height=326&originHeight=377&originWidth=864&status=done&width=746)

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550646579279-2d4a0d6b-3e2e-46e3-a088-3e48a62fe6a2.png#align=left&display=inline&height=373&name=image.png&originHeight=373&originWidth=1118&size=43887&width=1118#align=left&display=inline&height=249&originHeight=373&originWidth=1118&status=done&width=746)

使用Connection 首部字段，可控制不再转发给代理的首部字段（即Hop-by-hop首部）

<a name="81cf69e3"></a>
#### 管理持久连接
* HTTP/1.1 默认连接都是持久连接，当服务端想明确端开连接时，指定Connection字段为Close
* HTTP/1.1 之前默认连接都是非持久连接。如果希望在旧版本维持持久连接，则将Connection 字段设置为Keep-Alive

<a name="Date"></a>
### Date
表示创建 HTTP 报文的时间

<a name="e3127cc1"></a>
#### 格式
HTTP/1.1 协议使用 RFC1123 规定的格式：<br />
![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551596915547-851f8e2a-ac64-4e3d-ad02-1980873356c1.png#align=left&display=inline&height=115&name=image.png&originHeight=230&originWidth=1320&size=25231&status=done&width=660)

之前则使用 RFC850 规定的格式：


![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551596962150-803a7a54-45dd-4887-8275-cc6dad63a3a5.png#align=left&display=inline&height=98&name=image.png&originHeight=196&originWidth=1272&size=23548&status=done&width=636)

除此之外，还有一个格式。与 C 中 asctime() 输出的格式一致：


![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551597014283-501caf7e-f202-4e3a-bacd-6d0c5bc43333.png#align=left&display=inline&height=101&name=image.png&originHeight=202&originWidth=1176&size=20618&status=done&width=588)

<a name="Pragma"></a>
### Pragma
是 HTTP/1.1 之前版本的历史遗留字段，该首部字段属于通用首部字段，但仅作用于客户端发送的请求中：

有时为了兼容，需要同时设置两个字段：


![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551597233614-dc54561d-33fa-4737-ba59-10884cec81e6.png#align=left&display=inline&height=136&name=image.png&originHeight=272&originWidth=908&size=26577&status=done&width=454)

<a name="Trailer"></a>
### Trailer
事先说明在报文主体字段后记录了哪些首部字段

<a name="f4181202"></a>
#### 在报文主体后记录 Expires


![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551597347732-8c4381d6-b8d0-4d5a-a101-bb1900919bbf.png#align=left&display=inline&height=347&name=image.png&originHeight=694&originWidth=1232&size=97883&status=done&width=616)

<a name="Transfer-Encoding"></a>
### Transfer-Encoding
规定传输报文中使用的编码方式


![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551597470171-2c09ed20-9ad0-4434-ae55-617325c6dee3.png#align=left&display=inline&height=647&name=image.png&originHeight=1294&originWidth=1366&size=216338&status=done&width=683)

HTTP/1.1 传输编码方式仅对分块传输有效

<a name="Upgrade"></a>
### Upgrade
检测 HTTP 协议及其他协议是否可使用更高的版本进行通信

<a name="Via"></a>
### Via
追踪客户端与服务端之间的请求报文与响应报文的传输路径

<a name="Warning"></a>
### Warning
通常表示一些与缓存相关的警告

<a name="77a3e006"></a>
## 请求首部字段
<a name="Accept"></a>
### Accept
通知服务器，客户端可处理的媒体类型及媒体类型的相对优先级


![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551597839089-f633d0a8-d601-4090-9083-6db7c2798098.png#align=left&display=inline&height=91&name=image.png&originHeight=182&originWidth=2176&size=30783&status=done&width=1088)

<a name="Accept-Charset"></a>
### Accept-Charset
通知服务器，客户端可支持的字符集和字符集相对顺序<br />
![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551597896294-d7730dd1-d451-442b-acd6-38ecefde6677.png#align=left&display=inline&height=87&name=image.png&originHeight=174&originWidth=1644&size=30546&status=done&width=822)

<a name="Accept-Encoding"></a>
### Accept-Encoding
通知服务器，客户端可支持的内容编码及内容编码的顺序

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551597984874-4c829168-5dc5-4070-9b4c-e2f5ffccc254.png#align=left&display=inline&height=103&name=image.png&originHeight=206&originWidth=1128&size=21159&status=done&width=564)

<a name="4e695651"></a>
#### 内容编码


![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551598045484-90b8eef1-3b4d-4d2a-9e14-4a9d394ec7ff.png#align=left&display=inline&height=757&name=image.png&originHeight=1514&originWidth=2174&size=422818&status=done&width=1087)

<a name="Accept-Language"></a>
### Accept-Language
告知服务器，客户端支持的自然语言集


![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551598092886-06d4c27b-f996-4821-aa47-cea54e8795a3.png#align=left&display=inline&height=91&name=image.png&originHeight=182&originWidth=1700&size=27768&status=done&width=850)

<a name="Authorization"></a>
### Authorization
告知服务器，客户端的认证信息


![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551598197832-16637a9f-db47-4d77-8c8f-862f69da5f56.png#align=left&display=inline&height=104&name=image.png&originHeight=208&originWidth=1652&size=34549&status=done&width=826)

<a name="Expect"></a>
### Expect
告知服务器，客户端期望出现的某种特定行为


![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551599100790-2846eb8e-b88b-435d-b02f-9d5ab37a2f18.png#align=left&display=inline&height=96&name=image.png&originHeight=192&originWidth=804&size=14559&status=done&width=402)

<a name="From"></a>
### From
告知服务器，使用用户代理的用户的电子邮件地址

<a name="Host"></a>
### Host
告知服务器，请求的资源所处的互联网主机号+端口号

<a name="If-Match"></a>
### If-Match
形如 If-xxx 的请求首部字段，皆被称为条件请求；<br />告知服务器，匹配资源所用的实体标记（ETag）

<a name="If-Modified-Since"></a>
### If-Modified-Since
服务器只在所请求的资源在给定的日期时间之后对内容进行过修改的情况下才会将资源返回<br />
![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551599761302-da9acd63-ba0a-4f2e-9791-372b2dc31022.png#align=left&display=inline&height=91&name=image.png&originHeight=182&originWidth=1742&size=29996&status=done&width=871)

<a name="If-None-Cache"></a>
### If-None-Cache
当且仅当服务器上没有任何资源的 ETag 属性值与值相匹配时，服务器端会才返回所请求的资源

<a name="If-Range"></a>
### If-Range

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551600268568-fc7f15e1-a636-4051-9228-0a194cab379e.png#align=left&display=inline&height=109&name=image.png&originHeight=218&originWidth=1384&size=84453&status=done&width=692)

<a name="If-Unmodified-Since"></a>
### If-Unmodified-Since


![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551600333280-aab28da7-58c0-4272-82f0-d8825d2515f5.png#align=left&display=inline&height=107&name=image.png&originHeight=214&originWidth=1388&size=86192&status=done&width=694)

<a name="Max-Forwards"></a>
### Max-Forwards
限定可经过转发服务器的个数<br />
![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551600417185-35fed7f5-d61c-4ada-8090-ce58211cb72f.png#align=left&display=inline&height=99&name=image.png&originHeight=198&originWidth=706&size=15674&status=done&width=353)

<a name="Proxy-Authorization"></a>
### Proxy-Authorization

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551600480325-8816f17e-1a2c-4869-b3df-08600203154b.png#align=left&display=inline&height=91&name=image.png&originHeight=182&originWidth=1392&size=64951&status=done&width=696)

<a name="Range"></a>
### Range

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551600532042-d8fe7138-beec-45b6-b94b-a777644c684e.png#align=left&display=inline&height=143&name=image.png&originHeight=286&originWidth=1384&size=106970&status=done&width=692)

<a name="Referer"></a>
### Referer

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551600659630-da063b56-7aa9-4265-a471-69630b283a00.png#align=left&display=inline&height=167&name=image.png&originHeight=334&originWidth=1380&size=96774&status=done&width=690)

<a name="TE"></a>
### TE

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551600792413-c4e56cd7-6b85-4bdd-b1cf-a16d5e2e8281.png#align=left&display=inline&height=213&name=image.png&originHeight=426&originWidth=1392&size=129177&status=done&width=696)

<a name="User-Agent"></a>
### User-Agent
表示客户端的类型

<a name="b7f3c1e8"></a>
## 响应首部字段
<a name="Accept-Ranges"></a>
### Accept-Ranges

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551601004123-9f474911-9298-47f0-b048-7c6141b0415d.png#align=left&display=inline&height=168&name=image.png&originHeight=336&originWidth=1424&size=70632&status=done&width=712)

<a name="Age"></a>
### Age

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551601056540-e7b984d3-936d-4eb2-abf5-f9b2b81117c9.png#align=left&display=inline&height=106&name=image.png&originHeight=212&originWidth=1374&size=69077&status=done&width=687)

<a name="ETag"></a>
### ETag

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551601107858-6b440252-1912-4b04-b07c-0426d7f61d5e.png#align=left&display=inline&height=173&name=image.png&originHeight=346&originWidth=1376&size=113145&status=done&width=688)

<a name="Location"></a>
### Location

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551601222168-5744e8dd-c794-41ea-824f-199fc9af34f5.png#align=left&display=inline&height=507&name=image.png&originHeight=1014&originWidth=1418&size=279969&status=done&width=709)

<a name="Proxy-Authenticate"></a>
### Proxy-Authenticate

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551601324281-946bc1d5-c37c-4a8c-bcd9-7505e37d3b14.png#align=left&display=inline&height=169&name=image.png&originHeight=338&originWidth=1396&size=79865&status=done&width=698)

<a name="Retry-After"></a>
### Retry-After

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551601388082-23041a9f-b621-4d20-bbf6-1e9ff403527a.png#align=left&display=inline&height=207&name=image.png&originHeight=414&originWidth=1422&size=108462&status=done&width=711)

<a name="Server"></a>
### Server

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551601440762-3fc7493e-7732-42ce-b3b7-2e14eaa9e3f8.png#align=left&display=inline&height=124&name=image.png&originHeight=248&originWidth=1380&size=63901&status=done&width=690)

<a name="Vary"></a>
### Vary

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551601493514-0c4ad47f-750d-419e-9619-87fbc89653df.png#align=left&display=inline&height=165&name=image.png&originHeight=330&originWidth=1386&size=108008&status=done&width=693)

<a name="WWW-Authenticate"></a>
### WWW-Authenticate

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551601589010-0d7dc223-32fd-4483-b05c-4a5799828a5d.png#align=left&display=inline&height=102&name=image.png&originHeight=204&originWidth=1308&size=46054&status=done&width=654)

<a name="56e4abe9"></a>
## 实体首部字段
<a name="Allow"></a>
### Allow

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551601740438-d0aa8cc0-ba1c-4917-958c-fef90a6ab877.png#align=left&display=inline&height=140&name=image.png&originHeight=280&originWidth=1348&size=80697&status=done&width=674)

<a name="Content-Encoding"></a>
### Content-Encoding

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551601808341-739fdec6-4022-4225-aa08-a6b54d8aa986.png#align=left&display=inline&height=185&name=image.png&originHeight=370&originWidth=1388&size=130108&status=done&width=694)

<a name="Content-Language"></a>
### Content-Language

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551601877359-d87c922e-0f25-445f-b0cf-1a9892ea0982.png#align=left&display=inline&height=89&name=image.png&originHeight=178&originWidth=2130&size=85013&status=done&width=1065)

<a name="Content-Length"></a>
### Content-Length

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551601926978-cc4c1174-ebaa-4ad1-a41a-b96be56b9050.png#align=left&display=inline&height=62&name=image.png&originHeight=124&originWidth=1394&size=34647&status=done&width=697)

<a name="Content-Location"></a>
### Content-Location
![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551602071972-ccf23b91-db08-4c23-a173-ef19bf4e691e.png#align=left&display=inline&height=191&name=image.png&originHeight=382&originWidth=1436&size=117876&status=done&width=718)

<a name="Content-MD5"></a>
### Content-MD5

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551602145103-e05f5f42-e897-406d-a419-6877297f3979.png#align=left&display=inline&height=238&name=image.png&originHeight=476&originWidth=2176&size=263214&status=done&width=1088)

<a name="Content-Range"></a>
### Content-Range

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551602188174-e8fcbc40-2154-4c8e-a483-ddb7f61f9284.png#align=left&display=inline&height=53&name=image.png&originHeight=106&originWidth=1302&size=23134&status=done&width=651)

<a name="Content-Type"></a>
### Content-Type

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551602239701-63ad8989-8a02-4355-94f8-48031aeab276.png#align=left&display=inline&height=193&name=image.png&originHeight=386&originWidth=1398&size=106739&status=done&width=699)

<a name="Expires"></a>
### Expires

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551602442868-c3e545b4-fc7b-4437-846d-cc7b70679e90.png#align=left&display=inline&height=175&name=image.png&originHeight=350&originWidth=1408&size=69443&status=done&width=704)

<a name="Last-Modified"></a>
### Last-Modified

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551602505144-45f4139c-284f-4bd8-8f89-369c8bfd9b19.png#align=left&display=inline&height=116&name=image.png&originHeight=232&originWidth=1372&size=88246&status=done&width=686)

<a name="fee7df1a"></a>
## Cookie 首部字段

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551602735738-351d8a70-5d69-4ffe-96ed-eb2d05e999ed.png#align=left&display=inline&height=177&name=image.png&originHeight=354&originWidth=1862&size=115080&status=done&width=931)

<a name="Set-Cookie"></a>
### Set-Cookie

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551602796782-3b5a705d-96d5-417b-9259-81fdbbfa20dc.png#align=left&display=inline&height=89&name=image.png&originHeight=178&originWidth=2174&size=35215&status=done&width=1087)<br />


![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551602833363-7b4db6c9-719c-48e3-8746-3733d76bffe9.png#align=left&display=inline&height=436&name=image.png&originHeight=872&originWidth=2202&size=367366&status=done&width=1101)

<a name="Cookie"></a>
### Cookie

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1551602883290-7c645233-4473-4e04-b2e1-fd64ff647449.png#align=left&display=inline&height=120&name=image.png&originHeight=240&originWidth=1382&size=60687&status=done&width=691)

<a name="447564e4"></a>
## 资料

* [https://tools.ietf.org/html/rfc2616#section-14](https://tools.ietf.org/html/rfc2616#section-14)
* [https://tools.ietf.org/html/rfc4229](https://tools.ietf.org/html/rfc4229)
* [https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers)
* 《图解HTTP》

