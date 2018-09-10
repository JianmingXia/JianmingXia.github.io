---
title: 你不知道的JavaScript——this和对象原型
date: 2017/10/09 12:00:00
tags:
  - JavaScript
categories: 阅读笔记
---

## 关于this

### 为什么要使用this

this提供了一种更优雅的方式来隐式“传递”一个对象引用——这样我们可以将API设计得更加简洁并且易于复用

### 误解

#### 指向自身

并非指向自身：
<!-- more -->

```js
function foo(num) {
  console.log(`foo:${num}`);
  this.count++;
}

foo.count = 0;

for (let i = 0; i < 10; i++) {
  if (i > 5) {
    foo(i);
  }
}

console.log(foo.count);
// foo: 6
// foo: 7
// foo: 8
// foo: 9
// 0
```

此时，可以选择返回舒适区，用自己熟悉的方式去解决问题；还有一种，了解当前的实现细节，克服它。

#### 它的作用域

- 试图使用**this.bar()**来引用**bar()**函数
- 试图通过this联通**foo()**和**bar()**的词法作用域，来访问变量**a**

```js
"use strict";
function foo() {
  var a = 2;
  this.bar();
}

function bar() {
  console.log(this.a);
}

foo();
// Uncaught TypeError: Cannot read property 'bar' of undefined
```

### this 到底是什么

this是在运行时进行绑定的，并不是在编写时绑定，它的上下文取决于函数调用时的各种条件。

## this全面解析

### 调用位置

分析调用栈，找到调用位置。

### 绑定规则

#### 默认绑定

独立函数调用，当无法应用其它规则时，此为默认规则。

**foo()**是直接使用不带任何修饰的函数引用进行调用的：

```js
function foo() {
  var a = 1;
  console.log(this.a);
}

var a = 2;

foo();
// 2
```

在严格模式下，全局对象将无法使用默认绑定，因此this会绑定到undefined：

```js
"use strict";

function foo() {
  var a = 1;
  console.log(this.a);
}

var a = 2;

foo();
// Uncaught TypeError: Cannot read property 'a' of undefined
```

#### 隐式绑定

当函数引用有上下文对象时，**隐式绑定**规则会把函数调用时的this绑定到这个上下文对象。

```js
function foo() {
  console.log(this.a);
}

var obj = {
  a: 2,
  foo: foo
};

var a = 1;
obj.foo();
// 2
```

对象属性引用链中只有最后一层会影响调用位置：

```js
function foo() {
  console.log(this.a);
}

var obj2 = {
  a: 33333,
  foo: foo
};

var obj1 = {
  a: 2,
  obj2: obj2
};

var a = 1;
obj1.obj2.foo();
// 33333
```

隐式丢失：一个最常见的this绑定问题就是被**隐式绑定**的函数会丢失绑定对象，会应用默认绑定，最后将this绑定到全局对象或是undefined上则取决于是否是严格模式——比如上面的**obj.foo**及**obj1.obj2.foo**作为参数被传入其它函数。

#### 显式绑定

```js
function foo() {
  console.log(this.a);
}

var obj = {
  a: 2
};

foo.call(obj);
```

- 硬绑定：创建一个包裹函数，传入所有的参数并返回接收到的所有值。在ES5中提供了内置的方法**Function.prototype.bind**

- API调用的“上下文”

  ```js
  function foo(el) {
    console.log(el, this.id);
  }
  
  var obj = {
    id: "awesome"
  };
  
  [1, 2, 3].forEach(foo, obj);
  // 1 "awesome" 2 "awesome" 3 "awesome"
  ```

#### new绑定

构造函数：在JS中，构造函数只是一些使用new操作符时被调用的函数——并不会属于某个类，也不会实例化一个类

使用new来调用函数时，会自动执行下面的操作：

- 创建一个全新的对象
- 这个新对象会被执行[[原型]]连接
- 这个新对象会绑定到函数调用的this
- 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象

```js
function foo(a) {
  this.a = a;
}

var bar = new foo(2);
console.log(bar.a);
// 2
```

