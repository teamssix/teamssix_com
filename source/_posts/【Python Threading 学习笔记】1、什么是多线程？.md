---
title: 【Python Threading 学习笔记】1、什么是多线程？
date: 2019-10-31 20:22:53
id: 191031-202253
tags:
- Python
- 多线程
- 学习笔记
categories:
- Python 学习笔记
---
多线程类似于同时执行多个不同程序，比如一个很大的数据，直接运行的话可能需要10秒钟才能运行完。
<!--more-->
但如果使用Threading或者说使用多线程，我们把数据分成5段，每一段数据都放到一个单独的线程里面运算，所有线程同时开始。

这就好比原本一个工作只有一个人在做，但现在有了5个人同时在做，很明显可以大大的提高效率，节省时间。

如果平时有用过IDM下载东西的小伙伴，在下载文件的时候可以打开显示细节，就可以看到多个线程同时下载，传输速度基本能达到本地带宽的最高速度，下图可以很直观的看到多个线程同时下载的过程。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Threading1.gif)

>参考文章：https://morvanzhou.github.io/tutorials/python-basic/threading

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)