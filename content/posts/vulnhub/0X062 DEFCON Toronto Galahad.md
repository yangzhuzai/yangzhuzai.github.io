+++
ShowToc = true
TocOpen = true
title = '0x062 DEFCON Toronto Galahad'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCEe605071498edbb1bc00d96ed7a95b46bimage.png)

![](/vulnhub_img/WEBRESOURCE6cab4bd04da57ff2b2998519d55bc3f6image.png)

![](/vulnhub_img/WEBRESOURCEfed7689ccc7ddf553473c9dfc7c8b60cimage.png)

# 渗透测试

80端口

打开网页，发现一串二进制字符：

![](/vulnhub_img/WEBRESOURCE9aac4a6b445e87397df8a1ca5045bf48image.png)

解开后是这一串内容：

![](/vulnhub_img/WEBRESOURCEf9badb0256196b71484da9aba2948844image.png)

![](/vulnhub_img/WEBRESOURCE902c72824e62e7f84c1a922e0a7cae4cimage.png)

发现s.txt:

![](/vulnhub_img/WEBRESOURCEc9308d55abbf6717142358fcaf9e0438image.png)

解密出来：

![](/vulnhub_img/WEBRESOURCEda20193037a7fb2ac537179ffcf289c4image.png)

后面没思路了，我还是不太喜欢这种CTF式的靶机，我满脑子只想着拿shell，所以，在查阅大量资料后，我只尝试进行拿到root：

经过一段骚操作：

ssh账号nitro

密码：zeus

巨邪门儿，不喜欢ctf:

**井字棋先手，然后无脑下四角就能赢：**

![](/vulnhub_img/WEBRESOURCEf17e9fb292d901ac74b90508b43b505cimage.png)

# 提权

疑似计划任务：

![](/vulnhub_img/WEBRESOURCE569fdfd672ff73d173459942fefba7aeimage.png)

先秒了再说

![](/vulnhub_img/WEBRESOURCE94c28186996a1a1de12c4c273b2314acimage.png)

再来回头看这个计划任务，整个目录都有写入权限：

![](/vulnhub_img/WEBRESOURCE0f3e1f0a5fbb06da59cde274b266cb95image.png)

![](/vulnhub_img/WEBRESOURCE870cc7ab85a3443464183731d0ec884bimage.png)

![](/vulnhub_img/WEBRESOURCE44fc2ed10edbb79e89f21b3add1814e7image.png)

不过这个15分钟确实缺德：

![](/vulnhub_img/WEBRESOURCEcf61fd21b872ac55e80e952793ac3725image.png)