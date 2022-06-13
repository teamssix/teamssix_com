---
title: 【内网学习笔记】18、LLMNR 和 NetBIOS 欺骗攻击
date: 2021-07-29 19:10:48
id: 210729-191048
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210729190444.png
tags:
- 红队
- 学习笔记
- 内网
categories:
- 内网学习笔记
---

## 0、前言

如果已经进入目标网络，但是没有获得凭证，可以使用 LLMNR 和 NetBIOS 欺骗攻击对目标进行无凭证条件下的权限获取。

## 1、基本概念

### LLMNR

本地链路多播名称解析（LLMNR）是一种域名系统数据包格式，当局域网中的 DNS 服务器不可用时，DNS 客户端就会使用 LLMNR 解析本地网段中机器的名称，直到 DNS 服务器恢复正常为止。

从 Windows Vista 开始支持 LLMNR ，Linux 系统也通过 systemd 实现了此协议，同时 LLMNR 也支持 IPv6。

### NetBIOS

NetBIOS 协议是由 IBM 公司开发，主要用于数十台计算机的小型局域网，根据 NetBIOS 协议广播获得计算机名称，并将其解析成相应的 IP 地址。

从 Windows NT 以后版本的所有操作系统中都可以使用 NetBIOS，不过 NetBIOS 不支持 IPv6.

NetBIOS 提供的三种服务：

​	i、NetBIOS-NS（名称服务）：主要用于名称注册和解析，以启动会话和分发数据报，该服务默认监听 UDP 137 端口，也可以使用 TCP 的 137 端口进行监听。

​	ii、Datagram Distribution Service（数据报分发服务）：无连接服务，该服务负责进行错误检测和恢复，默认监听 UDP 138 端口。

​	iii、Session Service（会话服务）：允许两台计算机建立连接，默认使用 TCP 139 端口。

### Net-NTLM Hash

> NTLM 即 NT LAN Manager，NTLM 是指 telnet 的一种验证身份方式，即问询/应答协议，是 Windows NT 早期版本的标准安全协议。

Net-NTLM Hash 不同于 NTLM Hash，NTLM Hash 是 Windows 登录密码的 Hash 值，可以在 Windows 系统的 SAM 文件或者域控的 NTDS.dit 文件中提取到出来，NTLM Hash 支持哈希传递攻击。 

Net-NTLM Hash 是网络环境下 NTLM 认证的 Hash，使用 Responder 抓取的通常就是 Net-NTLM Hash，该 Hash 不能进行哈希传递，但可用于 NTLM 中继攻击或者使用 Hashcat 等工具碰撞出明文进行横向。

## 2、利用

Responder 是一款使用 Python 编写用于毒化 LLMNR 和 NBT-NS 请求的一款工具。

假设我们已连接到 Windows Active Directory 环境，当网络上的设备尝试用 LLMNR 和 NBT-NS（NetBIOS 名称服务）请求来解析目标机器时，Responder 就会伪装成目标机器。

当受害者机器尝试登陆攻击者机器，Responder 就可以获取受害者机器用户的 Net-NTLM 哈希值。

Responder 项目地址：[https://github.com/lgandx/Responder](https://github.com/lgandx/Responder)

Responder 不支持 Windows，这里使用 Kali 进行演示。

Responder 开启监听，-I 指定网卡，这里 eth1 的 IP 为 192.168.7.65

```
python Responder.py -I eth1
```

开启监听后，当目标主机上有人访问 Responder 主机的共享目录时，就会看到对方的 Net-NTLM 哈希值了。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210729190444.png)

再利用 Hashcat 进行碰撞

```
hashcat -m 5600 hash.txt password.txt -D 1
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210729190835.png)

> 参考文章：
>
> [https://www.jianshu.com/p/a210528f9b35](https://www.jianshu.com/p/a210528f9b35)
>
> [https://baike.baidu.com/item/NTLM/6371298](https://baike.baidu.com/item/NTLM/6371298)
>
> [https://baike.baidu.com/item/LLMNR/1116392](https://baike.baidu.com/item/LLMNR/1116392)
>
> [https://www.freebuf.com/articles/system/194549.html](https://www.freebuf.com/articles/system/194549.html)
>
> [https://baike.baidu.com/item/NetBIOS%E5%8D%8F%E8%AE%AE/8938996](https://baike.baidu.com/item/NetBIOS%E5%8D%8F%E8%AE%AE/8938996)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)
