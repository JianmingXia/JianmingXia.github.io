---
title: 你不知道的JavaScript（中卷）——异步和性能
date: 2017/11/28 12:00:00
tags:
  - JavaScript
categories: 阅读笔记
---

## 异步：现在与将来

### 分块的程序

新手JS程序员的问题：程序中将来执行的部分并不一定在现在运行的部分执行完后就立即执行

#### 异步控制台

由于IO可能会阻塞，故而浏览器中的console.*输出并不一定是同步阻塞的，遇见这种情况可以打断点解决

<!-- more -->

### 事件循环

### 并行线程

术语“异步”和“并行”常常被混为一谈，但实际上它们的意义完全不同。记住：异步是现在和将来的时间间隙，而并行是关于能够同时发生的事情

并行计算最常见的工具就是进程和线程。进程和线程独立运行，并可能同时运行：在不同的处理器，甚至不同的计算器上，但多个线程能够共享单个进程的内存。

JS从不跨线程共享数据，这意味着不需要考虑这一层次的不确定性——多线程的场景，如果没有锁机制，并行操作共享内存是非常可怕的一件事情

#### 完整运行

在JS中，这种函数（两个Ajax请求的回调）顺序的不确定性就是通常所说的竞态条件。

> 如果JS中的某个函数由于某种原因不具有完整运行特性。。。

### 并发

#### 非交互

如果进程间没有相互影响的话，不确定性是完全可以接受的

#### 交互

不确定性导致竞态条件bug

#### 协作

取到一个长期运行的“进程”，将其分割成多个步骤或多批任务

### 任务

总是挂在事件循环队列的每个tick之后的一个队列

### 语句顺序

编译器语句重排序几乎就是并发和交互的微型隐喻

## 回调

为什么更高级的异步模式是必需和备受期待的？？？

### continuation

代码变得更加难以理解、追踪、调试和维护

### 顺序的大脑

#### 执行与计划

> 可以把我们的大脑看成是类似于单线程运行的事件循环队列，就像JS引擎那样。

通过回调表达异步的方式可能并不能很好地映射到同步的大脑计划行为

#### 嵌套回调与链式回调

复杂的回调会导致：回调地狱（callback hell），也可被称为毁灭金字塔（pyramid of doom)

回调方式最主要的缺陷：对于它们在代码中表达异步的方式，我们的大脑需要努力才能同步上

### 信任问题

#### 五个回调的故事

不要过于相信第三方的回调

#### 不只是别人的代码

回调最大的问题是控制反转，它会导致信任链的完全断裂

### 省点回调

回调设计存在几个变体，意欲解决一些信任问题

- 分离回调：一个用于成功通知，一个用于出错通知
- error-first风格：又称为Node风格

## Promise

回调中异步和管理并发的两个主要缺陷：

- 顺序性
- 可信任性

### 什么是Promise

> 有一类工具：通过某人使用它的方式，很容易分辨出他是真正理解了这门技术，还是仅仅学习和使用API而已（共勉）

#### 未来值

##### 现在值与将来值

##### Promise值

Promise是一种封装和组合未来值的易于复用的机制

#### 完成事件

##### Promise "事件"

一旦Promise决议，无论是现在还是将来，下一个步骤总是相同的，可以按照需要多次查看

### 具有then方法的鸭子模型

如何判断某个值是否为Promise呢？ **p instanceof Promise?**

- Promise值可能从其它浏览器（iframe等）窗口接收到
- 库或框架可能会实现自己的Promise，而非使用ES6 Promise

### Promise 信任问题

回调编码的信任问题，如将一个回到传入工具foo(..)时可能会出现的问题：

- 调用回调过早
- 调用回调过晚（或不被调用）
- 调用回调次数过少或过多
- 未能传递所需的环境和参数
- 吞掉可能出现的错误和异常

#### 调用过早

一个任务有时同步完成，有时异步完成，这可能会导致竞态条件

```js
var p = new Promise(function(resolve) {
  resolve(42);
});

p.then(function(num) {
  console.log(num);
});

console.log(1);
// 1 42
```

而Promise即使是立即完成的Promise，也无法被同步观察到；提供给then的回调总是会被异步调用

#### 调用过晚

```js
var p = new Promise(function(resolve) {
  resolve();
});

p.then(function() {
  p.then(function() {
    console.log("C");
  });
  console.log("A");
});

p.then(function() {
  console.log("B");
});
// A B C
```

