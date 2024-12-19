+++
ShowToc = true
TocOpen = true
title = '0x064 Digitalworld.local(Development)'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE6e9d6767176575851400afe1f6e85bdfimage.png)

![](/vulnhub_img/WEBRESOURCE7bf405e2c4ceac84c782531af3b9b245image.png)

![](/vulnhub_img/WEBRESOURCEf8f5e7ca46baabf8b933a69bcb650b62image.png)

# 渗透测试

这个靶机不能扫，一扫就挂：

![](/vulnhub_img/WEBRESOURCE8c241f21891270aa43949935641c82ecimage.png)

![](/vulnhub_img/WEBRESOURCE7c35d64b7d2fda3c67bfce1fe9eb1fd0image.png)

8080端口

看这个的意思：

![](/vulnhub_img/WEBRESOURCE9af5b221cb6ca39f8413a2b8444fe09bimage.png)

发现信息：

![](/vulnhub_img/WEBRESOURCEe8b0fbbe017734f90269d83378ae7364image.png)

挨个儿看吧：

David

![](/vulnhub_img/WEBRESOURCEa94dfa19e088959e429a0d98c99fa51aimage.png)

发现信息：

![](/vulnhub_img/WEBRESOURCE42d71fbbe6224699733f8448ea71a510image.png)

![](/vulnhub_img/WEBRESOURCE9029157c3c1976199f8d60c4ede3ca1aimage.png)

访问这个目录，可以发现：

![](/vulnhub_img/WEBRESOURCE0899dba087f32136c79af1d535098cb0image.png)

那么，到现在，可能渗透才刚刚开始：

发现默认密码：

![](/vulnhub_img/WEBRESOURCEa8343fdf47b798b6084068059ab8ae78image.png)

发现疑似用户名

![](/vulnhub_img/WEBRESOURCEe7cd0fbd5443cbc3cfefe9b967e2cde1image.png)

然后反复尝试后，再次回到这个页面：

![](/vulnhub_img/WEBRESOURCEec529ddaade016e2a24be7f7754f0b08image.png)

google真好用：

![](/vulnhub_img/WEBRESOURCE34d58039876f1db76030259aa405f707image.png)

这里记录了两个洞，一个文件包含，一个信息泄露：

文件包含，试了半天好像用不了，信息泄露是可以的：

![](/vulnhub_img/WEBRESOURCEa7f707bace6266858b6c9924fe75d92cimage.png)

![](/vulnhub_img/WEBRESOURCE39bf19940e2a2671d5d7bc4f933cf99bimage.png)

试了一下intern成功了：

12345678900987654321

![](/vulnhub_img/WEBRESOURCE924dcc688efb9808a7b0fc88147e55c2image.png)

# 提权

受限的shell，想办法先把这个绕了：

![](/vulnhub_img/WEBRESOURCE1d1075423fabc8e115c920601b19c0f3image.png)

echo os.system('/bin/bash')

![](/vulnhub_img/WEBRESOURCE5cbd789b24d700fb67d8d6b064c873ffimage.png)

看看有哪些用户把：

![](/vulnhub_img/WEBRESOURCEed3f68cd096e51c3186f9add77d79b93image.png)

patrick的hash我们是知道的，只是刚刚没有解密：

![](/vulnhub_img/WEBRESOURCEa3cdd64a9942b3df381116da11b262d2image.png)

![](/vulnhub_img/WEBRESOURCE4913f72792890a3d2d0b427c783fc65cimage.png)

![](/vulnhub_img/WEBRESOURCE46f43658fa0fbb4ac96337b4ba415ce0image.png)