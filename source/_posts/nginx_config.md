---
title: nginx 配置
tags:
  - linux
  - nginx
  - config
categories: nginx
---

## 题外话
最近参与的项目做了较大的更新，原来通用的nginx配置需要做调整，所以对nginx配置这块看的也比较多，在此做个整理

## location 配置
### location 匹配规则
- =：普通字符精确匹配
- ^~：uri以字符串开头，一般用来匹配目录（空格可用20%匹配）
- ~：区分大小写的正则匹配（!~区分大小写不匹配）
- ~\*：不区分大小写的正则匹配（!~*不区分大小写不匹配）
- /：通用匹配，任何请求都会匹配到
<!-- more -->

### location 匹配优先级
- =：优先最高
- ^~: 第二优先级
- ~等：第三优先级，按顺序匹配
- /：最后匹配
当匹配成功后，会终止匹配

## root 和 alias 的区别
### alias
- 目录名后面一定要加"/"
- 作用域: location
```
    location ^~ /alias/ {
        alias /xjm/ali2/;
    }
```
访问 **http://ali2.ryoma.top/alias/img/11.png** ，会访问至   **/xjm/ali2/** 下的 **img/11.png** 文件——这里可以看出，会将uri未匹配的部分加到alias后面

### root
- root的限制就没有那么多了

```
    location ^~ /img/ {
        root /xjm/ali2;
    }
```
访问**http://ali2.ryoma.top/img/11.png**，会访问至**/xjm/ali2**
下的 **/img/11.png**文件————访问的是root路径 + uri

## try_files讨论
### try_files
- 语法: try_files file1 [file2 … filen] fallback 
- 作用域: location
```
    location ^~ /tryfiles/ {
        alias /xjm/ali2/;
        try_files $uri $uri/11.png /11.png =404;
    }
```
访问**http://ali2.ryoma.top/tryfiles/img/** 或者 **http://ali2.ryoma.top/tryfiles/img/11.png** 找到 **/xjm/ali2/** 中的 **img/11.png**文件

```
    location ^~ /tryfiles2/ {
        root /xjm/ali2/;
        try_files $uri $uri/11.png /11.png =404;
    }
```
访问*http://ali2.ryoma.top/tryfiles2/** 或者 **http://ali2.ryoma.top/tryfiles2/11.png** 找到 **/xjm/ali2/** 中的 **tryfiles2/11.png**文件

try_files 中的=404需要注意，当没有找到所需资源时，会返回对应的状态码