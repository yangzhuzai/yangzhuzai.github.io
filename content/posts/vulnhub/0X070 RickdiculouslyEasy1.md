+++
title = '0x070 RickdiculouslyEasy1'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE5cbc3ff694af78babf329faafbeb8c40image.png)

![](/vulnhub_img/WEBRESOURCE67a9197bd0fd1203c02ca7a89084b0e4image.png)

![](/vulnhub_img/WEBRESOURCEbc19427b819251d2b91a35a3ad2e6264image.png)

# 获得立足点

疑似密码：

Morty

winter

![](/vulnhub_img/WEBRESOURCE49cfa6d319bbeb7b1c4ecaf419e2ca07image.png)

命令执行：

![](/vulnhub_img/WEBRESOURCE31945c2cf71da08cff40f47ab704d5a1image.png)

![](/vulnhub_img/WEBRESOURCEcdcf7ea6ebab7672f94dc0dfe8d6b656image.png)

![](/vulnhub_img/WEBRESOURCEcb7bdedf7cddf620c13f569fa1ebafa2image.png)

但是绕不过：

只能试着读文件了：

![](/vulnhub_img/WEBRESOURCEddcc6ab190e9dbf6796ed17ab9ba113eimage.png)

root

RickSanchez

Morty

Summer

尝试登录：

![](/vulnhub_img/WEBRESOURCE8bd6eae8ed2a8c8f3b98646e070c0ad2image.png)

# 提权

这个机器上面全是这个猫猫图，多半是不会让我们这样方便的提权了，不喜欢ctf

![](/vulnhub_img/WEBRESOURCEdca0a5632c4ff0ac47e8c8802f3908afimage.png)

挨个到家目录下面翻查：

![](/vulnhub_img/WEBRESOURCE5de708d3010c45f84f10eacf363c7101image.png)

用scp拷贝到本地：

发现zip密码：

![](/vulnhub_img/WEBRESOURCE62665b8b88ac0bc5806a1ad69eb1d4deimage.png)

解开，注意这个flag，其实是safe命令的参数：

![](/vulnhub_img/WEBRESOURCEb5128f847ff32bd92bf899033bed3d2eimage.png)

后面到spluttered的家目录下面，可以发现safe程序：

![](/vulnhub_img/WEBRESOURCE51b4736573a6e576f59e23318439d00eimage.png)

这里提示Ricks的密码：

![](/vulnhub_img/WEBRESOURCE6ce4d89102e601752aff2508cb971051image.png)

一个大写字母，一个数字，和一个老乐队里面的单词：

百度百科：

![](/vulnhub_img/WEBRESOURCEb767c55ea517874f0b5b3acae6b23326image.png)

也就是说关键词为

用crunch生成字典：

crunch 7 7 -t ,%Flesh>passwd.txt

7 7 是指定长度为7到7

-t是指定模板

,代表大写字母

%代表数字

![](/vulnhub_img/WEBRESOURCE79b25e966cfee71ba7fc0702d53269faimage.png)

爆破

![](/vulnhub_img/WEBRESOURCE4ec64450c7b0269f7ec699b5e4ffb75fimage.png)

sudo提权

![](/vulnhub_img/WEBRESOURCEfe15b571e7cca2a3c50582531b9489e4image.png)

cat 命令被修改了，应该是：

![](/vulnhub_img/WEBRESOURCE49961a111e9fb6049e6ca1e1cac2389eimage.png)