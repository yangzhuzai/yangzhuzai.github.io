+++
ShowToc = true
TocOpen = true
title = '0X007 Acute'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCE7bced707209a355e5b0d8e37665b10c6image.png)

![](/htb_img/WEBRESOURCE1553e03a5a52725213cc98b0d93dad66image.png)

![](/htb_img/WEBRESOURCEe9bc11587966185546c54c64b1c03fdcimage.png)

# 获取立足点

端口扫描没发现东西：

![](/htb_img/WEBRESOURCEc3827f6e5c0c0ed9fe650ad4b0d794b8image.png)

打开网站看一下：

![](/htb_img/WEBRESOURCEfe70211233839785d604541ed4534e40image.png)

修改hosts:

![](/htb_img/WEBRESOURCEd18aed551b61f788a2d4b032bd126d42image.png)

然后这里是我看到了，但是不知道怎么利用的：

这里的用户名是看到了：

![](/htb_img/WEBRESOURCE7d4afbef001b7925f4b2b74890cd4006image.png)

同时，这个word也看到了：

![](/htb_img/WEBRESOURCE738cf0d3e074876b486600fd30f2d176image.png)

但是没看文件属性：

![](/htb_img/WEBRESOURCEba21a112c62051f8191d46bcefd555faimage.png)

然后是文件内容，英语挺重要：

原始密码是：Password1!

![](/htb_img/WEBRESOURCE93e40bc3cae9ca0fe0f345104db0c13fimage.png)

然后这里其实涉及了一个知识盲区，PSWA是什么东西：

![](/htb_img/WEBRESOURCE7787611761e770eccc9a1a9242b89118image.png)

![](/htb_img/WEBRESOURCEdfad486d271f9e44031a5f10ab92d612image.png)

然后上面说到的会议连接在这里：

![](/htb_img/WEBRESOURCE4d920afa8c0b68c03132d544e4a4fb7aimage.png)

打开页面：

