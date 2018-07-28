---
title: nginx 源码安装
date: 2017/12/31 12:00:00
tags:
  - linux
  - nginx
  - nginx 安装
categories: nginx
---

## 前言
很久之前就打算记录下nginx源码的安装记录，因为最初搭建服务器的使用，是直接使用了yum直接安装，毕竟安装方式粗暴简单。后端开发也很久了，但是从来也没怎么注意服务器这块，所以尝试使用源码包来安装nginx，而且在当前服务器编译，更符合系统的性能，性能效率也会更好。

## 我为什么会选择使用源码包安装
- 安装位置不受控制
  我更希望能够自定义安装目录，包括名称，确认我知道自己安装的nginx版本
- 启动方式不同
  使用yum安装后，需要使用系统服务命令 service来启动等操作：如service nginx start/stop/restart

<!-- more -->

## 源代码安装
### 准备依赖包
稍后再进行nginx编译的时候，需要提前准备一些依赖包，如openssl、pcre等等。我安装的如下：
```
yum -y install gcc gcc-c++ make libtool zlib zlib-devel openssl openssl-devel pcre pcre-devel
```

注意：使用yum安装依赖库的时候，可能会安装失败，如**Protected multilib versions**，从提示上看，应该是多个库版本的问题，如zlib，解决方法如下：
```
yum install -y zlib zlib-devel --setopt=protected_multilib=false
```

### 下载源码
从[nginx下载](http://nginx.org/en/download.html)1.10.3，可以放在**/usr/local/src**目录，挑选希望安装的版本：
```
wget http://nginx.org/download/nginx-1.10.3.tar.gz
```

然后解压缩：
```
tar -zxvf nginx-1.10.3.tar.gz
```

### 开始编译
编译开始之前，最好先了解下[编译参数](https://segmentfault.com/a/1190000002797601)。
nginx大部分常用模块，编译时./configure --help以--without开头的都默认安装。
  - --prefix=PATH ： 指定nginx的安装目录。默认 /usr/local/nginx
  - --conf-path=PATH ： 设置nginx.conf配置文件的路径。nginx允许使用不同的配置文件启动，通过命令行中的-c选项。默认为prefix/conf/nginx.conf
  - --user=name： 设置nginx工作进程的用户。安装完成后，可以随时在nginx.conf配置文件更改user指令。默认的用户名是nobody。--group=name类似
  - --with-pcre ： 设置PCRE库的源码路径，如果已通过yum方式安装，使用--with-pcre自动找到库文件。使用--with-pcre=PATH时，需要从PCRE网站下载pcre库的源码（版本4.4 - 8.30）并解压，剩下的就交给Nginx的./configure和make来完成。perl正则表达式使用在location指令和 ngx_http_rewrite_module模块中。
  - --with-zlib=PATH ： 指定 zlib（版本1.1.3 - 1.2.5）的源码解压目录。在默认就启用的网络传输压缩模块ngx_http_gzip_module时需要使用zlib 。
  - --with-http_ssl_module ： 使用https协议模块。默认情况下，该模块没有被构建。前提是openssl与openssl-devel已安装
  - --with-http_stub_status_module ： 用来监控 Nginx 的当前状态
  - --with-http_realip_module ： 通过这个模块允许我们改变客户端请求头中客户端IP地址值(例如X-Real-IP 或 X-Forwarded-For)，意义在于能够使得后台服务器记录原始客户端的IP地址
  - --add-module=PATH ： 添加第三方外部模块，如nginx-sticky-module-ng或缓存模块。每次添加新的模块都要重新编译（Tengine可以在新加入module时无需重新编译）

我的编译方式：
```
./configure --prefix=/usr/local/nginx-1.10.3 --with-http_stub_status_module --with-http_ssl_module --with-pcre

make && make install
```
编译结束后，绝对路径启动即可。

## 更多
在上面编译参数中，有个参数是`--add-module`，这是用来编译第三方模块的。这里有个[第三方模块](https://www.nginx.com/resources/wiki/modules/index.html)的列表，当然，你也可以根据需要手写模块。

## 结语
如果希望安装其他的nginx，也可以按照此方法继续安装，需要注意的是，如果想要启动多个nginx，需要注意端口的占用。

## 参考文档
- http://www.evanmiller.org/nginx-modules-guide.html
- https://www.nginx.com/resources/wiki/start/
- https://wujunze.com/ngx_module_dev.jsp