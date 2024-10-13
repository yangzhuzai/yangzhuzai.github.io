+++
title = '0X024 Intelligence'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCEf4f9356afcea710b2f0901fc2a4add22image.png)

![](/htb_img/WEBRESOURCEfa7745219d387cc800a4386832e34a59image.png)

![](/htb_img/WEBRESOURCE644bca0295f9adbcadfd2d37e73cf0fbimage.png)

# 获取立足点

根据端口开放情况进行判断，值得关注的端口：80、389、445、5985

先看445：

![](/htb_img/WEBRESOURCEfa6384699fc73d26bfc5c37730a2a038image.png)

389：

![](/htb_img/WEBRESOURCEa1db95b74dea5f6707387458a8d69784image.png)

那就只剩下80了：

![](/htb_img/WEBRESOURCEe55cbf3d3369f5348c7378bae8482f7dimage.png)

只有一个documents目录，whatweb识别一下：

![](/htb_img/WEBRESOURCEbbe3541eb52e7d4d102a8bc7d586f862image.png)

但是好像也没有历史漏洞，再看看这个pdf文件，发现可以枚举：

![](/htb_img/WEBRESOURCE7485d8a1d87462f40454ba059af38274image.png)

不能挨个看吧，我挨个看了：

[http://10.10.10.248/documents/2020-06-04-upload.pdf](http://10.10.10.248/documents/2020-06-04-upload.pdf)

默认密码：

NewIntelligenceCorpUser9876

![](/htb_img/WEBRESOURCE03c57ed047ab86d62173da91b40be201image.png)

拿到密码，想到的就是密码喷洒：

但是使用自带的字典失败了：

![](/htb_img/WEBRESOURCE6725a70ab9c274d66493a7f6d799ae72image.png)

但是有个地方是存在账号的，那就是文件的撰写人:

写个bash脚本，下载了2020年的所有文件：

![](/htb_img/WEBRESOURCE059979f0e61bd1b4e32d4910b8baecb2image.png)

这里明白了，有相关工具如果提供了详细参数的，建议就使用详细参数看看吧：

应该是工具或者环境的问题，这里没法爆破成功：

![](/htb_img/WEBRESOURCE94978b586d88e74a2b82e75734479c35image.png)

带上详细参数看看：

并非是密码错误，也是就说这个账号密码应该是正确的，只是请求kdc出错了：

![](/htb_img/WEBRESOURCE58ec1d32c8aca8572c6d76ff5de63726image.png)

拿到了账号密码，但是看了一下

ldap和winrm都不能登录：

![](/htb_img/WEBRESOURCE1a1d57adb84c7ea3de1f7411e9b48fa1image.png)

试试smb:

![](/htb_img/WEBRESOURCEbb836cee37273e3f81a3a46b0c5b5bbeimage.png)

大概看了一下这个脚本，他说每5分钟执行一次，请求一些东西，但是怎么利用呢，猜测是relay，再接着看：

发现flag:

![](/htb_img/WEBRESOURCE5c55f4785d655114866dd7675f33200bimage.png)

后面看了一下wp，确实知识面欠缺，需要添加一个dns解析，而且域内是默认可以添加的，有合法身份即可：

添加dns记录后，relay:

![](/htb_img/WEBRESOURCEf7c343b450a1772e87fc1e735ee208b6image.png)

破解：

![](/htb_img/WEBRESOURCE1a504314810bc96ac98c94b0275adaafimage.png)

不让登录：

![](/htb_img/WEBRESOURCEccd1611c31186d956fef5100cf44069cimage.png)

不过不要紧，正常渗透就行了：

![](/htb_img/WEBRESOURCE6eb416c86bb9f58e7aab3e73c7fc2d33image.png)

# 提权

bloodhound已经给出了比较完善的路线了：

python gMSADumper.py -u 'Ted.Graves' -p 'Mr.Teddy' -d 'intelligence.htb'

Users or groups who can read password for svc_int$:

> DC$itsupportsvc_int$:::e0dcda8d93bf71a6352ea7803c8f17f1svc_int$:aes256-cts-hmac-sha1-96:fd6235dbfd8a560d17433b22022633ed7188588277cf4d174f6582daf2c5333fsvc_int$:aes128-cts-hmac-sha1-96:059ae234e725682d00c3c278b3cff01b


![](/htb_img/WEBRESOURCE23bd53023714afa90b7a4ad79ef38ffeimage.png)

![](/htb_img/WEBRESOURCE68ba101c7452c60115bc54828954fcf9image.png)

![](/htb_img/WEBRESOURCE5798f447a5a945b87a6a939809ce6e56image.png)