### 优先级

- 函数是否在new中调用？
- 函数是否通过call、apply或硬绑定调用？
- 函数是否在某个上下文对象中调用？
- 如果都不是，使用默认绑定。

### 绑定例外

凡是都有例外，以上的绑定优先级判断，在某些场景下绑定行为会出乎意料

#### 被忽略的this

如果你把**null**或**undefined**作为this的绑定对象传入call、apply或bind，这些值在调用时会被忽略，实际运用的是默认绑定规则：

```js
function foo(a) {
  console.log(this.a);
}

var a = 2;
foo.call(null);
// 2
```

可能在某些情况下，不在意this的指向。但是如果某个函数确实使用了this，会导致不可预计的后果。

更安全的this：创建一个DMZ（demilitarized zone, 非军事区）对象——它是一个空的非委托的对象

#### 间接引用

间接引用经常发生在赋值：

```js
function foo(a) {
  console.log(this.a);
}

var a = 2;
var o = {
    a: 3,
    foo: foo
};
var p = {
    a: 4
};

o.foo();
(p.foo = o.foo)();
// 3
// 2
```

#### 软绑定

使用硬绑定之后，无法使用隐式绑定或显式绑定来修改this，此时可以考虑软绑定——可以给默认绑定指定一个全局对象和**undefined**以外的值，同时保留隐式绑定或显示绑定修改this的能力

### this词法

箭头函数不使用this的四种标准规则，而是根据外层作用域来决定this。

如果我们经常编写this风格的代码啊，但是绝大部分时候都会使用self = this或者箭头函数来否定this机制，或许应当：

- 只使用词法作用域并完全抛弃错误this风格的代码
- 完全采用this风格，在必要时使用bind()，尽量避免self = this和箭头函数

>  作者觉得这两种风格混杂在一起会更难维护，个人觉得self使得代码很丑陋，箭头函数还是可以使用的——前提是能掌握this的指向

## 对象

### 语法

两种构造方式：

- 声明形式
- 构造形式

### 类型

JS中的主要类型：

- string
- number
- boolean
- null
- undefined
- symbol
- object

其中：

- 简单基本类型本身并不是对象
- null有时会被当做一个对象（其实是个bug）
- 错误观点：JS中万物皆是对象

内置对象：

- String
- Number
- Boolean
- Object
- Function
- Array
- Date
- RegExp
- Error

### 内容

#### 可计算属性名

```js
var prefix = "foo";

var myObject = {
  [prefix + "bar"]: "hello",
  [prefix + "baz"]: "world"
};

myObject["foobar"];
myObject["foobaz"];
```

#### 属性和方法

> 作者仿佛一直在论述**属性**与**方法**：从技术的角度来说，函数永远不会”属于“一个对象

#### 数组

数组期望的是数值下标

在数组中，直接添加命名属性并不能导致数组的length发生变化

#### 复制对象

浅拷贝 & 深拷贝

#### 属性描述符

可以使用Object.defineProperty()来添加一个新属性或修改一个属性并对特性进行设置

```js
var object = {
  a: 2
};

Object.getOwnPropertyDescriptor(object, "a");
// {value: 2, writable: true, enumerable: true, configurable: true}
```

- value
- writable：决定是否可以修改属性的值
- enumable：可枚举性
- configurable：只要属性是可配置的，就可以使用**defineProperty()**方法来修改属性描述符

configurable相关操作：

- configurable修改成false是单向操作：

- 即使configurable值是false，writable属性可由true改为false，但是无法由false改为true
- configurable:false 会禁止删除这个属性

#### 不变性

JS程序中很少需要深不可变性，有些特殊情况可能需要这么做。但是根据通用的设计模式，如果发现需要密封或冻结所有的对象，可能需要重新思考一下，让它能更好地应对对象值的改变

