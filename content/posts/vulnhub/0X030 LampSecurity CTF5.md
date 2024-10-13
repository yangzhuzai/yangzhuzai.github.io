+++
title = '0x030 LampSecurity CTF5'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE08f99b02faa408bc8c2e03f9d9677446截图.png)

![](/vulnhub_img/WEBRESOURCEbc1513a46c402a3d395a06065c4edbd2截图.png)

# 渗透

smb\mysql渗透无果，看来突破口还是在80端口上：

![](/vulnhub_img/WEBRESOURCEed67ef867d263248b51fb46cd677fdb7截图.png)

很难受，大致逛了一圈后，基本思路是OK的，看到了NanoCMS的RCE，但是需要认证，然后是SquirrelMail的历史漏洞，那个版本的打完了，没有能用的，后面想办法获取凭证无果，无奈看WP，只能说，棋差一招：

![](/vulnhub_img/WEBRESOURCE5fcb3b6f55457caa801742765ba1fdbf截图.png)

[https://packetstormsecurity.com/files/76642/Security-Evaluation-Of-NanoCMS.html](https://packetstormsecurity.com/files/76642/Security-Evaluation-Of-NanoCMS.html)

其中有信息提到，NanoCMS存在信息泄露：

![](/vulnhub_img/WEBRESOURCE42fcd10e2bf1a7ebc414e678856a71a4截图.png)

![](/vulnhub_img/WEBRESOURCEb44c5e672a8125f23ee311c60276797b截图.png)

![](/vulnhub_img/WEBRESOURCE17914a647dde3cc1acd11e0161106266截图.png)

到最后也没用上脚本：

添加一个页面：

![](/vulnhub_img/WEBRESOURCE800c57352f8b65ac29a8c70bb6fab2c2截图.png)

然后访问即可：

![](/vulnhub_img/WEBRESOURCE5bc826032ea059785650a96a8e9d39a3截图.png)

# 提权：

提权使用常规思维又是卡了很久，但是卡住的时候往往答案很简单，信息检索就行，使用linpeas.sh并没有帮我提取出来需要的内容，只能手动查找：

查找pass关键词i文件：

```
grep -R -i pass /home/* 2>/dev/null
```

![](/vulnhub_img/WEBRESOURCE8f0d9c340eb525a5eae98c55dbcb9595截图.png)

![](/vulnhub_img/WEBRESOURCE97ef93e48505336d9b2f2ddb5176020b截图.png)

没有flag

# 总结

感觉是有点体力活儿的意思，不太好搞