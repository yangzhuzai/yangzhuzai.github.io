+++
title = '0x004 Mercury'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



开局就遇到一个问题：靶机无法获取IP地址，通过搜索解决办法记录如下：

重启虚拟机出现VM图标的时候就需要长按shift，然后进入GRUB：

![](/vulnhub_img/WEBRESOURCE813ca0f6ce86d53c24b1862908bf683e截图.png)

选择第二个Advanced options for Ubuntu:

![](/vulnhub_img/WEBRESOURCE41b83a18e48bf0054e408f052bb781b5截图.png)

按e编辑：

![](/vulnhub_img/WEBRESOURCE1cac39334ab07943bfbee4d9eda8a3ee截图.png)

修改

![](/vulnhub_img/WEBRESOURCEf44f2aa84809261a3aab6afd2c62ae87截图.png)

vi /etc/netplan/00-installer-config.yaml：

![](/vulnhub_img/WEBRESOURCEb851afecdc059a38417f5ca206a807f7截图.png)

改成ens33，随后重启即可：

![](/vulnhub_img/WEBRESOURCEf450dfa90e0953af37122fdfba7632b6截图.png)

# IP&端口扫描

![](/vulnhub_img/WEBRESOURCE924bc2595f28b8fcb760f2c4281a2467截图.png)


# 目录扫描：

![](/vulnhub_img/WEBRESOURCE83fc40ba45e07807aec93f8f9e463505截图.png)

robots.txt无有效页面，但是，手滑输错，发现以下页面，根据语句意思发现mercuryfacts目录：

![](/vulnhub_img/WEBRESOURCE28fcf0bc9ccb263f8edac002773aa226截图.png)

![](/vulnhub_img/WEBRESOURCE0d51ae14207ba33901c72a41704da006截图.png)

# web渗透

发现提示，可能存在sql注入？

![](/vulnhub_img/WEBRESOURCEa3e5cdf3841d13add279eadf42d7e5e6截图.png)

SQL注入！：

![](/vulnhub_img/WEBRESOURCEc9542df307fb30364ea18a50b02e1231截图.png)

注入发现账号密码：

![](/vulnhub_img/WEBRESOURCE8cc83d12618d2473bb14829fba7a5675截图.png)

# SSH爆破

账号：webmaster

密码：mercuryisthesizeof0.056Earths

![](/vulnhub_img/WEBRESOURCE31ba0ad457a5bdf0c9912b651df1a619截图.png)

# 提权

user_flag.txt

![](/vulnhub_img/WEBRESOURCEd0a1311e27b1c24aeb1c6b44798a258c截图.png)

sudo -l 没有东西

![](/vulnhub_img/WEBRESOURCE7906d55b068b8c0719de73135367174a截图.png)

发现一个新的账号密码：

linuxmaster

mercurymeandiameteris4880km

![](/vulnhub_img/WEBRESOURCE28e0dabd793b6b4cade949157b1059e8截图.png)

可以成功切换：

![](/vulnhub_img/WEBRESOURCEe1c27381ea1d479da806a3584c77ecf2截图.png)

sudo -l :

![](/vulnhub_img/WEBRESOURCE4ee19b49f17096e466076aa7b6c43ee7截图.png)

尝试运行命令，根据意思，猜想可能可以通过tail命令来进行提权：

![](/vulnhub_img/WEBRESOURCEf7b7e38fc77bb8ee6aca1a6d7744d782截图.png)


命令如下：

这里有个坑点，su -  用户名可能没法用环境变量

```
ln -s /bin/vi tail 创建软连接
export PATH=.:$PATH
sudo --preserve-env=PATH /usr/bin/check..... 
```

![](/vulnhub_img/WEBRESOURCE56b15f3df9cb30a5556ead8a732189b7截图.png)

# 使用CVE-2021-4034进行提权：

![](/vulnhub_img/WEBRESOURCEb3e24cab91b55ae4f677c784682f36d7截图.png)