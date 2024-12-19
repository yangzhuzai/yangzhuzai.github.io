+++
ShowToc = true
TocOpen = true
title = '0X009 StreamIO'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCE3257329be177fe9aef7e4aaf52293d27image.png)

![](/htb_img/WEBRESOURCEd3c1b4b24bc3595c19b76fd9c7a427e1image.png)

![](/htb_img/WEBRESOURCEbad7ae63e95fb7c7ab832c90dcb3c920image.png)

# 获取立足点

从端口开放情况来看，值得关注的端口有：80、389、443、445、3268、5985：

先看445和389，3268等：

![](/htb_img/WEBRESOURCE4a281f5b3be9d6918b712cc983ca01fdimage.png)

![](/htb_img/WEBRESOURCEf3f130c2816498be203fa0fb14fd62aaimage.png)

那么突破口应该就在80或者443了：

简单看了一下首页，没有东西，目录扫描看看：

![](/htb_img/WEBRESOURCEd0d40e50c454fdb84b6ec6ec134d9a84image.png)

有注册和登录：

![](/htb_img/WEBRESOURCE17ea2ef4df506d9f2b6d590de8e8896fimage.png)

简单试了一下，好像没有万能密码，注册一个看看：

![](/htb_img/WEBRESOURCEdfe5603f94b28380e2893807f702875aimage.png)

但是不让登录：

![](/htb_img/WEBRESOURCEbb1872229a80d5870f7a8b2c0ee61cadimage.png)

查看证书，发现域名好像不对：

![](/htb_img/WEBRESOURCE24fe63e5dd0fbbb8607cad6090bceda5image.png)

访问这个网站：

![](/htb_img/WEBRESOURCEf16ac18a8c8ec5e36e6e9823fed7ce1aimage.png)

好像也没有东西，再扫一下：

![](/htb_img/WEBRESOURCE6e2747aa5f9e5ed0c57c279d3ecfb9ffimage.png)

发现新内容：

![](/htb_img/WEBRESOURCE94b59761b65a13c2d34f8d708f1ce12dimage.png)

疑似sql注入：

输入1‘--+

![](/htb_img/WEBRESOURCE3f154c70c42d264e0354e08271598deeimage.png)

但是好像有拦截：

order by用不了，只能用select  挨个操作：

测了半天，发现是个sqlserver:

![](/htb_img/WEBRESOURCEd3526c4ab1b930e13333a6ba8fcc7230image.png)

当前数据库名称为

STREAMIO

![](/htb_img/WEBRESOURCE8e2af23e6e1232cf3c8c74ca9c6d6549image.png)

表名：

![](/htb_img/WEBRESOURCE092d98297cb3dcdf9425e1f52fd32d62image.png)

列名：

![](/htb_img/WEBRESOURCEf06a3da5fc6fa9f6c7592d1700052202image.png)

爆用户：

![](/htb_img/WEBRESOURCE73d4eb15471bc65d5caa1927142b8ce7image.png)

James ,Theodore ,Samantha ,Lauren ,William ,Sabrina ,Robert ,Thane ,Carmon ,Barry ,Oliver ,Michelle ,Gloria ,Victoria ,Alexendra ,Baxter ,Clara ,Barbra ,Lenord ,Austin ,Garfield ,Juliette ,Victor ,Lucifer ,Bruno ,Diablo ,Robin ,Stan ,yoshihide ,admin ,cxk ,cxk ,cxk ,cxk ,cxk ,cxk ,cxk ,cxk ,cxk ,cxk ,aikun

爆密码：

![](/htb_img/WEBRESOURCE16d052e8cda4f27ae34c2f651be88c96image.png)

现在拿到账户密码了，然后看看这个密码是什么格式：

看样子是md5：

![](/htb_img/WEBRESOURCE915c3f9a2cd79f574aa6bb34daa56feaimage.png)

从现在拿到的账号密码来看，最简单的方式是直接登录winrm:

但是好像失败了：

![](/htb_img/WEBRESOURCEe695ceda3036d8a7f1f10e59ae9ccc3bimage.png)

