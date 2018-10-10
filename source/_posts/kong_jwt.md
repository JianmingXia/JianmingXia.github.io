---
title: Kong——JWT文档
date: 2018/10/10 20:00:00
tags:
  - API Gateway
  - Kong
categories: 
  - [微服务]
  - [Kong]
---


验证包含HS256或RS256签名的JSON Web令牌(如RFC 7519中指定的)的请求。每个consumer都将拥有JWT证书(公共密钥和秘密密钥)，这些密钥必须用于签名JWT。Token可以通过以下方式传输：
- URL中的参数
- cookie
- 授权Header

如果验证签名有效，Kong将把请求代理到上游服务，如果没有验证，则丢弃请求。Kong can also perform verifications on some of the registered claims of RFC 7519 (exp and nbf).
<!-- more -->

## 相关术语
- plugin：在请求被代理到上游API之前或之后，在Kong内执行操作的插件
- Service：代表外部上游API或微服务的Kong实体
- Route：代表将下游请求映射到上游服务的Kong实体
- upstream service：这指的是位于Kong后面的API/服务，客户端请求转发的位置

## 配置
### 在Service上启用插件

```
curl -X POST http://kong:8001/services/{service}/plugins \
    --data "name=jwt"
```

service：插件要配置Service的id或名称

### 在Route上启用插件

```
curl -X POST http://kong:8001/routes/{route_id}/plugins \
    --data "name=jwt" 
```

route_id：插件要配置Route的id。

### 在API上启用插件
> 目前使用的版本是0.14，API已弃用

### 全局插件
所有插件都可以使用 http://kong:8001/plugins/ endpoint进行配置。不与任何服务、路由或consumer（或API，如果使用的是较旧版本的Kong）相关联的插件被认为是“全局的”，并将在每个请求上运行。

### 参数
在JWT中可配置的参数：

