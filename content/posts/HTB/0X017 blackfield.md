+++
ShowToc = true
TocOpen = true
title = '0X017 blackfield'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCEea18669927047071b2f016d741cd9f9cimage.png)

![](/htb_img/WEBRESOURCEfc75e9ba6e8d681ccc511c28beec4352image.png)

![](/htb_img/WEBRESOURCE4a7cc6b9b385446a255fed934430c98dimage.png)

# 获取立足点

值得关注服务：ldap\smb\5985

smb存在匿名登录：

![](/htb_img/WEBRESOURCEc7ba8abad6771efef30f690f769ad499image.png)

这个文件夹好像是个人文件夹，这些文件名可能是用户名，接着看ldap，没有什么有效信息：

![](/htb_img/WEBRESOURCE8f53bc5134ba6e86cf716f47ea2c4d1bimage.png)

使用刚刚smb获取到信息，进行用户名枚举：

获得可用用户名三个：

![](/htb_img/WEBRESOURCEa9472104fb02cd43db957d2ff9e59bebimage.png)

有了用户名可以尝试AS-REP：

![](/htb_img/WEBRESOURCE436a68b5e9c5043a128f88a8f9c59a09image.png)

破解成功：

#00^BlackKnight

support

![](/htb_img/WEBRESOURCEe8f7242a56316182305de35d369ef0f6image.png)

winrm登录失败：

![](/htb_img/WEBRESOURCEc578c613e35f8b16225d7e65422825e4image.png)

bloodhound.py看看吧：

但是并没给出什么有用的信息：

![](/htb_img/WEBRESOURCE9e3a4f3918f84ebb4a7715cb6438436bimage.png)

psexec也失败了：

![](/htb_img/WEBRESOURCE44c2f715a9111c4b0ce67010f55c0f7bimage.png)

kerberoasting：

![](/htb_img/WEBRESOURCE8ba0a51c180c93183dc8450b4f964333image.png)

再次查看bloodhound：

![](/htb_img/WEBRESOURCE87bd2e04203f2c43efbee1eb37d52375image.png)

我们可以修改AUDIT2020用户的密码：

![](/htb_img/WEBRESOURCE9917a62e8115593d051ae2621dd92d98image.png)

登录smb:

![](/htb_img/WEBRESOURCE2c928d8e4b93bd01c9d900695b627a78image.png)

发现关键内容lsass.zip

![](/htb_img/WEBRESOURCEd13f868e7c7adfb3d7f02664f65ae955image.png)

读了一堆hash:

![](/htb_img/WEBRESOURCEf3a02507e0161c2a105cf0f74dfa07f1image.png)

提取出来吧：

![](/htb_img/WEBRESOURCE91e34b8a176e74c2a8b29e4d138fbb13image.png)

使用crackmapexec来爆破smb以获取可用hash:

![](/htb_img/WEBRESOURCE7ef8dde872b3ece13cc9c59d6ee0738fimage.png)

bloodhound可以知道，这是BACKUP OPERATORS组的：

![](/htb_img/WEBRESOURCE06bb8613fc5fb728047558ecb7567c99image.png)

通过查找信息，发现以下解释：

![](/htb_img/WEBRESOURCE7a04b590659e373eebf7fa1592ad9a1eimage.png)

同时还是远程管理组的：

![](/htb_img/WEBRESOURCEd305227fb19a6f99cf1090fe281feb4fimage.png)

登录后，利用BACKUP OPERATORS组的权限，来备份

操作参考这个：

[https://gist.github.com/manesec/9e0e8000446b966d0f0ef74000829801](https://gist.github.com/manesec/9e0e8000446b966d0f0ef74000829801)

![](/htb_img/WEBRESOURCEf9acbb761bfff3fbaeb7fb73b355ee40image.png)

![](/htb_img/WEBRESOURCE5fff9a30efc11cd2422d513ed818c821image.png)

![](/htb_img/WEBRESOURCE32974b8a3b1a3daa61fdfa972b8a9be3image.png)

![](/htb_img/WEBRESOURCE0952c4426ad5dc8b832704977c2a52b3image.png)