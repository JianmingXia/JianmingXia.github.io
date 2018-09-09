---
title: 你不知道的JavaScript（中卷）——类型和语法
date: 2017/11/18 12:00:00
tags:
  - JavaScript
categories: 阅读笔记
---

## 类型

### 类型

全面掌握JS的类型之后，会改变我们对强制类型转换的成见，看到它的好处并意识到它的缺点被过分夸大了。

### 内置类型

内置类型，除object外，其它统称为“基本类型”

- null
- undefined
- boolean
- number
- string
- symbol
- object

可以使用**typeof**运算符来查看值的类型，返回类型的字符串值。

<!-- more -->

#### null

其中**null**类型比较特殊：

```js
typeof null === "object";	
```

> 这个问题由来已久，可能永远不会修复——历史包袱太重，“修复”会产生更多的bug

**null**是基本类型中唯一一个“假值”类型。

```js
typeof function a() {} === "function";
```

#### function

**function**是object的一个“子类型”——函数是“可调用对象”，有一个内部属性[[Call]]，使其可被调用。

```js
function a(b, c) {}

function b() {}

console.log(a.length);
console.log(b.length);
// 2 0
```

函数不仅是对象，还可以拥有属性；函数对象的length属性是其声明的参数个数。

#### 数组

```js
typeof [1, 2, 3] === "object";
```

数组也是对象，也是object的“子类型”。

### 值和类型

JS中的变量是没有类型的，只有值才有（这点倒是与Redis的数据结构很像），变量可以随时持有任何类型的值——JS引擎不要求变量总是持有与其初始值同类型的值。

**typeof**运算符总是会返回一个字符串：

```js
typeof typeof [1, 2, 3] === "string";
typeof typeof function a() {} === "string";
```

#### undefined 和 undeclared

变量在未持有值的时候为**undefined**的，此时**typeof**返回“undefined”：

```js
var a;
console.log(typeof a); // undefined

var b = 42;
var c;

b = c;
console.log(typeof b); // undefined
console.log(typeof c); // undefined
```

- 已在作用域中声明但还没有赋值的变量，是undefined的
- 未在作用域中声明的变量，是undeclared

```js
var a;
a; // undefined
b; // Uncaught ReferenceError: b is not defined
```

需要注意，“b is not defined”并不是"b is undefined"，另外：

```js
var a;
console.log(typeof a); // undefined
console.log(typeof b); // undefined
```

对于一个undeclared变量，typeof依然返回“undefined”——这是由于typeof有一个特殊的安全防范机制

#### typeof Undeclared

上述的安全防范机制对浏览器中运行的JS代码还是有帮助的，因为多个脚本文件会在共享的全局命名空间中加载变量。

比如我们需要在项目使用全局变量DEBUG作为“调试模式”的开关，顶层的全局变量声明**var DEBUG = true**只存在与**debug.js**文件，该文件只在开发与测试时才被加载到浏览器，生产环境不会加载：

```js
if (DEBUG) {
  console.log("Debuggin is starting");
}
// 报错

if (typeof DEBUG !== "undefined") {
  console.log("Debuggin is starting");
}
// 安全使用
```

除了typeof外，还可以通过**window**访问来实现——但是当JS不仅运行浏览器时，全局对象并非window

## 值

### 数组

与其它强类型语言不同，在JS中，数组可以容纳任何类型的值。

数组通过数字进行索引，但同时它们也是对象，可以包含字符串键值和属性（但不会计算在数组长度内，可强制转为十进制数组外）——建议使用对象存放键值对，用数据存储数字索引值

#### 类数组

有时需要将类数组转为真正的数组，可以通过工具函数（indexOf()、concat()、forEacth()等）来实现——如DOM查询操作会返回DOM元素列表、arguments对象等

### 字符串

字符串经常被当成字符数组，但是JS中的字符串和字符数组并不一致，最多只是看上去相似而已：

