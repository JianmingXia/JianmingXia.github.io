---
title: Redis 基础
date: 2018/08/05 21:00:00
tags:
  - Redis
  - 基础
  - 阅读笔记
categories: 缓存
---

## 说明
> Redis命令、数据持久化、事务、Pipeline等

从上周开始着手与Redis方面的工作，在之前的工作经历中，简单的使用过Memcache，用于缓存常用且固定的数据。
而这次，使用Redis的主要原因是之前的项目深受进程内缓存的约束（之前使用C++开发项目），导致没有解决方法来处理业务拓展的问题。所以，目前进行业务改造首先是在去进程内缓存，然后再启用Redis服务。Memcache与Redis的比较不用多说，选用Redis还有以下原因：
- 腾讯云有成熟的Redis服务——目前身处小团队，没有太多精力放在Redis的可用性上
- Redis服务能够解决我们团队目前的问题，更容易拓展
<!-- more -->

本文的主要参考是《Redis实战》，就Redis入门而言，目前我个人觉得比较好的路线是（非利益相关，只是就个人学习经历而言）：
- 先《Redis实战》：先讲解Redis基础，结合使用场景使用
- 再《Redis设计与实现》：从源码的角度来介绍Redis，更有茅舍顿开的感觉

## Redis命令

### 字符串（String）

可以存储的数据类型：

- 字节串（byte string）
- 整数
- 浮点数

Redis中的自增和自减命令：

- **INCR key**——加1
- **DECR key**——减1
- **INCRBY key increment**——加指定值
- **DECRBY key decrement**——减指定值
- **INCRBYFLOAT key increment**——加指定值（浮点数）

当用户将值存储到Redis字符串中时，如果这个值可以被解释为十进制整数或浮点数，那我们可以对这个字符串进行INCR\*和DECR\*操作：

- 如果不存在的键/或是空数据——会当作0处理
- 如果无法被解释为十进制整数或浮点数——会返回错误

处理子串和二进制位的命令：

- **APPEND key value**——追加子串至尾部（如果key不存在，等同于set）
- **GETRANGE key start end**——获取偏移量start-end的数据，包括start和end
- **SETRANGE key offset value** 
- **GETBIT key offset** 
- **SETBIT key offset value** 
- **BITCOUNT key [start][end]** 
- **BITOP operation destkey key [key ...]** 

### 列表（List）

Redis的列表允许用户从序列的两端推入或弹出元素、获取列表元素及常见的列表操作。

用处：

- 存储任务信息
- 最近浏览过的文章
- 常用联系人信息

常用的列表命令：

- **RPUSH key value [value ...]** —— 将元素插入到右端（表尾）
- **LPUSH key value [value ...]** —— 将元素插入到左端（表头）
- **RPOP key** ——移除并返回表尾元素
- **LPOP key** ——移除并返回表头元素
- **LINDEX key index** ——返回列表中下标为index的元素
- **LRANGE key start stop** ——返回指定区间内的元素（包括start 和 stop）,可以使用负数下标，-1表示最后一个元素，如此类推
- **LTRIM key start stop** ——修剪列表，只保留指定区间内的元素

阻塞弹出和元素移动命令：

- **BLPOP key [key ...] timeout** ——从第一个非空列表中弹出左端元素，或在timeout秒内阻塞等待可弹出的元素出现
- **BRPOP key [key ...] timeout** ——与BLPOP相对应
- **RPOPLPUSH source destination** ——在一个原子时间内：将列表source的尾元素弹出，返回客户端，并将该元素插入到destination头部（如果source、destination相同，可视为自旋操作）
- **BRPOPLPUSH source destination timeout** ——是RPOPLPUSH 命令的阻塞版本

对于阻塞弹出和弹出并推入命令，最常见的用例是——消息队列、任务队列。

### 集合（Set）

常用集合命令：

- **SADD key member [member ...]** ——将一个或多个元素添加到集合中，并返回新加入的元素数量
- **SREM key member [member ...]** ——从集合中移除一个或多个元素，并返回移除的元素数量
- **SISMEMBER key member** ——判断member是否是集合key的元素
- **SCARD key** ——返回集合key中的基数——即元素数量（card是集合中cardinality的简写）
- **SMEMBERS key** ——返回集合key中的所有元素
- **SRANDMEMBER key [count]** ——从集合key中返回一个或多个元素。当count为负数时，会出现重复情况
- **SPOP key** ——从集合key中随机移除一个元素并返回
- **SMOVE source destination member** ——将member元素由source集合移至destination集合。如果member被成功移除，返回1；否则返回0

