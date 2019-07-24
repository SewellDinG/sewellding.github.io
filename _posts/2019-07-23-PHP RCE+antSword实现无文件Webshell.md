---
layout: post
title: PHP RCE+antSword实现无文件Webshell
description: ""
keywords: ""
---

# 1. Inspiration

日前碰到个存在ThinkPHP RCE的站，但由于目标服务器存在某盾，普通Payload明文流量直接被ban，最终使用antSword的编解码功能得以bypass。

受到此Payload的启发：如果目标网站存在远程代码执行漏洞，那么就可以实现无文件Webshell，同时若“加密”了传输流量，某狗某盾均可被bypass。

本地复现了一下当时的环境：

![thinkphpRCE](/assets/images/2019-07-23/thinkphpRCE.png)

# 2. Achievement

如果没有RCE呢？那就利用其他漏洞来构造RCE。

这里假设目标网站存在修改源码的功能（类似WordPress），我们可以直接在目标程序文件内构造RCE代码，函数及参数均由外部传入，POST、GET、COOKIE方式均可。

这里仍拿ThinkPHP来演示：在index.php文件后加上如下代码。

NOTE that PHP>7.1 assert被定义为一种语言构造器，而不是一个函数，所以像eval一样不支持可变函数（如果一个变量名后有圆括号，PHP 将寻找与变量的值同名的函数，并且尝试执行它），即以下代码调用不可用。

![PHPWebshell1](/assets/images/2019-07-23/PHPWebshell1.png)

实现：

![PHPWebshell2](/assets/images/2019-07-23/PHPWebshell2.png)

再利用antSword的编解码功能“加密”传输流量，这里使用随机截取一位base64编码器。

![PHPWebshell3](/assets/images/2019-07-23/PHPWebshell3.png)

![PHPWebshell4](/assets/images/2019-07-23/PHPWebshell4.png)

完美实现Webshell功能，使用Wireshark抓取流量可以发现传输的关键数据被base64编码。

![PHPWebshell5](/assets/images/2019-07-23/PHPWebshell5.png)

构造PHP RCE的方式灵活多样，可以肆意发挥，但前提需要建立在适用目标PHP version特性上。
