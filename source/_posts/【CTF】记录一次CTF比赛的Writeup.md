---
title: 【CTF】记录一次CTF比赛的Writeup
date: 2019-09-25 11:44:20
id: 190925-114420
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf22.png
tags:
- CTF
- Writeup
- 比赛
categories:
- CTF
---
# 0x00 前言
最近因为省赛快来了，因此为实验室的小伙伴准备了这次比赛，总共10道题目，考虑到大多数小伙伴都刚从大一升到大二，因此整体难度不高，当然有几道难度还是有的。

题目大多数都是从网上东找西找的，毕竟我也是个菜鸟呀，还要给他们出题，我太难了。

废话不多说，直接上Writeup吧，以下题目的文件下载地址可以在我的公众号（TeamsSix）回复CTF获取。
<!--more-->

# 0x01 隐写 1
```
flag：steganoI

flag格式：passwd:

题目来源：http://www.wechall.net/challenge/training/stegano1/index.php
```
签到题，下载题目图片，利用记事本打开即可看到flag
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf1.png)

# 0x02 隐写 2
```
flag：teamssix

Hint:一般在公共场合才能看的见

题型参考：http://www.wechall.net/challenge/connect_the_dots/index.php
```
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf2.png)
打开图片，参考题目提示说一般在公共场合才能看见，因此通过盲文对照表可以得出flag是teamssix，图片中的AXHU只是用来干扰的，这道题也是我参考wechall里面的一道题型。
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf3.png)

# 0x03 Web 1
```
flag:iamflagsafsfskdf11223

Hint:站内有提示

题目地址：
http://lab1.xseclab.com/sqli2_3265b4852c13383560327d1c31550b60/index.php
参考来源：http://hackinglab.cn/ShowQues.php?type=sqlinject
```
1、打开题目地址
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf4.png)
2、查看源码找到提示
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf5.png)
3、根据提示使用admin登陆，并使用弱密码
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf6.png)
4、尝试多次都提示失败，利用万能密码再做尝试，找到flag
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf7.png)
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf8.png)

# 0x04 Web 2
```
flag:76tyuh12OKKytig#$%^&

题目地址：http://lab1.xseclab.com/upload3_67275a14c1f2dbe0addedfd75e2da8c1/

flag格式：key is :

题目来源：http://hackinglab.cn/ShowQues.php?type=upload
```
1、打开题目地址，发现是一个文件上传界面
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf9.png)
2、先把Burp挂上，随便上传一个JPG图片试试，并来到Burp重发这个包
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf10.png)
3、在Burp中对文件名进行修改，比如在jpg后加上.png或者其他东西，成功看到flag
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf11.png)

# 0x05 soeasy
```
flag:HackingLabHdd1b7c2fb3ff3288bff

Hint:在这个文件中找到key就可以通关

flag格式:key:

题目来源：http://hackinglab.cn/ShowQues.php?type=pentest
```
**解法一：**
1、下载文件后，发现是vmdk文件，利用DeskGenius打开后，发现Key，此为正确答案
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf12.png)
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf13.png)

**解法二：**
1、利用Vmware映射虚拟硬盘同样可以打开
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf14.png)
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf15.png)

# 0x06 Crack
```
flag:19940808

Hint:flag就是密码

题目：邻居悄悄把密码改了，你只知道邻居1994年出生的，能找到她的密码吗？

题目来源：http://hackinglab.cn/ShowQues.php?type=decrypt
```
1、下载题目文件，根据题意，需要对WiFi密码破解，而且密码很有可能是邻居的生日，因此我们利用工具生成字典。
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf16.png)
2、接下来利用ewsa进行破解，可以看到破解后的密码
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf17.png)
这道题目当时实验室有人用kali做的，kali下的工具感觉破解速度更快。

# 0x07 BiliBili
```
flag:Congratulations_you_got_it

题目：bilibili

flag格式：ctf{}
```
**解法一：**
1、使用Wireshark打开数据包，直接搜索ctf
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf18.png)
2、找到标识的那一行右击进行追踪对应的协议，比如这条是http协议就追踪http协议，之后再次查找ctf
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf19.png)
3、发现ctf括号后的内容为base64加密，解密即可得到flag
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf20.png)
**解法二：**
1、和解法一一样，对数据包进行追踪http流，不难看出这是访问space.bilibili.com/148389186的一个数据包
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf21.png)
2、打开这个网址，同样可以看到被base64加密的flag
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf22.png)
另外打个小广告，上面这个是我的bilibili号（TeamsSix），欢迎大家关注，嘿嘿

