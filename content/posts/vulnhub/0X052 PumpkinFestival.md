+++
title = '0x052 PumpkinFestival'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE2a2160290ff84d0bac25efc53f27e018image.png)

![](/vulnhub_img/WEBRESOURCEb2c2736f59113e1cb2ad1b45d9ca98deimage.png)

![](/vulnhub_img/WEBRESOURCE752c4087bfb18918c0e91024f81b73c9image.png)

# 渗透

21端口，可以匿名登录，发现一个可疑线索：

PumpkinToken : 2d6dbbae84d724409606eddd9dd71265

![](/vulnhub_img/WEBRESOURCEde054088f8daddc2450d85e1e216b4c8image.png)

80端口

![](/vulnhub_img/WEBRESOURCEc80fe9bbb367538c48dd50d69f4075ddimage.png)

![](/vulnhub_img/WEBRESOURCEdae99a97e1b4903b9f5f0049d1766561image.png)

发现可疑线索：

Hey Jack!

Thanks for choosing our local store. Hope you like the services.

Tracking code : 2542 8231 6783 486

-Regards

admin@pumpkins.local

![](/vulnhub_img/WEBRESOURCE24433be8e1bf21e56e0842ef74eebbdaimage.png)

token?:

![](/vulnhub_img/WEBRESOURCE48e54729ac264037152ab08e189b97feimage.png)

浏览器携带看看，可能是用来访问其他几个页面的：

多次组合都发现失败了：

![](/vulnhub_img/WEBRESOURCE9fac0fa1d058fd739eee174ff04bf584image.png)

然后修改了hosts文件

![](/vulnhub_img/WEBRESOURCE644d1159228a8e884adf35a89c828261image.png)

再次访问：

又发现一个token:

PumpkinToken : 06c3eb12ef2389e2752335beccfb2080

![](/vulnhub_img/WEBRESOURCEfa02f6e2dd77b2b7f7bbd9110f3da885image.png)

wpscan:

![](/vulnhub_img/WEBRESOURCE42d2b86845076acf037181b53653b3b6image.png)

cewl收集的字典爆破失败了，用rockyou挂着爆破吧：

![](/vulnhub_img/WEBRESOURCE341fd3ab0828cbf75a3e9aa4c91d0500image.png)

![](/vulnhub_img/WEBRESOURCE3075c9bcfaaf3e7bad5ebc32f6d6164fimage.png)

然后再对[http://pumpkins.local](http://pumpkins.local)进行一次扫描：

![](/vulnhub_img/WEBRESOURCEf2f6d65e040329b88340321c63d1566bimage.png)

发现这么一个东西：

K82v0SuvV1En350M0uxiXVRTmBrQIJQN78s

这些东西到底要怎么用呢？

![](/vulnhub_img/WEBRESOURCEe6f82a1a596f26d08edc31a055177be0image.png)

最后没法了，看WP了，说这个东西是base62，之前我试过CyberChef的智能破解，但是没破解出来：

![](/vulnhub_img/WEBRESOURCEbef070dca642c07f9e7459cf8a584afeimage.png)

服了：

morse & jack : Ug0t!TrIpyJ

![](/vulnhub_img/WEBRESOURCEaabc05ef5d69ed43c5ea260e75e57fc4image.png)

登录wordpress:

又发现一个token:

PumpkinToken : 7139e925fd43618653e51f820bc6201b

![](/vulnhub_img/WEBRESOURCE6465a110ca99f5fec8538857d6012ccfimage.png)

尝试getshell失败，后面又看了wp，不是我吐槽，我是真没想到这是密码：

![](/vulnhub_img/WEBRESOURCEfd335d1c67143e09bf0de0508eededf0image.png)

但是，讲道理我是用过cewl收集密码，然后用wpscan去爆破的，这个不应该啊，然后我看了一下cewl收集的字典：

收集是收集了的，但是：没有！

![](/vulnhub_img/WEBRESOURCEab0ed3d814712b14b279016f10d89f67image.png)

然后登录admin:

又是一个token，但是这个东西有什么用吗？

PumpkinToken : f2e00edc353309b40e1aed18e18ab2c4

![](/vulnhub_img/WEBRESOURCE049cd4636888652cb3f387ff8c883977image.png)

本来以为拿到了admin就能getshell了，但是兜兜转转好像还是不行，

给我说，可以用harry去爆破ftp，我觉得以后不太能直接用cewl了，感觉没有手动的好用：

密码为yrrah

![](/vulnhub_img/WEBRESOURCE53ea75aefdea5c13ec53fe76c2e6fd93image.png)

套了一层又一层的目录，被我找到一个data.txt以及又一个token,

![](/vulnhub_img/WEBRESOURCE5f34002b0c7ce6870f4b06245a142bd7image.png)

好像是个压缩包

![](/vulnhub_img/WEBRESOURCE0b37e6f5b3e1353891eddebb92e47200image.png)

这个压缩包解压也是一层套一层：

先是tar、然后是bzip2、然后是tar

![](/vulnhub_img/WEBRESOURCEd8060524b576c2f3435ba4a6afe6cc0eimage.png)

看样子是二进制文件：

![](/vulnhub_img/WEBRESOURCEa34c075f112b5049c87c1224bcd463a3image.png)

之前遇到过：

![](/vulnhub_img/WEBRESOURCE10aeca4aae4711712b4dcbc1dbae36c7image.png)

然后登录又发现这样的问题：

![](/vulnhub_img/WEBRESOURCE42df736b52817559c4ea396d677574c1image.png)

经过排错，发现是ssh版本的问题，这样的问题在打靶机的过程中不少见，但是每次遇到报错都容易误导人：

echo "PubkeyAcceptedKeyTypes +ssh-rsa" | sudo  tee -a /etc/ssh/ssh_config

![](/vulnhub_img/WEBRESOURCE645440de64c8db2707ecd1a40992253cimage.png)

![](/vulnhub_img/WEBRESOURCE44149999312038964eb750e20055d95cimage.png)

# 提权

这靶机打的时间有点长，人都打懵了，我sudo  -l，琢磨着半天，我没密码啊，没想到我其实是有密码的：

Ug0t!TrIpyJ

![](/vulnhub_img/WEBRESOURCE331e849e0bbad116097e6032ed39cbb9image.png)

反弹shell:

![](/vulnhub_img/WEBRESOURCEf5b75c6fe5a938f2b822ae3a46fa533aimage.png)

![](/vulnhub_img/WEBRESOURCE12bfd419b6e86db777dc45146fce0346image.png)

# 总结

相当长一段时间没有写总结了，主要是之前的靶机没有太多总结的必要，这个靶机的难度我觉得主要有两个，一个是base62的加密，一个是海量的信息，很容易产生误导，由此让我觉得必须做一个比较规范的操作方法才更好，让思路不存在死角。