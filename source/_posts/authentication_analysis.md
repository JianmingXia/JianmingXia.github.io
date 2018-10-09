---
title: 认证分析
date: 2018/03/01 20:00:00
tags:
  - 身份认证
categories: 技术杂集
---

## 说在前面
当我还是一个菜鸟的时候（当然现在也是），参与的都是内部项目，复杂度不高，再加上团队中可能没有有经验的同事。在这种传统的单体架构项目中，用户数据与当前服务数据存储在同一个数据库，服务端接收用户请求时，使用cookie进行校验；校验成功后才会路由到指定请求。。。
然后，当我们选择把服务商业化之后，我们加上了专门的身份验证服务器。相应的，服务的复杂度变高了，不仅是PCWeb端、还要支持移动Web端以及APP端等等。为了对用户更友好，我们支持多端登录。于是要根据不同的客户端类型生成对应的session_key。当我们有了不同的业务时，不同业务之间的登录态同步。。。
总之，是时候对原有的认证方案进行升级了

<!-- more -->

## 常见认证方案分析
### 单点登录（SSO）
> 之前采用的正是这个方案，每个服务都与SessionServer交互

每个面向用户的服务都必须与认证服务交互，会产生大量非常琐碎的网络流量和重复工作

### 分布式Session
> 可以考虑存储在Redis中

将用户认证的信息存储在共享存储中，通常会使用用户会话作为key来实现的简单分布式哈希映射。当用户访问微服务时，从共享存储中获取数据（共享存储需要一定的保护机制）

### 客户端Token
令牌在客户端生成，由身份验证服务进行签名，并包含足够的用户信息，以便在所有微服务中建立用户身份。
令牌会附加在每个请求上，为微服务提供用户身份验证，这个解决方案安全性不错，但身份注销是个问题——使用短期令牌和频繁检查认证服务

### 客户端Token与API Gateway结合
所有请求先进入API Gateway，从而隐藏其它服务。

## 常见认证方案
### 基于Session认证
用户登录完成后，将相关数据存储到Session中；如果是单体架构，默认会将Session存储在应用服务器，客户端一般存储在Cookie中；但是如果是分布式架构，存放在具体的应用服务器就无法满足使用了 。
简单的可以通过Session复制或者Session粘滞来解决：
* 复制：大量的Session数据复制会占用较多网络资源
* 粘滞：通过负载均衡，将用户分发到固定的服务器节点——只能满足水平扩展的集群场景，无法满足应用分割后的分布式场景

在微服务架构下，每个微服务拆分的粒度会很细，并且还会有微服务之间的调用，此时将Session从应用服务器剥离，在外部集中管理——数据库、分布式缓存：Memcached或Redis等

### 基于Token认证
与基于Session不同，Token一般会包含用户的相关信息，通过验证Token可以完成身份校验（Session相关信息会被服务端存储，会有SessionID与Session的对应关系）

#### 认证流程
* 用户输入登录信息，发送到身份认证服务
* 身份认证服务进行验证，返回验证结果（验证通过会包含用户基础信息、有效时间等）
* 用户将Token附加在请求头中，发起其它API调用
* 服务端验证Token权限，返回请求结果

#### 基于Token认证的优点
* 服务端无状态（与Cookie）
* 性能较好：Token验证不需要访问数据库或是其它远程服务
* 支持移动设备
* 支持跨程序调用：Cookie不支持跨域

两种基于Token的认证方案 JWT & OAuth2.0

### JWT
> JSON Web Token是一个基于JSON的开放标准，用于创建声明一些声明的访问令牌。例如，服务器可以生成具有声明“以管理员身份登录”并将其提供给客户端的令牌。然后，客户端可以使用该令牌来证明它以管理员身份登录。令牌由一方的私钥签名，以便双方能够验证令牌是否合法。

#### 认证流程
* 用户输入登录信息，发送到身份认证服务
* 身份认证服务进行验证，返回结果（创建JWT）
* 客户端在后续请求上附加JWT
* 服务端校验JWT，返回请求结果

