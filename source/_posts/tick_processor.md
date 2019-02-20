---
title: Tick Processor（性能分析）
date: 2018/07/14 22:00:00
tags:
  - Tick Processor
  - 《NodeJS 调试指南》
categories: NodeJS
---

V8 内置了一个性能分析工具：Tick  Processor，可以记录JS/C/C++代码的堆栈信息。Tick Processor 默认是关闭的，可以通过 **--prof** 开启。

## 同步调用

```
const crypto = require('crypto')

function hash (password) {
  const salt = crypto.randomBytes(128).toString('base64')
  const hash = crypto.pbkdf2Sync(password, salt, 10000, 64, 'sha512')
  return hash
}

console.time('pbkdf2Sync')
for (let i = 0; i < 100; i++) {
  hash('random_password')
}
console.timeEnd('pbkdf2Sync')
```

<!-- more -->

### 运行

```
node --prof .\app.js
pbkdf2Sync: 1103.329ms
```

运行100次hash函数共计1103ms，并且在当前目录多了一个isolate-xxx-v8.log——记录V8的性能日志。

### --prof-process 分析

```
node --prof-process .\isolate-000001F7B8B0F7E0-v8.log
```

结果如下：

```
Statistical profiling result from .\isolate-000001F7B8B0F7E0-v8.log, (667 ticks, 2 unaccounted, 0 excluded).

 [Shared libraries]:
   ticks  total  nonlib   name
    616   92.4%          D:\Install\NodeJs\node.exe
     42    6.3%          C:\WINDOWS\SYSTEM32\ntdll.dll
      3    0.4%          C:\WINDOWS\System32\KERNEL32.DLL
      1    0.1%          C:\WINDOWS\System32\win32u.dll

 [JavaScript]:
   ticks  total  nonlib   name
      1    0.1%   20.0%  LazyCompile: ~debuglog util.js:232:18
      1    0.1%   20.0%  LazyCompile: ~Module.load module.js:556:33
      1    0.1%   20.0%  Builtin: TypedArrayInitializeWithBuffer

 [C++]:
   ticks  total  nonlib   name

 [Summary]:
   ticks  total  nonlib   name
      3    0.4%   60.0%  JavaScript
      0    0.0%    0.0%  C++
      3    0.4%   60.0%  GC
    662   99.3%          Shared libraries
      2    0.3%          Unaccounted

 [C++ entry points]:
   ticks    cpp   total   name

 [Bottom up (heavy) profile]:
  Note: percentage shows a share of a particular caller in the total
  amount of its parent calls.
  Callers occupying less than 1.0% are not shown.

   ticks parent  name
    616   92.4%  D:\Install\NodeJs\node.exe
    605   98.2%    D:\Install\NodeJs\node.exe
    547   90.4%      LazyCompile: ~pbkdf2 crypto.js:691:16
    547  100.0%        LazyCompile: ~exports.pbkdf2Sync crypto.js:686:30
    547  100.0%          LazyCompile: ~hash F:\web\DemoTest\NodeJS\tick-processor\app.js:3:14
    547  100.0%            Script: ~<anonymous> F:\web\DemoTest\NodeJS\tick-processor\app.js:1:11
     27    4.5%      LazyCompile: ~Resolver dns.js:245:14
     27  100.0%        Script: ~<anonymous> dns.js:1:11
     27  100.0%          LazyCompile: ~NativeModule.compile bootstrap_node.js:588:44
     27  100.0%            LazyCompile: ~NativeModule.require bootstrap_node.js:520:34
     17    2.8%      LazyCompile: ~runInThisContext bootstrap_node.js:499:28
     17  100.0%        LazyCompile: ~NativeModule.compile bootstrap_node.js:588:44
     17  100.0%          LazyCompile: ~NativeModule.require bootstrap_node.js:520:34
      5   29.4%            Script: ~<anonymous> module.js:1:11
      1    5.9%            Script: ~<anonymous> util.js:1:11
      1    5.9%            Script: ~<anonymous> tty.js:1:11
      1    5.9%            Script: ~<anonymous> timers.js:1:11
      1    5.9%            Script: ~<anonymous> readline.js:1:11
      1    5.9%            Script: ~<anonymous> internal/trace_events_async_hooks.js:1:11
      1    5.9%            Script: ~<anonymous> internal/loader/ModuleRequest.js:1:11
      1    5.9%            Script: ~<anonymous> fs.js:1:11
      1    5.9%            Script: ~<anonymous> _stream_readable.js:1:11
      1    5.9%            LazyCompile: ~startup bootstrap_node.js:12:19
      1    5.9%            LazyCompile: ~setup_performance internal/process.js:14:27
      1    5.9%            LazyCompile: ~setupNextTick internal/process/next_tick.js:49:23
      1    5.9%            LazyCompile: ~setupGlobalVariables bootstrap_node.js:255:32

     42    6.3%  C:\WINDOWS\SYSTEM32\ntdll.dll
```

