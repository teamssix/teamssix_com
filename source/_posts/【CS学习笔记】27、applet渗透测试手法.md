---
title: 【CS学习笔记】27、apple渗透测试手法
date: 2020-04-19 15:07:32
id: 200419-150732
summary: 在 Cobalt Strike 的源码中内置了用于攻击 Java Applet 签名的 Applet 工具。
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/2019-12-04-101402.jpg
tags:
- Cobalt Strike
- 学习笔记
- 红队
categories:
- CS 学习笔记
---

# 0x00 前言

在开始今天的内容之前，先来看看什么情况下会进行云查杀：

1、首先判断文件是否为正常文件

2、如果判断为可疑文件，则把文件的 hash 上传到云上

3、同时把这个文件标记为可疑文件，而不是正常文件

因此可以通过修改我们的脚本来使其跳过云查杀，就像是在白名单里的程序一样。

# 0x01 Java Applet

接下来一起来看看 Cobalt Strike Java Applet 攻击，在 Cobalt Strike 的源码中内置了用于攻击 Java Applet 签名的 Applet 工具。

使用 Applet 工具的步骤如下：

1、到 `Help -> Arsenal`

2、如果需要的话就修改/混淆病毒文件 

3、使用代码签名证书进行签名

4、构建

5、使用 Applet Kit 加载脚本

大概在 2014 年 7 月，开始有人在钓鱼中使用宏攻击，在几年前，这是一种效果还很不错的攻击方式。

# 0x02 应用程序白名单

站在防御者的角度，一个好的防御应该是列出只允许自己运行的应用程序白名单而不允许他人运行。对于攻击者则是使用白名单应用程序将代理放到内存中的方法来进行攻击，Java Applet 攻击就是这样做的。

一种攻击的方法是直接插入内存进行攻击。Java Applet、Office 宏、CS 下的 PowerShell 命令行都是这样做的。

一些白名单免杀的资料：

[https://twitter.com/subTee](https://twitter.com/subTee)

[https://github.com/khr0x40sh/WhiteListEvasion](https://github.com/khr0x40sh/WhiteListEvasion)

> 注：由于以上只是我个人在学这一节时做的笔记，因此看起来可能会比较意识流，实际上视频中老师也是按照这个顺序讲解的。

> 参考链接：[https://www.bilibili.com/video/BV16b411i7n5](https://www.bilibili.com/video/BV16b411i7n5)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)