# 0x08 Check
```
flag:sAdf_fDfkl_Fdf

题目：简单的逆向

flag格式：flag{}

题目来源：https://www.cnblogs.com/QKSword/p/9095242.html
```
1、下载文件，发现是exe文件，放到PEiD里看看有没有壳以及是什么语言编写的，如果有壳需要先脱壳。
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf23.png)
2、可以看到使用的C语言写的，同时是32位，因此使用IDA32位打开，之后找到main函数
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf24.png)
3、按F5查看伪代码，并点击sub_401050子函数
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf25.png)
4、不难看出下列是一个10进制到ASCII码的转换
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf26.png)
5、利用在线网站转换即可获得flag，网站地址：[http://ctf.ssleye.com/jinzhi.html](http://ctf.ssleye.com/jinzhi.html)
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf27.png)

# 0x09 Android RE
```
flag:DDCTF-397a90a3267641658bbc975326700f4b@didichuxing.com

题目：安卓逆向

flag格式：DDCTF-

Hint:flag中包含chuxing

题目来源：https://xz.aliyun.com/t/1103
```
1、这道题是滴滴出行的一道CTF，下载题目可以看到一个apk文件，先在模拟器中运行看看是个什么东西
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf28.png)
2、功能很简单，一个输入框，输错会提示Wrong，那么利用Android killer给它反编译一下，查找字符“Wrong”
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf29.png)
3、可以看到Wrong字符的路径，接下来进行反编译，不过可能由于本身软件的文件，反编译提示未找到对应的APK源码，没关系，换ApkIDE对其进行编译
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf30.png)
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf31.png)
4、等待一段时间后，可以看到对应源码，简单分析就可以知道该代码从hello-libs.so文件加载，并且对mFlagEntryView.getText().toString()函数的内容即我们输入的内容和stringFromJNI()函数的内容做判断，如果一致就Correct，即正确，不一致就返回Wrong，即错误，那么接下来只需要分析stringFromJNI()的内容就行了，因此我们需要知道系统从hello-libs.so文件加载了什么
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf32.png)
5、将APK解压，找到hello-libs.so文件，由于现在手机都是用arm64位的CPU（我也不知道是不是的啊，听别人说的），因此我们找到arm64-v8a文件夹下的libhello-libs.so文件，用IDA打开
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf33.png)
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf34.png)
6、打开IDA后，根据题目提示，Alt +T　查找chuxing
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf35.png)
７、成功找到flag（DDCTF-397a90a3267641658bbc975326700f4b@didichuxing.com
）输入到模拟器中看到提示Correct，说明flag正确。
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf36.png)
# 0x10 Easy_dump
```
flag：F0rens1cs_St2rt

题目：Easy_dump

flag格式：LCTF{}

Hint：volatilty了解一下

题目来源：https://www.tr0y.wang/2016/12/16/MiniLCTF/index.html
```
**解法一：**
1、下载题目文件，提示利用volatilty工具，同时结合文件后缀为vmem（VMWare的虚拟内存文件），因此判断是一个内存取证的题目，关于volatilty的使用可以参考官方手册：[https://github.com/volatilityfoundation/volatility/wiki/Command-Reference](https://github.com/volatilityfoundation/volatility/wiki/Command-Reference)，废话不多说，先看看镜像信息
```bash
# volatility -f xp.vmem imageinfo
```
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf37.png)
2、可以看到该镜像信息的为WinXPSP2x86，接下来直接扫描查看一些系统文件中有没有flag文件
```bash
# volatility -f xp.vmem --profile=WinXPSP2x86 filescan | grep flag
```
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf38.png)
3、将该flag.txt文件dump下来
```bash
# volatility -f xp.vmem --profile=WinXPSP2x86 dumpfiles -Q 0x0000000005ab74c8 -D ./ -u
```
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf39.png)
4、直接cat flag文件即可看到flag
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf40.png)
**解法二：**
因为该题作者将flag复制到了自己电脑的粘贴板里的，所以直接获取粘贴板的内容也是可以看到flag的，不过谁能想到这种操作 [笑哭]
```bash
# volatility -f xp.vmem clipboard
```
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ctf41.png)

以上就是本次我为他们准备的CTF的全部内容，大多数都是很基础的题目，平时拿来练练手还是不错的，拓宽一下自己的了解面，发现一些自己以前不知道的东西，如果你也想拿上面的题目来玩玩，在公众号（TeamsSix）回复CTF就可以获取下载地址哦。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)