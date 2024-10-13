+++
title = '0X032 Aero'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCE3750b7f1fc2738082cd7871831d3edafimage.png)

![](/htb_img/WEBRESOURCEe0654ee3d610ebcd14d3a5ba9650ae07image.png)

![](/htb_img/WEBRESOURCEb6abefba514da9dd59d306ff8a843ea3image.png)

# 获取立足点

单从靶机的端口开放情况来看，只有80端口开放，web渗透获取立足点，或者有knockd？但是这不是linux，暂且看看吧，没有什么头绪，不是现场的框架，也没有其他端口开放，同时没有其他目录和域名，后面看了一下WP。

windows 主题 themes

是有个win11的主题漏洞，研究一下怎么打吧：

git clone [https://github.com/Jnnshschl/CVE-2023-38146.git](https://github.com/Jnnshschl/CVE-2023-38146.git)

![](/htb_img/WEBRESOURCE773a351fb7a3fcdd79d4258fc2f1cbf6image.png)

python3 -m pip install -r requirements.txt

![](/htb_img/WEBRESOURCEa8ce4703d341cf4a2f7355c08ecd11beimage.png)

sudo apt-get install g++-mingw-w64-x86-64

![](/htb_img/WEBRESOURCE8288500b6dee031de9e04ba372635e58image.png)

运行：

这里的IP和端口都是本地IP端口：

![](/htb_img/WEBRESOURCE789bb20b3b5d29bddb3ead726e61bd48image.png)

开启监听：

![](/htb_img/WEBRESOURCEd7cd34b427d7310495d28128b404d9efimage.png)

上传：

![](/htb_img/WEBRESOURCE19a69ea993df83c9cc54837c7de94b52image.png)

可能要多尝试几次：

后台应该是个计划任务，对于这些非服务端的攻击，其实真的很难有现成的复现资料：

![](/htb_img/WEBRESOURCEd6acd0c9b9b41a2e0eb889042889301aimage.png)

# 提权

![](/htb_img/WEBRESOURCE07735a3a4c81bcac2a8ec7b984f40cd0image.png)

那个ps1脚本看起来就是触发脚本，然后那个pdf的内容是再提示漏洞：

![](/htb_img/WEBRESOURCEb172412664b5050414524af622297decimage.png)

去github找现成的：

[https://github.com/duck-sec/CVE-2023-28252-Compiled-exe](https://github.com/duck-sec/CVE-2023-28252-Compiled-exe)

![](/htb_img/WEBRESOURCEc39591775bad6a6622f9ab5b84a67f52image.png)

使用msf生成一个木马，然后nc监听即可：

.\exploit.exe 1208 1 cxk.exe，比较简单

![](/htb_img/WEBRESOURCEbf99bb16e37461ce4aa58db5e114830aimage.png)