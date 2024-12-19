+++
ShowToc = true
TocOpen = true
title = '0X019 object'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCE8a4a5bd06cc5b94d2a4c7a4bb86ab5adimage.png)

![](/htb_img/WEBRESOURCE463dd639cc418cac23c20b51d76fbdffimage.png)

![](/htb_img/WEBRESOURCE2bb8b45352e30d415f216ae75dc4ef15image.png)

# 获取立足点

2个http，一个winrm：

80，改一下hosts:

![](/htb_img/WEBRESOURCEf61552d770e37239b793daad390960aaimage.png)

发现Jenkins，但是没有密码：

![](/htb_img/WEBRESOURCE0d95232379b880d4f594c14e44e82c81image.png)

注册一个就好：

记录一下，这里利用较为复杂：

![](/htb_img/WEBRESOURCE18acfef783a02105d50f29b7627cc67eimage.png)

选第一个就行，Freestyle

![](/htb_img/WEBRESOURCE341076286b10660bc98b9d81712b8c3fimage.png)

![](/htb_img/WEBRESOURCEd24048db8c3264b0df9274339bd7d30fimage.png)

![](/htb_img/WEBRESOURCE9d8b32a4cb933cf5a8507940245f2aa7image.png)

![](/htb_img/WEBRESOURCEd6fb38b188e5c54e893655c43ea7a214image.png)

最后就可以发现cxk2创建好了：

![](/htb_img/WEBRESOURCE5989113e37e3e91230705d5e9706515fimage.png)

获取一个token:

![](/htb_img/WEBRESOURCE292e7d59d2112195f79b429607e33003image.png)

添加一个项目：

