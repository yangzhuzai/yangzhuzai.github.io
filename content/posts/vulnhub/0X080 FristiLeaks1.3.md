+++
title = '0x080 FristiLeaks1.3'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



看描述，要改mac地址：

![](/vulnhub_img/WEBRESOURCE7cbcb733e1b1ee42b8d53ada55ff0c14image.png)

![](/vulnhub_img/WEBRESOURCE22cdc31d77af424071b51844e6dc9141image.png)

# 渗透

邪门儿，意思是说，这个页面的drink fristi的意思是，让我们看这个目录：

![](/vulnhub_img/WEBRESOURCE65c8ce50c1eab3d9279d86dbc8b47767image.png)

![](/vulnhub_img/WEBRESOURCE00711889079ee7e4c9741a0adee9e51dimage.png)

把上面的base64替换掉，然后浏览器访问，搞出来是这个样子：

keKkeKKeKKeKkEkkEk

![](/vulnhub_img/WEBRESOURCEc79b22f8f077ee98fb42f631fa0e7dd0image.png)

admin不对：

![](/vulnhub_img/WEBRESOURCE98303ddf7e04a91fc86195d5f496e617image.png)

但是eezeepz可以：

![](/vulnhub_img/WEBRESOURCE0675063cfe06ab5933de6b18d090b2b1image.png)

随后是文件上传，能上传shell,jpg但是没有文件包含：

![](/vulnhub_img/WEBRESOURCE4b9331c91a97b1f942286d5515ecaac2image.png)

呐应该是可以绕过，比较老生常谈的apache 解析漏洞：

![](/vulnhub_img/WEBRESOURCEfddf28994a9b7122fe89c05508bac2f2image.png)

![](/vulnhub_img/WEBRESOURCE46dd7f085d3897b40b7f572a1251f847image.png)

# 提权

在eezeepz的家目录下面发现了这样一句话：

![](/vulnhub_img/WEBRESOURCEa068681661dd102587820abb194a3520image.png)

计划任务：

![](/vulnhub_img/WEBRESOURCE9c40374329eefd3c82252362bc537fa6image.png)

然后，这里可以知道提示的原因是啥了：

![](/vulnhub_img/WEBRESOURCEa81f153d8cbee3660747b9236e5905c2image.png)

然后仔细看了一下，这些个命令，看看有啥能用的，其实我是想要创造一个属于admin用户的suid文件，然后提权的，但是试了半天没成功，看看网上的思路，是把admin用户的目录给777了：

然后看了一下：

这里面没有那么多命令，比图touch等，但是echo 和chmod有：

![](/vulnhub_img/WEBRESOURCEc3d83a01ccb4ac445f78281e18dba99eimage.png)

改一下脚本内容：

需要先用admin用户创建一个所有人可读可写的cxk文件：

![](/vulnhub_img/WEBRESOURCE67ad04eea8620e72fc19b6969f6e9af3image.png)

用当前用户修改cxk的内容为bash:

![](/vulnhub_img/WEBRESOURCEc9afa68897e0499c8a70f58b0675c429image.png)

重新写一个u+s权限即可：

![](/vulnhub_img/WEBRESOURCEedd0bb03c921141f29f1c0fc7a04503aimage.png)

![](/vulnhub_img/WEBRESOURCEec93b43361685d7037a470fab09917c8image.png)

切换成admin了：

![](/vulnhub_img/WEBRESOURCE139b4ee36b30b18f65bd8348a0cb57e5image.png)

damin目录下：

![](/vulnhub_img/WEBRESOURCE3b7ca5dc3c6e8d5320933e91933ac112image.png)

看样子是需要解密这串密文，还是比较好理解的，先是base64，然后rot13，想办法解开即可，这代码不难，只需要把en改成de即可：

![](/vulnhub_img/WEBRESOURCE0bad58714866f72484684c3d959306aeimage.png)

![](/vulnhub_img/WEBRESOURCEe8915ff60022cc426fb81603a61d2aa7image.png)

![](/vulnhub_img/WEBRESOURCEe9032ac27a5f80d87422861a93e1bf38image.png)

sudo -l:

![](/vulnhub_img/WEBRESOURCEc85f58a2e71f9b073d61af3e869c124bimage.png)

这里尝试后，就知道，作者这里在提示了：

![](/vulnhub_img/WEBRESOURCE62c480efd0cfd3d75a81fc8f79e4b16aimage.png)

![](/vulnhub_img/WEBRESOURCEc6dd12379b98b952cce136f9aed5f987image.png)

![](/vulnhub_img/WEBRESOURCE4ed9ab3397631744875e7ae63b37ab83image.png)