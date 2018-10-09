---
title: windows & linux 小功能List
date: 2017/11/11 12:00:00
tags:
  - windows
  - linux
categories: 技术杂集
---

## windows端口占用
  - 如查看端口4000：netstat -aon|findstr "4000"
  - 查看对应进程号对应的程序：tasklist|findstr "2696"
    - 注意：英文双引号，使用hexo生成后，变成了中文的了
  ![](https://img.ryoma.top/MiniFunctionList/8.png)
  - 杀死对应进程：taskkill /f /t /im FoxitProtect.exe
  ![](https://img.ryoma.top/MiniFunctionList/9.png)
  - 如果该进程是子进程，可切换至“任务管理器”结束任务
  
## linux 一次创建多个文件
  - touch file1 file2 file3
  
  ![](https://img.ryoma.top/MiniFunctionList/1.png)
<!-- more -->

## 删除文件
### 删除除某个文件外的所有文件
  - 使用 rm -rf !(file1)
  如列表中有file1 file2 file3三个文件，保留file1
    ![](https://img.ryoma.top/MiniFunctionList/2.png)
  - 可能遇到的问题：
  ```
  -bash: !: event not found
  ```
  这个时候应该是模式匹配操作符没有开启，使用```shopt -p```
  ![](https://img.ryoma.top/MiniFunctionList/3.png)
  这个时候可以发现```shopt -u extglob```,extglob处于关闭状态，使用```shopt -s extglob```开启，即可

### 保留file1 file2
  - rm -rf !(file1|file2)
  ![](https://img.ryoma.top/MiniFunctionList/4.png)

### 删除n天之前的log
- find ${DEST_LOG_PATH} -mtime +${MTIME} -name "*.log" -exec rm -rf {} \;
- 说明：
  - DEST_LOG_PATH：是待删除的目录
  - [mtime n](#find-mtime-n): 按照文件的更改时间来找文件，n为整数。
  - name: 表示待删除的文件
  - exec：固定写法；
  - rm -rf：强制删除文件，包括目录；
  - {} \; ：固定写法，一对大括号+空格+\+; 
- 实际使用：find /quqi/log/updown.server/ -mtime +1 -name "*.log" -exec rm -rf {} \;
![](https://img.ryoma.top/MiniFunctionList/10.png)



## 统计文件个数
  说明：
  - ls -lR
  长列表输出该目录下文件信息，R代表子目录（同理，使用ll也ok）
  - grep "^-"
  使用grep过滤。只保留一般文件，如果只保留目录就是 ^d
  - wc -l
  统计输出信息的行数，由于一行信息对应一个文件，即文件的个数。

### 统计文件夹下文件的个数
  - ls -l |grep "^-"|wc -l
  ![](https://img.ryoma.top/MiniFunctionList/5.png)

### 统计文件夹下文件夹的个数
  - ls -l |grep "^d"|wc -l
  ![](https://img.ryoma.top/MiniFunctionList/6.png)

### 统计文件夹下文件夹的个数，包括子目录的
  - ls -lR|grep "^-"|wc -l
  当前目录只有文件file1 file2，在目录1下创建一个文件，这是查看有3个
  ![](https://img.ryoma.top/MiniFunctionList/7.png)

## ln 软链接
- ln -s /test/index.html index.html
- ln -s /test/index.html /test/aaaaa/xxxxxxxxxx
第一条：在当前目录建立文件```index.html```，指向**/test/index.html** <br>
第二条：在**/test/aaaaa/**目录下（目录需要存在），建立文件```xxxxxxxxxx```，指向**/test/index.html**
 ![](https://img.ryoma.top/MiniFunctionList/11.png)

- 使用场景：
  - 代码发布到线上，线上适用于nginx的结构与代码打包的结构有所不同，使用软连接

## kill多个进程（知道进程名称）
如kill包含`Server`内容的进程号

### 解析
- ps -ef|grep Server(列出所有Server详情)
 ![](https://img.ryoma.top/MiniFunctionList/12.png)
- ps -ef|grep Server | grep -v grep(不显示grep对应的进程号)
 ![](https://img.ryoma.top/MiniFunctionList/13.png)
- ps -ef|grep Server | grep -v grep | awk '{print $2}'(使用awk输出进程号，$2表示第2列)
 ![](https://img.ryoma.top/MiniFunctionList/14.png)

- kill -9 $(ps -ef|grep Server | grep -v grep | awk '{print $2}')

## 其它
### find -mtime n
n表示文件更改时间距离为n天， -n表示文件更改时间距离在n天以内，+n表示文件更改时间距离在n天以前。（+ || -这块可能会觉得比较奇怪，但是可以试试）
例如：
-mtime 0 表示文件修改时间距离当前为0天的文件，即距离当前时间不到1天（24小时）以内的文件。
-mtime +1 表示文件修改时间为大于1天的文件，即距离当前时间2天（48小时）之外的文件
-mtime -1 表示文件修改时间为小于1天的文件，即距离当前时间1天（24小时）之内的文件