---
title: Mysql少有的使用
tags:
  - Mysql
  - left right concat
categories: Mysql
---

# Mysql 更新某个字段的部分数据
- 之所以会有Mysql相关，是因为最近版本对数据库进行了升级，之前的字段设计没有做好，此次升级会对原有字段进行修改。总体来说，即对原有字段进行截取，组合成其它的值。

## left
#### 截取左边
- update str set str = left(str, 8);

<!-- more -->

#### select左边
- SELECT LEFT(str, 5) from str;

## right
#### 截取右边（与左边类似）
- 文档：Returns the rightmost len characters from the string str, or NULL if any argument is NULL.
- update str set str = right(str, 7);

#### select右边
- SELECT right(str, 4) from str;

## concat
#### 连接
- update str set str = concat(left(str, 3), "aaa", right(str, 3))

# 题外话
- 看到了上次的blog，五月十二号，写博客真的是一件很难的事，之前好不容易开始了，没想到还是停下了。看的东西倒是挺多，但是很难提笔去写，这段时间沉迷农药不可自拔，该打。
- 竟然连本地hexo的使用都忘记了，幸好存了这篇[入门文章](http://blog.ryoma.top/2017/05/06/hello-world/)。
- 启动hexo之后，localhost:4000没办法打开，果然4000端口被占用：
  ![端口占用](http://img1-1253291688.cossh.myqcloud.com/Mysql/%E5%8D%A0%E7%94%A8%E7%AB%AF%E5%8F%A3.png)
由俭入奢易，由奢入俭难啊，篇幅不长，但是当做新的开始。捋起袖子努力干！！！
