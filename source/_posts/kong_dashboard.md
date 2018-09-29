---
title: Kong——Dashboard
date: 2018/09/28 19:00:00
tags:
  - Kong
  - kong-dashboard
  - konga
categories: 
  - [微服务]
  - [Kong]
---

> 虽然Kong提供了REST API用于配置，但是平时可能更习惯使用可视化界面操作，而且可视化界面在浏览上也更友好一点。

## RESTful API
> 如果使用Kong提供的RESTful API 进行配置，需要了解各种HTTP方法

常用的HTTP 方法：

- GET：从服务器取出资源
- POST：在服务器新建一个资源
- PUT：在服务器更新资源（提供完整属性）
- PATCH：在服务器更新资源（只需要提供改变的属性）
- DELETE：从服务器删除资源

## Kong UI选择
有两个使用数较多的Dashboard：
* kong-dashboard：[https://github.com/PGBI/kong-dashboard](https://github.com/PGBI/kong-dashboard)
* konga：[https://github.com/pantsel/konga](https://github.com/pantsel/konga)
从github上对比，Stars等konga会略逊一筹，但是较于kong-dashboard，它更紧跟Kong的版本，先尝试一下konga

<!-- more -->

## konga

### 运行
```powershell
docker run --rm -d -p 1337:1337 \
     --network kong-net \
     --name konga \
     -e "NODE_ENV=production" \
     pantsel/konga
```

### 激活
进入__Connections__点击__ACTIVATE__

![image.png | left | 827x403](https://cdn.nlark.com/yuque/0/2018/png/92822/1538117832597-93962bed-b4d5-4ea0-8872-7ba0c5982528.png "")



![image.png | left | 827x264](https://cdn.nlark.com/yuque/0/2018/png/92822/1538117854758-67870fa3-ffe1-48a8-aa57-d69bd26f4c97.png "")


## kong-dashboard
> [https://github.com/PGBI/kong-dashboard/issues/156](https://github.com/PGBI/kong-dashboard/issues/156)

### 运行
```powershell
docker run --rm -p 8080:8080 --name kong-dashboard --network=kong-net pgbi/kong-dashboard start --kong-url http://kong:8001
```

### 访问


![image.png | left | 827x400](https://cdn.nlark.com/yuque/0/2018/png/92822/1538114392480-37ee2500-5e6e-4af7-b716-917050623d2d.png "")



![image.png | left | 827x198](https://cdn.nlark.com/yuque/0/2018/png/92822/1538114443068-b1e809aa-3025-4887-acbb-2eaafc43aef8.png "")

## 资料
* [https://hub.docker.com/r/pgbi/kong-dashboard/](https://hub.docker.com/r/pgbi/kong-dashboard/)
* [https://github.com/pantsel/konga](https://github.com/pantsel/konga)
* [https://github.com/PGBI/kong-dashboard](https://github.com/PGBI/kong-dashboard)