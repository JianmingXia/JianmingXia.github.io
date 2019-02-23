---
title: Docker 部署 Node 项目
date: 2018/09/01 18:30:00
tags:
  - Docker
  - NodeJS
categories: 
  - [Docker]
  - [NodeJS]
---

> docker是一个开源的应用容器引擎，可以为我们提供安全、可移植、可重复的自动化部署的方式。


为什么要用 Docker 呢？这个[文档](https://blog.ryoma.top/posts/docker_base/)一开始就说得很清楚了，这里只关注如何使用 Docker 部署一个 Node 项目。

## 创建 Node 项目
### package.json
创建一个 package.json 文件，描述应用程序及其依赖：

```
{
  "name": "docker_web_app",
  "version": "1.0.0",
  "description": "docker web app in nodejs",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "keywords": [
    "docker"
  ],
  "author": "ryoma",
  "license": "ISC",
  "dependencies": {
    "express": "^4.16.4"
  }
}
```

<!-- more -->

### Demo 应用
> index.js


```
const express = require("express");

// Constants
const PORT = 8080;
const HOST = "0.0.0.0";

// App
const app = express();
app.get("/", (req, res) => {
  console.log("receiving request...");
  res.send("Hello Docker!\n");
});

app.listen(PORT, HOST);
console.log(`Running on http://${HOST}:${PORT}`);
```

## 准备工作
> 请提前安装好 Docker


### Dockerfile

```
FROM node:8.9

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install
# If you are building your code for production
# RUN npm ci --only=production

# Bundle app source
COPY . .

EXPOSE 8080
CMD [ "npm", "start" ]
```

现在来解释上面的定义：
* FROM：指定基础镜像（必须是 Dockerfile 文件的第一个指令），如果镜像不存在默认从 Docker Hub 下载
* WORKDIR：设置 Dockerfile 中的RUN、CMD 和 ENTRYPOINT 指令执行命令的工作目录；可以出现多次，如果使用相对路径则为相对于WORKDIR上一次的值，
* RUN：会在一个新的容器中执行任何命令，然后把执行后的改变提交到当前镜像
* EXPOSE：告诉 Docker 这个容器在运行时会监听哪些端口
* CMD：CMD指令中指定的命令会在镜像运行时执行，在 Dockerfile 中只能存在一个，如果使用了多个 CMD 指令，则只有最后一个 CMD 指令有效。
  * 当出现 ENTRYPOINT 指令时，CMD 中定义的内容会作为 ENTRYPOINT指令的默认参数
  * RUN 和 CMD 都是执行命令，差异在于 RUN 中定义的命令会在执行 **docker build** 命令创建镜像时执行，而 CMD 中定义的命令会在执行 docker run 命令运行镜像时执行

### .dockerignore
与 **.gitignore** 文件功能类似，用户忽略文件——这里指的是当构建镜像时，需要忽略的内容

```
node_modules
npm-debug.log
```

## 构建镜像
### 构建镜像
> 这里 ryoma1992 是我本人在 Docker Hub 中的账号，在构建镜像时，可以不用


```
docker build -t ryoma1992/node-web-app:1.0.0 .
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550927788761-0d98cb71-87e1-4018-ad12-22721d191cea.png#align=left&display=inline&height=394&linkTarget=_blank&name=image.png&originHeight=788&originWidth=1158&size=169300&status=done&width=579)

#### docker login
如果在 docker build 时遇到此类错误，可以使用 docker login 先登录

```
Step 1/7 : FROM node:8.9
Get https://registry-1.docker.io/v2/library/node/manifests/8.9: unauthorized: incorrect username or password
```

### 查看镜像
使用 docker image ls 命令即可查看之前构建的镜像

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550927871087-7eb5d9e5-24ee-4f9a-8f63-06b32442a43a.png#align=left&display=inline&height=48&linkTarget=_blank&name=image.png&originHeight=96&originWidth=1498&size=33684&status=done&width=749)

## 运行
### 运行

```
docker run -d -p 20000:8080 ryoma1992/node-web-app:1.0.0
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550927935904-aa04674e-f78f-46aa-bd83-c333270595ae.png#align=left&display=inline&height=35&linkTarget=_blank&name=image.png&originHeight=70&originWidth=1362&size=26210&status=done&width=681)

其中：
* -d：表示后台运行
* -p：表示端口映射关系，这里是本地 20000 映射容器内的8080

### 查看运行情况
使用 docker ps 来查看容器

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550927980365-f1dfd181-5331-45c2-9951-b0a503fc7119.png#align=left&display=inline&height=53&linkTarget=_blank&name=image.png&originHeight=106&originWidth=2540&size=41496&status=done&width=1270)

#### 浏览器访问
![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550927463090-05e9f8a4-2a73-4b03-8a3e-e5504a9749ce.png#align=left&display=inline&height=81&linkTarget=_blank&name=image.png&originHeight=162&originWidth=782&size=14152&status=done&width=391)

#### docker logs

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550927434164-599dedff-7816-4a49-9bcd-af661d77d901.png#align=left&display=inline&height=102&linkTarget=_blank&name=image.png&originHeight=204&originWidth=1004&size=37031&status=done&width=502)

## 上传镜像
### docker tag
> [https://docs.docker.com/engine/reference/commandline/tag/](https://docs.docker.com/engine/reference/commandline/tag/)
> 这里我提前编译了一个版本，镜像命名为 node-web-app:1.0.1


```
// 格式
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]

// 实际运行
docker tag node-web-app:1.0.1 ryoma1992/node-web-app:1.0.1
```

### 上传

```
docker push ryoma1992/node-web-app:1.0.1
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550929033517-5cb31640-162f-4857-9d23-633012448d52.png#align=left&display=inline&height=247&linkTarget=_blank&name=image.png&originHeight=494&originWidth=1594&size=150710&status=done&width=797)

### Docker Hub 查看
![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550928859692-f356839e-ad01-4e3c-a7ac-76da2d39ad6d.png#align=left&display=inline&height=457&linkTarget=_blank&name=image.png&originHeight=914&originWidth=1254&size=91543&status=done&width=627)

## 去部署服务器运行
之前的运行是在本地运行的，但是在上一步我们将镜像推到了 Docker Hub。现在在一台新服务器中运行：

### 拉取镜像 & 运行

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550934097845-018eeae2-8ab0-4b6e-af25-a0773c43ee85.png#align=left&display=inline&height=347&linkTarget=_blank&name=image.png&originHeight=694&originWidth=1516&size=415947&status=done&width=758)

注意到第一行：本地没有目标镜像，故而会从 Docker 仓库中下载

### 访问

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550934190513-2d5a1619-08fd-4426-ac78-8b1ca3135d64.png#align=left&display=inline&height=37&linkTarget=_blank&name=image.png&originHeight=74&originWidth=882&size=35269&status=done&width=441)

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550934219368-89291a01-9dad-4518-a1b6-6dec4abe8986.png#align=left&display=inline&height=129&linkTarget=_blank&name=image.png&originHeight=258&originWidth=808&size=88059&status=done&width=404)

## 资料
* [https://nodejs.org/zh-cn/docs/guides/nodejs-docker-webapp/](https://nodejs.org/zh-cn/docs/guides/nodejs-docker-webapp/)
* [https://blog.ryoma.top/posts/docker_base/](https://blog.ryoma.top/posts/docker_base/)