```js
var a = "foo";
var b = ["f", "o", "o"];

console.log(a.length); // 3
console.log(b.length); // 3

console.log(a.indexOf("o")); // 1
console.log(b.indexOf("o")); // 1

var c = a.concat("bar");
var d = b.concat(["b", "a", "r"]);
console.log(c); // foobar
console.log(d); // ["f", "o", "o", "b", "a", "r"]

console.log(a === c); // false
console.log(b === d); // false

console.log(a); // foo
console.log(b); // ["f", "o", "o"]
```

但这并不表示它们都是“字符数组”，如：

```js
a[1] = "O";
b[1] = "O";

console.log(a); // f00
console.log(b); // ["f", "O", "o"]
```

JS中字符串是不可变的，而数组是可变的；另外a[1]并非总是合法语法，比如老版本IE不支持，正确调用是**a.chatAt(1)**。

许多数组函数处理字符串很方便，虽然字符串没有这些函数，但可以通过“借用“数组的非变更方法来处理字符串：

```js
console.log(a.join); // undefined

var e = Array.prototype.join.call(a, "-");
console.log(e); // f-o-o
```

另外，字符反转也可以通过类似的方法解决（但是，如果字符串中有复杂字符，通过split、reverse、join的方法并不适用，如果经常处理字符串的话，推荐直接使用数组。

### 数字

JS中只有一种数值类型：number，包括”整数“和带小数的十进制数——JS中的整数就是没有小数的十进制数

#### 数字的语法

特别大和特别小的数字默认用指数格式显示

由于数字值可以使用Number对象进行封装，因此数字值可以调用Number.prototype中的方法，如**tofixed()**

#### 较小的数值

二进制浮点数最大的问题（不仅仅是JS，所有遵循IEEE 754规范的都是如此），是会出现以下情况：

```js
0.1 + 0.2 === 0.3; // false
```

最常见的办法是设置一个误差范围值，通常称为“机器精度”（machine epsilon），对JS来说，通常是2^-52

#### 整数的安全范围

- 能被“安全”呈现的最大整数是2^53 - 1

有时JS中需要处理较大的数组，如数据库中64位ID，此时需要使用字符串存储。

#### 整数检测

- 是否为整数：Number.isInteger()
- 是否是安全的整数：Number.isSafeInteger()

#### 32位有符号整数

虽然整数最大能达到53位，但是有些数字只适用于32位数字；使用 a | 0可以将变量a中的数值转换为32位有符号整数，因为运算符|只适用于32位整数。

### 特殊数值

JS数据类型中有几个特殊的值需要特别注意：

#### 不是值的值

- undefined：未赋值
- null：曾赋值过，目前没有值

**null**是一个特殊关键字，不是标识符，不能将其当做变量来使用和赋值；undefined是一个标识符，可以被当做变量来使用和赋值

#### undefined

在非严格模式下，可以为全局标识符undefined赋值——这也太可怕了——永远不要重新定义undefined

##### void 运算符

undefined是一个内置标识符，它的值为undefined，可以通过void运算符得到。

如果要将代码中的值设为undefined，可以使用void

#### 特殊的数字

##### 不是数字的数字

如果数学运算的操作数不是数字类型，就无法返回一个有效的数字，这种情况返回值为NaN。

NaN：不是一个数字（not a number），这个名字容易引起误会——理解为无效数值、失败数值

```js
var a = 2 / "foo";
console.log(a); // NaN

console.log(typeof a === "number"); // true
```

NaN是一个特殊值，它和自身不相等，是唯一一个**非自反**的值

##### 无穷数

比如在C中调用**1 / 0**会有编译错误或运行时错误，但在JS中，结果为**Infinity**，如果运算中某一个操作数为负数，则结果为**-Infinity**。

一旦结果被计算为无穷数，就无法再得到有穷树。

```js
Infinity / Infinity; // NaN
2 / Infinity; // 0
```

##### 零值

加法和减法不会得到负零，乘除会。

负零有什么意义呢？

#### 特殊等式

NaN与-0在作相等比较时的表现有点奇怪，在ES6中，添加了Object.js()来判断两个值是否绝对相等——能使用== 或 ===时尽量使用，因为效率更高

