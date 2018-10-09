---
title: 博客起步
date: 2017/5/31 12:00:00
tags:
  - jacman
categories: 搭建博客

---
> 最终选用**Github Page**+**hexo**

原本的初衷是为了小程序所以注册了一个域名，在弄小程序期间，觉得用个二级域名来搞定博客也是一件很棒的事情，毕竟自己平时写的东西太零散了，有个集中地，何乐不为。

## 选型

这块网上资料很多，最后决定选用**jekyll**+**Github Page**，其中Git当然是用来托管代码，jekyll则是选用的博客模板。选用这个的原因有几点：
- 用的人很多——既然对这块不怎么熟悉，那就先选择随波逐流
- 免费啊——github不用多说

<!-- more -->

网上资料很多，大家肯定都能部署，[在此贴出一个](http://blog.csdn.net/wyc12306/article/details/51445623)。

Tips:
 - 基础：
    - 你在git上的名字即为——新建的项目名称
 - 这个时候一定要仔细，要仔细，要仔细；
 - 可能你也会遇到一些问题，比如默认的4000端口被占用，遇到问题一个一个解决就是，别着急。
<!-- more -->

## 换主题

上述完结之后，当然是选个有逼格的主题了，毕竟默认的也就那样。但是在找主题的时候，看到jacman很对胃口，于是就去git上去[下载](https://github.com/wuchong/jacman)去了。
对，没错，我看到了这句话：Jacman is a fresh looking and responsive theme for Hexo with more features and some build-in Chinese service based on Pacman.
没办法，看在主题的面子上，继续看下去，发现[文档](http://jacman.wuchong.me/2014/11/20/how-to-use-jacman/)也很详细，于是，jekyll改hexo了。这块安装也很简单，很快就安装好了。

## 主题配置

- 友情链接
- 各类图片
- 首页显示模式：我设置成文章展开式，结合```<!-- more -->```
- 作者信息
- 目录 toc
- 评论：作者推荐使用**多说**或**disqus**，但在我配置的时候，多说提示2017-06-01将停止服务，于是我选择了disqus，disqus被墙了，大家可能需要翻下，如果没有的话，可以试下**Lantern**。
- 网站统计：使用的是[百度统计](https://tongji.baidu.com/web/welcome/login)
- 自定义搜索：使用的是[Google CSE](https://www.google.com/cse/)，但是现在还没有搜索成功过。。。

## 总结
以上就是博客选型的过程了，无论怎么说，也是将自己的博客搭了个框架，具体内容还是得靠自己添砖加瓦了。