| 属性 | 默认值 | 描述 |
| --- | --- | --- |
| name | | 要使用的插件的名称，在本例中是jwt |
| service_id | | 插件将针对的Service的id |
| route_id | | 插件将针对的Route的id |
| enabled | true | 插件是否会启用 |
| config.uri_param_names | jwt | 一个querystring参数列表，Kong将检查这些参数以检索JWTs |
| config.cookie_names | | 一个cookie名称的列表，Kong将检查这些cookie名称以检索JWTs |
| config.claims_to_verify | | 注册claims清单(根据[RFC 7519](https://tools.ietf.org/html/rfc7519))，Kong也可以核实。接受值:exp, nbf |
| config.key_claim_name | iss | The name of the claim in which the key identifying the secret must be passed. Starting with version 0.13.1, the plugin will attempt to read this claim from the JWT payload and the header, in this order. |
| config.secret_is_base64 | false | 如果为true，则插件假定证书的secret是base64编码的。需要为Consumer创建一个base64编码的密钥，并使用原始密钥在JWT上签名 |
| config.anonymous |  | An optional string (consumer uuid) value to use as an "anonymous" consumer if authentication fails. If empty (default), the request will fail with an authentication failure 4xx. Please note that this value must refer to the Consumer id attribute which is internal to Kong, and not its custom_id. |
| config.run_on_preflight |  | 指示插件是否应该在OPTIONS预发请求上运行(并尝试验证)，如果设置为false，那么始终允许OPTIONS请求 |
| config.maximum_expiration |  | 将JWT的生命周期限制为秒数。Any JWT that has a longer lifetime will rejected (HTTP 403). If this valeu is specified, exp must be specified as well in the claims_to_verify property. The default value of 0 represents an indefinite period. Potential clock skew should be considered when configuring this value. |

## 文档
为了使用插件，首先需要创建一个Consumer，并将一个或多个JWT证书（持有用于验证令牌的公钥和私钥）关联到它。Consumer代表使用最终服务的开发人员。

### 创建Consumer
需要将证书关联到现有的Consumer对象。要创建consumer，可以执行以下请求：

```
curl -X POST http://kong:8001/consumers \
    --data "username=<USERNAME>" \
    --data "custom_id=<CUSTOM_ID>"
```

消费者可以拥有多个JWT证书

### 创建JWT证书
可以通过发出以下HTTP请求来提供新的HS256 JWT证书：

```
curl -X POST http://kong:8001/consumers/{consumer}/jwt -H "Content-Type: application/x-www-form-urlencoded"
```

consumer：将证书关联到的Consumer实体的id或用户名属性

### 删除JWT证书
可以通过发出以下HTTP请求来删除用户的JWT证书：

```
curl -X DELETE http://kong:8001/consumers/{consumer}/jwt/{id}
```

consumer：将证书关联到的Consumer实体的id或用户名属性
id：JWT证书的id

### JWT证书列表
可以通过发出以下HTTP请求来列出consumer的JWT证书：

```
curl -X GET http://kong:8001/consumers/{consumer}/jwt
```

consumer：要列出证书的Consumer实体的id或用户名属性

### 用secret打造JWT (HS256)
现在Consumer已经有了证书，假设使用HS256签名，那么JWT应该按照以下方式制作（根据RFC 7519）:

#### header

```
{
    "typ": "JWT",
    "alg": "HS256"
}
```

#### claims 
声明必须包含配置声明中的密钥（来自config.key_claim_name）。该声明是iss(发行者字段)默认的。将其值设置为之前创建的证书的key。

```
{
    "iss": "a36c3049b36249a3c9f8891cb127243c"
}
```

#### 生成Token
> https://jwt.io/

### 使用JWT发送请求
JWT现在可以包含在对Kong的请求中，将其添加到授权头中：

```
curl http://kong:8000/{route path} \
    -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJhMzZjMzA0OWIzNjI0OWEzYzlmODg5MWNiMTI3MjQzYyIsImV4cCI6MTQ0MjQzMDA1NCwibmJmIjoxNDQyNDI2NDU0LCJpYXQiOjE0NDI0MjY0NTR9.AhumfY35GFLuEEjrOXiaADo7Ae6gt_8VLwX7qffhQN4'
```

或者作为一个querystring参数，如果在config.uri_param_names(默认jwt)中配置过：

```
curl http://kong:8000/{route path}?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJhMzZjMzA0OWIzNjI0OWEzYzlmODg5MWNiMTI3MjQzYyIsImV4cCI6MTQ0MjQzMDA1NCwibmJmIjoxNDQyNDI2NDU0LCJpYXQiOjE0NDI0MjY0NTR9.AhumfY35GFLuEEjrOXiaADo7Ae6gt_8VLwX7qffhQN4
```

或者作为cookie，如果在cookie.cookie_names中配置(默认不启用)：

```
curl --cookie jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJhMzZjMzA0OWIzNjI0OWEzYzlmODg5MWNiMTI3MjQzYyIsImV4cCI6MTQ0MjQzMDA1NCwibmJmIjoxNDQyNDI2NDU0LCJpYXQiOjE0NDI0MjY0NTR9.AhumfY35GFLuEEjrOXiaADo7Ae6gt_8VLwX7qffhQN4 http://kong:8000/{route path}
```

该请求将由Kong进行审查，其行为取决于JWT的有效性：

| 请求 | 代理到上游服务 | 返回状态码 |
| --- | --- | --- |
| has no JWT | no | 401 |
| iss claim 缺失或无效 | no | 401 |
| 无效签名 | no | 401 |
| 有效签名 | yes | 由上游服务决定 |
| 有效签名，claim无效（option） | no | 403 |

### (Optional) Verified claims

### (Optional) Base64 encoded secret

### Craft a JWT with public/private keys (RS256 or ES256)

### Generate public/private keys

### Using the JWT plugin with Auth0

### Upstream Headers
当JWT有效时，consumer经过验证，插件将在其代理到上游服务追加一些头信息，以便可以在代码中识别consumer：
- X-Consumer-ID：用户在Kong的ID
- X-Consumer-Custom-ID：consumer的custom_id（如果设置）
- X-Consumer-Username：consumer的用户名（如果设置）
- X-Anonymous-Consumer：will be set to true when authentication failed, and the 'anonymous' consumer was set instead.

可以使用这些信息来实现额外的逻辑，比如可以使用X-Consumer-ID值查询Kong Admin API并检索关于consumer的更多信息

### 通过JWTs分页
可以使用以下请求通过JWTs为所有consumer分页：

```
curl -X GET http://kong:8001/jwts
```

可以使用以下查询参数过滤列表：

| 属性 | 描述 |
| --- | --- |
| id | 基于JWT证书id字段在列表上的筛选 |
| key | |
| consumer_id | |
| size | |
| offset | |

### 检索与JWT关联的consumer
可以使用以下请求检索与JWT关联的consumer：

```
curl -X GET http://kong:8001/jwts/{key or id}/consumer
```

key or id：用于获取关联consumer的JWT的id或键属性

## 资料
- https://docs.konghq.com/hub/kong-inc/jwt/