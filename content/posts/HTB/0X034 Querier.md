+++
ShowToc = true
TocOpen = true
title = '0X034 Querier'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCEc2717bb159fbd9f01be8956292b90b01image.png)

![](/htb_img/WEBRESOURCEf5892ddf82aac0abf2a1a26cb399cf97image.png)

![](/htb_img/WEBRESOURCE6f9de32208211aadbf0563a45c011cfbimage.png)

# 获取立足点

值得关注的端口：445、1433、5985

445：

![](/htb_img/WEBRESOURCEa4313eb9098171061865f00046ed4d3cimage.png)

用户名：Luis

![](/htb_img/WEBRESOURCE79b050ac91ab01092e5a2f356f179652image.png)

不知道为啥，打开是空的：

![](/htb_img/WEBRESOURCE6972a42e706e1d8e28bfbc761afa1715image.png)

但是在windows中打开，发现好像是存在宏：

![](/htb_img/WEBRESOURCE20058768812ccb032c9d77dc357dceabimage.png)

使用unzip工具进行提取即可，在提取出来的文件中，可以发现：

reporting

PcwTWTHRwryjc$c6

![](/htb_img/WEBRESOURCEacfdd31640226a545cdc7aac5ec6adbfimage.png)

登录成功：

![](/htb_img/WEBRESOURCE2645d847fb0ba9683418e008d9f8a268image.png)

xp_cmd无法开启，试着中继了hash:

![](/htb_img/WEBRESOURCEe4c3fa49ff4d10b6a0229088f2fb9326image.png)

破解成功：

corporate568     (mssql-svc)

![](/htb_img/WEBRESOURCEd95c95fba89cec4719cb6873eb751b7bimage.png)

但是登录失败了：

![](/htb_img/WEBRESOURCE6c179ac5ea28ad2a0a9fe13a61ed64d7image.png)

试试psexec，也失败了：

![](/htb_img/WEBRESOURCE2a4c206b15df931e7c084fc0208ca632image.png)

使用新的账户登录MSSQL：

xp_cmd开启成功：

![](/htb_img/WEBRESOURCEb59b637dc5aaada4c8e8111d337c35c3image.png)

通过smb共享，反弹shell回来：

![](/htb_img/WEBRESOURCE5a42c25c9604d7fc6daef3e974af42fbimage.png)

# 提权

winpe,

看起来是打了补丁的，但还是看一下有没有比较好提权的把：

![](/htb_img/WEBRESOURCE6980fc50ca65e6e95890cb5887e988c7image.png)

好像没有，看看其他的：

![](/htb_img/WEBRESOURCE2beb04540ad6d45b006dac833e985fecimage.png)

简单查了一下，好像是可以提权的：

![](/htb_img/WEBRESOURCEe8293bbd2cc028f37bc552d77eade6d6image.png)

接着看：

![](/htb_img/WEBRESOURCEd0eb0cfe25d2a337ca4cafc19b1e53ccimage.png)

失败了：

![](/htb_img/WEBRESOURCE73a46b63097aabd443f5dfa829704e42image.png)

psexec，成功了：

![](/htb_img/WEBRESOURCEa6258c6dd99b0c4f2c0b9ffed6ebeb76image.png)