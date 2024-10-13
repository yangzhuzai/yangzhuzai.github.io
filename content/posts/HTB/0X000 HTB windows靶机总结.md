+++
title = '0X000 HTB windows靶机总结'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++

个人打靶经历，不追求完美，只是记录，早些记录较为粗糙，见谅。

# 0x001：Bastard  中等 非域

渗透路线：drupal web漏洞-->fuzz端点-->修改poc-->getshell-->提权使用winpeass-->ms15-051

考点：drupal漏洞信息查找、fuzz的理解、poc理解和使用

# 0x002：Arctic 简单 非域  

渗透思路：端口扫描无果-->http访问-->发现adobe coldfusion-->漏洞信息查找-->漏洞利用getshgell-->提权使用winpeass-->ms16-032

考点：较为简单，思路常规，正常渗透即可完成
# 0x003：Devel 简单 非域

渗透思路：FTP匿名登录-->发现web网站和匿名登录目录一致-->生成asp木马反弹shell-->提权使用winpeass-->ms16-032

考点：较为简单，思路常规，正常渗透即可完成

# 0x004：Bastion 中等 非域

渗透思路：smb匿名访问-->windows image backup -->发现vhd的备份文件 -->发现是C盘-->导出SAM和SYSTEM文件-->读取hash-->破解-->提权

提权无法直接通过脚本提示，需要查找软件目录，发现非常规软件mRemoteNG--->信息查找，发现该软件可能存在密码读取的情况-->漏洞利用，提取administrator密码成功

考点：VHD硬盘备份、sam文件读取、mRemoteNG密码提取；知识点盲区是本靶场的核心问题。

# 0x005：Blue 简单 非域

渗透思路：nmap告诉存在ms17-010-->需要了解python如何实现ms17-010

考点：在不依赖msf的情况下，如何打ms17-010


# 0x006：Bounty 简单 非域 

渗透思路：目录扫描发现上传功能-->但是无法上传正常的后缀-->通过fuzz发现可上传.config后缀-->该后缀在II7.0以上可以getshell-->提权ms16-032

考点：config getshell
# 0x007：Acute 困难 域渗透（非常规打法）

端口扫描：

发现443 https端口，浏览器访问失效，通过证书发现域名，修改hosts文件，成功访问到了网页。

web渗透：

首页about us，泄露部分出名员工名称，同时有个新人表单，通过新人表单，可以发现PSWA入口以及默认密码：Password1!，以及创建者主机信息：Acute-PC01以及Lois用户是网站管理员等信息；通过泄露的员工名，拼接首字母和后面的名，使用默认密码，成功撞库PSWA，获取立足点powershell，普通权限，用户名：acute\edavies。

后渗透阶段：

这台靶机目前最好还是使用meterpreter，完全避免使用meterpreter的话，会花费巨量时间，以及部分功能可能无法通过脚本实现。

后渗透-横向移动阶段1：

通过查看当前的远程登录会话，可以发现目前的用户acute\edavies是使用了RDP的，使用meterpreter的屏幕监控功能，查看当前用户的操作，发现了另外一台主机名为：ATSSERVER，以及一个相关凭证：密码：W3_4R3_th3_f0rce.用户：acute\imonks，使用powershell，申请凭证，获取到ATSSERVER普通权限，可以拿到第一个user.txt，同时使用这个用户可以发现Program Files目录下有个keepmeon目录，是用户安装的，非常规软件目录，但是当前\imonks用户没有权限进入。

后渗透-横向移动阶段2：

在第一个user.txt的附近有主机Acute-PC01的脚本信息，虽然密码进行了加密，但是可以修改脚本达到命令执行的效果，反弹一个脚本内用户的权限为：acute\jmorgan，属于本地管理员用户组，也就是说我们拿到了Acute-PC01主机的管理员权限。

后渗透-横向移动阶段3：

使用Acute-PC01主机的管理员权限dumphash，拿到本地administrator的密码，但是无法登录ATSSERVER，由于我们之前拥有ATSSERVER的imonks用户，通过查看用户目录，可以知道该主机有哪些用户登录过了，使用Acute-PC01主机的密码进行喷洒，成功获取了一个名为awallace的用户。

后渗透-横向移动阶段4：

awallace用户可以进去keepmeon目录，这个目录下面有个bat文件，内容为计划任务，执行者为Lois，该用户是最开始告诉我们的网站管理员，同时在域内查找用户组，可以发现，site_admin用户组，该用户组属于domain admin group用户组，也就是说这个计划任务可以让我们获得域管理员组权限，利用这个BAT提权成功，即可获得ATSSERVER主机的管理员权限，拿到最终的flag.

考点：1、信息收集；2、pswa；3、powershell熟练度；4、内网信息收集，密码获取；5、计划任务提权

放在最后：这个靶机的打法及其非常规，最开始的信息收集便在一直挑战打靶者的思路完善程度，内网的提权和信息收集过程，更是把两个机器当成一个整体的思维在进行渗透，信息收集的程度堪称离谱，powershell的熟悉程度也是一个不小的挑战，最后的来回横跳巨爽。

