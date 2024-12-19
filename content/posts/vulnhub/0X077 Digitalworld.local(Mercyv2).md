+++
ShowToc = true
TocOpen = true
title = '0x077 Digitalworld.local(Mercyv2)'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE74f817d050008763895c9aeeadb1bc6fimage.png)

![](/vulnhub_img/WEBRESOURCE76d1c146ffa9dc152a48b68a0cc29489image.png)

# 渗透测试

SMBA没有东西，那么就是HTTP 8080了：

![](/vulnhub_img/WEBRESOURCE3f77b34b64dd6df5c8b8ba2932aec50fimage.png)

![](/vulnhub_img/WEBRESOURCEe072604120ca403c1f374a964aad5940image.png)

解密出来：

It's annoying, but we repeat this over and over again: cyber hygiene is extremely important. Please stop setting silly passwords that will get cracked with any decent password list.

Once, we found the password "password", quite literally sticking on a post-it in front of an employee's desk! As silly as it may be, the employee pleaded for mercy when we threatened to fire her.

No fluffy bunnies for those who set insecure passwords and endanger the enterprise.

这里说有默认密码：

![](/vulnhub_img/WEBRESOURCEaaa435148680824b3adfe8ca0403fd1eimage.png)

这里有几个思路，一个是smb，一个tomcat的弱密码，tomcat简单试了一下，password是不行的，那就是smb了，用enum4linux扫描一下：

![](/vulnhub_img/WEBRESOURCE09c5ec58b1d83bdcdcbe16492c7c5893image.png)

![](/vulnhub_img/WEBRESOURCE9c5cc76f1179197cf5c84002ce0b44d6image.png)

登录成功：

![](/vulnhub_img/WEBRESOURCE77f60907ec6206ce1761e1a517b88415image.png)

看目录，应该是家目录了，但是没有开ssh，只能寻找其他路子：

发现隐藏目录：

![](/vulnhub_img/WEBRESOURCEbeec92c3ab1b5bbba6cf61a2609cbe67image.png)

可以开启HTTP和SSH：

![](/vulnhub_img/WEBRESOURCE0d70413cf97ad56c3e38f697986a4aaeimage.png)

![](/vulnhub_img/WEBRESOURCEd7465e72ada05140f3bba2de70df2247image.png)

把ssh和80开启来：

![](/vulnhub_img/WEBRESOURCEc125b6846face4504747e56047833366image.png)

![](/vulnhub_img/WEBRESOURCEf8efa8c62b004ec35f4b6c29911c5dfaimage.png)

![](/vulnhub_img/WEBRESOURCE3d9b8ff006881b75f1e0cc6e0c9677a4image.png)

记录一下，指定用户名的smb挂载：

```
sudo mount -t cifs -o username="qiu",password="password" //192.168.5.103/qiu  smb
```

当前目录没有写入ssh的权限，可能要看看80了：

![](/vulnhub_img/WEBRESOURCE7b7687136169fdc1ce2dd40ec9586f04image.png)

![](/vulnhub_img/WEBRESOURCE35b1904feee4ed4b4be1c746c5769cdfimage.png)

![](/vulnhub_img/WEBRESOURCE83f39e44a6fd01372f9cd80ee8109e2dimage.png)

搜索漏洞：

![](/vulnhub_img/WEBRESOURCEa28aa04ee2586a018729f8f18cf676beimage.png)

![](/vulnhub_img/WEBRESOURCE0f3da2d599456e73813d0f5318e4159cimage.png)

![](/vulnhub_img/WEBRESOURCE4de3c3f0117a28560f8fa7b46ff7f0fbimage.png)

文件读取，那么剩下的路就是读取有用信息了，我这里有几个想法：1、读取smb扫出来的另外一个用户的key；2、读取数据库密码；3、读取tomcat的密码。

第一个尝试，失败了，第二个不知道路径，第三个读取成功：

"thisisasuperduperlonguser" password="heartbreakisinevitable"

"fluffy" password="freakishfluffybunny"

![](/vulnhub_img/WEBRESOURCEa6caa2ef9d2f1a0aa38507eb28dccaa3image.png)

后面就是部署war包getshell了：

msfvenom生成shell的大全：

[https://blog.csdn.net/Sagittarius32/article/details/125779426](https://blog.csdn.net/Sagittarius32/article/details/125779426)

![](/vulnhub_img/WEBRESOURCE6eb7c4e274c6ff73592897ccd07d6f8eimage.png)

打包jsp文件：

```
jar -cvf  webshell.war shell.jsp
```

kali的原生环境java可能有些问题：

![](/vulnhub_img/WEBRESOURCE77ea58a0d94624057443b796b8442ecbimage.png)

![](/vulnhub_img/WEBRESOURCE4ee33ebba083f2c1ddc862041e0cb48dimage.png)

![](/vulnhub_img/WEBRESOURCE03bbfa8fad429cf79bb1ceb13c973368image.png)

# 提权

经过测试，qiu和fluffy都是可以登录的吗，但是也都没有sudo权限：

![](/vulnhub_img/WEBRESOURCEd4dab5d696f403786f4826bded763576image.png)

发现疑似提示：

![](/vulnhub_img/WEBRESOURCEe2a055dc4643c8c4b1fd78dcfcb30119image.png)

发现有趣的东西了：

这个地方的这个内容，其实和我们在web目录中的内容一致：

![](/vulnhub_img/WEBRESOURCE186883e61d4a24db2b439b2f36e51adeimage.png)

![](/vulnhub_img/WEBRESOURCE79d429a4067d7c3407d6df657b2cf7d3image.png)

所以有理由猜测，这个脚本是控制这个内容输出的，把这个内容添加反弹shel脚本：

![](/vulnhub_img/WEBRESOURCEacf8af5faf7b14ba4a424f5a6227c9fdimage.png)

![](/vulnhub_img/WEBRESOURCEd4f2976683bb1bc59abd5c78305c7f34image.png)