### 值和引用

- JS中没有指针，引用的工作机制也不太一样——在JS中变量不可能成为指向另一个变量的引用
- S引用指向的是值
- JS中值和引用的赋值/传递在语法上没有区别，由值的类型决定
- 简单值总是通过值复制的方式来赋值/传递
- 复合值——对象和函数，通过引用复制的方式来赋值/传递

## 原生函数

常见的原生函数有：

- String()
- Number()
- Boolean()
- Array()
- Object()
- Function()
- RegExp()
- Date()
- Error()
- Symbol()

实际上，它们也就是内建函数；原生函数也可以被当做构造函数来用：

```js
var a = new String("abc");
console.log(typeof a); // object

console.log(a instanceof String); // true

console.log(Object.prototype.toString.call(a)); // [object String]
```

通过构造函数创建出来的是封装了基本类型值的封装对象——new String()创建的是字符串的封装对象，而不是基本类型值

### 内部属性[[Class]]

所有typeof返回值为“object”的对象都包含一个内部属性[[Class]]，这个属性无法直接访问，可以通过Object.prototype.toString()查看

### 封装对象包装

封装对象（object wrapper）扮演着十分重要的角色，由于基本类型值没有**.length**和**.toString()**这样的属性和方法，需要通过封装对象才能访问，此时JS会自动为基本类型值包装一个封装对象。

Note：如果经常用到这些字符串属性和方法，比如for中的**a.length**，我们可能会觉得直接创建一个封装对象会更方便——但实际上浏览器已经为这样的常见情况做了性能优化，直接使用封装对象反而会降低执行效率

一般情况，我们不需要直接使用封装对象

#### 封装对象释疑

```js
var a = new Boolean(false);

if (!a) {
  console.log("no console");
}
```

如果想要自行封装基本类型值，推荐使用Object(..)（不带new关键字）

### 拆封

如果想获取封装对象中的基本类型值，可以使用valueOf()函数

### 原生函数作为构造函数

关于数组、对象、函数及正则表达式，我们通过喜欢使用常量的形式来创建——实际上，使用常量与使用构造函数的效率是一样的

#### Array(..)

构造函数Array(..)不要求必须带new关键字；

Array构造函数只带一个数字参数时，参数为数组的预设长度，而非充当数组的一个元素——这个情况比较诡异

map与join处理的差别：

- join假定数组不为空，使用length来遍历元素
- map并不做假定

Note：不要创建和使用空单元数组

#### Object(..)、Function(..)和RegExp(..)

- 尽量不要使用以上原生函数构造
- 建议使用常量形式定义正则表达式，不仅语法简单，执行效率也更高（JS引擎会进行预编译和缓存）
- 当正则内容是动态时，可以直接使用new RegExp()

#### Date(..)和Error(..)

相比与其它原生构造函数，Date(..)和Error(..)的用处会更大——因为没有对应的常量形式来替代

创建错误对象（error object）主要是为了获得当前运行栈的上下文——栈上下文信息包括函数调用栈信息和产生错误的代码行号，方便调试，一般与throw一起使用

#### Symbol(..)

Symbol是具有唯一性的特殊值，用它来命名对象属性不容易导致重名

- 但无论在代码还是开发控制台，无法查看和访问它的值
- 不能使用new去构造
- 经常会被用于私有或特殊属性，虽然并非私有属性
- Symbol并非对象，是一个简单的基本类型

#### 基本类型

原生构造函数有自己的.prototype对象——这些对象包含其对象子类型所特有的行为特征

但是，有些原生原型并非普通对象那么简单：

```js
console.log(typeof Function.prototype); // function
console.log(Function.prototype); // ƒ () { [native code] }

console.log(RegExp.prototype.toString()); // /(?:)/——空正则表达式
```

甚至，还可以修改它们

##### 将原型作为默认值

- Function.prototype 是一个空函数
- RegExp.prototype是一个空的正则表达式
- Array.prototype是一个空数组

