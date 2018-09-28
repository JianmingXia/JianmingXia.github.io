---
title: Kong——Proxy参考
date: 2018/09/28 20:30:00
tags:
  - API Gateway
  - Kong
categories: 
  - [微服务]
  - [Kong]
---

> 通过详细解释Kong的路由功能和内部工作原理来介绍Kong的代理功能

Kong公开几个接口，可以调整两个配置属性：
* proxy\_listen：定义了一个地址/端口列表——Kong将接受来自客户端的流量并代理上游服务
* admin\_listen：同样定义了一个地址/端口列表——但这些仅限于管理员访问

## 术语
* client：指下游客户端向Kong的代理端口发出请求
* upstream service：指位于Kong后面的API/服务，客户端请求被转发到该服务
* Service：服务实体是每个上游服务的抽象。如数据转换微服务、计费API等
* Route：指Kong实体——Route是进入Kong的入口，并为要匹配的请求定义规则，并路由到给定的服务
* Plugin：指Kong的Plugins，在代理生命周期中运行的业务逻辑。插件可以通过管理API进行配置——可以全局配置(所有入口)，也可以在特定的路由和服务上配置
<!-- more -->

## 概览
从上层看，Kong在其配置的代理端口上监听HTTP流量(默认为8000和8443)。Kong将根据配置的路由评估任何传入HTTP请求，并尝试找到匹配的路由。如果请求与路由规则匹配，Kong将处理代理请求。因为每个路由都链接到一个服务，Kong将运行在路由及其关联服务上配置的插件，然后代理上游的请求。
可以通过Kong的Admin API管理路由——hosts、paths和methods都可以作为匹配规则。

### 未匹配的请求
未匹配到路由/路由未配置：
```plain
HTTP/1.1 404 Not Found
Date: Fri, 28 Sep 2018 08:08:36 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Server: kong/0.14.1
Content-Length: 58

{"message":"no route and no API found with those values"}
```

> 返回的mesage提到了API，出于向后兼容的原因，Kong 0.13仍然支持API实体(如果匹配路由失败，则尝试匹配配置的API)