那么就只能继续回去web看看了：

这里的内容需要爆破，但是这里不追求速度，就直接用burp了：

![](/htb_img/WEBRESOURCEe2e64f9ce67a37872fe6093fe5309455image.png)

进来后确实没想到这里会有需要fuzz的地方，是这个参数：

![](/htb_img/WEBRESOURCE9daeb97208c2549be4829ab5c0f227ddimage.png)

通过FUZZ可以发现debug这个参数：

![](/htb_img/WEBRESOURCEc4c7b97d65f371bfa523aecd9f20e099image.png)

但是这里我没有怀疑过这个地方的使用方式，（因为写了只有管理员才可用，但是实际上这个用户就可以用了）

应该给一个index.php的参数给他，就可以发现有报错，其实就是文件包含了：

![](/htb_img/WEBRESOURCE4a88ca2830c5bfbe460f158057ccfa0eimage.png)

那么这里其实是想要读取文件，但是读取失败了，用为协议，然后base64带出来：

![](/htb_img/WEBRESOURCEf5ecbc92f55f32a79c53a4d3b47e36e0image.png)

就会发现这个东西：

数据库的密码：

("Database"=>"STREAMIO", "UID" => "db_admin", "PWD" => 'B1@hx31234567890');

![](/htb_img/WEBRESOURCE649ed1645b57ea105400ff181daf7713image.png)

但是这个数据库密码有什么用呢，数据库不对外开放，sql注入也不允许切换用户，

可以发现master.php

![](/htb_img/WEBRESOURCE60d7652cac85e7d7475af0df0d6e5a67image.png)

文件包含读取：

发现里面的内容：这里需要简单看一下代码，核心是这句：

eval(file_get_contents($_POST['include']));，使用post传参，会包含include的文件，然后执行里面的内容：

![](/htb_img/WEBRESOURCE056583c599f903abcabacbb3efbfe5edimage.png)

上传NC，反弹shell：

![](/htb_img/WEBRESOURCE70791017a5a84e8a41f89af2256740feimage.png)

这里别用start /b会出现bug:

![](/htb_img/WEBRESOURCE2999dc72307c0606ea6a2b9b103a8bd0image.png)

然后就是之前的数据库账号密码的用处了：

sqlcmd -S '(local)' -U db_admin -P 'B1@hx31234567890' -Q 'USE STREAMIO_BACKUP; select username,password from users;'

![](/htb_img/WEBRESOURCEcdd5cc6a5f8da1c4ee85397602d0e37dimage.png)

再次破解：

![](/htb_img/WEBRESOURCE2e9b34263148a4a1ec4f95eeb970fc14image.png)

![](/htb_img/WEBRESOURCEbaf6f2415f3ff8274d7d3b02eed4bf25image.png)

用户不多，挨个看看权限即可：

![](/htb_img/WEBRESOURCE43ac71d4f60a1f624f2abe7730c95550image.png)

远程登录：

![](/htb_img/WEBRESOURCEf6efd6d67150ee89528154a08f44adecimage.png)

# 提权

### bloodhound

bloodhound的分析结果大致意思为，需要本地提权，除非拿到MARTIN、JDGODD的用户权限：

![](/htb_img/WEBRESOURCEc55a9afea116da26f133e194262e641aimage.png)

然后是winpeass的分析结果：

![](/htb_img/WEBRESOURCE414b4aff5cfc4f3fc612bf453b99529cimage.png)

注意这个firefox：

然后去github上找了一个这个项目：

