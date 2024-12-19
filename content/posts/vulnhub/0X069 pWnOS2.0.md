+++
ShowToc = true
TocOpen = true
title = '0x069 pWnOS2.0'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



这台靶机建议本地开nat进行攻击

# 端口扫描

![](/vulnhub_img/WEBRESOURCEe16f35dafc56dcd6ec73510e25b714c3image.png)

![](/vulnhub_img/WEBRESOURCE8b647560bb7e79a98ce54eff1efcf506image.png)

![](/vulnhub_img/WEBRESOURCEb9f0fb7032ce6fe2a4212cdfcdb748d6image.png)

# 渗透

![](/vulnhub_img/WEBRESOURCEe0313ea0b07adb6cbbb8ef62a89ce8baimage.png)

![](/vulnhub_img/WEBRESOURCE3b2a9839877abbcd2cefd3c177db9883image.png)

有sql注入waf?

![](/vulnhub_img/WEBRESOURCE5b39249a3b9f008fe0828ad3f52d270dimage.png)

接着看：

![](/vulnhub_img/WEBRESOURCE6c95faf70a9502ef926960f19cd26602image.png)

![](/vulnhub_img/WEBRESOURCE1c292aa631ef05accfdcb3ebe0ac0415image.png)

可能有漏洞：

![](/vulnhub_img/WEBRESOURCE3a71ca80b73e448d1ff12d0a65051eceimage.png)

下载1191.pl

perl运行可能会出错

```
sudo apt install libswitch-perl
```

![](/vulnhub_img/WEBRESOURCE81147f53fe55cba06c5d1210e4b5b0acimage.png)

![](/vulnhub_img/WEBRESOURCEed3b1c959cb3e1036156779b25e43937image.png)

![](/vulnhub_img/WEBRESOURCE44ef020665741dd9f5a056dae6c2a1c1image.png)

![](/vulnhub_img/WEBRESOURCE5b918c59f7e360ae1a2ddf7d1a621e1fimage.png)

# 提权

/var下面有数据库连接文件：

也是root密码：

![](/vulnhub_img/WEBRESOURCEa29d0668f1532a49947403187e97d86dimage.png)