- 对象常量：writable:false和configurable:false创建一个常量属性
- 禁止扩展：使用Object.preventExtensions——无法创建新属性
- 密封：Object.seal会创建一个“密封”的对象——实际上在现有对象上调用Object.preventExtensions并将所有属性标记为configurable:false——无法创建新属性 && 不能重新配置或删除现有属性
- 冻结：Object.freeze会创建一个冻结对象——在调用Object.seal的基础上并将所有属性标记为writable:false，达成最高级别的不可变性

#### [[Get]]

在语言规范中，object.a实际上是实现了[[Get]]操作——对象默认的内置[[Get]]操作首先在对象中查找是否有名称相同的属性，如果找到就会返回这个属性的值；如果没有找到，就会按照[[Get]]算法的定义执行另外一个重要的行为——遍历可能存在的[[Prototype]]链

#### [[Put]]

如果属性已存在：

- 属性是否是访问描述符？如果是并且存在setter就调用setter
- 属性的数据描述符中writable是否是false？如果是，在非严格模式下静默失败；在严格模式下抛出异常
- 如果都不是，将该值设置为属性的值

#### Getter和Setter

对象默认的[[Put]]和[[Get]]操作分别可以控制属性值的设置和获取

#### 存在性

```js
var object = {
  a: 2
};

console.log("a" in object);
console.log("b" in object);

console.log(object.hasOwnProperty("a"));
console.log(object.hasOwnProperty("b"));
// true false true false
```

in与hasOwnProperty的区别：

- in操作符会检查属性是否在对象及其原型链中
- hasOwnProperty只会检查属性

### 遍历

#### for...in

用来遍历对象的可枚举属性列表

## 混合对象“类”

实例化、继承、多态

### 类理论

类/继承描述了一种代码的组织结构方式——一种在软件中对真实世界中问题领域的建模方法。

#### “类”设计模式

- 面向对象设计模式
- 过程化编程
- 函数式编程

####  JS中的“类”

最初，JS中只有一些近似类的语法元素：new  和 instanceof，ES6中新增如class——是否意味着JS中实际上有类呢？

其他语言中的类与JS中的“类”并不一样！！！

### 类的机制

#### 建造

“类”和“实例”的概念来源于房屋建造。

#### 构造函数

类构造函数属于类，而且通常与类同名。

构造函数通常需要使用new调用，这样语言引擎才知道我们想要构造一个新的类实例。

### 类的继承

#### 多态

在继承链中不同层次中一个方法名可以被多次定义，当调用方法时会自动选择合适的定义。

#### 多重继承

钻石问题：子类D继承自两个父类（BC），两个父类继承自A。A中有drive方法，并且BC都重写了方法，当D引用drive时会选择哪个呢？

在JS中，本身并不提供“多重继承”功能。

### 混入

JS中只有对象，并不存在可以被实例化的“类”——一个对象并不会复制到其他对象，它们会被关联起来

#### 显式混入

- 再说多态：在ES6之前，JS中并没有相对多态的机制
- 混合复制：在原mixin的基础上，再次进行拓展
- 寄生继承

#### 隐式混入

通过使用this，来实现隐式混入

### 总结

传统的类被实例化时，它的行为会被复制到实例中。类被继承时，行为也会被复制到子类中。

## 原型

上文所有模拟类复制行为的方法，都没有使用[[Prototype]]链

### [[Prototype]]

JS中的对象有一个特殊的[[Prototype]]内置属性，其实就是对于其他对象的引用。

#### Object.prototype

尽头：所有普通的[[Prototype]]链最终都会指向内置的Object.prototype

#### 属性设置和屏蔽

在原型链存在的情况下，设置属性可能会发生屏蔽现象。

### “类”

JS与面向类的语言不同，它并没有类作为对象的抽象模式。

#### “类”函数

JS中模仿类的操作

#### “构造函数”

```js
function Foo() {}

var a = new Foo();
```

```js
function Foo() {}

console.log(Foo.prototype.constructor === Foo);

var a = new Foo();
console.log(a.constructor === Foo);
// true true
```

因为看到了**new**，所以我们会认为**Foo**是一个类；以及另外通过**constructor**这个属性的判断。