# 0x008：Timelapse 简单 域渗透

渗透思路：
smb匿名访问-->zip文件-->破解发现pfx文件-->再次破解-->提取出私钥和公钥-->wirm登录-->提权，通过查看powershell历史记录，发现账号密码-->发现该账号具有laps_readers组权限，可以读取administraor密码

考点：1、PFX；2、laps_readers

# 0x009：StreamIO 中等 域渗透

渗透思路：
web渗透-->注意证书的域名变化-->再次目录扫描-->发现sql注入-->获取网站的用户名和密码-->通过burp爆破，成功获取到合法用户-->进入后台，注意url存在?-->fuzz发现debug参数-->是个文件包含，需要使用base64带出-->这里的思路有点非常规，爆破当前目录，发现master.php，读取它-->发现它接收post提交的参数，并执行-->通过文件包含，并携带参数，命令执行反弹shell-->读取数据库账号密码-->winrm登录-->读取浏览器密码-->获取到JDgodd用户权限-->修改CORE STAFF组的拥有者为JDgodd，把自己添加到组内，并修改dacl，读取密码

考点：1、web渗透的细节；2、sql注入；3、fuzz；4、文件包含；5、代码审计；6、密码获取；7、writeowner权限；8、ldap密码获取

# 0x012：Active 简单 域渗透

渗透思路：
smb匿名登录-->发现SYSVOL目录下的gpp组策略文件-->恢复密码-->登录smb可以发现flag-->提权kerberoasting-->获取administrator成功登录

考点：1、gpp组策略目录C:\\Windows\\SYSVOL\\DOMAIN\\Policies\\ ；2、kerberoasting

# 0x013：Forest 简单 域渗透

渗透思路：
ldap匿名访问-->获取用户名-->AS-REP Roasting-->提权-->bloodhound--->当前用户拥有写入EXCHANGE WINDOWS PERMISSIONS组的权限-->EXCHANGE WINDOWS PERMISSIONS可以修改权限，赋予自己dcsync权限-->dump hash-->登录administrator

考点：1、AS-REP Roasting；2、写入权限；3、组完全修改控制权；4、bloodhound即可完成

# 0x014：Return 简单 域渗透


渗透思路：

打印机web NTLM relay -->winpeass -->服务提权

考点：1、NTLM relay；2、服务提权

# 0x015：Sauna 简单 域渗透

渗透思路：
web页面信息收集，需要手动，名字姓名组合-->AS-REP Roasting-->bloodhound发现一个用户具有dcsync权限-->winpeass发现密码-->dump即可

考点：1、信息收集；2、AS-REP Roasting；3、bloodhound

# 0x016：Support 简单 域渗透

渗透思路：
smb匿名登录-->添加域名的情况下运行程序-->抓包获取ldap账号密码-->ldap查找到密码信息-->基于资源的约束型委派

考点：1、ldap信息获取；2、基于资源的约束型委派

# 0x017：blackfield 困难 域渗透

渗透思路：
smb匿名登录-->发现用户名-->AS-REP-->bloodhound-->该用户拥有修改AUDIT2020用户的权限-->再次登录smb-->发现lsass程序内存dump-->发现几个用户的可用信息-->通过bloodhound的指引，利用BACKUP OPERATORS组的用户的权限备份ntds.dit数据库，里面有整个域内的hash-->成功获得administrator

考点：1、AS-REP；2、changepassword权限；3、lsass dump；4、BACKUP OPERATORS组权限；5、ntds.dit数据库；

# 0x018：matis 困难 域渗透

渗透思路：
web 目录扫描-->发现数据库账号和加密了的密码-->登录mssql-->数据库内发现账号密码-->ldap可登录-->几乎试完了所有的常规思路-->ms14-068

考点：1、目录扫描仔细程度；2、16进制转字符串；3、域渗透思路完善程度，最后尝试ms14-068

# 0x019：object 困难 域渗透

渗透思路：
web渗透发现jenkins页面-->可以注册用户-->可以通过build project来执行命令-->不过没有创建项目的按钮，需要使用api的方式来进行-->反弹shell失败，但是可以读取文件-->读取jenkins用户配置文件--->复原密码-->最后成功登录-->bloodhound 其实提示得比较明显-->changepasswd-->Genericwrite，但是这里有问题，理论上我们可以直接拿到目标的权限，但是这里有防火墙干扰，同时又无回显，最后发现桌面的xls表格成功跳转-->Writeowner-->domain admins

考点：1、大量的实操经验；2、信息收集的能力；3、jenkins利用中的小细节，比如二进制读取和复原；4、bloodhound

# 0x020：Reel 困难 域渗透

渗透思路：

ftp匿名登录-->发现提示留言，会查看rtf格式的文件-->发送钓鱼邮件，RTF调用宏进行上线-->桌面存在提示信息cred.xml，该文件是 PSCredential 对象当中Export-CliXml方法输出的 XML 文档，可以恢复-->成功登录后，发现一个csv文件，为bloodhound的结果文件-->WriteOwner-->WriteDacl-->通过Backup_Admins组权限，读取桌面上的Backup文件夹，发现有管理员密码

