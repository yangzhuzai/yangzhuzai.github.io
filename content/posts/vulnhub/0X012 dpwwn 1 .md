+++
ShowToc = true
TocOpen = true
title = '0x012 dpwwn 1 '
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE1f0727e791e5d361babce67301cc1809截图.png)

# 渗透

web没发现问题，去找mysql的问题，直接开始爆破，发现被封了，第二天直接登录发现可以无密码：

本来是想直接写webshell的，看来没成功，就算了，进行一下信息收集：

![](/vulnhub_img/WEBRESOURCE248bacdf89021b9f019ca1bef4541829截图.png)

![](/vulnhub_img/WEBRESOURCE965d2782fe1912683dbf5cb285426fd5截图.png)

mistic   | testP@$$swordmistic

# 提权

计划任务提权？

![](/vulnhub_img/WEBRESOURCE4be0d39f45116e66bfa8af0d116420ea截图.png)

![](/vulnhub_img/WEBRESOURCEabbe5569312e1be757dce04ea33cc0a5截图.png)

![](/vulnhub_img/WEBRESOURCE3fce700795054f0835f4332c24ae197e截图.png)

![](/vulnhub_img/WEBRESOURCE9c055e31b09972adca71e35ede944292截图.png)