一旦Promise决议后，这个Promise上所有通过then(..)注册的回调都会在下一个异步时机点上依次被立即调用。

此处“C”无法打断或抢占“B”

##### Promise 调度技巧

两个独立Promise上链接的回调的相对顺序无法可靠预测。

如果两个Promise p1 和 p2都已经决议，那么p1.then 及 p2.then最终会先调用p1的回调，然后是p2。但是还有一些微妙的场景可能并非如此：

```js
var p3 = new Promise(function(resolve) {
  resolve("B");
});

var p1 = new Promise(function(resolve) {
  resolve(p3);
});

var p2 = new Promise(function(resolve) {
  resolve("A");
});

p1.then(function(v) {
  console.log(v);
});

p2.then(function(v) {
  console.log(v);
});
// A B
```

要避免这样的细微区别带来的问题，永远不应该依赖于不同Promise间回调的顺序和调度

#### 回调未调用

没有任何东西能够阻止Promise向你通知它的决议

使用一种“竞态”的高级抽象机制，来避免Promise本身永远不被决议的问题

#### 调用次数过少或过多

如果Promise试图调用resolve和reject多次，或者试图两者都调用，那么这个Promise将只会接收第一次决议，并默默忽略任何后续调用

#### 未能传递参数/环境值

Promise至多只能有一个决议值（完成或拒绝）

如果没有用任何值显示决议，那么这个值就是undefined；希望要传递多个参数，第一个参数后的所有参数都会被忽略

#### 吞掉错误或异常

```js
var p = new Promise(function(resolve, reject) {
  foo.bar();

  resolve(42);
});

p.then(
  function(v) {
    console.log(v);
  },
  function(err) {
    console.log(err);
  }
);
```

foo.bar()中发生了JS异常导致Promise被拒绝

#### 是可信任的Promise吗

Promise并没有完全摆脱回调，它们只是改变了传递回调的位置

Promise.resolve(..)可以接受任何thenable，将其解封为它的非thenable值；从Promise.resolve()得到的是一个真正的Promise，是一个可以信任的值

如果传入的已经是一个真正的Promise，那么得到的就是它本身，通过Promise.resolve()过滤来获得可信任性完全没有坏处

#### 建立信任

### 链式流

- 每次对Promise调用then，它都会创建并返回一个新的Promise，可以将其链接起来
- 不过从then调用的完成回调返回的值是什么，它都会被自动设置为链接Promise的完成？？

```js
var p = Promise.resolve(21);

var p2 = p.then(function(v) {
  console.log(v);

  return v * 2;
});

p2.then(function(v) {
  console.log(v);
});
// 21 42
```

如果还有更多呢？

```js
var p = Promise.resolve(21);

p.then(function(v) {
  console.log(v);

  return v * 2;
}).then(function(v) {
  console.log(v);
});
```

使链式流程控制可行的Promise固有特性：

- 调用Promise中的then()会自动创建一个新的Promise从调用返回
- 在完成或拒绝处理函数内部，如果返回一个值或抛出一个异常，新返回的Promise就相应的决议
- 如果完成或拒绝处理函数返回一个Promise，它将会被展开，不管它的决议是什么，都会成为当前then返回的链接Promise的决议

#### 术语：决议、完成以及拒绝

- resolve
- fulfill
- reject

reject不会像resolve一样进行展开，如果向reject传入一个Promise/thenable值，它会把这个值原封不动地设置为拒绝理由

### 错误处理

try..catch无法跨异步

```js
function foo() {
  setTimeout(function() {
    baz.bar();
  }, 100);
}

try {
  foo();
} catch (err) {
  console.log(err); // 捕捉不到
}
```

Promise没有采用流行的error-first回调设计风格，而是使用了分离回调风格：一个回调用于完成情况，另一个用于拒绝情况

#### 绝望的陷阱

仅在Promise末尾添加catch可行么？

#### 处理未捕获情况

Promise中的done？

#### 成功的坑

### Promise模式

#### Promise.all([..])

> 如果想要同时执行两个或更多，如何实现？

从.all返回的主promise仅在所有的成员promise都完成后才会完成；如果有任意一个被拒绝，主promise会立即被拒绝，并丢弃其他所有promise的全部结果

#### Promise.race([..])

