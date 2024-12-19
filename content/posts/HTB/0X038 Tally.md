+++
ShowToc = true
TocOpen = true
title = '0X038 Tally'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCE2e65f32af6430600375a0f308e85fe70image.png)

![](/htb_img/WEBRESOURCE5bc4aae6d24dbfb004aca4c9548c9cecimage.png)

![](/htb_img/WEBRESOURCE1e113e5e44899274a4b48480d7e76007image.png)

![](/htb_img/WEBRESOURCE5e37c1a6db88c53a1ace675970cd5b08image.png)

# 获取立足点

值得关注的服务：

ftp、http、smb、mysql、winrm

先看ftp:

![](/htb_img/WEBRESOURCE6ed7e599ecd0488c2e86079a77b99131image.png)

smb：

![](/htb_img/WEBRESOURCE8fe345e31db3bdb7cf273e07915c3984image.png)

只能看看http了：

![](/htb_img/WEBRESOURCE090ee03356170ef55f38ba11bf7f2564image.png)

我扫了半天也没扫出东西来，看了才知道，得用其他字典:

/usr/share/wordlists/dirb/vulns/sharepoint.txt

![](/htb_img/WEBRESOURCEf0bfc009a4a3d09250fd72083120869aimage.png)

这三个都可以用：

![](/htb_img/WEBRESOURCE7824a383e5e426d8266d5dada79e45dfimage.png)

![](/htb_img/WEBRESOURCEef34adcded028c6e4b5c310e3a0ffdcdimage.png)

发现文档：

![](/htb_img/WEBRESOURCEae376517451fc4c05164c1bc4e7ac35dimage.png)

有密码了，但是好像没名字

![](/htb_img/WEBRESOURCEee5a5b5cb2e5518d45cbd75613e68974image.png)

先试试，但是失败了，继续信息收集，通过删除#号前面的内容，可以成功看到信息：

![](/htb_img/WEBRESOURCE541055b77ce347ec03309eb5d4f714a6image.png)

用户名为：ftp_user，同时告诉我们一个信息，他们会定时查看htlm文件，在Intranet文件夹中，可能可以钓鱼：

![](/htb_img/WEBRESOURCE6a08df7416a01876c4308183d3470cdbimage.png)

发现用户名：

![](/htb_img/WEBRESOURCE810f2e4a66c0c88cdad0af2af768b61bimage.png)

![](/htb_img/WEBRESOURCEf02fd893f94e0bf1dcac3f93def99a68image.png)

继续信息收集，发现一个keepass软件：

![](/htb_img/WEBRESOURCE1c41a58845ab8cd87721b31604d23358image.png)

这个软件好像是个密码保存软件：

![](/htb_img/WEBRESOURCEb74cb4f7fe8e5557ee0657c8fcaba253image.png)

并且查询后发现存储的密码是可以破解的：

![](/htb_img/WEBRESOURCE8949ce1edca07994e3f46dcd4b958945image.png)

simplementeyo    (tim)

![](/htb_img/WEBRESOURCE41302bbc7352a378dbb7059f2d716cf1image.png)

通过keepass2读取文件：

![](/htb_img/WEBRESOURCEd79827b229c7ced20ad4c600e0997da0image.png)

winrm登录失败了，但是成功登录了smb，说实话，东西太多了，这样增加的难度我不是很喜欢：

在zz_Archived中发现密码：

![](/htb_img/WEBRESOURCEe5939a893170acd30b2cbc81bfbb434cimage.png)

但是好像不对：

![](/htb_img/WEBRESOURCEe7195889aad737dab014b521ce28e36cimage.png)

在zz_Migration文件夹下面发现程序：

通过strings读取，发现新的账号密码：

UID=sa;PWD=GWE3V65#6KFH93@4GWTG2G;

![](/htb_img/WEBRESOURCE5103af651a716cd09f0c05f098e50cf0image.png)

登录成功：

![](/htb_img/WEBRESOURCEa841b09495bbbf2b098624efbe1320e5image.png)

可以使用xp_cmd执行命令：

![](/htb_img/WEBRESOURCE8b11f5dd77ccc3abca638e4901f27507image.png)

反弹shell:

使用nishang反弹：

![](/htb_img/WEBRESOURCEd14fda54829ae8aced70628d8ce35a06image.png)

# 提权

上winpeass：

![](/htb_img/WEBRESOURCE1f11df8eeb32e6ebc44d67d022f9547dimage.png)

可以尝试内核提权：

![](/htb_img/WEBRESOURCEb554eba66df744b0a7c1c6fa7a9b5ac1image.png)

烂土豆提权：

![](/htb_img/WEBRESOURCEfc619e5b4227fa2704e01749a5ab1286image.png)

![](/htb_img/WEBRESOURCE5b901e37ff51c38513239d0428094a04image.png)