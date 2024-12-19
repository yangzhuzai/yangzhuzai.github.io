+++
ShowToc = true
TocOpen = true
title = '0x006 Misdirection1'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描：

![](/vulnhub_img/WEBRESOURCE809976807a8d1443447a68328cbbfe3e截图.png)


![](/vulnhub_img/WEBRESOURCE94b844bf08b42b37147f53240688edd8截图.png)

# web渗透

两个web端口，例行公事，目录扫描：

8080端口扫描结果：

![](/vulnhub_img/WEBRESOURCE526ca1d7906e40fab934a82c9c054579截图.png)

80端口扫描极慢，先看了一下8080端口的扫描结果，发现一个命令行页？

![](/vulnhub_img/WEBRESOURCE5361cf692f18a441e669af1c32a006b0截图.png)

反弹shell，直接弹shell 弹不回来，远程下载webshell 反弹shell成功:

![](/vulnhub_img/WEBRESOURCE368bce6dd5c62305d699ea987ffb9841截图.png)

# 提权

读取user.txt提示没有权限，然后sudo -l 发现可以无密码执行/bin/bash即可查看，并且成功提权到brexit

![](/vulnhub_img/WEBRESOURCEa6980f069602f836394cf544e4dae8c9截图.png)

## 发现passwd文件可写

![](/vulnhub_img/WEBRESOURCEf327939a9b96fc5054efed9aeb1621d7截图.png)

### 生成密码

![](/vulnhub_img/WEBRESOURCE975746eadd5935d93cf04daffd804e8c截图.png)

### 修改密码为这个：$1$ZytY2Uoj$1.aU20M/0/xD4Wo1ymW9Z0    Abcd1234

root:$1$ZytY2Uoj$1.aU20M/0/xD4Wo1ymW9Z0:0:0:root:/root:/bin/bash




这里修改文件极其诡异，VI盲改老是找不到正确的位置，sed命令修改也总是有问题，最后把passwd文件通过kali传输进去，直接覆盖掉就可以了

![](/vulnhub_img/WEBRESOURCE9fc6e33cfcbc02551f5ecc9bef897bbd截图.png)