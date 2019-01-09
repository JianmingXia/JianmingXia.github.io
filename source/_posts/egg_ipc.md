---
title: 进程间通讯（egg）
date: 2018/11/11 20:30:00
tags:
  - Egg
  - IPC
categories: 
  - [Egg]
  - [NodeJS]
---

## 进程间通讯

```
const cluster = require("cluster");

if (cluster.isMaster) {
  const worker = cluster.fork();
  worker.send("hi there");
  worker.on("message", msg => {
    console.log(`msg: ${msg} from worker#${worker.id}`);
  });
} else if (cluster.isWorker) {
    process.on("message", msg => {
    console.log(`msg: ${msg} from master`);
    process.send(msg);
  });
}
```
<!-- more -->

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1546915664193-22f0f0d7-3986-426a-997e-2ea4dc1b68b8.png#align=left&display=inline&height=49&linkTarget=_blank&name=image.png&originHeight=49&originWidth=375&size=5827&width=375)

### 转发
由上可以看出，IPC通道只存在于cluster 及 子进程之间，而在egg中——则只存在与Master 和 Agent/Worker进程之间，Agent 和 Worker进程是没有的，对此egg的解决方式是通过Master 来转发

#### Agent 广播所有 Worker
![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1547001752989-4507c844-b51c-471e-b72b-d20aec7a9d81.png#align=left&display=inline&height=260&linkTarget=_blank&name=image.png&originHeight=260&originWidth=416&size=9598&width=416)

#### 指定Worker

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1547001790631-7cc96672-c8a1-4d8a-872a-727d4f4c0968.png#align=left&display=inline&height=240&linkTarget=_blank&name=image.png&originHeight=240&originWidth=399&size=10282&width=399)

为了方便调用，egg中封装了messenger对象挂载在app/agent实例上

### 发送

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1547001892799-daed7acc-3a0e-417a-9c41-65a280df0bd4.png#align=left&display=inline&height=387&linkTarget=_blank&name=image.png&originHeight=387&originWidth=663&size=54850&width=663)

### 接受

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1547001953040-7bcc17ce-659b-4f58-ad6a-8cadf59338ae.png#align=left&display=inline&height=129&linkTarget=_blank&name=image.png&originHeight=129&originWidth=331&size=8194&width=331)

## 举个栗子
### 需求
> 这个是 egg 官方的例子：我们有一个接口需要从远程数据源中读取一些数据，对外部提供 API，但是这个数据源的数据很少变化，因此我们希望将数据缓存到内存中以提升服务能力，降低 RT


为了实现上述的目标，本服务有一定的缓存机制：
* 定时从远程数据源获取数据，更新内存缓存，为了降低对数据源压力，更新的间隔时间会设置的比较长
* 远程数据源提供一个检查是否有数据更新的接口，我们的服务可以更频繁的调用检查接口，当有数据更新时才去重新拉取数据
* 远程数据源通过消息中间件推送数据更新的消息，我们的服务监听消息来更新数据

在实际的项目中，第一个方案一般用于兜底；第二个方案或第三个用于提高数据的实时性

### 准备工作

```
// app/service/source.js
let memoryCache = {};

class SourceService extends Service {
  get(key) {
    return memoryCache[key];
  }

  async checkUpdate() {
    // check if remote data source has changed
    const updated = await mockCheck();
    this.ctx.logger.info('check update response %s', updated);
    return updated;
  }

  async update() {
    // update memory cache from remote
    memoryCache = await mockFetch();
    this.ctx.logger.info('update memory cache from remote: %j', memoryCache);
  }
}
```

### 第一个方案
使用 schedule 每隔10分钟从远程数据源获取数据

all：schedule中的all表示每隔Worker都会执行，因为数据是在每个进程中缓存的，统一更新

```
// app/schedule/force_refresh.js
exports.schedule = {
  interval: '10m',
  type: 'all', // run in all workers
};

exports.task = async ctx => {
  await ctx.service.source.update();
  ctx.app.lastUpdateBy = 'force';
};
```

### 第二个方案
每隔10s去check数据是否有更新——这里type: worker，由一个工作进程处理即可。如果有更新，则通知所有的Worker，去拉取源数据进行更新

```
// app/schedule/pull_refresh.js
exports.schedule = {
  interval: '10s',
  type: 'worker', // only run in one worker
};

exports.task = async ctx => {
  const needRefresh = await ctx.service.source.checkUpdate();
  if (!needRefresh) return;

  // notify all workers to update memory cache from `file`
  ctx.app.messenger.sendToApp('refresh', 'pull');
};
```

```
// app.js
module.exports = app => {
  app.messenger.on('refresh', by => {
    app.logger.info('start update by %s', by);
    // create an anonymous context to access service
    const ctx = app.createAnonymousContext();
    ctx.runInBackground(async () => {
      await ctx.service.source.update();
      app.lastUpdateBy = by;
    });
  });
};
```

### 第三个方案
一般来说，本服务作为消息中间件的客户端来监听事件，当接受到数据更新的操作时，通知其他Worker进程——故而放在Agent进程中：

```
// agent.js

const Subscriber = require('./lib/subscriber');

module.exports = agent => {
  const subscriber = new Subscriber();
  // listen changed event, broadcast to all workers
  subscriber.on('changed', () => agent.messenger.sendToApp('refresh', 'push'));
};
```

使用mock的方式来测试：

```
module.exports = class Subscriber extends EventEmitter {
  constructor() {
    super();

    this._start();
  }

  _start() {
    const interval = Math.random() * 5000 + 5000;
    setTimeout(() => {
      this.emit('changed');
      this._start();
    }, interval);
  }
};
```

### 结果
> 完整代码：[https://github.com/JianmingXia/StudyTest/tree/master/egg/ipc](https://github.com/JianmingXia/StudyTest/tree/master/egg/ipc)


![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1547003454484-f063b300-64df-412c-a8b3-eef9a52434e4.png#align=left&display=inline&height=76&linkTarget=_blank&name=image.png&originHeight=76&originWidth=375&size=4453&width=375)

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1547003464886-60594520-c400-4f43-bbfa-619350bb2233.png#align=left&display=inline&height=68&linkTarget=_blank&name=image.png&originHeight=68&originWidth=368&size=4664&width=368)

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1547003736447-87a9dacc-2e4f-4b8f-9dff-38bcebc56515.png#align=left&display=inline&height=66&linkTarget=_blank&name=image.png&originHeight=66&originWidth=383&size=4375&width=383)

## 资料
* [https://eggjs.org/zh-cn/core/cluster-and-ipc.html#进程间通讯ipc](https://eggjs.org/zh-cn/core/cluster-and-ipc.html#%E8%BF%9B%E7%A8%8B%E9%97%B4%E9%80%9A%E8%AE%AFipc)
* [http://nodejs.cn/api/process.html#process_event_message](http://nodejs.cn/api/process.html#process_event_message)
* [https://medium.com/js-imaginea/clustering-inter-process-communication-ipc-in-node-js-748f981214e9](https://medium.com/js-imaginea/clustering-inter-process-communication-ipc-in-node-js-748f981214e9)
