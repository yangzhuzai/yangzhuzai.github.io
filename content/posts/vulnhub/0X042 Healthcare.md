+++
ShowToc = true
TocOpen = true
title = '0x042 Healthcare'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE721e51dce6cbb51dd77f0103d630b374截图.png)

![](/vulnhub_img/WEBRESOURCE1f8a325a62d5ea53cf7d732b00a6ea0a截图.png)

![](/vulnhub_img/WEBRESOURCEcb5fed76c836fa3fb24bc8775283f861截图.png)

# 渗透

发现root目录，同时发现疑似提示，htdig to index our doc-tree。

![](/vulnhub_img/WEBRESOURCEe5bee896284035ca7d5dfa57d08d5eeb截图.png)

收集到用户名：mailto

![](/vulnhub_img/WEBRESOURCE2f81e86127e92461a9368572c79122e2截图.png)

去收集了一下htdig是什么，然后没发现有用的利用方式：

![](/vulnhub_img/WEBRESOURCEc715cb48f261bce244ad2bc8ff1e19bb截图.png)

完犊子，

/openemr目录：

![](/vulnhub_img/WEBRESOURCE2dc9e93042d44e8c1611470bfd44be00截图.png)

![](/vulnhub_img/WEBRESOURCE669763ecc9aada6e60dcb92515013bdb截图.png)

**已知漏洞**

![](/vulnhub_img/WEBRESOURCEe99c9babf39ad167769ebd33cde84b3a截图.png)

修改url:

![](/vulnhub_img/WEBRESOURCE7372197d2030bd3d5099ef705dd492bc截图.png)

admin:3863efef9ee2bfbc51ecdca359c6302bed1389e8

medical:ab24aed5a7c4ad45615cd7e0da816eea39e4895d

![](/vulnhub_img/WEBRESOURCE9438afb5408cf36f1f6d824423833aaa截图.png)

ackbar

medical

![](/vulnhub_img/WEBRESOURCE6433f10c8f0b2588dfc5e4748f932a6d截图.png)

在这个地方修改文件即可getshell:

![](/vulnhub_img/WEBRESOURCE3336070e4d90be577f491a170b6be89a截图.png)

![](/vulnhub_img/WEBRESOURCEb1289f6b9292b8f0a3ba13d0f7e4a6f1截图.png)

# 提权

有点懒了，看了一下提示：/usr/bin/healthcheck：

strings查看一下：

就可以琢磨一下该如何提权了，执行了ifconfig、fdisk、du等命令

![](/vulnhub_img/WEBRESOURCEfab4d42f7c9735cfd7bf58067508686d截图.png)

环境变量劫持：

![](/vulnhub_img/WEBRESOURCE8e614d21cd198ebe1ed8ee176dcb1a64截图.png)