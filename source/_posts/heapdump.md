---
title: heapdump（查看内存信息）
date: 2018/07/15 18:00:00
tags:
  - heapdump
  - 《NodeJS 调试指南》
categories: NodeJS
---

之前提到 v8-profiler 可以收集 CPU 和 内存的信息，在这里我们使用 heapdump 工具来分析 NodeJS 内存信息。

## heapdump
### 测试代码

```
const heapdump = require('heapdump')
let leakObject = null
let count = 0

setInterval(function testMemoryLeak() {
  const originLeakObject = leakObject
  const unused = function () {
    if (originLeakObject) {
      console.log('originLeakObject')
    }
  }
  leakObject = {
    count: String(count++),
    leakStr: new Array(1e7).join('*'),
    leakMethod: function () {
      console.log('leakMessage')
    }
  }
}, 1000)
```
<!-- more -->

### 闭包的原理
同一个函数内部的**闭包作用域只有一个**，所有闭包共享。<br />在执行函数时，如果遇到闭包，则会创建闭包作用域的内存空间，将该闭包所用的局部变量加入；再遇到闭包时，会在之前创建的闭包作用域中添加之前未加入的局部变量。函数结束时，会清除没有被闭包作用域引用的变量

### 内存泄露
在 testMemoryLeak 中有两个闭包：
* unused
* leakMethod

进行分析：
* unused：unused 闭包引用了父作用域中的 originLeakObject 变量；若无leakMethod，会在函数结束后被清除，此时闭包作用域也会被清除
* 但是由于 leakObject 是全局变量，故而 leakObject 也成了全局变量，故而引用的闭包作用域（包含unused引用的originLeakObject）也不会是否。

随着函数不断被调用，originLeakObject指向前一次的leakObject，而 leakObject 通过 leakMethod 又引用之前的 originLeakObject，从而形成闭包引用链。而leakStr 是个超大的字符串，一直未释放故而造成内存泄露

### 解决内存泄露
在 testMemoryLeak 函数最后添加 originLeakObejct=null 即可

### 获取堆快照
> 通过 pgrep 获取当前进程id


```
kill -USR2 `pgrep -n node`
```

可以获取多个堆快照方便后续进行比较

## Chrome DevTools
可以使用 Chrome DevTools 来分析堆快照文件。通过 Chrome DevTools - Memory - Load，按生成快照的顺序加载。

### 基本介绍
点击第二个堆快照 - 点击左上角，可以看到：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550743942821-85158eca-d8a8-4437-8cc3-59c74ab9d3a5.png#align=left&display=inline&height=209&linkTarget=_blank&name=image.png&originHeight=209&originWidth=388&size=23725&status=done&width=388)

* Summary：以构造函数名 分类显示
* Comparison：比较多个堆快照之间的差异（加载单个堆快照时没有此项）
* Containment：查看整个GC的路径
* Statistics：以饼状图显示内存信息

#### Statistics

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550744110090-7f4e4243-eaa5-4f14-9126-ee58977e28ea.png#align=left&display=inline&height=400&linkTarget=_blank&name=image.png&originHeight=400&originWidth=270&size=20283&status=done&width=270)

#### Summary
表结构有四列：
* Constructor：
  * 构造函数名：如Object、Module、Socket，(array)、(string)等有括号的分别代表内置的Array、String
  * 对象个数：构造函数名后的数字
* Distance：到GC roots（GC 根对象）的距离。距离越大，则说明引用越深
* Shallow Size：对象自身的大小，不包含引用的对象
* Retained Size：对象自身的大小和引用对象的大小——当对象被GC之后可回收的大小
* Objects Count：对象个数（Chrome70 之后，对象个数在Constructor内的名称后面）

#### Retained Size & Shallow Size
Retained Size = 对象的 shallow Size + 对象树上其它子节点的Retained Size<br />Shallow Size = Retained Size中（boolean number string），因为这些类型无法引用其它值，并且始终是叶子节点

### 基本分析
单击 Retained Size 按降序显示，其中closure引用内容达到99%，继续展开：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550799406960-f29403e0-b9f9-4950-ab82-cdf74582ff1d.png#align=left&display=inline&height=608&linkTarget=_blank&name=image.png&originHeight=608&originWidth=1710&size=113026&status=done&width=1710)

其中：
* leakStr 占了4% 的内存
* leakMethod 占了 95% 的内存（第二个leakMethod 占了91%的内存）
* originLeakMethod 中 leakMethod 引用的即为 第二个 leakMethod（都是@63503，Distance 也是一致的）
* 对象保留树（即Retainers）展示了对象的GC Path（可能会随着Chrome 版本迭代有变化）

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550799819575-abfdb8f4-b323-4c48-8c58-677953c4cb75.png#align=left&display=inline&height=182&linkTarget=_blank&name=image.png&originHeight=182&originWidth=1709&size=34709&status=done&width=1709)

### 查看引用关系
继续展开下一层的 leakMethod：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550799956171-017e9b47-95a2-4766-9bcd-3f6db0080b85.png#align=left&display=inline&height=653&linkTarget=_blank&name=image.png&originHeight=653&originWidth=1698&size=126544&status=done&width=1698)

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550800091944-66842a79-1193-4897-b35f-bb05a4e81825.png#align=left&display=inline&height=548&linkTarget=_blank&name=image.png&originHeight=548&originWidth=1618&size=103832&status=done&width=1618)

可以发现：
* count=24 的 originLeakObject 中 leakMethod 函数的 context 引用了一个有 count=23 的 originLeakObject
* 同理，这个 originLeakObject 继续引用了下一层
* 每个 originLeakObject 对象上都有一个大字符串 leakStr，从而造成内存泄露

## 对比堆快照
切至 Comparison 视图下，有 #New、#Deleted、#Delta等属性：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550800216443-27dcf985-69cb-4769-abed-e5463608f31a.png#align=left&display=inline&height=536&linkTarget=_blank&name=image.png&originHeight=536&originWidth=1709&size=62467&status=done&width=1709)

增加了 10个 (string)，每个大小为 10000024个字节

## 解决内存泄露
按照上面的方法，加入如下代码（使用 let 定义）：

```
originLeakObject = null
```

查看堆栈：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550800973022-e90f935b-0c85-45fe-8a3b-391b40b3cca8.png#align=left&display=inline&height=671&linkTarget=_blank&name=image.png&originHeight=671&originWidth=1717&size=121510&status=done&width=1717)

对象引用树：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550801011458-b09b8cc9-d93c-403a-afa7-15d8ed1f0e58.png#align=left&display=inline&height=278&linkTarget=_blank&name=image.png&originHeight=278&originWidth=1718&size=52582&status=done&width=1718)

此时内存信息就属于正常了，只有 leakObject 中的 leakStr 占用了大半内存，没有形成闭包引用链。

## 资料
* [https://www.zhihu.com/question/56806069](https://www.zhihu.com/question/56806069)
* [https://developers.google.com/web/tools/chrome-devtools/memory-problems/memory-101](https://developers.google.com/web/tools/chrome-devtools/memory-problems/memory-101)
* 《NodeJS》 调试指南
