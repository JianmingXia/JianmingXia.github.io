---
title: 将一个项目中的提交应用到其它项目-Git
tags:
  - git
  - 远程库
  - 同步更新
  - cherry-pick
categories: Git
---

## 缘由
有这个需求的原因是由于此次版本的更新，由于产品方向修改，原有项目需要一分为二，我们目前设定为A 和 B项目。此时A和B项目都基于同一个项目的代码，但之后，A和B项目前进的方向会不同，但在版本推进的过程中，会有某些feature两个项目都需要做。如果修改A之后，需要手动将代码同步至B，如果修改的文件太多，对于开发来说，简直是不可想象的事。
<!-- more -->

## 行动
### A B项目基于同一套代码，包括之前的commit全部一样
![](http://img1-1253291688.cossh.myqcloud.com/Git/2_0.png)

### git remote add ryoma-test git@git.quqi.com:html/html-doc-draft.git
- git remote
![](http://img1-1253291688.cossh.myqcloud.com/Git/2_2.png)
在A项目中，建立一个远程库，指向B项目
![](http://img1-1253291688.cossh.myqcloud.com/Git/2_1.png)

### git push ryoma-test bugfix/ryoma/git-test
- 在A上完成了feature，现在希望同步到B上
![](http://img1-1253291688.cossh.myqcloud.com/Git/2_4.png)
![](http://img1-1253291688.cossh.myqcloud.com/Git/2_3.png)

### 切换至B项目：git remote show origin
- 在A项目中的feature分支出现在B项目中
![](http://img1-1253291688.cossh.myqcloud.com/Git/2_5.png)

### 使用cherry-pick
- merge 指定的commit

## 结语
此次更新之前，没有如此使用git，不得不佩服git的智慧。