curl [http://cxk:111925c1dec8a58ad68b891d2c47ce010d@object.htb:8080/job/cxk2/build?token=cxk](http://cxk:111925c1dec8a58ad68b891d2c47ce010d@object.htb:8080/job/cxk2/build?token=cxk)

![](/htb_img/WEBRESOURCEf4ddae0037f9c4e6abf55c49acb44dafimage.png)

然后点进去就可以看到输出了：

![](/htb_img/WEBRESOURCE75afbb54e1a84516be5f44eb8ae040d7image.png)

接下里就是命令执行：

失败了，好像是说网络错误：

![](/htb_img/WEBRESOURCEb3cbf0a230caceaa0e8e7e2ffc70e134image.png)

这里涉及知识盲区：

jenkins的用户信息存放位置：

c:\Users\oliver\Appdata\local\jenkins\.jenkins\users

然后可以看到admin用户的文件夹：

![](/htb_img/WEBRESOURCE62acc9d58da7e28c97ddba278c03bccaimage.png)

读取里面的config.xml文件：

就可以看到加密的密码：

![](/htb_img/WEBRESOURCE5894a52362dc6d332dfc3bc3c0a7a3cbimage.png)

详情可以参见：

[https://github.com/hoto/jenkins-credentials-decryptor](https://github.com/hoto/jenkins-credentials-decryptor)

上面那个文件，其实是

![](/htb_img/WEBRESOURCEcf27eed06a4e20c411325bb133328d47image.png)

master.key:

f673fdb0c4fcc339070435bdbe1a039d83a597bf21eafbb7f9b35b50fce006e564cff456553ed73cb1fa568b68b310addc576f1637a7fe73414a4c6ff10b4e23adc538e9b369a0c6de8fc299dfa2a3904ec73a24aa48550b276be51f9165679595b2cac03cc2044f3c702d677169e2f4d3bd96d8321a2e19e2bf0c76fe31db19

![](/htb_img/WEBRESOURCEc05251d5c94adab59ea51a062aa689ccimage.png)

hudson.util.Secret是二进制文件，使用base64读取出来：

powershell.exe -c "$c=[convert]::ToBase64String((Get-Content -path 'c:\Users\oliver\Appdata\local\jenkins\.jenkins\secrets\hudson.util.Secret' -Encoding byte));Write-Output $c"

![](/htb_img/WEBRESOURCE3b84fdace41026f625a8d2d3bcf052afimage.png)

gWFQFlTxi+xRdwcz6KgADwG+rsOAg2e3omR3LUopDXUcTQaGCJIswWKIbqgNXAvu2SHL93OiRbnEMeKqYe07PqnX9VWLh77Vtf+Z3jgJ7sa9v3hkJLPMWVUKqWsaMRHOkX30Qfa73XaWhe0ShIGsqROVDA1gS50ToDgNRIEXYRQWSeJY0gZELcUFIrS+r+2LAORHdFzxUeVfXcaalJ3HBhI+Si+pq85MKCcY3uxVpxSgnUrMB5MX4a18UrQ3iug9GHZQN4g6iETVf3u6FBFLSTiyxJ77IVWB1xgep5P66lgfEsqgUL9miuFFBzTsAkzcpBZeiPbwhyrhy/mCWogCddKudAJkHMqEISA3et9RIgA=

最后使用的这个：

[https://raw.githubusercontent.com/gquere/pwn_jenkins/master/offline_decryption/jenkins_offline_decrypt.py](https://raw.githubusercontent.com/gquere/pwn_jenkins/master/offline_decryption/jenkins_offline_decrypt.py)

得到密码：

c1cdfun_d2434

![](/htb_img/WEBRESOURCEc3658fdfb49b5c2689fdd4f3091ea4acimage.png)

用户名是：oliver

![](/htb_img/WEBRESOURCE19f8eb3e601ac3ee31c68eae2f07d21eimage.png)

获取立足点：

![](/htb_img/WEBRESOURCEf0ff78f5b371170070a4e0a5ef5cb079image.png)

# 提权

这里脑子卡壳了，以为是防火墙限制了，所以无法上传文件来着，但是实际上可以通过各种方法写入，甚至是winrm可以上传下载文件：

使用bloodhound就可以成功获取路径了：

![](/htb_img/WEBRESOURCE23899d8fbbc03284e217c2eae1748cafimage.png)

![](/htb_img/WEBRESOURCE9f5cdf45bf6f61f96d6383ad53ef2f81image.png)

登录成功：

![](/htb_img/WEBRESOURCEcdb628f4242453756045a60c98412ed4image.png)

根据bloodhound的指示，我们可以修改MARIA对象的所有属性：

![](/htb_img/WEBRESOURCE545a8ba1a64b2205f338b585e4388289image.png)

但是根据查找，这个属性对于用户的利用方式，一般是修改登录脚本，以获取那个用户的权限：

这里由于存在防火墙，所以只能我弹我自己了：

![](/htb_img/WEBRESOURCEb4bde666479c1abf7f4796063abc1357image.png)

这里其实已经成功执行脚本，弹回来了：

![](/htb_img/WEBRESOURCE55acd7ceda13ea4ad86a814113ed7708image.png)

但是核心问题在于这是一个非交互式的命令行，但是我们可以通过写入文件的方式来获取回显：

![](/htb_img/WEBRESOURCE00e0f2c77929b6bb71946191f6c71362image.png)

但是根据bloodhound的提示，我们可以使用WriteOwner权限来更改domain admins组的owner，理论上，我们可以直接执ps脚本，达到修改owner为已有用户的情况，但是实际操作发现不知道哪里不对，通过信息收集，在桌面发现一个xls表格：

![](/htb_img/WEBRESOURCE32c98b88e6fe12c931b24d59d8809746image.png)

最后发现W3llcr4ft3d_4cls是对的：

![](/htb_img/WEBRESOURCEf53c80ab3657e65e2b3a4c243fb2f3cfimage.png)

接下里就按照bloodhound的指示操作即可：

powerview害人不浅：

唯一指定：

[https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1)

![](/htb_img/WEBRESOURCE475a94cbb49414333165fef78ac2b712image.png)

重新登录即可：

![](/htb_img/WEBRESOURCE570e4c2059c69024fa6ab552d2cec2abimage.png)