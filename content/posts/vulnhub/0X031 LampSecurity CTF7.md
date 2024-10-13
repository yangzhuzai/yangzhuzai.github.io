+++
title = '0x031 LampSecurity CTF7'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



坏了，又不能自动获取IP地址，然后看了一下随靶机附带的文件，密码还是错的

![](/vulnhub_img/WEBRESOURCE166850e169c593dffba25e28fa031cc4截图.png)

只能进centos6的单用户模式进去修改配置文件了：[https://blog.csdn.net/qq_40576436/article/details/125164748](https://blog.csdn.net/qq_40576436/article/details/125164748)

DHCP半天获取不到，索性直接配置静态了，配置文件如下，复制了lo环回口的配置文件，修改了一些：

![](/vulnhub_img/WEBRESOURCEef61dfc71754f421d1e825906c0eb630截图.png)

# 端口扫描

![](/vulnhub_img/WEBRESOURCE20d083d70777bce78b8fa53c62f8d20e截图.png)

# 渗透

ssh和smb暂时没有发现东西，看来突破口应该是在80，901，8080，10000这几个HTTP端口：

说实话这台靶机有运气的成分在里面，web扫描都没用，因为是抽时间做的，

首先是注册框存在sql注入：

![](/vulnhub_img/WEBRESOURCEd6a086820bb2c996b4473638eaec971f截图.png)

然后我就直接上sqlmap跑了一下数据库，但是存在一个问题，这地方是注册的地方，导致数据库内容插入了很多脏东西，后面我就换了一个地方注入：

![](/vulnhub_img/WEBRESOURCEde63507c30ac9a453b4f101506bed750截图.png)

然后这个地方我试了几个ID，其中就有3，这里是个提示，说brian是专家，后面就用上了：

![](/vulnhub_img/WEBRESOURCEdd4431984a0afcb6db355a604fb70a73截图.png)

随后我继续注入，dump了所有用户，下面一坨就是注入加入的用户，删掉：

![](/vulnhub_img/WEBRESOURCE584c80a55c2955aa02b585cd4b541504截图.png)

把hash解密一下：

![](/vulnhub_img/WEBRESOURCE1df040a8d2101ce4c83677a9b260638b截图.png)

做成账号密码本，进行ssh爆破：

![](/vulnhub_img/WEBRESOURCEfc0db90b0b07cbd1b9e2574d5dd73c22截图.png)

这里爆破了一堆账号：

![](/vulnhub_img/WEBRESOURCEde5c2a4465dc9247b1a33092ca8934db截图.png)

# 提权

我随机登录了一个账号，看了一下sudo权限，然后看了一下家目录，确实是这么多用户，而且其中大部分是有密码的：

![](/vulnhub_img/WEBRESOURCE5733097a4a04d7b501d8755edb9b3742截图.png)

这个时候我想起来了先前sql注入的地方的ID，说brian是专家，有没有可能brian的权限会高一些，随后我就切换到brian去了：



![](/vulnhub_img/WEBRESOURCE7e60ab52d4fd0000b3a713fad9c8c100截图.png)

# 总结

后面看了一下红笔视频