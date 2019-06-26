---
layout: post
title: 使用 nativefier 快速将网站转换成桌面应用
description: ""
keywords: "Sword"
---

当遇到一个好的工具网站，如[https://gchq.github.io/CyberChef](https://gchq.github.io/CyberChef)，总想着能不能转换成桌面应用，配合Mac下Spotlight或Launchpad快速打开。好吧，我大概是闲的...

公司针对同一种服务，面向不同平台都有一套不同的代码，为了简化程序员工作了，同时提高效率，出现了API的概念，不同平台代码调用相应的API来完成任务；还有的是使用跨平台编程语言编写一套代码，在不同平台上使用，如Electron等。

但如果已经做出了Web平台的服务，如何将其快速转换成桌面应用呢？

答案是使用 **[nativefier](https://github.com/jiahaog/nativefier)**。

## Introduction

Nativefier is a command-line tool to easily create a desktop application for any web site with succinct and minimal configuration. Apps are wrapped by [Electron](http://electron.atom.io/) in an OS executable (`.app`, `.exe`, etc.) for use on Windows, macOS and Linux.

## Installation

- macOS 10.9+ / Windows / Linux
- [Node.js](https://nodejs.org/) `>=6` (4.x may work but is no longer tested, please upgrade)
- See [optional dependencies](https://github.com/jiahaog/nativefier#optional-dependencies) for more.

```
npm install nativefier -g
```

## Usage

查看命令参数介绍：

```
nativefier --help
```

实际使用：

```
nativefier -n "CyberChef" -p osx "https://gchq.github.io/CyberChef"
```

## Run

命令执行结束后会在当前目录生成以app为后缀的应用，很简单很便捷，傻瓜式操作。

![QQ20190202-160926@2x.png](/assets/images/2019-02-23/QQ20190202-160926@2x.png)

## 制作Mac安装包

目的是将上面生成的.app应用放到.dmg中，方便传送。

1、打开Mac自带的磁盘工具，新建空白磁盘；

![QQ20190202-154738@2x.png](/assets/images/2019-02-23/QQ20190202-154738@2x.png)

2、设置dmg名称、磁盘位置、磁盘大小200M即可，最后是可以压缩的但大小必须比生成的软件要大（后来发现使用此工具生成的应用50多M）；

![QQ20190202-155009@2x.png](/assets/images/2019-02-23/QQ20190202-155009@2x.png)

3、新建磁盘后，双击打开磁盘，将app文件复制拷贝进去，新建一个指向应用程序文件夹的快捷方式，同样复制拷贝进去（使用ln -s创建指向应用程序目录的软链接），右键磁盘点击查看显示选项，进行图标大小、网格间距、背景等设置（一般背景选择带箭头的图片进行提示）；

![QQ20190202-160422@2x.png](/assets/images/2019-02-23/QQ20190202-160422@2x.png)

![QQ20190202-160348@2x.png](/assets/images/2019-02-23/QQ20190202-160348@2x.png)

4、关闭磁盘后弹出，再次打开磁盘工具，点击菜单映像、转换，将其大小压缩一下，并存储为dmg文件。

![QQ20190202-160536@2x.png](/assets/images/2019-02-23/QQ20190202-160536@2x.png)

![QQ20190202-160634@2x.png](/assets/images/2019-02-23/QQ20190202-160634@2x.png)