> 如果只响应第一个Promise？

一旦有任何一个Promise决议为完成，.race就会完成；一旦有任何一个Promise决议为拒绝，.race就会拒绝

##### 超时竞赛

结合setTimeout使用

##### finally

做一个清理的工作

#### all 和 race的变体

##### none

与all类似，不过完成和拒绝的情况互换了：所有的Promise都要被拒绝

##### any

与all类似，但是会忽略拒绝，只需要完成一个而不是全部

##### first

类似于any的竞争，即只要第一个Promise完成，忽略后续的任何拒绝和完成

##### last

类似于first，但却是只有最后一个完成胜出

#### 并发迭代

### Promise API 概述

#### new Promise() 构造器

new Promise中 resolve 和 reject的不同

#### Promise.resolve() 和 Promise.reject()

Promise.resolve的特殊性

#### then() 和 catch()

每个Promise实例都有then和catch方法（不是Promise API 命名空间）

##### then

then接收一个或两个参数：第一个用于完成回调，第二个用于拒绝回调；

如果两者中任何一个被省略，就会被替换为相应的默认回调：默认完成回调只是把消息传递下去，而默认拒绝回调则只是重新抛出其接收到的出错原因

##### catch

catch只接受一个拒绝回调作为参数，并自动替换默认完成回调，等价于 then(null, ..)

#### Promise.all() 和 Promise.race

```js
var p1 = Promise.resolve(42);
var p2 = Promise.resolve("Hello world");
var p3 = Promise.reject("Oops");

Promise.race([p1, p2, p3]).then(function(v) {
  console.log(v);
});

Promise.all([p1, p2, p3]).catch(function(v) {
  console.log(v);
});

Promise.all([p1, p2]).then(function(v) {
  console.log(v);
});
// 42
// Oops
// [42, "Hello world"]
```

Note: 如果向Promise.all()传入空数组，它会立即完成；而Promise.race()会挂住，且永远不会决议

### Promise 局限性

#### 顺序错误处理

Promise链中的错误很容易被无意中默默忽略掉

#### 单一值

##### 分裂值

有时可以把这一点当作是把问题分解为两个或更多Promise的信号

##### 展开/传递参数

ES6提供了更好的解决方案：解构

#### 单决议

Promise只能被决议一次

比如 如何解决事件监听的这类场景

#### 惯性

如何在现存项目(大量使用了回调地狱)中使用Promise

#### 无法取消的Promise

一旦创建了一个Promise并为其注册了完成或拒绝处理函数，如果出现某种情况使得Promise未决议，没有办法从外部停止它的进程

#### Promise 性能

Promise 性能与其带来的好处

## 生成器

回调的两个关键缺陷：

- 基于回调的异步不符合大脑对任务步骤的规划方式
- 由于控制反转，回调并不是可信任或可组合的

通过Promise，我们将控制反转反转回来，恢复了可信任性/可组合性

### 打破完整运行

> 一个函数一旦开始执行，就会运行到结束，期间不会有其他代码能够打算它并插入其间

生成器是一类特殊的函数，可以一次或多次启动和停止，并不一定非得要完成

#### 输入和输出

##### 迭代消息传递

除了能够接受参数并提供返回值外，生成器甚至提供了更强大更引入注目的内建消息输入输出能力，通过yield和next()实现

一般来说，需要的next会比yield语句多一个——因为第一个next总是启动一个生成器，并运行到第一个yield处

##### 两个问题的故事

通过yield和next()的组合，在生成器的执行过程中构成了一个双向消息传递系统

````js
function* foo(x) {
  var y = x * (yield "hello");
  return y;
}

var it = foo(6);

var res = it.next();
console.log(res.value); // "hello"

res = it.next(7);
console.log(res.value); // 42
````

#### 多个迭代器

> 同一个生成器的多个实例并发运行的最常用处并不是交互，而是在生成器没有输入的情况下，可能从某个独立连接的资源产生自己的值

```js
function* foo(x) {
  var x = yield 2;
  z++;
  var y = yield x * z;

  console.log(x, y, z);
}

var z = 1;

var it1 = foo();
var it2 = foo();

var val1 = it1.next().value;
var val2 = it2.next().value;
console.log(val1, val2); // 2 2

val1 = it1.next(val2 * 10).value;
val2 = it2.next(val1 * 5).value;
console.log(val1, val2); // 40 600

it1.next(val2 / 2); // 20 300 3
it2.next(val1 / 4); // 200 10 3
```

