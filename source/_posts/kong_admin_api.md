---
title: Kong——Admin API文档
date: 2018/09/29 20:00:00
tags:
  - API Gateway
  - Kong
categories: 
  - [微服务]
  - [Kong]
---

> Endpoint非特殊说明，默认为GET
> 本文是基于官方文档做的实践，主要内容是基于官方文档内容做的翻译以及在实践中的个人理解

Kong为了便于管理附带了一个内部RESTful Admin API。对Admin API的请求可以发送到集群中的任何节点，而Kong将保持所有节点的配置一致
* 8001：Admin API监听的默认端口
* 8444：Admin API的HTTPS默认端口

这个API是为内部使用而设计的，它提供了对Kong的完全控制，所以在设置Kong环境时要小心，以避免不必要的公开暴露这个API
<!-- more -->

## 特殊说明
本文还有Certificate Object及SNI Object未完成

## 支持Content Types
Admin API 接受两种类型：

### application/x-www-form-urlencoded
对于基本请求body来说足够简单，可能会经常使用。
注意：在发送嵌套值时，Kong希望嵌套对象使用带点的键引用，如下：
```plain
config.limit=10&config.period=seconds
```

### application/json
对于复杂的body(复杂的插件配置)来说很方便，在这种情况下，只需发送想要发送的数据的JSON：
```json
{
    "config": {
        "limit": 10,
        "period": "seconds"
    }
}
```

## Information routes
### 检索节点信息
检索关于节点的通用详细信息

#### Endpoint
```plain
/
```

#### Response
```json
{
    "plugins":{
        "enabled_in_cluster":[
            "key-auth"
        ],
        "available_on_server":Object{...}
    },
    "tagline":"Welcome to kong",
    "configuration":Object{...},
    "version":"0.14.1",
    "node_id":"1e9d57bb-c2ae-445c-8ac7-068dac657348",
    "lua_version":"LuaJIT 2.1.0-beta3",
    "prng_seeds":{
        "pid: 55":233132241169
    },
    "timers":{
        "pending":5,
        "running":0
    },
    "hostname":"0afb364e8473"
}
```

* node\_id：表示正在运行的Kong节点的UUID。这个UUID是在Kong启动时随机生成的，因此节点在每次重新启动时都会有一个不同的node\_id
* available\_on\_server：节点上安装的插件名称
* enabled\_in\_cluster：启用/配置插件的名称。也就是说，当前数据存储中的插件配置由所有Kong节点共享。

### 检索节点状态
检索有关节点的使用信息，并提供一些关于基础nginx进程正在处理的连接的基本信息，以及数据库连接的状态
如果想要监视Kong进程，因为Kong是构建在nginx之上的，可以使用每个现有的nginx监视工具或代理

#### Endpoint
```plain
/status
```

#### Response
```json
{
    "database":{
        "reachable":true
    },
    "server":{
        "connections_writing":1,
        "total_requests":1801,
        "connections_handled":1837,
        "connections_accepted":1837,
        "connections_reading":0,
        "connections_active":1,
        "connections_waiting":0
    }
}
```

* server：关于nginx HTTP/S服务器的度量
    * total\_requests：客户端请求的总数
    * connections\_active：当前活动客户端连接的数量，包括等待连接
    * connections\_accepted：接受的客户端连接的总数
    * connections\_handled：已处理连接的总数。通常，除非达到了某些资源限制，否则参数值与接受值相同
    * connections\_reading：Kong正在读取请求头的当前连接数
    * connections\_writing：nginx将响应写回客户端的当前连接数
    * connections\_waiting：等待请求的空闲客户端连接的当前数量
* database：关于数据库的度量
    * reachable：反映数据库连接状态的布尔值。注意：此标志并不反映数据库本身的健康状况

## Service Object
顾名思义，服务实体是每个上游服务的抽象。服务的示例包括数据转换微服务、计费API等
服务的主要属性是其URL (Kong应该代理到的地方)，可以将其设置为单个字符串，也可以单独指定其protocol、host、port和path