#### JWT结构

```plain
xxxxx.yyyyy.zzzzz
```

##### Header(头部)
描述关于该JWT的最基本的信息，如类型、签名使用的算法等

```json
{
"typ": "JWT",
"alg": "HS256"
}
```

##### Payload（载荷）
载荷用于存储有效信息：
* 标准中注册的声明
* 公共的声明
* 私有的声明

标准中注册的声明（建议但非强制）：
* iss：JWT 签发者
* sub：JWT 所面向的用户
* aud：接收 JWT 的一方
* exp：JWT 的过期时间，这个过期时间必须要大于签发时间
* nbf：定义在什么时间之前，该 JWT 都是不可用的
* iat：JWT 的签发时间
* jti：JWT 的唯一身份标识，主要用来作为一次性 token, 从而回避重放攻击。

公共的声明 ：公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息
私有的声明 ：私有声明是提供者和消费者所共同定义的声明

声明中不要存储私密信息，Base64编码解码大家都懂的
> Do note that for signed tokens this information, though protected against tampering, is readable by anyone. Do not put secret information in the payload or header elements of a JWT unless it is encrypted.

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

##### Signature
将Base64编码后的header及payload使用 `.` 连接，通过header中声明的加密方式结合 __secret__ 进行加密，成为JWT的第三部分：

```cpp
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

#### JWT的优点
* 跨语言，使用JSON格式
* 基于Token，无状态
* 占用字节小，方便传输

#### 关于Token注销
由于Token存储在客户端，当用户注销时，Token依然有效，有几种方式来处理：
* 存于cookie，当客户端注销时，自然注销
* 注销时，将Token存于分布式缓存时，每次校验时去检查Token是否已被注销——额外校验
* 采用短期Token，一定程度上降低注销后的Token可用性风险

### OAuth2.0
> OAuth2.0是OAuth协议的延续版本，但不向后兼容OAuth 1.0即完全废止了OAuth1.0。 
> OAuth 2.0关注客户端开发者的简易性。要么通过组织在资源拥有者和HTTP服务商之间的被批准的交互动作代表用户，要么允许第三方应用代表用户获得访问的权限。
> 同时为Web应用，桌面应用和手机，和起居室设备提供专门的认证流程。2012年10月，OAuth 2.0协议正式发布为RFC 6749

#### OAuth2.0特点
* 简单：易于理解与使用
* 安全：没有涉及到用户秘钥等信息，更安全、更灵活
* 开放：任何服务提供商都可以实现OAuth，任何软件提供商都可以使用

#### 名词说明
* __Resource Owner__：资源所有者，即用户
* __Client：__客户端
* __Authorization server__：认证服务器
* __Resource server__：资源服务器


![image.png | left | 766x411](https://cdn.nlark.com/yuque/0/2018/png/92822/1537167506890-38a8f167-f61d-4a1e-b11d-69bd44b95374.png "")


* 用户打开客户端以后，客户端要求用户给予授权
* 用户同意给予客户端授权
* 客户端使用上一步获得的授权，向认证服务器申请令牌
* 认证服务器对客户端进行认证以后，确认无误，同意发放令牌
* 客户端使用令牌，向资源服务器申请获取资源
* 资源服务器确认令牌无误，同意向客户端开放资源

#### OAuth 2.0授权方式
> [http://www.ruanyifeng.com/blog/2014/05/oauth\_2\_0.html](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)

* 授权码模式（authorization code）
* 简化模式（implicit）
* 密码模式（resource owner password credentials）
* 客户端模式（client credentials）

## 资料
* [http://www.infoq.com/cn/articles/identity-authentication-of-architecture-in-micro-service](http://www.infoq.com/cn/articles/identity-authentication-of-architecture-in-micro-service)
* [https://jwt.io/introduction/](https://jwt.io/introduction/)
* [https://oauth.net/2/](https://oauth.net/2/)
* [http://www.ruanyifeng.com/blog/2014/05/oauth\_2\_0.html](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)

