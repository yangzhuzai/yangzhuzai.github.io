+++
title = '0x010 My File Server 1'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE607803e9db4e385a56041f94df03737c截图.png)

![](/vulnhub_img/WEBRESOURCE240a54f550c9359a780308c07f403444截图.png)

![](/vulnhub_img/WEBRESOURCEdf0dece76cb4302705883bc750ea25dd截图.png)

# 信息收集

21端口，有匿名登录，但是目前来看没有发现什么有效的东西：

![](/vulnhub_img/WEBRESOURCE677f3992f3e0efff0fe2c03e96da0ba8截图.png)

2049 nfs端口，这里遇到一个问题，似乎只有同一网段才能挂载成功：

![](/vulnhub_img/WEBRESOURCE8007cff462e94048660b983350617ae7截图.png)

NFS内容看起来和FTP一样，但是权限高不少：

![](/vulnhub_img/WEBRESOURCEf375917bae4dd8a17b222f00e470afa3截图.png)

![](/vulnhub_img/WEBRESOURCEbb98612aa38d32ded88fcafb7336aabf截图.png)

![](/vulnhub_img/WEBRESOURCE109d19ed67363acb230a99bbd97e3209截图.png)

![](/vulnhub_img/WEBRESOURCE50d6e5cd0e6a2a2f46411b02eaa27f3e截图.png)

![](/vulnhub_img/WEBRESOURCE97b604408d6b3284662362fcedb6acdc截图.png)

发现多个用户名，根据家目录和环境判断，其中smbuser大概率是可以登录的：

![](/vulnhub_img/WEBRESOURCE5317791457913edc6ac6a9ad0860b71d截图.png)

80端口：

![](/vulnhub_img/WEBRESOURCE7ee02d9077943cf97805a4e36613514b截图.png)

![](/vulnhub_img/WEBRESOURCEae947895ae357079bbd9cdaf81bb4d6e截图.png)

发现密码rootroot1

![](/vulnhub_img/WEBRESOURCE16a0df2d3c0a92a7fabd73b97d0be665截图.png)

但是ssh连接失败：

![](/vulnhub_img/WEBRESOURCE071efa53ccab0d3ddac78cc78582e819截图.png)

那就剩下最后一个端口：445

![](/vulnhub_img/WEBRESOURCE438cea2412426bd21bfbc4e1c10e90d8截图.png)

匿名登录：

![](/vulnhub_img/WEBRESOURCE83f99c60a3f2f3697796a29a70c87b16截图.png)

![](/vulnhub_img/WEBRESOURCE1bbe1161602cbe40f7a46b5dfd5c26bf截图.png)

说实话打到这里基本上，已经没有思路了，基本能看的地方都看了，手里拿到了一个账户和密码，但是无法登录，看了一下wp，让我登录FTP再看看，没想到再次登录FTP的：

![](/vulnhub_img/WEBRESOURCEc7134c5e3fb395216419ede80cc585be截图.png)

生成公私钥：

![](/vulnhub_img/WEBRESOURCE108ba492ec7b53f957a68f6787ae116e截图.png)

上传：

![](/vulnhub_img/WEBRESOURCE4b584b30ca7646c25a77a865c385aa39截图.png)

这里有坑，使用kali默认的私钥名字连不上，改成其他名字一下就可以了

# 提权

有GCC可以用内核提权：

![](/vulnhub_img/WEBRESOURCE79fe9927799a5d921bc450f15d8075d3截图.png)

随便找了一个2021-4034：

![](/vulnhub_img/WEBRESOURCE7b3aaa2c0007045e363d161bd0eb24b0截图.png)

# 总结

此靶机不是单纯的集中某个技能点，而是需要反复的进行信息收集，极其容易导致遗漏。