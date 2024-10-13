+++
title = '0x068 INFOSEC PREP OSCP'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCEc7090209b1b8c753e8ded2dbfd6ef70eimage.png)

![](/vulnhub_img/WEBRESOURCEa9be547cd8d23dbb3c3823fde68d067aimage.png)

![](/vulnhub_img/WEBRESOURCE66a10c428b6b09fe973048f265c272aeimage.png)

# 渗透

![](/vulnhub_img/WEBRESOURCEc71dfdb60f6f8d38b96ffee26e0e3be3image.png)

发现密钥：

![](/vulnhub_img/WEBRESOURCE17bc4a41f578f0b1acbb16aeb58aef1cimage.png)

![](/vulnhub_img/WEBRESOURCE656d74894b1a898df3ee519aea038026image.png)

![](/vulnhub_img/WEBRESOURCE0c69c7bd92844295c3723e96e2e22cb2image.png)

但是没有用户名：

wpsacn

![](/vulnhub_img/WEBRESOURCEecfb5e7df0381492b450397fe5922596image.png)

![](/vulnhub_img/WEBRESOURCE47349bcb65d263c19ac29637c529693cimage.png)

通过在主页中翻找，可以知道目前的用户名是oscp：

![](/vulnhub_img/WEBRESOURCE4d35e04a70c802f25acebca567b1b8f9image.png)

使用密钥登录：

![](/vulnhub_img/WEBRESOURCE5251908451867fbb7ab801276da8461fimage.png)

# 提权

SUID提权

![](/vulnhub_img/WEBRESOURCE38bb0502d18f6985b918c05925889725image.png)

![](/vulnhub_img/WEBRESOURCE299ccef541432d2578ded2885210307cimage.png)