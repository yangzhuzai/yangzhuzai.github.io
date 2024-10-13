+++
title = '0x049 Geisha'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE5860bad91663e99cb75a73250cbd9ddeimage.png)

![](/vulnhub_img/WEBRESOURCEca0ecaa1b593036f264e606d18221da2image.png)

# 渗透

所有端口都看了一下，几乎都是这个样子，还怪好看的嘞，但是下下来看了一下，没有发现什么隐藏信息：

![](/vulnhub_img/WEBRESOURCEcf8e03c3798527180fff0c8870acc214image.png)

21端口：

![](/vulnhub_img/WEBRESOURCE487a3c6dc3e1865339a426200dc02287image.png)

这样的话，就只剩一个22端口了，尝试爆破，由于啥信息都没有，就只能试试geisha了：

大概跑了10多分钟，爆破出了密码为letmein

![](/vulnhub_img/WEBRESOURCE45b536b1f16ee673da74c6911c8a31e1image.png)

# 提权

![](/vulnhub_img/WEBRESOURCE10b59b205779b708a9081d153a3bec3bimage.png)

查看root密码：

![](/vulnhub_img/WEBRESOURCE71a02329d5b92176037fa5f55dee1944image.png)

爆破hash:

跑了20分钟了，都没跑成功：

![](/vulnhub_img/WEBRESOURCE531f8eb5ea361cad7b07d629643e2ee2image.png)

忘了能读密钥啊：

![](/vulnhub_img/WEBRESOURCEdce7b64c708086c6cb8be8ad654659b6image.png)

![](/vulnhub_img/WEBRESOURCE2d19f2a8cb2e5cb094cc083d56f8988cimage.png)