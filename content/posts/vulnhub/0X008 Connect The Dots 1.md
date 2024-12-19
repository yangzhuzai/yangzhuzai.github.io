+++
ShowToc = true
TocOpen = true
title = '0x008 Connect The Dots 1'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE74d11f3e5f68560e3557fa280859a177截图.png)

# 目录扫描

![](/vulnhub_img/WEBRESOURCE5aafe361346778506a374494f17024ac截图.png)

# WEB渗透

web渗透一段实践后无果，查看其他端口：

nfs可挂载：

![](/vulnhub_img/WEBRESOURCEaf4e31ab75baf2a39f3d550ac992d1fb截图.png)

收集信息：

![](/vulnhub_img/WEBRESOURCE30dacffaafc7e3ffebe689d4fb53732c截图.png)

发现密钥：

![](/vulnhub_img/WEBRESOURCEd441d3a42f03800fb56f1cf91b0676d6截图.png)

但是无法连接：

![](/vulnhub_img/WEBRESOURCE1e8aa9370835721da8c0f6c19a2faae4截图.png)

### 至此思路断了，后面看WP，说看mysite中的cs文件，我最开始打眼一看，以为是css就是没管，没想到是CS，随后又是一个JSfuck编码和解码：

![](/vulnhub_img/WEBRESOURCE506e7eb038b45845f782ff7524199b5e截图.png)

![](/vulnhub_img/WEBRESOURCE6ded1483009027423da26ba1b396ba7d截图.png)

获得密码：TryToGuessThisNorris@2k19

尝试登录：

![](/vulnhub_img/WEBRESOURCE1dfe8d3b5beafe60e4f12209d8c5cf20截图.png)

![](/vulnhub_img/WEBRESOURCE237afc63eb80de6bbd426df2063a1703截图.png)

# 提权

![](/vulnhub_img/WEBRESOURCE2dd90e1a2b42bacc7ae16f8529e138db截图.png)

搜查了一圈没看到太多信息，盲猜可能需要提权到morris才能提到root

在html目录下查到了一个swp文件

想办法复原后出现密码：blehguessme090

![](/vulnhub_img/WEBRESOURCEccaf0ed13f6fa8d2cee83c93448cecaf截图.png)

获得morris成功

![](/vulnhub_img/WEBRESOURCE5544e450c2e97cebb7dcd465e645e1ea截图.png)


polkit提权

![](/vulnhub_img/WEBRESOURCE192a194ad5350ecfce348ac0ff89d611截图.png)

除了这个办法之外，看了一下评论区的[https://github.com/dzonerzy/poc-cve-2021-4034/releases/tag/v0.2](https://github.com/dzonerzy/poc-cve-2021-4034/releases/tag/v0.2)

![](/vulnhub_img/WEBRESOURCE1e0ca9547397d652778dc3ef1ffe9ae1截图.png)

