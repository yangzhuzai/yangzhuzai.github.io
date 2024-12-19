+++
ShowToc = true
TocOpen = true
title = 'shiro'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'Java安全'
+++


# 一、shiro

Apache Shiro是用来做认证和授权的框架，执行身份验证、授权、密码和会话管理。


Shiro主要配合一些容器的使用，如Tomcat、Weblogic等；同时有些框架也会将Shiro集成用来做身份认证和授权，比如：SpringBoot等；

# 二、漏洞原理

### CVE-2016-4437/Shiro-550

Shiro < 1.2.5

第1个注意的事项：要在response中返回含有remenberMe的字段对应的请求中去构造poc


第2个注意的事项 ：获取cookies的请求包resquest的必须是登录提交的http请求


（1）shiro在登录处提供了Remember Me这个功能，来记录用户登录的凭证，然后shiro会对用户传入的cookie进行解密并进行反序列化，服务端接收rememberMe的cookie值后的操作是：


Cookie中rememberMe字段内容      	        


        ---> Base64解码       		      


                         ---> 使用密钥进行AES-CBC解密            			 


                                                    ---> 反序列化


（2）由于该版本AES加密的密钥Key被硬编码在代码里（漏洞能够被利用的本质），且大部分项目未修改默认AES密钥，这意味着攻击者只要通过源代码找到AES加密的密钥，就可以构造一个恶意对象，对其进行序列化，AES加密，Base64编码，然后将其作为cookie的Remember Me字段值发送，Shiro将数据进行解密并且反序列化，最终触发反序列化漏洞。

官方针对这个漏洞的修复方式将使用默认硬编码的Key改为生成随机的Key加密。

密钥保存位置：

![](/zj_img/WEBRESOURCEe42a394535820c01dae3f4e45464ea1dimage.png)

最后的触发位置是标准的readobject()，所以利用链基本上就是CC链或者其他依赖链了。

### CVE-2019-12422/Shiro-721

Shiro < 1.4.2

在Apache Shiro 中用户进行登录的时候, 提供RemenberMe功能处理cookie，使用的是AES-128-CBC加密，可以通过Padding Oracle加密生成的攻击代码来重新构造一个恶意的rememberMe字段，可进行反序列化攻击。

跟Shiro无关，而是对Shiro采用的加密方式进行的攻击，所以略过，只要了解了Padding Oracle Attack 原理就能理解这个攻击的原理，这里推荐Padding Oracle Attack(填充提示攻击)详解及验证。

简单来说，Padding Oracle Attack是根据CBC字节翻转攻击、Padding规则以及服务端解密后返回的不同状态来穷举中间值进而获取明文的攻击。解密出的明文填充的正确与否可通过服务端的响应进行区分。通过不断调整发送的密文分组，然后根据服务端的结果进行爆破测试，可推测出确定的解密值。当应用程序接受到加密后的值以后，可能有以下三种返回：





1、接受到正确的密文之后（填充正确且包含正确的值），应用程序正常返回：200 OK。





2、接受到合法的密文（填充正确，但解密后得到一个不正确的值），应用程序显示自定义错误消息：200 OK。





3、接受到非法的密文之后（解密后发现填充不正确），应用程序抛出一个解密异常：500 Internal Server Error。

我们不需要要求解密后必须是正确的值，只要返回200 ok即可。

aes gcm加密方式不一样的

### CVE-2020-1957/认证绕过漏洞

Shiro <  1.5.2

spring Boot中使用Apache Shiro进行身份验证、权限控制时，可以精心构造恶意的URL，利用Apache Shiro和Spring Boot对URL的处理的差异化，可以绕过Apache Shiro对Spring Boot中的Servlet的权限控制，越权并实现未授权访问。

漏洞发生的跟本原因是shiro对于URL的处理和spring Boot的处理逻辑存在偏差，当URL进入服务器后，处理顺序为先经过shiro后经过spring Boot

首先，/admin是需要鉴权才可以访问的路径。

我们发送的恶意/xxxxx/..;/admin请求首先经过Shiro进行处理，在shiro中经过处理变为/xxxxx/..

```
替换反斜线
替换 // 为 /
替换 /./ 为 /
替换 /../ 为 /
```

随后传入spring Boot，经过JDK解析从Mapping中得到Servlet结果为/admin，导致了未授权访问。

[http://192.168.5.105:8080/xxx/..;/admin/](http://192.168.5.105:8080/xxx/..;/admin/)

![](/zj_img/WEBRESOURCEc397eaa92631e158d0c6710695e4454aimage.png)

# 三、漏洞利用

梭哈、内存马注入和key修改基本上都已经集成了，没有操作难度。

# 四、其他

面试常问shiro rememberMe的字段超过header头限制该怎么解决，目前的解决方案：

1、通过类动态加载机制，使用post方法提交数据，加载post体中的字节码。包括目前主流的工具基本上都是这么做的。

2、通过动态修改tomcat maxHttpHeaderSize属性的值

3、Gzip+Base64压缩编码类字节码：将Payload的一部分添加到ClassLoader的classes字段