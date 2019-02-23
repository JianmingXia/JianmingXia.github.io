---
title: 负载均衡算法
date: 2018/06/26 22:30:00
tags:
  - 负载均衡
categories: 技术杂集
---

## 说在前面
我们知道，单节点的服务器性能总是有上限的，你能将服务器性能从1核1G升级到4核8G，或者更可怕一点，32核64G。虽然性能提升上去了，但是如果机房崩溃了呢？所以，一般为了保证服务的高可用，我们会选择部署多服务节点。<br />部署多服务节点随之带来的问题是：接收请求时，选择哪个节点提供服务呢？这就是今天的重点：负载均衡算法。<br />为什么要引入负载均衡算法呢：
* 考虑调用的均衡：让每个节点都能去处理请求（否则多节点的意义呢）
* 考虑调用的性能：比如哪个节点处理请求快就分配给哪个节点

## 常见的复杂均衡算法
### 随机算法
随机算法：**从可用节点中随机取出一个**
<!-- more -->

一般使用随机数实现：比如可用节点数量为3，随机生成0-2的数字，对应3个可用节点。<br />在**访问量比较大**的情况下，随机算法还是比较均衡的

这里分享一件事：我有一次在查看后端日志时，发现3个服务节点中只有一个服务节点在提供服务——后面看了源代码时发现，随机算法并不随机，只会固定返回一个节点，想想真是可怕。

### 轮询算法
轮询算法：以循环的方式依次调度不同的服务节点。

比如可用节点数量为3，每次调度i % 3 + 1节点。

轮询算法能够保证每个节点被访问的概率是相同的——无论访问量多大，每个节点提供服务的次数偏差也不会>1。

### 加权轮询算法
加权轮询算法：是在轮询算法的基础上，给每个节点赋予一定的权重，从而使每个节点被访问的概率不同。当然，权重高的节点会更高概率被访问。

加权轮询有很多实现方式：比如有3个可用节点，权重分别为1、2、3，每次调度i % (1 + 2 +3)：
* < 1：访问节点1
* < 3：访问节点2
* < 6：访问节点3

但是这个实现有个问题：比如三个节点的权重分别为1000、1100、1200，这样前1000个请求都会落到节点1

在实现加权轮询时，需要注意均衡各个节点的排列，使节点在被访问时更加均匀一点

### IP 哈希
IP 哈希：IP 哈希会根据IP地址的哈希值去选择服务节点。

IP哈希有个问题是，当某个服务节点出现故障时，会出现访问失败的情况。除了IP 哈希外，还有url哈希等

### 最少活跃连接
最少活跃连接算法：每一次访问时选择连接数更小的节点。因为不同节点处理请求的速度不同，这也导致了每个节点的连接数也不同。

最少活跃连接与前几个不同，需要额外记录跟每一个节点的连接数，这样才能在选择节点时，选择连接数最小的节点。

### 最少响应时间
最少响应时间算法：这个与最少活跃连接不同，最少响应时间算法会将请求优先转发给平均响应时间较短的服务节点。

与最少活跃连接相同的是，需要额外记录每个节点之前处理请求的平均时间，一般来说可能选择近10分钟、1小时；当然，也需要对应的刷新策略，比如1分钟更新一次等

## 如何选择
### 随机算法
实现简单，均衡程序取决于随机算法 及 请求数。在保证随机的前提下，如果请求数很高时，各服务节点被访问的概率相同，经常应用在各服务节点的性能差异不大的情况下。<br /><br />
### 轮询算法
与随机算法类似。同时，这也是Nginx默认选择的方式。

### 加权轮询算法
基于轮询算法做了一定的优化，可以对服务节点设置权重，适用于节点性能不均匀的情况。

### IP 哈希
IP 哈希适用于按物理位置区别提供服务的场景

### 最少活跃连接
当节点性能不均匀，但是又不能很好的预定权重时，可以使用最少活跃连接

### 最少响应时间
基于之前服务节点的响应速度来选择服务节点——这个经常会出现一个场景，性能最高的服务节点在最开始，会接受大部分的请求，当有了一定负载压力时，响应时间会增加，此时又会分配到另外一个节点上

## 资料
* [http://nginx.org/en/docs/http/load_balancing.html](http://nginx.org/en/docs/http/load_balancing.html)