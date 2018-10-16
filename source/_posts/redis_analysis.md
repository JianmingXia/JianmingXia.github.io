---
title: Redis 引入分析
date: 2018/07/28 15:00:00
tags:
  - Redis
categories: Redis
---

> 在本文中，**数据库**与**MySQL**的意义相同

## 缓存策略分析
> 根据场景选择缓存策略，不要在一个场景下混用策略

### Cache Aside Pattern
适用场景：
- 数据非常重要
- 多读少写

<!-- more -->

流程：
- 读数据
	- 命中：从cache中取数据
	- 失效：从数据库中取数据，并将数据放到缓存中
- 写数据
	- 先让缓存失效（不要使用set，使用delete）
		- 如果成功，再操作数据库（无论操作数据库成功/失败，缓存数据都需要从数据库重新加载）
		- 失败，直接返回
	- [canal](https://github.com/alibaba/canal)：canal通过模拟MySQL slave的交互协议，将自己伪装成MySQL slave，然后通过解析MySQL master推送的binlog文件，最后再进行redis数据的同步（使用canal算是Cache Aside Pattern策略的加强，但使用Canal要引入Java技术栈，故暂不引入，后续可根据需要再做考虑）

![Cache Aside](https://img.ryoma.top/Redis/cache_aside_1.png)

可能看到这，有人觉得很奇怪，因为写数据这部分，是先让缓存失效还是后让缓存失效，是很有争议的。我原先也是打算使用后缓存失效的，直至看了这一篇[文章](https://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651961341&idx=1&sn=e27916b8e96bd771c72c055f1f53e5be&chksm=bd2d02218a5a8b37ecffd78d20b65501645ac07c7ba2eb65b7e501a3eb9de023febe63bfdb36&scene=21#wechat_redirect)：
- 类型1：先操作数据库，再操作缓存；
- 类型2：先操作缓存，再操作数据库；

但是，我们希望保证两个操作的原子性，要么同时成功，要么同时失败。如果第一步操作那还好说，就当做请求失败就有，如果第二个请求时失败，那就另说了（第二个请求失败情况分析）：
- 类型1：数据库里是新数据，而缓存里是旧数据——出现了数据不一致情况
- 类型2：缓存里没有数据，数据库里是之前的数据——对业务没有影响（记得要使用delete操作，不要使用set更新数据，否则就是：缓存里是新数据，数据库里是旧数据的情况了）

### Write Behind Caching Pattern
> Write Behind 又名 Write Back

适用场景：
- 频写且数据不太重要

流程：
更新数据的时候，只更新缓存，不同步更新数据库，能够让更新操作不受MySQL的限制。

- 读数据
	- 命中：从cache中取数据
	- 失效：从数据库中取数据，并将数据放到缓存中
- 写数据
	- 直接将数据写到缓存中
		- 写入成功，将更新操作发到消息中间件，消息中间件负责将数据更新至数据库
		- 失败，直接返回

![Cache Aside](https://img.ryoma.top/Redis/write_back.png)
上面的消息中间件是同步redis数据的一个解决方案，还有另外一个解决方案，是通过分析Redis的RDB/AOF文件，来实现数据同步的功能。工具目前发现[redis-replicator](https://github.com/leonchen83/redis-replicator)，但同样也是需要使用Java去书写。

## 数据同步分析
无论是使用消息中间件还是Canal等解决方案，合理的构建出Redis和数据库中对应的数据模型很重要，没有直接更新Redis数据就能同步到数据库的银弹，数据同步要求开发者对Redis和数据库中的数据结构胸有成竹。

## 防雪崩设计
### 缓存穿透预防
> 缓存穿透是指查询一个根本不存在的数据，缓存层和存储层都不会命中

缓存穿透会导致不存在的数据每次都要到数据库去查询，失去了缓存保护后端存储的意义。
一般造成缓存穿透的原因有两个：
- 业务自身代码/数据出现问题
- 恶意攻击、爬虫等造成大量空命中

#### 解决方案
- 缓存空对象
缓存空对象会有两个问题：
	- 使用空值做了缓存，这样Redis中会存储更多的键，从而造成更高的内存开销（如果是攻击，造成的影响会更大）——可以结合设置较短的过期时间来解决。

- 布隆过滤器
在访问Redis和数据库之前，将存在的key用[布隆过滤器](https://en.wikipedia.org/wiki/Bloom_filter)提前保存，做第一层的过滤——适合数据相对固定的场景

### 缓存雪崩问题
Redis缓存的存在，有效的保护了数据库。但是如果Redis由于某些原因不能提供服务，所有的请求都会到达数据库，可能会造成MySQL宕机。
我们即将使用的是腾讯云的Redis服务，高可用性有一定的保障，我们可以做的：
- 保持对Redis实例使用情况的监控
- 一定的降级策略（如有需要，提前做演练）

## 说明
以上分析是基于团队要使用腾讯云Redis服务为前提，大家如果要是自建Redis，还需要具体情况具体分析。

## 资料
- 中文官网：http://www.redis.cn/
- 英文官网：https://redis.io/
- Redis命令掌握：http://try.redis.io/
- https://coolshell.cn/articles/17416.html
- https://github.com/alibaba/canal
- https://github.com/leonchen83/redis-replicator
- https://en.wikipedia.org/wiki/Bloom_filter
- https://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651961368&idx=1&sn=82a59f41332e11a29c5759248bc1ba17&chksm=bd2d0dc48a5a84d293f5999760b994cee9b7e20e240c04d0ed442e139f84ebacf608d51f4342&scene=27#wechat_redirect