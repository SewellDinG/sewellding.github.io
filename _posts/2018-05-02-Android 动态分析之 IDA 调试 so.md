---
layout: post
title: Android 动态分析之 IDA 调试 so
comments: false
description: ""
keywords: "Reverse"
---

## 方法一

适用于无需调试.init_array和JNI_OnLoad块情况，省去am和jdb；

1、安装apk并打开，手机开启IDA调试服务：【adb shell】【su】【./data/local/tmp/android_server】

2、PC端进行端口转发：【adb forward tcp:23946 tcp:23946】

3、IDA打开so文件，定位到待分析函数并下断点：【New】【Export或Function window搜索函数双击定位】

4、附加程序，进入动态调试界面：【Debugger—>Process options】【Hostname:127.0.0.1 Port:23946】然后【Debugger—>Attach to Process】【OK】

5、点击绿色三角【F9】执行程序，弹窗选Same，开始调试，【F7】单步步入，【F8】单步步过

注意：

1、步骤1：如果使用【adb shell /data/local/tmp/android_server】，若没有权限则无法读出可调试进程。

2、步骤2：步骤1开启android_server后为手机终端开启了23946端口，转发是为了使手机应用与IDA进行通信。

3、步骤3：此时下断点会一直保留到动态调试中，且定位到相应的内存位置，省去寻找偏移来计算函数地址。

4、步骤4：如果弹出的附加进程窗口没有待调试应用进程，则取消窗口后在手机上打开应用，从新进入附加进程窗口进行搜索；

5、步骤5：IDA进入动态调试界面后F9执行程序会弹窗验证这个so和应用中的so是否一致【Same】。

## 方法二

最常用 :-) 

1、安装apk并打开，手机开启IDA调试服务：【adb shell】【su】【./data/local/tmp/android_server】

2、PC端进行端口转发：【adb forward tcp:23946 tcp:23946】

3、打开DDMS，以调试方式来启动应用【adb shell am start -D -n com.yaotong.crackme/com.yaotong.crackme.MainActivity】

4、IDA打开so文件，下断点，然后进行attach附加

5、使用jdb恢复进程运行，终端出现停顿：【jdb -connect com.sun.jdi.SocketAttach:hostname=│
127.0.0.1,port=8700】

6、jdb恢复，点击绿色三角【F9】执行程序，弹窗选Same，开始调试，【F7】单步步入，【F8】单步步过

注意：

1、步骤3：会打开应用并弹框：Waiting For Debugger；DDMS中会出现红色虫子。

2、步骤5：端口可在DDMS中看到，若是选中状态，则可使用8700端口通信【第三步默认使DDMS选中当前调试应用】；DDMS红色虫子变成绿色；应用上弹框消失；若不使用jdb，则无法使程序在waiting状态下跑起来，则无法将so加载到内存中；若不使用DDMS，为了jdb能恢复进程则需要端口转发【adb forward tcp:8700 jdwp:11111】，11111为应用PID，可在IDA的attach界面找到进程ID、也可进入shell使用【ps|grep xxx】找到PID；如果jdb失败，即提示“无法附加到目标VM”，则考虑犯了最低级错误，应用apk没有开启可调式，在AndroidManifest.xml写入后重新打包或者直接使用BDOpener xposed框架一劳永逸。

3、在IDA的Modules界面中搜索so文件并点入，会快速找到待分析函数，双击函数跳转定位；也可以新建IDA打开so文件，确定函数偏移地址；在正调试的IDA Debug View中搜索so的基地址【Ctrl+s】；计算函数所在内存地址，并定位【G】。

4、最好在开始就将DDMS打开，以便读取进程PID，因为后打开会出现捕捉不到早已打开的可调试程序进程的情况。

## BTW

1、IDA中，【空格】进行流程图界面和汇编界面转换，【Shift+TAB】进行汇编和伪c转换。

2、ida现在不需要手动导入Jni.h，直接在变量上按【y】，改为【JNIEnv*】后回车即可。

3、查看断点并双击定位：【Debugger-Breakpoints-Breakpoint list】。