考点：1、大量的实操经验；2、rtf文件getshell；3、PSCredential 对象存储密码；4、csv bloodhound；5、WriteOwner和WriteDacl

# 0x021：Search 困难 域渗透

渗透思路：

经过大量的常规思路，发现无果后，发现web页面的图片有账号密码-->bloodhound-->kerberoasting-->返回看smb-->发现大量用户名-->密码喷洒发现新可用账户-->再次查看smb-->在当前用户名的目录下发现xlsx文件-->第二行有隐藏，但是使用wps敲一下行数就看得到-->再次爆破，成功获得新可用账户-->bloodhound路线获取domain admin

考点：1、信息收集；2、信息收集；3、信息收集

# 0x022：cascade  中等 域渗透

渗透思路：

ldap匿名访问-->查看敏感信息，发现账号密码-->登录smb-->发现vnc注册表信息-->破解后登录成功-->使用新的账号密码再次登录smb-->发现sqlite文件，密码加密-->同目录的exe程序可解密-->逆向获取aes信息-->破解成功-->AD Recycle组-->查看删除用户的详细信息-->该密码和administraotr一样，这一点最开始的smb文件中有提及

考点：1、信息收集；2、VNC；3、aes解密；4、AD Recycle组

# 0x023：Escape 中等 域渗透

渗透思路：

smb匿名访问-->发现mssql账号密码-->登录mssql进行relay-->查看mssql 日志-->发现账户密码-->adcs ESC1

考点：1、mssql relay；2、mssql 日志；3、adcs ESC1

怎么说呢，最短路径很短，但是无脑打的话，就很漫长了

# 0x024：Intelligence 中等 域渗透

渗透思路：

web渗透发现文件可枚举-->文件中有可用密码信息-->提取所有文件的创建人信息-->密码喷洒-->成功获取到账户-->登录smb-->发现计划任务脚本--->添加dns解析到本机-->等待中继-->破解-->bloodhound-->提权到administrator

考点：1、文件创建者；2、信息获取；3、中继；4、dns解析

# 0x025：monteverde 中等 域渗透

渗透思路：

ldap匿名登录-->获取用户名-->爆破，用户名即是密码-->登录smb-->发现其中一个文件含有密码-->密码喷洒-->登录-->通过Azure AD来获取密码-->登录administrator

考点：1、ldap匿名登录；2、Azure AD

# 0x032：Aero 中等 非域

渗透思路：

web页面发现这是一个win11主题的网站 -->通过github的信息复现漏洞-->用户文档文件夹中存在一个cve，通过那个cve直接打就行

考点：漏洞识别和复现

# 0x033：Atom 中等 非域

渗透思路：

smb发现文档和程序-->程序抓包-->发现会更新程序-->利用这种更新机制，以及smb文档的解释，可以构造POC来getshell-->读取redis密码-->登录redis-->发现加密密码-->破解-->administrator

考点：1、更新机制getsgell；2、PortableKanban密码解密

# 0x034：Querier 中等 非域

渗透思路：

smb匿名登录-->发现带宏的excel表格-->提取宏脚本-->获取sqlserver账密信息-->中继获取hash-->重新登录sqlserver-->xp_cmd-->winpeass发现gpp组策略密码-->administrator登录成功

考点：1、宏提取；2、中继；3、gpp组策略

# 0x035：Sniper 中等 非域

渗透思路：

web渗透发现文件包含-->通过文件包含读取session文件-->发现seesion文件中存在用户名-->把webshell写入用户名中-->反弹shell-->数据库账号密码-->爆破发现是web中某个用户的密码-->反弹shell-->在用户文件夹中发现chm文件-->构造chm的payload-->中继获取administrator的ntlm hash-->破解登录

考点：1、php在windows中的session格式共和存储位置；2、powershell切换身份；3、chmpayload构造

# 0x036：BreadCrumbs 困难 非域

渗透思路：

目录扫描发现登录框-->注册账户发现用户名-->发现文件包含-->读取登录相关代码-->代码审计-->伪造登录身份-->jwt也得伪造，可以读取文件，有key可以伪造-->替换关键词，免杀-->web目录发现记录用户信息的文件-->发现密码-->桌面发现提示-->曾经使用过Microsoft Store Sticky Notes记录密码-->可以进行恢复-->获取新用户身份-->在用户目录下发现linux程序-->strings读取发现web访问记录-->端口转发本地1234端口--->发现administrator的aes_key-->发现sql注入--->注入获取aes加密密码-->解密登录

考点：这个靶机考点很多，包括：文件包含、代码审计、端口转发、简单免杀绕过、简单逆向、JWT、SQL注入、AES加密算法，一定的脚本编写能力


# 0x038：Tally 困难 非域

渗透思路：

web目录扫描得使用漏洞扫描字典-->发现文档信息-->发现ftp用户名和密码-->登录ftp-->发现keepass软件-->破解-->登录smb-->在某个目录下看到程序-->读取程序-->发现mssql账户密码-->登录mssql-->xp_cmd-->烂土豆提权

考点：1、目录扫描；2、keepass；3、逆向；4、xp_cmd；5、烂土豆提权