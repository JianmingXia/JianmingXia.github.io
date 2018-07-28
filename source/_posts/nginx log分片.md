---
title: nginx log分片
tags:
  - linux
  - nginx
  - nginx log
categories: nginx
---

## Nginx log分片

### 创建cut_log.sh脚本文件

```
#!/bin/bash

LOG_PATH=$(cd `dirname $0`; pwd)

YESTERDAY=$(date -d yesterday "+%Y-%m-%d")

PID=/usr/local/nginx/logs/nginx.pid

mv ${LOG_PATH}/access.log ${LOG_PATH}/access-${YESTERDAY}.log
mv ${LOG_PATH}/error.log ${LOG_PATH}/error-${YESTERDAY}.log

kill -USR1 `cat ${PID}`
```
<!-- more -->

- 此脚本用于自动分割Nginx的日志，包括access.log和error.log
- 说明：
	- LOG_PATH: nginx文件所在目录（cut_log.sh脚本也在当前目录）
	- YESTERDAY: 昨天的日期，用于log重命名
	- PID：pid文件路径
	- kill -USR1 `cat ${PID}`： 向nginx主进程发送USR1信号，会从配置文件中读取日志文件名称，重新打开日志文件

### 给cut_log.sh脚本赋予权限
- chmod 777 cut_log.sh

### 设置脚本在每晚12点整执行
```
crontab -e
00 00 * * * /bin/bash /...../创建cut_log.sh
```

## 升级cut_log.sh
### cut_log.sh可传入的参数
	- nginx路径：自定义Nginx路径，否则得将.sh脚本放在logs目录中
	- 存储log的日志：指定存储log的位置，默认在/.../logs目录
	- pid名称：默认值nginx.pid，logs下pid的名称（如果有pid文件不在logs目录下，需要继续添加参数）
	
### 使用方法
```
00 00 * * * /quqi/run/cut.log/cut_log.sh /usr/local/nginx/logs /quqi/log/nginx_log/nginx &
00 00 * * * /quqi/run/cut.log/cut_log.sh /usr/local/nginx_center/logs /quqi/log/nginx_log/nginx_center nginx_center.pid &
```

### 新的cut_log.sh

```
	#!/bin/bash

	PID_NAME="nginx.pid"
	if [ $# -lt 1 ];then
	NGINX_LOG_PATH=$(cd `dirname $0`; pwd)
	DEST_LOG_PATH=${NGINX_LOG_PATH}
	elif [ $# -eq 1 ]; then
	NGINX_LOG_PATH=$1
	DEST_LOG_PATH=${NGINX_LOG_PATH}
	elif [ $# -eq 2 ]; then
	NGINX_LOG_PATH=$1
	DEST_LOG_PATH=$2
	else
	NGINX_LOG_PATH=$1
	DEST_LOG_PATH=$2
	PID_NAME=$3
	fi

	PID=${NGINX_LOG_PATH}"/"${PID_NAME}
	YESTERDAY=$(date -d yesterday "+%Y-%m-%d")

  /bin/mkdir -p $DEST_LOG_PATH 

	mv ${NGINX_LOG_PATH}/access.log ${DEST_LOG_PATH}/access-${YESTERDAY}.log
	mv ${NGINX_LOG_PATH}/error.log ${DEST_LOG_PATH}/error-${YESTERDAY}.log

	kill -USR1 `cat ${PID}`

```

### 即时使用.sh
![](https://img.ryoma.top/NginxLogCut/1.png)
- nginx log被移至目标目录
- nginx logs目录中，log重新开始

### 更多
- 可以搭配清理日志一起使用，可以避免目标目录文件越来越多