+++
title = '0X002 Arctic'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCEf6d72da494f38de5fed08804d34a61c3image.png)

![](/htb_img/WEBRESOURCE833aee3fd7f537d2c3b8a04e9f154944image.png)

![](/htb_img/WEBRESOURCE53d7733878f164ca14703d16760544e1image.png)

# 获取立足点

端口扫描没有发现什么东西，使用浏览器看看：

在8500端口发现目录：

![](/htb_img/WEBRESOURCE41b83d461cb7a5a03146d8fa1eb8d1afimage.png)

多点几个，就可以发现一个登录页面：

![](/htb_img/WEBRESOURCE2541ac8fbc25bd846fc4973eb39cb6f4image.png)

![](/htb_img/WEBRESOURCE94bcf131227a9d45707970a4b861a941image.png)

简单修改一下：

![](/htb_img/WEBRESOURCE89c15fee79f0f23d1e5d9b727b13a407image.png)

getshell:

![](/htb_img/WEBRESOURCE53d6caae5bd8100a6f39643b67ab2aadimage.png)

user.txt

![](/htb_img/WEBRESOURCEb942387a31d460081eefbb1455493548image.png)

# 提权

![](/htb_img/WEBRESOURCE218a61c3eda589112c633a28f358c0a6image.png)

![](/htb_img/WEBRESOURCEfb0873c52764bdcd00533bb59327ba5aimage.png)

脚本跑一半卡住了：

![](/htb_img/WEBRESOURCE5c4e536796440628e46856af9c8fd32bimage.png)

暂且试试这个吧：

找了很多地方，最后在github找到了这个：

[https://github.com/zcgonvh/MS16-032/releases/tag/Release](https://github.com/zcgonvh/MS16-032/releases/tag/Release)

![](/htb_img/WEBRESOURCEeb78205aafb5cd8aa28bdf152bbbd822image.png)

![](/htb_img/WEBRESOURCE3bf028759e5e97dc57898d9c1a8f18ceimage.png)

![](/htb_img/WEBRESOURCEce39594a2e63d6e20c40de5570dba99eimage.png)