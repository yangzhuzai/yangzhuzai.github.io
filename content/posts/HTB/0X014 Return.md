+++
ShowToc = true
TocOpen = true
title = '0X014 Return'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCEe382420ae6c92d98a37732267ce502eaimage.png)

![](/htb_img/WEBRESOURCEda43ac946da8dcb59c3afe808daeb124image.png)

![](/htb_img/WEBRESOURCEbd2dd6cc4db68b83646dbf96165b441eimage.png)

# 获取立足点

简单看了一下端口开放情况，存在一个80 http端口，5985 winrn端口，445端口，389 ldap等。

域名：return.local

主机名：PRINTER

先看445吧：

失败，没有匿名共享

![](/htb_img/WEBRESOURCEba2a5a86757af35aac982a2fdd0867e9image.png)

然后看ldap有没有匿名访问：

有，匿名访问：

罗列用户名：

发现没有东西：

![](/htb_img/WEBRESOURCEfcbb23001a2bfd7fc6ae55ca963c4049image.png)

看来目标应该是从80端口下手了，80端口是个打印机，用户名为svc-printer，多半是主机用户名了：

尝试进行密码爆破，失败了：

![](/htb_img/WEBRESOURCE4d749002945688c86db1e87c25d04735image.png)

AS-REP Roasting试一下

失败了：

![](/htb_img/WEBRESOURCEb3e47c3b025498e889b144cd31cd044bimage.png)

研究一下这个打印机怎么打：

NTLM relay:

居然直接成功了，还抓的明文密码：

![](/htb_img/WEBRESOURCE9a9b4e0b1a11e12d39024ea17e2b2ddbimage.png)

[LDAP] Cleartext Client   : 10.10.11.108

[LDAP] Cleartext Username : return\svc-printer

[LDAP] Cleartext Password : 1edFg43012!!

登录winrm，拿到flag

![](/htb_img/WEBRESOURCE2b70a385d46e20c450d99b858254e3ffimage.png)

使用bloodhound.py:

分析结果，其实只能通过本地提权来进行，然后利用dcsync达到域管理员组：

查看该成员的信息，可以发现该成员属于高价值目标组，SERVER [OPERATORS@RETURN.LOCAL](http://OPERATORS@RETURN.LOCAL)

![](/htb_img/WEBRESOURCE287aadd75af37792b33b0d28e6d9cc59image.png)

winpe:

![](/htb_img/WEBRESOURCE4b03948e76b801743e229db82977d14fimage.png)

改组成员可以修改服务，我们可以创建一个服务来提权：

![](/htb_img/WEBRESOURCEafd9dff26e6b60b5736c08db8187220fimage.png)

提权成功：

![](/htb_img/WEBRESOURCEd0b0d47f3716e70a6f97c6022f336ed9image.png)

使用mimikatz进行dcsync获取admininstrator hash:

![](/htb_img/WEBRESOURCEc5ccbe8a8393236630985bd0ffbf912eimage.png)

使用administrator登录：

![](/htb_img/WEBRESOURCE32b2c1e74f5ebdafaefaa6ffc0247614image.png)

# 总结

这台靶机的关键点在于ntlm中继和主机提权，其实不难