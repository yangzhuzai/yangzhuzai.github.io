+++
title = '0X035 Sniper'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCE20b61384be2dcd3300751271b979708fimage.png)

![](/htb_img/WEBRESOURCE94818edea997aaa6c02110672ea8a3e7image.png)

![](/htb_img/WEBRESOURCE32f6533e95fc2a071e14a07191136763image.png)

# 获取立足点

值得关注端口：80、445：

445：

![](/htb_img/WEBRESOURCE561c3c4f7e0f862188ee76a9da3fa392image.png)

80：

![](/htb_img/WEBRESOURCE3e935eb2a42bcede5cdca66ad468fb72image.png)

![](/htb_img/WEBRESOURCE0854af8da2254fde4e8682825de83667image.png)

发现文件包含：

![](/htb_img/WEBRESOURCEb4e4391a1d22e7fc62bec8c352c7efe8image.png)

经过测试远程文件包含不存在，那么该如何getshell呢？

读取session文件：

![](/htb_img/WEBRESOURCE715f79d4a6643651f118b5bb71966cb8image.png)

发现里面有我们的名字，于是，我们可以在这里面写入webshell，反弹shell:

大概就是传入nc，然后反弹：

![](/htb_img/WEBRESOURCEea4b53a25034134e89de1cd3c119e533image.png)

数据库账号密码：

![](/htb_img/WEBRESOURCE5e49f988b69e8b1621324f9b2e7114ebimage.png)

这个密码是Chris用户的密码：

![](/htb_img/WEBRESOURCEca5132517c6753321883ec10b92a0450image.png)

使用powershell弹个Chris的shell回来：

直接调用temp里面的失败了：

![](/htb_img/WEBRESOURCE61c9060dafa18695af5b56a0b89e8839image.png)

得重新下一个到Chris的目录里面：

![](/htb_img/WEBRESOURCE99c632406bcb6160a0721975c2ae5c93image.png)

# 提权

常规提权思路无果，需要信息收集：

![](/htb_img/WEBRESOURCE03f9e29a5be930b0cb46796178fa777cimage.png)

其实这里的意思是，让我们做一个帮助文档，然后会有人查看这个文档，结合downloads里面的chm文档，上面应该放置的就是这个文档，我们需要利用chm做一个payload：

![](/htb_img/WEBRESOURCEb7cca88758e4558e97a83032bc41616aimage.png)

后续使用htmlhelp.exe来进行制作，然后就可以获取到administrator的ntlmhash进行破解即可：

![](/htb_img/WEBRESOURCE70e8271e92b629901057130b90db7884image.png)