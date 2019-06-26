---
layout: post
title: Android 动态分析之 Android Studio 调试 Smali
comments: false
description: ""
keywords: "Reverse"
---

apk样本：[https://github.com/Go0s/APKSample/blob/master/9999-Tutorial/AS 调试 Smali](https://github.com/Go0s/APKSample/blob/master/9999-Tutorial/AS%20调试%20Smali)

工具：[https://github.com/Go0s/BombAPK](https://github.com/Go0s/BombAPK)

## ideasmali

Android Studio如果要调试Smali代码，需要安装第三方插件：[ideasmali](https://github.com/JesusFreke/smali/wiki/smalidea)

AS中【Android Studio-->Preferences-->Plugins-->Install plugin from desk...】，安装插件；

![3182872696](/assets/images/2018-03-25/3182872696.png)

## apk反编译

使用apktool将apk反编译，拿到smali代码；

```
[Go0s]: ~/Desktop 
➜  apk 1 jwx02.apk 
[+] BombBombAPK ready to start.
[+] Current command's input: jwx02.apk
[+] Current command: java -jar /Users/go0s/Security/_Tools/Android/BombAPK/bin/apktool.jar d -f jwx02.apk -o jwx02-debug
I: Using Apktool 2.3.1 on jwx02.apk
I: Loading resource table...
I: Decoding AndroidManifest.xml with resources...
I: Loading resource table from file: /Users/go0s/Library/apktool/framework/1.apk
I: Regular manifest package...
I: Decoding file-resources...
I: Decoding values */* XMLs...
I: Baksmaling classes.dex...
I: Copying assets and libs...
I: Copying unknown files...
I: Copying original files...
[+] This command completed execution.
```

## DDMS

以调试状态启动app，应用进入Waiting For Debugger状态：

```
[Go0s]: ~/Desktop 
➜  apk 10 jwx02.apk 
[+] BombBombAPK ready to start.
[+] Current command's input: jwx02.apk
[+] Current command: bash /Users/go0s/Security/_Tools/Android/BombAPK/bin/AmStart jwx02.apk
adb shell am start -D -n hfdcxy.com.myapplication/hfdcxy.com.myapplication.MainActivity
[+] This command completed execution.
[Go0s]: ~/Desktop 
➜  adb shell am start -D -n hfdcxy.com.myapplication/hfdcxy.com.myapplication.MainActivity
Starting: Intent { cmp=hfdcxy.com.myapplication/.MainActivity }
```

打开DDMS，查看通信端口；

![1976022301](/assets/images/2018-03-25/1976022301.png)

左边红圈标注的是应用进程PID，右边红圈标注的是终端和PC通信的端口，选中时DDMS默认多赋予一个8700端口；

```
[Go0s]: ~/Desktop 
➜  adb shell ps | grep hfdcxy.com.myapplication
(standard input):188:u0_a130   12489 762   1506812 42540 ffffffff 00000000 S hfdcxy.com.myapplication
```

BTW：打开DDMS为了省去端口转发的过程，即：adb forward tcp:8700 jdwp:12489

## AS导包

从反编译的文件夹中拿到smali文件夹，以此为源文件，进行Android Studio导包；

此时，网上多为import project，这里因为不需要特殊gradle等环境，只需要打开项目【Open an existing Android Studio Project】，打开后将Android目录窗口换成Project项目窗口；

![3834289278](/assets/images/2018-03-25/3834289278.png)

## AS调试配置

新建调试配置，【Run-->Edit Configurations--> + -->Remote】

name随意，端口8700或DDMS上进程端口，图方便；

![1164071432](/assets/images/2018-03-25/1164071432.png)

## 动态调试

在Smali怀疑处下断点，即语句左边栏点红即可；

![3249554277](/assets/images/2018-03-25/3249554277.png)

调试【Run-->Debug-->smali debug】，应用上Waiting For Debugger框消失，DDMS上进程左边红色虫子变绿；

输入用户名、密码，点击登录键，程序停到断点处；

右键watch添加寄存器，F7单步步入，F8单步步过，进行动态调试；

![3511390517](/assets/images/2018-03-25/3511390517.png)

## BTW

1、为什么省去端口转发？

参考：[http://www.cnblogs.com/gordon0918/p/5570811.html](http://www.cnblogs.com/gordon0918/p/5570811.html)

![1945005667](/assets/images/2018-03-25/1945005667.png)

2、大众流程？

导包，Import Project(Eclipse ADT, Gradle)-->Create project from existing sources-->Next

右键目录，Mark Directory As-->Sources Root

配置JDK，File-->Project Structure