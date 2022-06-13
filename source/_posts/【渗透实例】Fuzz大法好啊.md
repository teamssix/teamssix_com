---
title: 【渗透实例】Fuzz大法好啊
date: 2020-03-10 22:10:08
id: 200310-221008
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/2Snipaste_2020-03-10_14-19-23.png
tags:
- 渗透实例
- Fuzz
- Xray
categories:
- 渗透实例
---

# 0x00 前言

> 本文仅供学习分享用途，严禁用于违法用途

在搞站的时候，经过一顿操作后只发现了一堆低危，过了一段时间看看Xray，居然发现一个XSS

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/1Snipaste_2020-03-10_14-10-35.png)

嗯~ Xray 真香！

<!--more-->

随后进行复现，无奈多次尝试无果，好玩的是每换一个Payload，流量经过 Xray ，Xray 还会继续报这个漏洞，并且每次Payload都不一样，这感情好，直接用 Xray 给的 Payload 吧，尝试了半天，居然没有一个成功的，当时 Xray 的界面是这样的

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/2Snipaste_2020-03-10_14-19-23.png)

虽然没有一个成功复现的，但是这看着还是挺爽的，一会儿报个漏洞一会儿报个漏洞的

# 0x01 Fuzz大法好

屡试不行之后便想到了Fuzz，那先去GitHub上找个字典

> https://github.com/TheKingOfDuck/fuzzDicts

有了字典之后，上Burp

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/3Snipaste_2020-03-10_14-26-42.png)

两千多的字典最后只有3个Payload响应200，不管了，先一个一个试试吧，那就先来试试第一个Payload

```
"+alert(16)+"
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/4Snipaste_2020-03-10_14-29-54.png)

哎呀呀，不得不说，妙啊！

之后我再试另外几个 Payload 就不管用了

啧啧，Xray 真香、Fuzz 真香。

> 更多信息欢迎关注我的微信公众号：TeamsSix
>

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)