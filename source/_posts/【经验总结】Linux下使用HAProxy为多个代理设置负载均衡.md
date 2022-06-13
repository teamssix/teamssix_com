---
title: 【经验总结】Linux下使用HAProxy为多个代理设置负载均衡
date: 2020-07-05 01:31:56
id: 200705-013156
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2020-07-05_00-19-41.png
summary: 在平时进行一些挖洞、扫描或者爬虫工作的时候，被封 IP 的情况时有发生，解决这个问题较好的方法可能就是挂代理了。但是代理有时也会被封，并且有的代理质量可能还不太高，这时采用负载均衡个人觉着是一种不错的解决方法。
tags:
- 经验总结
- 代理
- 负载均衡
categories:
- 经验总结
---

# 0x00 前言

在平时进行一些挖洞、扫描或者爬虫工作的时候，被封 IP 的情况时有发生，解决这个问题较好的方法可能就是挂代理了。但是代理有时也会被封，并且有的代理质量可能还不太高，这时采用负载均衡个人觉着是一种不错的解决方法。

在开启负载均衡时，系统能够自动切换到较为优质的代理线路上；同时由于系统经常自动切换 IP ，因此个人觉着也能在一定程度上减轻被封 IP 的风险。

<!--more-->

在 Windows 上，平时个人使用的代理工具自带就有负载均衡选项，但是无奈 Linux 下个人暂时还没能找到合适的工具，后来在网上查资料得知可以通过 Nginx 或 HAProxy 去配置负载均衡，虽然这样没有 Windows 下那样方便，但好在能解决这个问题。

通过 Nginx 或 HAProxy 的对比，HAProxy 有 Web 可视化页面，因此个人觉着会更直观些，当然这个因人而异。

在折腾了一天的时间后，终于在 Linux 下利用 HAProxy 配置好了负载均衡，下面就简单记录一下配置过程以及中间踩得一些坑。

# 0x01 准备工作

## 一些设备

* 一台 Linux 主机，用来做负载均衡服务器，这里以 Ubuntu 为例，其他 Linux 发行版基本上就一个安装命令与之不同。
* 一些可用的代理，这里以酸酸乳为例。

## 一些条件

所有代理的密码、加密方式、协议、混淆方式都必须一致，简而言之，除了代理的IP之外的信息都尽可能的保持一致。

达到以上条件后，就可以在 Linux 下利用 HAProxy 配置负载均衡了。

# 0x02 HAProxy 的安装与使用

HAProxy 可直接使用`apt install`进行安装，安装之前建议先将系统`apt-get update`一下。

```bash
apt install haproxy
```

安装之后，编辑配置文件，编辑之前建议先将配置文件进行备份。

```bash
mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.bak
vim /etc/haproxy/haproxy.cfg
```

为了便于直接复制配置文件信息，这里使用的`mv`命令进行备份，因此使用`vim`打开时，直接将以下配置文件信息根据自己情况修改复制到`/etc/haproxy/haproxy.cfg`即可。

```bash
global
    chroot  /var/lib/haproxy
    pidfile /var/run/haproxy.pid
    user    haproxy
    group   haproxy

defaults
    mode    tcp                         #服务器默认的工作模式
    balance roundrobin                  #服务器默认使用的均衡模式
    retries 3                           #三次连接失败表示服务器不可用
    maxconn 5000                        #最大连接数
    timeout connect 500ms               #连接超时
    timeout client  3s                  #客户端超时
    timeout server  3s                  #服务器超时

listen WebPanel
    mode    http                        #这里使用HTTP模式
    bind    127.0.0.1:50000             #WEB服务端口
    stats   refresh 5s                  #自动刷新时间
    stats   uri  /                      #WEB管理地址
    stats   auth admin:admin		   #账号密码
    stats   hide-version                #隐藏版本
    stats   admin if TRUE               #验证通过则赋予管理权

listen Server
    bind 127.0.0.1:8880	    			#服务IP端口	
    server proxy_name1 proxy_ip1:proxy_port1 check inter 500 rise 2 fall 4 weight 100	#酸酸乳服务器地址与端口
    server proxy_name2 proxy_ip2:proxy_port2 check inter 500 rise 2 fall 4 weight 100	#酸酸乳服务器地址与端口
    server proxy_name3 proxy_ip3:proxy_port3 check inter 500 rise 2 fall 4 weight 100	#酸酸乳服务器地址与端口
```

