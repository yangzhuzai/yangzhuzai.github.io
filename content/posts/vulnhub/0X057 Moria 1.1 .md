+++
ShowToc = true
TocOpen = true
title = '0x057 Moria 1.1 '
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCEcacaae954e8d23744a4041baa57d1ba3image.png)

![](/vulnhub_img/WEBRESOURCEe8b6d1d91ecfaf277eea7160cc9902ddimage.png)

![](/vulnhub_img/WEBRESOURCE3ff8278fa61995fcc0fe922ea689a529image.png)

# 渗透

21端口

没有匿名登录，但是好像有用户名提示

![](/vulnhub_img/WEBRESOURCEbe35638162f0c84f995eb0b88335b584image.png)

80端口

![](/vulnhub_img/WEBRESOURCE6bfd8cad1a1e44e51e99de3184ad4c0eimage.png)

![](/vulnhub_img/WEBRESOURCEf42024073e9017b2269167ae04e3eaa9image.png)

什么意思呢：

![](/vulnhub_img/WEBRESOURCEd6b8e169b957283965b6df36f9826fd1image.png)

收集一下目前的单词：

![](/vulnhub_img/WEBRESOURCE09e358376ca873d91f00339ce2f9eceeimage.png)

可能用到账户号密码的地方：ftp\ssh\首页图片，那就挨个试了吧

![](/vulnhub_img/WEBRESOURCE74ded5f2a963ebce62001c53fe911babimage.png)

![](/vulnhub_img/WEBRESOURCEdd14bc3a9f58394795c65487b3fa40baimage.png)

发现还是不对，线索断了，搜了一下，发现这个靶机相当有意思

没想到，这个页面是随机的，刷新后就不一样了：

![](/vulnhub_img/WEBRESOURCE38e9387b3accccc4cdcf115e22207bd9image.png)

对这个目录再次爆破：

![](/vulnhub_img/WEBRESOURCE173673037a134a2f3426539a6d654f4cimage.png)

但是看这段对话还是一头雾水：

![](/vulnhub_img/WEBRESOURCE4c96b7f77d0612b3416b5d8423f21d43image.png)

然后看wp，我是真的没想到这里面能藏信息的，注意看本机发给靶机的几个包：

按顺序为：77，101，108，108，111，110，54，57

ascii码对应为：

![](/vulnhub_img/WEBRESOURCEf371fe4b288f2d5fc3af1e4301ce02e9image.png)

这是ftp的密码：

虽然没有写入权限，但是有线索：

QlVraKW4fbIkXau9zkAPNGzviT3UKntl

![](/vulnhub_img/WEBRESOURCE38551cb5a4ad272ceea65021ad5fff7cimage.png)

访问：

![](/vulnhub_img/WEBRESOURCEe327071b007faf520cce5645a85d9dbfimage.png)

![](/vulnhub_img/WEBRESOURCEeb12e472507b9dadaba29cb1df575c5cimage.png)

加盐的MD5破解：

这里我其实是没找到hashcat的破解格式的：

![](/vulnhub_img/WEBRESOURCE9740b10664c6d3d7e62a15013b762e06image.png)

后面又查了一些资料：

![](/vulnhub_img/WEBRESOURCE10f1e4e395d7e67cdd63c66747a4c01eimage.png)

![](/vulnhub_img/WEBRESOURCE9e0a2b090648b3e398a203c56743040aimage.png)

![](/vulnhub_img/WEBRESOURCEaa15cbedaa5c45788606113d41edc59cimage.png)

稍微处理了一下：

![](/vulnhub_img/WEBRESOURCE31fde46dd5dcb40d19ac7a079405bb39image.png)

hydra爆破有拒绝，用

![](/vulnhub_img/WEBRESOURCE7509ca6d065293626fc1062738f8adffimage.png)

有点严格，算了，手动试吧：

服了：

Ori

spanky

![](/vulnhub_img/WEBRESOURCE8de11624856aa7d68aef845b97c5f3b4image.png)

# 提权

![](/vulnhub_img/WEBRESOURCE85d0e343de32d40499980fa7f0579067image.png)

2021-4034可以提权：

![](/vulnhub_img/WEBRESOURCEf566db66025c742fbf89057131917360image.png)

作者留下的提权方式：

![](/vulnhub_img/WEBRESOURCEaee5155ac4ff608461870a6230a2c268image.png)

 