输出结果共计六部分：
* Shared libraries
* JavaScript：JS代码执行所占用的CPU ticks
* C++：C++代码执行所占用的CPU ticks
* Summary：各部分的占比
* C++ entry points
* Bottom up (heavy) profile：所有CPU占用时间从大到小的函数及堆栈信息，小于1%的不显示

**90.4%** 的时间在 **crypto.js** 文件的 **pbkdf2Sync** 函数上，调用位置在tick-processor\app.js:3:14，即hash函数。

## 异步调用

```
const crypto = require('crypto')

function hash (password, cb) {
  const salt = crypto.randomBytes(128).toString('base64')
  crypto.pbkdf2(password, salt, 10000, 64, 'sha512', cb)
}

let count = 0
console.time('pbkdf2')
for (let i = 0; i < 100; i++) {
  hash('random_password', () => {
    count++
    if (count === 100) {
      console.timeEnd('pbkdf2')
    }
  })
}
```

### 运行

```
node --prof .\async-app.js
pbkdf2: 386.013ms
```

### 分析

```
Statistical profiling result from .\isolate-000002C7775A1640-v8.log, (305 ticks, 0 unaccounted, 0 excluded).

 [Shared libraries]:
   ticks  total  nonlib   name
    232   76.1%          C:\WINDOWS\SYSTEM32\ntdll.dll
     66   21.6%          D:\Install\NodeJs\node.exe
      5    1.6%          C:\WINDOWS\System32\KERNEL32.DLL
      1    0.3%          C:\WINDOWS\System32\win32u.dll

 [JavaScript]:
   ticks  total  nonlib   name
      1    0.3%  100.0%  LazyCompile: ~hash F:\web\DemoTest\NodeJS\tick-processor\async-app.js:11:29

 [C++]:
   ticks  total  nonlib   name

 [Summary]:
   ticks  total  nonlib   name
      1    0.3%  100.0%  JavaScript
      0    0.0%    0.0%  C++
      0    0.0%    0.0%  GC
    304   99.7%          Shared libraries

 [C++ entry points]:
   ticks    cpp   total   name

 [Bottom up (heavy) profile]:
  Note: percentage shows a share of a particular caller in the total
  amount of its parent calls.
  Callers occupying less than 1.0% are not shown.

   ticks parent  name
    232   76.1%  C:\WINDOWS\SYSTEM32\ntdll.dll

     66   21.6%  D:\Install\NodeJs\node.exe
     59   89.4%    D:\Install\NodeJs\node.exe
     24   40.7%      LazyCompile: ~Resolver dns.js:245:14
     24  100.0%        Script: ~<anonymous> dns.js:1:11
     24  100.0%          LazyCompile: ~NativeModule.compile bootstrap_node.js:588:44
     24  100.0%            LazyCompile: ~NativeModule.require bootstrap_node.js:520:34
     15   25.4%      LazyCompile: ~runInThisContext bootstrap_node.js:499:28
     15  100.0%        LazyCompile: ~NativeModule.compile bootstrap_node.js:588:44
     15  100.0%          LazyCompile: ~NativeModule.require bootstrap_node.js:520:34
      4   26.7%            Script: ~<anonymous> module.js:1:11
      2   13.3%            Script: ~<anonymous> util.js:1:11
      2   13.3%            LazyCompile: ~setupGlobalTimeouts bootstrap_node.js:300:31
      1    6.7%            Script: ~<anonymous> tty.js:1:11
      1    6.7%            Script: ~<anonymous> stream.js:1:11
      1    6.7%            LazyCompile: ~startup bootstrap_node.js:12:19
      1    6.7%            LazyCompile: ~setupNextTick internal/process/next_tick.js:49:23
      1    6.7%            LazyCompile: ~setupGlobalConsole bootstrap_node.js:310:30
      1    6.7%            LazyCompile: ~createWritableStdioStream internal/process/stdio.js:142:35
      1    6.7%            LazyCompile: ~Module._load module.js:448:24
      2    3.4%      LazyCompile: ~hash F:\web\DemoTest\NodeJS\tick-processor\async-app.js:3:14
      2  100.0%        Script: ~<anonymous> F:\web\DemoTest\NodeJS\tick-processor\async-app.js:1:11
      2  100.0%          LazyCompile: ~Module._compile module.js:609:37
      2  100.0%            LazyCompile: ~Module._extensions..js module.js:661:37
      1    1.7%      Script: ~<anonymous> url.js:1:11
      1  100.0%        LazyCompile: ~NativeModule.compile bootstrap_node.js:588:44
      1  100.0%          LazyCompile: ~NativeModule.require bootstrap_node.js:520:34
      1  100.0%            Script: ~<anonymous> internal/loader/ModuleRequest.js:1:11
      1    1.7%      Script: ~<anonymous> perf_hooks.js:1:11
      1  100.0%        LazyCompile: ~NativeModule.compile bootstrap_node.js:588:44
      1  100.0%          LazyCompile: ~NativeModule.require bootstrap_node.js:520:34
      1  100.0%            LazyCompile: ~setup_performance internal/process.js:14:27
      1    1.7%      Script: ~<anonymous> path.js:1:11
      1  100.0%        LazyCompile: ~NativeModule.compile bootstrap_node.js:588:44
      1  100.0%          LazyCompile: ~NativeModule.require bootstrap_node.js:520:34
      1  100.0%            Script: ~<anonymous> fs.js:1:11
      1    1.7%      Script: ~<anonymous> net.js:1:11
      1  100.0%        LazyCompile: ~NativeModule.compile bootstrap_node.js:588:44
      1  100.0%          LazyCompile: ~NativeModule.require bootstrap_node.js:520:34
      1  100.0%            Script: ~<anonymous> tty.js:1:11
      1    1.7%      Script: ~<anonymous> internal/util.js:1:11
      1  100.0%        LazyCompile: ~NativeModule.compile bootstrap_node.js:588:44
      1  100.0%          LazyCompile: ~NativeModule.require bootstrap_node.js:520:34
      1  100.0%            Script: ~<anonymous> internal/encoding.js:1:11
      1    1.7%      Script: ~<anonymous> internal/url.js:1:11
      1  100.0%        LazyCompile: ~NativeModule.compile bootstrap_node.js:588:44
      1  100.0%          LazyCompile: ~NativeModule.require bootstrap_node.js:520:34
      1  100.0%            Script: ~<anonymous> module.js:1:11
      1    1.7%      Script: ~<anonymous> crypto.js:1:11
      1  100.0%        LazyCompile: ~NativeModule.compile bootstrap_node.js:588:44
      1  100.0%          LazyCompile: ~NativeModule.require bootstrap_node.js:520:34
      1  100.0%            LazyCompile: ~Module._load module.js:448:24
      1    1.7%      Script: ~<anonymous> bootstrap_node.js:10:10
      1    1.7%      LazyCompile: ~stat module.js:50:14
      1  100.0%        LazyCompile: ~Module._findPath module.js:182:28
      1  100.0%          LazyCompile: ~Module._resolveFilename module.js:514:35
      1  100.0%            LazyCompile: ~Module._load module.js:448:24
      1    1.7%      LazyCompile: ~setupProcessFatal bootstrap_node.js:358:29
      1  100.0%        LazyCompile: ~startup bootstrap_node.js:12:19
      1  100.0%          Script: ~<anonymous> bootstrap_node.js:10:10
      1    1.7%      LazyCompile: ~makeSafe internal/safe_globals.js:15:18
      1  100.0%        Script: ~<anonymous> internal/safe_globals.js:1:11
      1  100.0%          LazyCompile: ~NativeModule.compile bootstrap_node.js:588:44
      1  100.0%            LazyCompile: ~NativeModule.require bootstrap_node.js:520:34
      1    1.7%      LazyCompile: ~inherits util.js:932:18
      1  100.0%        Script: ~<anonymous> _stream_duplex.js:1:11
      1  100.0%          LazyCompile: ~NativeModule.compile bootstrap_node.js:588:44
      1  100.0%            LazyCompile: ~NativeModule.require bootstrap_node.js:520:34
      1    1.7%      LazyCompile: ~fs.openSync fs.js:642:23
      1  100.0%        LazyCompile: ~fs.readFileSync fs.js:548:27
      1  100.0%          LazyCompile: ~Module._extensions..js module.js:661:37
      1  100.0%            LazyCompile: ~Module.load module.js:556:33
      1    1.7%      LazyCompile: ~createWriteReq net.js:786:24
      1  100.0%        LazyCompile: ~Socket._writeGeneric net.js:711:42
      1  100.0%          LazyCompile: ~Socket._write net.js:782:35
      1  100.0%            LazyCompile: ~doWrite _stream_writable.js:389:17
      1    1.7%      LazyCompile: ~createWritableStdioStream internal/process/stdio.js:142:35
      1  100.0%        LazyCompile: ~getStdout internal/process/stdio.js:12:21
      1  100.0%          Script: ~<anonymous> console.js:1:11
      1  100.0%            LazyCompile: ~NativeModule.compile bootstrap_node.js:588:44
      1    1.7%      LazyCompile: ~createUnsafeArrayBuffer buffer.js:90:33
      1  100.0%        LazyCompile: ~createPool buffer.js:99:20
      1  100.0%          Script: ~<anonymous> buffer.js:1:11
      1  100.0%            LazyCompile: ~NativeModule.compile bootstrap_node.js:588:44
      1    1.7%      LazyCompile: ~WritableState _stream_writable.js:41:23
      1  100.0%        LazyCompile: ~Writable _stream_writable.js:194:18
      1  100.0%          LazyCompile: ~Duplex _stream_duplex.js:47:16
      1  100.0%            LazyCompile: ~Socket net.js:185:16
      1    1.7%      LazyCompile: ~InnerArrayJoin native array.js:274:24
      1  100.0%        LazyCompile: ~join native array.js:287:46
      1  100.0%          LazyCompile: ~setupConfig internal/process.js:113:21
      1  100.0%            LazyCompile: ~startup bootstrap_node.js:12:19

      5    1.6%  C:\WINDOWS\System32\KERNEL32.DLL
```

