+++
title = '0X004 Bastion'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCE20936b99374514d5e6ce2add517036fbimage.png)

![](/htb_img/WEBRESOURCE78871723166dee836b3bcf80aa89cb7bimage.png)

![](/htb_img/WEBRESOURCE0b9f7b952b6c488dadfaf68b6a3b4497image.png)

![](/htb_img/WEBRESOURCEbd2917511f778e19cf733f1cb967c44cimage.png)

# 获取立足点

smb：

![](/htb_img/WEBRESOURCE901e48ea26c5f42847cfbec1c0ac19d4image.png)

挂载到本地：

![](/htb_img/WEBRESOURCE1084e891b2be7b0ed64a326eaa4ef7d4image.png)

![](/htb_img/WEBRESOURCE15cd0f823721b338a90296ab512810d7image.png)

打开后，慢慢翻查，没有发现什么有效的东西，但是后面看了一下，最重要的是哪个backup文件夹，里面有两个和其他不一样的文件：

![](/htb_img/WEBRESOURCEf62a948c97343673c5df70a32bae460bimage.png)

借助GPT，我们可以知道是什么东西，其实是个硬盘，是硬盘就可以看：

![](/htb_img/WEBRESOURCEddc0d0b86fbc12fe370d2363e0107786image.png)

使用guestmount进行挂载：

```
安装：
sudo apt-get install libguestfs-tools


使用
sudo guestfish --rw -a 9b9cfbc3-369e-11e9-a17c-806e6f6e6963.vhd
run
list-filesystems  ****查看要挂载的文件系统
exit
挂载：
sudo guestmount -a 9b9cfbc3-369e-11e9-a17c-806e6f6e6963.vhd -m /dev/sda1 /mnt
```

![](/htb_img/WEBRESOURCEbf560e890ecfa9ba6ade9959b8c2c2a5image.png)

![](/htb_img/WEBRESOURCE36aa2d778925d3064fa40a5274ca18a7image.png)

sam文件地址：

```
C:\WIndows\System32\config\SAM
```

SYSTEM hives地址：

```
 C:\Windows\System32\config\SYSTEM
```

把这两个文件拷贝出来：

![](/htb_img/WEBRESOURCEb7cb4501d2f43017c75bef6bf5a87669image.png)

samdump2 读取hash:

![](/htb_img/WEBRESOURCEa56df4487c87e17799af463c1dc86506image.png)

john破解ntml hash:

bureaulampje     (L4mpje)

![](/htb_img/WEBRESOURCE391353a350aa7df977722483227b0e26image.png)

ssh登录：

![](/htb_img/WEBRESOURCEb22b71fa3836d077e80b209f723d6598image.png)

# 提权

![](/htb_img/WEBRESOURCE5bac4735e412f477b26f1397a1e79b79image.png)

systeminfo看不到东西，查看软件目录：

![](/htb_img/WEBRESOURCE87962e8dd5a898433f6f8bfe3e6b9dd1image.png)

搜索信息：

![](/htb_img/WEBRESOURCE91150adf79cbb05f5d7697f0c6d01cdbimage.png)

大概看一下信息：

可以知道我们可以获取到密码：

![](/htb_img/WEBRESOURCE5a2784617d52e93a7065d3c33e7e1b10image.png)

访问里面写的脚本网站：

![](/htb_img/WEBRESOURCEa55b9b8a472e9c0f7c1348ad809d4c39image.png)

利用之前的smb传输进去，http和scp传输失败了：

![](/htb_img/WEBRESOURCEf663c62aa5dbf9b72fd6497d027574b7image.png)

在使用脚本之前，需要先把服务开启来，否则识别不了：

运行一下：

![](/htb_img/WEBRESOURCE49a8c257334167d5ace1403a7b163eefimage.png)

失败了，不过无所谓，这个工具只是一个提取工具，手动看就行：

![](/htb_img/WEBRESOURCEaeeef4579a17faa71d926e6ebf8cf381image.png)

打开%appdata%/mRemoteNg/路径：

![](/htb_img/WEBRESOURCEbac4adc7fa7d2d45c035da99b02058c6image.png)

可以发现加密密码：

![](/htb_img/WEBRESOURCEb33c5a07b35d9e7c647744e6ea8efd1dimage.png)

使用刚刚哪个工具破解：

得到密码：thXLHM96BeKL0ER2

![](/htb_img/WEBRESOURCE4dd9542098725fc2c2a0fbf80007d0c4image.png)

ssh登录成功：

![](/htb_img/WEBRESOURCEdb90d7ae85df3f9e3a988880d0861b6bimage.png)

![](/htb_img/WEBRESOURCEcd15df3e326b28100d0d1ec39f9a7edbimage.png)