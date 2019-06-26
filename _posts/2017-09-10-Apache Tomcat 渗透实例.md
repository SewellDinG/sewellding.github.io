---
layout: post
title: Apache Tomcat 渗透实例
comments: false
description: ""
keywords: "PenetrationTest"
---

遇到JAVA Web，不要忘了中间件；

她可能会爱上你的；

## 信息搜集

网站搭建为 CMS4J 2010 的CMS，很少见；

基于JAVA的内容管理系统，服务器为Tomcat，可简单在url后加上manager判断；

![1.png](/assets/images/2017-09-10/985033934.png)

## 省时间？

都知道，渗透一个网站或者说渗透一个内网，前期最主要的就是信息搜集，但这里为什么看到tomcat就没又继续从CMS下手了？

一个字，懒...

但这种习惯是不好的，这里只是想快点拿下shell；

还想说点题外话，有的小伙伴刚入门，思想仅仅局限于去找CMS的公开漏洞，碰到RCE还好，如果是注入一类的，只是把思路局限于找后台，进后台，日后台...

这样会大大浪费时间，反而可能会得不到好的效果；

碰到这类Tomcat站或者说这类JAVA Web，应该首先想到如果存在**任意文件读取**的漏洞那就不很方便了；

为了让大家方便看到效果，把站丢到AWVS11去扫了扫，果然存在；

![2.png](/assets/images/2017-09-10/179095176.png)

存在任意文件读取，碰到这里应该很容易想到去读取tomcat的配置文件，里面躺着manager的管理账号和密码；

默认位置在：/usr/local/tomcat/conf/tomcat-users.xml

/usr/local/tomcat 为安装目录；

直接访问：http://www.xxxxxxx.com/DownloadFile?file=../../../../../../../../../../usr/local/tomcat/conf/tomcat-users.xml

![3.png](/assets/images/2017-09-10/114591550.png)

可以拿到管理账户密码：

```
<user username="tomcat" password="xxxxxx@xxxx.xxxx" roles="tomcat,manager-gui"/>
```

访问登陆成功；

## Tomcat管理Getshell

轻车熟路，上传war包；

随便zip压缩一个jsp的shell，更改后缀为.war；

然后部署到服务器即可；

![4.png](/assets/images/2017-09-10/2581385396.png)

部署成功，假如war包名为shell.war，shell为shell.jsp，访问根目录/shell/shell.jsp即可；

![5.png](/assets/images/2017-09-10/3288782935.png)

很顺利拿到shell；

## 除此之外？

通过AWVS11又发现了意外收获，这个Apache Tomcat存在axis2；

axis2也可以理解为一个部署app的后台，只不过上传的不是war包，而是aar包，这并不是简单zip就能搞定的了；

后台路径：/axis2/axis2-admin/login

默认管理密码：admin axis2 ；并没有修改....

![6.png](/assets/images/2017-09-10/2624932035.png)

部署aar包；

![7.png](/assets/images/2017-09-10/1262705154.png)

至于怎么生成aar，和怎么利用网上很多，如这篇最新的，作者前几个月已经更新cat.aar：[http://javaweb.org/?p=1548](http://javaweb.org/?p=1548)

![8.png](/assets/images/2017-09-10/3938949182.png)

## 后

碰到一个站，并不一定非得从应用上下手，从中间件下手可能会有出乎意料的结果，省事省力省时间，和乐而不为？