集合真正的强大之处在于组合和关联多个结合，相关命令：

- **SDIFF key [key ...]** ——返回存在于第一个集合、但不存在于其他集合中的元素（查集）
- **SDIFFSTORE destination key [key ...]** ——基于SDIFF——将结果存储于destination集合，返回结果集的元素数量。如果destination已存在，则覆盖数据。
- **SINTER key [key ...]** ——返回同时存在于所有集合的元素（交集）
- **SINTERSTORE destination key [key ...]** ——基于SINTER——将结果存储于destination集合，返回结果集的元素数量。
- **SUNION key [key ...]** ——返回至少存在于一个集合中的元素（并集）
- **SUNIONSTORE destination key [key ...]** ——基于SUNION——将结果存储于destination集合，返回结果集的元素数量。

### 散列（Hash）

常用的散列命令：

- **HMGET key field [field ...]** ——返回散列中的一个或多个键的值
- **HMSET key field value [field value ...]** ——为散列中的一个或多个键设置值
- **HDEL key field [field ...]** ——删除散列中的一个或多个键值对，返回成功删除的键值对数量
- **HLEN key** ——返回散列中键值对数量

HGET和HSET分别是HMGET和HMSET命令的单参数版本。

更高级的特性：

- **HEXISTS key field** ——检查field是否存在于散列中
- **HKEYS key** ——返回散列中所有键
- **HVALS key** ——返回散列中所有值
- **HGETALL key** ——以列表形式返回散列中所有键值对
- **HINCRBY key field increment** ——对散列key中的域field的值加上增量increment
- **HINCRBYFLOAT key field increment** ——与HINCRBY相似

尽管有HGETALL存在，但HKEYS和HVALS也是非常有用的：如果散列包含的值非常大，用户可以先使用HKEYS取出键，再使用HGET去取值，避免一次性获取多个大体积的值而导致服务器阻塞。

### 有序集合（SortedSet）

与散列存储着键与值之间的映射类似，有序集合也存储着成员与分值之间的映射。

常用的命令如下：

- **ZADD key score member [[score member][score member] ...]** ——将带有指定分值的成员添加到有序集合中。如果成员已在有序集合中，可视为更新分值操作
- **ZREM key member [member ...]** ——从有序集合中移除指定的成员，返回移除的成员数量
- **ZCARD key** ——返回有序集合的基数
- **ZINCRBY key increment member** ——将member成员的分值加上increment
- **ZCOUNT key min max** ——返回分值介于min-max之间的成员数量，包括min及max
- **ZRANK key member** ——返回成员member在有序集合中的排名
- **ZSCORE key member** ——返回成员member的分值
- **ZRANGE key start stop [WITHSCORES]** ——返回有序集合中排名介于start-stop之间的成员，如果有WITHSCORES选项，会将分值一起返回

更高级的特性：

- **ZREVRANK key member** ——与ZRANK相对应，规则改为从大到小
- **ZREVRANGE key start stop [WITHSCORES]** ——与ZRANGE相对应
- **ZRANGEBYSCORE key min max [WITHSCORES\][LIMIT offset count]** ——返回有序集合中，分值在min-max之间的所有成员
- **ZREVRANGEBYSCORE key max min [WITHSCORES\][LIMIT offset count]** ——与ZRANGEBYSCORE相对应
- **ZREMRANGEBYRANK key start stop** ——移除有序集合中排名在start-stop之间的所有成员，包括start和stop
- **ZREMRANGEBYSCORE key min max** ——移除有序集合中分值在min-max之间的所有成员，包括min和max
- **ZINTERSTORE destination numkeys key [key ...\][WEIGHTS weight [weight ...]\][AGGREGATE SUM|MIN|MAX]** ——对给定的有序集合执行类似集合的交集运算。
  - **WEIGHTS**：乘法因子，每个有序集合所有成员的score都需要乘以乘法因子
  - **AGGREGATE**：指定聚合方式，默认是SUM
- **ZUNIONSTORE destination numkeys key [key ...\][WEIGHTS weight [weight ...]\] [AGGREGATE SUM|MIN|MAX]** ——对给定的有序集合执行类似集合的并集运算

### 发布与订阅（pub/sub）

发布与订阅的特点是订阅者（listener）负责订阅频道（channel），发送者（publisher）负责向频道发送二进制字符串消息。

