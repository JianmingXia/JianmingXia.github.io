---
title: Kong 入门
date: 2018/09/27 19:00:00
tags:
  - API Gateway
  - Kong
categories: 
  - [微服务]
  - [Kong]
---

## 说在前面
之前做了Kong与Express-Gateway的比较，毫无疑问，Kong更胜一筹；同时在另一方面，Tyk也战胜了Ambassador。Kong与Tyk进行最终角逐，在这两者之间，也没有进行过多比较，只是考虑了当前的场景：我们需要一个API Gateway，需要支持JWT、支持日志处理、方便监控，而这些Kong的插件完全支持，而且从社区活跃上看，Kong的>18K的stars也给了我更大的信心，这也是选择Kong的最终原因，并不是Tyk不够好。

## 安装
最初尝试使用CentOS安装，使用yum直接安装但是一直提示无法下载；后选择下载安装包，然后本地安装，启动时又提示需要安装数据库。考虑到安装这部分还是轻量级的比较好，所以选用了以Docker方式去安装，按照[官方文档](https://docs.konghq.com/install/docker/?_ga=2.106120899.1705340860.1537930352-980333107.1537168769)即可

### 创建Docker网络
```plain
docker network create kong-net
```

### 使用数据库
这里使用PostgreSQL：

```
docker run -d --name kong-database \
    --network=kong-net \
    -p 5432:5432 \
    -e "POSTGRES_USER=kong" \
    -e "POSTGRES_DB=kong" \
    postgres:9.6
```

<!-- more -->

### 准备数据库
初始化数据库表：

```
docker run --rm \
    --network=kong-net \
    -e "KONG_DATABASE=postgres" \
    -e "KONG_PG_HOST=kong-database" \
    -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
    kong:latest kong migrations up
```

### 启动
```
docker run -d --name kong \
    --network=kong-net \
    -e "KONG_DATABASE=postgres" \
    -e "KONG_PG_HOST=kong-database" \
    -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
    -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
    -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
    -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
    -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
    -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
    -p 8000:8000 \
    -p 8443:8443 \
    -p 8001:8001 \
    -p 8444:8444 \
    kong:latest
```

## 基本概念
### 端口
#### 代理
* 8000：监听客户端传入的HTTP流量，并将其转发至上游服务(上线后修改成80)
* 8443：与8000类似，监听HTTPS(上线后修改成443)

#### 管理API
这些端口暴露出来是Kong用于管理API，如果在生产环境请使用防火墙保护，防止未经授权的访问：

* 8001：可用于操作Kong的 Admin API
* 8444：与8001类似，使用HTTPS

## 配置服务
### 使用Admin API添加服务
```powershell
curl -i -X POST \
  --url http://localhost:8001/services/ \
  --data 'name=example-service' \
  --data 'url=http://mockbin.org'
```

#### 返回结果
```plain
HTTP/1.1 201 Created
Date: Thu, 27 Sep 2018 07:49:54 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Server: kong/0.14.1
Content-Length: 259

{"host":"mockbin.org","created_at":1538034594,"connect_timeout":60000,"id":"4bdb97b4-4735-403d-afd6-811770b8717b","protocol":"http","name":"example-service","read_timeout":60000,"port":80,"path":null,"updated_at":1538034594,"retries":5,"write_timeout":60000}
```

#### 继续访问
```powershell
curl -i http://localhost:8001/services/
```
```plain
HTTP/1.1 200 OK
Date: Thu, 27 Sep 2018 08:22:30 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Server: kong/0.14.1
Content-Length: 282

{"next":null,"data":[{"host":"mockbin.org","created_at":1538034594,"connect_timeout":60000,"id":"4bdb97b4-4735-403d-afd6-811770b8717b","protocol":"http","name":"example-service","read_timeout":60000,"port":80,"path":null,"updated_at":1538034594,"retries":5,"write_timeout":60000}]}
```

### 给服务添加一个路由
```powershell
curl -i -X POST \
  --url http://localhost:8001/services/example-service/routes \
  --data 'hosts[]=example.com'
```

#### 返回结果
```plain
HTTP/1.1 201 Created
Date: Thu, 27 Sep 2018 08:30:36 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Server: kong/0.14.1
Content-Length: 290

{"created_at":1538037036,"strip_path":true,"hosts":["example.com"],"preserve_host":false,"regex_priority":0,"updated_at":1538037036,"paths":null,"service":{"id":"4bdb97b4-4735-403d-afd6-811770b8717b"},"methods":null,"protocols":["http","https"],"id":"a4935728-b01e-436b-bf27-0ce6c73a3730"}
```

#### 继续访问
```powershell
curl -i http://localhost:8001/services/example-service/routes
```
```plain
HTTP/1.1 200 OK
Date: Thu, 27 Sep 2018 08:30:50 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Server: kong/0.14.1
Content-Length: 285

{"next":null,"data":[{"created_at":1538037036,"strip_path":true,"hosts":["example.com"],"preserve_host":false,"regex_priority":0,"updated_at":1538037036,"service":{"id":"4bdb97b4-4735-403d-afd6-811770b8717b"},"protocols":["http","https"],"id":"a4935728-b01e-436b-bf27-0ce6c73a3730"}]}
```

### 通过Kong来转发请求
```powershell
curl -i -X GET \
  --url http://localhost:8000/ \
  --header 'Host: example.com'
```

向[http://localhost:8000/](http://localhost:8000/)发起Get请求，并附加Header
返回最初配置的[http://mockbin.org](http://mockbin.org)内容=>[链接](#bo2ihd)

> 我在使用Postman去访问搭建Kong的服务器时，遇到了一个问题，简单来说就是访问失败，又从另外一台腾讯云机器访问
> 
> 
> ![image.png | left | 719x50](https://cdn.nlark.com/yuque/0/2018/png/92822/1538039841324-8ae99265-8150-4e58-bc1f-2565e703b3d6.png "")
> 
> 简单来说，就是__example.com__未备案，在本地上访问还是OK的

## 启用插件
> Kong的核心原则之一——通过插件实现可扩展性

### 配置key-auth插件
```powershell
curl -i -X POST \
  --url http://localhost:8001/services/example-service/plugins/ \
  --data 'name=key-auth'
```

#### 返回结果
```plain
HTTP/1.1 201 Created
Date: Thu, 27 Sep 2018 09:20:04 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Server: kong/0.14.1
Content-Length: 276

{"created_at":1538040005000,"config":{"key_in_body":false,"run_on_preflight":true,"anonymous":"","hide_credentials":false,"key_names":["apikey"]},"id":"153b0d83-9c5d-454f-9f3e-e392be5b830b","enabled":true,"service_id":"4bdb97b4-4735-403d-afd6-811770b8717b","name":"key-auth"}
```

### 验证插件是否配置成功
```powershell
curl -i -X GET \
  --url http://localhost:8000/ \
  --header 'Host: example.com'
```

#### 返回结果
```plain
HTTP/1.1 401 Unauthorized
Date: Thu, 27 Sep 2018 09:23:50 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
WWW-Authenticate: Key realm="kong"
Server: kong/0.14.1
Content-Length: 41

{"message":"No API key found in request"}
```

## 增加消费者
### 创建消费者
```powershell
curl -i -X POST \
  --url http://localhost:8001/consumers/ \
  --data "username=Jason"
```

#### 返回结果
```plain
HTTP/1.1 201 Created
Date: Thu, 27 Sep 2018 09:28:49 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Server: kong/0.14.1
Content-Length: 106

{"custom_id":null,"created_at":1538040529,"username":"Jason","id":"0ad63029-c2fd-4a25-ba97-433aec6edafe"}
```

#### 再试一次
```plain
HTTP/1.1 409 Conflict
Date: Thu, 27 Sep 2018 09:30:15 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Server: kong/0.14.1
Content-Length: 164

{"strategy":"postgres","message":"UNIQUE violation detected on '{username=\"Jason\"}'","name":"unique constraint violation","fields":{"username":"Jason"},"code":5}
```

### 为消费者提供凭证
```powershell
curl -i -X POST \
  --url http://localhost:8001/consumers/Jason/key-auth/ \
  --data 'key=ENTER_KEY_HERE'
```

#### 返回结果
```plain
HTTP/1.1 201 Created
Date: Thu, 27 Sep 2018 09:34:04 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Server: kong/0.14.1
Content-Length: 149

{"id":"e0376b92-613f-452c-bf80-b6e8671d16d6","created_at":1538040845000,"key":"ENTER_KEY_HERE","consumer_id":"0ad63029-c2fd-4a25-ba97-433aec6edafe"}
```

#### 再试一次
```plain
HTTP/1.1 409 Conflict
Date: Thu, 27 Sep 2018 09:34:15 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Server: kong/0.14.1
Content-Length: 53

{"key":"already exists with value 'ENTER_KEY_HERE'"}
```

### 验证凭证是否有效
在原有请求基础上，附加一个apikey header：
```powershell
curl -i -X GET \
  --url http://localhost:8000 \
  --header "Host: example.com" \
  --header "apikey: ENTER_KEY_HERE"
```

## 资料
* [https://konghq.com/install/](https://konghq.com/install/)
* https://docs.konghq.com/0.14.x/network/#proxy
* [https://docs.konghq.com/0.14.x/getting-started/configuring-a-service/](https://docs.konghq.com/0.14.x/getting-started/configuring-a-service/)
* [https://docs.konghq.com/0.14.x/getting-started/enabling-plugins/](https://docs.konghq.com/0.14.x/getting-started/enabling-plugins/)
* [https://docs.konghq.com/0.14.x/getting-started/adding-consumers/](https://docs.konghq.com/0.14.x/getting-started/adding-consumers/)