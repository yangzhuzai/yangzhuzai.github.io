+++
title = '0x032 lazysysadmin'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



这个靶机的网络环境建议直接使用自带的NAT，折腾了一会儿桥接，算了

# 端口扫描

![](/vulnhub_img/WEBRESOURCE72d08c1b1b33442a24aefef7e2dd85e2截图.png)

![](/vulnhub_img/WEBRESOURCE913fadb661d1be6ad997f2b8d151d3fd截图.png)

![](/vulnhub_img/WEBRESOURCEce67c11b54ab1775c3260d79679abae9截图.png)

![](/vulnhub_img/WEBRESOURCE1b7c82ea0bef70c52d8f88983586720d截图.png)

# 渗透

大概看了一下开放端口和脚本扫描情况，大部分是常规端口，比较让人在意的是6667端口的irc服务，非常规服务，搜索了一下如何利用：

试了几个payload都不行：

![](/vulnhub_img/WEBRESOURCE648c582fd1610675ca160aeca15f5046截图.png)

smb:

![](/vulnhub_img/WEBRESOURCE5348002957b2fe8528d516f5d6f5bd4c截图.png)

挂载到本地慢慢看：

![](/vulnhub_img/WEBRESOURCE3d4c787b2b3d27b9471048a0a7995ef4截图.png)

疑似可用信息：

![](/vulnhub_img/WEBRESOURCE5387be05bf182af18b80f053abc6a20c截图.png)

![](/vulnhub_img/WEBRESOURCE5c390678d56ad9bb45609de099ca2ae2截图.png)

![](/vulnhub_img/WEBRESOURCEc1a4aa7ddae6bb4905197064fa054fe4截图.png)

mysql passwd:

admin

TogieMYSQL12345^^

![](/vulnhub_img/WEBRESOURCE3cb813f0b54a85fa2d08ea6896737f55截图.png)

但是不让连：

![](/vulnhub_img/WEBRESOURCEf3dc66aee2cd2605d826cde9cddbbf1e截图.png)

那就web渗透吧：

![](/vulnhub_img/WEBRESOURCE99bcc89ff554b4ba8f30a175111e1016截图.png)

wp疑似用户名？

![](/vulnhub_img/WEBRESOURCEc7e55b84059143f8886483c6a8a61144截图.png)

wpscan扫一下：

两个用户名没有插件：

![](/vulnhub_img/WEBRESOURCEf4a5cce86e62459ed90cbb1e111868d7截图.png)

用刚刚收集的密码试了一下，果然，毕竟靶机名字叫lazysysadmin

![](/vulnhub_img/WEBRESOURCEc03b87b06cb11c44d26db41edc00afd9截图.png)

后台getsghell：

[http://192.168.1.138/wordpress/wp-content/themes/twentyfifteen/404.php](http://192.168.1.138/wordpress/wp-content/themes/twentyfifteen/404.php)

![](/vulnhub_img/WEBRESOURCEa206dfd6e8a9dc0d5803560d01bd0047截图.png)

# 提权

额，早该想到的，lazysysadmin

![](/vulnhub_img/WEBRESOURCE3049624f0d62c6b7e14d39312d167d3b截图.png)

![](/vulnhub_img/WEBRESOURCEbb5af2dc8c822faf67f910aa63b5bba2截图.png)

果然懒啊，sudo权限这样给：

![](/vulnhub_img/WEBRESOURCE81f08b76f4f9d223db807d8d1270b253截图.png)