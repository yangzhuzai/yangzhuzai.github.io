+++
ShowToc = true
TocOpen = true
title = '0X015 Sauna'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCE49ac89b8d9fc975a54b7bbd64c81db25image.png)

![](/htb_img/WEBRESOURCEdf27ea5a3375f8849e59ef7135199357image.png)

![](/htb_img/WEBRESOURCE9051728dcb75e68b87fda147a0f15623image.png)

基本信息：主机名：SAUNA，域名：EGOTISTICAL-BANK.LOCAL

# 获取立足点

大致看了一下端口开放情况，主要是445\389\5985\80比较关键：

先看80：

看样子是个金融贷款公司，可能有漏洞？

然后是445，不存在匿名访问：

![](/htb_img/WEBRESOURCEf395e6bfb4e6a5391c26bc246f0182caimage.png)

然后是389，看看有没有匿名枚举，有的：

![](/htb_img/WEBRESOURCEe6e0627ba3dd46051579f4fd3135285cimage.png)

试试:

![](/htb_img/WEBRESOURCE43345cdd445fe6117b1b694db608d3d9image.png)

失败了：

![](/htb_img/WEBRESOURCEf8a1b0a0822276cbe6ce2f1bfc8ebac3image.png)

然后用kerbrute看看这些用户是否是可用的，发现没有可用的用户，那么只能说ldap泄露的账户不完全正确：

![](/htb_img/WEBRESOURCE48192919b79c49373f50b68bbcdaeeb1image.png)

那么只能试试cewl获取的信息是否正确了，有一个用户：sauna

![](/htb_img/WEBRESOURCEcd3e18432c5305db44589e2b48082706image.png)

这个用户也失败了：

![](/htb_img/WEBRESOURCE4d8ac5327052431929313383e3827decimage.png)

cewl可能不太行，试试手动：

首字母+名字

![](/htb_img/WEBRESOURCEe1ed4fa76f0606bf81040ab0b065c13eimage.png)

AS-REP成功：

![](/htb_img/WEBRESOURCE99cb7b56363e8a08f851455e378a73aaimage.png)

破解hash:

Thestrokes23     ($krb5asrep$FSmith@EGOTISTICAL-BANK.LOCAL)

![](/htb_img/WEBRESOURCEbf086419fe9e18a0f6e1a78b3aeab7c1image.png)

由于5985的存在，可用winrm:

![](/htb_img/WEBRESOURCE5a8c2e2dca84e63b0034bc97d00fb825image.png)

# 提权

bloodhound和winpeass操作：

疑似可以提权，但是好像没有配合的地方：

![](/htb_img/WEBRESOURCEcd7c2c92a2ab22d533cbd50fb06894caimage.png)

然后看看bloodhound，不知道为什么，使用exe输出的内容无法上传分析，使用py 生成的可以：

![](/htb_img/WEBRESOURCE24daefd682c0181e3f3a5c3bf89b4ae5image.png)

那么关键就是他了：

SVC_LOANMGR

![](/htb_img/WEBRESOURCEdc67e192d7681daccaf2764dd61b32c1image.png)

Kerberoasting失败：

![](/htb_img/WEBRESOURCEbfa3e719d7c02949707bc314de0fa788image.png)

再仔细看看内容：

原来有密码：

DefaultDomainName             :  EGOTISTICALBANK

DefaultUserName               :  EGOTISTICALBANK\svc_loanmanager

DefaultPassword               :  Moneymakestheworldgoround!

![](/htb_img/WEBRESOURCE07a0c6bb3d457a0bb5f9a836a568077dimage.png)

我服了，这里没注意看，硬是没发现，名字不一样，我拿着上面的名字一直打，一直打不成功：

![](/htb_img/WEBRESOURCE8ad97574bbbea4f30297e01d90aca4feimage.png)