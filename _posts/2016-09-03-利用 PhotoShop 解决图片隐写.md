---
layout: post
title: 利用 PhotoShop 解决图片隐写
comments: false
description: "PhotoShop 在CTF中可谓是神器。"
keywords: "Sword"
---

## ISCC 2014-秦晋之好

瞎胡看看到了ISCC CTF 2014 的Writeup，有道隐写题很有意思，查了一下网上的writeup，通通都来自一人的python程序；

那我来用ps来实现一番吧；

**附件：**![ifs.bmp](/assets/images/2016-09-03/1568170932.bmp)

没有描述，只有这一张图片；

丢到ps中并没有发现什么，看他们wp说放大后可以看到有一些字符，我拉大了仍然不清楚，只是存在有几个零零散散的**大块像素点**；

![20160903225035.png](/assets/images/2016-09-03/3985499504.png)

先放上网上统一的Writeup吧：[http://joychou.org/index.php/Misc/iscc-ctf-2014-writeup.html](http://joychou.org/index.php/Misc/iscc-ctf-2014-writeup.html)

```
#coding:utf-8
import Image
img = Image.open('ifs.bmp')
X = img.size[0]
Y = img.size[1]
print X,Y
for i in range(X-2):
	for j in range(Y-2):
		a = img.getpixel((i,j))[0]+img.getpixel((i,j))[1]+img.getpixel((i,j))[2]
		b = img.getpixel((i,j+1))[0]+img.getpixel((i,j+1))[1]+img.getpixel((i,j+1))[2]
		c = img.getpixel((i,j+2))[0]+img.getpixel((i,j+2))[1]+img.getpixel((i,j+2))[2]
		if (a &gt; b and c &gt; b) or (a &lt; b and c &lt; b):
			pass
		else:
			img.putpixel((i,j),(255,255,255))
img.show()
```

关键是两个函数：

**getpixel（）**函数，获取像素位置的值；

**putpixel（）**函数，写某个像素位置的值；

大致意思就是取纵坐标连着三个像素点，中间一点像素值不能同时大于或同时小于旁边两个像素点的值；

类似于浮雕，壁画一类的效果了（减弱三点的差值）；

大致过程如下：

1、打开滤镜-滤镜库，使用 **壁画** 效果：

参数如下（实际参数自己把握）：

![20160903224203.png](/assets/images/2016-09-03/518385414.png)

效果如下：

![20160903224213.png](/assets/images/2016-09-03/4246784394.png)

可见已经可以看到部分字体了；

返回主页面；

2、由于都是黑乎乎的，对比度很低；

使用 **加深工具** 现将看清楚的字符加深，然后 **调节曲线** 将颜色压深；

![20160903230711.png](/assets/images/2016-09-03/1805501449.png)

3、现在看来就是头发有问题，影响了第三第四排的第二个字符；

我们来使用 **减弱工具**，将黑色压低；

![20160903232132.png](/assets/images/2016-09-03/1180018764.png)

4、可以看到隐约的2，但第四排仍然看不到，不搞了，实战的时候肯定能猜出来；

附上对比图：

![20160903232647.png](/assets/images/2016-09-03/4234751386.png)

## AIS3 CTF-Throw the ball to the pokemon!

**附件**：[misc1.txt](/assets/images/2016-09-03/2684895039.txt)

**Tips**：Throw the ball to the pokemon!

神奇宝贝哎

打开txt文件，并没有出现乱码，甚至还有点规则，每行首字母都是M；

![20160828124228.png](/assets/images/2016-09-03/1419651360.png)

他们都猜测是某种加密或者隐写；

利用 **binwalk，file** 来看下：

![20160828124323.png](/assets/images/2016-09-03/2791839493.png)

file显示可能是 **UUencode** 编码，去查了下确实是：[https://en.wikipedia.org/wiki/Uuencoding#Uuencode_table](https://en.wikipedia.org/wiki/Uuencoding#Uuencode_table)

特征：

1、文件首行begin 664 quiz；

2.每行64个字符；

编码解码网站：[http://web.chacuo.net/charsetuuencode](http://web.chacuo.net/charsetuuencode)

但这毕竟这是个杂项而不是Crypto；

我一般在测试之前都会先把后缀改为压缩包来试试看，在kali下改为tar.gz，但失败了，也总是怀疑是密码学的题目；

拿到win下，改为rar，里面有个文件：quiz

没多想继续解压缩，还是quiz文件；

继续，出来了两张图片；

继续分析，用 **binwalk** 看了下：

![20160828125145.png](/assets/images/2016-09-03/345322759.png)

结果可以看到，都是压缩包；

两张图片，一个精灵球，一个杰尼龟（我还认识...）

想起Tips：Throw the ball to the pokemon!

应该就是两种图片看 **容差** 了；

我看有用beyond compare的，还有用Stegsolve.jar的；

前者确实可以，后者可以叠加，但都是规定的容差，对此题没有用；

我这里还是用PS，尽量详细；

1、打开ps，两张图片 <strong>叠</strong> 在一起；

看到没，边框有字隐约出现了，这里谁前谁后自己判断；

![20160828125243.png](/assets/images/2016-09-03/1219925625.png)

2、在上面的先**大致**调整下**混合模式**，目的是让字体更清晰起来；

我这里测出来是减去模式，这种视情况而定；

![20160828125317.jpg](/assets/images/2016-09-03/1592052871.jpg)

3、把每张图的<strong>阈值</strong>调一下，这里都归为了最小值；

这里为啥用阈值而不用去色，大家实际操作一下就很明白了；

![20160828125713.png](/assets/images/2016-09-03/634154837.png)

4、可以看到，flag已经很清楚了，如果还看不清楚，可以 **Ctrl+Alt+Shift+e** 将图层合并继续调整：

调节曲线；

单通道输出：

加深工具加深；

调节对比度；

.....

方法很多，也是具体情况具体来操作；

ps其实和ctf关联也是很大的，是很容易玩起来的；

## 阿里云CTF-年薪100w,快到碗里来

吐司有个人发了道题，地址：[http://121.42.149.60/rmb100w_per_year.html](http://121.42.149.60/rmb100w_per_year.html)

看最下面著有©1999-2016 阿里云盾校园招聘

说是在微博上发的，阿里云的好几个人都转发了，可能是校招？

不管这么多了，看下题；

![zhaopin.png](/assets/images/2016-09-03/4150297210.png)

打开页面只有一个图片和一行字--“**这个熊猫的颜色为什么这么奇怪？**”

首先想到是隐写，另存为本地发现并不是；

熊猫下面很像残缺的条形码，应该是让我们补全条形码的；

打开ps，我尽量描述详细：

1、选择一个合适的选择框，我这选择矩形；

![20160823212913.png](/assets/images/2016-09-03/2494014897.png)

2、因为条形码是每个竖条宽细都一样，所以选择一小块复制，Ctrl+v粘贴后，**Ctrl+t** 选中，上下拉长即可；

![20160823213013.png](/assets/images/2016-09-03/2244870136.png)

3、两边都拉完全了，这时候 **Ctrl+Alt+Shift+e** 将所有图层全部重叠到一个图层上；

便于接下来继续修补；

![20160823213124.png](/assets/images/2016-09-03/335409924.png)

4、继续选择合适方块，拉伸；

![20160823213336.png](/assets/images/2016-09-03/163435054.png)

5、条形码出来了。

还记得原题有句话“**这个熊猫的颜色为什么这么奇怪？**”；

看这个熊猫看着好别扭，颜色反了；

回到ps，**Ctrl+i** 颜色反相，就是黑变白，白变黑；

![20160823215643.png](/assets/images/2016-09-03/3142222624.png)

熊猫这才看的顺眼起来...

去扫描，浏览器自动跳转到：[http://121.42.149.60/1cc88340](http://121.42.149.60/1cc88340)

进入到下一题。