##### 交替执行

有兴趣的可以试一下，可以更加深入理解多个生成器如何在共享作用域上并发执行

### 生成器产生值

#### 生产者与迭代器

假定要产生一系列值，其中每个值都与前面一个有特定的关系——需要一个有状态的生产者能够记住其生成的最后一个值

#### iterable

```js
var a = [1, 3, 5, 7, 9];
var it = a[Symbol.iterator]();

console.log(it.next().value);
console.log(it.next().value);
console.log(it.next().value);
console.log(it.next().value);
console.log(it.next().value);
// 1 3 5 7 9
```

#### 生成器迭代器

```js
function* something() {
  var nextVal;

  while (true) {
    if (nextVal === undefined) {
      nextVal = 1;
    } else {
      nextVal = 3 * nextVal + 6;
    }

    yield nextVal;
  }
}

for (var v of something()) {
  console.log(v);

  if (v > 500) {
    break;
  }
}
```

- 为什么使用something()——因为这里的something是生成器，并不是iterable
- something()调用产生一个迭代器，但for..of需要的是iterable——生成器的迭代器也有Symbol.iterable函数

##### 停止生成器

在生成器中添加try..finally语句，如果需要清理资源，可以使用——break会触发finally

```js
function* something() {
  try {
    var nextVal;

    while (true) {
      if (nextVal === undefined) {
        nextVal = 1;
      } else {
        nextVal = 3 * nextVal + 6;
      }

      yield nextVal;
    }
  } finally {
    console.log("cleaning up");
  }
}
```

也可以通过.return来立即终止生成器

```js
var it = something();
for (var v of it) {
  console.log(v);

  if (v > 500) {
    console.log(it.return("hello world").value);
  }
}
```

### 异步迭代生成器

通过使用生成器，将异步作为实现细节抽象出去，以同步顺序处理。

#### 同步错误处理

在异步代码中实现看似同步的错误处理

### 生成器 + Promise

Promise + 生成器最大效用的最自然的方法就是：yield出来一个Promise，然后通过这个Promise来控制生成器的迭代器

#### 支持Promise的Ganerator Runner

##### ES7: async 与 await?

从本质上讲，ES7的提案实际上就是上述的实现

#### 生成器中的Promise并发

Promise所有的并发能力在 生成器 + Promise 中都可以使用

##### 隐藏Promise

如果想要实现一系列高级流程控制，有用的做法是：把Promise逻辑隐藏在一个只从生成器代码中调用的函数内部

> 创建代码除了要实现功能和保持性能外，尽可能使代码易于立即与维护

### 生成器委托

```js
function* foo() {
  console.log("*foo() starting");

  yield 3;
  yield 4;
  console.log("*foo() finished");
}

function* bar() {
  yield 1;
  yield 2;
  yield* foo();
  yield 5;
}

var it = bar();

console.log(it.next().value);
console.log(it.next().value);
console.log(it.next().value);
console.log(it.next().value);
console.log(it.next().value);
```

> yield *暂停了迭代控制，而不是生成器控制

#### 为什么用委托

yield委托的主要目的是代码组织，以达到与普通函数调用的对称

#### 消息委托

> need keep learning

##### 异常也被委托

#### 异步委托

#### 递归委托

### 生成器并发

### 形实转换程序

形实转换程序（thunk)：JS中的thunk是指一个用于调用另外一个函数的函数，而且没有任何参数

#### s/promise/thunk

### ES6之前的生成器

如何在前ES6中使用生成器？

#### 手工变换

深入其原理来实现

#### 自动转换

使用工具

### 小结

生成器为异步代码保持了顺序、同步、阻塞的代码模式，使得我们可以更自然的追踪代码，解决了基于回调异步的两个关键缺陷之一

## 程序性能

为什么要利用异步？——性能

### Web Worker

如果要处理一些密集型的任务，但不希望它们都在主线程运行（可能会阻塞浏览器），我们会希望JS能够以多线程的方式运行

Worker之间以及它们与主程序之间，不会共享任何作用域或资源，它们通过一个基本的事件消息机制相互联系

Note: 专用Worker和创建它的程序之间是一对一的关系——"message"事件没有任何歧义需要消除，确定它只能来自这个一对一的关系：要么来自这个Worker，要么来自主页面

