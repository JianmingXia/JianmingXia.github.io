---
title: Kong——JWT实战
date: 2018/10/10 21:00:00
tags:
  - API Gateway
  - Kong
categories: 
  - [微服务]
  - [Kong]
---

## 说在前面
基于[Kong——负载均衡](/posts/kong_practice_0)一文继续实践，集成JWT插件认证，不了解Plugin或JWT可查看：
- [Kong Admin-API](/posts/kong_admin_api/#Plugin-Object)
- [JWT介绍](/posts/kong_jwt)

## 准备工作
### 添加Service及Route

```
curl -i -X POST http://localhost:8001/services/ \
     -d 'name=jwttest' \
     -d 'url=http://ryoma-5000:5000'

curl -i -X POST http://localhost:8001/routes/ \
    -d 'paths[]=/jwttest' \
    -d 'service.id=799a9b1b-3ed9-45e4-b73a-48b2c9cd0616'
```

<!-- more -->

### 访问

```
curl -i localhost:8000/jwttest/api
```

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 24
Connection: keep-alive
X-Powered-By: Express
ETag: W/"18-O9zc/d5YYjBdXBSyfpSesvSVNnQ"
Date: Wed, 10 Oct 2018 03:19:10 GMT
X-Kong-Upstream-Latency: 1
X-Kong-Proxy-Latency: 0
Via: kong/0.14.1

Hello World by port 5000
```

## 引入JWT插件
### 给Service指定插件

```
curl -X POST http://localhost:8001/services/jwttest/plugins \
    -d "name=jwt"
```

### 访问

```
HTTP/1.1 401 Unauthorized
Date: Wed, 10 Oct 2018 03:22:44 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Server: kong/0.14.1
Content-Length: 27

{"message":"Unauthorized"}
```

## 创建Consumer
> /posts/kong_admin_api/#Consumer-Object

```
curl -i -X POST http://localhost:8001/consumers \
    -d 'custom_id=1809' \
    -d 'username=ryoma'
```

## 创建JWT证书

```
curl -X POST http://localhost:8001/consumers/ryoma/jwt
```

### Response

```
{
    "created_at":1539142931000,
    "id":"f8c6f088-f2ce-4128-b313-f799cbaf9e3c",
    "algorithm":"HS256",
    "key":"1VOVNS2DeQk0WdK8O7VEX34Dmkms873q",
    "secret":"bxSH8fJe9ayk3cBcJ33IAN9BNmurtJjQ",
    "consumer_id":"f0101b12-d00f-4ae6-9a30-b829d62f2fc0"
}
```

## 用secret打造JWT (HS256)
### header

```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### payload

```
{
  "name": "ryoma",
  "id": 1809,
  "iss": "1VOVNS2DeQk0WdK8O7VEX34Dmkms873q"
}
```

### 生成JWT
再加上secret，就可以生成JWT了

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoicnlvbWEiLCJpZCI6MTgwOSwiaXNzIjoiMVZPVk5TMkRlUWswV2RLOE83VkVYMzREbWttczg3M3EifQ._RE1bn2YyXyMcf426KcQQ5MBYz8aCi-irPQ26Usm8J8
```

## 带上JWT访问

```
curl -i localhost:8000/jwttest/api -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoicnlvbWEiLCJpZCI6MTgwOSwiaXNzIjoiMVZPVk5TMkRlUWswV2RLOE83VkVYMzREbWttczg3M3EifQ._RE1bn2YyXyMcf426KcQQ5MBYz8aCi-irPQ26Usm8J8'
```

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 24
Connection: keep-alive
X-Powered-By: Express
ETag: W/"18-O9zc/d5YYjBdXBSyfpSesvSVNnQ"
Date: Wed, 10 Oct 2018 07:01:09 GMT
X-Kong-Upstream-Latency: 0
X-Kong-Proxy-Latency: 1
Via: kong/0.14.1

Hello World by port 5000
```

## 小结
在上文中，我是在官网使用secret生成JWT的，但是在实际业务中，还是需要自建服务来处理这个。比如有多端登录、登录被挤下线等需求时，更依赖自建服务来处理了。

## 资料
- https://docs.konghq.com/hub/kong-inc/jwt/
- https://blog.ryoma.top/posts/authentication_analysis/#JWT
- [Kong Admin-API](/posts/kong_admin_api/#Plugin-Object)
- [JWT介绍](/posts/kong_jwt)