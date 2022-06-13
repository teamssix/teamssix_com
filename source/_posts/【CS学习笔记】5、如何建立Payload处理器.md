---
title: 【CS学习笔记】5、如何建立Payload处理器
date: 2020-04-19 15:04:54
id: 200419-150454
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs5-2.png
summary: 一旦监听器建立起来，团队成员只需要知道这个监听器的名称即可，不用关心监听器背后的基础环境，接下来将深入了解如何准确配置监听器。
tags:
- Cobalt Strike
- 学习笔记
- 红队
categories:
- CS 学习笔记
---

这一小节学起来感觉有些吃力，里面很多概念理解的不是很清楚，如果有大佬看到描述错误的地方欢迎留言指正，避免误导他人。

再次声明，这只是我的个人学习笔记，不要当成教程去看，建议想学习CS的小伙伴可以看看A-TEAM的中文手册或者网上的一些视频教程。

# 0x00 监听器管理

* 什么是监听器

  顾名思义，监听器就是等待被入侵系统连接自己的一个服务。

* 监听器的作用

  主要是为了接受payload回传的各类数据，类似于MSF中handler的作用。

  比如payload在目标机器执行以后，就会回连到监听器然后下载执行真正的shellcode代码。

一旦监听器建立起来，团队成员只需要知道这个监听器的名称即可，不用关心监听器背后的基础环境，接下来将深入了解如何准确配置监听器。

一个监听器由用户定义的名称、payload 类型和几个特定于 payload 的选项组成。

监听器的名字一般由以下结构组成：

```
Operating System/Payload/Stager
```

例如：

```
windows/beacon_http/reverse_http
```

# 0x01 什么是传输器

攻击载荷`payload`就是攻击执行的内容。攻击载荷通常被分为两部分：传输器`stager` 和传输体`stage`。

传输器`stager`是一个小程序，用于连接、下载传输体`stage`，并插入到内存中。

我个人理解为：攻击载荷里真正用于攻击的代码是在传输体里。

所以为什么要有传输体？直接把攻击载荷插入到内存中不更方便快捷、更香么，搞得又是传输器又是传输体的。

需要传输体是因为在很多攻击中对于能加载进内存，并在成功漏洞利用后执行的数据大小存在严格限制。这就导致在攻击成功时，很难嵌入额外的攻击载荷，正是因为这些限制，才使得传输器变得有必要了。

# 0x02 创建监听器

在CS客户端中打开 Cobalt Strike —》Listeners，之后点击Add，此时弹出New Listener窗口，在填写监听器的相关信息之前，需要先来了解监听器有哪些类型。

Cobalt Strike有两种类型的监听器：

* Beacon

  Beacon直译过来就是灯塔、信标、照亮指引的意思，Beacon是较为隐蔽的后渗透代理，笔者个人理解Beacon类型的监听器应该是平时比较常用的。Beacon监听器的名称例如：

  ```
  windows/beacon_http/reverse_http
  ```

* Foreign

  Foreign直译就是外部的，这里可以理解成`对外监听器`，这种类型的监听器主要作用是给其他的Payload提供别名，比如Metasploit 框架里的Payload，笔者个人理解Foreign监听器在一定程度上提高了CS的兼容性。对外监听器的名称例如：

  ```
  windows/foreign/reverse_https
  ```

# 0x03 Beacon是什么

* Beacon是CS的Payload
* Beacon有两种通信模式。一种是异步通信模式，这种模式通信效率缓慢，Beacon回连团队服务器、下载任务、然后休眠；另一种是交互式通信模式，这种模式的通信是实时发生的。
* 通过HTTP、HTTPS和DNS出口网络
* 使用SMB协议的时候是点对点通信
* Beacon有很多的后渗透攻击模块和远程管理工具

# 0x04 Beacon的类型

* HTTP 和 HTTPS Beacon

  HTTP和HTTPS Beacon也可以叫做Web Beacon。默认设置情况下，HTTP 和 HTTPS Beacon 通过 HTTP GET 请求来下载任务。这些 Beacon 通过 HTTP POST 请求传回数据。

  ```
  windows/beacon_http/reverse_http
  windows/beacon_https/reverse_https
  ```

