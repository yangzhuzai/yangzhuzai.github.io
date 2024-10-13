+++
title = '0x026 eLection 1'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE3a7a3707ff0480a0948261e2a5ae96a7截图.png)

![](/vulnhub_img/WEBRESOURCEacc157c8b770af83a7fdfa7b79a19f58截图.png)

# 渗透

东西不多，就那么几个，重点看起来要么在phpmyadmin要么在那个election中：

![](/vulnhub_img/WEBRESOURCEf71c52abf5b7ef6f0b312fddd9d604e7截图.png)

翻查了一圈，好像没看到什么明显的漏洞，邪门儿，冷静后再扫一次，打靶就是这样，很多时候是被自己困住的：

![](/vulnhub_img/WEBRESOURCE9b3ec4f6acd4c92c952745dbdd66fa86截图.png)

得到账号密码：love: P@$$w0rd@123

![](/vulnhub_img/WEBRESOURCE2a1d6c9732cb45df21541d41003b1e6e截图.png)

# 提权

![](/vulnhub_img/WEBRESOURCEa7e69367b98b8a5723777b4fb0b5dea0截图.png)

有GCC，考虑一下内核提权：

没什么好说的，秒了

![](/vulnhub_img/WEBRESOURCEb893305b7ff9173f72f19dff2a2c52b7截图.png)