+++
title = '0x018 dpwwn 2'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



该靶机开机可能遇到当前硬件版本不支持设备“sata”。的提示，修改兼容性即可

![](/vulnhub_img/WEBRESOURCE5e8448c17e7dbc6f12d292a73b987dfe截图.png)

![](/vulnhub_img/WEBRESOURCE2c4e548f321bc7003e8021136ef5ac6e截图.png)

好好好，搞了半天进不去grub，去vulnhub上看了一下，主机是静态IP，10.10.10.10，算了我改VM网段好伐

# 端口扫描

![](/vulnhub_img/WEBRESOURCEe4cbea163f3b93afa0c6463ca16f409e截图.png)

![](/vulnhub_img/WEBRESOURCEb26dc0d1cc89188c2285171de1576623截图.png)

# 渗透

![](/vulnhub_img/WEBRESOURCE201181ef39dab2bfdb67919c9e137c04截图.png)

可以发现是dpwwn02的家目录，但是没有写入权限，

![](/vulnhub_img/WEBRESOURCEfcf393a4901f21a9e7fddfcd89f340e5截图.png)

web

![](/vulnhub_img/WEBRESOURCEd30195d348e34f265adaf019d628abd4截图.png)

wpscan

插件：

![](/vulnhub_img/WEBRESOURCEb60bbfd4d88ae45387f31b37466db3fd截图.png)

用户名：

![](/vulnhub_img/WEBRESOURCE57e0f16b3786c2f84f40c34f292f6a4c截图.png)

存在任意文件读取，但是不是远程文件读取：

![](/vulnhub_img/WEBRESOURCE8e4d591203dca720b457726197741a9e截图.png)

后面再次看了一下nfs，给了sudo成功写入了文件，配合文件包含即可getshell

![](/vulnhub_img/WEBRESOURCE685cd7686c5524ecfc7a7cb4a2feb216截图.png)

![](/vulnhub_img/WEBRESOURCEfea5a81fb7a15cdc7b3c294d53e16f2b截图.png)

# 提权

比较简单

![](/vulnhub_img/WEBRESOURCEa8767a10b626be9e072be731dfd560ab截图.png)