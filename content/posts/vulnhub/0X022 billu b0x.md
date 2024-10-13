+++
title = '0x022 billu b0x'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCEb16b1c1751f84b969df6604d5a1e8ca6截图.png)

![](/vulnhub_img/WEBRESOURCE7d996f5b20ee5336741028b9be2db0de截图.png)

# getshell

### 方法一、代码审计

首先有个坑，burp改POST不行，必须得浏览器才能触发文件下载：

这里可以知道web目录：

![](/vulnhub_img/WEBRESOURCE8d6398eb6954ccd1aa2b35d7c90d534c截图.png)

下载登录主页的源码：

![](/vulnhub_img/WEBRESOURCEeb258297dd772074ae4fc728a74f1ee2截图.png)

审计：

![](/vulnhub_img/WEBRESOURCE43509cd1947b9c62a3136bda0f4ef53f截图.png)

我打眼一看是过滤了单引号’     然后我就尝试了对单引号进行绕过，16进制、base64、两次URL编码都以失败告终，然后这里的判断逻辑是，sql语句查询出来有内容即可。

最后无奈看了WP

payload是：

password: \

username: or 1=1 #

带入到代码里面其实是这样的：

```
$run='select * from auth where  pass=\''.$pass.'\' and uname=\''.$uname.'\'';
带入后：
select * from auth where pass='\' and uname='or 1=1
```

这里要注意，将sql语句和php分开来理解，要不然会比较麻烦，但是不好理解的是，讲道理这里直接让password=or 1=1 #\就可以实现了啊。后面借着phpmyadmin试了一下，可以清楚的看到闭合结果：

![](/vulnhub_img/WEBRESOURCE2fa0bf7c770bd4ce88620d4fdc95bdf9截图.png)

这样就非常好理解了因为这样的闭合方式，'' and uname='成为了password的查询方式，后面没有查询username，而是直接or 1=1，也就是说必须由password传递转义符\，必须由username传递注释符号#

然后回到登录后的页面，看起来是有文件上传的，但是直接上传被拦截了，好吧，再次代码审计：

![](/vulnhub_img/WEBRESOURCE57bb2b1f89278bbb717fd406939b09b7截图.png)

判断文件的代码，白名单，基本无法绕过：

![](/vulnhub_img/WEBRESOURCEd5df3ea123376f27b54fac74fc9073d1截图.png)

然后又去看了一下PW，好好好，确实没发现这个地方可能有文件包含，因为数据包中并没有文件后缀，看起来像是传递参数，但是看了一下代码，是文件包含：

![](/vulnhub_img/WEBRESOURCE38ebf5aaa3610f2477df9b8015f9e4c7截图.png)

图片马getsgell：

![](/vulnhub_img/WEBRESOURCE743c6855b2e9b184524bf4c0cb63c81f截图.png)

### 方法二

开局扫描的时候，没有扫出来phpmy，两个方法解决，1是使用dirb的big.txt，二是使用dirsearch的默认参数即可

![](/vulnhub_img/WEBRESOURCE1331932f49ccb2d829bb9a148bc7b08b截图.png)

然后在文件包含时，会发现两个php文件，将他们也下载下来：

![](/vulnhub_img/WEBRESOURCE5d7e136115a110a2a9783c87d79faac0截图.png)

就可以发现数据库账号密码

![](/vulnhub_img/WEBRESOURCE3340637884f8dea580ac2d133e596b5b截图.png)

然后就可以登录phpmyadmin了，直接提权是没有权限的，所以就直接看账号密码吧

![](/vulnhub_img/WEBRESOURCE5db30f6da36da70ea7097c3c1d363edf截图.png)

# 提权

#### 方法一、

同样有GCC，内核提权吧：

![](/vulnhub_img/WEBRESOURCEf8e751f1a907905da2e24099e54208d7截图.png)

#### 方法二

当然，该方法理论上在web渗透的时候就可以文件包含来getroot权限，路径为/var/www/phpmy/config.inc.php：

![](/vulnhub_img/WEBRESOURCEe09413ffb1da480b86ef91b9e4ccb8f0截图.png)

![](/vulnhub_img/WEBRESOURCE6a565fd800115957e9ba4c70a33f56fa截图.png)

![](/vulnhub_img/WEBRESOURCE469903c99348ff639cd2e688cb2a7886截图.png)

# 总结

这台靶机是目前我比较喜欢的一台靶机，思路各种各样，打法也千奇百怪，非常锻炼思维。