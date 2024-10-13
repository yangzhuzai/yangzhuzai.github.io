+++
title = '反序列化'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'Java自编码代码审计'
+++

# 前言

**JAVA反序列化其实并没有那么神秘，只需要理解几个概念即可！！**

如果是从开发的角度来看，序列化是让不同的应用程序之间交换逻辑数据的，具体内容应该是

从渗透的角度来看，和大多数漏洞一样，我们考虑两个问题：

# **基础**

**来看一个简单的反序列化例子：**

类的代码如下，可以看到类中有一串恶意代码，位于重写的

![](/sj_img/WEBRESOURCEa98204dc30e9afbbc51d8f05dee04925image.png)

然后看后端代码，这里的代码意思是，服务端接收客户端传来的二进制数据，然后通过readobject方法反序列化成对象，那么这里调用了readobject方法，就会执行类中的恶意代码：

![](/sj_img/WEBRESOURCE69059152a8eb2e215b55cdad94fcebe5image.png)

然后看看客户端怎么操作：

![](/sj_img/WEBRESOURCE1367ecec723bf3562dc69526d1e55691image.png)

观察上面的整个过程，我们不难发现，漏洞发生点在于ObjectInputStream.readObject这个地方。我们总结几个东西：

两个问题怎么解决：

**先说第二个问题**

**再说第一个问题**

第一个是开发者自己写的类存在这么一个反射的地方，

第二个是利用java的基础包中的漏洞，因为java开发中，肯定会导入一些基础的代码，其中一个叫做

引入GPT的回答:

![](/sj_img/WEBRESOURCEec692797c2a69439d66c3aa54e8e11d3image.png)

#### **好，看懂这里，其实就理解了整体思路了，我们现在来理解反射机制**

先看看官方的回答，其他的都是废话，我们就看这两个红框就行了：

![](/sj_img/WEBRESOURCE394a7e2b59200f87b1013379c959bd49image.png)

那么反射一般应用于什么情况下呢？，正常开发是大概率不会考虑这些问题的，因为普通开发者开发的应用就是为了完成功能，当前跑着的应用能正常使用就行了，也就不会去使用反射这个机制：

![](/sj_img/WEBRESOURCE50b52a9d808768ca28299131a90365f0image.png)

#### 好，现在我们去看看一个反射是如何调用的：

反射的作用在于让我们可以调用任何方法，我们看看如何实现调用runtime.exec：

```
public class ExecClass {
    public static void main(String[] args) throws Exception {
        // 获取Runtime类对象
        Class runtimeClass1 = Class.forName("java.lang.Runtime");

        // 使用Runtime的Class对象获取构造方法
        Constructor constructor = runtimeClass1.getDeclaredConstructor();
        // 反射修改构造器访问控制
        constructor.setAccessible(true);

        // 构造器反射创建Runtime类实例
        Object runtimeInstance = constructor.newInstance();

        // 获取Runtime的exec(String cmd)方法
        Method runtimeMethod = runtimeClass1.getMethod("exec", String.class);

        // 调用exec方法
        Process process = (Process) runtimeMethod.invoke(runtimeInstance, "calc");
        //精简：runtimeMethod.invoke(runtimeInstance, "calc");
```

上面注释已经写了，首先通过Class.forName接完全限定域名的方式，获取类对象，然后通过getDeclaredConstructor获取构造方法，修改构造方法中的setAccessible为true，随后创建了一个名为runtimeInstance的实力，使用的构造方式是上一步中修改好了的构造方法，这一步中，我们的类就修改完毕了，随后获取原来的runtimeClass1对象的exec(String cmd)方法，最后调用我们的方法即可，最后一句可以精简为：runtimeMethod.invoke(runtimeInstance, "calc");

再来看一个实例，这是一个类中存在的反射机制：

简单解释一下，前面是传入一个对象，只要不为空，即可进入try里面的内容：

**然后看看try干了什么：**

**首先是获取了input也就是传入的实例的class对象，其实和上面代码的第一步一模一样；**

**第二步是，获取class对象的iMethodName方法；**

**第三步是，通过该方法调用传入参数iArgs并执行；**

![](/sj_img/WEBRESOURCE435cff8ae98ed54e7f3414a0c8acabe4image.png)

**我们魔改一下，假如，我们传入的是一个**

**第一步就是获取runtime的class对象；**

**第二步就是获取exec执行方法；**

**第三步就是传入calc调用exec方法；**

![](/sj_img/WEBRESOURCE421dadb116b15d95b18e357a0f47846fimage.png)

**那么这串代码出自哪里呢？来自apache **

**看懂这里，就基本搞明白了反射机制了**

![](/sj_img/WEBRESOURCEd3a054d72972bb4eb8ea26a135b3390eimage.png)

**但是既然它本身没有重写readObject**

**CC链的构成其实比较复杂，非常考验逻辑性，但是如果作为代码审计来说，反序列化其实并不难，因为我们不需要现挖CC链（而且我也没那个能力），有现成的链能用就行，常见工具为：**

**支持列表如下：**

![](/sj_img/WEBRESOURCE9bd1419cfefbb895b20357501fcf92a9image.png)

# **审计**

在 java 中，对象的序列化和反序列化被广泛的应用到 RMI（远程方法调用）及网络传输中：

我们在审计中关注的关键词为：

```
ObjectInputStream.readObject
ObjectInputStream.readExternal
ObjectInputStream.readUnshared
XMLDecoder.readObject
Yaml.load
XStream.fromXML
ObjectMapper.readValue
JSON.parseObject
```

审计层为controller层：

![](/sj_img/WEBRESOURCE61c592a7f354d0b84272f63cea54d732image.png)

跟进代码：

发现readObject方法，这个地方的解释很简单，接口为/Deserialize/readObject/vul，参数为base64

简单构造一下，就是/Deserialize/readObject/vul?base64=后面就跟反序列后base64了的对象，CC链慢慢试即可：

![](/sj_img/WEBRESOURCEe707f7d1b019e366f7e6176045daff9cimage.png)

实操如下：

**ysoserial成payload:**

java -jar ysoserial-all.jar CommonsCollections5 calc.exe > 1.txt

![](/sj_img/WEBRESOURCE26fae081af89e0dc6ad533a3ed2c5978image.png)

![](/sj_img/WEBRESOURCEecba0d6b5ad227e5954a47befb937150image.png)

**base64编码：**

![](/sj_img/WEBRESOURCEb007c78dbf7503a2ec18363e35ff7a3fimage.png)

**清除换行符等：**

![](/sj_img/WEBRESOURCE911c6fb0700dba8bc9606dc9b9736d8aimage.png)

**发送payload:**

![](/sj_img/WEBRESOURCEbfdb716bdb63a1ea113655acde4b88ebimage.png)

**经过实测，CC5、CC6都可以**

![](/sj_img/WEBRESOURCEa51fc6235a9b8e5729f7fa732057d5dbimage.png)