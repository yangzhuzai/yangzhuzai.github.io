+++
title = '0x058 Orcus'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



![](/vulnhub_img/WEBRESOURCEd025b8ab10aa135fb9a5859ce8df8e5dimage.png)

![](/vulnhub_img/WEBRESOURCE59e59a940a7e0c8e7ff3ca81de317ddfimage.png)

![](/vulnhub_img/WEBRESOURCE7fc4ba468d7681ab6f38eee1a3727eccimage.png)

![](/vulnhub_img/WEBRESOURCEc785c237ce995475c7e5e7101df3a286image.png)

![](/vulnhub_img/WEBRESOURCEbc2e5ee2c6a7e5ab47d0f2191d0eb393image.png)

![](/vulnhub_img/WEBRESOURCEe345087502a5935e1152a8156c3a042eimage.png)

# 渗透测试

看样子是开了samba先看看吧，无果：

![](/vulnhub_img/WEBRESOURCE9cdfe965b0f6ffc62ffdc133e9f570f5image.png)

80端口看看：

发现已知CMS

![](/vulnhub_img/WEBRESOURCEfccb4e29c60fcffa14ecb277083999a0image.png)

但是很可惜，已知漏洞利用失败了：

![](/vulnhub_img/WEBRESOURCE91ecc5c1ef60885fcd4b0dbaea5c1019image.png)

_0c7aa37c1e5b7386c5d18dba80bb5d3b^118e4111302d35936b445390f58fb9f006cda2dd_0.file._maintenance.tpl.php

疑似线索：

![](/vulnhub_img/WEBRESOURCE8a82638c40e6a3857f20eb25cfa77755image.png)

发现密码：

DEFINE ('DB_USER', 'dbuser');

DEFINE ('DB_PASSWORD', 'dbpassword');

DEFINE ('DB_HOST', 'localhost');

DEFINE ('DB_NAME', 'quizdb');

![](/vulnhub_img/WEBRESOURCEe54065478db499e28f7bd7891e058162image.png)

![](/vulnhub_img/WEBRESOURCE7ec53e00aaba62f4b36d897bea408a50image.png)

有绝对路径，但是没权限：

![](/vulnhub_img/WEBRESOURCEd40b7ebd91012cdb5eac78cd21844b31image.png)

通过数据库发现路径：

![](/vulnhub_img/WEBRESOURCEefd585cd4ab8bcc0325a21678b70ddbdimage.png)

需要设置一下，然后就可以进去了：

![](/vulnhub_img/WEBRESOURCEda8c1bc7549f245c7e01a8d0f1b3b48bimage.png)

![](/vulnhub_img/WEBRESOURCEa0df844bcce0229871917de1f531834dimage.png)

![](/vulnhub_img/WEBRESOURCEc8fd6a2c42c0fe727f24488d8e1a9122image.png)

# 提权

内核：

![](/vulnhub_img/WEBRESOURCEc6c2b024081b8267206f416fd7f998abimage.png)

NFS

当服务器中存在NFS共享，且开启了no_root_squash选项时，这时如果客户端使用的是root用户，那么对于共享目录来说，该客户端就有root权限，可以使用它来提升权限。

![](/vulnhub_img/WEBRESOURCEa3ae3bcea285120de225ba2e8e6d24d1image.png)

![](/vulnhub_img/WEBRESOURCE1f2644616e804e3b8c187f83404230f4image.png)

![](/vulnhub_img/WEBRESOURCE8531d9dd5547cc90b149c72f0d05db8eimage.png)

挂载

![](/vulnhub_img/WEBRESOURCE4f26878fa032d799055311eeb25d77feimage.png)

复制bash

![](/vulnhub_img/WEBRESOURCE9f306fe98385004b89e12805db974838image.png)

给权限：

![](/vulnhub_img/WEBRESOURCE2342d6021b8f2d657d5a64dcd2da38e1image.png)

![](/vulnhub_img/WEBRESOURCE9ebaaa31c35dcb205bb054608f222762image.png)

./shell -p

![](/vulnhub_img/WEBRESOURCE4bf71bd741a2c431a182f313ee527ddcimage.png)