发布与订阅命令：

- **SUBSCRIBE channel [channel ...]** ——订阅一个或多个频道
- **UNSUBSCRIBE [channel [channel ...]]** ——退订一个或多个频道；如果没有传频道，那么退订所有频道
- **PUBLISH channel message** ——向指定频道发布消息
- **PSUBSCRIBE pattern [pattern ...]** ——订阅与给定模式相匹配的所有频道
- **PUNSUBSCRIBE [pattern [pattern ...]]** ——退订与给定模式相匹配的频道；如果没有模式，那么退订所有频道

缺点：

- Redis的稳定性：对于旧版的Redis来说，客户端读取消息的速度会是一个瓶颈，不断积压的消息使得Redis输出缓冲区越来越大，可能会导致Redis的速度变慢，甚至崩溃。新版可以通过设置client-output-buffer-limit来避免这个问题。
- 数据传输的可靠性：断线时发送的消息会丢失

### 其他命令

#### 排序

- **SORT key [BY pattern\][LIMIT offset count\] [GET pattern [GET pattern ...\]\][ASC | DESC\][ALPHA\][STORE destination\]** ——可以做的功能有很多，[查看文档](http://doc.redisfans.com/key/sort.html)

#### 基本的Redis事务

Redis的基本事务：可以让一个客户端在不被其他客户端打断的情况下执行多个命令。和关系型数据库那种可以在执行的过程中进行回滚的事务不同，在Redis里面，被MULTI命令和EXEC命令包围的所有命令会一个接一个地执行，直至所有命令都执行完。

#### 键的过期时间

处理过期时间的命令：

- **PERSIST key** ——移除键的过期时间，即持久
- **TTL key** ——返回给定Key距离过期的时间（秒）；当key不存在时，返回-2；如果key没有设置过期时间，返回-1
- **EXPIRE key seconds** ——设置过期时间（秒）
- **EXPIREAT key timestamp** ——设置过期时间为时间戳
- **PTTL key** ——与TTL类似，单位为毫秒
- **PEXPIRE key milliseconds** ——与EXPIRE类似，单位为毫秒
- **PEXPIREAT key milliseconds-timestamp** ——与EXPIREAT类似，单位为毫秒

### 小结

了解Redis最常用的一些命令：

- 不同数据类型的命令
- 发布与订阅
- Sort命令
- Redis事务
- 过期相关的命令

## 数据安全与性能保障

### 持久化选项

- 快照：可以将存在于某一时刻的所有数据都写入硬盘
- 只追加文件（append-only file, AOF）：会在执行写命令时，将被执行的写命令复制到硬盘中

#### 快照持久化

创建快照的方法：

- BGSAVE：Redis会调用fork来创建一个子进程，由子进程负责将快照写入硬盘，父进程可以继续处理请求
- SAVE：在创建快照完毕之前不再响应其他命令
- SAVE配置选项：当条件被满足时，会触发BGSAVE命令
- SHUTDOWN：执行SAVE命令，并阻塞所有客户端，SAVE完毕后关闭服务器
- 当Redis服务器连接另一个Redis服务器，并发送SYNC命令

如果系统发生崩溃，将会丢失最近一次生成快照之后更改的所有数据——快照持久化只适用于对数据要求没那么高的场景。

快照持久化场景分析：

- 个人开发：只设置了save 900 1；主要考虑是尽可能降低快照持久化带来的资源消耗
- 对日志进行聚合计算：如果Redis因为崩溃而未能成功创建快照，我们能承受丢失多长时间以内产生的新数据。同时记录处理进度，使用事务流水线保证日志的处理结果和处理进度会同时被记录。
- 大数据：Redis缓存的数据越大，快照持久化花费的时间就越长

#### AOF持久化

简单来说，AOF持久化会将被执行的写命令写到AOF文件的末尾，以此来记录数据发生的变化。 

appendfsync配置选项：

| 选项     | 同步频率                                                 |
| -------- | -------------------------------------------------------- |
| always   | 每个Redis写命令都要同步写入硬盘——这会严重降低Redis的速度 |
| everysec | 每秒执行一次同步，显式地将多个写命令同步到硬盘           |
| no       | 让操作系统来决定应该何时进行同步                         |

Redis每秒同步一次AOF文件时的性能和不使用任何持久化特性时的性能相差无几。

缺点：AOF文件的体积大小

#### 重写/压缩 AOF文件

用户可以向Redis发送BGREWRITEAOF命令——通过移除AOF文件中的冗余命令来重写AOF文件，使得AOF文件尽可能地小。

BGREWRITEAOF和BGSAVE很相似：Redis会创建一个子进程，然后由子进程负责对AOF文件进行重写。

自动重写配置：

- auto-aof-rewrite-percentage 100
- auto-aof-rewrite-min-size 64mb

### 复制

#### 对Redis的复制相关选项进行配置

- 启动配置：slaveof host port
- 正在运行的Redis服务器：
  - SLAVEOF no one：让服务器终止复制操作
  - SLAVEOF host port：让服务器开始复制

#### Redis复制的启动过程

从服务器连接主服务器时的操作：

| 步骤 | 主服务器操作                                                 | 从服务器操作                                                 |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | （等待命令进入）                                             | 连接（或重连）主服务器，发送SYNC命令                         |
| 2    | 开始执行BGSAVE，并使用缓冲区记录BGSAVE之后执行的所有写命令   | 根据配置选项来决定是继续使用现有的数据，还是返回错误         |
| 3    | BGSAVE执行完毕，向从服务器发送快照文件，并在发送期间继续使用缓冲区记录被执行的写命令 | 丢弃所有旧数据，开始载入主服务器发送的快照文件               |
| 4    | 快照文件发送完毕，开始向从服务器发送存储在缓冲区里的写命令   | 完成对快照文件的解释操作，继续接受命令请求                   |
| 5    | 缓冲区存储的写命令发送完毕：从现在开始，每执行一个写命令，就向从服务器发送相同的写命令 | 执行主服务器发来的所有存储在缓冲区里的写命令；并从现在开始，接受并执行主服务器传来的每个写命令 |

需要注意的是：

- 从服务器在进行同步时，会清空自己的所有数据
- 不支持主主复制

当多个从服务器尝试连接同一个主服务器时：

| 当有新的从服务器连接主服务器时      | 主服务器的操作                                               |
| ----------------------------------- | ------------------------------------------------------------ |
| 当上述的步骤3尚未执行时             | 所有从服务器都会接受到相同的快照文件和相同的缓冲区写命令     |
| 当上述的步骤3正在执行或已经执行完毕 | 主服务器与较早进行连接的从服务器执行完复制所需步骤之后；再与新连接的从服务器进行以上步骤 |

#### 主从链

直接扩展多个从服务器会受到主服务器负载能力的限制，这是我们可以通过使用Redis主从节点组成的中间层来解决这个问题。

#### 检查硬盘写入

通过判断aof_pending_bio_fsync属性的值是否为0，知悉服务器是否将已知数据都保存到硬盘中。

> INFO命令提供了大量的与Redis服务器当前状态相关的信息，如内存占用量、客户端连接数、每个数据库包含的键的数量、上一次创建快照文件之后执行的命令数量等等。

### 处理系统故障

#### 验证快照文件和AOF文件

- redis-check-aof
- redis-check-dump

可以在系统故障发生之后，检查AOF文件和快照文件的状态

#### 更换故障主服务器

假设AB分别为主从服务器，机器A出现故障

- 方案1：
  - 向机器B发送SAVE命令，创建快照文件
  - 将快照文件发送至C，在机器C上启动Redis
  - 将机器B设为C的从服务器
- 方案2：
  - 将机器B升级为主服务器
  - 为机器B设置从服务器

>  Redis Sentinel可以监视指定的Redis主服务器及从服务器，并在主服务器下线时自动进行故障转移。

### 事务

结合一个场景来分析，如何使用事务。

#### 定义用户信息与用户包裹

#### 将商品放到市场上销售

#### 购买商品

通过组合WATCH、MULTI和EXEC命令，正确地使用事务防止数据错误发生。

### 非事务型流水线（non-transactional pipeline）

在命令互不影响的情况下，通过使用流水线提升Redis性能。

### 关于性能方面的注意事项

可以根据redis-benchmark来了解性能，但实际客户端的性能达不到这么高——redis-benchmark不会处理执行命令所获得的命令回复，节约了对命令回复进行语法分析的时间。

一般来说，在不使用流水线的情况下(python客户端)，性能只有redis-benchmark的50%-60%。（这个可以具体场景具体分析）

### 小结

主要讲述了数据安全与性能保障这部分：

- 数据安全
  通过使用持久化和复制来预防并应对系统故障

- 性能保障
  如何使用事务来防止出错、如何使用流水线提升性能

## 资料
- http://redisinaction.com/