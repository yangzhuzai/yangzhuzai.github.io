+++
ShowToc = true
TocOpen = true
title = '0X018 matis'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCE3d970b6a64ceb598de9c74b2f94023ceimage.png)

![](/htb_img/WEBRESOURCEf0a3b62c138df63b319e3dff3916bca1image.png)

![](/htb_img/WEBRESOURCE1dbd01021a15c5e120cb0eb10a714b3cimage.png)

# 获取立足点

值得关注的服务：smb\ldap\http\mssql

smb

![](/htb_img/WEBRESOURCEf38b5e691dd72fe96c35f2103d829abaimage.png)

ldap

![](/htb_img/WEBRESOURCE8ab1d1ac0bda3c32a45f81e853ba38a7image.png)

看8080了：

![](/htb_img/WEBRESOURCE1b155670fd396ef80562714c70a830adimage.png)

是两个xss：

![](/htb_img/WEBRESOURCE81e7b16c4e1676a9e4102ce64f00e7fdimage.png)

再看1377：

打开是默认页面，目录扫描发现：

![](/htb_img/WEBRESOURCE582bb4244a55f2c76bf1042a2e0800c2image.png)

管理员名为admin：

![](/htb_img/WEBRESOURCEd737440b28e8ed71794a1f53a1f9fa36image.png)

文件名即为密码：

![](/htb_img/WEBRESOURCEa8c8db80985487890c1077cb6b3b7b38image.png)

![](/htb_img/WEBRESOURCEdc831e0ec4488d069f2d399e09da4b70image.png)

那么这个密码，可能是web、sql、ldap的：

web不行

![](/htb_img/WEBRESOURCE016642942a591e30ef1e0c7a56cb7fb3image.png)

ldap没有账户，试试mssql先：

成功了：

![](/htb_img/WEBRESOURCE23c3f86220690713e83de14c2b2f1948image.png)

这里几个思路：1、收集数据库里面的数据；2、中继

中继出来，发现不让用：

![](/htb_img/WEBRESOURCEf0c47c4eb493491ef0bcbe77fca1c980image.png)

![](/htb_img/WEBRESOURCE169245b6c62c284503376520afca5e2dimage.png)

第二次进行尝试的时候发现：

![](/htb_img/WEBRESOURCEb1eac2c59199c4587cbfaaf927ed5325image.png)

xp_cmd也不让用，只能看看敏感信息了：

和user有关的表就三个，查起来很快：

james                J@m3s_P@ssW0rd!

![](/htb_img/WEBRESOURCEddda5b09f0c922d0aa24c2596f1c07e8image.png)

再次获得账户名密码，同样的，思路有几个，1、web；2、smb；3、ldap

smb看来没什么东西：

![](/htb_img/WEBRESOURCE8c13136459512adf64739a76e6190a23image.png)

web也登录失败：

![](/htb_img/WEBRESOURCE52f383acbeb2c8257997bae84fd5ccf3image.png)

只剩下ldap了，直接使用bloodhound来看看吧：

这个用户是可以登录的，但是没有3389和5689开放啊：

![](/htb_img/WEBRESOURCE6d0ea9fcab56aac87c67e0825c55b75cimage.png)

好像也没有什么好的线路了：

![](/htb_img/WEBRESOURCEd55be7a59456eefba8308b1686795c30image.png)

根据目前的情况，无法登录，如果使用kerberoasting等方式也无法登录，那么只能想办法使用当前身份，直接获取到管理员的方式来获取权限：

ms14-068：

通过rpc获取sid:

![](/htb_img/WEBRESOURCE7eb88e22e23849f61aa7554faa1f4905image.png)

impacket:

python ../impacket/examples/goldenPac.py -dc-ip 10.10.10.52 -target-ip 10.10.10.52 htb.local/james:'J@m3s_P@ssW0rd!'@MANTIS.htb.local

![](/htb_img/WEBRESOURCEb680abd3e46e72a8a0dde6b38680982aimage.png)