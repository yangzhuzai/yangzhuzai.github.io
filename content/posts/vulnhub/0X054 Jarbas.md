+++
title = '0x054 Jarbas'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE38785096148b8e5db673cb8e0a12a48aimage.png)

![](/vulnhub_img/WEBRESOURCE96e0162121af7aa516be4203f61ef1b8image.png)

![](/vulnhub_img/WEBRESOURCE4153e59ef9980813fd6c8c0c603c047aimage.png)

# 渗透测试

80端口

说实话，多次感觉gobuster的中等字典，咋还没dirsearch的默认字典好用呢：

![](/vulnhub_img/WEBRESOURCE022266c6cfe43babc606c2ab5cbd5b7dimage.png)

![](/vulnhub_img/WEBRESOURCE9f9b29c59f56d410384fdd7c080f9701image.png)

![](/vulnhub_img/WEBRESOURCEf009f6657d339445153be758f70c74ecimage.png)

mysql和ssh失败了：

![](/vulnhub_img/WEBRESOURCEb327c8056211b7ccf7bead9a0c456d2fimage.png)

看看8080端口

使用eder/vipsu登录成功：

![](/vulnhub_img/WEBRESOURCE3b42fda86ab2c833ad155d2d38c47ff2image.png)

![](/vulnhub_img/WEBRESOURCE558426b14403f96ae3f676fc6a5f0013image.png)

# 提权

2021-4034

![](/vulnhub_img/WEBRESOURCE22d44548d40ed578bf49ca27ced9b3fcimage.png)

作者留下的提权方式：

![](/vulnhub_img/WEBRESOURCEfb2acbbd44019f0997132d74052b4655image.png)

![](/vulnhub_img/WEBRESOURCE58114db794bd63b828702a17d852bf86image.png)

![](/vulnhub_img/WEBRESOURCE94c440b6b6ea4410c9a21974e7de6fcbimage.png)

![](/vulnhub_img/WEBRESOURCE69bc17edc3990b28ffe2ad5f86684bb5image.png)