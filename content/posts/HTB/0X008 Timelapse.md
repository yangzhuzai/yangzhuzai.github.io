+++
title = '0X008 Timelapse'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCE018e74c2d1d6b4774bee9e153f9fa396image.png)

![](/htb_img/WEBRESOURCE971f10663a36b5709df0dd5fdc3b60c4image.png)

![](/htb_img/WEBRESOURCE275264544e3a192dbc7b35ebacad3c79image.png)

# 获取立足点

smb匿名访问：

![](/htb_img/WEBRESOURCE7e4e95320e2efb7e81cb3556d4704ff7image.png)

疑似重置后的密码？

7c3XlgsE

![](/htb_img/WEBRESOURCE7987f78949a8217bb11355a4a8ca844eimage.png)

有密码：

![](/htb_img/WEBRESOURCE3c4eeca319f202c25dab2bc1ec1e574dimage.png)

破解：

![](/htb_img/WEBRESOURCEacda20c829e62857094e69c518ee487aimage.png)

但是这东西怎么用呢，后面刚好培训遇到了，pfx是一种存储个人信息的文件，简单理解可以理解为ssh密钥文件，不过普遍都是加密状态，需要解密，破解使用john即可：

thuglegacy

![](/htb_img/WEBRESOURCE4db93b4a2a32347beeac337f1e1ac932image.png)

然后需要将从pfx格式的文件，

openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out key.pem -nodes

openssl pkcs12 -in legacyy_dev_auth.pfx -nokeys -out cert.pem

![](/htb_img/WEBRESOURCE4bf5687eb4cea770d74add9eb7c84c4fimage.png)

evil-winrm -i 10.10.11.152 -c cert.pem -k key.pem -S

![](/htb_img/WEBRESOURCE3744727b51858143258abb2fdcf6a429image.png)

# 提权

powershell历史文件位置：

$env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt

可以看到账号密码：

$p = ConvertTo-SecureString 'E3R$Q62^12p7PLlC%KWaxuaV' -AsPlainText -Force

$c = New-Object System.Management.Automation.PSCredential ('svc_deploy', $p)

![](/htb_img/WEBRESOURCE5315faf8cc6fcc4214a5969b8fa62464image.png)

使用账号密码登录：

![](/htb_img/WEBRESOURCE94bf3055f926c5dc11c032608831c240image.png)

查看当前权限：

又laps的读取权限：

![](/htb_img/WEBRESOURCE125390578e5d691443c473f5b336c969image.png)

laps的解释：

![](/htb_img/WEBRESOURCE66556495d8691b46e715104676085f76image.png)

命令：

Get-ADComputer DC01 -property *

l@11Z8394&pq%$ksR(/87QMx

![](/htb_img/WEBRESOURCE3875cb9197c80142e55b097533d1388cimage.png)

![](/htb_img/WEBRESOURCEd350d4a6c591d0dfc6759b642dc1a9f5image.png)

# 总结

这台靶机的难度其实不太好评判，因为主要涉及的问题在于认知问题，而不是思路上的缺陷。