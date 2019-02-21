---
title: hiredis 连接池
date: 2018/08/26 15:30:00
tags:
  - hiredis
  - C++
  - Redis
categories: 缓存
---

## 说在前面

前面也说了，最近团队内部打算开始使用Redis，由我来负责推进。

在将NodeJS部分Redis引入完成之后，开始着手封装C++这方面，本来预想会与NodeJS一般顺利的，但是在引入hiredis时，发现一个很棘手的问题，它竟然是非线程安全的。
<!-- more -->

## 非线程安全

> *Note: A redisContext is not thread-safe.*

### 为什么要使用hiredis

- 腾讯云推荐的最佳实践（C客户端推荐，支持C++）
- 官方出品
- 有大厂也正在使用——比如：网易

### 是否真的是非线程安全

- 从github上看，非线程安全是在**README.md**中重重说明的
- 从源代码看，通过**redisConnect**获得的redis上下文，它的obuf只有一个，当在多线程中使用时，obuf会被其它线程释放
- 写了一个开启多线程的程序去执行Redis命令，马上就挂了

### 解决方案

既然hiredis是非线程安全的，而我们的C++项目一般又都是开启多线程的，如何处理呢？

- 换一个C++库：从官网中推荐的库来看，hiredis应该是被使用最多的，毕竟在github上star数最多，故而一时也没有找到更好的方案
- 所以在考虑可以让hiredis变成线程安全的么？
  - 每次使用时，重新连接
  - 使用连接池，保证线程安全

虽然每次连接消耗不太大，但是在高并发的情况下，影响还是蛮大的，故而还是试一试连接池

## hiredis连接池

之前底层项目中有连接池相关的代码，在写的时候也狠狠的借鉴了一番，主要概念其实就两个：

- 信号量：保证获取redis连接实例时能正常获取
- 自旋锁：保证线程获取redis连接实例时不会发生冲突

下面贴出部分代码：

```c++
#include "RedisBase.h"

class RedisPool
{
public:
	RedisPool();
	virtual ~RedisPool();

	bool initRedis(RedisBase *&redis);
	bool start(const RedisClientConfig& config);
	void stop();

	RedisBase *pop();
	void push(RedisBase *redis);

private:
	queue<RedisBase*> _redis_queue;
	Mutex _redis_queue_lock;
	Semaphore *_sem;

	bool _is_running;
};
```

```c++
RedisBase * RedisPool::pop()
{
	if (_is_running == false) {
		return NULL;
	}
	_sem->wait();
	AutoLock lock(_redis_queue_lock);
	RedisBase *redis = _redis_queue.front();
	if (false == redis->connectSucc()) {
		redis->free();
		initRedis(redis);
	}
	_redis_queue.pop();
	return redis;
}

void RedisPool::push(RedisBase *redis)
{
	if (_is_running == false) {
		return;
	}

	AutoLock lock(_redis_queue_lock);
	_redis_queue.push(redis);
	_sem->post();
}
```

### 测试

再次拿出之前测试是否线程安全的测试用例，没有问题，测试通过。

## 总结

感觉真的好久没有写过这么底层的代码，写出来之后感觉特别爽。上次感觉这么爽的，好像还是去年针对业务中的需求，写了一个多路归并的实现（虽然最后由于时间关系，并没有使用败者树的思路去优化）。一直在写业务代码，老是感觉出现了瓶颈，加油加油~~~