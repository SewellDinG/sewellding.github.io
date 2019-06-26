---
layout: post
title: Nexus5 刷机、Root 并安装 Xposed 框架
comments: false
description: ""
keywords: "Sword"
---

咸鱼300块淘了一台羊毛党的 Nexus5，成色很新，但系统默认 Android 4.4.2，这里记录下升级的过程；

## 目标

Nexus5 代号 hammerhead，即以下需要搜索对应hammerhead的服务；

官方镜像版本：[Android 5.0.1](https://developers.google.com/android/images)

Auto Root：[CF-Auto-Root-hammerhead-hammerhead-nexus5](https://autoroot.chainfire.eu)

Root权限管理：[SuperSU V2.82](http://www.supersu.com/download)

第三方Recovery：[twrp-3.2.1-1-hammerhead](https://twrp.me/Devices)

Xposed框架：[Xposed-v89-sdk21-arm](http://dl-xda.xposed.info/framework)

Xposed应用：[XposedInstaller_3.1.5.apk](https://forum.xda-developers.com/attachment.php?attachmentid=4393082&d=1516301692)

## 解锁

机子之前已经解锁；

如果没有解锁，可以直接使用`OEM`解锁：

```
[Go0s]: ~ 
➜  adb devices
List of devices attached
0831ee9d2131fe19	device
[Go0s]: ~ 
➜  adb reboot bootloader
[Go0s]: ~ 
➜  fastboot oem unlock
...
FAILED (remote: Already Unlocked)
finished. total time: 0.098s
```

## 刷机

搜索关键词，找到目标机器的镜像包：

![2680796669.png](/assets/images/2018-01-30/2680796669.png)

下载 Android 5.0.1 镜像包 hammerhead-lrx22c-factory-3b22f481.zip 并解压进入此目录；

让手机进入bootloader【可adb命令进入，也可以关机使用音量下键+电源键进入】，然后直接运行`./flash-all.sh`；

```
[Go0s]: ~/Desktop
➜  adb reboot bootloader
[Go0s]: ~/Desktop 
➜  fastboot devices               
0831ee9d2131fe19	fastboot
[Go0s]: ~/Desktop 
➜  cd hammerhead-lrx22c 
[Go0s]: ~/Desktop/hammerhead-lrx22c 
➜  ls
bootloader-hammerhead-hhz12d.img        flash-base.sh
flash-all.bat                           image-hammerhead-lrx22c.zip
flash-all.sh                            radio-hammerhead-m8974a-2.0.50.2.22.img
[Go0s]: ~/Desktop/hammerhead-lrx22c 
➜  ./flash-all.sh        
target didn't report max-download-size
sending 'bootloader' (2579 KB)...
OKAY [  0.300s]
writing 'bootloader'...
OKAY [  0.485s]
finished. total time: 0.785s
rebooting into bootloader...
OKAY [  0.100s]
finished. total time: 0.100s
target reported max download size of 1073741824 bytes
...
sending 'cache' (428 KB)...
OKAY [  0.230s]
writing 'cache'...
OKAY [  0.219s]
rebooting...

finished. total time: 111.580s
```

等待一会，手机即可刷机完成并重启进入初始化设置；

## Root

搜索关键词：

![3490284871.png](/assets/images/2018-01-30/3490284871.png)

下载Auto Root包 CF-Auto-Root-hammerhead-hammerhead-nexus5.zip 并解压进入目录；

让手机进入bootloader，赋予root-mac.sh执行权限，然后直接运行脚本`./root-mac.sh`；

```
[Go0s]: ~/Desktop/CF-Auto-Root-hammerhead-hammerhead-nexus5   
➜  adb reboot bootloader
[Go0s]: ~/Desktop/CF-Auto-Root-hammerhead-hammerhead-nexus5  
➜  fastboot devices               
0831ee9d2131fe19	fastboot
[Go0s]: ~/Desktop/CF-Auto-Root-hammerhead-hammerhead-nexus5 
➜  chmod +x root-mac.sh 
[Go0s]: ~/Desktop/CF-Auto-Root-hammerhead-hammerhead-nexus5 
➜  ./root-mac.sh 

----- CF-Auto-Root-hammerhead-hammerhead-nexus5 -----
...
Press Ctrl+C to cancel !
Press ENTER to continue
...
```

这里一直失败，开启虚拟机使用windows的脚本来root；

双击root-windows.bat，一路回车，无需等待，手机界面出现红色安卓logo，并自动重启；

## SuperSU

直接使用`adb`安装，但打开需要更新其二进制文件，这里最好使用TWRP刷入SuperSU的源文件压缩包来安装；

```
adb install SuperSU_V2.82.apk
```

## TWRP

搜索关键词：

![995251908.png](/assets/images/2018-01-30/995251908.png)

下载twrp-3.2.1-1-hammerhead.img，让手机进入bootloader，使用`fastboot`刷入这个第三方Recovery，并重启；

```
[Go0s]: ~/Desktop
➜  adb reboot bootloader     
[Go0s]: ~/Desktop
➜  fastboot devices
0831ee9d2131fe19	fastboot
[Go0s]: ~/Desktop 
➜  fastboot flash recovery twrp-3.2.1-1-hammerhead.img 
target reported max download size of 1073741824 bytes
sending 'recovery' (13430 KB)...
OKAY [  0.640s]
writing 'recovery'...
OKAY [  1.060s]
finished. total time: 1.700s
[Go0s]: ~/Desktop 
➜  fastboot reboot
rebooting...
finished. total time: 0.100s
```

## Xposed 框架

按需寻找，Android 5.0.1 使用SDK21，32位机器，下载最新的v89版本的Xposed框架；

![2228442564.png](/assets/images/2018-01-30/2228442564.png)

将框架压缩包push到`/sdcard`中；

```
[Go0s]: ~/Desktop 
➜  adb push xposed-v89-sdk21-arm.zip /sdcard
xposed-v89-sdk21-arm.zip: 1 file pushed. 6.1 MB/s (3480379 bytes in 0.548s)
```

进入recovery，可以设置语言，滑动“Swipe to Allow Modifications”，在TWRP中点击`install`，选择xposed-v89-sdk21-arm.zip，勾选“Reboot after installation is complete”，滑动“Swipe to comfirm Flash”，开始安装，完成后自动重启；

期间会提示是否下载TWRP的APP，按需所取；

```
[Go0s]: ~/Desktop 
➜  adb reboot recovery
```

## Xposed 应用

可以下载酷安，然后安装应用；

这里我在官网下好了最新版，直接使用`adb`；

```
[Go0s]: ~/Desktop 
➜  adb install XposedInstaller_3.1.5.apk 
XposedInstaller_3.1.5.apk: 1 file pushed. 6.3 MB/s (3105672 bytes in 0.471s)
	pkg: /data/local/tmp/XposedInstaller_3.1.5.apk
Success
```

打开软件，出现了问题，提示：xposed已安装，但未激活；

看日志报错：Cannot read log/data/data/de.robv.android.xposed.installer/log/error.log: open failed

解决办法：在设置中勾选“禁用资源钩子”，然后重启机器即可激活；
