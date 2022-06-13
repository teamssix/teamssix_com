---
title: 【内网学习笔记】12、nps 的使用
date: 2021-06-12 21:37:04
id: 210612-213704
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210609172144.png
tags:
- 红队
- 学习笔记
- 内网
categories:
- 内网学习笔记
---

## 1、介绍

nps 项目地址：[https://github.com/ehang-io/nps](https://github.com/ehang-io/nps)

也是一款还在更新的内网穿透工具，相较于 frp，nps 的 web 管理就要强大很多了。

nps 和 frp 一样功能都很多，这里就主要记录下平时经常用到的 SOCKS5 代理模式。

## 2、安装

nps 不同于 frp 的开箱即用，nps 的服务端需要安装才能使用，这里以 kali 下的安装为例。

在 nps 项目的 releases 中下载好自己对应系统的版本后，解压安装

```
tar -zxvf linux_amd64_server.tar.gz
./nps install
```

## 3、使用

官方使用文档：[https://ehang-io.github.io/nps](https://ehang-io.github.io/nps)

启动服务端，默认 Web 管理界面端口 8080 

```
nps start
```

启动 nps 后，直接访问服务端的 8080 端口，输入默认密码 admin/123 进行登录，不难看出，这 web 界面确实比 frp 的丰富很多。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210609170306.png)

nps 的使用也很简单，界面语言也可选择中文。

首先新增一个客户端，点击 “客户端” --》“新增”，打开新增客户端页面，填写相关信息后，点击新增即可

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210609171610.png)

新增之后，刷新一下可以看到刚刚添加的记录，点击刚刚新增记录里的“加号”还能直接看到在客户端上要运行的命令，这个可谓是很贴心了。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210609172254.png)

复制命令到客户端上运行，服务端这边就能看到目标已经上线了，连接状态也由离线变成了在线。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210609172144.png)

如果想创建一个 SOCKS5 代理也很简单，直接点击 “SOCKS 代理”--》“新增”，输入客户端的 ID 和代理的端口，然后新增即可。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210609172512.png)

之后直接设置 SOCKS5 代理 IP 为 nps 服务端 IP ，端口这里设置的是 1080，这样就建立了一个 SOCKS 代理，如果新增设置客户端的时候，设置了认证账号密码，那么在连接 SOCKS 代理的时候，也要添加上对应的账号和密码。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210609173211.png)

在这整个过程中都没有修改配置文件等等操作，真的是很方便了。

> 参考文章：
>
> [https://ehang-io.github.io/nps/](https://ehang-io.github.io/nps/)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)
