+++
title = '0x035 sunset decoy'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE099bd507449b73081ce8f37bb0c3b667截图.png)

![](/vulnhub_img/WEBRESOURCE4ed549d6c8935a50f5a62ae9788818d1截图.png)

![](/vulnhub_img/WEBRESOURCE712433f17ff09c37753a77f69697e0a3截图.png)

# 渗透测试

目录扫描没发现东西，

![](/vulnhub_img/WEBRESOURCEd227bd235445cdae1bb35ab4796d32e4截图.png)

![](/vulnhub_img/WEBRESOURCEba3ec5c726f1c67862a1026106044d8f截图.png)

然后看了一下，是目录浏览，没有继续扫的必要了：

![](/vulnhub_img/WEBRESOURCE7cefd82800ac58b0887f66f81d8f9155截图.png)

需要密码：

![](/vulnhub_img/WEBRESOURCE56ddbc7b1f25e43d175564d98016075b截图.png)

破解

![](/vulnhub_img/WEBRESOURCE059f6a976e7736b356a4c26d5a786783截图.png)

![](/vulnhub_img/WEBRESOURCE99d9883f9e9f42e234f6d84c8558e272截图.png)

shadow账号就这两个，破解一下试试把：

![](/vulnhub_img/WEBRESOURCEd0117c14613feb84e5fe4ddf79949c34截图.png)

爆破shadow密码：

```
john shadow --format=crypt --wordlist=/usr/share
```

出现密码:

296640a3b825115a47b68fc44501c828

server

![](/vulnhub_img/WEBRESOURCE5249ccc4c52791fa67f13347c10e98f2截图.png)

登录成功：

![](/vulnhub_img/WEBRESOURCE5ac0ac186ebba33b81881c96ed937cfb截图.png)

# 提权

上来发现是受限的shell，想办法逃逸一下：

发现文章：

[https://blog.csdn.net/qq_43168364/article/details/111830233](https://blog.csdn.net/qq_43168364/article/details/111830233)

```
sh 296640a3b825115a47b68fc44501c828@192.168.5.101 "bash --noprofile"
```

![](/vulnhub_img/WEBRESOURCEb8036cbe20c42bf6425eef9ca19517b3截图.png)

一个哑shell，升级成交互式shell，然后环境变量又没了，恢复一下

```
export PATH=$PATH:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/games:/usr
```

![](/vulnhub_img/WEBRESOURCEcdeb4b007f8bf7bdb104ae14a4a7f48a截图.png)

SUID和sudo无果，有GCC，来个内核提权吧：

CVE-2021-4034秒了

![](/vulnhub_img/WEBRESOURCE8bf37b242e62ef6b76bfc2232d2e165c截图.png)