+++
ShowToc = true
TocOpen = true
title = '0X022 cascade'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

端口扫描就出现问题了，不让扫描？

最开始7000速率不行，改成2000就可以了：

![](/htb_img/WEBRESOURCE5b4510dec7bf52d0a6fdb5778f11656eimage.png)

![](/htb_img/WEBRESOURCE7c3558936730c318e4f16a7a4710881aimage.png)

![](/htb_img/WEBRESOURCEe59f9ec75d3b2e07289bfda3ede34301image.png)

# 获取立足点

关键端口开放情况，445、389、3268、5985

![](/htb_img/WEBRESOURCEef367b5e20ce2fe2e6c5c7af8f0a1a33image.png)

![](/htb_img/WEBRESOURCE534850d681dd10754905937f9f8f01a7image.png)

有匿名访问：

![](/htb_img/WEBRESOURCEf72e7106a3b68884c2de05b953cd7289image.png)

获取了用户名，可以尝试密码喷洒，或者AS-REP Roasting：

失败了，但是这里面密码喷洒一般很少：

![](/htb_img/WEBRESOURCE81b62e47f4b347b26233abc593d71817image.png)

查看用户所有信息：

![](/htb_img/WEBRESOURCE80e99820f3a63b441c8f8ae31c5f680aimage.png)

发现密码：

r.thompson

rY4n5eva

![](/htb_img/WEBRESOURCE5ebcb345680f2cf3e99d1a5346ef60e2image.png)

winrm登录失败，回头看smb:

![](/htb_img/WEBRESOURCE341c8acb1447d66cb8f6db4647876625image.png)

翻查了一圈，把所有东西都下载下来了：

![](/htb_img/WEBRESOURCE487616986a1b1f7e82dbaa82257d6cc2image.png)

新用户？

![](/htb_img/WEBRESOURCE664b982284b36d00711ac12bf5d60859image.png)

别的不说，在目前的情况下，我认为这些文件里面，应该是存在登录成功的方式的：

破解vnc的密码：

![](/htb_img/WEBRESOURCE08d05305af661ad047e2733e0594d088image.png)

用户名为：

![](/htb_img/WEBRESOURCE73254ba3dd435455c5bcae596a3cb7b9image.png)

登陆成功：

![](/htb_img/WEBRESOURCE9fa334cf4469d5319461bc6c3a2e0144image.png)

# 提权

bloodhound：

好像没路子：

![](/htb_img/WEBRESOURCEc9fb672ffed27c19776681e4b0ed4a49image.png)

看看winpe:

![](/htb_img/WEBRESOURCE9126de058238ce21cb0fec0d0eb12e3cimage.png)

![](/htb_img/WEBRESOURCE0df3b5f3f25492129fe291418d862bf1image.png)

好像没有头绪，应该是得继续信息收集：

再去使用s.smith登录smb:

![](/htb_img/WEBRESOURCEb1693bd21024dc11b08f1032713bfaedimage.png)

发现db文件：

![](/htb_img/WEBRESOURCE6ee9797f68c4c925e44ed3e9677d69c4image.png)

发现是sqllite文件：

![](/htb_img/WEBRESOURCE4c8ea33241bc97ef76aab785e2d4df3cimage.png)

sqlitebrowser Audit.db：

![](/htb_img/WEBRESOURCE47d54905d13811eee58153b714032093image.png)

这个文件识别了一下，好像破解不了，需要逆向：

核心是这两个文件：

![](/htb_img/WEBRESOURCE5d948efd5625735e0c0ee0fd9d0f8ab8image.png)

.net开发的：

![](/htb_img/WEBRESOURCEae97a3fff3e660d696bebaa91993314fimage.png)

用dnSpy逆向：

密码在这一步进行解密：

![](/htb_img/WEBRESOURCEed9d51e1f035b9c51aec6851f488f98dimage.png)

点进去：

![](/htb_img/WEBRESOURCE649064cb57348f12dec80c56adc84c80image.png)

加密方式、密钥、偏移量都有了：

[https://www.fly63.com/tool/cipher/](https://www.fly63.com/tool/cipher/)

在线解密：

w3lc0meFr31nd

![](/htb_img/WEBRESOURCE1513dc9cf23a5040e0dce1c1ce0916d5image.png)

远程管理组，应该可以登录：

![](/htb_img/WEBRESOURCEe2236654b51ab6b67e1bf8912790a836image.png)

再次bloodhound查询：

也没有明显的路径：

![](/htb_img/WEBRESOURCE54ec7fbdb07b3ecf9ec32f51660ed814image.png)

再次peass:

也没发现什么东西：

![](/htb_img/WEBRESOURCE931c077f2abc10a8a00e426819a2b14cimage.png)

查看该用户权限:

属于：AD Recycle Bin组

简单查了一下：

![](/htb_img/WEBRESOURCEeef207c568ec226626746c5df31ba92fimage.png)

![](/htb_img/WEBRESOURCEa64587efe5202afd0e0672ee4953d997image.png)

查看已经删除了的用户：

Get-ADObject -filter 'isDeleted -eq $true -and name -ne "Deleted Objects"' -includeDeletedObjects

这是之前的文件里面提到了的：

![](/htb_img/WEBRESOURCE9647b4ec50fca6cb6b35a72ee2b9e7dfimage.png)

获取该用户的详细信息：

Get-ADObject -filter { SAMAccountName -eq "TempAdmin" } -includeDeletedObjects -property *

![](/htb_img/WEBRESOURCEc6c4e2052faeef82924cc673e11c0446image.png)

密码是：

baCT3r1aN00dles

![](/htb_img/WEBRESOURCE824fd0894b4157ef1ffbceb91af8375dimage.png)

之前的文章里面说到，这个密码是管理员密码：

![](/htb_img/WEBRESOURCEc36e5c6ed78702a0350303bcf752feeeimage.png)

# 总结：

这个靶机涉及的东西有点多，包括对ldap的理解、简单的逆向分析、细心的信息收集等。

流程和思路梳理：

1、端口扫描速率过会导致扫不出来，得降低速率。

2、LDAP匿名访问，需要查询详细的用户信息，通过工具windapsearch.py实现，效果较好

3、获取到一个普通用户r.thompson，但是该用户无法登录winrm，只能另寻他法，使用该用户登录smb

4、通过smb发现一些信息，其中包括tempadmin的密码是管理员密码、vnc注册表文件

5、通过VNC注册表文件恢复，复原出密码，存在与s.smith 目录下，为该用户的密码

6、s.smith可以登录winrm，但是该主机任然无法主机提权或者bloodhound有路线到domain admin

7、再次回到smb，使用s.smith账户，进入audit目录，该目录下存在db文件，但是加密状态

8、使用dnspy反编译.net程序，得到解密的偏移量和密钥，成功破解db密码，获得arksvc账户

9、再次登录winrm，发现该账户存在AD Recycle Bin组内，这个是负责回收站功能的组

10、通过这个组权限，成功获取到已经删除了的账户tempadmin的密码

11、使用该密码成功登录administrator