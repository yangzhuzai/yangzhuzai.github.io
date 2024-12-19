+++
ShowToc = true
TocOpen = true
title = '0x056 BTRSys v2.1'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE8653e5494ecd348115306a8aee5136c4image.png)

![](/vulnhub_img/WEBRESOURCE0f9af20f133963c0637d25e5b4a2615aimage.png)

![](/vulnhub_img/WEBRESOURCE7065f2574e44403268b89c8dfdf0371eimage.png)

# 渗透

21端口

啥也没有

![](/vulnhub_img/WEBRESOURCE4f09c87a1e4fbc616038b85b165cec27image.png)

这靶机打得san值狂掉，那个图片看得很难受，主页里面两个图片，1.gif，2.gif难受，别看，很难受

疑似数据库名称：

Lepton

![](/vulnhub_img/WEBRESOURCE6c74345a23a3aacc00ce044c33ac7b52image.png)

有一个叫LEPTON的cms:

![](/vulnhub_img/WEBRESOURCEc165e6fa3c7ec90a8aaeb6ccca6d5a45image.png)

还有wordpress:

![](/vulnhub_img/WEBRESOURCEcf5e4096223bb26f91f3888cceb485f8image.png)

wpsacn

插件：

![](/vulnhub_img/WEBRESOURCEb403993834fa0222bf4f90ba994e1e24image.png)

用户

![](/vulnhub_img/WEBRESOURCEf1084c55838b01985d5fa44e010c3373image.png)

直接爆破

![](/vulnhub_img/WEBRESOURCE598d8be91bbbf69ffc35a6a2ac842d19image.png)

后台getshgell:

http://192.168.5.101/wordpress/wp-content/themes/twentyfourteen/404.php

![](/vulnhub_img/WEBRESOURCEf1795c64f2261ff15d9ab7bb222beab7image.png)

# 提权

![](/vulnhub_img/WEBRESOURCE80afc6ca0d2b22f75f954eff9bd177afimage.png)

home目录下有用户btrisk，可以尝试登录数据库看看：

![](/vulnhub_img/WEBRESOURCE9ca045245b72930d0efd79dae4f36277image.png)

![](/vulnhub_img/WEBRESOURCEe201c14d1b4a093061f099b0d344de15image.png)

![](/vulnhub_img/WEBRESOURCEf92edf4c6eeea7aecc3ee3ce32581a62image.png)