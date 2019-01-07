---
title: 多进程模型（egg）
date: 2018/11/7 20:00:00
tags:
  - Egg
  - Multi-Process
categories: 
  - [Egg]
  - [NodeJS]
---


大家都知道JS代码是运行在单线程上的——NodeJS也同理，这也就导致了进程只能利用一个CPU，没有办法享受到多核带来的优势；同时也带来另外一个问题，如果代码出了问题，单个进程不确定性的问题也就暴露出来。<br />而Egg作为企业级的解决方案，首先解决的是：如果榨干服务器资源，利用多核CPU的并发优势

## Cluster 是什么
### 简单说明

```
A single instance of Node.js runs in a single thread. To take advantage of multi-core systems, the user will sometimes want to launch a cluster of Node.js processes to handle the load.

The cluster module allows easy creation of child processes that all share server ports.
```

即：单个NodeJS实例在单线程中运行。为了充分利用多核系统，有时需要启动一组NodeJS 进程来处理负载任务——cluster 模块可以创建共享服务器端口的子进程

### 有什么作用？
* 在服务器上启动多个进程（每个进程运行的都是同一份代码）
* 多个进程监听同一个端口

### 举个栗子
> [https://nodejs.org/api/cluster.html](https://nodejs.org/api/cluster.html)

参考官方文档的例子（设置了返回的编码）：

```
const cluster = require("cluster");
const http = require("http");
const numCPUs = require("os").cpus().length;

if (cluster.isMaster) {
  console.log(`主进程 ${process.pid} 正在运行`);

  // 衍生工作进程。
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on("exit", (worker, code, signal) => {
    console.log(`工作进程 ${worker.process.pid} 已退出`);
  });
} else {
  // 工作进程可以共享任何 TCP 连接。
  // 在本例子中，共享的是 HTTP 服务器。
  http
    .createServer((req, res) => {
      res.writeHead(200, { "Content-Type": "text/html;charset=utf-8" });
      res.end("你好世界\n");
    })
    .listen(8000);

  console.log(`工作进程 ${process.pid} 已启动`);
}
```

#### 启动情况
8核故而启动8个工作进程：<br />
![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1546568627218-48701461-fddf-4855-905f-7e7f9c8bbc21.png#align=left&display=inline&height=203&linkTarget=_blank&name=image.png&originHeight=203&originWidth=501&size=29125&width=501)

#### 访问请求

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1546568425979-8385d2fd-c39d-4c6e-800c-73b1ce664906.png#align=left&display=inline&height=77&linkTarget=_blank&name=image.png&originHeight=77&originWidth=360&size=4683&width=360)

#### 关闭某个工作进程
cluster中注册了exit事件，当任何一个工作进程关闭时，都将触发exit事件：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1546568723098-b1af7b63-bc45-456f-a1a8-b18c4b9199f9.png#align=left&display=inline&height=232&linkTarget=_blank&name=image.png&originHeight=232&originWidth=550&size=32186&width=550)

可以在exit事件中重启一个新的工作进程来继续处理

## Egg 中的多进程模型
作为企业级的解决方案，还需要考虑：
* Worker进程异常退出如何处理？
* 多个Work进程之间如何共享资源？
* 多个Work进程之间如何调度？
* 等等等等

### 守护进程
健壮性是开发线上服务必须要考虑的问题，除了保证代码质量之外，框架一般还需要有一定的机制来保证在极端情况下的可用性。在NodeJS中，进程退出一般可以分为两类：

#### 未捕获异常
> [https://nodejs.org/dist/latest-v6.x/docs/api/process.html#process_event_uncaughtexception](https://nodejs.org/dist/latest-v6.x/docs/api/process.html#process_event_uncaughtexception)


这个在官方文档中说的很清楚：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1546859593254-d3edd8be-b79a-4f42-8acd-dcca48026bf4.png#align=left&display=inline&height=405&linkTarget=_blank&name=image.png&originHeight=404&originWidth=1624&size=86429&width=1624)

简而言之，就是当遇到uncaughtException时，应当是在进程结束前执行已分配资源的清理工作（如文件描述符、句柄等），而不应当恢复。在Egg中，当Worker进程遇到uncaughtException时，会让Worker进程优雅的退出：

* Worker进程
  * 关闭异常Worker进程所有的TCP Server（将已有的连接快速断开，且不再接收新的连接）
  * 断开与Master进程的IPC通道
* Master 立即fork一个新的Worker进程
* 异常Worker等待一段时间，处理完已接受的请求后退出

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1546860010966-248afaf5-73fe-4b34-88cc-3d1642311d4d.png#align=left&display=inline&height=358&linkTarget=_blank&name=image.png&originHeight=358&originWidth=568&size=13419&width=568)

#### OOM、系统异常
当一个进程出现异常导致crash 或 OOM被系统kill时——此时没有机会来让进程继续运行，当前进程直接退出，Master进程立即fork一个新的Worker进程

### Agent
除Master进程及Worker进程外，还有一种进程——Agent：来处理每个Worker都要做，但是一起做又可能会发生错误的场景。比如日志文件的生成，在多进程模式下可能会发生资源访问冲突。<br />可以将Agent进程视为其他Worker进程的代理，Agent进程不对外提供服务，只处理Worker进程的公共事务

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1546860349198-758999ed-ba4d-403c-aa2c-d38378494948.png#align=left&display=inline&height=250&linkTarget=_blank&name=image.png&originHeight=250&originWidth=403&size=7817&width=403)

#### 启动时序

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1546860387762-2b18cd9d-3b03-481a-9046-46740303bd0a.png#align=left&display=inline&height=316&linkTarget=_blank&name=image.png&originHeight=316&originWidth=463&size=13108&width=463)

* Master 启动后先fork Agent进程
* Agent 初始化成功后，通过IPC通过通知Master
* Master fork 多个 Worker
* Worker 初始化成功后，同样通知Master
* 所有进程初始化成功后，Master 通知 Agent 及 Worker 应用启动成功

#### 注意
* 由于Worker依赖Agent，等Agent初始化完成后才能fork Worker
* Agent 虽然能对Worker提供服务，但是业务相关的工作不应放到Agent中
* 由于Agent的定位，需要保证相对稳定——当发生未捕获异常时，框架不会像处理Worker进程一样退出重启——而是记录异常日志、报警等待处理

#### Agent使用
> 可参考启动自定义：[https://eggjs.org/zh-cn/basics/app-start.html](https://eggjs.org/zh-cn/basics/app-start.html)

可以在应用或插件根目录下的agent.js中实现逻辑

```
// agent.js
module.exports = agent => {
  // 在这里写你的初始化逻辑

  // 也可以通过 messenger 对象发送消息给 App Worker
  // 但需要等待 App Worker 启动成功后才能发送，不然很可能丢失
  agent.messenger.on('egg-ready', () => {
    const data = { ... };
    agent.messenger.sendToApp('xxx_action', data);
  });
};

// app.js
module.exports = app => {
  app.messenger.on('xxx_action', data => {
    // ...
  });
};
```

在本地demo中简单修改了agent.js中的data的内容，启动后可以看到对应输出（使用npm run dev的方式启动，慕默认只会启动一个工作进程）：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1546862676218-1015f6c5-8dbb-4162-8315-710c40735e52.png#align=left&display=inline&height=67&linkTarget=_blank&name=image.png&originHeight=67&originWidth=843&size=31962&width=843)

### Master VS Agent VS Worker

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1546861818195-46510772-cc56-4ada-99a9-25374f81f9bc.png#align=left&display=inline&height=213&linkTarget=_blank&name=image.png&originHeight=213&originWidth=781&size=20894&width=781)

#### Master
在Egg的架构下，Master进程承担了进程管理的工作——不运行任何业务代码，只需要运行一个Master进程即可——Master进程会负责Agent、Worker进程的初始化及重启；同时Master进程的稳定性是极高的，这点与Nginx也比较类似。

#### Agent
在大部分场景下，无需考虑Agent进程的存在——但是总会遇到一些场景，代码只允许在一个进程上，此时就是Agent发挥作用的时候了。

#### Worker
Worker进程负责处理真正的用户请求，除此之外，Egg的定时任务也提供了只让一个Worker进程运行的能力——所以能让定时任务解决的问题，就不要让Agent来处理

## 资料
* Egg中的文档：[https://eggjs.org/zh-cn/core/cluster-and-ipc.html](https://eggjs.org/zh-cn/core/cluster-and-ipc.html)
* 官方文档：[https://nodejs.org/api/cluster.html](https://nodejs.org/api/cluster.html) （中文文档：[http://nodejs.cn/api/cluster.html](http://nodejs.cn/api/cluster.html)）
* Cluster实现原理：[https://cnodejs.org/topic/56e84480833b7c8a0492e20c](https://cnodejs.org/topic/56e84480833b7c8a0492e20c)
