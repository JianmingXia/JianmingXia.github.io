---
title: Redis 安装
date: 2018/07/28 15:30:00
tags:
  - Redis
categories: Redis
---

## windows 版
官方没有维护Windows版的Redis，但是[微软](https://github.com/MicrosoftArchive/redis/releases)有支持。不过很久不维护了，最近版本是3.2.100，本地调试够用了。

## Linux版本
我们要在腾讯云购买的 “社区版Redis-主从版”是基于开源Redis引擎的高性能版本，兼容Redis2.8版本，所以我在测试机上安装的也是2.8版本。
<!-- more -->
### 安装

```
wget http://download.redis.io/releases/redis-2.8.7.tar.gz
tar -zxvf redis-2.8.7.tar.gz
cd redis-2.8.7/
make
```

通过下载、解压、编译，Redis安装成功。

### 启动
![Redis 启动](https://img.ryoma.top/Redis/redis_start.png)

但是此时Redis在前台运行，通过修改redis.conf中的daemonize属性：
```
daemonize: true
```
重新启动Redis，此时Redis已在后台运行。

### 运行
可以通过内置的客户端命令redis-cli来进行操作：

![Redis 启动](https://img.ryoma.top/Redis/redis-cli.png)

## 基准测试
Redis 自带了一个叫 [redis-benchmark](http://www.redis.cn/topics/benchmarks.html) 的工具，如果对redis性能感兴趣可以试试。

## GUI 工具
如果大家喜欢用GUI工具，给大家推荐一个GUI工具——[Medis](https://github.com/luin/medis)。

## 参考资料
- https://github.com/MicrosoftArchive/redis
- http://www.redis.cn/download.html
- http://www.redis.cn/topics/benchmarks.html
- https://github.com/luin/medis