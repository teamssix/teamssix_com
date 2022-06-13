---
title: 【CS学习笔记】2、如何连接团队服务器
date: 2020-04-19 15:04:32
id: 200419-150432
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs2-6.png
summary: Cobalt Strike使用C/S架构，Cobalt Strike的客户端连接到团队服务器，团队服务器连接到目标，也就是说Cobalt Strike的客户端不与目标服务器进行交互，那么Cobalt Strike的客户端如何连接到团队服务器就是本文所学习的东西。
tags:
- Cobalt Strike
- 学习笔记
- 红队
categories:
- CS 学习笔记
---

# 0x00 前言

上一篇说了一些有的没的，现在来正式学习Cobalt Strike。

Cobalt Strike使用C/S架构，Cobalt Strike的客户端连接到团队服务器，团队服务器连接到目标，也就是说Cobalt Strike的客户端不与目标服务器进行交互，那么Cobalt Strike的客户端如何连接到团队服务器就是本文所学习的东西。

# 0x01 准备工作

Cobalt Strike的客户端想连接到团队服务器需要知道三个信息：

* 团队服务器的外部IP地址
* 团队服务器的连接密码
* （此项可选）决定Malleable C2工具的哪一个用户配置文件被用于团队服务器

知道这些信息后，就可以使用脚本开启团队服务器了，值得注意的是Cobalt Strike团队服务器只能运行在Linux环境下。

# 0x02 开启团队服务器

开启团队服务器命令一般如下所示：

```
./teamserver your_ip your_passowrd [config_file]
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs2-1.png)

服务端开启后，就可以开启客户端进行连接了

# 0x03 连接到团队服务器

在Linux下，直接运行start.sh脚本文件，输入团队服务器的IP、密码和自己的用户名进行连接

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs2-2.png)

点击Connect连接后，会有个提示信息，如果承认提示信息中的哈希值就是所要连接团队服务器的哈希值就点击Yes，随后即可打开CS客户端界面

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs2-3.png)

在Windows下的连接方法也基本一致，直接双击start.bat文件，输入IP、密码、用户名，点击Connect即可

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs2-4.png)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs2-5.png)

在连接后，团队之间就可以通过客户端进行沟通，信息共享

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs2-6.png)

Cobalt Strike不是用来设计指导在一个团队服务器下进行工作的，而是被设计成在一次行动中使用多个团队服务器。

这样设计的目的主要在于运行安全，如果一个团队服务器停止运行了，也不会导致整个行动的失败，所以接下来看看如何连接到多个团队服务器。

# 0x05 连接到多个团队服务器

Cobalt Strike连接到多个团队服务器也很简单，直接点击左上角的加号，输入其他团队服务器的信息后，即可连接

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs2-7.png)

> 参考链接：
>
> [https://www.bilibili.com/video/BV16b411i7n5](https://www.bilibili.com/video/BV16b411i7n5)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)