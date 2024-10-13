+++
title = '0x021 BNE0x03-Simple'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

忘了截图，扫完只开了80端口

# getshell

文件上传

![](/vulnhub_img/WEBRESOURCE54e65f7dee2516e8b4c03ce7d0e1b6f1截图.png)

![](/vulnhub_img/WEBRESOURCEa08866e4af08da14a20aa3db3bb6bd4e截图.png)

# 提权

发现users.txt

![](/vulnhub_img/WEBRESOURCEeb8dec7176e3e59d6d5f048fed7e0557截图.png)

有GCC，算了，不找了，内核提权吧：

![](/vulnhub_img/WEBRESOURCEa9a83bf1f4c0aaa7753b66370b7e3a10截图.png)