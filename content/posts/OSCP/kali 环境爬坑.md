+++
title = 'kali 环境爬坑'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'OSCP笔记'
+++

python2 安装pip问题：

```
wget https://bootstrap.pypa.io/pip/2.7/get-pip.py
python2 get-pip.py
python2 -m pip install --upgrade setuptools -i https://pypi.tuna.tsinghua.edu.cn/simple
```
pip2 安装impacket:

```
python2 -m pip install impacket==0.9.22
```

### kali安装bloodhound


```
sudo apt-get install neo4j
sudo neo4j console
控制台默认账密：neo4j
配置好后，安装bloodhound
sudo apt-get install bloodhound
输入修改后的密码

ps1脚本使用：
Import-Module ./SharpHound.ps1; Invoke-BloodHound -c all
```


### kali 使用impacket--docker

```
安装docker
apt-get install docker.io
systemctl start docker

下载相关文件夹：
构建 Impacket 的镜像：
docker build -t "impacket:latest" .
使用 Impacket 的镜像：
docker run -it --rm "impacket:latest"
```

### 安装impacket--推荐此方式

```
git clone https://github.com/CoreSecurity/impacket.git 
cd impacket/ 
python3 -m pip install . 
sudo python3 setup.py install
```
### No module named "Crypto"

```
解决方案：
pip install pycryptodome
```

# openvpn over socsk5

```
socks-proxy 192.168.5.2 7890
```


# /etc/samba/smb.conf 配置


注释：

map to guest = bad user

```
[mane]
   comment = mane testing
   path = /home/kali/Desktop/smb
   writeable = yes
   guest ok = yes
   read only = no
   force user = root
```

# version `GLIBC_2.34' not found

两个办法：

```
https://www.cnblogs.com/tolele/p/16534014.html



更换链接器：
patchelf /home/kali/Desktop/pen-200/Linux/glibc-all-in-one/libs/2.27-3ubuntu1_amd64/ld-linux-x86-64.so.2 ./easy_heap

更换glibc搜索路径：
patchelf --set-rpath /home/kali/Desktop/pen-200/Linux/glibc-all-in-one/libs/2.27-3ubuntu1_amd64 ./easy_heap(注意路径和可执行文件（参数）要按自己的来修改)
```

```
查看版本：
strings /lib/x86_64-linux-gnu/libc.so.6 |grep GLIBC_

https://blog.csdn.net/weixin_65527369/article/details/127973141


gcc -Wl,-rpath='/home/kali/Desktop/pen-200/Linux/glibc-all-in-one/libs/2.27-3ubuntu1_amd64',-dynamic-linker='/home/kali/Desktop/pen-200/Linux/glibc-all-in-one/libs/2.27-3ubuntu1_amd64/ld-linux-x86-64.so.2' -s rootshell.c -o rootshell4

```

二是升级目标主机的libc6

```
sudo apt update 
sudo apt install libc6
```