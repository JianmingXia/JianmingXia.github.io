---
title: Ant Design + dva + Electron
tags:
  - Ant Design
  - dva
  - Electron
categories: 前端
---

## 说明
上周一开会后，确认了接下来的开发方向，之前研究的Blockly暂且放置，先完成一个内部的UI编辑器，基于React, Ant Design及Electron。

## 技术说明
### UI库——Ant Design
[React](https://reactjs.org/)自不必说，如今已是非常火了。[Ant Design](https://ant.design/docs/react/introduce-cn)有基于React的实现，这也是我们选用Ant Design的原因，而且就Ant Design而言，现在已是非常成熟的技术，它的特性：
- 提炼自企业级中后台产品的交互语言和视觉风格
- 开箱即用的高质量 React 组件
- 使用 TypeScript 构建，提供完整的类型定义文件
- 基于 npm + webpack + babel 的工作流，支持 ES2015 和 TypeScript

<!-- more -->

### 数据应用框架——dva
之前在学习React的时候，使用Redux进行数据管理，新增代码时需要在数个目录之间来回切换，较为繁琐。
dva——是一个基于 React 和 Redux 的轻量应用框架，概念来自 elm，支持 side effects、热替换、动态加载、react-native、SSR 等
[dva + antd实战应用文档](https://ant.design/docs/react/practical-projects-cn)

### 桌面端——Electron
使用 JavaScript, HTML 和 CSS 构建跨平台的桌面应用，我对此了解不多，文档也尚未看完。
- 主进程使用 BrowserWindow 实例创建页面
- 渲染进程顾名思义，负责渲染页面
- 渲染进程与主进程可通过ipc进行通信

## 成果
### 网页应用页面
![](https://img.ryoma.top/antd%26dva%26electron/0.png)

### 桌面应用页面（electron）
![](https://img.ryoma.top/antd%26dva%26electron/1.png)

### 项目源码（https://github.com/JianmingXia/StudyTest/tree/master/antd-dva/dva-restart）