> 实际上a本身并没有**.constructor**属性；虽然指向Foo函数，但是这个属性并非表示a由Foo“构造”

#### 技术

在使用new Foo时，看上去会把Foo.prototype对象复制到新对象中，但事实并非如此。

.constructor并不是一个不可变属性，它是不可枚举的，值是可修改的——所以.constructor是一个非常不可靠并且不安全的应用。

### （原型）继承

```js
function Foo(name) {
  this.name = name;
}

Foo.prototype.myName = function() {
  return this.name;
};

function Bar(name, label) {
  Foo.call(this, name);
  this.label = label;
}

Bar.prototype = Object.create(Foo.prototype);

Bar.prototype.myLabel = function() {
  return this.label;
};

var a = new Bar("a", "obj a");
console.log(a.myName());
console.log(a.myLabel());
// a
// obj a
```

以上代码核心是**Bar.prototype = Object.create(Foo.prototype);**——这会创建一个新对象并把新对象内部的**[[Prototype]]**关联到指定对象。

```
// 错误做法1
Bar.prototype = Foo.prototype;
// 错误做法2
Bar.prototype = new Foo();
```

- 做法1：当我们执行**Bar.prototype.myLabel =  ...**时会直接修改**Foo.prototype**
- 做法2：基本满足条件，但如果**Foo**有副作用（比如修改状态、注册其他对象等待），会产生副作用

### 对象关联

[[Protitype]]：如果对象上没有找到需要的属性或是方法引用，引擎会继续在[[Prototype]]关联的对象上进行查找——原型链

#### 创建关联

> Object.create(null)会创建一个拥有空[[Prototype]]链接的对象，这个对象无法进行委托，非常适合存储数据。

#### 关联关系是备用

看起来对象之间的关联关系是处理“缺失”属性或方法时的一种备用选项——但是不要完全依赖这种使用，否则对于其他的开发者来说，使用起来会很奇怪，可以遵循**委托设计模式**来解决这个问题。

## 行为委托

[[Prototype]]机制就是指对象中的一个内部链接引用另一个对象——JS这个机制的本质就是对象之间的关联关系

### 面向委托的设计

#### 类理论

类设计模式是如何使用类的

#### 委托理论

换一个思想：使用委托行为而不是类来思考同样的问题

但是在实现上，对象关联风格的代码会有一些不同之处。

#### 比较思维模型

对象关联风格的代码更加简洁（相较于原型继承的代码来说），因为这种代码只关注一件事：对象之前的关联关系。

### 类与对象

#### 控件“类”

丑陋的显示伪多态，各种方法使用**.call**去调用

#### 委托控件对象

对象关联可以更好地支持关注分离原则，创建和初始化并不需要合并为一个步骤。

### 更简洁的设计

对象关联除了能让代码看起来更简洁（并且更具扩展性）外，还可以通过行为委托模式简化代码结构。

### 更好的语法

- ES6中的class
- 函数名简写

但是需要注意，简写的本质是一个**匿名函数表达式**，而这个弊端也很明显

### 内省

instanceof语法会产生语义困惑且不直观——instanceof实际上表达的是两者是互相关联的，而不是我们想象中**是否是类的实例**。

“鸭子类型”——如果看起来像鸭子，叫起来像鸭子，那就一定是鸭子。事实上，这个模式会带来很多风险。

推荐使用：isPrototypeOf、getPrototypeOf方法

## 小结

### 关于JS中的类

类是一种可选（而不是必须）的设计模式，在JS这样的语言中实现类也是很别扭的

### class

ES6中的class虽然解决了一些问题，但是还有一些隐藏的问题是没有解决的：

- 静态属性
- 静态方法
- 遮蔽

## 其它

### 对象的[[Get]]、[[Put]]、Getter和Setter

### 原型链

- for...in：遍历对象时原理和查找[[Prototype]]链类似，任何可以通过原型链访问到（并且是enumerable）的属性都会被枚举
- in：查找对象的整条原型链，无论属性是否可被枚举

### Proxy