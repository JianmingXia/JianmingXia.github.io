---
title: Redis 深度历险——总结(续)
date: 2018/08/20 19:30:00
tags:
  - Redis
  - 阅读笔记
categories: 缓存
---

## 说在前面

继[Redis 深度历险——总结](/posts/redis_depth_adventure)一文后，这是第二篇我在阅读[Redis 深度历险：核心原理与应用实践](https://juejin.im/book/5afc2e5f6fb9a07a9b362527)一文的阅读笔记。
主要内容是集群部分及Redis的一些拓展，因为时间及工作内容的关系，集群这部分目前我只做了一些简单的了解（目前团队打算使用的是腾讯云的Redis主从服务）；

<!-- more -->
## 集群

### Sentinel（哨兵）

当Redis主从运行中，主节点挂掉后，手动切？

Sentinel负责持续监控主从节点的健康，当主节点挂掉时，自动选择一个最优的从节点切换为主节点，然后其它的节点与新的主节点建立复制关系。

Sentinel一般用于读写分离，从库也会提供服务，Cluster中的从库一般只是备用库。

### Codis

在高并发场景下，单个Redis实例会有很多限制：

- 内存：内存太大会导致rdb文件过大，进一步导致主从同步时全量同步时间过长，重启恢复也会消耗很长的时间
- CPU利用率：单个Redis实例只能利用单个核心

Redis集群：将众多小内存的Redis实例综合

###  Cluster

Redis官方的集群化方案，与Codis不同，它是去中心化的。

## 拓展

### Stream

一个强大的支持多播的可持久化的消息队列——疯狂借鉴Kafka

### Info

#### 主要参数
- Server 服务器运行的环境参数 
- Clients 客户端相关信息 
- Memory 服务器运行内存统计数据 
- Persistence 持久化信息 
- Stats 通用统计数据 
- Replication 主从复制相关信息 
- CPU CPU 使用情况 
- Cluster 集群信息 
- KeySpace 键值对统计数量信息
- Commandstats: Redis命令统计

#### 其它参数
- all: 返回所有信息
- default: 值返回默认设置的信息

### 过期策略

过期词典——redisDB（底层的数据结构）中的expires保存着数据中所有键的过期时间 

#### 过期键删除策略

- 定时删除
- 惰性删除
- 定期删除

#### 定时删除

- 对内存是最友好的：通过使用定时器，能够保证过期键会尽可能地被删除，并释放过期键所占用的内存
- 对CPU时间最不友好：在过期键比较多的情况下，删除过期键可能会占用相当一部分的时间

#### 惰性删除

- 对CPU时间是最友好的：只会在取出键时，才对键进行过期检查
- 对内存是最不友好的

#### 定期删除

以上两种策略单一使用时都有明显的缺陷，定期删除策略是前两种策略的整合和折中：

- 每隔一段时间执行一次删除过期键操作，并通过限制删除操作执行的时长和频率来减少删除操作对CPU时间的影响
- 通过定期删除过期键，有效减少了因为过期键而带来的内存浪费

比如：

- 从过期字典中随机 20 个 key
- 删除这 20 个 key 中已经过期的 key
- 如果过期的 key 比率超过 1/4，那就重复步骤 1

为了保证过期扫描不会出现循环过度，导致线程卡死现象，算法还增加了扫描时间的上限——默认25ms 

#### 如何合适的设置过期

设想一个大型的Redis实例中所有的key在同一时间都过期了，会发生什么呢？

Redis会持续扫描过期字典，直至过期字典中过期的key变得稀疏，才会停止重复。这会导致线上读写请求出现明显的卡顿——如果同时有101个请求，按照上述的默认值，第101个需要等待2500ms。

如果存在大批的key过期，设置一个随机范围的过期时间。

### LRU

当Redis内存超出物理内存限制时，内存的数据会开始与磁盘产生频繁的交换，交换会让Redis的性能急剧下降。

在生产环境，我们肯定是不希望这种情况发生的，可以使用**maxmemory**来限制。当实际内存超出**maxmemory**时，Redis有一些可选策略(maxmemory-policy)来让用户自己决定该如何处理：

- noeviction 不会继续服务写请求 (DEL 请求可以继续服务)，读请求可以继续进行。这样可以保证不会丢失数据，但是会让线上的业务不能持续进行。这是默认的淘汰策略。 
- volatile-lru 尝试淘汰设置了过期时间的 key，最少使用的 key 优先被淘汰。没有设置过期时间的 key 不会被淘汰，这样可以保证需要持久化的数据不会突然丢失。 
- volatile-ttl 跟上面一样，除了淘汰的策略不是 LRU，而是 key 的剩余寿命 ttl 的值，ttl 越小越优先被淘汰。 
- volatile-random 跟上面一样，不过淘汰的 key 是过期 key 集合中随机的 key。 
- allkeys-lru 区别于 volatile-lru，这个策略要淘汰的 key 对象是全体的 key 集合，而不只是过期的 key 集合。这意味着没有设置过期时间的 key 也会被淘汰。 
- allkeys-random 跟上面一样，不过淘汰的策略是随机的 key

volatile-xxx 策略只会针对带过期时间的 key 进行淘汰，allkeys-xxx 策略会对所有的 key 进行淘汰——如果只是拿Redis做缓存，应该使用allkeys-xxx；如果使用Redis的持久化功能，可以使用volatile-xxx策略。

#### LRU 算法

使用一个链表，当某元素被访问时，就移至表头；需要移除时，清理链表尾部即可。

#### 近似LRU

使用LRU算法，需要消耗大量额外的内存，需要对现有的数据结构进行较大的改造。

Redis为实现近似LRU算法，给每个key增加了一个小字段——记录最近一次被访问的时间戳

当Redis进行写操作时，如果内存超出maxmemory，执行一次LRU淘汰算法：

- 随机取出5个key
- 淘汰掉最旧的key，如果内存还是超出，继续

### 懒惰删除

#### Redis为什么要懒惰删除(lazy free)？

**del**会直接释放对象的内存，大部分情况下，这个指令非常快，但是如果删除的key是一个大的对象，删除就会导致单线程卡顿。

在Redis4.0中，可以使用**unlink**对删除操作进行懒处理，交由异步回收线程。

#### flush

- flushdb
- flushall

可以在指令后增加**async**参数，交由异步回收线程

#### 异步队列

懒惰删除后，都会将回收操作包装成一个任务塞进异步队列，后台线程会从异步队列中取任务——主线程与异步线程会同时操作异步队列，故而是一个线程安全队列

#### AOF Sync

执行AOF Sync操作的线程也是一个独立的异步线程，也有一个属于自己的异步队列，只存放AOF Sync任务。

#### 更多

- key的过期
- LRU淘汰
- rename指令

在Redis4.0中，这些也有异步机制，但是需要额外的配置：

- slave-lazy-flush 从库接受完 rdb 文件后的 flush 操作 
- lazyfree-lazy-eviction 内存达到 maxmemory 时进行淘汰 
- lazyfree-lazy-expire key 过期删除 
- lazyfree-lazy-server-del rename 指令删除 destKey

### 保护Redis

- 避免数据泄露与丢失
- 避免所在主机权限被黑客获取
- 避免人为操作失误

[被黑客通过Redis端口黑的亲身经历](../redis_bug)

#### 指令安全

- keys：可能导致Redis卡顿
- flushdb、flushall：清空数据

使用**rename-command**指令修改名称：

```
rename-command keys abckeysabc
```

禁止命令：

```
rename-command flushall ""
```

#### 端口安全

redis默认监听6379端口，很容易就会被扫描到，需要警惕Redis服务会被外网直接访问。

也可以使用密码来降低风险——个人觉得禁止被外网访问即可，如果会被外网访问，即使有了密码也是一件很危险的事，而且密码也会影响主从复制（需要设置密码才能进行同步）

#### Lua脚本安全

- 开发者必须禁止Lua脚本由用户输入的内容生成(UGC)，否则可能会被黑客利用植入恶意的攻击代码——这个目前不太理解细节，目前的理解是：结合在Web开发的经验，不要相信用户的输入，不要随意拼接sql语句，类似于此处的禁止Lua措施
- Redis以普通用户的身份启动

#### SSL代理

Redis并不支持SSL连接，故而客户端与服务器之间的数据也不应该直接暴露在公网上传输，会有被窃听的可能。

如果一定要放在公网上，可以考虑使用SSL代理——Redis推荐spiped工具

## 总结

为了让团队成员更方便的使用Redis，最近一直都在封装Redis：
- NodeJS已做了一个简单的封装，引入egg-redis的速度比较快，也没有遇到什么问题；
- C++选用了hiredis，开展得不是很顺利，
    - 在windows上的编译就花费较长的时间，各种奇怪的问题；
    - hiredis是非线程安全的，目前的方案是封装一个连接池（最初还比较担心freeReplyObject会依赖redis连接实例，查看了底层代码，发现是没有影响的，使用静态方法去释放reply即可）

虽然这段期间工作不是很顺利，但是感觉自己也学习到很多东西，很享受这样的感觉。
Redis的基础知识及运用就到此为止，后续也会安排时间去系统的看一下Redis源码，应该会有更多的收获。

## 参考
- https://juejin.im/book/5afc2e5f6fb9a07a9b362527
- http://redisinaction.com/