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