[https://github.com/AlessandroZ/LaZagne](https://github.com/AlessandroZ/LaZagne)

![](/htb_img/WEBRESOURCE172fcb1222093a887b1ffe28133dcda1image.png)

这里拿到了我们的目标用户权限，JDGODD，用这个用户可以根据bloodhound提权到domain admin，注意，上面显示的密码有误，可多尝试几次就可以了：

[+] Password found !!!

URL: [https://slack.streamio.htb](https://slack.streamio.htb)

Login: JDgodd

Password: JDg0dd1s@d0p3cr3@t0r

这个用户没有远程管理权限，可能得域外打

![](/htb_img/WEBRESOURCE2bdfac7bea4a8c2947ad730be848d373image.png)

先拿到CORE STAFF组权限：

修改所有者：

python owneredit.py -action write -new-owner 'JDgodd' -target 'CORE STAFF' 'STREAMIO.HTB'/'JDgodd':'JDg0dd1s@d0p3cr3@t0r'

![](/htb_img/WEBRESOURCE4eac49f935eb4e0e5f0c41f797a9aa2aimage.png)

修改DACL权限，以便于读取密码：

授予JDgodd向CORE STAFF组添加用户的权限：

dacledit.py -action 'write' -rights 'WriteMembers' -principal 'JDgodd' -target-dn 'CN=CORE STAFF,CN=USERS,DC=STREAMIO,DC=HTB' 'STREAMIO.HTB'/'JDgodd':'JDg0dd1s@d0p3cr3@t0r'

![](/htb_img/WEBRESOURCE8819df6fcb6f6386216581c065c9ef59image.png)

把自己添加到CORE STAFF组内：

net rpc group addmem "CORE STAFF" "JDgodd" -U "STREAMIO.HTB"/"JDgodd"%"JDg0dd1s@d0p3cr3@t0r" -S "dc.STREAMIO.HTB"

![](/htb_img/WEBRESOURCE772010643d40cc7b821083c458e09fc1image.png)

再次查看权限：

![](/htb_img/WEBRESOURCE03f1c7d3ca316a89b42bc634c661dc16image.png)

读取密码：

![](/htb_img/WEBRESOURCEabb6041f94fd326c8331254cc9829084image.png)

登录：

![](/htb_img/WEBRESOURCE3df148660b982264a76982c9c32a6b94image.png)

# 总结

这台靶机的难点在于获取立足点较为曲折，简单梳理一下：

web：

1、首先是注册用户，添加域名，需要反复查看证书是否变化，域名有两个，第一次卡住，没发现域名变化。

2、使用新域名，再次目录扫描，发现查找页面，sql注入，是个sqlserver手动注入，需要熟悉。

3、由于是通过websql注入获取的账户密码，通过获取到的账户密码，进行web爆破，获取到一个合法用户，这里的手动登录有问题，不会弹出错误，也不清楚是不是系统存在bug，所以遇到这种情况还是尽量使用工具爆破一波再说。

4、登录后没有变化，但是可以进去之前403的admin目录，注意页面功能的？后面的参数可以fuzz，第二次卡住。

5、fuzz发现debug，参数给index.php发现有反应，是个文件包含，需要配合伪协议base64导出，发现数据库账号密码，但是没有连接条件。

6、第三次卡住，需要再次目录扫描admin目录，发现master.php文件，通过文件包含可以读取源码。

7、代码审计，发现include参数可以包含执行php代码，通过debug文件包含+post传参+远程文件包含，成功拿到shell，这里面细节很多，多次导致web访问崩掉，要么重启，要么等很长时间。

提权

8、使用拿到的高权限数据库账密，再次查询sqlserver，发现USE STREAMIO_BACKUP数据库，疑似备份数据库，发现新账户密码。

9、再次破解后，经过尝试，终于获取到域内用户身份，登录winrm。

10、bloodhound成功告诉我们提权路径，但是目前身份欠缺，需要寻找立足点。

11、通过winpeass发现疑似firefox的账号密码信息，通过LaZagne项目，成功读取，但是密码乱序，需要爆破。

12、通过尝试，成功拿到了JDgodd用户身份，该身份是bloodhound提示的可提权身份之一。

13、通过JDgodd的writeowner权限，成功获取了CORE STAFF组的owner，owner拥有writedacl权限。

14、通过向CORE STAFF组添加自己的身份，成功进入该组。

15、CORE STAFF组拥有ldap的密码读取权限，成功读取dc的密码

16、登录dc，administrator