[https://atsserver.acute.local/Acute_Staff_Access](https://atsserver.acute.local/Acute_Staff_Access)

![](/htb_img/WEBRESOURCE70c4428b1e1da9e3183cf2eb22a8ab8eimage.png)

然后这里Computer name应该是Acute-PC01，是word里面的作者，密码是Password1!，账户是网页上的，但是经过尝试，均为失败：

然后问了一下GPT：

![](/htb_img/WEBRESOURCEd5435cb71dc3ff2e46c3830c9455a079image.png)

也就是说可能是这种变形：

![](/htb_img/WEBRESOURCEc12c78fb2716be13fa889352ed571759image.png)

账户名是：EDavies

![](/htb_img/WEBRESOURCE21116ef509fd1d912543fb44a5a5de8eimage.png)

反弹shell:

这里看是下载成功了，但是好像有杀软，被杀了：

![](/htb_img/WEBRESOURCE711ea82608ebd740c11c84dc2ed4bf7bimage.png)

然后是我没想到的，我认为应该是进行免杀，但是作者是给了免杀目录的，我不太明白这是否具备普适性，所以我问了一下GPT：

![](/htb_img/WEBRESOURCEa107fca6b135c6706f24712112b290bcimage.png)

所以还是英语不太好，吃大亏了：

![](/htb_img/WEBRESOURCEbf62ff533327d3c3bdb581f11c9b0800image.png)

使用Get-ChildItem -Force可以查看隐藏文件

![](/htb_img/WEBRESOURCE56fea9b0b68d0d25947440b0deff30f2image.png)

# 权限提升

好吧，这里还是没办法，参考官方文档，只能使用msf了：

把进程迁移到了explorer:

![](/htb_img/WEBRESOURCE1d04ece71a53cad75c59c2dcc83cabe1image.png)

通过qwinsta /server:127.0.0.1命令，可以看到edavies用户，也就是当前用户，使用了远程：

![](/htb_img/WEBRESOURCE3d3e3c87d12ed77d64a16b1c9c5030d5image.png)

使用

![](/htb_img/WEBRESOURCEc10c91c33c3e7f57c708e4b62c9b3906image.png)

![](/htb_img/WEBRESOURCEa4bbe2512f05f52f825ad5a9598db108image.png)

那么这里可以看到几个信息，主机名：atsserver，密码：w3_4R3_th3_f0rce，用户名：acute\imonks:

使用账号密码申请凭据，执行命令：

```
$passwd = ConvertTo-SecureString "W3_4R3_th3_f0rce." -AsPlainText -force
$cred = New-Object System.Management.Automation.PSCredential ("acute\imonks", $passwd)
Invoke-Command -computername ATSSERVER -ConfigurationName dc_manage -ScriptBlock {whoami} -credential $cred
```

![](/htb_img/WEBRESOURCE8c57785662c70b06d2d10f64141a5e52image.png)

此时应该是在另外一台主机了，重新侦察信息：

![](/htb_img/WEBRESOURCE822a1b5bf9dd011a887773c39683a30bimage.png)

拿到第一个flag和第二个凭据：

$securepasswd = '01000000d08c9ddf0115d1118c7a00c04fc297eb0100000096ed5ae76bd0da4c825bdd9f24083e5c0000000002000000000003660000c00000001000000080f704e251793f5d4f903c7158c8213d0000000004800000a000000010000000ac2606ccfda6b4e0a9d56a20417d2f67280000009497141b794c6cb963d2460bd96ddcea35b25ff248a53af0924572cd3ee91a28dba01e062ef1c026140000000f66f5cec1b264411d8a263a2ca854bc6e453c51'

$passwd = $securepasswd | ConvertTo-SecureString

$creds = New-Object System.Management.Automation.PSCredential ("acute\jmorgan", $passwd)

Invoke-Command -ScriptBlock {Get-Volume} -ComputerName Acute-PC01 -Credential $creds

这个用户叫做jmorgan，主机名是 Acute-PC01，也就是我们当前的这台主机，看看这个用户的权限：

可以发现是这台机器的管理员用户，也就是说我们可以尝试获取到本台主机的管理员权限，

![](/htb_img/WEBRESOURCE9b6c292f2dd5358bc643c4dcf55dc8bcimage.png)

这时候想到的就是，反弹一个shell回来：

修改原来的代码为：

```
Invoke-Command -computername ATSSERVER -ConfigurationName dc_manage -ScriptBlock {((Get-Content "c:\users\imonks\Desktop\wm.ps1" -Raw) -replace 'Get-Volume','cmd.exe /c c:\utils\cxk2.exe') | set-content -path c:\users\imonks\Desktop\wm.ps1} -credential $cred
```

查看修改效果：

![](/htb_img/WEBRESOURCE6354719b4de59b68b78287412390064fimage.png)

执行一下这个代码：

```
Invoke-Command -computername ATSSERVER -ConfigurationName dc_manage -ScriptBlock {c:\users\imonks\Desktop\wm.ps1} -credential $cred
```

![](/htb_img/WEBRESOURCEd927742bc58e03aecc5cfeca070ccd62image.png)

当前用户：

![](/htb_img/WEBRESOURCE3670a85091910f29d239876eb7f0857bimage.png)

当前IP：

![](/htb_img/WEBRESOURCE626e5ebbe93d1b678ae73f205a687427image.png)

hashdump:

![](/htb_img/WEBRESOURCE9d6bc54f9003da4b6df1ef2e40bf4f06image.png)

administrator 是Password@123，两个为空：

![](/htb_img/WEBRESOURCE076bf7651b32e43e6c18dd3e213e63c4image.png)

这个密码，是这台机器的，别忘了还有一台叫做ATSSERVER的机器：

这台机器有以下账户：

![](/htb_img/WEBRESOURCE2d8fe2fd10640446cc226d657d6c37f9image.png)

尝试使用Password@123进行登录：

使用awallace登录成功：

![](/htb_img/WEBRESOURCE8d115bedfa3667d93c6e6d1daee12cc4image.png)

然后在C:\Program Files目录中，找到一个目录，叫做keepmeon，这个目录使用imonks用户是进不去的，只能使用awallace用户登录：

![](/htb_img/WEBRESOURCE02571ae062e38d364f5032e54fa7a190image.png)

![](/htb_img/WEBRESOURCE8a55cb33c5bbbfcba8ad4ce9bf2167adimage.png)

查看：

![](/htb_img/WEBRESOURCEab98a8da667f79bd0b25134d9091fa8aimage.png)

这个就比较简单了，计划任务提权：

![](/htb_img/WEBRESOURCE33560b9cde096a1eaef15961ca6c1ad1image.png)

但是这里说了，只有lois使用，这个lois是什么权限，我们就能提升到什么权限：

![](/htb_img/WEBRESOURCEb8c5fcf39ec4d8c50651e97f701ffd89image.png)

路易斯是网站管理员，看看site_admin

![](/htb_img/WEBRESOURCEd8bb8af4a1866afd9caa06d80a23de6fimage.png)

根据描述，这个组可以访问domain group:

![](/htb_img/WEBRESOURCEac0406db4f1aac83a408aaed0cb5e9a6image.png)

本来想重新弹个shell回来，但是琢磨着，可能有杀软，就还是改一下管理组吧：

```
Invoke-Command -ComputerName ATSSERVER -ConfigurationName dc_manage -Credential $cred -ScriptBlock {Set-Content -Path 'c:\program files\Keepmeon\admin.bat' -Value 'net group site_admin awallace /add /domain'}
```

![](/htb_img/WEBRESOURCEd625692abf29b668da34941be5ba93b4image.png)

![](/htb_img/WEBRESOURCEfc1fcb1dfff440911f4f2071b34871ddimage.png)

再次查看就可以看到awallace在组内了：

![](/htb_img/WEBRESOURCE5399b3b4ef7c14a3a2ce614b2f4b674eimage.png)

然后就可以进去administrator目录了：

![](/htb_img/WEBRESOURCE1cbbf1da4aecc19b82a9638130647c14image.png)

查看：

![](/htb_img/WEBRESOURCEff16b188cdfc6eb080522f372c11532dimage.png)

# 总结

这个实验室巨难，是一个域内的靶场，两台主机，来回反复获取凭证。

梳理一下流程：

#### 入口IP：10.10.11.145 

#### 端口扫描：

发现443 https端口，浏览器访问失效，通过证书发现域名，修改hosts文件，成功访问到了网页。

#### web渗透：

首页about us，泄露部分出名员工名称，同时有个新人表单，通过新人表单，可以发现PSWA入口以及默认密码：Password1!，以及创建者主机信息：Acute-PC01以及

#### 后渗透阶段：

这台靶机目前最好还是使用

#### 后渗透-横向移动阶段1：

通过查看当前的远程登录会话，可以发现目前的用户acute\edavies是使用了RDP的，使用meterpreter的屏幕监控功能，查看当前用户的操作，发现了另外一台主机名为：ATSSERVER，以及一个相关凭证：密码：W3_4R3_th3_f0rce.用户：acute\imonks，使用powershell，申请凭证，获取到ATSSERVER普通权限，可以拿到第一个user.txt，同时使用这个用户可以发现Program Files目录下有个keepmeon目录，是用户安装的，非常规软件目录，但是当前\imonks用户没有权限进入。

#### **后渗透-横向移动阶段2：**

在第一个user.txt的附近有主机Acute-PC01的脚本信息，虽然密码进行了加密，但是可以修改脚本达到命令执行的效果，反弹一个脚本内用户的权限为：acute\jmorgan，属于本地管理员用户组，也就是说我们拿到了Acute-PC01主机的管理员权限。

#### **后渗透-横向移动阶段3：**

使用Acute-PC01主机的管理员权限dumphash，拿到本地administrator的密码，但是无法登录ATSSERVER，由于我们之前拥有ATSSERVER的imonks用户，通过查看用户目录，可以知道该主机有哪些用户登录过了，使用Acute-PC01主机的密码进行喷洒，成功获取了一个名为awallace的用户。

#### **后渗透-横向移动阶段4：**

awallace用户可以进去keepmeon目录，这个目录下面有个bat文件，内容为计划任务，执行者为