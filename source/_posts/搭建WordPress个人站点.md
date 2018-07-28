---
title: 搭建WordPress个人站点
tags:
  - WordPress
  - 个人博客
  - Mysql
categories: 基础
---

> WordPress 是一款常用的搭建个人博客网站软件，使用 PHP 语言和 MySQL 数据库开发。

## 前言
近期我的女神希望建立自己的个人博客，明确表示不喜欢平时我写博客的方式——目前我使用hexo+github的方式。她希望有一种创建博客的感觉，能够所见即所得，而且不喜欢用markdown语法，就目前而言，我能想到的只有WordPress，恰好最近在腾讯云上买了一台服务器，于是立即着手。<br>
在构建WordPressd的时候，想起之前在腾讯云上买服务器的时候，看到类似的教程，于是搜索到了对应的[教程](https://cloud.tencent.com/document/product/213/8044)，安装的时候也都是参照此文档。
<!-- more -->

## 创建并运行云服务器
服务器之前已经买过，连接服务器我还是更喜欢用XShell，大家看喜好。

## 搭建 LNMP 环境
### 使用 Yum 安装必要软件
#### 安装基础部分
```
yum install php php-fpm php-mysql mysql-server -y
```
注意：
- nginx之前使用源码包已经安装过，在此不再重新安装。

但是安装完成之后，发现已安装的列表中，不包括mysql，这让我很是费解。在查看这个[链接](https://cloud.tencent.com/document/product/213/2046)的时候，发现如下这段话：
```
   从 CentOS 7 系统开始，MariaDB 成为 yum 源中默认的数据库安装包。在 CentOS 7 及以上的系统中使用 yum 安装 MySQL 包将无法使用 MySQL。您可以选择使用完全兼容的 MariaDB，或点击 参阅此处 进行较低版本的 MySQL 的安装。
```

#### 安装mysql(我的服务器是CentOS 7.2)
根据这个[链接](https://www.linode.com/docs/databases/mysql/how-to-install-mysql-on-centos-7/)进行安装。安装花费了很长时间，很多源码包下载的速度从几KB降到几B，重复换源下载，这个过程浪费了很多时间。

### 软件配置

#### 配置Nginx
按照我的习惯，我在conf/vhost目录下，建立了对应的配置文件**xx.conf**，然后重启nginx

#### 配置mysql

- 启动msyql
```
service mysqld start
```

- 设置密码
```
/usr/bin/mysqladmin -u root password "123456"
```

#### 配置 PHP
- 启动 PHP-FPM 服务
```
service php-fpm start
```

- 配置 PHP Session 的存储路径
```
vi /etc/php.ini
修改 session.save_path = "/var/lib/php/session"
```

#### 验证环境配置
在nginx配置的目录中，使用**index.php**验证，确认是否配置成功。

## 安装和配置 WordPress

### 下载 WordPress
- 先删除nginx配置目录中的**index.php**

### 依次下载 WordPress 并解压到当前目录
我使用的是**/usr/local/src**目录：
```
wget https://cn.wordpress.org/wordpress-4.7.4-zh_CN.tar.gz
tar zxvf wordpress-4.7.4-zh_CN.tar.gz
```
### 配置数据库
- 登录 MySQL 服务器
```
mysql -u root -p
```
输入之前设置的密码

- 为 WordPress 创建数据库并设置用户名和密码（可自定义）
```
CREATE DATABASE wordpress;
CREATE USER user@localhost;
SET PASSWORD FOR user@localhost=PASSWORD("wordpresspassword");
```

- 为创建的用户开通数据库 “wordpress” 的完全访问权限
```
GRANT ALL PRIVILEGES ON wordpress.* TO user@localhost IDENTIFIED BY 'wordpresspassword';
```

- 使所有配置生效
```
FLUSH PRIVILEGES;
```
- 退出mysql

### 写入数据库信息
- 创建新配置文件
将wp-config-sample.php文件复制到名为wp-config.php的文件
```
cd wordpress/
cp wp-config-sample.php wp-config.php
```

- 编辑新创建的配置文件
```
vi wp-config.php
```
配置如下：

```
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'user');

/** MySQL database password */
define('DB_PASSWORD', 'wordpresspassword');
```

### 安装 WordPress
- 移至nginx配置的目录
```
mv * /xjm/run/wordpress/
```
访问即可，这个时候填入基本信息，点击**安装WordPress**，这个速度应该是很快的。

## 其它
### Navicat访问数据库
这个时候通过navicat访问数据库是无法访问的，我们在创建user时，是否还记得@后面的localhost，它表示只能在这台服务器进行登录。
```
mysql -u root -p
use mysql;
update user set host = '%' where user = 'user';
```
再使用Navicat试试

### Https
可以在腾讯云申请一个证书，按照文档进行配置即可。
![](https://img.ryoma.top/WordPress/stella.blog.png)

### 自定义配置
如果不满足WordPress默认的内容，可以通过后台修改，或者直接修改源代码