对于未赋值的变量来说，它们是很好的默认值

## 强制类型转换

### 值类型转换

将值从一种类型转换为另一种类型通常称为“类型转换”，这是显式的情况；隐式的情况称为强制类型转换

- 类型转换发生在静态类型语言的编译阶段
- 强制类型转换则发生在动态类型语言的运行时

然而在JS中通常统称为强制类型转换，当然我们可以用**隐式强制类型转换**和**显示强制类型转换**来区分

### 抽象值操作

#### ToString

基本类型值的字符串化：

- null => "null"
- undefined => "undefined"
- true => "true"
- 数字的字符串化遵循通用规则，极大或极小的数使用指数形式

对象：

- 普通对象除非自行定义，默认返回内部属性[[Class]]的值

如果对象有自己的toString方法，字符串化时就会调用该方法

- 数组的默认toString()方法经过了重新定义，所有单元字符串化后使用“,”拼接

  ```js
  var a = [1, 2, 3];
  console.log(a.toString()); // 1,2,3
  ```

##### JSON字符串化

工具函数JSON.stringify()在将JSON对象序列化为字符串时也用到了ToString

- 字符串、数字、布尔值和null的JSON.stringify()规则与ToString基本相同
- 如果传递给JSON.stringify的对象中定义了toJSON方法，那么该方法会在字符串化前调用，以便将对象转换为安全的JSON值

Note：这块内容比较复杂

#### ToNumber

- true => 1
- false => 0
- undefined => NaN
- null => 0

对象（包括数组）会首先被转换为相应的基本类型值，如果返回的是非数字的基本类型值，则再遵循以上规则将其强转为数字。

为了将值转为相应的基本类型值，首先会检查师傅有valueOf()方法，如果有并且返回基本类型值，则使用该值进行强制类型转换；如果没有就使用toString()（如果存在）的返回值来进行类型转换；如果两者都不返回基本类型值，产生TypeError

#### ToBoolean

在JS中，true和false 与 1和0是不一样的。

##### 假值

在JS中的值可以分为两类：

- 可以被强制类型转换为false的值
- 其他

假值：

- undefined
- null
- false
- +0，-0，NaN
- “”

##### 假值对象

如之前提到的：

```js
var a = new Boolean(false);
```

##### 真值

真值就是假值列表之外的值

### 显示强制类型转换

显示强制类型转换是那些显而易见的类型转换，在编码时尽量将类型转换表达清楚，以免留坑。

#### 字符串和数字之间的显示转换

使用如：

- String()
- Number()
- +

##### 日期显示转换为数字

不建议对日期类型使用强制类型转换，推荐使用Date.now()获取当前时间戳

##### 奇特的~运算符

##### 字位截除

~~

#### 显示解析数字字符串

解析字符串中的数字和将字符串强制类型转换为数字的返回结果都是数字，但是解析和转换两者之间还是有明显的差别：

```js
var a = "42";
var b = "42px";

console.log(Number(a)); // 42
console.log(parseInt(a)); // 42

console.log(Number(b)); // NaN
console.log(parseInt(b)); // 42
```

解析：允许字符串中包含非数字字符；解析从左往右，如果遇到非数字字符就停止

转换：不允许出现非数字字符，否则会失败并范围NaN

#### 显示转换为布尔值

### 隐式强制类型转换

隐式强制类型转换指的是那些隐蔽的强制类型转换，副作用也不是很明显——显示强制类型转换旨在让代码更加清晰易读，而隐式强制类型转换则像是对立面，会让代码变得晦涩难懂。

隐式强制类型转换的作用是减少冗余，让代码更简洁。

#### 隐式地简化

作者表示：隐式强制类型转换同样可以用来提高代码可读性；虽然它会带来一些负面影响，有时甚至弊大于利，因此更需要开发者会去其糟粕，取其精华——不要因噎废食

#### 字符串和数字之间的隐式强制类型转换

```js
var a = "42";
var b = "0";

var c = 42;
var d = 0;

console.log(a + b); // "420"
console.log(c + d); // 42
```

