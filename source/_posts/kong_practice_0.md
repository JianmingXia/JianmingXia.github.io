---
title: Kong——负载均衡
date: 2018/10/08 20:00:00
tags:
  - API Gateway
  - Kong
categories: 
  - [微服务]
  - [Kong]
---

## 说在前面
这段时间看了Kong的官方文档，按照文档的接口也做了一些实践，安装了Kong及对应的UI页面，这次打算在Kong中如何做负载均衡。

## Nginx中的负载均衡

```
upstream test {
	server localhost:5000 weight=100;
	server localhost:5001 weight=50;
}

server {
	listen	80;
	location /api {
		proxy_pass http://test;
	}
}
```
<!-- more -->

## 开启服务
### 普通Node服务
为了快速测试，这里我使用了NodeJS来开启两个服务，分别监听5000及5001端口：
```
// 5000.js
var express = require('express');
var app = express();

app.get('/api', function (req, res) {
  res.send('Hello World by port 5000');
});

app.listen(5000, function () {
  console.log('app is listening at port 5000');
});
```

5001.js文件与之类似

### 以Docker方式运行
如果Kong是使用Docker运行的，此时推荐NodeJS项目也使用Docker运行，通过容器互联的方式进行访问。

```
// 构建镜像
docker build -t ryoma/5000 .

// 启动自己构建的镜像ryoma/5000
docker run -d --name ryoma-5000 --network=kong-net -p 5000:5000 ryoma/5000
```
另一个做类似处理。

### 访问
启动Node服务，根据Nginx中的配置进行访问（虽然样本比较小，但是5000出现的概率还是更高）

![根据Nginx配置访问](https://img.ryoma.top/Kong/kong_practice_0/kong_loadbalance.gif)

## 配置Upstream及Target
### 创建Upstream
名称为 test
```
curl -i -X POST http://localhost:8001/upstreams/ \
    -d 'name=test'
```

### 添加Target
不同的服务部署方式，使用不同的方式来添加Target：

#### 普通方式运行
```
curl -i -X POST http://localhost:8001/upstreams/test/targets \
    -d 'target=localhost:5000' \
    -d 'weight=100'

curl -i -X POST http://localhost:8001/upstreams/test/targets \
    -d 'target=localhost:5001' \
    -d 'weight=50'
```

![配置结果](https://img.ryoma.top/Kong/kong_practice_0/kong_target.png)

#### 以Docker方式运行
```
curl -i -X POST http://localhost:8001/upstreams/test/targets \
    -d 'target=ryoma-5000:5000' \
    -d 'weight=100'

curl -i -X POST http://localhost:8001/upstreams/test/targets \
    -d 'target=ryoma-5001:5001' \
    -d 'weight=50'
```

![配置结果](https://img.ryoma.top/Kong/kong_practice_0/kong_target_docker.png)

## 配置Service 及 Route
### 创建Service
```
curl -i -X POST http://localhost:8001/services/ \
     -d 'name=test' \
     -d 'host=test'
```

![配置结果](https://img.ryoma.top/Kong/kong_practice_0/kong_service.png)

### 配置路由
```
curl -i -X POST http://localhost:8001/routes/ \
    -d 'paths[]=/api' \
    -d 'service.id=5cb23544-b44e-4519-94e9-a9b9d8fe2a51'
```

![配置结果](https://img.ryoma.top/Kong/kong_practice_0/kong_route.png)

至此，就可以根据Route匹配规则来直接访问了
```
curl -i http://localhost:8000/api
```

![访问http://localhost:8000/api](https://img.ryoma.top/Kong/kong_practice_0/kong_result.png)

需要注意Route中Strip path默认为true，它会在匹配路由后删除匹配前缀，要根据需要进行设置，在这个案例中修改成false

## 资料

- https://docs.konghq.com/0.14.x/admin-api/
- https://www.cnkirito.moe/kong-loadbalance/
- 可视化页面使用的是KONGA：https://github.com/pantsel/konga