---
title: Quill
tags:
  - QuillJS
  - JavaScript
  - Rich Text Editor
categories: JavaScript
---

## 闲谈

上篇说到会说下Quill，这款UI编辑器我也忘记是在什么地方看到的，但是第一眼瞧上，就觉得很棒，很简洁（可能是由于之前使用的是Ueditor，觉得太笨重了）。<br>
那么说下我觉得Quill的优点：
- 文档丰富，方法调用实在简单，并且在git上有足够的活跃度，有问题随时提
- js文件小（min.js 40kb左右），没有依赖项
- 开源（在开源无比活跃的当下，开源就意味着无限可能）
- 兼容性好
![这张图来自https://github.com/quilljs/quill](https://camo.githubusercontent.com/5c3c542f0783c0c19c59d13927247502aa55c98a/68747470733a2f2f63646e2e7175696c6c6a732e636f6d2f62616467652e7376673f763d32)
- 可拓展性强（如果不满足于现有的接口，笔交给你，你来改）

<!-- more -->

## 开始使用这款富文本编辑器
- [引用Quill](https://quilljs.com/docs/download/)
    - 选择直接引用js文件或者npm安装

- [Formats](https://quilljs.com/docs/formats/)
    - 支持多种格式

- [Api](https://quilljs.com/docs/api/)
    - 各种Api调用，通过Api完美holdQuill

- [Delta](https://quilljs.com/docs/delta/)
    - 基本是以JSON格式存储的，包括文本及格式

- [Modules](https://quilljs.com/docs/modules/)
    - 模块使得Quill的行为和功能皆可被定制，当然，如果你有需要，你也可以拓展或重写这些模块

## demo
我也简单写了个[demo](https://demo.ryoma.top/quill/)，查阅与编辑，大家如果感兴趣，赶快去研究文档吧。
![quill操作](https://img.ryoma.top/quill/quill.gif)
