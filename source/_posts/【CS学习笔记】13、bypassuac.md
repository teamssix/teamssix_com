---
title: 【CS学习笔记】13、bypassuac
date: 2020-04-19 15:05:52
id: 200419-150552
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs13-2.png
summary: Bypass UAC 有两个步骤，分别是：利用 UAC 漏洞来获取一个特权文件副本；使用 DLL 劫持进行代码执行。
tags:
- Cobalt Strike
- 学习笔记
- 红队
categories:
- CS 学习笔记
---

Bypass UAC 有两个步骤，分别是：

1、利用 UAC 漏洞来获取一个特权文件副本

2、使用 DLL 劫持进行代码执行

首先使用`shell whoami /groups`查看当前上线主机用户的所属组及 UAC 等级

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs13-1.png)

通过返回信息可以看出，当前用户为管理员权限，UAC 等级为中，根据上一节中关于的介绍，此时可以使用`bypassuac`进行提权。

首先，右击会话，选择`Access --> Elevate`，这里选择一个 SMB Beacon，Exploit 选择`uac-token-duplication`，最后 Launch 即可。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs13-2.png)

待 Beacon Check in 后，当前用户 UAC 为高权限的会话便会上线了。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs13-3.png)

> 参考链接：
>
> [https://www.bilibili.com/video/BV16b411i7n5](https://www.bilibili.com/video/BV16b411i7n5)
>
> [https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf](https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)