> 如果需要，Worker也可以实例化自己的子Worker——subworker

#### Worker环境

在Worker内部是无法访问主程序的任何资源的：无法访问它的任何全局变量，不能访问页面的DOM和其它资源——这是一个独立的线程

可以执行网络操作、设定定时器、访问几个重要的全局变量和功能的本地副本，包括navigatar、location、JSON和applicationCache

还可以通过**importScripts**向Worker加载额外的JS脚本（脚本加载是同步的）

Web Worker适用场景：

- 处理密集型数学计算
- 大数据集排序
- 数据处理（压缩、音频分析、图像处理等）
- 高流量网络通信

#### 数据传递

在早期的Worker中，唯一的选择是将所有数据序列化到一个字符串值中：除了双向序列化导致的速度损失之外；数据需要被复制，花费两倍内存

更好的选择：

- 结构化克隆算法
- 使用Transferable对象

#### 共享Worker

**SharedWorker**

#### 模拟Web Worker

### SIMD

单指令多数据（SIMD）是一种数据并行（data parallelism）方式，与Web Worker的任务并行（task parallelism）相对。

通过SIMD，线程不再提供并行。取而代之的是，现代CPU通过数字“向量”（特定类型的数组），以及可以在所有这些数字上并行操作的指令，来提供SIMD功能，这是利用低级指令级并行的底层运算。

### asm.js

asm.js这个标签是指JS语言中可以高度优化的一个子集。通过小心避免某些难以优化的机制和模式（垃圾收集、类型强制转换等待），asm.js风格的代码可以被JS引擎识别并进行特别激进的底层优化。

#### 如何使用asm.js优化

#### asm.js模块

### 小结

本部分前四章都基于一个前提：异步编码模式使我们能够编写更高效的代码，通常能够带来非常大的改进——但是异步也只能到此为止，因为本质上还是绑定在一个单时间循环线程上

## 性能测试与调优

> 关注于微观性能，关注点在单个表达式和语句

### 性能测试

```js
var start = Date.now();
// dosomething
var end = Date.now();

console.log("Duration: ", end - start);
```

- 如果测试时间是0呢
- 平台的精度并没有达到1ms
- 这次特定的运行消耗了多长时间
- Data.now()的消耗？
- 被JS引擎优化了测试用例

#### 重复

重复执行的时间长度应该根据使用的定时器的精度而定，专门用来最小化不精确性。

#### Benchmark.js

任何有意义且可靠的性能测试都应该基于统计学上合理的实践。

### 环境为王

#### 引擎优化

### jsPerf.com

尽管在所有的JS运行环境下，Benchmark.js都可用于测试代码的性能，但有一点一定要强调，如果你想要得到可靠的测试结论的话，就需要在很多不同的环境中测试汇集测试结果。

#### 完整性检查

### 写好测试

> 本章及“完整性检查“部分已经列举了很多例子

要写好测试，需要认真分析和思考两个测试用例之间有什么区别，以及这些区别是有意还是无意的

### 微性能

当我们关注与使用性能测试来确定哪个更好时，在更大的上下文中，引擎可能并不会运行其中任何一行代码，而选择自行优化

> 实际上，在某些像V8这样的引擎中，预先缓存长度而不是让引擎为你做这件事情，会使性能稍微下降一点——不要试图和JS引擎比谁重名，对性能优化来说，你可能会输。。。

#### 不是所有的引擎都类似

通过分析V8 JS引擎的特定内部实现细节，决定编写裁剪过的JS代码来最大程度地利用V8的工作模式：

- 不要从一个函数到另外一个函数传递arguments变量，因为这样的泄露会降低函数实现速度
- 把try..catch分离到单独的函数中——浏览器对任何有try..catch的函数实行优化都有一些困难，所以将这部分移到独立的函数中意味着我们控制了反优化的害处，并让其包含的代码可以优化

#### 大局

> 过早优化是万恶之源

![](https://img.ryoma.top/YouDontKnowJS/mid_2-1.png)

"非关键路径上的优化是万恶之源"——如果代码在关键路径上，就应该优化！

### 尾调用优化

尾调用优化：Tail Call Optimization, TCO

## 附录A asynquence库

> 了解作者的异步库

## 附录B 高级异步模式

## 其它

### 生成器部分可以继续学习

需要继续深入理解