+++
title = '0x053 PumpkinGarden '
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE8f48023944d46d52b206328bd86b2806image.png)

![](/vulnhub_img/WEBRESOURCE2af349a70c97663909a94dfad0a654a5image.png)

![](/vulnhub_img/WEBRESOURCEc04468d25bf498f4746b82fc374ec41bimage.png)

# 渗透

21端口

可能存在jack用户，以及提到了PumpkinGarden

![](/vulnhub_img/WEBRESOURCE394da7310347d4efc508845a87ac3861image.png)

1515端口

提示说，南瓜图片可能能帮助我，以及在under the hood？

![](/vulnhub_img/WEBRESOURCE31af89c66e13be78e1dc4cbf70ceafc5image.png)

尝试查看了一下首页的图片，没有发现东西

遂端口扫描，发现img

![](/vulnhub_img/WEBRESOURCE1e0557e1b2eb79716a48a6595499f8fbimage.png)

看看里面有什么好东西：

![](/vulnhub_img/WEBRESOURCEbec7482571ab3c2ee2891c8e0233313aimage.png)

c2NhcmVjcm93IDogNVFuQCR5

![](/vulnhub_img/WEBRESOURCE1c20635d4ab26d58da5d912765175a85image.png)

这些图片又看了一下，没有隐藏信息，那么这个字符串是用来干嘛的？

scarecrow : 5Qn@$y

![](/vulnhub_img/WEBRESOURCE364aaf5263d963514551a40263b72b54image.png)

居然登录成功了：

![](/vulnhub_img/WEBRESOURCE87d8fe7ee6dc93ebe5898828550c0089image.png)

# 提权

![](/vulnhub_img/WEBRESOURCEa727f8fca930152128d6142faa9de61dimage.png)

![](/vulnhub_img/WEBRESOURCE2b0855dd7a86366034cec844c4bd9e22image.png)

发现是goblin用户的密码：

![](/vulnhub_img/WEBRESOURCE34bdc0351488c9c1fbd50601e0b3caefimage.png)

还有提示：

![](/vulnhub_img/WEBRESOURCEd6d0988c0498cacd89b2773bf0b0f7b9image.png)

但是sudo就可以提权：

![](/vulnhub_img/WEBRESOURCE2a48d17de6802a9d5adc2973e9efe4f3image.png)

![](/vulnhub_img/WEBRESOURCE0c7f73d2455ec8afc6f9b8221b303295image.png)

Congratulations!

![](/vulnhub_img/WEBRESOURCE065c564a08f04e0260ced7b1217a4038image.png)