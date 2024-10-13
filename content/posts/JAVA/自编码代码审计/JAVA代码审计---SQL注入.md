+++
title = 'SQL注入'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'Java自编码代码审计'
+++

# 前言

SQL注入较为简单，花个几天时间即可入门，笔者认为相当划算。但是多多少少需要了解一下java基础，javase，至少需要知道类、对象、接口、封装、重载等等概念，并且建议上手实现几个小功能，从零开始的话，菜鸟教程就行。

然后就可以学习spring+mybatis了，笔者跟着这个视频学习的：[https://www.bilibili.com/video/BV1Qj41197my/?spm_id_from=333.880.my_history.page.click&vd_source=48a3e1b690916d7ddc8d6d70327a775c](https://www.bilibili.com/video/BV1Qj41197my/?spm_id_from=333.880.my_history.page.click&vd_source=48a3e1b690916d7ddc8d6d70327a775c)

实际操作要点：知道怎么实现接口以及怎么到最后的sql查询，怎么映射，建议完全跟着视频手敲一遍。

概念要点：理解什么是controller层、service层、dao层，以及部分idea使用方式（这部分弯路可能必须得走，就和最开始使用burp一样），还有就是maven的使用。

有了这些基础，SQL注入就很简单了。

# SQL注入，mybatis：

审计层：dao、service、controller

需要先知道MyBatis易产生SQL注入漏洞的场景： 

1. Like 模糊查询： Select * from student where name like ‘%${name}%’ 

2. IN 查询： Select * from news where id in (${id}) 

3. order by、group by 查询： select * from studentwhere id order by ${id}  

目标采用MyBatis持久层框架，MyBatis造成的sql注入主要是由于使用${param}方式传参导致，我们在持久层中查找$符号，主要是dao层和对应的xml文档：

发现大量目标：

![](/sj_img/WEBRESOURCE8d2aee95558aa0ebe56db67b26fa1619image.png)

点进去，发现$拼接：

![](/sj_img/WEBRESOURCE9f56d3a4659c8902862c5d4c58dfdb63image.png)

追溯到dao层：

![](/sj_img/WEBRESOURCEb92d8a3264265c732f99bd3d7f66d0f4image.png)

继续追溯到service层：

![](/sj_img/WEBRESOURCEf4687568d8c6e2de0084d9db898fde14image.png)

继续追溯用法到controller层：

![](/sj_img/WEBRESOURCE93c4c81c29cc65367291d42c98bccb6dimage.png)

拼接上层的url:

路径为：

/uc/deleteFaveorite/{ids}

ids为注入点：

由于存在身份校验，所以需要合法身份，是个后台sql注入：

![](/sj_img/WEBRESOURCE1f514d829ec7e8d667d43b2615886fd4image.png)

但是第二处，虽然最后是同一个dao层的注入点，但是service层调用的controller层不同：

![](/sj_img/WEBRESOURCEd85f37d82cbb0c23a82088f797f8c1f3image.png)

阅读controller层的代码，不难发现，此处没有此处没有身份校验：

![](/sj_img/WEBRESOURCE1bd5505bf00e48f6426b0e29d74da93bimage.png)

路径为：

/webapp/deleteFaveorite

看代码是get获取参数，构造一个id=1

发现返回成功的提示，那基本上就确定是未授权的sql注入了：

![](/sj_img/WEBRESOURCE54d10b928713caa8be7f4f201007fa4dimage.png)

构造路径和传参，-u参数即可注入成功：

![](/sj_img/WEBRESOURCE045b233795c4bfb31616de8a98f4fa89image.png)

# SQL注入，JDBC

JDBC场景下的SQL注入其实就一个，那就是使用

```
String sql = "select * from users where id = '"+id+"' ";
```