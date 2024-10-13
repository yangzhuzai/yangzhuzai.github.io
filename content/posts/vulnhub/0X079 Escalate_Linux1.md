+++
title = '0x079 Escalate_Linux1'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCEb55240db28b2e77a57f5f3151ea17adaimage.png)

![](/vulnhub_img/WEBRESOURCE35991696c3607edd690bb5defd09f925image.png)

# 渗透

smb:

![](/vulnhub_img/WEBRESOURCEf97b1254f7dfaffb6398df2db386d488image.png)

http：

![](/vulnhub_img/WEBRESOURCEb086601261df6d310d3c93c94e5c0b79image.png)

![](/vulnhub_img/WEBRESOURCE387357b46fadc32fe608557307cc945eimage.png)

url编码后，直接反弹shell:

![](/vulnhub_img/WEBRESOURCE61dca625e919722f12a86e4f105f23e0image.png)

# 提权

根据描述来看，这个靶机主要就是看的提权：

有GCC，先秒了再说：

![](/vulnhub_img/WEBRESOURCEdf6440643a53d1b889f6f942d89d531dimage.png)

NFS提权：

![](/vulnhub_img/WEBRESOURCE7ea76102c0a0cc932b010ce7994b3865image.png)

挂载：

![](/vulnhub_img/WEBRESOURCEcbae99cfb08649110d01e9737809be5aimage.png)

复制bash:

![](/vulnhub_img/WEBRESOURCEf2fcd1f0f1985c93ac9d89b0c0ccc2edimage.png)

![](/vulnhub_img/WEBRESOURCE4bc21032b0ade087bef445dfab4984ceimage.png)

给suid权限：

![](/vulnhub_img/WEBRESOURCE527e2705a41a0115d43618593b91760dimage.png)

![](/vulnhub_img/WEBRESOURCEb492d226b2b3c0d82fa5f9d26c4ceb70image.png)