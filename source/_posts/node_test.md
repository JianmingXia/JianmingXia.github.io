---
title: Node 单元测试
date: 2019/02/10 18:30:00
tags:
  - NodeJS
  - Mocha
categories: NodeJS
---

## 为什么要有单元测试
先问自己几个问题：

- 代码质量是如何度量的？
- 如何保证代码的质量？
- 代码可以随时重构代码吗？
- 重构之后代码的质量如何保障？
- 有足够信心在没有测试的情况下随时发布代码码？

在职业生涯之初，面对上面的问题我也回答不了。<br />当偶尔线上发现紧急 bug 时，定位到问题 & 改完 bug 后——急速发了一个版本，结果带来了更多问题。
<!-- more -->

等到后面，我了解到了单元测试，有了单元测试就可以回答上面的问题：

- 代码质量可度量
- 代码质量有保障
- 重构后代码质量保障
- 敢于随时发布

下面介绍如何在 Node 中进行单元测试。

## 单元测试框架
> [https://mochajs.org/](https://mochajs.org/)


首先是单元测试框架，在单元测试框架这方面，我一直用的是 Mocha：

```
Mocha is a feature-rich JavaScript test framework running on Node.js and in the browser, making asynchronous testing simple and fun. Mocha tests run serially, allowing for flexible and accurate reporting, while mapping uncaught exceptions to the correct test cases. 
```

在 Mocha 中，有 describe 和 it 两个基础部分。

- describe 块称为**测试套件**，表示一组相关的测试
- it 块称为**测试用例**，表示一个单独的测试，是测试的最小单位

除此之外，还提供了如 before 及 after 等钩子，会在指定的时间执行。

当然，除了 Mocha 外，还有 Jest 和 Ava 等，感兴趣的也可以去试试。

## 断言库
断言库这方面，使用过 should、expect 和 power-assert，在这里贴一下这三个断言库测试的demo：

```
  // 这里只贴一下部分代码
  const arr = [1, 2, 3];
  
  it('should', async () => {
    should.equal(arr[0], 0);
  });

  it('expect', async () => {
    expect(arr[0]).eq(0);
  });

  it('assert', async () => {
    assert(arr[0] === 0);
  });
```

测试结果：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1558186250426-96c55f9d-bf93-493f-a390-c7fa5fe1302c.png#align=left&display=inline&height=674&name=image.png&originHeight=1348&originWidth=1166&size=127527&status=done&width=583)

从提示来看，power-assert 无疑是最优秀的，报错信息比较丰富。除此之外，power-assert 还支持不使用 API 的方式，写起来足够简单。<br />当然，如果大家对上述这两个优点不在乎的话，我相信 should 和 expect 也都是支持使用的。

## 测试覆盖率
> [https://istanbul.js.org/](https://istanbul.js.org/)


上面说到如何度量代码的质量，这个重任就交给测试覆盖率了——如果测试覆盖率足够高，这就表示每行代码都经历过测试，代码的质量相对来说也会更高一点。<br />关于测试覆盖率，它是有 4 个测试维度的：

- 语句覆盖率
- 分支覆盖率
- 函数覆盖率
- 行覆盖率

下面这是使用 Nyc 后的截图，

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1558186737441-f6ec117c-3a0d-41b8-af29-e64190bc3468.png#align=left&display=inline&height=294&name=image.png&originHeight=588&originWidth=1048&size=79195&status=done&width=524)

## 网络请求处理
在单元测试中，我们总是需要对服务提供的 API 进行测试。而此时我们就需要一个 [supertest](https://github.com/visionmedia/supertest)，它是一个HTTP 服务器测试模块，让 HTTP 断言变得非常简单。<br />同时，supertest 也跟 express 和 koa 结合也很简单。这里可以看看例子：

```
const request = require('supertest');
const express = require('express');

const app = express();

app.get('/user', function(req, res) {
  res.status(200).json({ name: 'john' });
});

request(app)
  .get('/user')
  .expect('Content-Type', /json/)
  .expect('Content-Length', '15')
  .expect(200)
  .end(function(err, res) {
    if (err) throw err;
  });
```

## Mock 服务
上面说到，我们需要对服务提供的 API 进行测试，但是每个服务都难免会有服务依赖。但是从微服务的角度来说，每个服务都是单独治理的。有时候，我们依赖的服务是其它团队维护，我们没有办法部署这个服务，这个时候我们就需要 Mock 了。<br />在 Mock 服务这，我使用了 [Nock](https://github.com/nock/nock)：

```
const nock = require('nock')

const scope = nock('https://api.github.com')
  .get('/repos/atom/atom/license')
  .reply(200, {
    license: {
      key: 'mit',
      name: 'MIT License',
      spdx_id: 'MIT',
      url: 'https://api.github.com/licenses/mit',
      node_id: 'MDc6TGljZW5zZTEz',
    },
  }
```

上面的例子表示：拦截请求到域名：[https://api.github.com](https://api.github.com) 及 path：/repos/atom/atom/license 的请求。

## 测试结构
在我之前的习惯中，测试是分为四段的：
- 前置准备：就是单元测试执行时所需的依赖，比如要调用第三方服务的 API 时，提前准备 Mock 数据
- 执行：执行即为此次测试的目标
- 断言：断言即为我们的预期
- 清理现场：测试结束后，需要将前置准备 或是 执行时产生的副作用清理掉

## 什么样的测试是好的测试
- Automatic（自动化）：测试能直接被机器执行，无需人工判断；
- Thorough（全面）：需要尽可能覆盖各种场景——这也是两个方面，一是写代码时要考虑各种场景；另一个则是测试需要覆盖更多的分支；
- Repeatable（可重复）：测试用例可以重复执行
- Independent（独立）：测试和测试之间不应该有依赖——每个测试用例都可以单独执行
- Professional（专业）：写测试代码也要按代码的标准去维护，不要有双标

这就是好的测试 A-Trip