服务与路由相关联(服务可以有许多与之关联的路由)。路由是Kong的入口，定义规则以匹配客户端请求。匹配路由后，Kong将请求代理给其关联的服务，见[文档](https://docs.konghq.com/0.14.x/proxy/)。

### 添加Service
#### Endpoint
```plain
POST /services/
```

#### Request Body

| 属性 | 描述 |
| --- | --- |
| name | 服务名称 |
| protocol | 与上游通信使用的协议。http(默认)及https二者之一 |
| host | 上游服务host |
| port | 上游服务port，默认是80 |
| path | 请求至上游服务的path，默认为空 |
| retries | 代理失败时的重试次数，默认为5 |
| connect\_timeout | 与上游服务建立连接的超时时间（以毫秒为单位），默认为60000 |
| write\_timeout | 将请求发送至上游服务两个连续写操作之间的超时（以毫秒为单位） |
| read\_timeout | 与write\_timeout类似，__读__操作 |
| url | 快捷属性——protocol host port path。这个属性是只写的（Admin API 不会返回这个属性） |

#### Response
```powershell
curl -i -X POST http://localhost:8001/services/ \
     -d 'name=admin-service' \
     -d 'url=http://admin.com'

```
```json
{
    "host":"admin.com",
    "created_at":1538192759,
    "connect_timeout":60000,
    "id":"eceb7e8c-5009-4c4b-8018-9eca69116749",
    "protocol":"http",
    "name":"admin-service",
    "read_timeout":60000,
    "port":80,
    "path":null,
    "updated_at":1538192759,
    "retries":5,
    "write_timeout":60000
}
```

### 检索Service
#### Endpoints
##### /services/{name or id}

| 属性 | 描述 |
| --- | --- |
| name or id | 要检索服务的唯一标识符或名称 |

##### /routes/{route id}/service

| 属性 | 描述 |
| --- | --- |
| route id | 属于要检索的服务的Route的唯一标识符 |

#### Response
```powershell
curl -i http://localhost:8001/services/admin-service
```
```json
{
    "host":"admin.com",
    "created_at":1538192759,
    "connect_timeout":60000,
    "id":"eceb7e8c-5009-4c4b-8018-9eca69116749",
    "protocol":"http",
    "name":"admin-service",
    "read_timeout":60000,
    "port":80,
    "updated_at":1538192759,
    "retries":5,
    "write_timeout":60000
}
```

### Services 列表
#### Endpoint
```plain
/services/
```

#### Request Querystring Parameters

| 属性 | 描述 |
| --- | --- |
| offset | 用于分页的游标——offset是一个对象标识符，它定义列表中的一个位置 |
| size | 每页返回的数量 |

#### Response
```plain
curl -i http://localhost:8001/services?size=2
```
```json
{
    "next":"/services?offset=WyI0YmRiOTdiNC00NzM1LTQwM2QtYWZkNi04MTE3NzBiODcxN2IiXQ",
    "data":[
        {
            "host":"foo-service.com",
            "created_at":1538122593,
            "connect_timeout":60000,
            "id":"1c0a70da-ca56-44bb-8b4b-3e10a154450f",
            "protocol":"http",
            "name":"foo-service",
            "read_timeout":60000,
            "port":80,
            "path":null,
            "updated_at":1538122593,
            "retries":5,
            "write_timeout":60000
        },
        {
            "host":"mockbin.org",
            "created_at":1538034594,
            "connect_timeout":60000,
            "id":"4bdb97b4-4735-403d-afd6-811770b8717b",
            "protocol":"http",
            "name":"example-service",
            "read_timeout":60000,
            "port":80,
            "path":null,
            "updated_at":1538034594,
            "retries":5,
            "write_timeout":60000
        }
    ],
    "offset":"WyI0YmRiOTdiNC00NzM1LTQwM2QtYWZkNi04MTE3NzBiODcxN2IiXQ"
}
```

### 更新Service
#### Endpoints
##### PATCH /services/{name or id}

| 属性 | 描述 |
| --- | --- |
| name or id | 要更新服务的id或name属性 |

##### PATCH /routes/{route id}/service

| 属性 | 描述 |
| :--- | :--- |
| route id | 要更新服务的路由id属性 |

#### Request Body
> 参考__添加Service__时的参数

#### Response
```powershell
curl -i -X PATCH http://localhost:8001/services/admin-service \
    -d 'url=http://admin.com/api'
```
```json
{
    "host":"admin.com",
    "created_at":1538192759,
    "connect_timeout":60000,
    "id":"eceb7e8c-5009-4c4b-8018-9eca69116749",
    "protocol":"http",
    "name":"admin-service",
    "read_timeout":60000,
    "port":80,
    "path":"/api",
    "updated_at":1538200257,
    "retries":5,
    "write_timeout":60000
}
```

### 更新或创建Service
#### Endpoint
```plain
PUT /services/{name or id}
```

#### Request Body
> 参考__添加Service__时的参数

* 用Body中指定的定义插入(或替换)请求资源下的服务，服务将通过名称或id属性进行标识
* 当名称或id属性具有UUID的结构时，插入/替换的服务将由其id标识；否则，它将由其名称来标识
* 在不指定id(既不在URL中也不在正文中)的情况下创建新服务时，它将自动生成
* 不允许在URL中指定的名称与在请求Body中的名称不同

#### Response
```powershell
curl -i -X PUT http://localhost:8001/services/admin-service \
    -d 'url=http://admin.com/api/doc'
```
```json
{
    "host":"admin.com",
    "created_at":1538200849,
    "connect_timeout":60000,
    "id":"eceb7e8c-5009-4c4b-8018-9eca69116749",
    "protocol":"http",
    "name":"admin-service",
    "read_timeout":60000,
    "port":80,
    "path":"/api/doc",
    "updated_at":1538200849,
    "retries":5,
    "write_timeout":60000
}
```

### 删除Service
#### Endpoint
```plain
DELETE /services/{name or id}
```

#### Response
```plain
curl -i -X DELETE http://localhost:8001/services/admin-service
```
```plain
HTTP/1.1 204 No Content
Date: Sat, 29 Sep 2018 06:02:59 GMT
Connection: keep-alive
Access-Control-Allow-Origin: *
Server: kong/0.14.1
```

## Route Object
路由实体定义规则以匹配客户端请求，每个路由都与服务相关联，服务可能有多个路由与之相关联——每个匹配给定路由的请求都将代理到其关联服务。

路由和服务的组合(以及它们之间的关注点分离)提供了一种强大的路由机制，可以通过这种机制在Kong定义细粒度的入口点，从而为基础设施提供不同的上游服务。

重新添加测试Service：
```plain
curl -i -X POST http://localhost:8001/services/ \
     -d 'name=admin-service' \
     -d 'url=http://admin.com/api'
```
```json
{
    "host":"admin.com",
    "created_at":1538202152,
    "connect_timeout":60000,
    "id":"50adcbf0-8553-466a-b957-49b6acdbbe6d",
    "protocol":"http",
    "name":"admin-service",
    "read_timeout":60000,
    "port":80,
    "path":"/api",
    "updated_at":1538202152,
    "retries":5,
    "write_timeout":60000
}
```
### 添加Route
#### Endpoints
##### POST /routes/
##### POST /services/{name or id}/routes/
> 这个在Admin API文档中漏了，但是Proxy部分的文档中有类似使用，亲测有效

name or id 是service的标识符

#### Request Body
| 属性          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| protocols     | 此路由允许的协议列表，默认情况下，是["http"， "https"]，两种协议都接收。当设置为["https"]时，HTTP请求将被响应为升级到https的请求。使用form-encoded时像这样protocols[]=http&protocols[]=https；用JSON时，使用数组即可。 |
| methods       | 匹配此路由的HTTP方法列表，如["GET", "POST"]，必须至少设置一个hosts, paths 或 methods。编码方式与protocols类似 |
| hosts         | 匹配此路由的域名列表，如example.com。其它与methods类似       |
| path          | 匹配此路径的路径列表，如/my-path。其它与methods类似          |
| strip_path    | 当匹配一条路径路由时，从上游请求URL中删除匹配前缀，默认值为true |
| preserve_host | 当通过主机域名匹配路由时，在上游请求头中使用请求Host header替换。默认设置为false，上游Host header将为服务的host |
| service       | 此路由关联的服务，这是路由代理通信到的地方。两种方式：service.id=<service_id> "service":{"id":"<service_id>"} |

#### Response
```powershell
curl -i -X POST http://localhost:8001/routes/ \
    -d 'hosts[]=example.com' \
    -d 'paths[]=/foo' \
    -d 'service.id=50adcbf0-8553-466a-b957-49b6acdbbe6d'
```
```json
{
    "created_at":1538202438,
    "strip_path":true,
    "hosts":[
        "example.com"
    ],
    "preserve_host":false,
    "regex_priority":0,
    "updated_at":1538202438,
    "paths":[
        "/foo"
    ],
    "service":{
        "id":"50adcbf0-8553-466a-b957-49b6acdbbe6d"
    },
    "methods":null,
    "protocols":[
        "http",
        "https"
    ],
    "id":"35bd4d4e-cde7-4f7e-8b55-a688be3d28d6"
}
```

### 检索Route
#### Endpoint
```plain
/routes/{id}
```

id：要检索Route的id属性

#### Response
```powershell
curl -i -X GET http://localhost:8001/routes/35bd4d4e-cde7-4f7e-8b55-a688be3d28d6
```
```json
{
    "created_at":1538202438,
    "strip_path":true,
    "hosts":[
        "example.com"
    ],
    "preserve_host":false,
    "regex_priority":0,
    "updated_at":1538202438,
    "paths":[
        "/foo"
    ],
    "service":{
        "id":"50adcbf0-8553-466a-b957-49b6acdbbe6d"
    },
    "protocols":[
        "http",
        "https"
    ],
    "id":"35bd4d4e-cde7-4f7e-8b55-a688be3d28d6"
}
```

### Routes列表
#### Endpoint
```plain
/routes    
```

#### Request Querystring Parameters
与__Services列表__类似，offset及size属性

#### Response
```powershell
curl -i -X GET http://localhost:8001/routes?size=2
```
```json
{
    "next":"/routes?offset=WyJhNDkzNTcyOC1iMDFlLTQzNmItYmYyNy0wY2U2YzczYTM3MzAiXQ",
    "data":[
        {
            "created_at":1538202438,
            "strip_path":true,
            "hosts":[
                "example.com"
            ],
            "preserve_host":false,
            "regex_priority":0,
            "updated_at":1538202438,
            "paths":[
                "/foo"
            ],
            "service":{
                "id":"50adcbf0-8553-466a-b957-49b6acdbbe6d"
            },
            "methods":null,
            "protocols":[
                "http",
                "https"
            ],
            "id":"35bd4d4e-cde7-4f7e-8b55-a688be3d28d6"
        },
        Object{...}
    ],
    "offset":"WyJhNDkzNTcyOC1iMDFlLTQzNmItYmYyNy0wY2U2YzczYTM3MzAiXQ"
}
```

### 列出与服务相关的路由
#### Endpoint
```plain
/services/{service name or id}/routes
```
要检索服务（id或名称）的所属路由——在使用此Endpoint时，只列出属于指定服务的路由

#### Response
```plain
curl -i -X GET http://localhost:8001/services/admin-service/routes
```
```plain
{
    "next":null,
    "data":[
        {
            "created_at":1538202438,
            "strip_path":true,
            "hosts":[
                "example.com"
            ],
            "preserve_host":false,
            "regex_priority":0,
            "updated_at":1538202438,
            "paths":[
                "/foo"
            ],
            "service":{
                "id":"50adcbf0-8553-466a-b957-49b6acdbbe6d"
            },
            "protocols":[
                "http",
                "https"
            ],
            "id":"35bd4d4e-cde7-4f7e-8b55-a688be3d28d6"
        },
        Object{...},
        Object{...}
    ]
}
```

### 更新Route
#### Endpoint
```plain
PATCH /routes/{id}
```
要更新的路由的id属性

#### Request Body
同__添加Route__部分

#### Response
```powershell
curl -i -X PATCH http://localhost:8001/routes/35bd4d4e-cde7-4f7e-8b55-a688be3d28d6 \
    -d 'paths[]=/api'
```
```json
{
    "created_at":1538202438,
    "strip_path":true,
    "hosts":[
        "example.com"
    ],
    "preserve_host":false,
    "regex_priority":0,
    "updated_at":1538204091,
    "paths":[
        "/api"
    ],
    "service":{
        "id":"50adcbf0-8553-466a-b957-49b6acdbbe6d"
    },
    "methods":null,
    "protocols":[
        "http",
        "https"
    ],
    "id":"35bd4d4e-cde7-4f7e-8b55-a688be3d28d6"
}
```

### 更新或创建Route
#### Endpoint
```plain
PUT /routes/{id}
```
id：要更新的路由id属性

#### Request Body
同__添加Route__部分

* 用body中指定的定义插入(或替换)请求资源下的路由
* Route将由URL中给出的id属性标识

#### Response
```powershell
curl -i -X PUT http://localhost:8001/routes/35bd4d4e-cde7-4f7e-8b55-a688be3d28d6 \
    -d 'paths[]=/apixxx' \
    -d 'hosts[]=example.com' \
    -d 'service.id=50adcbf0-8553-466a-b957-49b6acdbbe6d'
```
```json
{
    "created_at":1538204412,
    "strip_path":true,
    "hosts":[
        "example.com"
    ],
    "preserve_host":false,
    "regex_priority":0,
    "updated_at":1538204412,
    "paths":[
        "/apixxx"
    ],
    "service":{
        "id":"50adcbf0-8553-466a-b957-49b6acdbbe6d"
    },
    "methods":null,
    "protocols":[
        "http",
        "https"
    ],
    "id":"35bd4d4e-cde7-4f7e-8b55-a688be3d28d6"
}
```

### 删除Route
#### Endpoint
```plain
DELETE /routes/{id}
```
id：要删除的Route id属性

#### Response
```powershell
curl -i -X DELETE http://localhost:8001/routes/35bd4d4e-cde7-4f7e-8b55-a688be3d28d6
```

## Consumer Object
消费者对象表示服务的消费者或用户，可以依赖Kong作为主要数据存储，也可以将消费者列表映射到数据库，以保持Kong与现有主数据存储之间的一致性

### 创建Consumer
#### Endpoint
```plain
POST /consumers/
```

#### Request Form Parameters

| 属性 | 描述 |
| --- | --- |
| username | 用户的唯一用户名。必须将此字段或custom\_id与请求一起发送 |
| custom\_id | 为消费者存储现有的唯一ID——用于在现有数据库中与用户进行映射，必须将此字段或username与请求一起发送 |


#### Response
username 与 custom\_id都传上：
```powershell
curl -i -X POST http://localhost:8001/consumers \
    -d 'custom_id=10' \
    -d 'username=10'
```
```json
{
    "custom_id":"10",
    "created_at":1538205455,
    "username":"10",
    "id":"24d9cf29-f6e0-40d6-a1ec-7fb00161ee50"
}
```

### 检索Consumer
#### Endpoint
```plain
/consumers/{username or id}
```
要检索的消费者唯一的用户名或id（不是custom\_id）

#### Response
```powershell
curl -i -X GET http://localhost:8001/consumers/24d9cf29-f6e0-40d6-a1ec-7fb00161ee50

curl -i -X GET http://localhost:8001/consumers/10
```
```json
{
    "custom_id":"10",
    "created_at":1538205455,
    "username":"10",
    "id":"24d9cf29-f6e0-40d6-a1ec-7fb00161ee50"
}
```

### Consumers列表
#### Endpoint
```plain
/consumers/
```

#### Request Querystring Parameters
> custom\_id这个属性应该是官网错误，与之前类似，传入的应当是size 与 offset

#### Response
```plain
curl -i -X GET http://localhost:8001/consumers?size=1
```
```json
{
    "next":"/consumers?offset=WyIwYWQ2MzAyOS1jMmZkLTRhMjUtYmE5Ny00MzNhZWM2ZWRhZmUiXQ",
    "data":[
        {
            "custom_id":null,
            "created_at":1538040529,
            "username":"Jason",
            "id":"0ad63029-c2fd-4a25-ba97-433aec6edafe"
        }
    ],
    "offset":"WyIwYWQ2MzAyOS1jMmZkLTRhMjUtYmE5Ny00MzNhZWM2ZWRhZmUiXQ"
}
```

### 更新Consumer
#### Endpoint
```plain
PATCH /consumers/{username or id}
```
要更新消费者的唯一标识符或用户名

#### Request Body
参考__创建Consumer__部分，username与custom\_id属性

#### Response
```plain
curl -i -X PATCH http://localhost:8001/consumers/10 \
    -d 'username=first-user'
```
```json
{
    "custom_id":"10",
    "created_at":1538205455,
    "username":"first-user",
    "id":"24d9cf29-f6e0-40d6-a1ec-7fb00161ee50"
}
```

### 更新或创建Consumer
#### Endpoint
```plain
PUT /consumers/{username or id}
```

#### Request Body
参考__创建Consumer__部分，username与custom\_id属性

* 用body中指定的定义在请求的资源下插入(或替换)使用者，消费者将通过用户名或id属性进行标识
* 当用户名或id属性具有UUID结构时，被插入/替换的使用者将由其id标识，否则将由其用户名标识
* 在不指定id(既不在URL中也不在body中)的情况下创建新使用者时，将自动生成它
* 在URL中指定的用户名和请求body中不允许不同

#### Response
参考 POST 和 PATCH 的Response

### 删除Consumer
#### Endpoint
```plain
DELETE /consumers/{username or id}
```
要删除消费者的唯一标识符或名称

#### Response
```plain
curl -i -X DELETE http://localhost:8001/consumers/first-user
```

## Plugin Object
插件实体表示将在HTTP请求/响应生命周期中执行的插件配置，它向运行在Kong后面的服务添加功能，例如身份验证或限流，更多插件见[文档](https://docs.konghq.com/hub/)。

当向服务添加插件配置时，客户端向该服务发出的每个请求都将运行该插件。如果某个插件需要为某些特定的消费者调整到不同
的值，可以通过指定consumer\_id值来实现

### 优先了解
插件每次请求只运行一次，但是它运行的配置取决于它所配置的实体。插件可以为各种实体、实体组合甚至全局配置。这很有用，例如，当希望为大多数请求配置插件，但是身份验证请求的行为略有不同时。
因此，当插件被应用到具有不同配置的不同实体时，就存在运行插件的优先顺序。经验法则是：插件配置的实体越多，它的优先级就越高。
当插件被多次配置时，优先级的完整顺序是：
* 配置在一个组合上的插件：路由、服务和消费者（消费者表示请求必须经过身份验证）
* 在路由和消费者的组合上配置的插件（消费者表示请求必须经过身份验证）
* 在服务和消费者的组合上配置的插件（消费者表示请求必须经过身份验证）
* 在路由和服务的组合上配置的插件
* 在消费者上配置的插件（消费者表示请求必须经过身份验证）
* 在路由上配置的插件
* 在服务上配置的插件
* 全局运行的插件

比如：如果限流插件应用两次（不同配置）：服务（插件配置A）和消费者（插件配置B），然后验证该消费者的请求将运行插件配置B并忽略A。但是，如果请求没有验证此消费者身份，则回退运行插件配置A。
注意：如果配置B被禁用(它的启用标志被设置为false)，config A will apply to requests that would have otherwise matched config B.（这段英文不理解。。。）

### 添加Plugin
添加插件的几种方式：
- 对每个Service/Route 和 Consumer：不要设置consumer_id并同时设置service_id或route_id
- 对每个Service/Route 和 特定的Consumer：只设置consumer_id
- 对每个Consumer 和 特定Service：只设置service_id（有的插件只能设置route_id）
- 对每个Consumer 和 特定Route：只设置route_id（有的插件只能设置service_id）
- 对特定的Service/Route 和 Consumer：同时设置 service_id/route_id 及 consumer_id

注意：并不是所有的插件都允许设置特定的consumer_id，以插件文档为准。

#### Endpoint
```
POST /plugins/
```

```
POST /services/{service}/plugins
```

#### Request Body

| 属性 | 描述 |
| --- | --- |
| name | 要添加的插件名称。目前插件必须单独安装在每个Kong实例中。 |
| consumer_id | consumer的唯一标识符，在传入请求时覆盖此特定使用者的现有设置。 |
| service_id | 服务的唯一标识符，在传入请求时覆盖此特定服务的现有设置 |
| route_id | 路由的唯一标识符，在传入请求时覆盖此特定路由的现有设置 |
| config.{property} | 插件的配置属性可以在插件文档页面的[Kong Hub](https://docs.konghq.com/hub/)找到 |
| enabled | 是否运用插件。默认为 true |

#### Response

```
curl -i -X POST http://localhost:8001/plugins/ \
    -d "name=jwt" \
    -d "route_id=e193d63e-3340-4c6a-b95f-601f3d4042b3"
```

```
{
    "created_at":1539075988000,
    "config":{
        "secret_is_base64":false,
        "key_claim_name":"iss",
        "cookie_names":{

        },
        "maximum_expiration":0,
        "anonymous":"",
        "run_on_preflight":true,
        "uri_param_names":[
            "jwt"
        ]
    },
    "id":"934b2a41-ae0d-48dd-94e4-c1f4fb36be85",
    "enabled":true,
    "route_id":"e193d63e-3340-4c6a-b95f-601f3d4042b3",
    "name":"jwt"
}
```

### 检索Plugin
#### Endpoint
```
/plugins/{id}
```

#### Request Querystring Parameters
id：要检索插件的唯一标识符

#### Response

```
curl -i -X GET http://localhost:8001/plugins/934b2a41-ae0d-48dd-94e4-c1f4fb36be85
```

```
{
    "created_at":1539075988000,
    "config":{
        "secret_is_base64":false,
        "key_claim_name":"iss",
        "cookie_names":{

        },
        "maximum_expiration":0,
        "anonymous":"",
        "run_on_preflight":true,
        "uri_param_names":[
            "jwt"
        ]
    },
    "id":"934b2a41-ae0d-48dd-94e4-c1f4fb36be85",
    "name":"jwt",
    "enabled":true,
    "route_id":"e193d63e-3340-4c6a-b95f-601f3d4042b3"
}
```

### 列出所有Plugins
#### Endpoint

```
/plugins/
```

#### Request Querystring Parameters

| 属性 | 描述 |
| --- | --- |
| id | 一个基于id字段的列表过滤器 |
| name |  |
| service_id |  |
| route_id |  |
| consumer_id |  |
| size |  |
| offset |  |

#### Response

```
curl -i -X GET http://localhost:8001/plugins/ \
    -d "size=1"
```

```
{
    "total":1,
    "data":[
        {
            "created_at":1539075988000,
            "config":{
                "secret_is_base64":false,
                "key_claim_name":"iss",
                "cookie_names":{

                },
                "maximum_expiration":0,
                "anonymous":"",
                "run_on_preflight":true,
                "uri_param_names":[
                    "jwt"
                ]
            },
            "id":"934b2a41-ae0d-48dd-94e4-c1f4fb36be85",
            "name":"jwt",
            "enabled":true,
            "route_id":"e193d63e-3340-4c6a-b95f-601f3d4042b3"
        }
    ]
}
```

### 更新Plugin
#### Endpoint

```
PATCH /plugins/{plugin id}
```

plugin id：要更新插件配置的唯一标识符

#### Request Body
与添加时的配置相同

#### Response

```
curl -i -X PATCH http://localhost:8001/plugins/934b2a41-ae0d-48dd-94e4-c1f4fb36be85 \
    -d "enabled=false"
```

```
{
    "created_at":1539075988000,
    "config":{
        "key_claim_name":"iss",
        "cookie_names":{

        },
        "secret_is_base64":false,
        "anonymous":"",
        "maximum_expiration":0,
        "run_on_preflight":true,
        "uri_param_names":[
            "jwt"
        ]
    },
    "id":"934b2a41-ae0d-48dd-94e4-c1f4fb36be85",
    "enabled":false,
    "route_id":"e193d63e-3340-4c6a-b95f-601f3d4042b3",
    "name":"jwt"
}
```

### 更新或添加Plugin
#### Endpoint

```
PUT /plugins/
```

#### Request Body
与添加时的配置相同
经过实践，created_at同样需要

PUT endpoint 的行为如下：如果请求payload不包含实体的主键(插件的id和名称)，那么将使用给定的payload创建实体。如果请求payload确实包含实体的主键，那么payload将“替换”给定主键指定的实体。如果主键不是现有实体的主键，将返回404 not FOUND

#### Response

```
curl -i -X PUT http://localhost:8001/plugins/ \
    -d "id=934b2a41-ae0d-48dd-94e4-c1f4fb36be85" \
    -d "name=jwt" \
    -d "route_id=e193d63e-3340-4c6a-b95f-601f3d4042b3" \
    -d "enabled=true" \
    -d "created_at=1539075988000"
```

```
{
    "created_at":1539075988000,
    "config":{
        "secret_is_base64":false,
        "key_claim_name":"iss",
        "cookie_names":{

        },
        "maximum_expiration":0,
        "anonymous":"",
        "run_on_preflight":true,
        "uri_param_names":[
            "jwt"
        ]
    },
    "id":"934b2a41-ae0d-48dd-94e4-c1f4fb36be85",
    "enabled":true,
    "route_id":"e193d63e-3340-4c6a-b95f-601f3d4042b3",
    "name":"jwt"
}
```

### 删除Plugin
#### Endpoint

```
DELETE /plugins/{plugin id}
```

#### Response

```
curl -i -X DELETE http://localhost:8001/plugins/934b2a41-ae0d-48dd-94e4-c1f4fb36be85
```

### 检索启用Plugins
检索Kong节点上所有已安装插件的列表

#### Endpoint

```
/plugins/enabled
```

#### Response

```
curl -i -X GET http://localhost:8001/plugins/enabled
```

```
{
    "enabled_plugins":[
        "response-transformer",
        "oauth2",
        "acl",
        "correlation-id",
        "pre-function",
        "jwt",
        "cors",
        "ip-restriction",
        "basic-auth",
        "key-auth",
        "rate-limiting",
        "request-transformer",
        "http-log",
        "file-log",
        "hmac-auth",
        "ldap-auth",
        "datadog",
        "tcp-log",
        "zipkin",
        "post-function",
        "request-size-limiting",
        "bot-detection",
        "syslog",
        "loggly",
        "azure-functions",
        "udp-log",
        "response-ratelimiting",
        "aws-lambda",
        "statsd",
        "prometheus",
        "request-termination"
    ]
}
```

### 检索Plugin Schema
检索插件配置的模式。这有助于理解插件接受哪些字段，并可用于构建到Kong插件系统的第三方集成

#### Endpoint

```
/plugins/schema/{plugin name}
```

#### Response

```
curl -i -X GET http://localhost:8001/plugins/schema/jwt
```

```
{
    "fields":{
        "secret_is_base64":{
            "default":false,
            "type":"boolean"
        },
        "cookie_names":{
            "default":{

            },
            "type":"array"
        },
        "key_claim_name":{
            "default":"iss",
            "type":"string"
        },
        "anonymous":{
            "default":"",
            "func":"function",
            "type":"string"
        },
        "claims_to_verify":{
            "type":"array",
            "enum":[
                "exp",
                "nbf"
            ]
        },
        "maximum_expiration":{
            "default":0,
            "func":"function",
            "type":"number"
        },
        "run_on_preflight":{
            "default":true,
            "type":"boolean"
        },
        "uri_param_names":{
            "default":[
                "jwt"
            ],
            "type":"array"
        }
    },
    "no_consumer":true,
    "self_check":"function"
}
```

## Certificate Object
证书对象表示SSL证书的公共证书/私钥对，Kong使用这些对象来处理加密请求的SSL/TLS，证书有选择地与SNI对象关联，以便将证书/密钥对绑定到一个或多个主机名

### 添加Certificate
### 检索Certificate
### Certificates列表
### 更新Certificate
### 更新或创建Certificate
### 删除Certificate

## SNI Objects
SNI对象表示主机名到证书的多对一映射，也就是说，证书对象可以有许多与之关联的主机名；当Kong收到SSL请求时，它使用Client Hello中的SNI字段去查找与SNI关联证书的证书对象。

### 添加SNI
### 检索SNI
### SNIs 列表
### 更新SNI
### 更新或创建SNI
### 删除SNI

## Upstream Objects
上游对象表示虚拟主机名，可用于在多个服务（targets）上负载均衡传入请求。比如host是service.v1.xyz的服务对象，其上游名为service.v1.xyz。此服务的请求将被代理到上游中定义的targets。
上游还包括[健康检查](https://docs.konghq.com/0.14.x/health-checks-circuit-breakers/)，能够根据是否有能力去处理请求来启用和禁用targets——健康检查的配置存储在上游对象中，并应用于其所有目标。

### 添加upstream
#### Endpoint
```plain
POST /upstreams/
```

#### Request Body
> 参考原文：[https://docs.konghq.com/0.14.x/admin-api/#add-upstream](https://docs.konghq.com/0.14.x/admin-api/#add-upstream)


| 属性                                         | 描述                                                         |
| -------------------------------------------- | ------------------------------------------------------------ |
| name                                         | 这是一个hostname，它必须等于服务的host                       |
| slots                                        | 负载均衡算法中的槽数（10-65536，默认为1000）——接口返回的是10000 |
| hash_on                                      | 使用什么作为散列输入：none、consumer、ip、header或cookie（默认为none，导致加权循环模式） |
| hash_fallback                                | 如果主hash_on不返回哈希值，应该使用什么作为哈希输入（比如缺少header，或者没有标识消费者）。其中之一：none、consumer、ip、header或cookie（默认为none，如果hash_on设置为cookie，则不可用） |
| hash_on_header                               | 要将值作为散列输入的头名称（只有当hash_on被设置为header时才需要） |
| hash_fallback_header                         | 要将值作为散列输入的头名称（只有当hash_fallback设置为header时才需要） |
| hash_on_cookie                               | 将值作为散列输入的cookie名称（只有当hash_on或hash_fallback设置为cookie时才需要）。如果指定的cookie不在请求中，Kong将生成一个值并在响应中设置cookie |
| hash_on_cookie_path                          | 要在响应标头中设置的cookie路径（仅当hash_on或hash_fallback设置为cookie时才需要，默认值为"/"） |
| healthchecks.active.timeout                  | 用于主动健康检查的套接字超时(以秒为单位)                     |
| healthchecks.active.concurrency              | 在主动健康检查中要同时检查的目标数量                         |
| healthchecks.active.http_path                | 在GET HTTP请求中使用的路径，以探测主动健康检查               |
| healthchecks.active.healthy.interval         | 主动健康检查与健康目标之间的间隔（以秒为单位），值为0表示不应该执行针对健康目标的主动探测 |
| healthchecks.active.healthy.http_statuses    | 一个HTTP状态数组，用于表明成功，表示在主动健康检查中由探测返回时的健康状态 |
| healthchecks.active.healthy.successes        | 在主动探测(由healthcheck.active.health.http_statuses定义)中目标健康的成功次数 |
| healthchecks.active.unhealthy.interval       | 主动检查不健康目标的间隔时间(以秒为单位)，值为0表示不健康目标的主动探测不应该执行 |
| healthchecks.active.unhealthy.http_statuses  | 一个HTTP状态数组，用于表明失败，在主动健康检查中由探测返回时表示不健康 |
| healthchecks.active.unhealthy.tcp_failures   | 在主动探测中TCP失败的数量以认为目标的不健康                  |
| healthchecks.active.unhealthy.timeouts       | 在主动探测中考虑不健康目标的超时次数                         |
| healthchecks.active.unhealthy.http_failures  | 主动探测中的HTTP失败次数(由healthcheck.active.unhealth.http_statuses定义)以认为目标不健康 |
| healthchecks.passive.healthy.http_statuses   | 一种HTTP状态数组，当代理流量产生时，它表示健康状态，通过被动健康检查可以观察到 |
| healthchecks.passive.healthy.successes       | 代理流量(由healthcheck.passive.health.http_statuses定义)中认为目标健康的成功次数，通过被动健康检查观察 |
| healthchecks.passive.unhealthy.http_statuses | 一种HTTP状态数组，它表示代理流量产生的不健康状态，如被动健康检查所观察到的 |
| healthchecks.passive.unhealthy.tcp_failures  | 通过被动健康检查可以观察到，代理流量中TCP失败的数量，以考虑不健康的目标 |
| healthchecks.passive.unhealthy.timeouts      | 通过被动的健康检查，在代理流量中考虑不健康目标的超时次数     |
| healthchecks.passive.unhealthy.http_failures | 被代理的流量(由healthcheck.passive.unhealth.http_statuses定义)中认为目标不健康的HTTP故障数，由被动健康检查观察到 |

#### Response
```plain
curl -i -X POST http://localhost:8001/upstreams/ \
    -d 'name=service.v1.xyz'
```
```json
{
    "healthchecks":{
        "active":{
            "unhealthy":{
                "http_statuses":[
                    429,
                    404,
                    500,
                    501,
                    502,
                    503,
                    504,
                    505
                ],
                "tcp_failures":0,
                "timeouts":0,
                "http_failures":0,
                "interval":0
            },
            "http_path":"/",
            "healthy":{
                "http_statuses":[
                    200,
                    302
                ],
                "interval":0,
                "successes":0
            },
            "timeout":1,
            "concurrency":10
        },
        "passive":{
            "unhealthy":{
                "http_failures":0,
                "http_statuses":[
                    429,
                    500,
                    503
                ],
                "tcp_failures":0,
                "timeouts":0
            },
            "healthy":{
                "successes":0,
                "http_statuses":[
                    200,
                    201,
                    202,
                    203,
                    204,
                    205,
                    206,
                    207,
                    208,
                    226,
                    300,
                    301,
                    302,
                    303,
                    304,
                    305,
                    306,
                    307,
                    308
                ]
            }
        }
    },
    "created_at":1538215964826,
    "hash_on":"none",
    "id":"d878123e-e295-4801-a231-912bcd3148d5",
    "hash_on_cookie_path":"/",
    "name":"service.v1.xyz",
    "hash_fallback":"none",
    "slots":10000
}
```

### 检索upstream
#### Endpoint
```plain
/upstreams/{name or id}
```
要检索的upstream的标识符或名称

#### Response
```plain
curl -i -X GET http://localhost:8001/upstreams/service.v1.xyz
```
```json
{
    "healthchecks":{
        "active":Object{...},
        "passive":Object{...}
    },
    "created_at":1538215964826,
    "hash_on":"none",
    "id":"d878123e-e295-4801-a231-912bcd3148d5",
    "hash_on_cookie_path":"/",
    "name":"service.v1.xyz",
    "hash_fallback":"none",
    "slots":10000
}
```

### upstreams列表
#### Endpoint
```plain
/upstreams/
```

#### Request Querystring Parameters

| 属性 | 描述 |
| --- | --- |
| id | 一个基于上游id字段的列表过滤器 |
| name | 一个基于上游名称字段的列表过滤器 |
| hash\_on | 一个基于上游hash\_on字段的列表过滤器 |
| hash\_fallback | 列表中的一个基于上游hash\_fallback字段的过滤器 |
| hash\_on\_header |  |
| hash\_fallback\_header |  |
| slots |  |
| size |  |
| offset |  |


#### Response
```plain
curl -i -X GET http://localhost:8001/upstreams?size=1
```
```json
{
    "next":"http://localhost:8001/upstreams?offset=Mg%3D%3D&size=1",
    "total":2,
    "data":[
        {
            "healthchecks":Object{...},
            "created_at":1538215964826,
            "hash_on":"none",
            "id":"d878123e-e295-4801-a231-912bcd3148d5",
            "hash_on_cookie_path":"/",
            "name":"service.v1.xyz",
            "hash_fallback":"none",
            "slots":10000
        }
    ],
    "offset":"Mg=="
}
```

### 更新upstream
#### Endpoint
```plain
PATCH /upstreams/{name or id}
```
要更新的上游标识符或名称

#### Request Body
参考__添加upstream__部分

#### Response
```plain
curl -i -X PATCH http://localhost:8001/upstreams/service.v1.xyz \
    -d 'slots=2000'
```
```json
{
    "healthchecks":Object{...},
    "created_at":1538215964826,
    "hash_on":"none",
    "id":"d878123e-e295-4801-a231-912bcd3148d5",
    "hash_on_cookie_path":"/",
    "name":"service.v1.xyz",
    "hash_fallback":"none",
    "slots":2000
}
```

### 更新或创建upstream
#### Endpoint
```plain
PUT /upstreams/
```

#### Request Body
参考__添加upstream__部分，除此之外，还需要更多参数，可见__Response__部分

PUT Endpoint的行为如下：如果请求payload不包含实体的主键（上游id），实体将使用给定的payload创建。如果请求payload确实包含实体的主键，那么有效负载将“替换”给定主键指定的实体。如果主键不是现有实体的主键，将返回404 not FOUND

#### Response
```plain
curl -i -X PUT http://localhost:8001/upstreams/ \
    -d 'name=service.v1.xyza'
```
可参考POST 和 PATCH的响应
```plain
curl -i -X PUT http://localhost:8001/upstreams/ \
    -d 'name=service.v1.xyzab' \
    -d 'id=0ce1e400-aba5-4c66-8e83-309fac8f6bcc'
```
```json
{
    "created_at":"created_at is required"
}
```

### 删除upstream
#### Endpoint
```plain
DELETE /upstreams/{name or id}
```
要删除的上游标识符或名称

#### Response
```plain
curl -i -X DELETE http://localhost:8001/upstreams/service.v1.xyza
```

### 显示节点的upstream健康情况
根据特定的Kong节点的透视图，显示给定上游的所有目标的健康状态。注意，作为特定于节点的信息，向Kong集群的不同节点发出相同的请求可能会产生不同的结果。例如，Kong集群的一个特定节点可能遇到网络问题，导致它无法连接到某些Targets：这些Targets将被该节点标记为不健康（将流量从该节点定向到它能够成功到达的其他目标），但对所有其他Kong节点（使用该目标没有任何问题）都是健康的。
响应的数据字段包含Target对象数组，每个Target的health在其health字段中返回：
* 如果一个Target由于DNS问题而无法在环平衡器中激活，它的状态将显示为DNS\_ERROR
* 当上游配置中没有启用健康检查时，活动Target的健康状态显示为HEALTHCHECKS\_OFF
* 当启用健康检查并确定Target为健康时，无论是自动或手动，其状态显示为HEALTHY。这意味着该Target目前包含在上游的负载均衡器环中
* 当Target被主动或被动的健康检查(断路器)或手动关闭时，其状态显示为UNHEALTHY。负载均衡器没有通过上游将任何流量定向到此目标

#### Endpoint
```plain
/upstreams/{name or id}/health/
```
显示Target健康状态的上游标识符或名称

#### Response
```plain
curl -i -X GET http://localhost:8001/upstreams/service.v1.xyz/health/
```

## Target Object
target是带有一个端口的ip地址/主机名，该端口标识后端服务的实例。每个upstream都可以有多个target，并且可以动态添加target，并即时改变。
由于upstream维护target更改的历史记录，因此无法删除或修改target。要禁用一个target，发布一个新的权重为0的target；或者，使用DELETE便利方法来完成。

### 添加target
#### Endpoint
```plain
POST /upstreams/{name or id}/targets
```
name or id：要添加target的upstream标识符或名称

#### Request Body
| 属性 | 描述 |
| --- | --- |
| target | target地址(ip或主机名)和端口。如果省略，端口默认为8000。如果主机名解析为SRV记录，则端口值将被dns记录中的值覆盖。 |
| weight | 此target在upstream负载均衡中获得的权重（0-1000，默认为100）。如果主机名解析为SRV记录，则权重值将被dns记录中的值覆盖 |

#### Response
```
curl -i -X POST http://localhost:8001/upstreams/service.v1.xyz/targets \
    -d 'target=localhost:3000'
```

```
{
    "created_at":1538978616921,
    "weight":100,
    "upstream_id":"d878123e-e295-4801-a231-912bcd3148d5",
    "target":"localhost:3000",
    "id":"aa716c26-a250-43b4-a767-610562416e18"
}
```

### targets列表
列出upstream负载均衡轮上当前活动的所有目标

#### Endpoint
```
/upstreams/{name or id}/targets
```
name or id：要列出target的upstream标识符或名称

#### Request Querystring Parameters
| 属性 | 描述 |
| --- | --- |
| id | 一个基于target id字段的列表过滤器 |
| target |  |
| weight |  |
| size |  |
| offset |  |

#### Response
```
curl -i http://localhost:8001/upstreams/service.v1.xyz/targets
```

```
{
    "data":[
        {
            "created_at":1538979123961,
            "id":"17193554-f367-49f9-8188-6483774219d0",
            "upstream_id":"d878123e-e295-4801-a231-912bcd3148d5",
            "target":"localhost:3001",
            "weight":300
        },
        {
            "created_at":1538978616921,
            "id":"aa716c26-a250-43b4-a767-610562416e18",
            "upstream_id":"d878123e-e295-4801-a231-912bcd3148d5",
            "target":"localhost:3000",
            "weight":100
        }
    ],
    "total":2
}
```

### 所有targets列表
列出上游的所有target。可以返回同一target的多个target对象，显示特定target的更改历史。最新created_at的target对象是当前定义。

#### Endpoint
```
/upstreams/{name or id}/targets/all/
```
name or id：要列出target的upstream标识符或名称

#### Response
```
curl -i http://localhost:8001/upstreams/service.v1.xyz/targets/all/
```

```
{
    "total":3,
    "data":[
        {
            "created_at":1538978616921,
            "id":"aa716c26-a250-43b4-a767-610562416e18",
            "upstream_id":"d878123e-e295-4801-a231-912bcd3148d5",
            "target":"localhost:3000",
            "weight":100
        },
        {
            "created_at":1538979123961,
            "id":"17193554-f367-49f9-8188-6483774219d0",
            "upstream_id":"d878123e-e295-4801-a231-912bcd3148d5",
            "target":"localhost:3001",
            "weight":300
        },
        {
            "created_at":1538979099139,
            "id":"c266b0b9-f56e-4531-9897-6c7ec7578627",
            "upstream_id":"d878123e-e295-4801-a231-912bcd3148d5",
            "target":"localhost:3001",
            "weight":100
        }
    ]
}
```

### 删除target
禁用负载均衡器中的target。Under the hood，该方法为给定的target定义创建一个权重为0的新入口

#### Endpoint
```
DELETE /upstreams/{upstream name or id}/targets/{target or id}
```

| 属性 | 描述 |
| --- | --- |
| upstream name or id | 要删除target的上游标识符或名称 |
| target or id | 要删除的target的主机/端口组合元素，或现有目标条目的id |

#### Response
```
curl -i -X DELETE http://localhost:8001/upstreams/service.v1.xyz/targets/17193554-f367-49f9-8188-6483774219d0/
```

此时再查看所有target列表，发现有一个weight:0的记录进行了覆盖：
```
{
    "total":4,
    "data":[
        {
            "created_at":1538980110838,
            "id":"547eef27-6d23-46d6-af81-370dbacef819",
            "upstream_id":"d878123e-e295-4801-a231-912bcd3148d5",
            "target":"localhost:3001",
            "weight":0
        },
        {
            "created_at":1538978616921,
            "id":"aa716c26-a250-43b4-a767-610562416e18",
            "upstream_id":"d878123e-e295-4801-a231-912bcd3148d5",
            "target":"localhost:3000",
            "weight":100
        },
        {
            "created_at":1538979123961,
            "id":"17193554-f367-49f9-8188-6483774219d0",
            "upstream_id":"d878123e-e295-4801-a231-912bcd3148d5",
            "target":"localhost:3001",
            "weight":300
        },
        {
            "created_at":1538979099139,
            "id":"c266b0b9-f56e-4531-9897-6c7ec7578627",
            "upstream_id":"d878123e-e295-4801-a231-912bcd3148d5",
            "target":"localhost:3001",
            "weight":100
        }
    ]
}
```

### Set target as healthy
在整个Kong集群中，将负载均衡器中target的当前健康状态设置为“健康”。
此endpoint可用于手动重新启用以前由upstream的健康检查器禁用的target。Upstream只将请求转发给健康的节点，因此这个调用告诉Kong重新开始使用这个target。
这将重置在Kong节点的所有workers中运行的健康检查程序的健康计数器，并广播一个集群范围的消息，以便将“健康”状态传播到整个Kong集群。

#### Endpoint
```
POST /upstreams/{upstream name or id}/targets/{target or id}/healthy
```

| 属性 | 描述 |
| --- | --- |
| upstream name or id | upstream的唯一标识符或名称 |
| target or id | 要设置为健康的target的主机/端口组合元素，或现有目标条目的id |

### Set target as unhealthy
在整个Kong集群中，将负载均衡器中target的当前健康状态设置为“不健康”。
此endpoint可用于手动禁用target并停止对请求的响应。上行流只将请求转发给健康的节点，因此这个调用告诉Kong开始跳过环平衡器算法中的这个target。
这将重置在Kong节点的所有workers中运行的健康检查程序的健康计数器，并广播一个集群范围的消息，以便将“不健康”状态传播到整个Kong集群。
[积极健康检查](https://docs.konghq.com/0.14.x/health-checks-circuit-breakers/#active-health-checks)继续执行不健康的目标。注意：如果激活了active health check，并且探测检测到target实际上是健康的，那么它将自动重新启用。如果希望永久从环平衡器中删除target，可以使用[删除target](#删除target)

#### Endpoint
```
POST /upstreams/{upstream name or id}/targets/{target or id}/unhealthy
```


## 问题
### service
是每个上游服务的抽象，通过路由匹配进行转发
Service可以是一个实际地址；也可以指向一个Kong内部的upstream

### route
路由定义规则以匹配客户端请求，每个路由都与服务相关联；而服务可能会有多个路由与之相关联

### upstream
当我们部署集群时，可以使用upstream来进行负载均衡

### target
target就是在使用upstream时进行负载均衡的终端

### 流程
#### 普通使用
Route => Service

#### 使用upstream
Route => Service => Upstream => Target 

### 关系
#### service 和 route
1 对 n

#### upstream 和 target
1 对 n

## 资料
* [https://docs.konghq.com/0.14.x/admin-api/](https://docs.konghq.com/0.14.x/admin-api/)

