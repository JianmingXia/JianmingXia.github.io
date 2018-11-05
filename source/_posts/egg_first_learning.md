---
title: Egg 学习总结
date: 2018/10/26 19:00:00
tags:
  - Egg
categories: Egg
---

> [https://eggjs.org/en/](https://eggjs.org/en/)

![image.png | left | 525x728](https://cdn.nlark.com/yuque/0/2018/png/92822/1540284905297-5632fc2b-24c8-4aa7-90f7-647f22ad0fca.png "")


## 基础部分
### 设计原则
* Egg没有集成如数据库、模板引擎、前端框架等功能，而是专注提供Web开发的核心功能和灵活可扩展的插件机制
* 通过Egg，可以非常容易地基于Egg扩展出适合自身业务场景的框架
* Egg奉行__约定优于配置__
<!-- more -->

### 特性
* 提供基于Egg [定制上层框架](https://eggjs.org/zh-cn/advanced/framework.html)的能力
* 高度可扩展的[插件机制](https://eggjs.org/zh-cn/basics/plugin.html)
* 内置[多进程管理](https://eggjs.org/zh-cn/advanced/cluster-client.html)
* 基于 [Koa](http://koajs.com/) 开发，性能优异
* 框架稳定，测试覆盖率高
* [渐进式开发](https://eggjs.org/zh-cn/tutorials/progressive.html)

## 快速入门
### 环境依赖
* 操作系统：支持 macOS，Linux，Windows
* 运行环境：建议选择 [LTS 版本](http://nodejs.org/)，最低要求 8.x

### 快速初始化
推荐直接使用脚手架，使用简单命令即可初始化项目

#### 生成项目
通过在本地全局安装egg-init，由egg-init来生成项目：

```plain
$ npm i egg-init -g
$ egg-init egg-example --type=simple
$ cd egg-example
$ npm i
```

#### 启动项目

```plain
$ npm run dev
$ open localhost:7001
```

### 逐步搭建
> [https://eggjs.org/en/intro/quickstart.html#step-by-step](https://eggjs.org/en/intro/quickstart.html#step-by-step)

一般来说，Egg推荐直接使用脚手架，但依然可以通过手动创建的方式来了解Egg

#### 初始化项目
初始化目录结构，并安装egg及egg-bin：

```plain
$ mkdir egg-example
$ cd egg-example
$ npm init
$ npm i egg --save
$ npm i egg-bin --save-dev
```

在package.json中添加__npm scripts__：

```plain
{
  "name": "egg-example",
  "scripts": {
    "dev": "egg-bin dev"
  }
}
```

#### 创建Controller
##### app/controller/xx.js
在app/controller目录中创建controller

##### app/router.js
配置路由

##### 添加配置
在config/config.default.js中添加Cookie安全字符串

##### 目录结构
此时的目录结构：



![image.png | left | 322x168](https://cdn.nlark.com/yuque/0/2018/png/92822/1540275551275-d848c4ae-e54f-49c2-b4a1-17f657781d2a.png "")


#### 静态资源
> 我们目前前后端分离的方式应该是用不到的

Egg内置egg-static插件（线上环境建议部署至CDN），插件默认映射__/public/\* -> app/public/\*__，此时，将静态资源放到app/public即可：



![image.png | left | 244x138](https://cdn.nlark.com/yuque/0/2018/png/92822/1540275831857-a89f285e-01a0-4b30-9060-c6087be4c6c6.png "")


#### 模板渲染
Egg并不强制使用某种模板引擎，但约定了 [View 插件开发规范](https://eggjs.org/zh-cn/advanced/view-plugin.html)

#### 创建service
在实际运用中，Controller不会产出数据，也不会包含复杂的逻辑，复杂的过程应抽象为业务逻辑层——在Egg中，即[Service](https://eggjs.org/en/basics/service.html)（不要再像之前使用C++开发一样）

service代码在app/service目录

#### 添加扩展
> [https://eggjs.org/en/basics/extend.html](https://eggjs.org/en/basics/extend.html)

Egg提供了一个快速扩展的方式，只需在app/extend目录下提供扩展脚本即可

#### 添加Middleware
> [https://eggjs.org/en/basics/middleware.html](https://eggjs.org/en/basics/middleware.html)

比如某些请求禁止百度爬虫访问

#### 配置文件
* 支持按环境变量加载不同的配置文件
* 应用/插件/框架都可以配置自己的配置文件，框架将按顺序合并加载

#### 单元测试
> [https://eggjs.org/en/core/unittest.html](https://eggjs.org/en/core/unittest.html)

## 内置对象
> [https://eggjs.org/en/basics/objects.html](https://eggjs.org/en/basics/objects.html)

### Application
Application是全局对象，在一个应用中只会实例化一个，继承自Koa.Application，可以在Application上挂载全局的方法和对象

#### 事件
* server：一个worker进程只会触发一次，在HTTP服务完成启动后，会将HTTP server通过这个事件暴露给开发者
* error：运行时有任何的异常被onerror捕获后，都会触发error事件
* request 和 response：应用收到请求和响应请求时，分别触发这个两个事件

#### 获取方式
几乎所有被框架Loader加载的文件，都可以export一个函数——这个函数会被Loader调用，并使用app作为参数

* Controller：可以通过this.app访问
* Service：可以通过this.app访问
* Schedule
* 。。。

### Context
Context是一个__请求级别的对象__，继承自Koa.Context。
每次请求框架都会实例化一个Context对象，封装这次请求的信息。框架会将所有的Service挂载到Context上，部分插件也会将方法和对象挂载

#### 获取方式
* Middleware：在v2中是在middleware函数的参数中（v1中是函数中的this）
* Controller：this.ctx
* Service：this.ctx
* 非用户请求的场景下：通过Application.createAnonymousContext()创建匿名Context实例
* 定时任务：每个task都接收一个Context实例作为参数

### Request & Response
分别继承Koa.Request及Koa.Response

#### 获取方式
* 在Context实例上获取当前请求的Request及Response实例

另外，Koa会在Context实例上代理一部分Request和Response上的方法和属性

### Controller
框架提供了一个Controller基类，并推荐所有的Controller都继承该基类实现。Controller基类的属性：
* ctx：当前请求的Context实例
* app：应用的Application实例
* config：应用的配置
* service：应用所有的service
* logger：为当前controller封装的logger对象

#### 引用方式
* 从egg上获取（推荐）
* 从app实例上获取

### Service
框架提供了一个Service基类，并推荐所有Service都继承该基类实现。

Service基类的属性与Controller基类的属性一致，引用方式也一致

### Helper
用来提供一些实用函数——将常用的动作抽离在helper.js中成为一个独立的函数

Helper自身也是一个类，有和Controller基类一样的属性，也会在每次请求时进行实例化

#### 获取方式
* 在Context实例上获取当前请求的Helper实例

### Config
> config目录

可以自动合并应用、插件、框架的配置，按顺序覆盖，且可以根据环境维护不同的配置。
所有框架、插件和应用级别的配置都可以通过Config对象获取到

#### 获取方式
* 从Application实例上获取config对象
* Controller、Service、Helper：通过this.config获取config对象

#### 多环境配置



![image.png | left | 288x101](https://cdn.nlark.com/yuque/0/2018/png/92822/1540288652384-1a667927-8342-43d7-898a-1426e82c0f3e.png "")


config.default.js为默认的配置文件，所有环境都会加载这个配置文件，一般也会作为开发环境的默认配置文件。
当指定env时会同时加载对应的配置文件，并覆盖默认配置文件的同名配置

#### 配置加载顺序
应用 > 框架 > 插件



![image.png | left | 332x143](https://cdn.nlark.com/yuque/0/2018/png/92822/1540291384976-a67169b8-a2a9-4148-9949-948c3b978010.png "")


后加载的会覆盖前面的同名配置

#### 配置结果
> [https://eggjs.org/en/basics/config.html#configuration-result](https://eggjs.org/en/basics/config.html#configuration-result)

框架在启动时会把合并后的最终配置dump到run/application\_config.json（work进程）和run/agent\_config.json（agent进程）中

### Logger
4个级别的方法：
* logger.debug()
* logger.info()
* logger.warn()
* logger.error()

同时，框架中还有多个Logger对象：
* App Logger
* App CoreLogger
* Context Logger
* Context CoreLogger
* Controller Logger & Service Logger

### Subscription
订阅模型是比较常见的开发模式——比如消息中间件的消费者或调度任务

## 运行环境
> 一个Web应用本身应该是无状态的，并拥有根据运行环境设置自身的能力

### 指定运行环境
* 通过config/env文件指定，文件内容即为环境——一般使用构建工具来生成这个文件
* 通过EGG\_SERVER\_ENV环境变量指定

### 应用内获取运行环境
```plain
app.config.env
```

## Middleware
> [https://eggjs.org/en/basics/middleware.html](https://eggjs.org/en/basics/middleware.html)
> app/middleware目录

同Koa中的中间件形式一样，Egg的中间件也是洋葱圈模型

### 配置
中间件文件需要exports一个普通的function，接受两个参数：
* options：中间件的配置项，框架会将 app.config[\${middlewareName}] 传递进来
* app：当前Application实例

### 使用中间件
有以下方式支持中间件挂载：

#### 在应用中使用中间件
在config.default.js中加入配置：



![image.png | left | 481x182](https://cdn.nlark.com/yuque/0/2018/png/92822/1540292497398-5fa242eb-bfba-4924-bb9f-3556d2a6e0de.png "")


配置最终会被合并至app.config.appMiddleware

#### 在框架和插件中使用中间件



![image.png | left | 449x305](https://cdn.nlark.com/yuque/0/2018/png/92822/1540292574938-b7d0584b-9591-46a7-a0a7-81198122fc63.png "")


应用层定义的中间件（app.config.appMiddleware）和框架默认中间件（app.config.coreMiddleware）都会被加载器加载，并挂载到 app.middleware 上

#### route中使用中间件
以上两种配置的中间件是全局的，会处理每一次请求。可以在app/route.js中实例化和挂载，针对单个路由生效：

app/route.js：



![image.png | left | 508x91](https://cdn.nlark.com/yuque/0/2018/png/92822/1540292768853-514f5e2f-dc1f-4e37-9939-7ac3dafeb491.png "")


### 通用配置
* enable：控制中间件是否开启
* match：设置只有符合某些规则的请求才会经过这个中间件
* ignore：设置符合某些规则的请求不经过这个中间件

match和ignore支持多种类型的配置方式：
* 字符串
* 正则
* 函数

## Router
> [https://eggjs.org/en/basics/router.html](https://eggjs.org/en/basics/router.html)
> app/router.js

Router主要用来描述请求URL和具体承担执行动作的Controller的对应关系，框架约定了app/router.js用于统一所有路由规则

### Router详细定义说明

```plain
router.verb('path-match', app.controller.action);
router.verb('router-name', 'path-match', app.controller.action);
router.verb('path-match', middleware1, ..., middlewareN, app.controller.action);
router.verb('router-name', 'path-match', middleware1, ..., middlewareN, app.controller.action);
```

### RESTful风格的URL定义



![image.png | left | 590x371](https://cdn.nlark.com/yuque/0/2018/png/92822/1540293584136-e3d345ba-7ba7-4522-bb4c-2cbdf280ede7.png "")


### 太多路由拆分
可以将路由逻辑进行拆分，或者使用[egg-router-plus](https://github.com/eggjs/egg-router-plus)

## Plugin
> 不但可以保证框架核心的足够精简、稳定、高效，还可以促进业务逻辑的复用

### 为什么要插件
* 中间件加载其实是有先后顺序的，但是中间件自身无法管理
* 中间件的定位是拦截用户请求，并在其前后做一些事，比如：鉴权、安全检查等，但有些功能是和请求无关的：定时任务、消息订阅等
* 有些功能包含非常复杂的初始化逻辑，需要在应用启动的时候完成（不适合放在中间件中）

#### 中间件、插件、应用的关系
一个插件就是一个【迷你的应用】，和应用【app】几乎一样：
* 包含Service、中间件、配置、框架扩展等
* 没有独立的Router和Controller
* 没有plugin.js，只能声明跟其他插件的依赖，而不能决定其他插件是否开启



![image.png | left | 409x151](https://cdn.nlark.com/yuque/0/2018/png/92822/1540294137192-9ee509b1-ff21-4a12-8545-805f7ae41d51.png "")


## Extend
> [https://eggjs.org/en/basics/extend.html](https://eggjs.org/en/basics/extend.html)

* Application
* Context
* Request
* Response
* Helper

### 按照环境进行扩展
除基本的扩展之外，还可以根据环境进行扩展



![image.png | left | 361x121](https://cdn.nlark.com/yuque/0/2018/png/92822/1540295331278-ea439d23-4501-4fcf-84a2-27bac5d3c480.png "")


## 个人理解
从目录结构上看，Egg通过Controller、Router、Service、Middleware、Extend及Config即可满足基本开发，其中
* Router：负责配置路由规则
* Controller：即处理具体请求
* Service：编写业务逻辑层
* Middleware：提供中间件功能
* Extend：用于扩展框架
* Config：除基本配置外，还可以通过plugin.js来配置插件

而在具体业务开发中，接触更多的可能是框架中内置的基础对象，其中：
* Application：在一个应用中只会实例化一个
* Context：是一个请求级别的对象，每个请求都会实例化一个
* Request & Response：同样也是一个请求级别的对象，而且Context实例对象中也会代理一部分

