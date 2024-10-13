+++
title = '0x050 DevRandom CTF 1.1'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE2ac6cee533010f128468f11e3f047d60image.png)

![](/vulnhub_img/WEBRESOURCEee81955c2be115c056ffb645efbaa2f7image.png)

![](/vulnhub_img/WEBRESOURCEa6f9ba35d3d592fb76e7d213ad6dfc11image.png)

# 渗透

80端口看看吧：

![](/vulnhub_img/WEBRESOURCE0accb3df785170efd2f1361544b6ce22image.png)

![](/vulnhub_img/WEBRESOURCE64fb3a61efdadf803006f933c9a107f5image.png)

对发现的目录挨个访问，发现可能利用的目标：

![](/vulnhub_img/WEBRESOURCE32b097cc210fb1209978ee56e2d0d666image.png)

通过日志写进去后，读取出现了错误：

![](/vulnhub_img/WEBRESOURCE81592191b1ef6f48cf356dd87a243695截图.png)

发现一个API，试了路径，暂时不知道什么意思：

4395874598yt3r9iy98r7r90t87treterrrrr

![](/vulnhub_img/WEBRESOURCEc21072bdcee72d3f094679eb5c5faadf截图.png)

还有一个：

![](/vulnhub_img/WEBRESOURCEcaef03ec16a22d30f5ffb75729f45ea6截图.png)

发现了文件包含：

../../../../../../../etc/passwd

/var/www/html/

![](/vulnhub_img/WEBRESOURCEb8fe1a6296bc4e6f0c458dc5038426f4截图.png)

呐其实我就开始琢磨包含刚刚写的webshell了：

试了半天，包含不上去，暂时不清楚是什么原因，待会儿看看代码：

![](/vulnhub_img/WEBRESOURCEa41c10150b2d4011fe0f478f00990458截图.png)

下策，ssh 爆破:

得到账号密码，时间巨长，建议挂着

trevor：qwertyuiop[]

![](/vulnhub_img/WEBRESOURCEd8a43cc8fe3cad9c156dba9744ddfb7b截图.png)

# 提权

sudo 提权

![](/vulnhub_img/WEBRESOURCEc4b3afe566408dcb9b2445f33c9cd84d截图.png)

![](/vulnhub_img/WEBRESOURCEc7d63f8f9599b785e86fc65c203bd5c1截图.png)

这个日志确实是可以getshell的，但是日志数量多了后面可能就会出现问题，可能是闭合的问题

![](/vulnhub_img/WEBRESOURCEea81bbe41588835b92673a1b1aa822f7截图.png)