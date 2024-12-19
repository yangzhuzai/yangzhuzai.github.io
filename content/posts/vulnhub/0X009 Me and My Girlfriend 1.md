+++
ShowToc = true
TocOpen = true
title = '0x009 Me and My Girlfriend 1'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCEdfffdca74157c998c57878189cc37a67截图.png)

![](/vulnhub_img/WEBRESOURCE936c2661ff43147fa789ee757caf2611截图.png)

# 目录扫描

![](/vulnhub_img/WEBRESOURCEd022d77849741000475cd353aa170648截图.png)

# WEB渗透

打开页面会提示只允许本地登录，通过插件添加x-forwarded-for字段即可：

![](/vulnhub_img/WEBRESOURCE6e4d266c1607287caa3b678a449cb67a截图.png)

注册账号登录，发现越权，获得账号密码，可能存在sql注入？后面跑了一下，看样子是没有：

Eweuh Tandingan

eweuhtandingan

skuyatuh

![](/vulnhub_img/WEBRESOURCEb070de05e0ed111d12d493530ad08e76截图.png)

用burp把所有的用户名提取出来：

两个问题：1是kali自带的burp为社区版，不支持导出，2是脚本有点垃圾，需要手动删除一点儿垃圾

```
import os

def extract_name(file_path):
    with open(file_path, 'r') as file:
        content = file.read()
        start_index = content.find('id="password" value="')
        if start_index != -1:
            start_index += len('id="password" value="')
            end_index = content.find('"', start_index)
            if end_index != -1:
                return content[start_index:end_index]
    return None

def process_files(folder_path):
    output_file = 'password.txt'
    with open(output_file, 'w') as file:
        for root, dirs, files in os.walk(folder_path):
            for file_name in files:
                file_path = os.path.join(root, file_name)
                name = extract_name(file_path)
                if name is not None:
                    file.write(name + '\n')

folder_path = './'
process_files(folder_path)

```

成功获得账户密码：

User: alice Password: 4lic3

![](/vulnhub_img/WEBRESOURCE0fe127b749bbdbc6b3f786614fce9ce7截图.png)

# 提权

![](/vulnhub_img/WEBRESOURCE586222caf7198b22b063776ddedb09c7截图.png)

已经比较熟练了，开局就是sudo php：

是个哑shell，我还以为没成功呢：

![](/vulnhub_img/WEBRESOURCE6ad93c7b3acbe609130bf3344b5a0670截图.png)

找一下userflag:

![](/vulnhub_img/WEBRESOURCE1a24fee8d7239c77c2fe33c007bd08f0截图.png)

![](/vulnhub_img/WEBRESOURCEcea40d003288ba6ca6fea8416f01ad61截图.png)