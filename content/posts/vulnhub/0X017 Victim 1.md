+++
title = '0x017 Victim 1'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



同样要修改IP地址：

![](/vulnhub_img/WEBRESOURCEb12de8fd0af939ee2d9da7a68a39e8d5截图.png)

# 端口扫描

![](/vulnhub_img/WEBRESOURCE3108ff06e2df581c14da45fcde7e0d11截图.png)

![](/vulnhub_img/WEBRESOURCEe0a2852908fe6b35bf9965c4f6ff2250截图.png)

# 渗透

80端口：

![](/vulnhub_img/WEBRESOURCEc4c7e213ef2eefb9febc89f71c9a564a截图.png)

有joomla?，发现疑似账户名or密码，h@ck3rz!

![](/vulnhub_img/WEBRESOURCE05d28b1ed75c579999e1f006f5a051b9截图.png)

![](/vulnhub_img/WEBRESOURCE787e9829b0e94f812afed574d08835ce截图.png)

access.txt，但是看起来没什么用：

![](/vulnhub_img/WEBRESOURCE32d255f6f924886490262bf49a047a6e截图.png)

8080端口：

file.php，看起是想文件包含？，但是没有php解析环境

![](/vulnhub_img/WEBRESOURCE1deab00812c8a7d9fc7c908c7d6f9bc1截图.png)

8999端口，目录遍历，又是wordpress?，但是好像没有php解析环境：

![](/vulnhub_img/WEBRESOURCEe9b5d43e5c4bb786dc4a800759a0fe93截图.png)

9000端口：

![](/vulnhub_img/WEBRESOURCE6c6fe22aa4f6390ba8ebc08e061051ca截图.png)

![](/vulnhub_img/WEBRESOURCEc75173120084a152588e61583d0b2672截图.png)

后面还是看了WP，我是发现那个cap文件了的，也看到了802.11协议，但是没细想要去破解无线账号密码：

aircrack-ng -w /usr/share/wordlists/rockyou.txt WPA-01.cap

SSID为账号：

dlink

![](/vulnhub_img/WEBRESOURCE5c19b2950fa79ba144636f318f6230fc截图.png)

p4ssword

![](/vulnhub_img/WEBRESOURCE081e4805e0cb9a15764d0cfe823c162c截图.png)

# 提权

非常简单

![](/vulnhub_img/WEBRESOURCE4d2dcd47522ad0c75ff05ff3ec43a48b截图.png)