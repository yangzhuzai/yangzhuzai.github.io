+++
title = '0x061 Bsides Vancouver 2018'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCEd4a208d5ba6cdb952ee2bedc82c80b5bimage.png)

![](/vulnhub_img/WEBRESOURCE939d567f974e37d69e6998aaf8980c32image.png)

![](/vulnhub_img/WEBRESOURCE30bd3b6720afb4a6c2397fb4013ed596image.png)

# 渗透

21匿名登录：

![](/vulnhub_img/WEBRESOURCE5fc4d760efb2900c10a83966e3c8fb12image.png)

用户名：

![](/vulnhub_img/WEBRESOURCE98c80455ce49bcda563fcde343d435afimage.png)

ftp切换失败：

![](/vulnhub_img/WEBRESOURCE3d34aa3f26a3a2ce913f086334c567bdimage.png)

80端口

目录扫描

![](/vulnhub_img/WEBRESOURCEfaa345832def0cdf643ea5cd033fd356image.png)

发现wordpress

![](/vulnhub_img/WEBRESOURCE7174001dca5312c001b2bf8248f466a0image.png)

wpscan

用户

![](/vulnhub_img/WEBRESOURCE96ed499c674f2558315d7e380e87b1baimage.png)

插件

![](/vulnhub_img/WEBRESOURCEf2bf6ba3cc367d65b39a9209567e00baimage.png)

试了一下刚刚的文件：

![](/vulnhub_img/WEBRESOURCE8cfab4f813095664bca44462cce0f930image.png)

不行

![](/vulnhub_img/WEBRESOURCE3e7b3acad6161d06b02b32c0e84d4c87image.png)

上rockyou

![](/vulnhub_img/WEBRESOURCEcd4dfdfb8aa8fa5b6669fc8bd717b9e9image.png)

![](/vulnhub_img/WEBRESOURCE53fcf80257d3245a8ac84fa438576486image.png)

# 提权

sudo -l没有密码

SUID发现可疑文件：

但是似乎用不了：

![](/vulnhub_img/WEBRESOURCEce4824daee28ea08c183383d1a83a660image.png)

发现五个用户：

和ftp里面的一样

![](/vulnhub_img/WEBRESOURCE9d06815302c181ea6a67dfc46c1b5c07image.png)

数据库信息：

![](/vulnhub_img/WEBRESOURCE221591cc45b9c5410b305b810f592080image.png)

但是这个密码所有用户都用不了

![](/vulnhub_img/WEBRESOURCE8a641193b29534192d4d91588b6002d7image.png)

上脚本:

计划任务：

![](/vulnhub_img/WEBRESOURCE954743f866eace0c6b2e697dcaf46381image.png)

![](/vulnhub_img/WEBRESOURCEc84f4adb368542c99c32d97b6566f56bimage.png)

![](/vulnhub_img/WEBRESOURCE675bcdef1b815f9b99edf390413cc2ddimage.png)

![](/vulnhub_img/WEBRESOURCE5f9faf406223032eed55a89fd9afd0f9image.png)

内核提权：

![](/vulnhub_img/WEBRESOURCE4f01156ff9c0d4691a883af7d8958917image.png)