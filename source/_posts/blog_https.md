---
title: 博客启用Https
date: 2018/5/3 12:00:00
tags:
  - Https
categories: 博客相关
---

## 说明
当初使用Github Pages服务搭建博客完毕之后，因为之前自己买了个域名，故而也配置了自定义域名。配置之后，就发现左上角那个美丽的锁不见了，当时也查了很多办法，但是最后都不了了之。但是最近，在逛github时，突然看到一个新闻：![](https://img.ryoma.top/HttpsDomain/0.png)
[原文链接](https://blog.github.com/2018-05-01-github-pages-custom-domains-https/)
<!-- more -->

## 启用Https
按照github的说明，启用https很简单。但是现实很残酷，启用之后，我的博客依然没有美丽漂亮的锁。此时需要注意的是：[https://blog.ryoma.top](https://blog.ryoma.top)可以正常访问。这时我的反应是——我的博客中应该引用了非https的资源。
看了之前接入的畅言及leancloud，没有发现问题，但是发现我引用的图片都是http的，于是来解决这个问题。
我的图片资源皆存储在腾讯云cos中：
- 首先，给cos中的bucket配置自定义域名[对象存储中配置自定义域名支持 HTTPS 访问](https://cloud.tencent.com/developer/article/1008127)
- 将博客中的http链接全部改成https

如下：![](https://img.ryoma.top/HttpsDomain/1.png)

## 后记
很久没有更新博客了，今天写之前，给自己找了诸多理由、借口：
- 最近工作忙，正在研究H5引擎，在研究基于cocos2d-JS使用TS做游戏
- 最近还会刷题（刷题的目的是希望自己能够保持活跃的思维），在刷下一题之前，必须把之前一题的思路总结出来
- 在憋一个大招，并为此不断努力

一直坚信，理解一个事物，并能将之分享出来，才是上策。