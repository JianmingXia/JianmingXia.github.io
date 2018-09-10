---
title: 云服务器体验及简单使用Nginx部署
date: 2017/6/6 12:00:00
tags:
  - 云服务器
  - Nginx
categories: 
  - [Nginx]
  - [博客相关]
---

## 成果
手上没有合适的页面，贴个公益性的网页：
![结果页](https://img.ryoma.top/%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%BD%93%E9%AA%8C%E5%8F%8A%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8Nginx%E9%83%A8%E7%BD%B2/404.png)

## 云服务器
本来云服务器应该是跟域名一起弄的，但是昨天弄博客时间比较晚，到了晚上发现腾讯云云服务器的体验名额已经没了（后面我找了多家云服务器产商，发现，他们真的都很贵= =，据说亚马逊有一年的免费，有时间去看看），所以这件事就放到了今天。

<!-- more -->

下午一点多申请的云服务器体验，很幸运还有，但是中间腾讯云页面出了点问题：
- 不清楚有没有申请成功——尝试了继续申请，提示“已申请过了”，最后在控制台找到了服务器的信息；
- 密码不知道多少——幸好在页面上发现重置密码的选项，果断重置（结果发现腾讯云在我申请成功后已经给我发到邮箱了。。。）
至此，云服务器可以使用账号密码登录，当然这个时候在网页上是没办法正常访问的。

云服务器这块不清楚的可以参考[这个](https://segmentfault.com/a/1190000008830593)，内容写的很详细。
- 其中上传下载文件，我比较习惯使用rz sz命令，服务器没有安装自己安装就好了。
{% codeblock %}
yum install lrzsz
{% endcodeblock %}
- 之前注册了一个域名，这个时候我将test.xxx.xxx解析到云服务器上了。

## 安装Nginx

**安装**：
{% codeblock %}
yum install nginx
{% endcodeblock %}
**查看nginx状态**：
{% codeblock %}
service nginx status
{% endcodeblock %}
![Nginx状态](https://img.ryoma.top/%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%BD%93%E9%AA%8C%E5%8F%8A%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8Nginx%E9%83%A8%E7%BD%B2/nginx_status_inactive.png)
继续**验证nginx配置**：
{% codeblock %}
nginx -t
{% endcodeblock %}
![Nginx配置](https://img.ryoma.top/%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%BD%93%E9%AA%8C%E5%8F%8A%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8Nginx%E9%83%A8%E7%BD%B2/nginx_t.png)

- 可以看到nginx配置没问题，而且能够看到nginx的安装位置。

可以去nginx安装目录下查看
**启动nginx**：
{% codeblock %}
service nginx start
{% endcodeblock %}
**查看nginx运行情况**：
{% codeblock %}
ps aux|grep nginx
{% endcodeblock %}
![查看Nginx](https://img.ryoma.top/%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%BD%93%E9%AA%8C%E5%8F%8A%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8Nginx%E9%83%A8%E7%BD%B2/nginx_start.png)

**访问**（没有解析IP的话，使用域名访问也一样。域名解析可能需要几分钟，别着急）：
![Nginx配置成功](https://img.ryoma.top/%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%BD%93%E9%AA%8C%E5%8F%8A%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8Nginx%E9%83%A8%E7%BD%B2/nginx.png)

## nginx配置
大家可以看到nginx目录下有**nginx.conf**文件，其中有个server就是默认的server。
![Nginx.conf](https://img.ryoma.top/%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%BD%93%E9%AA%8C%E5%8F%8A%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8Nginx%E9%83%A8%E7%BD%B2/default_server.png)
如果我们需要在这台服务器上部署其它服务怎么办？
同样，加server配置就可以了，在这里我没有在**nginx.conf**中继续添加，而是在server后加上了```include vhost/*;```，这样有个好处就是我们可以为用单个文件为每个域名来配置，更加清晰，也方便了后续的修改。
![添加vhost](https://img.ryoma.top/%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%BD%93%E9%AA%8C%E5%8F%8A%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8Nginx%E9%83%A8%E7%BD%B2/nginx_add_vhost.png)
在nginx目录下添加**vhost文件夹**，添加**test.ryoma.top.conf**文件，编辑文件。
![test.ryoma.top.conf](https://img.ryoma.top/%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%BD%93%E9%AA%8C%E5%8F%8A%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8Nginx%E9%83%A8%E7%BD%B2/test.ryoma.top.conf.png)

这个时候再重新加载nginx配置，使用命令```service nginx reload```。
在server中指定的root目录添加文件。
![上传index.html](https://img.ryoma.top/%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%BD%93%E9%AA%8C%E5%8F%8A%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8Nginx%E9%83%A8%E7%BD%B2/test.ryoma.top.png)
再次访问，就是之前的成果了。
![完成](https://img.ryoma.top/%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%BD%93%E9%AA%8C%E5%8F%8A%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8Nginx%E9%83%A8%E7%BD%B2/404.png)
