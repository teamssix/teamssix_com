---
title: 【CS学习笔记】3、如何进行分布操作
date: 2020-04-19 15:04:40
id: 200419-150440
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/2019-12-04-101402.jpg
summary: 这里介绍最基本的团队服务模型，具体由三个服务器构成，具体如下所示。
tags:
- Cobalt Strike
- 学习笔记
- 红队
categories:
- CS 学习笔记
---

# 0x00 最基本的团队服务模型

这里介绍最基本的团队服务模型，具体由三个服务器构成，具体如下所示：

* 临时服务器（Staging Servers）

  临时服务器介于持久服务器和后渗透服务器之间，它的作用主要是方便在短时间内对目标系统进行访问。

  它也是最开始用于传递payload、获取初始权限的服务器，它承担初始的权限提升和下载持久性程序的功能，因此这个服务器有较高暴露风险。

* 持久服务器（Long Haul Servers）

  持久服务器的作用是保持对目标网络的长期访问，所以持久服务器会以较低的频率与目标保持通信。

* 后渗透服务器（Post-Exploitation Servers）

  主要进行后渗透及横向移动的相关任务，比如对目标进行交互式访问

# 0x01 可伸缩红队操作模型

可伸缩红队操作模型（Scaling Red Operations）分为两个层次，第一层次是针对一个目标网络的目标单元；第二层次是针对多个目标网络的权限管理单元。

目标单元的工作：

* 负责具体目标或行动的对象
* 获得访问权限、后渗透、横向移动
* 维护本地基础设施

访问管理单元的工作：

* 保持所有目标网络的访问权限
* 获取访问权限并接收来自单元的访问
* 根据需要传递对目标单元的访问
* 为持续回调保持全局基础环境

# 0x02 团队角色

* 开始渗透人员

  主要任务是进入目标系统，并扩大立足点

* 后渗透人员

  主要任务是对目标系统进行数据挖掘、对用户进行监控，收集目标系统的密钥、日志等敏感信息

* 本地通道管理人员

  主要任务有建立基础设施、保持shell的持久性、管理回调、传递全局访问管理单元之间的会话

# 0x03 日志记录

Cobalt Strike的日志文件在团队服务器下的运行目录中的`logs`文件夹内，其中有些日志文件名例如`beacon_11309.log`，这里的`11309`就是beacon会话的ID。

按键的日志在`keystrokes`文件夹内，截屏的日志在`screenshots`文件夹内，截屏的日志名称一般如`screen_015321_4826.jpg`类似，其中`015321`表示时间（1点53分21秒），`4826`表示ID

> 参考链接：
>
> [https://www.bilibili.com/video/BV16b411i7n5](https://www.bilibili.com/video/BV16b411i7n5)
>
> [http://blog.leanote.com/post/snowming/62ec1132a2c9](http://blog.leanote.com/post/snowming/62ec1132a2c9)
>
> [https://blog.cobaltstrike.com/2014/09/09/infrastructure-for-ongoing-red-team-operations/](https://blog.cobaltstrike.com/2014/09/09/infrastructure-for-ongoing-red-team-operations/)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)