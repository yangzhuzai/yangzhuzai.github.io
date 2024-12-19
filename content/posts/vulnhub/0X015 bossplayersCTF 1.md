+++
ShowToc = true
TocOpen = true
title = '0x015 bossplayersCTF 1'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



开机需要解决一下网络问题，这次是debian，配置文件为/etc/network/interface

# 端口扫描：

![](/vulnhub_img/WEBRESOURCE65491538197371c76ba50372cd5caa08截图.png)

# WEB渗透

![](/vulnhub_img/WEBRESOURCEb244f6293e33d32b5bd9fb119e925f7f截图.png)

打开页面提示有域名，修改hosts文件，sudocuong.com

![](/vulnhub_img/WEBRESOURCE9b7fce60ba5dc47949aa630f1bddf3f2截图.png)

再扫一次：

![](/vulnhub_img/WEBRESOURCEc5621ff989caee27d13a5452097fb451截图.png)

bG9sIHRyeSBoYXJkZXIgYnJvCg==

![](/vulnhub_img/WEBRESOURCEebd2792333eb5767cf8ac49244f17d6d截图.png)

可能不是密码？

![](/vulnhub_img/WEBRESOURCE1383bf04463fdd72e2af4136069e29f1截图.png)

然后是logs.php，没发现什么可用信息：

![](/vulnhub_img/WEBRESOURCE45e443abea27b61380df0adc5a7f3c62截图.png)

发现线索，看起来有点像base64：

![](/vulnhub_img/WEBRESOURCE1250ca566a6e5f70633be2ae65862bf8截图.png)

多次解码后：

![](/vulnhub_img/WEBRESOURCE27baa57e633cda4c4a6a7908a0580a2a截图.png)

![](/vulnhub_img/WEBRESOURCEfe2a17c81605db7e6dd74a99e90f5d1e截图.png)

![](/vulnhub_img/WEBRESOURCEe2535ab08bc2b4d8948c6a9b88690fb1截图.png)

值得一提这里可以使用wfuzz把cmd使出来：

sudo wfuzz -w /usr/share/wordlists/wfuzz/general/common.txt -u [http://sudocuong.com/workinginprogress.php?FUZZ=id](http://sudocuong.com/workinginprogress.php?FUZZ=id) --hw 36

![](/vulnhub_img/WEBRESOURCE2cb2e270dfe2bed862f2b3985558a268截图.png)

# 提权

![](/vulnhub_img/WEBRESOURCEce43b0818634200760fd4941be3b7df9截图.png)

![](/vulnhub_img/WEBRESOURCEd60795c09ff134ad0cf4e0aab006b3e8截图.png)