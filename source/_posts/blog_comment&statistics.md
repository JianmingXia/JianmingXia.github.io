---
title: 博客优化
tags:
  - 博客
  - 评论
  - 阅读统计
  - 畅言
  - 阅读次数统计
categories: 基础
---

## 前言
想做就做，说那么多干嘛？

## 评论——[畅言](https://changyan.kuaizhan.com/)
`多说`很久之前就停了，`disqus`经常被墙，`网易云跟帖`好像也挂了，最近看到我使用的主题——jacman可支持的评论中加入了`畅言`，于是来试试。
从文档中没有搜索到`畅言`的使用方式，对主题部分的代码搜索了一番，配置好appid及appkey即可。
![](https://img.ryoma.top/blog/1.png)
<!-- more -->

## 阅读统计——[leancloud](https://leancloud.cn/)
今天一个朋友推荐了一篇阅读统计的文章，虽然之前给博客接入了统计，但是具体博客是没有的，恰好今天时间空闲，就试了下。leancloud创建参考这篇[参考文档](http://www.icafebolger.com/hexo/hexopostcount.html)，在项目中的引用也可以参考，但是我在原基础上做了修改。

- 配置部分（主题中的_config.yml文件）：
```
leancloud_counter:
  enable: true
  app_id: LkhcrGAzL7hq4Sa3QUC7r2ua-gzGzoHsz
  app_key: gg6plik3hO1URckNCwpJGuUs
```

- 阅读次数的展示（主题中layout\_partial\post\header.ejs文件），按自己的习惯调整位置:
```
  <p class="article-counter"> 阅读次数:
    <% if(theme.leancloud_counter.enable){ %>
      <span id="<%= url_for(item.path) %>" class="leancloud_visitors" data-flag-title="<%- item.title %>">--</span>
      <% }else{ %>
        --
        <% } %>
  </p>
```

- 阅读次数统计及显示（主题中layout\_partial\after_footer.ejs文件）:
  - 改了什么
    addCount：如果是在首页，统计hostname；否则统计对应博客
    showTime：显示每个博客的阅读次数（如果是在首页，请求会比较多，如果觉得不合适，也可以做修改）

```
<% if (theme.leancloud_counter.enable){ %>
<script src="//cdn1.lncld.net/static/js/2.5.0/av-min.js"></script>
<script>
  var APP_ID = '<%- theme.leancloud_counter.app_id %>';
  var APP_KEY = '<%- theme.leancloud_counter.app_key %>';
  AV.init({
    appId: APP_ID,
    appKey: APP_KEY
  });
  // 显示次数
  function showTime(Counter) {
    $(".leancloud_visitors").each(function (i) {
      var query = new AV.Query("Counter");
      var url = $(this).attr('id').trim();
      // where field
      query.equalTo("words", url);
      // count 
      query.count().then(function (number) {
        // There are number instances of MyClass where words equals url.
        $(document.getElementById(url)).text(number ? number : '--');
      }, function (error) {
        // error is an instance of AVError.
      });
    })
  }
  // 追加pv
  function addCount(Counter) {
    var url = location.hostname;
    if (location.pathname.length > 1 && $(".leancloud_visitors").length > 0) {
      url =  $(".leancloud_visitors").attr('id').trim();
    }

    var Counter = AV.Object.extend("Counter");
    var query = new Counter;
    query.save({
      words: url
    }).then(function (object) {
    })
  }
  $(function () {
    var Counter = AV.Object.extend("Counter");

    addCount(Counter);
    showTime(Counter);
  });
</script>
<% } %>
```

- 最终效果
![](https://img.ryoma.top/blog/2.png)