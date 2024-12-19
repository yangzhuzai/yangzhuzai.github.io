+++
ShowToc = true
TocOpen = true
title = '0x011 Broken Gallery'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCEf44623b5f75937cb5a8a04f722234d1c截图.png)

![](/vulnhub_img/WEBRESOURCE47292f9b3fcda8b6bfd726dc07c3afe4截图.png)

# WEB渗透

![](/vulnhub_img/WEBRESOURCE3ed52382191572118c01223551bc1cfb截图.png)

大致看了一下，目录浏览了都，就没必要目录扫描了，发现readme.md的时候，我就觉得不妙：

16进制字符串？

![](/vulnhub_img/WEBRESOURCEefa2e4e6bd9739b13c5222b8c98ca0f3截图.png)

然后大致看了一下图片，暂时没发现有用信息：

![](/vulnhub_img/WEBRESOURCE5404203c3e37ed592f62009b205803ed截图.png)

我花了很长时间研究这个16进制文件，甚至尝试了强行爆破，但是没有太多办法，只能查看WP了：

关于二进制恢复和查看信息的方法：

xxd工具：

```
xxd -r -ps xxxx
```

可读字符查看，strings

```
strings xxx
```

![](/vulnhub_img/WEBRESOURCE686ec1d9acfd0ca31dd920d9ecdf185a截图.png)

恢复成图片就可以看到隐藏信息了：

![](/vulnhub_img/WEBRESOURCE8fb0868e9423c689a2bf3ec14cd8ad85截图.png)

后续根据这个图片和网站生成针对性的字典，就可以爆破出SSH密码了，密码为broken:broken

# 提权

提权就比较简单了，sudo

![](/vulnhub_img/WEBRESOURCE5356dace1fa9f76a48a0234021e59307截图.png)

# 总结

其实我不是很喜欢这个靶场，因为偏离实战太多，但是站在靶场设计者的角度思考，可能是想要补全一些关于隐藏信息的利用方式。