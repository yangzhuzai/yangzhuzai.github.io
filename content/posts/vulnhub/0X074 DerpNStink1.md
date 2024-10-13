+++
title = '0x074 DerpNStink1'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



# 端口扫描

![](/vulnhub_img/WEBRESOURCE1ad25f3af513423dda14b03983712467image.png)

![](/vulnhub_img/WEBRESOURCEa3fd8424179526660ffeecdc2b404745image.png)

![](/vulnhub_img/WEBRESOURCEda526eff56628b7909d849afcee815aaimage.png)

# 渗透测试

先看21端口：

![](/vulnhub_img/WEBRESOURCE9f7c8e9ed9b0f451fd64108ef906fcefimage.png)

80端口：

![](/vulnhub_img/WEBRESOURCE442d118e755c3374824b4e24636ab1c5image.png)

首页源码发现

![](/vulnhub_img/WEBRESOURCE0c6e057bc38ecc5c7148cafcf17aaa2cimage.png)

二级目录：

![](/vulnhub_img/WEBRESOURCE788c77bc986a3f50c2dfe00ff2acb9acimage.png)

gobuster：

![](/vulnhub_img/WEBRESOURCE164263042b6af447eda01fd9d8df3c0bimage.png)

修改hosts访问：

![](/vulnhub_img/WEBRESOURCEd7e5e68bf58a987ba8b6e0cbe29cc6feimage.png)

wpscan:

插件：

![](/vulnhub_img/WEBRESOURCEb3bc6732e8ea9f1607a778cff288dc39image.png)

用户名：

![](/vulnhub_img/WEBRESOURCEc70bff00f3331f6243c4ff5a8872ce36image.png)

当前插件漏洞：

当前版本应该是1.4.6，但是很可惜，是个后台漏洞：

![](/vulnhub_img/WEBRESOURCE0a83ffabd3e4e50f8550ae2791a9552bimage.png)

后台弱口令:admin/admin

![](/vulnhub_img/WEBRESOURCEd66edb68792f30c73f2124a1ff7933ebimage.png)

根据刚刚的插件漏洞，直接上传shell:

![](/vulnhub_img/WEBRESOURCE84c42e249a55cd66fb1ae8bce52f3191image.png)

或者手动传也是可以的：

![](/vulnhub_img/WEBRESOURCE0f3329107e44b7bfae709eac386f7142image.png)

![](/vulnhub_img/WEBRESOURCE1d7c3d27cb67e47cc7f1f5b9fddb66a3image.png)

# 提权

家目录：

![](/vulnhub_img/WEBRESOURCE426dc1478cd8a8b6552796cc1f53059bimage.png)

![](/vulnhub_img/WEBRESOURCE8f28abf1e2d822451bef40fd020afe24image.png)

![](/vulnhub_img/WEBRESOURCEe8c63ef35b17ee4c89abf240eafe7f42image.png)

翻查密码：

![](/vulnhub_img/WEBRESOURCE775b07842d5ef1f99bf24dd0863170baimage.png)

wedgie57

![](/vulnhub_img/WEBRESOURCE7e6c2dc779e3e05bb1a1ca2981538371image.png)

尝试成功：

![](/vulnhub_img/WEBRESOURCE499f3d34f3b2c403c165ba03b64eddb3image.png)

在家目录里面翻查了半天：

![](/vulnhub_img/WEBRESOURCE0aa890d2dd53208bf8abbc4a59e86879image.png)

我以为是另外一个用户的，没想到还是自己的：

![](/vulnhub_img/WEBRESOURCE7fdc73753d66d89beb422f5d3a9de97cimage.png)

算了，有GCC，上脚本：

![](/vulnhub_img/WEBRESOURCE8f4090ef160241e9d5c4f421beba6525image.png)

然后是作者留下的路：

在Documents下面有个pacp包，直接筛选，用wireshark太费时间：

![](/vulnhub_img/WEBRESOURCE9a50756bcd282ce11019ab848c595745image.png)

那就很舒服了：

![](/vulnhub_img/WEBRESOURCEecaea81a0958e2f841fa930aa0cce043image.png)

![](/vulnhub_img/WEBRESOURCEcd8dce696ea27679b3e211255c9d63b9image.png)

![](/vulnhub_img/WEBRESOURCEf9e89594391bb4ef9b3744fae5f16770image.png)