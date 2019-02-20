---
title: v8-profiler（分析CPU使用情况）
date: 2018/07/8 21:00:00
tags:
  - v8-profiler
  - 《NodeJS 调试指南》
categories: NodeJS
---

NodeJS 是一个基于Chrome V8 引擎的 JS 运行环境，而V8暴露了一些 profiler API，可以通过 [v8-profiler](https://github.com/node-inspector/v8-profiler) 收集一些运行时数据（如CPU、内存）。

## 使用v8-profiler
### 初始化代码

```
const fs = require('fs')
const crypto = require('crypto')
const Bluebird = require('bluebird')
const profiler = require('v8-profiler')
const Paloma = require('paloma')
const app = new Paloma()

app.route({ method: 'GET', path: '/encrypt', controller: function encryptRouter (ctx) {
  const password = ctx.query.password || 'test'
  const salt = crypto.randomBytes(128).toString('base64')
  const encryptedPassword = crypto.pbkdf2Sync(password, salt, 10000, 64, 'sha512').toString('hex')

  ctx.body = encryptedPassword
}})

app.route({ method: 'GET', path: '/cpuprofile', async controller (ctx) {
   //Start Profiling
   profiler.startProfiling('CPU profile')
   await Bluebird.delay(30000)
   //Stop Profiling after 30s
   const profile = profiler.stopProfiling()
   profile.export()
     .pipe(fs.createWriteStream(`cpuprofile-${Date.now()}.cpuprofile`))
     .on('finish', () => profile.delete())
   ctx.status = 204
}})
 
app.listen(3000)
```

<!-- more -->

### 说明
其中，GET /encrypt 中有一个 CPU 密集型的计算函数：**crypto.pbkdf2Sync**，GET /cpuprofile 收集30s的 V8 log并dump到文件中。

* 调用 cpuprofile接口：curl localhost:3000/cpuprofile
* 调用 encrypt：ab -c 20 -n 2000 "http://localhost:3000/encrypt?password=123456"

## Chrome DevTools
Chrome 自带了分析 CPU profile 日志的工具。打开 Chrome -> 调出开发者工具（DevTools） -> 单击右上角三个点的按钮 -> More tools -> JavaScript Profiler -> Load。

### 三种模式

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1548140078872-2725ac4d-e2cc-413a-a772-3953b5d2dc7c.png#align=left&display=inline&height=168&linkTarget=_blank&name=image.png&originHeight=168&originWidth=829&size=28868&width=829)

* Chart：显示按时间顺序排列的火焰图。
* Heavy (Bottom Up)：按照函数对性能的影响排列，同时可以检查函数的调用路径。
* Tree (Top Down)：显示调用结构的总体状况，从调用堆栈的顶端开始。

### Tree（Top Down）模式
按Totol Time 降序：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1548140155457-53f2dbd0-8710-4922-8e08-a540d0ec9786.png#align=left&display=inline&height=540&linkTarget=_blank&name=image.png&originHeight=540&originWidth=1462&size=75701&width=1462)

定位到encryptRoute路由中的exports.pbkdf2Sync方法占用了绝大部分CPU时间

#### 列说明
* Self Time：函数调用所耗费的时间，仅包含函数本身的声明，不包含任何子函数的执行时间。
* Total Time：函数调用所耗费的总时间，包含函数本身的声明及所有子函数执行时间。即：父函数的 Total Time = 父函数的 Self Time + 所有子函数的 Total Time。
* Function：函数名及路径，可展开查看子函数。

## 火焰图
### 安装 flamegraph 模块

```
npm i flamegraph -g
```

### 生成svg文件

```
flamegraph -t cpuprofile -f cpuprofile-xxx.cpuprofile -o cpuprofile.svg
```

### 问题

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1548141347959-be11d3d7-69b8-49a0-a306-97bc13072e42.png#align=left&display=inline&height=168&linkTarget=_blank&name=image.png&originHeight=168&originWidth=568&size=15995&width=568)

将 & 替换为 &amp; 即可

### 结果

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1548141438729-8e1b8d9c-f932-445d-be5b-003de3be37a7.png#align=left&display=inline&height=347&linkTarget=_blank&name=image.png&originHeight=347&originWidth=1195&size=70912&width=1195)

定位到v8-profile.js中的11行的encryptRouter这个路由中的 exports.pbkdf2Sync 方法占用了大部分的CPU时间

## v8-analytics
> [https://github.com/hyj1991/v8-analytics/blob/master/README_ZH.md](https://github.com/hyj1991/v8-analytics/blob/master/README_ZH.md)


[v8-analytics](https://github.com/hyj1991/v8-analytics) 是社区开源的一个解析 v8-profiler 和 heapdump 等模块生成的 CPU 和 heap-memory 日志的工具。它提供以下功能：
* 将 V8 引擎逆优化或者优化失败的函数标红展示，并展示优化失败的原因
* 在函数执行时长超过预期时标红展示
* 展示当前项目中可疑的内存泄漏点

试一下 **函数执行时长超过预期标红！**

### 安装 v8-analytics 模块

```
npm i v8-analytics -g
```

### 输出

```
va timeout cpuprofile-xxx.cpuprofile 100 --only
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1548142349639-1206f733-d6c5-44c8-86d3-2fe894f825b4.png#align=left&display=inline&height=506&linkTarget=_blank&name=image.png&originHeight=506&originWidth=1001&size=133504&width=1001)

## 参考
* [https://github.com/nswbmw/node-in-debugging/blob/master/1.2 v8-profiler.md](https://github.com/nswbmw/node-in-debugging/blob/master/1.2%20v8-profiler.md)
* [https://developers.google.com/web/tools/chrome-devtools/rendering-tools/js-execution](https://developers.google.com/web/tools/chrome-devtools/rendering-tools/js-execution)
* [https://github.com/node-inspector/v8-profiler](https://github.com/node-inspector/v8-profiler)
* 《NodeJS》 调试指南

