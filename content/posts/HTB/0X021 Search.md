+++
title = '0X021 Search'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCEbcc7bd75966846206fcbb8e9fed7e937image.png)

![](/htb_img/WEBRESOURCEf8d179bcc8a0b24c3c2da0380dc7d5a6image.png)

![](/htb_img/WEBRESOURCE19e74db1c6e96895df83a9c994f2512aimage.png)

![](/htb_img/WEBRESOURCEcdc1b87c9c82b1e12d33f29819ca7d81image.png)

# 获取立足点

值得关注的服务:http、LDAP、smb：

先看smb:

![](/htb_img/WEBRESOURCEf6ced49653d9791722037b9723f60c4dimage.png)

接着看ldap：

![](/htb_img/WEBRESOURCEa9a646175e1e6421dceb8f62aacaf44aimage.png)

剩下的就只有http了，重点关注：用户名与密码，漏洞

about us是重灾区：

![](/htb_img/WEBRESOURCE667dba0de9f8977eabc2c07ee0f2f2daimage.png)

![](/htb_img/WEBRESOURCEd7407382aa354570fdc5406c9f1e8121image.png)

暂且这样吧：

![](/htb_img/WEBRESOURCEb379e193d6d14f258b02a54ef5ca0741image.png)

用户名枚举：

枚举失败了：

![](/htb_img/WEBRESOURCEbbcf140df7fd8c265b82e1cc28ee5508image.png)

但是dirsearch发现了东西：

![](/htb_img/WEBRESOURCE8899e2a6a81205dbe6090d87204e596fimage.png)

![](/htb_img/WEBRESOURCE658829953b871cb4dedf12e5da750154image.png)

但是好像没什么用：

![](/htb_img/WEBRESOURCE6f7a492420ebd6b8ac06cef394abb7aaimage.png)

峰回路转：

cewl收集的字典发现信息：

![](/htb_img/WEBRESOURCE22841233ed8649a2b798726adbca825bimage.png)

有了用户名，尝试AS-REP Roasting：

失败了

![](/htb_img/WEBRESOURCE37aa5438b09281904842ad07bd4f11faimage.png)

只能尝试密码爆破了：

但是获取策略失败了，也只能试试了：

![](/htb_img/WEBRESOURCE5c3197bfdfba3aa3a66835cac92423a8image.png)

密码爆破失败：

![](/htb_img/WEBRESOURCE44a01eb2596f5de56af7c1fa3f424798image.png)

服了，很好，手写体的英语，里面说了密码和用户名：

发送密码给Hope sharp

IsolationIsKey?

哦，这对我的视力和英语都是一个挑战：

![](/htb_img/WEBRESOURCE58812f51dcbc62b315a6a4c3294cb435image.png)

很好，名字的组合也是一个问题：

![](/htb_img/WEBRESOURCE3fdb5186c8772f8a4a045478322878e0image.png)

# 提权

机器没有开明显的远程登录端口，如果要登录，那么只能获取管理权限，目前只能围绕着域渗透的思路来进行，先上bloodhound：

看来主要是集中在这条路线上了：

![](/htb_img/WEBRESOURCE6ddc00a4b74a93c6fca3a86638a36a94image.png)

目前的身份较为边缘，看来得想办法获取到其他用户身份:

![](/htb_img/WEBRESOURCEe0f5ebb7e61f6eae907e5f1390b6708bimage.png)

kerberoasting:

发现一个

![](/htb_img/WEBRESOURCE1031d0b9ec86c0312380c1657e7ecccaimage.png)

破解：

账号：web_svc

密码：@3ONEmillionbaby

![](/htb_img/WEBRESOURCE8eb1fc4646f151611fda300614f09291image.png)

但是发现这个账户也没有什么特殊的，根据用户名猜测这是web服务的账户：

返回去看smb:

两个账户的权限是一样的：

![](/htb_img/WEBRESOURCEd2bb563e267ae61aa66f2e73d283976aimage.png)

发现一些有趣的东西：

![](/htb_img/WEBRESOURCE3a681c82e82a647727b99588af3f359fimage.png)

用户名：

![](/htb_img/WEBRESOURCE47f756b963eb66f5d98a7a81ac7a9201image.png)

做成字典：

使用@3ONEmillionbaby这个密码，再次爆破一个用户名：

![](/htb_img/WEBRESOURCEa90e04d87b7eb89590f9d3342332ff93image.png)

这个用户好像也没法直接打域控，但是发现有趣的东西了：

helpdesk是之前smb没有权限访问的目录：

![](/htb_img/WEBRESOURCE0567b5292edc59b140e1124695a8e7edimage.png)

![](/htb_img/WEBRESOURCEe3c6ae5607ce5d33d396538afdc0c35bimage.png)

但是该目录没有任何东西：

![](/htb_img/WEBRESOURCEc7a01672c70e5205162a344037b97115image.png)

只能继续信息收集：

发现user.txt

![](/htb_img/WEBRESOURCEf10257cf6dc64afc8e51ae6d75c873d6image.png)

但是没权限：

![](/htb_img/WEBRESOURCE8a5ecf3f9359ba7f09a9e1c20f993be4image.png)

那就看自己的，这个用户的组名字和桌面有关，可能真得看自己的桌面呢:

发现信息：

![](/htb_img/WEBRESOURCEde018b14990bff2a0bee293c004677f1image.png)

好像表格里面的名字和smb里面的有些许不一样：

![](/htb_img/WEBRESOURCE907cf08e434e8a9585554b6aef9392e5image.png)

但是没找到有效密码：

![](/htb_img/WEBRESOURCE8e5ac86569cf2cd758ee54523c082681image.png)

再仔细观察表格，可以发现列C被隐藏了，但是可以修改左上角的标签查看：

然后点击左上角的复制标签即可，但是只能在表格内复制：

![](/htb_img/WEBRESOURCE7f5d81d8fae4c831892654c576f3fb16image.png)

做成这种格式：

![](/htb_img/WEBRESOURCE79053f2b6453975bd513fdead7faccbdimage.png)

再次爆破：

![](/htb_img/WEBRESOURCEd74cc363a3dd0fa9ec0ff5f8f61c6a69image.png)

获得Sierra.Frye用户：

接下来就可以按照bloodhound的路子走了：

![](/htb_img/WEBRESOURCE30a9013a20218d27f4fa7a75a0c403edimage.png)

ReadGMSAPassword读取BIR-ADFS-GMSA$的NThash：

![](/htb_img/WEBRESOURCE5581448a5d885a7c29462cedb2355fe3image.png)

使用GenericAll权限来进行利用，可以获取到TRISTAN.DAVIES用户的hash:

这里利用方式就比较多了，还可以直接修改密码等：

![](/htb_img/WEBRESOURCEee5f19a90178a2d0e6cd1dd2bae258e7image.png)

破解失败了：

!!12Honey..*7¡Vamos!

![](/htb_img/WEBRESOURCE3b3c6bb450fb5a50fa9f28c320c2f6caimage.png)

使用net user来修改TRISTAN.DAVIES用户的密码了：

使用--pw-nt-hash来进行hash传递：

![](/htb_img/WEBRESOURCE833f0859c6cb1140fae4bdc4461b397cimage.png)

wmiexec登录即可：

![](/htb_img/WEBRESOURCE451b9c16a93ba4c22a0ed46d45c7319dimage.png)