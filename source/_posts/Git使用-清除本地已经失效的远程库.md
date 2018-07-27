---
title: 清理本地已经失效的远程库——Git
tags:
  - git
  - 远程库
categories: Git
---

## 现状
平时使用SourceTree在本地开发中，往往从远程库中拉取，久而久之，随着开发功能的不断进行，就会发现自己的远程库特别长，虽然在线上将远程库中的分支清理了，但是本地拉取的远程分支中，依然存在。
- 使用SourceTree查看：
![](http://img1-1253291688.cossh.myqcloud.com/Git/1.png)
<!-- more -->

- 命令行查看
![](http://img1-1253291688.cossh.myqcloud.com/Git/2.png)

## 清理
- ```git remote prune origin``` 
![](http://img1-1253291688.cossh.myqcloud.com/Git/3.png)

## 结果
- 使用SourceTree查看：
![](http://img1-1253291688.cossh.myqcloud.com/Git/4.png)
- 命令行查看
![](http://img1-1253291688.cossh.myqcloud.com/Git/5.png)

与清理前的相比，简直不要太清爽。