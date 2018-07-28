---
title: TypeScript & Webpack 
tags:
  - Webpack
  - TypeScript
categories: 前端
---

## 前言
作为一个走在时尚前沿的人，身被十八般武艺是必须的。这次说的TypeScript及Webpack，关于TypeScript:
>TypeScript 是微软开发的 JavaScript 的超集，TypeScript兼容JavaScript，可以载入JavaScript代码然后运行。TypeScript与JavaScript相比进步的地方。包括：加入注释，让编译器理解所支持的对象和函数，编译器会移除注释，不会增加开销；增加一个完整的类结构，使之更新是传统的面向对象语言。
大家完全不用担心浏览器对TypeScript的兼容问题，因为我们在编译期生成JavaScript代码。

而TypeScript吸引到我的就是**静态类型检查功能**，更多精彩，大家可以去[体验](https://www.tslang.cn/docs/home.html)。
<!-- more -->

## 安装TypeScript
> npm install -g typescript

 ```
 // greeter.ts
 function greeter(person) {
    return "Hello, " + person;
}

var user = "Jane User";

document.body.innerHTML = greeter(user);
 ```

 运行TypeScript编译器：
 ```
 tsc greeter.ts
 ```
 就可以转成greeter.js了。

 ## webpack
 上面讲了单个文件的编译，但是如果想在项目中使用TypeScript，应该怎么用呢。
 参照[官方文档](https://www.tslang.cn/docs/handbook/react-&-webpack.html)，官方文档中结合了React，我在这儿，只引入Webpack。

 ### 初始化项目结构
 ```
 mkdir demo_1
 cd demo_1
 ```
 在demo_1中新建两个文件夹，"src"及"dist"，分别存储项目源码及打包后代码。

 
 ### 初始化工程
现在把这个目录变成npm包。
```
npm init
```
如果不理解意思，可以使用默认值，最后会在项目根目录生成package.json，后续可以继续修改。

### 安装依赖
首先安装webpack，如果大家之前没有接触webpack，[传送门](https://webpack.js.org/configuration/)
```
npm install -g webpack
```

```
npm install --save-dev typescript awesome-typescript-loader source-map-loader
```
这些依赖会让TypeScript和webpack在一起良好地工作。 awesome-typescript-loader可以让Webpack使用TypeScript的标准配置文件 tsconfig.json编译TypeScript代码。 source-map-loader使用TypeScript输出的sourcemap文件来告诉webpack何时生成 自己的sourcemaps。

### 添加TypeScript配置文件
创建tsconfig.json
```
{
    "compilerOptions": {
        "outDir": "./dist/",
        "sourceMap": true,
        "noImplicitAny": true,
        "module": "commonjs",
        "target": "es5"
    },
    "include": [
        "./src/**/*"
    ]
}
```

### 开始code
接下来在src中添加代码:
Student.ts
```
class Student {
  fullName: string;
  constructor(public firstName: string, public lastName: string) {
    this.fullName = firstName + " " + lastName;
  }
}

export default Student;
```
Utils.ts
```
interface Person {
  firstName: string;
  lastName: string;
}

function greeter(person: Person) {
  return "Hello, " + person.firstName + " " + person.lastName + " !";
}

function addHtmlContent(id: string, content: string) {
  let div = document.getElementById(id);
  div.innerText += content;
}

export { greeter, addHtmlContent }
```
index.ts
```
import { greeter, addHtmlContent } from './Utils';
import Student from './Student';

let user = new Student("TypeScript", "User");


addHtmlContent("content", greeter(user));

addHtmlContent("content", "\n" + user.fullName);
```
index.html
```
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>

<body>
  <div id="content"></div>

  <!-- Main -->
  <script src="../dist/bundle.js"></script>
</body>

</html>
```

### webpack配置文件
webpack.config.js
```
module.exports = {
  entry: "./src/index.ts",
  output: {
    filename: "bundle.js",
    path: __dirname + "/dist"
  },

  // Enable sourcemaps for debugging webpack's output.
  devtool: "source-map",

  resolve: {
    // Add '.ts' and '.tsx' as resolvable extensions.
    extensions: [".ts", ".tsx", ".js", ".json"]
  },

  module: {
    rules: [
      // All files with a '.ts' or '.tsx' extension will be handled by 'awesome-typescript-loader'.
      { test: /\.tsx?$/, loader: "awesome-typescript-loader" },

      // All output '.js' files will have any sourcemaps re-processed by 'source-map-loader'.
      { enforce: "pre", test: /\.js$/, loader: "source-map-loader" }
    ]
  }
};
```

### 模块化打包
```
webpack
```

## 结语
至此，TypeScript结合Webpack的demo就完成了，可以放心在自己的项目结合TypeScript使用[源码传送门](https://github.com/JianmingXia/StudyTest/tree/master/TypeScript/demo_1)

