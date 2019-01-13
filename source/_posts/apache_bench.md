---
title: Apache HTTP服务器基准测试工具(ab)
date: 2018/04/03 21:00:00
tags:
  - Apache Benchmark
categories: 技术杂集
---

> ab is a tool for benchmarking your Apache Hypertext Transfer Protocol (HTTP) server. It is designed to give you an impression of how your current Apache installation performs. This especially shows you how many requests per second your Apache installation is capable of serving.

ab是Apache超文本传输协议(HTTP)服务器的基准测试工具，平时可以用于来评测服务的并发性能，个人觉得它是非常简单 & 好用的一个工具。

Note：虽然ab占用的资源不太高，但是如果参数设置的很多，还是有可能对本机器所在的服务造成影响——为了更严谨的测试，不要在部署服务的机器上做测试

<!-- more -->

## 准备工作
### 安装
#### CentOS
* yum install httpd-tools

#### Windows
安装的Apache中bin目录中，就有ab.exe文件

### 参数配置

```plain
Usage: ab [options] [http[s]://]hostname[:port]/path
Options are:
    -n requests     Number of requests to perform
    -c concurrency  Number of multiple requests to make at a time
    -t timelimit    Seconds to max. to spend on benchmarking
                    This implies -n 50000
    -s timeout      Seconds to max. wait for each response
                    Default is 30 seconds
    -b windowsize   Size of TCP send/receive buffer, in bytes
    -B address      Address to bind to when making outgoing connections
    -p postfile     File containing data to POST. Remember also to set -T
    -u putfile      File containing data to PUT. Remember also to set -T
    -T content-type Content-type header to use for POST/PUT data, eg.
                    'application/x-www-form-urlencoded'
                    Default is 'text/plain'
    -v verbosity    How much troubleshooting info to print
    -w              Print out results in HTML tables
    -i              Use HEAD instead of GET
    -x attributes   String to insert as table attributes
    -y attributes   String to insert as tr attributes
    -z attributes   String to insert as td or th attributes
    -C attribute    Add cookie, eg. 'Apache=1234'. (repeatable)
    -H attribute    Add Arbitrary header line, eg. 'Accept-Encoding: gzip'
                    Inserted after all normal header lines. (repeatable)
    -A attribute    Add Basic WWW Authentication, the attributes
                    are a colon separated username and password.
    -P attribute    Add Basic Proxy Authentication, the attributes
                    are a colon separated username and password.
    -X proxy:port   Proxyserver and port number to use
    -V              Print version number and exit
    -k              Use HTTP KeepAlive feature
    -d              Do not show percentiles served table.
    -S              Do not show confidence estimators and warnings.
    -q              Do not show progress when doing more than 150 requests
    -g filename     Output collected data to gnuplot format file.
    -e filename     Output CSV file with percentages served
    -r              Don't exit on socket receive errors.
    -h              Display usage information (this message)
    -Z ciphersuite  Specify SSL/TLS cipher suite (See openssl ciphers)
    -f protocol     Specify SSL/TLS protocol
                    (SSL3, TLS1, TLS1.1, TLS1.2 or ALL)
```

### 关键参数说明
> 个人常用参数

* n：请求总数
* c：并发请求数
* C：设置cookie；可重复
* H：设置header；可重复
* k：是否启用HTTP KeepAlive功能；默认不开启
* v：设置打印信息的详细程度

## 测试
这里使用了并发100，总请求数为1000做了测试：

```plain
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking xxxx (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Completed 600 requests
Completed 700 requests
Completed 800 requests
Completed 900 requests
Completed 1000 requests
Finished 1000 requests


Server Software:        nginx/1.10.3
Server Hostname:        xxxx
Server Port:            443
SSL/TLS Protocol:       TLSv1.2,ECDHE-RSA-AES128-GCM-SHA256,2048,128

Document Path:          xxxx
Document Length:        89 bytes

Concurrency Level:      100 (-c 参数)
Time taken for tests:   1.044 seconds (1000个测试请求的总时间)
Complete requests:      1000
Failed requests:        0 (失败的请求数)
Write errors:           0
Total transferred:      411000 bytes (1000个请求传输的数据)
HTML transferred:       89000 bytes
Requests per second:    958.13 [#/sec] (mean) (每秒执行的请求数)
Time per request:       104.370 [ms] (mean) (每组并发请求的时间，我使用的是100)
Time per request:       1.044 [ms] (mean, across all concurrent requests) (所有请求的平均时间)
Transfer rate:          384.56 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       18   67  15.7     67     100
Processing:     7   29  13.0     27      72
Waiting:        7   20   8.2     18      67
Total:         33   96  13.7     94     152
(每个请求经历的：连接、发送数据、接收数据花费的时间——最小、平均数、标准差、中位数及最大值)

Percentage of the requests served within a certain time (ms)
  50%     94
  66%     99
  75%    104
  80%    107
  90%    114
  95%    120
  98%    132
  99%    133
 100%    152 (longest request)
```

## 资料
* [https://httpd.apache.org/docs/2.4/programs/ab.html](https://httpd.apache.org/docs/2.4/programs/ab.html)
* [https://blog.getpolymorph.com/7-tips-for-heavy-load-testing-with-apache-bench-b1127916b7b6](https://blog.getpolymorph.com/7-tips-for-heavy-load-testing-with-apache-bench-b1127916b7b6)

