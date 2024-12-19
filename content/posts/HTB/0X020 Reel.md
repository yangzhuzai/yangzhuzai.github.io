+++
ShowToc = true
TocOpen = true
title = '0X020 Reel'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCEb5ad5902eaed1cba670f021c3c57b41eimage.png)

![](/htb_img/WEBRESOURCEf16cd19536cd0d56229a8932c3f7ae8dimage.png)

![](/htb_img/WEBRESOURCE507c1c6553aaf03d2f202216cec5e25aimage.png)

# 获取立足点

值得关注的服务：ftp\ssh\smb

先看ftp:

有匿名登录，发现信息：

![](/htb_img/WEBRESOURCEc1d9c2f6a55ae6070924f9978eb027a4image.png)

里面的信息大致理解为：

有程序回记录hash，然后上传到documents下的文件将会被查看，然后是操作记录？

![](/htb_img/WEBRESOURCEebd586d457654b21f2bce6d856aaa0c4image.png)

接着看smb:

![](/htb_img/WEBRESOURCE3c6f18499bb0b6b6ee076a6daddc9565image.png)

ssh目前没有其他信息，就不查看了，研究一下上面写的rtf格式的文件：

![](/htb_img/WEBRESOURCE0a2bca99e6897513ee294a0de8e21d5dimage.png)

了解了一下，RTF格式的文档也能调用宏来进行攻击：

[https://github.com/bhdresh/CVE-2017-0199](https://github.com/bhdresh/CVE-2017-0199)

![](/htb_img/WEBRESOURCEd90abd08c5b1adac77be6fdef6f6c4b6image.png)

这里的意思是，可以让rtf格式的文档来下载HTA文件执行，hta再进行上线，然后发现前面理解好像有些问题

FTP并没有上传权限：

![](/htb_img/WEBRESOURCE0ea5c66efa52b80eaf7d75214eb43870image.png)

呐意思其实是向文件创建人的邮箱发信息：

因为25端口开放的：

![](/htb_img/WEBRESOURCE6b042694b831f21a63b58fce782e675dimage.png)

配置Empire，生成hta：

![](/htb_img/WEBRESOURCEcde94f4bf5107a8e9e9dec93509e6fd9image.png)

发送邮件：

![](/htb_img/WEBRESOURCEa71e39f5b5e4387ed9f3b6c3049c4a7cimage.png)

![](/htb_img/WEBRESOURCEcc8fd18eea6201ec71b8dc75fd4e841fimage.png)

成功反弹shell:

![](/htb_img/WEBRESOURCEba70200d15c647d2d6da1052cfa5eba8image.png)

别的不谈，这玩意儿，延迟是真的高：

![](/htb_img/WEBRESOURCE432ff945dd6abe46fc753eea5f84b0a2image.png)

# 提权

当前在域内：

![](/htb_img/WEBRESOURCEe3dbfd5a3293d8e8d09af062bd6158f6image.png)

上bloodhound把：

不让用：

![](/htb_img/WEBRESOURCE7cdb5207d45b80cf3fd62f35c794318fimage.png)

继续信息收集，桌面上有一个cred.xml文件

该文件是 PSCredential 对象当中

可以通过powershell恢复：

$credential = import-clixml -path c:\Users\nico\Desktop\cred.xml;$credential.GetNetworkCredential().username;$credential.GetNetworkCredential().password




![](/htb_img/WEBRESOURCE34d94a35113dc09d309109c4c45ac6a2image.png)

成功获取到了用户名密码：

Tom

1ts-mag1c!!!

通过剩下的ssh进行登录：

![](/htb_img/WEBRESOURCEa97a38f97567ba68e6e8c1a4ee84bce1image.png)

奇怪的是，不允许使用其他程序，但是机器上默认存在bloodhound的文件夹：

![](/htb_img/WEBRESOURCE8dd15558161f7ef6370c2f0985f86facimage.png)

下载里面的csv文件，然后筛选当前用户：

![](/htb_img/WEBRESOURCE5b8d174d1674c96dc6ca2d09548a84daimage.png)

发现tom用户对claire有WriteOwner权限：

![](/htb_img/WEBRESOURCE623ba4659e9858db553f6878c54ea2d2image.png)

这个用户拥有Backup_Admins组的WriteDacl权限：

![](/htb_img/WEBRESOURCE7cb54b9ad9b68a294f2bd812f32c941aimage.png)

自带的Powerview倒是可以用：

通过claire有WriteOwner权限，修改权限，修改密码：

![](/htb_img/WEBRESOURCEbccaec184c1639debf922a689f763e71image.png)

登录claire：

![](/htb_img/WEBRESOURCEe1f5fadf869cdfcfea0bcf25142b2f80image.png)

由于拥有WriteDacl权限，我们可以把自己添加进去：

![](/htb_img/WEBRESOURCE10b7ed429f757908bc23090051b099caimage.png)

当前用户登录进去，发现对admininstrator目录有读取权限：

![](/htb_img/WEBRESOURCE95ba6bc078c2dd19817c469b8743c0fcimage.png)

直接查看root发现失败：

![](/htb_img/WEBRESOURCE5983da869d0cc04c3e035f9fa0d042d0image.png)

但是desktop目录下存在很多东西：

![](/htb_img/WEBRESOURCEda78c4179b877ea6a8c66c5e4562fb69image.png)

把所有文件读取，然后丢到文件夹里面就可以查看关键词：

![](/htb_img/WEBRESOURCE3b8ab54c32c223f3f98867b2bababe4eimage.png)

提权成功：

![](/htb_img/WEBRESOURCE96926f9bf4423136c09367b3954fc785image.png)