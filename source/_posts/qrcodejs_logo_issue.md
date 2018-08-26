---
title: QRCodeJS——logo
date: 2017/9/10 12:00:00
tags:
  - QRCodeJS
  - JavaScript
  - canvas
  - cross domain
categories: JavaScript
---

## 资源
详情见[项目地址](https://github.com/JianmingXia/QRCodeJS)

## 闲谈&缘由
QRCode加了logo之后，很久也没有修改过了，毕竟现在线上也都在稳定使用。但是本周项目中有人提了一个issue-[用toDataURL获取不到LOGO内容,想把canvas变成img，数据中没有LOGO内容](https://github.com/JianmingXia/QRCodeJS/issues/1)，总结下问题是：希望渲染出来的二维码是使用img标签的。
为什么此时的QRCodeJS做不到呢？因为我上次添加logo这个特性的时候，设置默认的就是canvas表现。
<!-- more -->

## 实现原理&解决
- img.src = canvas.toDataURL("image/png")
  直接如上设置，想法很简单，将canvas转换成图片，但是在实际运行后发现，img中只有一个二维码。原因很简单，没有考虑到图片加载的时间，执行canvas.toDataURL时，图片并没有渲染成功。


- 等待图片onload
  是的，QRCodeJS如果需要让渲染出来的图片是img标签展示的，需要等待logo图片加载完毕之后，所以，在时间上相对于直接使用canvas会长一点。


- 还有问题?
  我把临时版本发给了使用者，他反馈我说还有问题，如下：
  ![error](https://user-images.githubusercontent.com/28302478/28298978-08fe623e-6ba9-11e7-949e-f976c6c25855.png)
  这个问题其实很明显了，我们看[文档](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLCanvasElement/toDataURL)就知道了，引用的图片跨域了，canvas.toDataURL自然会报错。
  相关文章:
  https://stackoverflow.com/questions/20424279/canvas-todataurl-securityerror

  ## 后记
  上篇说到，最近其实看了不少东西，只是没有动笔，本来计划聊聊[quill](https://github.com/quilljs/quill)，但是恰逢QRCodeJs的问题，所以下次再说，再加一句，这款编辑器真的很不错。
