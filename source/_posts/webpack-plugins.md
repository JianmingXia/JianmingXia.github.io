---
title: Webpack Plugins
date: 2017/7/21 12:00:00
tags:
  - Webpack
  - Plugins
categories: 前端
---

## 前言
最近接触的很多项目都使用了webpack，也在更深入的了解，做些学习的笔记，这篇主讲webpack中的插件，会持续更新中~[Github](https://github.com/JianmingXia/StudyTest/tree/master/TypeScript/demo_2)
<!-- more -->

## Plugins
### CommonsChunkPlugin
> The CommonsChunkPlugin is an opt-in feature that creates a separate file (known as a chunk), consisting of common modules shared between multiple entry points. By separating common modules from bundles, the resulting chunked file can be loaded once initially, and stored in cache for later use. This results in pagespeed optimizations as the browser can quickly serve the shared code from cache, rather than being forced to load a larger bundle whenever a new page is visited.

如文档中介绍般，会将多个入口中的共享模块组成，这样在页面加载中，这部分共享模块只需要加载一次，后续如果使用到，可以快速调用，而不需要每次都加载。

#### demo

```
  new webpack.optimize.CommonsChunkPlugin({
    name: 'vendor',
    filename: 'vendor.min.js',
  })
```
生成的文件需要引入，如果你是在html中手动引入html文件。需要保证文件**在入口文件前引入**，切记！

```
<script src="../dist/vendor.min.js"></script>

<script src="../dist/bundle.js"></script>
```

更多参数见[文档](https://webpack.js.org/plugins/commons-chunk-plugin/)。

### HtmlWebpackPlugin
>The HtmlWebpackPlugin simplifies creation of HTML files to serve your webpack bundles. This is especially useful for webpack bundles that include a hash in the filename which changes every compilation. You can either let the plugin generate an HTML file for you, supply your own template using lodash templates, or use your own loader.

我们可以使用这个插件生成html文件，或者使用我们自己的模板文件。
这是一个我特别喜欢的插件，让我们构建html文件超级简单，容我细细说来。

#### 安装

```
npm install --save-dev html-webpack-plugin
```

#### 无配置使用

- 如果我们不打算写html文件，直接使用HtmlWebpackPlugin:
```
new HtmlWebpackPlugin()
```

- 生成的html文件(dist目录)：
```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Webpack App</title>
  </head>
  <body>
  <script type="text/javascript" src="vendor.min.js"></script><script type="text/javascript" src="bundle.js"></script></body>
</html>
```
看，连使用CommonsChunkPlugin生成**vendor.min.js**文件都自动帮我们引入了。但是如果我们想修改title等属性怎么办？接着看下去。

#### 配置使用

- 新的配置：
```
  new HtmlWebpackPlugin({
    title: 'My App',
    filename: 'index.html',
    template: "./src/index.html"
  })
```



- 引用的template文件：
```
<!DOCTYPE html>
<html>

<head>
  <meta charset="UTF-8" />
  <title>
    <%= htmlWebpackPlugin.options.title %>
  </title>
</head>

<body>
  <div id="content">

  </div>
</body>

</html>
```

- 生成的文件：
```
<!DOCTYPE html>
<html>

<head>
  <meta charset="UTF-8" />
  <title>
    My App
  </title>
</head>

<body>
  <div id="content">

  </div>
<script type="text/javascript" src="vendor.min.js"></script><script type="text/javascript" src="bundle.js"></script></body>

</html>
```

### DefinePlugin
>The DefinePlugin allows you to create global constants which can be configured at compile time. This can be useful for allowing different behavior between development builds and release builds. If you perform logging in your development build but not in the release build you might use a global constant to determine whether logging takes place. That's where DefinePlugin shines, set it and forget it rules for development and release builds.

使用**DefinePlugin**，就可以在编译时就配置全局变量，比如项目中引用的cdn地址，本地、测试、线上等使用的设置成不同的地址，超级方便。

#### 配置
```
var yargs = require('yargs');
var env = yargs.argv.env;
var tmp_global_var = {
  local_develop: false,
  is_production: false
};

switch (env) {
  case "local":
    tmp_global_var.local_develop = true;
    break;
  default:
    tmp_global_var.is_production = true;
    break;
}

plugins.push(new webpack.DefinePlugin({
  LOCAL_DEVELOP: JSON.stringify(tmp_global_var["local_develop"]),
  IS_PRODUCTION: JSON.stringify(tmp_global_var["is_production"])
}));
```

结合**yargs**，获取webpack打包时添加的参数，与此同时，在**package.json**中添加配置：
```
  "scripts": {
    "local": "webpack --env local",
    "production": "webpack"
  }
```

这样就可以通过**npm run xxx**来决定在什么场景下打包。最后是使用：
```
if (IS_PRODUCTION) {
  addHtmlContent("content", "\nIS_PRODUCTION");
} else if (LOCAL_DEVELOP) {
  addHtmlContent("content", "\nLOCAL_DEVELOP");
}
```

## 结语
webpack中的plugins，更加高效的使用Webpack