在上面的配置文件中，修改最后几行代理信息即可，即将其中的`proxy_name、proxy_ip、proxy_port`替换成自己的就行。

## 关于配置文件中的一些注意事项

* 配置文件中的注释信息需要删除，尤其是中文，否则 HAProxy 启动时会报错
* HAProxy 启动时如果报错，建议检查配置文件中是否存在格式错误、缺字多字的情况，这都会导致报错
* 如果 HAProxy 在公网服务器上建议将 Web 管理地址与登录的账号密码设置为较难猜解的信息
* 如果想了解配置文件中的更多信息可以查看参考链接中的文章

HAProxy 配置完后，直接使用`service`命令启动即可。

```bash
service haproxy start
```

此时，浏览器打开 HAProxy 的 Web 管理地址，输入账号密码后，看到以下页面，就说明 HAProxy 已经正在运行了。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2020-07-05_00-19-41.png)

在这个 Web 页面中，绿色的线路表示当前可用，红色线路表示当前可能是不可用状态，其他颜色所表示的意思可参考页面上方的示意信息。

# 0x03 代理客户端上的配置

配置好 HAProxy，就开始配置代理工具了。根据上面 HAProxy 配置文件，服务 IP 端口配置的为`127.0.0.1:8880`。

打开酸酸乳的配置文件，我这里是`user-config.json`文件，原文件内容如下：

```bash
{
    "server":"xxxxxx",
    "server_port":xxxx,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"xxxx",
    "timeout":600,
    "method":"xxxx",
    "protocol": "xxxx",
    "obfs": "xxxx",
    "obfs_param": "xxxx"
}
```

现在需要把文件中的`server`和`server_port`修改为 HAProxy 的服务 IP 端口信息，即 `127.0.0.1:8880`，修改后如下：

```bash
{
    "server":"127.0.0.1",
    "server_port":8880,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"xxxx",
    "timeout":600,
    "method":"xxxx",
    "protocol": "xxxx",
    "obfs": "xxxx",
    "obfs_param": "xxxx"
}
```

接下来，重启酸酸乳，使用 curl 进行一下测试。

```bash
curl --socks5-hostname 127.0.0.1:1080 cip.cc
```

```bash
root@ubuntu:~# curl --socks5-hostname 127.0.0.1:1080 cip.cc
IP      : 200.xxx.xxx.41
……省略……

root@ubuntu:~# curl --socks5-hostname 127.0.0.1:1080 cip.cc
IP      : 200.xxx.xxx.48
……省略……

root@ubuntu:~# curl --socks5-hostname 127.0.0.1:1080 cip.cc
IP      : 200.xxx.xxx.42
……省略……
```

通过测试，可以发现每次请求时，自己的 IP 都会改变，这就是 HAProxy 起到了作用。

这里只是进行一下测试，平时在`Linux`下使用代理的时候，更推荐使用`proxychains4`对命令进行代理。

## 一些注意事项

* 在修改代理客户端的代理配置文件之前，一定要确保当前配置文件是没问题的，以便于后期排错。
* 只有代理客户端的配置文件是配置的 HAProxy 服务IP端口信息，其他比如浏览器、curl 等都是和原来一样代理到代理工具的 IP、端口上即可，这个是我陷的比较深的一个坑。

# 0x04 总结

在平时不管挖洞还是扫描、爬虫，个人觉着开启负载均衡之后，在一定程度上应该是能降低被封 IP 的风险的，同时代理质量也能上去。

但是在涉及账号登录情况下，就建议不要开启负载均衡了，不然有的系统监测到异地登录就会把账号退出了，手动登录之后，发现系统又检测到了异地登录，然后又给账号退出了……

总的来说，以上就是我个人的一些小总结以及踩过的一些坑，希望大家碰到同类问题时，这篇文章能够对其有所帮助。

> 参考链接：
>
> [https://44i.im/index.php/2020/02/10/ss-ssr-v2ray-balance/](https://44i.im/index.php/2020/02/10/ss-ssr-v2ray-balance/)
>
> [https://tianws.github.io/skill/2019/07/11/gfw/](https://tianws.github.io/skill/2019/07/11/gfw/)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)