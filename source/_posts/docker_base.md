---
title: Docker 基础
date: 2018/08/28 20:30:00
tags:
  - Docker
  - 阅读笔记
categories: Docker
---

## 为什么要使用Docker?

- 更高效的利用系统资源：容器不需要进行硬件虚拟以及进行完整操作系统等额外开销
- 更快速的启动时间：由于直接运行于宿主内核，无需启动完整的操作系统，可以做到秒级甚至毫秒级的启动
- 一致的运行环境：Docker的镜像提供了除内核外完整的运行时环境，确保了应用运行环境一致性
- 持续交付与部署：使用Dockerfile使镜像构建透明化，结合CI/CD
- 更轻松的迁移：Docker确保了执行环境的一致性，随时随地迁移
- 更轻松的维护与拓展：Docker使用的分层存储以及镜像的技术，使得应用重复部分的复用更为容易，也使得应用的维护更新更加简单，基于基础镜像进一步扩展镜像也变得非常简单

<!-- more -->

## 基本概念

### 镜像（Image）

> 关键字：分层存储

Docker镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。

镜像不包含任何动态数据，其内容在构建之后也不会再改变。

### 容器（Container）

镜像和容器的关系，就像是面向对象程序设计中的类和实例的关系。

镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的命名空间。

### 仓库（Repository）

集中的存储、分发镜像的服务

## 使用镜像

###  获取镜像

```dockerfile
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

#### 运行镜像

```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

### 列出镜像

```shell
docker image ls
```

#### 镜像体积

- Docker Hub上显示的大小是网络传输中更关注的流量大小；

- **docker image ls**展示的是镜像下载后展开的大小

#### 虚悬镜像

显示：

```
docker image ls -f dangling=true
```

删除：

```
docker image prune
```

#### 中间层镜像

为了加速镜像构建、重复利用资源，Docker会利用中间层镜像。

显示包括中间层镜像在内的所有镜像：

```
docker image ls -a
```

以特定格式显示

**docker image ls**支持很多参数

### 删除本地镜像

```
docker image rm [OPTIONS] IMAGE [IMAGE...]
```

#### 用ID、镜像名、摘要删除镜像

#### UnTagged 和 Deleted

镜像的唯一标识是其ID及摘要，一个镜像可以有多个标签。

当我们删除镜像时，实际上是在要求删除某个标签的镜像。

- 首先将镜像标签取消
- 如果没有其它标签指向这个镜像，就会发生Delete行为

#### 与docker image ls配合

使用**docker image ls -q**配合使用**docker image rm**，批量删除镜像。

## 操作容器

容器是独立运行的一个或一组应用，以及它们的运行态环境。

### 启动容器

两种方式：

- 基于镜像新建一个容器并启动
- 将正在终止状态的容器重新启动

#### 新建并启动

```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

当使用**docker run**来创建容器时，Docker在后台进行的标准操作：

- 检查本地是否存在指定的镜像，不存在即从仓库下载
- 利用镜像创建并启动一个容器
- 分配一个文件系统，并在只读的镜像层外挂载一个可读写层
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中
- 从地址池配置一个ip地址给容器
- 执行用户指定的应用程序
- 执行完毕后容器被终止

#### 启动已终止容器

```
docker container start [OPTIONS] CONTAINER [CONTAINER...]
```

### 后台运行

> 容器是否会长久运行，和**-d**无关

如果需要Docker在后台运行，而不是将执行命令的结果输出在当前宿主机上，可以使用**-d**参数。

使用**-d**启动会返回一个唯一的ID，也可以使用**docker container ls**命令来查看容器信息。

使用**docker container logs**命令可以获取容器的输出信息。

### 终止容器

```
docker container stop [OPTIONS] CONTAINER [CONTAINER...]
```

当Docker容器中指定的应用终结时，容器也自动停止。

处于终止状态的容器，可以使用**docker container start**命令来重新启动。

**docker container restart**命令会将一个运行态的容器终止，然后再启动它。

### 进入容器

> 可以使用**docker attach**或**docker exec**命令，推荐使用**docker exec**

#### attach 命令

```docker attach [OPTIONS] CONTAINER
docker attach [OPTIONS] CONTAINER
```

在attach时使用**exit**，会导致容器的停止

#### exec命令

```
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

> 主要是**-i -t**参数

此时使用**exit**不会导致容器的停止

### 导出和导入容器

#### 导出容器

```
docker export [OPTIONS] CONTAINER
```

#### 导入容器快照

```
docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
```

既可以使用**docker load**来导入镜像存储文件到本地镜像库，也可以使用**docker import**来导入一个容器快照到本地镜像库——两者的区别在于容器镜像快照文件将丢弃历史记录和元数据信息（仅保存容器当时的快照状态）

### 删除容器

```
docker container rm [OPTIONS] CONTAINER [CONTAINER...]
```

如果需要删除一个运行中的容器，可以传入**-f**参数，Docker会发送**SIGKILL**信号给容器。

#### 清理所有处于终止状态的容器

```
docker container prune [OPTIONS]
```

## 访问仓库

仓库是集中存储镜像的位置。

一个容易混淆的概念是注册服务器（Registry）——实际上注册服务器是管理仓库的具体服务器，每个服务器上有多个仓库，而每个仓库下面有多个镜像。

### Docker Hub

#### 注册

[官方地址](https://cloud.docker.com)

#### 登录

```
docker login [OPTIONS] [SERVER]
docker logout [SERVER]
```

#### 拉取镜像

可以使用**docker search**来查找镜像，**docker pull**下载镜像。

```
docker search [OPTIONS] TERM
```

#### 推送镜像

```
docker push [OPTIONS] NAME[:TAG]
```

#### 自动创建

对于需要经常升级镜像内程序来说，十分方便。

## Docker数据管理

在容器管理数据主要有两种方式：

- 数据卷
- 挂载主机目录

### 数据卷

数据卷是一个可供一个或多个容器使用的特殊目录，绕过UFS：

- 可以在容器之间共享和重用
- 对数据卷的修改会马上生效
- 对数据卷的更新不会影响镜像
- 数据卷默认会一直存在，即使容器被删除

相关命令：

```
docker volume COMMAND
```

### 挂载主机目录

#### 挂载一个主机目录作为数据卷

#### 挂载一个本地主机文件作为数据卷

## 资料
- https://legacy.gitbook.com/book/yeasy/docker_practice/details