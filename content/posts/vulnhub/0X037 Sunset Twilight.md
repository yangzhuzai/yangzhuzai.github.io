+++
ShowToc = true
TocOpen = true
title = '0x037 Sunset Twilight'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE27f14d745b2f0e154015f2be58cf76f5截图.png)

![](/vulnhub_img/WEBRESOURCE4c8e41b066c0f14495d135172bef00b8截图.png)

![](/vulnhub_img/WEBRESOURCEce027da8b90fba789bc868b2012e5a20截图.png)

# 渗透

21匿名登录失败：

![](/vulnhub_img/WEBRESOURCE5dafc474980e02adf7619137190fd50a截图.png)

mysql无密码直接登录成功，翻查敏感信息：

![](/vulnhub_img/WEBRESOURCEf2ee80d54fdc3dbbaa97963964ae99e1截图.png)

不允许写入：

![](/vulnhub_img/WEBRESOURCE6a8c5f13fef2d9beb6db5a94bd9b5993截图.png)

数据库也没有其他信息了，暂时先放一下，看一下smb，疑似根目录：

![](/vulnhub_img/WEBRESOURCEa9d0876f74b9b7eaf03897df23f9975a截图.png)

挂载下来，慢慢看：

![](/vulnhub_img/WEBRESOURCE4b78d0f155f7a79208a0410057de6fe4截图.png)

用户名：miguel

![](/vulnhub_img/WEBRESOURCE890831d187809ac88f6afee9593be6cb截图.png)

翻查敏感目录，发现web目录存在写入权限：

![](/vulnhub_img/WEBRESOURCEf87333824f34208bdf6868fdb3c6fe1f截图.png)

![](/vulnhub_img/WEBRESOURCEba201ad6ab6d29cccb526ea14d823f3c截图.png)

反弹shell:

![](/vulnhub_img/WEBRESOURCEb654ec6d76b7f6148d18325a63e3fbed截图.png)

# 提权

![](/vulnhub_img/WEBRESOURCE5680adbf5993927db6df3f7bec5129e4截图.png)

![](/vulnhub_img/WEBRESOURCE6890ad4950dfe525259db6d1e213e0cb截图.png)

![](/vulnhub_img/WEBRESOURCE251105d444290b22cfac1af86f62116e截图.png)

跑完一圈脚本后，咋感觉提权点有点多呢：

这个密码看起来是破解出来了，咋用不了呢：

![](/vulnhub_img/WEBRESOURCE230c38b38c06ef41ce9e30bbda642143截图.png)

算了，用openssl生成一个passwd的密码，然后修改passwd文件，就可以提权了：

```
root:$1$ZytY2Uoj$1.aU20M/0/xD4Wo1ymW9Z0:0:0:root:/root:/bin/bash
```

密码Abcd1234

![](/vulnhub_img/WEBRESOURCE3bdda34dbb832e5886736c5f9a5e8e08截图.png)

![](/vulnhub_img/WEBRESOURCEefaa079a7e85e4784a717f16b8e6a634截图.png)

![](/vulnhub_img/WEBRESOURCEb297ae3985c5f010566a23128e848a6d截图.png)

邪门儿，我咋先拿了root再拿的用户权限呢，我去看看其他人咋做的

其他人是web渗透拿到的，但是好像也没去拿用户权限

![](/vulnhub_img/WEBRESOURCE7a716d079a4ee49b77241618fb04e6d8截图.png)