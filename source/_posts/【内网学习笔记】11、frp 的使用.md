---
title: 【内网学习笔记】11、frp 的使用
date: 2021-06-11 17:15:26
id: 210611-171526
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210609155435.png
tags:
- 红队
- 学习笔记
- 内网
categories:
- 内网学习笔记
---

## 1、介绍

相较于前一篇文章介绍的 ew 的年代久远，frp 就好的多了，基本上隔几天就会发布新的版本，最新的一版更新还就在几天前。

在实战中，大家较多使用的也是 frp，frp 项目地址：[https://github.com/fatedier/frp](https://github.com/fatedier/frp)

至于下载安装直接在项目的 releases 里下载自己对应的系统版本就行。

## 2、使用

官方使用文档：[https://gofrp.org/docs/](https://gofrp.org/docs/)

frp 分成服务端和客户端，分别叫 frps 和 frpc，配置文件分别对应 frps.ini 和 frpc.ini

> 以下环境均为本地环境，VPS IP 为 172.16.214.52，目标主机 IP 为 192.168.7.110

### a、内网端口穿透

 场景：内网主机可出网，想从公网访问内网主机的 3389 端口

在 VPS 上开启服务端，这里以 kali 为例，首先修改配置文件 frps.ini

```
[common]
bind_port = 4444
```

然后启动服务端

```
frps -c frps.ini
```

```
> ./frps -c frps.ini
2021/06/09 03:45:03 [I] [root.go:200] frps uses config file: frps.ini
2021/06/09 03:45:03 [I] [service.go:192] frps tcp listen on 0.0.0.0:4444
2021/06/09 03:45:03 [I] [root.go:209] frps started successfully
```

配置客户端配置文件

```
[common]
# 服务端 IP
server_addr = vps_ip
# 服务端端口
server_port = 4444

[rdp]
type = tcp
local_ip = 127.0.0.1
local_port = 3389
# 连接 vps 的端口
remote_port = 3389
```

```
> .\frpc.exe -c frpc.ini
2021/06/09 15:50:29 [I] [service.go:304] [72904e8037a7fdf8] login to server success, get run id [72904e8037a7fdf8], server udp port [0]
2021/06/09 15:50:29 [I] [proxy_manager.go:144] [72904e8037a7fdf8] proxy added: [rdp]
2021/06/09 15:50:29 [I] [control.go:180] [72904e8037a7fdf8] [rdp] start proxy success
```

此时，在 vps 上访问本地的 3389 端口就会访问到内网主机的 3389 端口了。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210609155435.png)

### b、建立 socks 代理

场景：内网主机可出网，想把内网主机作为跳板机使用

上面的场景只是利用 frp 访问了内网指定机器的指定端口，我们还可以利用 frp 将内网主机作为跳板机使用。

这次我们用上 frp 的 web 控制面板以及访问密码等功能，让我们建立的连接更加安全、方便。

在 VPS 上开启服务端，服务端配置文件如下：

```
[common]
bind_port = 4444

# 客户端认证 token
token = 123456

# 设置 frps 仪表盘端口、账号和密码，实战中用处貌似不大，但如果设置一定要设置强密码
dashboard_port = 8000
dashboard_user = admin
dashboard_pwd = password
```

> 实战中，为了更好的隐藏自己，最好还是要设置通过域名访问

配置好文件后，启动服务端

```
frps -c frps.ini
```

```
./frps -c frps.ini
2021/06/09 04:06:34 [I] [root.go:200] frps uses config file: frps.ini
2021/06/09 04:06:35 [I] [service.go:192] frps tcp listen on 0.0.0.0:4444
2021/06/09 04:06:35 [I] [service.go:294] Dashboard listen on 0.0.0.0:8000
2021/06/09 04:06:35 [I] [root.go:209] frps started successfully
```

配置客户端文件

```
[common]
server_addr = vps_ip
server_port = 4444
# 客户端认证 token，需要和服务端 token 保持一致
token = 123456

# 启用加密，防止流量被拦截
use_encryption = true
# 启用压缩，提升流量转发速度
use_compression = true

[socks5]
type = tcp
# 连接 vps 的端口
remote_port = 1080
plugin = socks5
```

开启客户端

```
frpc -c frpc.ini
```

```
> .\frpc.exe -c frpc.ini
2021/06/09 16:11:21 [I] [service.go:304] [ee7ad330ab4e6036] login to server success, get run id [ee7ad330ab4e6036], server udp port [0]
2021/06/09 16:11:21 [I] [proxy_manager.go:144] [ee7ad330ab4e6036] proxy added: [socks5]
2021/06/09 16:11:21 [I] [control.go:180] [ee7ad330ab4e6036] [socks5] start proxy success
```

测试 VPS IP 的 1080 的 socks5 代理，发现已经连通了。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210609161458.png)

打开 frps 仪表盘，登录后，可以看到当前连接数据的相关信息

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210609161859.png)

frp 的参数远不止文章中提到的这些，更多功能可以参考下面的参考文章。

> 参考文章：
>
> [https://www.jianshu.com/p/331aa59fff5d](https://www.jianshu.com/p/331aa59fff5d)
>
> [https://www.anquanke.com/post/id/184855](https://www.anquanke.com/post/id/184855)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)