## 如何配置服务
> 在[文档](https://docs.konghq.com/0.14.x/getting-started/configuring-a-service/)中解释了如何通过Admin API配置

### 添加Service
```plain
curl -i -X POST http://localhost:8001/services/ \
    -d 'name=foo-service' \
    -d 'url=http://foo-service.com'
HTTP/1.1 201 Created
Date: Fri, 28 Sep 2018 08:16:33 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Server: kong/0.14.1
Content-Length: 259

{"host":"foo-service.com","created_at":1538122593,"connect_timeout":60000,"id":"1c0a70da-ca56-44bb-8b4b-3e10a154450f","protocol":"http","name":"foo-service","read_timeout":60000,"port":80,"path":null,"updated_at":1538122593,"retries":5,"write_timeout":60000}
```
在Kong注册一个名为“foo-service”的服务，指向__http://foo-service.com__(上游)

Note: url参数是用来填充的简单参数——一次添加protocol, host, port, and path

### 添加Route
为了通过Kong向该服务发送请求——我们需要指定一条路由，它充当Kong的入口

#### routes
其中service.id是添加Service时返回的id
```plain
curl -i -X POST http://localhost:8001/routes/ \
    -d 'hosts[]=example.com' \
    -d 'paths[]=/foo' \
    -d 'service.id=1c0a70da-ca56-44bb-8b4b-3e10a154450f'
HTTP/1.1 201 Created
Date: Fri, 28 Sep 2018 08:28:04 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Server: kong/0.14.1
Content-Length: 295

{"created_at":1538123284,"strip_path":true,"hosts":["example.com"],"preserve_host":false,"regex_priority":0,"updated_at":1538123284,"paths":["\/foo"],"service":{"id":"1c0a70da-ca56-44bb-8b4b-3e10a154450f"},"methods":null,"protocols":["http","https"],"id":"b3bc45dc-0805-4a03-9298-e76e1175c096"}
```

#### services/xxx/routes
在入门篇中，请求的是指定service对应接口，这里有个小不同
```plain
curl -i -X POST \
  --url http://localhost:8001/services/example-service/routes \
  --data 'hosts[]=example.com'
```

现在，我们已经配置了一条路由来匹配给定hosts和paths的请求，并将它们转发到我们配置的__foo-service__，从而将该流量代理到http://foo-service.com

Kong是一个透明的代理，默认情况下将把请求转发至未受影响的上游服务，除了HTTP规范要求的Connection、Date等不同的header之外

## 路由及匹配功能
> Kong如何将请求与已配置的hosts、paths和methods匹配
> 注意：这三个字段是可选的，但至少指定一个

路由匹配的要求：
* 请求必须包含所有已配置字段
* 请求中字段的值必须至少匹配一个配置值(虽然field配置接受一个或多个值，但请求只需要其中一个值就可以认为是匹配的)
Note：这里第二个要求指的是具体field匹配一个即可，比如methods可能有多个，只匹配一个即可，毕竟也没办法带多个method类型

### 匹配案例
#### 路由配置
```json
{
    "hosts": ["example.com", "foo-service.com"],
    "paths": ["/foo", "/bar"],
    "methods": ["GET"]
}
```

#### 匹配通过
```plain
GET /foo HTTP/1.1
Host: example.com

GET /bar HTTP/1.1
Host: foo-service.com

GET /foo/hello/world HTTP/1.1
Host: example.com
```

#### 匹配失败
```plain
// paths不匹配
GET / HTTP/1.1
Host: example.com

// methods不匹配
POST /foo HTTP/1.1
Host: example.com

// hosts不匹配
GET /foo HTTP/1.1
Host: foo.com
```

### Host Header
基于Host header路由请求是通过Kong代理通信的最直接的方式，特别是因为这是HTTP Host header的预期用法——Kong使得通过Route实体的host字段进行操作变得很容易

* hosts 接受多个值，当通过Admin API指定它们时，必须用逗号分隔
* hosts 接受多个值，在JSON payload中很容易表示

#### 基础配置
JSON payload：
```powershell
curl -i -X POST http://localhost:8001/routes/ \
    -H 'Content-Type: application/json' \
    -d '{"hosts":["example.com", "foo-service.com"]}'
```

hosts[]：由于Admin API也支持form-urlencoded内容类型
```plain
curl -i -X POST http://localhost:8001/routes/ \
    -d 'hosts[]=example.com' \
    -d 'hosts[]=foo-service.com'
```

按照上述的方式添加Route后，如果要匹配，则在请求头中需要满足以下二者之一：
```plain
Host: example.com
Host: foo-service.com
```

#### 使用通配符的主机名
为了提供灵活性，Kong允许在hosts字段中使用通配符指定主机名。通配符主机名允许任何匹配的Host header满足条件，从而匹配给定的路由。
通配符主机名必须在域的最左或最右标签上仅包含一个星号：
* \*.example.com：允许如a.example.com和x.y.example.com匹配
* example.\*：将允许如example.com和example.org匹配

完整例子如下：
```json
{
    "hosts": ["*.example.com", "service.com"]
}
```

匹配成功：
```plain
GET / HTTP/1.1
Host: an.example.com

GET / HTTP/1.1
Host: service.com
```

#### preserve\_host


![image.png | left | 751x127](https://cdn.nlark.com/yuque/0/2018/png/92822/1538126272900-eb1ca537-d0d3-433e-bb77-15ddd385e8b4.png "")

代理时，Kong的默认行为是将上游请求的主机头设置为服务主机中指定的主机名。preserve\_host字段接受一个布尔标记，指示Kong不要这样做

当没有配置preserve\_host 时，及Route的配置如下：
```json
{
    "hosts": ["service.com"],
    "service": {
        "id": "..."
    }
}
```

客户端请求：
```plain
GET / HTTP/1.1
Host: service.com
```

Kong将从服务的主机属性中提取Host header，向上游发起请求：
```plain
GET / HTTP/1.1
Host: <my-service-host.com>
```

在路由中配置preserve\_host=true ：
```json
{
    "hosts": ["service.com"],
    "preserve_host": true,
    "service": {
        "id": "..."
    }
}
```

发起同样的请求，Kong将保留客户端请求上的Host，向上游请求:
```plain
GET / HTTP/1.1
Host: service.com
```

### Path
要匹配路由的另一种方法是通过请求路径——为了满足这个路由条件，客户端请求的路径必须以paths属性的一个值为前缀
* 默认情况下，Kong将在不改变URL路径的情况下向上游代理请求
* 首先计算最长路径：允许用两条路径定义两条路由：如/service和/service/resource

#### 基础配置
假设路由配置如下：
```json
{
    "paths": ["/service", "/hello/world"]
}
```

匹配成功：
```plain
GET /service HTTP/1.1
Host: example.com

GET /service/resource?param=value HTTP/1.1
Host: example.com

GET /hello/world/resource HTTP/1.1
Host: anything.com
```

#### 使用正则表达式
Kong支持通过PCRE (Perl兼容正则表达式)对路由路径字段进行正则匹配，可以同时将路径作为前缀和regex分配给路由。

假设路由配置如下：
```json
{
    "paths": ["/users/\d+/profile", "/following"]
}
```

匹配成功：
```plain
GET /following HTTP/1.1
Host: ...

GET /users/123/profile HTTP/1.1
Host: ...
```

##### 匹配顺序
前面提到过，Kong按长度计算前缀路径：最长的前缀路径首先计算。然而，Kong将根据路由的regex\_priority属性评估regex路径，以下列路由为例：
```json
[
    {
        "paths": ["/status/\d+"],
        "regex_priority": 0
    },
    {
        "paths": ["/version/\d+/status/\d+"],
        "regex_priority": 6
    },
    {
        "paths": ["/version"],
        "regex_priority": 3
    },
]
```

匹配顺序如下：
* /version
* /version/\d+/status/\d+
* /status/\d+

前缀路径总是先处理；请求必须仍然匹配路由的主机和方法属性，而Kong将遍历您的路由，直到找到匹配最多规则的路由

##### 捕获组
支持捕获组，并且匹配的组将从路径中提取出来，可供插件使用

考虑以下正则表达式：
```perl
/version/(?<version>\d+)/users/(?<user>\S+)
```

以下的请求路径：
```plain
/version/1/users/john
```

Kong将把请求路径视为匹配，如果整个路由匹配(考虑到hosts和methods字段)，则提取的捕获组将从ngx中的插件中可用。ctx变量:
```plain
local router_matches = ngx.ctx.router_matches

-- router_matches.uri_captures is:
-- { "1", "john", version = "1", user = "john" }
```

##### 转义特殊字符
值得注意的是，regex中发现的字符通常是根据RFC 3986保留的字符，因此应该使用percent-encoded。在通过Admin API配置regex路径的路由时，如果需要—确保URL对payload进行编码。例如，使用curl和使用 application/x-www-form-urlencoded MIME类型：
```plain
curl -i -X POST http://localhost:8001/routes \
    --data-urlencode 'uris[]=/status/\d+'
```

Note：curl不会自动地对payload进行URL编码，并注意——data-urlencode的使用情况，它可以防止+字符被Kong的Admin API解码并解释为空格

#### strip\_path
可能需要指定path前缀以匹配路由，但不应将其包含在上游请求中。为此，通过配置这样的路由，使用strip\_path boolean属性：
```json
{
    "paths": ["/service"],
    "strip_path": true,
    "service": {
        "id": "..."
    }
}
```

启用此flag将指示Kong，当匹配此路由并继续代理到服务时，它不应该在上游请求的URL中包含URL路径的匹配部分。

例如，请求如下：
```plain
GET /service/path/to/resource HTTP/1.1
Host: ...
```

向上游发送的请求：
```plain
GET /path/to/resource HTTP/1.1
Host: ...
```

同样，如果regex路径是在启用strip\_path的路由上定义的，那么请求URL匹配序列的全部将被删除：
```json
{
    "paths": ["/version/\d+/service"],
    "strip_path": true,
    "service": {
        "id": "..."
    }
}
```

以下HTTP请求匹配提供的regex路径：
```plain
GET /version/1/service/path/to/resource HTTP/1.1
Host: ...
```

向上游发送的请求：
```plain
GET /path/to/resource HTTP/1.1
Host: ...
```

### HTTP method
methods字段允许根据其HTTP方法匹配请求，接受多个值，默认值为空(HTTP方法不用于路由)

以下路由允许通过 GET 及 HEAD：
```json
{
    "methods": ["GET", "HEAD"],
    "service": {
        "id": "..."
    }
}
```

匹配成功：
```plain
GET / HTTP/1.1
Host: ...

HEAD /resource HTTP/1.1
Host: ...
```

上述的规则不会匹配POST或DELETE请求——这使得在路由上配置插件的粒度更细
例如，可以设想有两种指向同一服务的路由：
* 无限的未经身份验证的GET请求
* 只允许经过身份验证和速率限制的POST请求(通过对此类请求应用身份验证和速率限制插件)

## 匹配优先级
Route可以根据hosts、paths和methods字段定义匹配规则。要使传入请求与路由匹配，必须满足所有现有字段。然而，Kong允许使用包含相同值的字段配置两个或多个路由，从而提供了相当大的灵活性——当发生这种情况时，Kong应用优先级规则。
规则：当评估请求时，Kong将首先尝试匹配与大多数规则匹配的路由

如下路由配置：
```json
{
    "hosts": ["example.com"],
    "service": {
        "id": "..."
    }
},
{
    "hosts": ["example.com"],
    "methods": ["POST"],
    "service": {
        "id": "..."
    }
}
```

第二种路由有一个hosts字段和一个methods字段，因此它将首先由Kong进行评估。这样做，我们就避免了第一个路由覆盖第二个。

这个请求将匹配第一个：
```plain
GET / HTTP/1.1
Host: example.com
```

这个请求将匹配第二个：
```plain
POST / HTTP/1.1
Host: example.com
```

按照这种逻辑，如果第三条路由配置为一个hosts字段、一个methods字段和一个uris字段，Kong将首先对其进行评估

## 代理行为
上述代理规则详细介绍了Kong如何将传入请求转发到上游服务。下面将详细介绍在Kong与注册路由匹配HTTP请求和请求实际转发之间的内部情况

### 负载均衡
Kong实现了负载平衡功能，以便在上游服务的实例池中分发代理请求，见[官方文档](https://docs.konghq.com/0.14.x/loadbalancing/)

### 插件执行
Kong是可扩展的，可以通过“插件”将自己hook到代理请求的请求/响应生命周期中。插件可以环境中对代理请求执行各种操作 __和/或__ 转换。
插件可以配置为全局运行(对于所有代理流量)或在特定的路由和服务上运行。在这两种情况下，都必须通过Admin API创建插件配置。
一旦匹配了路由(及其关联的Services实体)，Kong将运行与实体相关联的插件。__在Route上配置的插件在Service上配置的插件之前运行__，但在其他情况下，应用通常的插件关联规则。
这些配置好的插件将在它们的访问阶段运行，见[官方文档](https://docs.konghq.com/0.14.x/plugin-development/)

### 代理 & 上游超时
一旦Kong执行了所有必要的逻辑(包括插件)，它就可以将请求转发到上游服务。这是通过Nginx的ngx\_http\_proxy\_module完成的。

可以通过服务的以下属性为Kong和给定的上游连接配置所需的超时：
* upstream\_connect\_timeout：以毫秒为单位定义建立到上游服务的连接的超时，默认为60000
* upstream\_send\_timeout：以毫秒为单位定义两个连续写操作之间的超时，用于将请求发送到上游服务，默认为60000
* upstream\_read\_timeout：以毫秒为单位定义从上游服务接收请求的两个连续读操作之间的超时，默认为60000

Kong将通过HTTP/1.1发送请求，并设置以下headers：
* Host: <your\_upstream\_host>
* Connection: keep-alive：允许重用上游连接
* X-Real-IP: <remote\_addr>：其中$remote\_addr是ngx\_http\_core\_module提供的同名变量。注意：$remote\_addr可能被ngx\_http\_realip\_module覆盖
* X-Forwarded-For: <address\>：其中<address\>是由ngx\_http\_realip\_module以相同的名称附加到请求头的$realip\_remote\_addr的内容
* X-Forwarded-Proto: <protocol>：其中<protocol>是客户端使用的协议。在$realip\_remote\_addr是受信任地址之一的情况下，如果提供相同名称的请求header，就会转发。否则，将使用ngx\_http\_core\_module提供的$scheme变量的值
* X-Forwarded-Host: <host>：其中<host>是客户端发送的host名。在$realip\_remote\_addr是受信任地址之一的情况下，如果提供相同名称的请求header，就会转发。否则，将使用ngx\_http\_core\_module提供的$host变量的值
* X-Forwarded-Port: <port>：其中<port>是接受请求的服务器端口。在$realip\_remote\_addr是受信任地址之一的情况下，如果提供相同名称的请求header，就会转发。否则，将使用ngx\_http\_core\_module提供的$server\_port变量的值

所有其他请求header都按原样由Kong转发。
在使用WebSocket协议时，有一个例外。在这种情况下，Kong将设置以下头信息，以便在客户端和您的上游服务之间升级协议：
* Connection: Upgrade
* Upgrade: websocket

关于这部分，更多[可见文档](https://docs.konghq.com/0.14.x/proxy/#proxy-websocket-traffic)

### 错误 & 重试
当代理期间发生错误时，Kong将使用底层的Nginx重试机制将请求传递给下一个上游
这里有两个可配置的元素：
* 重试次数：可以使用retries 属性为每个服务配置重试次数，见[文档](https://docs.konghq.com/0.14.x/admin-api/)
* 什么构成了一个错误：Kong使用Nginx默认值，这意味着在与服务器建立连接、向其传递请求或读取响应头时发生错误或超时

第二个选项基于Nginx的[proxy\_next\_upstream][proxy\_next\_upstream]指令。此选项不能通过Kong直接配置，但可以使用自定义Nginx配置添加，见[文档](https://docs.konghq.com/0.14.x/configuration/)

### 响应
Kong接收来自上游服务的响应，并以流的方式将其发送回下游客户端。此时，Kong将执行添加到Route和/或Service中的后续插件，这些插件在header\_filter阶段实现了一个钩子。
一旦执行了所有已注册插件的header\_filter阶段，Kong将添加以下标题，并将全部标题发送给客户：
* Via: kong/x.x.x：x.x.x是Kong的版本
* X-Kong-Proxy-Latency: <latency>：latency是Kong从客户端接收请求并将请求发送到上游服务之间的时间(以毫秒为单位)
* X-Kong-Upstream-Latency: <latency>：等待时间是Kong等待上游服务响应的第一个字节(以毫秒为单位)

一旦消息头被发送到客户端，Kong将开始为实现body\_filter钩子的Route和/或Service执行注册插件。由于Nginx的流特性，这个钩子可能被调用多次。由body\_filter钩子成功处理的上游响应的每个块都被发送回客户端，可见[文档](https://docs.konghq.com/0.14.x/plugin-development/)

## 配置回退路由
作为一个Kong提供实际用例和灵活性的例子，尝试实现一个"fallback Route"，所以为了避免Kong与HTTP 404响应，"no route found"，我们可以捕捉这些请求，并将它们代理给一个特殊的上游服务，或者应用插件（例如，这样的插件可以使用不同的状态码或响应终止请求，而无需代理请求）

fallback Route：
```json
{
    "paths": ["/"],
    "service": {
        "id": "..."
    }
}
```

对Kong发出的任何HTTP请求实际上都与此路由匹配，因为所有uri都以根字符/作为前缀。从请求路径部分我们知道，最长的URL路径首先由Kong计算，因此/ path最终将由Kong计算，并且有效地提供了一个“后备”路径，仅作为最后的手段进行匹配。然后，可以将流量发送到一个我们希望的特殊Service或应用任何插件

## 为路由配置SSL
### 限制客户端协议(HTTP/HTTPS)

## 代理WebSocket流量
### WebSocket和TLS

## 资料
* [https://docs.konghq.com/0.14.x/configuration/#proxy\_listen](https://docs.konghq.com/0.14.x/configuration/#proxy_listen)
* [https://docs.konghq.com/0.14.x/proxy](https://docs.konghq.com/0.14.x/proxy)

