---
title: 【CS学习笔记】1、如何搭建自己的渗透测试环境
date: 2020-04-19 15:04:07
id: 200419-150407
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/2019-12-04-101402.jpg
summary: 由于这只是学习笔记，因此不会像教程一样详尽，一些我个人已经了解的东西或许不会记在笔记里，因此把笔记当做教程阅读是不合适的。
tags:
- Cobalt Strike
- 学习笔记
- 红队
categories:
- CS 学习笔记
---

# 0x00 前言

第一次接触CS的时候，是有人在群里发了一个CS最新版的安装包，当时第一反应，CS ？？？

作为小白的我，在角落里看着群里的大佬们讨论的十分起劲儿，而我这个萌新对于他们所讨论的东西却听都没听过。

于是乎，新的一期学习笔记开整，本期学习笔记如题：《Cobalt Strike学习笔记》，简称《CS学习笔记》，这期笔记预计会更新28篇文章，学习资源来自B站视频，视频链接在文章底部。

由于这只是学习笔记，因此不会像教程一样详尽，一些我个人已经了解的东西或许不会记在笔记里，因此把笔记当做教程阅读是不合适的。

# 0x01 CS是什么？

Cobalt Strike是一款渗透测试神器，常被业界人称为CS神器。Cobalt Strike已经不再使用MSF而是作为单独的平台使用，它分为客户端与服务端，服务端是一个，客户端可以有多个，可被团队进行分布式协团操作。

Cobalt Strike集成了端口转发、扫描多模式端口Listener、Windows exe程序生成、Windows dll动态链接库生成、java程序生成、office宏代码生成，包括站点克隆获取浏览器的相关信息等。

早期版本Cobalt Srtike依赖Metasploit框架，而现在Cobalt Strike已经不再使用MSF而是作为单独的平台使用1。

这个工具的社区版是大家熟知的Armitage(一个MSF的图形化界面工具)，而Cobalt Strike大家可以理解其为Armitage的商业版。

# 0x02 CS的发展

* Armitage [2010-2012]

  Armitage是一个红队协作攻击管理工具，它以图形化方式实现了Metasploit框架的自动化攻击。Armitage采用Java构建，拥有跨平台特性。

* Cobalt Strike 1.x [2012-2014]

  Cobalt Strike 增强了Metasploit Framework在执行目标攻击和渗透攻击的能力。

* Cobalt Strike 2.x [2014-?]

  Cobalt Strike 2是应模拟黑客攻击的市场需求而出现的，Cobalt Strike 2是以malleable C2技术的需求为定位的，这个技术使Cobalt Strike的能力更强了一些。

* Cobalt Strike 3.x [2015-?]

  Cobalt Strike 3的攻击和防御都不用在Metasploit Framework平台（界面）下进行。

# 0x03 使用到的工具

* Cobalt Strike
* Kali
* Metasploit Framework
* PowerSploit
* PowerTools
* Veil Evasion Framework

> 参考链接：
>
> [https://zhuanlan.zhihu.com/p/93718885](https://zhuanlan.zhihu.com/p/93718885)
> 
>[https://www.bilibili.com/video/BV16b411i7n5](https://www.bilibili.com/video/BV16b411i7n5)
> 
>[https://blog.csdn.net/l1028386804/article/details/86675559](https://blog.csdn.net/l1028386804/article/details/86675559)
> 
>[https://www.freebuf.com/company-information/167460.html](https://www.freebuf.com/company-information/167460.html)
> 
>更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)

