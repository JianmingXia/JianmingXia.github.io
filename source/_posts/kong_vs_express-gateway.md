---
title: Kong VS express gateway
date: 2018/09/26 20:00:00
tags:
  - API Gateway
  - Kong
  - express-gateway
categories: 微服务
---

> 下文简写：Kong => K；express gateway => E
## 前言
在[如何选择 API Gateway](/posts/how_to_select_api_gateway)一文中，列举了选择API Gateway的一些标准。基于这个标准，对Kong和express-gateway做了一系列对比。

## 高优先级

### 高可用性

K支持多种环境，最简单的部署方式是使用Docker或K8S。支持集群，内部存储K节点、API、消费者、插件等信息，有两种存储方式，PostgreSQL及Cassandra——高可用建议使用Cassandra

<!-- more -->

E同样支持多种方式部署，支持集群，但是部分功能无法支持集群：如没有全局计算器。与K不同，E有两种方式存储数据，分别为内存数据库和Redis，一般在生产环境中，推荐使用Redis持久化数据

结论：K ≈ E

### 功能需求

#### Authentication 与Authorization

K及E在这部分的实现很类似，都是通过插件或策略的方式引入Authentication 与 Authorization

结论： K ≈ E

### 高性能

K是基于OpenResty实现的，性能上无需多言。

E是基于Express实现的，基于NodeJS的高I/O特性，性能也是可以保证，较于K，性能可能会低一点

结论： K > E

#### 安全

限流：与认证、鉴权处的类似，K与E通过插件或策略的方式来进行限流

结论： K ≈ E

#### 日志监控

K依然是同样的思路，可以通过插件转发请求，支持的日志类型有很多：File Log、HTTP Log、TCP/UDP Log等

E只有简单的日志配置，文档很匮乏

结论： K > E

## 中优先级

### 部署

都使用K8S部署，这部分差异不大

结论： K = E

### 扩展性和维护性

都支持开发者写扩展，但lua及JS的差异还是比较大的，毕竟JS目前是团队的技术栈

结论： K < E

### 定制

这两者的使用方式有点类似，K提供REST 接口或CLI的方式来定制插件；E使用YAML文件来进行配置，在灵活性上，K会略胜一筹

结论： K > E

### 生命力

截至2018.09.26 18:00:00的数据对比：

| 项目         | Kong      | express-gateway |
| ------------ | --------- | --------------- |
| Star         | 18003     | 1166            |
| Open Issue   | 253       | 53              |
| Closed Issue | 1807      | 381             |
| Commit       | 4130      | 1055            |
| Release      | 50        | 37              |
| Contributors | 116       | 14              |
| Fork         | 2102      | 114             |
| Last Commit  | 2018.9.26 | 2018.9.21       |

结论： K >> E

## 低优先级

### 支持API访问

上面提到过，K支持，E不支持；值得一提的是，E通过在启动时自动监视配置文件的修改，从而实现热重载

### 黑白名单

K支持黑白名单， E不支持

### 良好的监控体系

K在监控这部分有着丰富的插件：比如Prometheus及Zipkin

E在这部分比较弱，需要修改代码才能实现，比如集成Prometheus

### 升级

根据升级说明，重新部署即可

## 小结

总体上Kong是优于express-gateway，如果语言不是问题的话，强烈推荐使用Kong。另外在两者的官网上，无论是内容还是样式，Kong无疑要胜出太多，给人的感觉就是认真做事（抱歉，是个颜值控）。