* DNS Beacon

  ```
  windows/beacon_dns/reverse_dns_txt
  windows/beacon_dns/reverse_http
  ```

* SMB Beacon

  SMB Beacon也可以叫做pipe beacon

  ```
  windows/beacon_smb/bind_pipe
  ```

# 0x05 创建一个HTTP Beacon

点击 Cobalt Strike  --> Listeners 打开监听器管理窗口，点击Add，输入监听器的名称、监听主机地址，因为这里是要创建一个HTTP Beacon，所以其他的默认就行，最后点击Save

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs5-1.png)

此时可以测试一下刚才设置的监听器，点击Attack --> Web Drive-by --> Scripted Web Delivery(s) ，在弹出的窗口中选择刚才新添的Listener，因为我的靶机是64位的，所以我把Use x64 payload也给勾选上了，最后点击Launch

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs5-2.png)

复制弹窗的命令，放到靶机中运行

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs5-3.png)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs5-4.png)

此时，回到CS，就可以看到已经靶机上线了

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs5-5.png)

# 0x06 重定向器

刚才创建了一个HTTP Beacon，接下来来看一下重定向器`Redirectors`

重定向器是一个位于CS团队服务器和目标网络之间的服务器，这个重定向器通俗的来说就是一个代理工具，或者说端口转发工具，担任CS服务器与目标服务器之间的跳板机角色，整体流量就像下面这样。

```
目标靶机 <-------->多个并列的重定向器<------>CS服务器
```

重定向器在平时的攻击或者防御的过程中起到很重要的作用，主要有以下两点：

* 保护自己的CS服务器，避免目标发现自己的真实IP
* 提高整体可靠性，因为可以设置多个重定向器，因此如果有个别重定向器停止工作了，整体上系统依旧是可以正常工作的

# 0x07 创建一个重定向器

这里就使用自己的内网环境作为测试了，首先理清自己的IP

CS服务器IP：192.168.175.129

目标靶机IP：192.168.175.130

重定向器IP：192.168.175.132、192.168.175.133

首先，需要先配置重定向器的端口转发，比如这里使用HTTP Beacon，就需要将重定向器的80端口流量全部转发到CS服务器上，使用socat的命令如下：

```
socat TCP4-LISTEN:80,fork TCP4:192.168.175.129:80
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs5-7.png)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs5-6.png)

如果提示没有socat命令，安装一下即可。重定向器设置好之后，就新建一个HTTP Beacon，并把重定向器添加到HTTP Hosts主机列表中

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs5-8.png)

此时可以测试一下重定向器是否正常工作，在CS中打开 View --> Web Log，之后浏览器访问CS服务器地址，也就是这里的192.168.175.129

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs5-9.png)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs5-10.png)

可以看到CS是能够正常接收到流量的，说明重定向器已经配置OK了，此时按照上面创建一个HTTP Beacon的操作，创建一个HTTP Beacon，并在靶机中运行

当靶机上线的时候，观察靶机中的流量，可以看到与靶机连接的也是重定向器的IP

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs5-11.png)

在CS中也可以看到上线主机的外部IP也是重定向器的IP，此时如果关闭一个重定向器，系统依旧可以正常工作。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs5-12.png)

# 0x08 HTTPS Beacon

HTTPS Beaocn和HTTP Beacon一样，使用了相同的Malleable C2配置文件，使用GET和POST的方式传输数据，不同点在于HTTPS使用了SSL，因此HTTPS Beacon就需要使用一个有效的SSL证书，具体如何配置可以参考：[https://www.cobaltstrike.com/help-malleable-c2#validssl](https://www.cobaltstrike.com/help-malleable-c2#validssl)

> 参考链接：
>
> [https://www.bilibili.com/video/BV16b411i7n5](https://www.bilibili.com/video/BV16b411i7n5)
>
> [https://klionsec.github.io/2017/09/23/cobalt-strike/](https://klionsec.github.io/2017/09/23/cobalt-strike/)
>
> [https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf](https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)