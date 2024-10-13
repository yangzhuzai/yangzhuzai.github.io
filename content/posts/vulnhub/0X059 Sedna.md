+++
title = '0x059 Sedna'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCEc135975b771204af23cdab25abe64e82image.png)

![](/vulnhub_img/WEBRESOURCEaa778fd28e434b1b3b9adbe24cb85a9eimage.png)

# 渗透

先看samb吧：

![](/vulnhub_img/WEBRESOURCE5e29a48c49bd12b7b99e6fc9671af808image.png)

80端口

![](/vulnhub_img/WEBRESOURCE45c8ad22204e88076863d1d3f486853cimage.png)

疑似线索：

tests/_data/dump.sql

![](/vulnhub_img/WEBRESOURCE18da2beffdffc3acbddc3bafd1ca247fimage.png)

恼火，没想到，中间件漏洞，没见过的：

![](/vulnhub_img/WEBRESOURCE4b888a8cfc792dfd5a560b414e0dd46aimage.png)

![](/vulnhub_img/WEBRESOURCE718aab8b67187e563187c6e28f306b39image.png)

修改poc:

![](/vulnhub_img/WEBRESOURCE366be6ed4caac38271c4a58bccf02ff1image.png)

![](/vulnhub_img/WEBRESOURCE4811b1e61ed5b39aafab3687c248ca06image.png)

![](/vulnhub_img/WEBRESOURCEa815e56eee76a861fd14af9504f6a870image.png)

# 提权

试了无数的POC

最后无奈找到这个了：

![](/vulnhub_img/WEBRESOURCE6c2f26be610b59cf38575fc60d7cbb49image.png)

![](/vulnhub_img/WEBRESOURCE1bf036cedb6d1e303899d2f797ccaf45image.png)

![](/vulnhub_img/WEBRESOURCE5cf27c92f92e4873419c8366bf2f80aeimage.png)

![](/vulnhub_img/WEBRESOURCE6161a6e5f75e884bce7fc4c7aec612adimage.png)

![](/vulnhub_img/WEBRESOURCE477bbe6399bbfe714432f7946efb8842image.png)