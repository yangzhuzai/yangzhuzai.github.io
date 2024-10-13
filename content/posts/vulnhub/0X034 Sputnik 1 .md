+++
title = '0x034 Sputnik 1 '
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE4e40d195b7a91574331b069e0ff75b22截图.png)

![](/vulnhub_img/WEBRESOURCE145236ecce5b7c10a067371b943cdf5a截图.png)

![](/vulnhub_img/WEBRESOURCE4ec6eb8ba5277c38092995f5b1e178f5截图.png)

# 渗透测试

这台靶机的端口很奇葩，而且三个HTTP，直接开始吧

先把看看nday漏洞，但是没发现有用的东西：

![](/vulnhub_img/WEBRESOURCE44c937b5d91911c0a0f738206cf2e69c截图.png)

目录扫描发现有用的东西了.git目录：

![](/vulnhub_img/WEBRESOURCE105babf9c7ca7af157473f369427329e截图.png)

稍微研究一下这个东西怎么利用：

![](/vulnhub_img/WEBRESOURCE75d700fbf7dd5ac0d410e58dd266b34f截图.png)

到这里就误入歧途了，通过工具翻查了半天，然后没找到有用东西，最后无奈看了WP，提示信息在这个文件里面：

![](/vulnhub_img/WEBRESOURCEb68a060896131d786c841ffe45949e79截图.png)

当时看过这个文件，然后看到了ROOT，但是一想没有开放ssh端口，所以就没管，没想到是因为克隆的其他人仓库导致的：

![](/vulnhub_img/WEBRESOURCE6a1e5a5757e8e8ca4558c20ce46777b8截图.png)

然后把其他人的仓库克隆下来，再查看日志：

git log -p

![](/vulnhub_img/WEBRESOURCE8dc1c87f6fbc9f9f0ed81ddf5f13aaf5截图.png)

不过要吐槽一下，这谁知道是账号密码啊，难不成标准格式就长这样？

+sputnik:ameer_says_thank_you_and_good_job

![](/vulnhub_img/WEBRESOURCE1b02d91bba0cea95ad565527cbf7f34e截图.png)

登录后台：

![](/vulnhub_img/WEBRESOURCE988a1f40ac2ec781a44c71a411a06f17截图.png)

到这里就可以用已知漏洞了

![](/vulnhub_img/WEBRESOURCE8045bf7cde54279b8a9c879c005e4116截图.png)

这里需要处理一下pip2的问题：

```
wget https://bootstrap.pypa.io/pip/2.7/get-pip.py
sudo python2 get-pip.py
```

![](/vulnhub_img/WEBRESOURCE114203728c63af1c0dcfe0a4e6f0f674截图.png)

然后下载模块：

![](/vulnhub_img/WEBRESOURCEa9ff4adbb4dc1abf9f500736a45bf20e截图.png)

接着下载依赖：

![](/vulnhub_img/WEBRESOURCEfa3c4633e94e01218c4f0da8da0693a2截图.png)

![](/vulnhub_img/WEBRESOURCE8d7f8c200a47ff8eb8e4420f52a42db4截图.png)

![](/vulnhub_img/WEBRESOURCE7ad0feac33962d493006c2d3466ab8fc截图.png)

![](/vulnhub_img/WEBRESOURCE68edc12a743e5e98b2d4250e1ea1729a截图.png)

修改一下代码：

![](/vulnhub_img/WEBRESOURCE53f2d2ec4ea87c219629c6c3499d0475截图.png)

getshell:

![](/vulnhub_img/WEBRESOURCE3da457feb327417cd2f4651c00c83f29截图.png)

# 提权

秒了

![](/vulnhub_img/WEBRESOURCEcd4529348a7956f70436a5c780863d5f截图.png)

![](/vulnhub_img/WEBRESOURCE3513a6ba57ac13425ab9cb4eca585f81截图.png)

![](/vulnhub_img/WEBRESOURCE557442b4ba88f4480f89d6f163498df9截图.png)