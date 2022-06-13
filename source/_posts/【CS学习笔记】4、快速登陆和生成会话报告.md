---
title: 【CS学习笔记】4、快速登陆和生成会话报告
date: 2020-04-19 15:04:47
id: 200419-150447
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/2019-12-04-101402.jpg
summary: Cobalt Strike生成报告的目的在于培训或帮助蓝队，在`Reporting`菜单栏中就可以生成报告，关于生成的报告有以下特点。
tags:
- Cobalt Strike
- 学习笔记
- 红队
categories:
- CS 学习笔记
---

# 0x00 介绍

Cobalt Strike生成报告的目的在于培训或帮助蓝队，在`Reporting`菜单栏中就可以生成报告，关于生成的报告有以下特点：

* 输出格式为PDF或者Word格式
* 可以输出自定义报告并且更改图标（Cobalt Strike --> Preferences -->Reporting）
* 可以合并多个团队服务器的报告，并可以对不同报告里的时间进行校正

# 0x01 导出报告类型

* 活动报告（Activity Report）
  此报告中提供了红队活动的时间表，记录了每个后渗透活动。
* 主机报告（Hosts Report）
  此报告中汇总了Cobalt Strike收集的主机信息，凭据、服务和会话也会在此报告中。
* 侵害指标报告（Indicators of Compromise）
  此报告中包括对C2拓展文件的分析、使用的域名及上传文件的MD5哈希。
* 会话报告（Sessions Report）
  此报告中记录了指标和活动，包括每个会话回连到自己的通信路径、后渗透活动的时间线等。
* 社工报告（Social Engineering Report）
  此报告中记录了每一轮网络钓鱼的电子邮件、谁点击以及从每个点击用户那里收集的信息。该报告还显示了Cobalt Strike的System profiler发现的应用程序。
* 战术、技巧和程序报告（Tactics,Techniques,and Procedures）
  此报告将自己的Cobalt Strike行动映射到MITRE的ATT&CK矩阵中的战术，具体可参考[https://attack.mitre.org/](https://attack.mitre.org/)

> 参考链接：
>
> [https://www.bilibili.com/video/BV16b411i7n5](https://www.bilibili.com/video/BV16b411i7n5)
>
> [https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf](https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)