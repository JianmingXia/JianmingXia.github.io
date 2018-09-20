---
title: 如何选择 API Gateway
date: 2018/09/20 20:00:00
tags:
  - API Gateway
categories: 微服务
---

## 说在前面
最近在参与项目的重构工作，由于之前的经验问题，导致目前的项目服务架构极其不合理。既然选择了重构，不仅仅是代码层面，架构上也是需要调整的。但是在选择的API网关上遇到了难题，之前团队也没有这方面的经验，目前也只能算是摸着石头过河。列举了一些在选择API Gateway时需要注意的内容，大家可以根据需要设定考虑API Gateway时的优先级，然后进行比对选择。

## 部署简单
当我们需要创建新实例时，需要足够简单

## 可扩展性、可维护性
* 若出现无法hold住的场景，是否能进行二次开发
* 团队其它成员能否有能力维护

## 高可用性
* 网关必须支持集群部署
<!-- more -->

## 功能需求
### Authentication 与 Authorization
接入认证与授权是否方便

### 高性能
比如系统采用非阻塞的IO

### 安全
比如SQL注入、限流

#### Rate-Limiting
比如可以限制第三方接口调用次数

### 日志监控
* 是否可以产出报告，或是接入其它日志服务
* 注入一个request\_id，达到日志更好的效果

### 其它
* 当后端服务不可用时，能否提供有意义的响应

## 定制
* 有定制功能优先
* 但如果需要安装大量插件才能使用，需要评估

## 简易升级
确保可以用到最新的改进

## 生命力
确保这个API Gateway在一定时间内是能得到支持的，可以关注社区的活跃性

## 其它关注点
### 网关支持API访问，可以动态更新某些配置

### 黑白名单功能

### 能不能扩展支持gRPC、Thrift这类序列化框架

### 良好的监控体系
监控网关本身的，比如与Prometheus契合如何

## 可选方案
- Kong
- Netflix Zuul
- Express-Gateway
- Ambassador
- tyk

## 资料
- https://medium.com/@craig552uk/how-to-choose-an-api-gateway-ac64f04f3683
- https://tyk.io/blog/7-critical-factors-selecting-api-management-layer/