---
title: 二维码添加图片
tags:
  - QRCodeJS
  - JavaScript
  - canvas
categories: JavaScript
---

## 成果
话不多说，先上成果
[不加密的源码](https://github.com/JianmingXia/QRCodeJS)
![二维码](https://img.ryoma.top/QRCodeJS/qrcode.png)
<!-- more -->

## 起源
由于公司的项目中需要展示二维码，但是由于每个链接都需要生成二维码并且节点还在不断增加中，二维码如果由后端生成并保存，消耗太大。所以二维码的生成确定在前端完成，首先从Github上找到了两个不错的资源，
![Github资源](https://img.ryoma.top/QRCodeJS/github_qrcode.png)
其中：
- qrcodejs不依赖任何库
- jquery-qrcode很明显，依赖jquery

但是现有的需求是：
- 生成二维码
- 添加图片
以上两个都仅支持生成二维码，没有添加图片的方法。
在做之前，也在网上搜索了很多解决方法：
- jquery-qrcode有人实现了添加图片，本来想一窥实现方法的，可是贴出来的代码是加密的（很尴尬，加密的代码看得也太累了，很难满足好奇心啊）
- 可以选择在生成二维码的div里面放个**<img\>**标签，然后调整img位置至中间——其实这个效果是有了，但是保存图片时，二维码还是单纯只有二维码。

所以，我打算自己写了，基于QRCodeJS（项目保持着尽少的依赖）。

## 贴代码
其实实现原理很简单，看QRCodeJS代码就知道绘制二维码是使用canvas的，我们的实现也很简单，使用canvas将图片绘制进去，显示canvas。
- 核心代码
{% codeblock %}
if (this._htOption.img_src && _isSupportCanvas) {
    var base_image = new Image();
    base_image.src = this._htOption.img_src;

    var qrcode_width = this._htOption.width;
    var img_width = this._htOption.img_width;

    var margin = (qrcode_width - img_width) / 2;

    // 一定要等到图片加载好了才开始绘制
    base_image.onload = function(){
        _oContext.drawImage(base_image, margin, margin, img_width, img_width);
    }
}
{% endcodeblock %}

## 注意
- 原来代码中在使用canvas绘制出图片后，直接将canvas中的内容转成了img，同时将canvas设置为
{% codeblock %}
display:none;
{% endcodeblock %}
这块我也修改了：
{% codeblock %}
if(this._htOption.img_src) {
    this._elImage.style.display = "none";
    this._elCanvas.style.display = "block";
} else {
    this._elImage.src = this._elCanvas.toDataURL("image/png");
    this._elImage.style.display = "block";
    this._elCanvas.style.display = "none";
}
{% endcodeblock %}
- 我复制出来QRCodeJS.js在webpack打包后，引用的过程中提示"_android is undefined",
{% codeblock %}
if (this._android && this._android <= 2.1)
{% endcodeblock %}
修改为：
{% codeblock %}
if (this && this._android && this._android <= 2.1)
{% endcodeblock %}
原因可能是因为webpack打包后加上了"use strict"，而严格模式下，this的值为undefined。
具体的原因不是很清楚，毕竟前端技术有点薄弱，哪位大神路过讲解下。。。

## 总结
无论如何，给二维码添加图片这个问题解决了。
![二维码](https://img.ryoma.top/QRCodeJS/qrcode.png)
