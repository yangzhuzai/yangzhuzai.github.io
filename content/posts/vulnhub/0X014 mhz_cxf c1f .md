+++
ShowToc = true
TocOpen = true
title = '0x014 mhz_cxf c1f '
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



该靶机和靶机4问题一样，需要修改网卡配置文件才能获取到IP地址，修改方式一样

# 端口扫描

![](/vulnhub_img/WEBRESOURCEf1d943a5f44d6ecd2daedc184efd56b2截图.png)

# web渗透

目录扫描：

![](/vulnhub_img/WEBRESOURCEbe31d832ad093168a0140649fa3c24d6截图.png)

![](/vulnhub_img/WEBRESOURCE7f713e83c70b30fa2d4cfc48a156f137截图.png)

first_stage:flagitifyoucan1234

尝试登录，说实话有点意外，这么简单？

![](/vulnhub_img/WEBRESOURCE8925d14b14e6b644806a65a2207dc24d截图.png)

# 提权

看了一圈，没发现提权点，看home目录下还有一个用户，可能需要再切换一些用户?

后面卡住没办法了，看了WP，我大概也猜到了是家目录下的图片有隐藏信息，但是这一块儿的知识确实较为欠缺。这里隐藏图片信息的获取方式：

steghide工具：

![](/vulnhub_img/WEBRESOURCE2f9fc7625f157171892c98f2ebbd5849截图.png)

提取

```
sudo steghide extract -sf xxx
```

![](/vulnhub_img/WEBRESOURCEcfa5c5c90d855536d043708ab9fd1fd5截图.png)

![](/vulnhub_img/WEBRESOURCE5f767ee1e5e6baa794ece946bdc70adb截图.png)

![](/vulnhub_img/WEBRESOURCE467fab89458d0b4d9092ba6e22cf9150截图.png)

# 总结

在知道如何提取图片隐藏信息的情况下，该靶机几乎就没有难度了