+++
title = '0x003 HA Narak'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE459cd1832e7352042ce534b82ac463ad截图.png)

# 目录扫描

这里遇到一个问题，我下意识的认为靶机403没有价值所以没去看，导致错过入口

![](/vulnhub_img/WEBRESOURCEc3203e0ab9995b5cca660f80f5b0d39a截图.png)

## 字典收集

```
cewl 192.168.154.131 -w 1.txt
```

![](/vulnhub_img/WEBRESOURCE619c86ff1531cc0781cd27b2e6e3bead截图.png)

### hydry 爆破

```
hydra -L 1.txt -P 1.txt 192.168.154.131 http-get /webdav
```

![](/vulnhub_img/WEBRESOURCEbf08351ca395b8459b80fb7a9c998b8a截图.png)

这里有几个疑问，burp抓包如下：

看数据包内可能是MD5的加密方式，但是，没发现MD5的加密数据

![](/vulnhub_img/WEBRESOURCE49a8844d9c9cc2515ee4da893fcc56d2截图.png)

同时使用wireshark抓包，如下，均未发现MD5密码的痕迹，前端又不让抓JS，有点疑惑hydry是怎么做到的

![](/vulnhub_img/WEBRESOURCEe0a283cb172a94c140181163d835e77c截图.png)

通过查阅资料发现HTTP协议身份认证的相关知识，

上面的例子属于DIGEST 认证(摘要认证），客户端请求时会采用质询码计算生成响应码。最后将响应码返回给接收方进行认证的方式，所以不会存在bas64编码的明文密码。

# webdav利用


涉及一些新的内容：

WebDAV服务漏洞利用工具DAVTest，大致看了一下，可以知道html\php文件上传执行成功：

![](/vulnhub_img/WEBRESOURCEf59a54686243b2714c291cc8d16fb1ab截图.png)

上传PHP反弹shell脚本：

![](/vulnhub_img/WEBRESOURCE75384ebcd019134631c13fdfc3afbc55截图.png)

# 提权

inferno的user.txt

![](/vulnhub_img/WEBRESOURCE4e1a6d67a59956273ba4ab3a4127e0af截图.png)

提权这里遇到的问题挺多的，进行一下记录：

首先常规的sudo以及SUID均无法提权，所以遇到阻塞，随后，翻查敏感信息，发现了以下内容：

主要是TIPS.txt和creds.txt，creds.txt为base64加密内容，解密出来为web爆破的密码，尝试过登录，但是没有成功：

![](/vulnhub_img/WEBRESOURCE7d95dbff73673ef17704b74ef1a17af1截图.png)

后续又上传了自动化工具，进行提权，但是都失败了：

![](/vulnhub_img/WEBRESOURCE79eef3e89d742333d6bc61066fd732b4截图.png)

随后只能看WP了：

发现问题，信息收集不够全面，没发现/mnt目录下的hell.sh，同时没见过BF编码：

![](/vulnhub_img/WEBRESOURCE43fcfebea100a18469d8d8d427433a4e截图.png)

[https://www.splitbrain.org/services/ook](https://www.splitbrain.org/services/ook)

![](/vulnhub_img/WEBRESOURCE8f88720599b6542d2e61700db9c289ad截图.png)

通过密码去尝试登录可登录的账户名：

![](/vulnhub_img/WEBRESOURCEcc47450022e0b62817d866431edce998截图.png)

修改文件达到提权:

命令为：

```
cd /etc/update-motd.d/
echo "echo 'root:d1no' | sudo chpasswd" >> 00-header
```

![](/vulnhub_img/WEBRESOURCEb74a9876258db237271160cbb82611e0截图.png)

坑点为必须退出后，使用ssh登录，否则没有触发更新，不能提权成功：

![](/vulnhub_img/WEBRESOURCE866a7f4dab903278de99ece05a42c159截图.png)

# 总结

补足以下知识点：

### 1、http摘要认证

### 2、WEBdav利用

### 3、Brainfuck**编码**

### 4、00-header 文件提权