此时在 **Bottom up (heavy) profile** 没有pbkdf2相关堆栈信息

## Web UI
V8 同时还提供了一个 Web 可视化工具来查看生产的v8 日志

### 生成 JSON 文件

```
node --prof-process --preprocess .\isolate-000001F7B8B0F7E0-v8.log > v8.json
```

### 准备工作
* 克隆 V8 仓库（如下载速度过慢直接选择Download Zip）：git clone https://github.com/v8/v8.git
* 在浏览器打开：v8/tools/profview/index.html
* 选择上一部分生成的 v8.json

### 预览结果

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550651686492-f2187b87-2335-40ea-91b6-eb4b56cf5246.png#align=left&display=inline&height=840&linkTarget=_blank&name=image.png&originHeight=840&originWidth=1898&size=170569&width=1898)

在Bottom up 视图中，依然可以看到。

另外：
* 上图中的展示了CPU的timeline：X 轴表示时间，Y轴表示当前时间点不同部分占用CPU的比例
* 上图中的表展示了当前时间段内CPU占用从大到小降序排列的函数；点击任意函数，timeline 底部会展示该函数的执行时间分布

## 资料
* [https://v8.dev/docs/profile](https://v8.dev/docs/profile)
* [https://stackoverflow.com/questions/23934451/how-to-read-nodejs-internal-profiler-tick-processor-output](https://stackoverflow.com/questions/23934451/how-to-read-nodejs-internal-profiler-tick-processor-output)
* 《NodeJS》 调试指南
