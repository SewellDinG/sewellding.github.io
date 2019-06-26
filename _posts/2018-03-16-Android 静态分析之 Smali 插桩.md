---
layout: post
title: Android 静态分析之 Smali 插桩
comments: false
description: ""
keywords: "Reverse"
---

Smali强大之处莫过于可以随心所欲的进行插桩操作【Smali Instrumentation】，即寻找特殊位置进行针对性代码插入，以此达到目的；

smali武器库：[https://github.com/Go0s/smaliArmory](https://github.com/Go0s/smaliArmory)

apk样本：[https://github.com/Go0s/APKSample/blob/master/9999-Tutorial/Smali%20插桩/crackme1.apk](https://github.com/Go0s/APKSample/blob/master/9999-Tutorial/Smali%20插桩/crackme1.apk)

工具：[https://github.com/Go0s/BombAPK](https://github.com/Go0s/BombAPK)

## Smali插桩自用代码库

Go0sLog.java —> Go0sLog.smali

1. 无参数，判断怀疑函数是否执行，如何触发，执行的顺序...
2. 一个参数，字符串或者数组，输出怀疑函数的返回值...
3. 两个参数，即多一个Tag自定义的key值...

## 插桩代码

```
invoke-static {}, LGo0sLog;->Log()V
invoke-static {v1}, LGo0sLog;->Log(Ljava/lang/Object;)V
invoke-static {v1}, LGo0sLog;->Log([Ljava/lang/Object;)V
```

## 在哪插桩？

1、在invoke-static/invoke-virtual指令返回类型是V之后可以加入；

2、在invoke-static/invoke-virtual指令返回类型不是V，那么在move-result-object命令之后可以加入；

## 查看日志

```
adb logcat -s Go0s
```

## 实例 crackme1.apk

1、功能查看

安装应用，发现界面只有一个输入框和一个按钮，随便输入后点击按钮，提示“No!”；

2、逻辑分析

功能简单，使用jadx反编译后快速查看java逻辑；

![2625857244](/assets/images/2018-03-16/2625857244.png)

代码很简单，通过输入一个密码，以此与正确密码相对比，正确时弹出yes，错误时弹出no；

正确的密码是通过getkey()函数返回，代码很长，感觉逻辑也很复杂，这里一点也不需要读懂；

这时候就可以体现出Smali 插桩的优势了，随地插入smali代码，让应用自己返回正确密码，即getkey函数返回值；

当然这里亦可以直接修改判断，让其输入错误的情况下返回yes，这里不赘述；

3、apktool反编译

```
[Go0s]: ~/Desktop 
➜  apk 1 crackme1.apk 
[+] BombBombAPK ready to start.
[+] Current command's input: crackme1.apk
[+] Current command: java -jar /Users/go0s/Security/_Tools/Android/BombAPK/bin/apktool.jar d -f crackme1.apk -o crackme1-debug
I: Using Apktool 2.3.1 on crackme1.apk
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

4、smali插桩

①、将编译好的Go0sLog.smali放在smali根目录下，好习惯，方便应用中有多个功能或者多个app是统一调用：

```
[Go0s]: ~/Desktop/crackme1-debug 
➜  tree
.
├── AndroidManifest.xml
├── apktool.yml
├── build
├── original
├── res
└── smali
    ├── Go0sLog.smali
    ├── android
    └── com
        └── mzheng
            └── crackme1
                ├── BuildConfig.smali
                ├── MainActivity$1.smali
                ├── MainActivity.smali
                ├── R$attr.smali
                ├── R$dimen.smali
                ├── R$drawable.smali
                ├── R$id.smali
                ├── R$layout.smali
                ├── R$menu.smali
                ├── R$string.smali
                ├── R$style.smali
                └── R.smali
```

②、在反编译后的smali代码中，需要寻找equal判断的位置，在其进行输出getkey()返回值时插桩，由于功能唯一，代码位置很容易找到，在`com/mzheng/crackme1/MainActivity$1`文件中；

BTW：为什么是MainActivity$1.smali而不是MainActivity.smali呢？

因为主要的判断逻辑是在OnClickListener这个类里，而这个类是MainActivity的一个内部类，同时我们在实现的时候也没有给这个类声明具体的名字，所以这个类用$1表示。实际环境会有很多`$+id`的文件，大多如此，多功能情况下还是搜索特殊字符或对怀疑位置进行插桩来判断实际功能位置；

点击事件OnClickListener对应的smali：

```
# virtual methods
.method public onClick(Landroid/view/View;)V
    .locals 4
    .param p1, "arg0"    # Landroid/view/View;
    .prologue
    const/4 v3, 0x1
    .line 27
    iget-object v1, p0, Lcom/mzheng/crackme1/MainActivity$1;->val$editText0:Landroid/widget/EditText;
    invoke-virtual {v1}, Landroid/widget/EditText;->getText()Landroid/text/Editable;
    move-result-object v1
    invoke-interface {v1}, Landroid/text/Editable;->toString()Ljava/lang/String;
    move-result-object v0
    .line 29
    .local v0, "str":Ljava/lang/String;
    const-string v1, "mrkxqcroxqtskx"
    const/16 v2, 0x2a
    invoke-static {v1, v2}, Lcom/mzheng/crackme1/MainActivity;->access$0(Ljava/lang/String;I)Ljava/lang/String;
    move-result-object v1
    invoke-static {}, LGo0sLog;->Log()V
    invoke-static {v1}, LGo0sLog;->Log(Ljava/lang/Object;)V
    invoke-virtual {v0, v1}, Ljava/lang/String;->equals(Ljava/lang/Object;)Z
    move-result v1
    if-eqz v1, :cond_0
    .line 31
    iget-object v1, p0, Lcom/mzheng/crackme1/MainActivity$1;->this$0:Lcom/mzheng/crackme1/MainActivity;
    const-string v2, "Yes!"
    invoke-static {v1, v2, v3}, Landroid/widget/Toast;->makeText(Landroid/content/Context;Ljava/lang/CharSequence;I)Landroid/widget/Toast;
    move-result-object v1
    invoke-virtual {v1}, Landroid/widget/Toast;->show()V
    .line 37
    :goto_0
    return-void
    .line 35
    :cond_0
    iget-object v1, p0, Lcom/mzheng/crackme1/MainActivity$1;->this$0:Lcom/mzheng/crackme1/MainActivity;
    const-string v2, "No!"
    invoke-static {v1, v2, v3}, Landroid/widget/Toast;->makeText(Landroid/content/Context;Ljava/lang/CharSequence;I)Landroid/widget/Toast;
    move-result-object v1
    invoke-virtual {v1}, Landroid/widget/Toast;->show()V
    goto :goto_0
.end method
```

可以看到在equal()判断前有个access函数，由于返回类型不是V，且对于返回值【即getkey的返回值】赋予了v1，`move-result-object v1`，所以要在此后面进行插桩；

```
invoke-static {}, LGo0sLog;->Log()V
invoke-static {v1}, LGo0sLog;->Log(Ljava/lang/Object;)V
```

添加一个位置判断和参数返回的代码；

5、回编译

使用apktool进行回编译：

```
[Go0s]: ~/Desktop 
➜  apk 2 crackme1-debug 
[+] BombBombAPK ready to start.
[+] Current command's input: crackme1-debug
[+] Current command: java -jar /Users/go0s/Security/_Tools/Android/BombAPK/bin/apktool.jar b crackme1-debug -o crackme1-debug.apk
I: Using Apktool 2.3.1
I: Checking whether sources has changed...
I: Smaling smali folder into classes.dex...
I: Checking whether resources has changed...
I: Building resources...
I: Building apk file...
I: Copying unknown files/dir...
[+] This command completed execution.
```

6、签名

对apk进行签名：

```
[Go0s]: ~/Desktop 
➜  apk 3 crackme1-debug.apk 
[+] BombBombAPK ready to start.
[+] Current command's input: crackme1-debug.apk
[+] Current command: java -jar /Users/go0s/Security/_Tools/Android/BombAPK/bin/signapk/signapk.jar /Users/go0s/Security/_Tools/Android/BombAPK/bin/signapk/testkey.x509.pem /Users/go0s/Security/_Tools/Android/BombAPK/bin/signapk/testkey.pk8 crackme1-debug.apk crackme1-debug-S.apk
[+] This command completed execution.
```

7、测试

打开logcat，设置对应字段：

```
[Go0s]: ~/Desktop 
➜  adb install crackme1-debug-S.apk 
crackme1-debug-S.apk: 1 file pushed. 3.6 MB/s (286508 bytes in 0.075s)
	pkg: /data/local/tmp/crackme1-debug-S.apk
Success
[Go0s]: ~/Desktop 
➜  adb logcat -s "Go0s"
--------- beginning of system
--------- beginning of main
D/Go0s    (19555): DeBug ...
D/Go0s    (19555): changshengjian
```

发现输出changshengjian，即为密码；