通常我们的理解是当某个或者两个操作数是字符串时，执行的是拼接字符串操作，解释是对的，但是无法解释下面的情况：

```js
var a = [1, 2];
var b = [3, 4];

console.log(a + b); // 1,23,4
```

> 按照ES5规范11.6.1节，如果某个操作数是字符串或者能够通过以下步骤转换为字符串的，+将进行拼接操作。如果某个操作数是对象（包括数组），则首先对其调用ToPrimitive抽象操作，该抽象操作再调用[[DefaultValue]]，以数字作为上下文。在这里数组的valueOf()无法得到简单基本类型值，继续调用toString()

#### 布尔值到数字的隐式强制类型转换

比如需要判断传的参数中有多少个true时，可以派上用场了

#### 隐式强制类型转换为布尔值

- if(..)语句中的条件判断表达式
- for(..; ..; ..)语句中的条件判断表达式（第二个）
- while和 do..while中的条件判断表达式
- ?: 中的
- 逻辑运算符 || 和 &&左边的操作数

以上情况中，非布尔值会被隐式强制类型转换为布尔值

#### || 和 &&

> ES5规范11.11节：&& 和 || 运算符的返回值不一定是布尔类型，而是两个操作数其中一个的值

```js
var a = 42;
var b = "abc";
var c = null;

console.log(a || b); // 42
console.log(a && b); // "abc"

console.log(c || b); // "abc"
console.log(c && b); // null
```

在C和PHP中，运算符的结果应该是true或false

#### 符号的强制类型转换

- ES6允许从符号到字符串的显示强制类型转换，然后隐式强制类型转换会产生错误。
- 符号不能够被强制类型转换为数字（无论是显示还是隐式）
- 可以被强制类型转换为布尔值（显示和隐式都可以，true）

当然，鉴于符号的特殊用途，很少会用到它的强制类型转换

```js
var s1 = Symbol("cool");
console.log(String(s1)); // "Symbol(cool)"

var s2 = Symbol("not cool");
console.log(s2 + ""); // TypeError: Cannot convert a Symbol value to a string
```

### 宽松相等和严格相等

- 宽松相等——loose equals
- 严格相等——strict equals

常见的误区是：

- == 检查值相等
- === 检查值和类型是否相等

作者：虽然听起来很正确，但是还不够准确。

正确的解释：==允许在相等比较中进行强制类型转换，而===不允许

#### 相等比较操作的性能

不用在乎性能

#### 抽象相等

> ES5规范的11.9.3节的“抽象相等比较算法”定义了==运算符的行为

- 如果两个值的类型相等，仅比较它们是否相等

  比较特殊的情况：

  - NaN 不等于 NaN
  - +0 等于 -0

- 对象宽松相等：两个对象指向同一个值时视为相等，不发生强制类型转换（=== 也是一样）

- ==在比较两个不同类型的值时会发生隐式强制类型转换，会将其中一个或两个都转换为相同的类型再进行比较

##### 字符串和数字之间的比较

```js
var a = 42;
var b = "42";

console.log(a === b); // false
console.log(a == b); // true
```

- 如果Type(x)是数字，Type(y)是字符串，则返回 x == ToNumber(y)的结果
- 如果Type(x)是字符串，Type(y)是数字，则返回ToNumber(x) == y的结果

##### 其它类型和布尔类型之间的相等比较

== 最容易出错的是true和false与其他类型之间的比较：

```js
var a = "42";
var b = true;

console.log(a == b); // false
```

- 如果Type(x)是布尔类型，则返回ToNumber(x) == y的结果
- 如果Type(y)是布尔类型，则返回x == ToNumber(y)的结果

建议不要使用 == true 和 == false，=== true 和 === false不允许强制类型转换，关系不大

##### null 和 undefined之间的相等比较

null 和 undefined之间的==也涉及隐式强制类型转换：

- 如果x为null，y为undefined，返回true
- 如果x为undefined，y为null，返回true

