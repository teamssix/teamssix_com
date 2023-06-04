---
title: CF 云环境利用框架，一键化渗透云上内网
date: 2022-07-11 11:24:05
id: 220711-112405
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207111134217.png
tags:
- 云安全
- 工具分享
- 云服务
- CF
categories:
- 云安全
---

## 前言

当我们平时拿到云服务的访问凭证即 Access Key 时，通常的做法可能是看下对方的 OSS 对象存储、或者在实例上执行个命令，但 AK 的利用远不止这些，通过 AK 我们可以做太多太多的事情，为了方便 AK 的利用，于是有了这个工具。

CF 是一个云环境利用框架，主要用来方便红队人员在获得云服务的访问凭证的后续工作。

项目地址：[github.com/teamssix/cf](https://github.com/teamssix/cf)

下载地址：[github.com/teamssix/cf/releases](https://github.com/teamssix/cf/releases)

代码完全开源，师傅们可以放心使用，提前祝师傅打下一个又一个点、拿下一个又一个云上管理员权限。

截止到 2022 年 7 月 10 日，CF 已迭代到 v0.2.2 版本，目前 CF 仅支持阿里云，当前 CF 已支持以下功能：

- 一键列出目标 AK 的 OSS、ECS、RDS 服务
- 一键获得实例上的临时访问凭证
- 一键为实例反弹 Shell
- 一键接管控制台
- 一键为所有实例执行三要素，方便你懂得
- 一键查看当前配置的权限
- ……

## 使用手册

使用手册请参见：[wiki.teamssix.com/cf](https://wiki.teamssix.com/cf)

[![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207111133023.png)](https://wiki.teamssix.com/cf)

## 简单上手

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207111134217.png)

配置 CF

```bash
cf configure
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207111134880.png)

一键列出当前访问凭证的云服务资源

```bash
cf ls
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207111135902.png)

一键列出当前访问凭证的权限

```bash
cf ls permissions
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207111135732.png)

一键接管控制台

```bash
cf console
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207111135941.png)

查看 CF 为实例执行命令的操作的帮助信息

```bash
cf ecs exec -h
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207111135572.png)

一键为所有实例执行三要素，方便 HVV

```
cf ecs exec -b
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207111136653.png)

一键获取实例中的临时访问凭证数据

```bash
cf ecs exec -m
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207111136684.png)

如果感觉还不错的话，师傅记得给个 Star 呀 ~，另外 CF 的更多使用方法可以参见使用手册：[wiki.teamssix.com/cf](wiki.teamssix.com/cf)

>  更多信息欢迎关注我的个人微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202204152148071.png)
