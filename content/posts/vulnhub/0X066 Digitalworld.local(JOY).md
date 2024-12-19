+++
ShowToc = true
TocOpen = true
title = '0x066 Digitalworld.local(JOY)'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE80c4e4fe12432e3f499bf39b261d42e1image.png)

![](/vulnhub_img/WEBRESOURCE53a54a2b4c947a451ead9b10fbdd65b8image.png)

![](/vulnhub_img/WEBRESOURCEd2b03ba5d32020131ff654b6de24bf89image.png)

![](/vulnhub_img/WEBRESOURCE7d5177d3316456e8f947dc98b3c4171cimage.png)

# 渗透

21端口：

用mget给全部下下来了：

![](/vulnhub_img/WEBRESOURCE13070bcb8d8137c5ebbf321f646d6fc3image.png)

用户名

Patrick

![](/vulnhub_img/WEBRESOURCEd50a7a112b3647342c943a70e82da968image.png)

提示信息，或者账号密码？

![](/vulnhub_img/WEBRESOURCE0d0f952d8937cb2a3607719bead669bbimage.png)

80端口查了一下，是个入侵检测：

并且没有远程漏洞：

![](/vulnhub_img/WEBRESOURCE804ea9cbb6b0c692bc037ba32894f2e0image.png)

后面又没思路了，看了wp，这个地方涉及FTP和telnet的区别：

telnet连接后，用户主机实际成为远程TELNET服务器的一个虚拟终端（或称是哑终端），一切服务完全在远程服务器上执行，但用户决不能从远程服务器中下载或上传文件，或拷贝文件到用户主机中来。

ftp则不同，它是采用客户机/服务器模式，用户能够操作FTP服务器中的目录，上传或下载文件，但用户不能请求服务器执行某个文件。

site cpfr /home/patrick/version_control

site cpto /home/ftp/download/version_control

![](/vulnhub_img/WEBRESOURCEc20426b0c10c5b1d2362ddf7bc8f5716image.png)

![](/vulnhub_img/WEBRESOURCEd3aec73d6a3e5e802b32fa9d7544c0dcimage.png)

这里有几个关键信息：

ProFTPd: 1.3.5

/var/www/tryingharderisjoy

![](/vulnhub_img/WEBRESOURCE417ff73a2a8e1b75391e80ab3b2b2876image.png)

查看版本漏洞信息：

![](/vulnhub_img/WEBRESOURCE4b6c6dc3ec66e3fc494c23d5842905ecimage.png)

![](/vulnhub_img/WEBRESOURCEe0e42a2b8efce86185eaf588706188adimage.png)

![](/vulnhub_img/WEBRESOURCEddf2f6bc47fead95c94284bd995547b4image.png)

![](/vulnhub_img/WEBRESOURCEa37f31593e697025f4f58ef3b6659391image.png)

![](/vulnhub_img/WEBRESOURCEc29831ed97e61ec1102a9cfad4c8e6b9image.png)

![](/vulnhub_img/WEBRESOURCEfd03c46b4272b362d537191db2ae249aimage.png)

# 提权

![](/vulnhub_img/WEBRESOURCE0e3b97cdc68993334d8361050e38dd3aimage.png)

patrick:apollo098765

![](/vulnhub_img/WEBRESOURCE856c095ca978ee038ec7b4126b01e39cimage.png)

经过多次尝试：

![](/vulnhub_img/WEBRESOURCE9d9c70b733fae0bc536c5d8dcf0fcdf6image.png)

别改sudoers文件，要出问题，改passwd文件：

```
root:$1$ZytY2Uoj$1.aU20M/0/xD4Wo1ymW9Z0:0:0:root:/root:/bin/bash
```

![](/vulnhub_img/WEBRESOURCEedd8b32d7fa2920fdc5f21e4637824aaimage.png)

![](/vulnhub_img/WEBRESOURCE07ffa93668cee30492b71c540ab1dcdeimage.png)

![](/vulnhub_img/WEBRESOURCE1fd1205f7dc64892f59042faad3d76e7image.png)