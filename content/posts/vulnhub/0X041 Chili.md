+++
title = '0x041 Chili'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCEeeadc90a6640f73b17281a4e92454ff3截图.png)

![](/vulnhub_img/WEBRESOURCE9ca1efef375b765dfd3179dcc8f00fbe截图.png)

![](/vulnhub_img/WEBRESOURCEcca80187dba627272593584d5f1ca45d截图.png)

# 渗透

21端口匿名登录失败吗，那就只能看80端口了：

![](/vulnhub_img/WEBRESOURCE314b058a81141db97eae8b02efe157fe截图.png)

80端口和图片大致看了一下，没有发现隐藏信息，然后就用cewl收集了一下账户密码，但是这里有个地方没考虑到，那就是chili开头是大写，然后这里应该尝试改成小写：

![](/vulnhub_img/WEBRESOURCEca70c67f64233f0005d2b2eeb10215b1截图.png)

然后用hydra进行爆破即可，但是这里完了截图，爆破出来应该是a1b2c3d4:

![](/vulnhub_img/WEBRESOURCE0fe391cc8c464547415fd2093ee97c26截图.png)

看到当前是chili的家目录，第一反应是写密钥，但是很可惜，可以开放ssh服务：

![](/vulnhub_img/WEBRESOURCE9003b2be5cbc0076f109d201c80f50b5截图.png)

从web入手，写shell吧：

可以发现.nano目录是具有权限的，然后直接把反弹shell的php代码上传上去即可，记得给木马执行权限，直接给777：

![](/vulnhub_img/WEBRESOURCEb0727ec019c27a75388ad7ecbb13c472截图.png)

# 提权

passwd文件可写：

![](/vulnhub_img/WEBRESOURCE6c76748d05ae9d0839cfcde67980cafe截图.png)

chili用户是可以切换的，直接改passwd文件密码吧，密码Abcd1234

```
root:$1$ZytY2Uoj$1.aU20M/0/xD4Wo1ymW9Z0:0:0:root:/root:/bin/bash
```

![](/vulnhub_img/WEBRESOURCE62e5b3d7d2fb2aba0052d425977bacbf截图.png)

![](/vulnhub_img/WEBRESOURCEc588481c1817e4c5db8d7cea64ab5bc7截图.png)