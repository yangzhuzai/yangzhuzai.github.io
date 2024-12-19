+++
ShowToc = true
TocOpen = true
title = '0x027 FOWSNIFF1 '
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



要改网络

# 端口扫描

![](/vulnhub_img/WEBRESOURCE14a2ee882388d33ff985c0ad8862ce93截图.png)

![](/vulnhub_img/WEBRESOURCE720c1716364a146d7d7bff5f592fa15f截图.png)

# web渗透

![](/vulnhub_img/WEBRESOURCE7f7dd3103e49016f81c1e51f23175c3d截图.png)

官网有相关提示，说密码泄露了，twitter账号上可能有线索：

![](/vulnhub_img/WEBRESOURCE0144f3218f1bb2dc1a2ef171cbca8391截图.png)

去twitter上搜相关账号，结果发现相关链接404了：

![](/vulnhub_img/WEBRESOURCEcd12db52b26cb29f60841da70b99f26c截图.png)

![](/vulnhub_img/WEBRESOURCE6b9a855ff01705aeca4cb5c61946ad80截图.png)

只能网上找现成的WP上的图片了，里面说这是POP邮箱服务的账号密码：

![](/vulnhub_img/WEBRESOURCEa1c3637bed4dc1ed3451996caa2ce43f截图.png)

捡回我的AWK命令处理一下数据：

```
cat hash | awk -F: '{print$1}' > pop3username
```

![](/vulnhub_img/WEBRESOURCE42995391e967fc9899dde93b8c3edc2a截图.png)

本地跑hash速度还是慢了，我还是建议批量跑：

[https://crackstation.net/](https://crackstation.net/)

![](/vulnhub_img/WEBRESOURCEd28be300a3b0ce3a10d35e604df52a93截图.png)

然后爆破，这里有个坑，第一次用的用户名@域名的方式爆破没爆出来，然后把用户名单独提取出来才爆出来

![](/vulnhub_img/WEBRESOURCE4cb2e22638e28598253b21cebf345b2b截图.png)

然后链接方式就比较多了，wp使用的NC，redis其实也可以使用nc，邮件其实还是图形化好一点，kali我用了thunderbird，windows直接使用foxmail就行了：

![](/vulnhub_img/WEBRESOURCE82d101b53c58dec83905d7599d382206截图.png)

使用thunderbird需要点开高级设置：

![](/vulnhub_img/WEBRESOURCE8931ab0842845b4b60f2d0d5382e4c94截图.png)

得到ssh密码：S1ck3nBluff+secureshell

根据收件人信息再次爆破：

![](/vulnhub_img/WEBRESOURCE4f0545d0e0fd5f226875388e111d42cc截图.png)

# 提权

常规手动查的差不多，没有发现明显的提权点，然后上脚本查找，发现可疑的执行脚本：

![](/vulnhub_img/WEBRESOURCEa12c89750bf80fcd36f4d97499bc847c截图.png)

![](/vulnhub_img/WEBRESOURCEd2675bc5e5748d740cf098aefef32f1c截图.png)

发现和登录信息一样：

![](/vulnhub_img/WEBRESOURCEb3674d9c08407f58df270222ac4747dd截图.png)

写反弹shell，重新登录：

![](/vulnhub_img/WEBRESOURCE02bf985f28fa5b4d635bb9e62121418e截图.png)