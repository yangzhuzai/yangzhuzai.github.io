+++
title = '0x048 Loly'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCEde1e61c7e57f09e815a0032c0fd11d67image.png)

![](/vulnhub_img/WEBRESOURCE7b53bb254eede2f1784e84384fdf8712image.png)

# 渗透

![](/vulnhub_img/WEBRESOURCEc324ca6592b23308ee422700e12de567image.png)

![](/vulnhub_img/WEBRESOURCEa9e28aad8a73bd4a43a09e041ad35a13image.png)

改一下hosts文件：

![](/vulnhub_img/WEBRESOURCE221fda2b20a603f6d2d3ba7f4c22d8acimage.png)

wpscan扫描

插件

![](/vulnhub_img/WEBRESOURCEaa966ed8ab637209797eb0e8ee19f739image.png)

用户

![](/vulnhub_img/WEBRESOURCE8dcf8b474e296f3bc5948b612fdcac19image.png)

直接使用rockyou爆破得到密码：

![](/vulnhub_img/WEBRESOURCE5de246a980baef1a1b26d104c7d5f97dimage.png)

Username: loly, Password: fernando

![](/vulnhub_img/WEBRESOURCE8d9b91ad2b3508092f2ad26524fea912image.png)

后台上传webshell，找了老半天了：

![](/vulnhub_img/WEBRESOURCE865fd551d6713d868094379641d808b9image.png)

路径：

![](/vulnhub_img/WEBRESOURCE2fc2369393c45aad527cabafcf38d948image.png)

![](/vulnhub_img/WEBRESOURCE9b6a05899fa1153cf3f5a4ec96673bc2image.png)

# 提权

提权到loly失败：

![](/vulnhub_img/WEBRESOURCE9f02128b5d446631dcb1e826333b02b0image.png)

通过查找网站的数据库密码，切换成功：

lolyisabeautifulgirl

![](/vulnhub_img/WEBRESOURCE3c52bc16c4a066df46d18c84d41cc3c7image.png)

本来是不想用内核提权的，找了半天好像没其他方式了：

CVE-2017-16995

![](/vulnhub_img/WEBRESOURCEed0a4d73351cb75a592ac4e1e4cf0f5dimage.png)