```js
var a = null;
var b;

console.log(a == b); // true
console.log(a == null); // true
console.log(b == null); // true

console.log(a == false); // false
console.log(b == false); // false
console.log(a == ""); // false
console.log(b == ""); // false
console.log(a == 0); // false
console.log(b == 0); // false
```

##### 对象与非对象之间的相等比较

关于对象（对象、函数和数组）和标量基本类型之间的比较，规范如下：

- 如果x是字符串或数字，y是对象，则返回 x == Toprimitive(y)的结果
- 如果x是对象，y是字符串或数字，则返回 ToPrimitive(x) == y的结果

```js
var a = 42;
var b = [42];

console.log(a == b); // true
```

#### 比较少见的情况

##### 返回其他数字

```js
Number.prototype.valueOf = function() {
  return 3;
};

console.log(new Number(2) == 3); // true
```

```js
var i = 2;
Number.prototype.valueOf = function() {
  return i++;
};

var a = new Number(42);

if (a == 2 && a == 3) {
  console.log("this happened");
}
```

##### 假值的相等比较

![](https://img.ryoma.top/YouDontKnowJS/mid_1-1.png)

##### 极端情况

```js
[] == ![]; // true
```

##### 完整性检查

![](https://img.ryoma.top/YouDontKnowJS/mid_1-2.png)

- 避免使用 == false
- 避免 == []

##### 安全运用隐式强制类型转换

- 如果两边的值有true 或 false，千万不要使用==
- 如果两边有[]，“”或者0，尽量不要使用==

此时尽量使用===来避免不经意的强制类型转换

### 抽象关系比较

```js
var a = { b: 42 };
var b = { b: 43 };

console.log(a < b); // false
console.log(a == b); // false
console.log(a > b); // false

console.log(a <= b); // true
console.log(a >= b); // true
```

根据规范，a <= b被处理为b < a，然后再将结果反转

## 语法

### 语句和表达式

> statement and expression

“句子”（sentence）是完整表达某个意思的一组词，有一个或多个“短语”（phrase）组成，它们之间由标点符号或连接词（and 和 or等）连接起来。短语可以由更小的短语组成，有些短语是不完整的，不能独立表达意思；有些短语则相对完整，并且能够独立表达某个意思。

以上是英语中语法，JS中的语法也是如此——语句相当于句子，表达式相当于短语，运算符则相当于标点符号和连接词。

#### 语句的结果值

> 语句都有一个结果值（statement completion value, undefined也算）

我们可以在控制台上看到结果值；规范定义var的结果值是undefined。

#### 表达式的副作用

大部分的表达式都没有副作用

#### 上下文规则

##### 大括号

对象常量：使用大括号定义对象常量

标签：**foo: bar()**

##### 代码块

```js
[] + {}; // "[object Object]"
{} + []; // 0
```

不要使用console.log包裹去运行

##### 对象解构

##### else if和可选代码块

> 大家都认为JS中else if，其实不然——只是由于它能省略一层代码缩进，所以用得比较多

### 运算符优先级

- 用,来连接一系列语句时，它的优先级最低

[运算符优先级文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Operator_Precedence)

#### 短路

对&& 和 ||来说，如果从左边的操作数能够得到结果，就可以忽略右边的操作数——这种现象称为短路（即执行最短路径）

#### 更强的绑定

> 还是与运算符优先级有关

#### 关联

一般来说，运算符的关联不是从左往右，就是从右往左，这取决于组合是从左开始还是从右开始——关联和执行顺序不是一回事！！！

右关联：

- = 运算符
- 条件运算符

#### 释疑

如果运算符优先级/关联规则能够令代码更为简洁，就使用运算符优先级/关联规则；而如果（）有助于提高代码可读性，就使用（）

### 自动分号

有时JS会自动为代码补上缺失的分号，即自动分号插入（Automatic Semicolon Insertion, ASI）

#### 纠错机制

推荐在所有需要的地方加上分号，将对ASI的依赖降到最低

### 错误

JS中有各种类型的运行时错误：

- TypeError
- ReferenceError
- SyntaxError

当然，也定义了一些编译时错误——在编译阶段发现的叫做“早期错误”，语法错误是早期错误的一种（如 a = ,）；另外，语法正确但不符合语法规则的情况也存在。

这些错误在代码执行之前是无法用try..catch捕获的，相反，它们会导致解析/编译失败

#### 提前使用变量

在ES6规范中，有个新概念：TDZ——Temporal Dead Zone，暂时性死区

TDZ 指的是由于代码中的变量还没有初始化而不能被引用的情况：

```js
{
  a = 2;
  let a;
}
```

a = 2试图在let a初始化a之前使用变量，这里就是a的TDZ，会产生错误

另外，对未声明变量使用typeof不会产生错误，但是在TDZ中会报错：

```js
{
  typeof a;
  typeof b;
  let b;
}
```

### 函数参数

另一个TDZ的例子是ES6中的参数默认值：

```js
var b = 3;

function foo(a = 42, b = a + b + 5) {}
foo();
```

**b = a + b + 5**在参数b的TDZ中访问b

在ES6中，如果参数被省略或者值为undefined，则取该参数的默认值

向函数传递参数时，arguments数组中的对应单元会和命名参数建立关联以得到相同的值；不传递参数则不就建立关联——在严格模式中，并没有建立关联这一说，因此在开发中也不要依赖这种关联机制——事实上，它是JS语言引擎底层实现的一个抽象泄露

不要同时访问命名参数和其对应的arguments数组单元

### try..finally

finally中的代码总是会在try之后执行，如果有catch的话则在catch之后执行。

try中的返回值与finally的关系（如果要在开发中使用，值得一看）

### switch

## 附录A 混合环境JS

如果JS仅仅在引擎中运行的话，它会严格遵循规范并且是可预测的。但JS总是在宿主环境中运行，使得它在一定程度上不可预测。

### Annex B(ECMAScript)

JS的官方名称是ECMAScript，那JS又是指什么呢？——是该语言的通用称谓，更确切的说是该规范在浏览器上的实现

官方ECMAScript规范包括Annex B，其中介绍了由于浏览器兼容性问题导致的与官方规范的差异：

- 在非严格模式运行八进制数值常量存在
-  window.escape(..)和window.unescape(..)能够转移和回转带有%分隔符的十六进制字符串
- String.prototype.substr和substring十分相似，前者第二个参数是结束位置索引（非自包含），后者的第二个参数是长度

### 宿主对象

宿主对象的行为差异有：

- 无法访问正常的object内建方法，如toString()
- 无法写覆盖
- 包含一些预定义的只读属性
- 包含无法将this重载为其他对象的方法
- 其他。。。

在针对运行环境进行编码时，宿主对象扮演着一个十分关键的角色，要特别注意其行为特性。

比如console是由宿主对象提供，以便从代码中输出各种值

### 全局DOM变量

由于浏览器演进的历史遗留问题，在创建带有id属性的DOM元素也会创建同名的全局变量

### 原生原型

JS的最佳实践：不要扩展原生原型

- 首先，不要扩展原生方法，除非确认代码在运行环境中不会有冲突
- 其次，在扩展原生方法时需要加入判断条件

### <script\>

> 标签中的文件和内联代码是相互独立的，还是一个整体呢？

### 保留字

- 关键字
- 预留关键字
- null常量
- true false布尔常量

### 实现中的限制

- 字符串常量中允许的最大字符数
- 可以作为参数传递到函数中的数据大小
- 函数声明中的参数个数
- 未经优化的调用栈（例如递归）的最大层数，即函数调用链的最大长度
- JS以阻塞方式在浏览器中运行的最长时间
- 变量名的最大长度

## 其它

### Array.slice()

返回一个从开始到结束（**不包括结束**）选择的数组的一部分**浅拷贝**到一个新数组对象，且原始数组不会被修改

###  Array.from()

从一个类似数组或可迭代对象中创建一个新的数组实例

### 稀疏数组

将包含至少一个”空单元“的数组称为稀疏数组