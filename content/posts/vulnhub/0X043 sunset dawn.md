+++
title = '0x043 sunset dawn'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



要改网络环境

# 端口扫描

![](/vulnhub_img/WEBRESOURCEfcf39f4415ab4ce05541b96bb66aa5d2截图.png)

![](/vulnhub_img/WEBRESOURCE66050efeac080fa8a5a93f698f44e958截图.png)

![](/vulnhub_img/WEBRESOURCE75256777e579269d7ff0a2113a8a01f1截图.png)

# 渗透

先看了一下smb，发现有未授权，但是没有什么信息：

![](/vulnhub_img/WEBRESOURCE163afbe54b75d8e755334f7d0a3b7a3d截图.png)

![](/vulnhub_img/WEBRESOURCE9b92fda3276da5d2dc45874f5392f64d截图.png)

80端口

扫描发现log目录，说了以下几个目录，可能可以利用？其他东西就没了，

![](/vulnhub_img/WEBRESOURCE358e16ae51a4ac296e188f3c54b43f38截图.png)

再看看日志：

![](/vulnhub_img/WEBRESOURCEb1b6c2a118d1760aded5c8f0bc256e1b截图.png)

那么有没有可能我传的东西会在日志里面，但是很遗憾，我上传的东西并不在，但是发现有其他东西：

![](/vulnhub_img/WEBRESOURCE195726d9a2c0c7cbf28c87d33f6c8714截图.png)

呐就有思路了，上传文件，等反弹shell:

![](/vulnhub_img/WEBRESOURCE70d07aa093b3be950a373c83cb1df2f9截图.png)

# 提权

**CVE-2021-4034秒了**

![](/vulnhub_img/WEBRESOURCE6dc33c940cf5873ebf35289ea88b9797截图.png)

SUID也可以：

![](/vulnhub_img/WEBRESOURCE8e32c8235ef5206f6fe940d18e18537f截图.png)