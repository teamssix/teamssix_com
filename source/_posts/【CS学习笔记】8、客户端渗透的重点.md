---
title: 【CS学习笔记】8、客户端渗透的重点
date: 2020-04-19 15:05:17
id: 200419-150517
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs8-6.png
summary: 客户端攻击根据教程直译过来就是一种依靠应用程序使用控制端来进行的可视化攻击。
tags:
- Cobalt Strike
- 学习笔记
- 红队
categories:
- CS 学习笔记
---

# 0x00 前言

**什么是客户端攻击**

客户端攻击根据教程直译过来就是一种依靠应用程序使用控制端来进行的可视化攻击。

`原文：A client-side attack is an attack against an application used to view attacker controlled content.`

**为什么要进行客户端攻击**

随着时代发展到了今天，在有各种WAF、防火墙的情况下，各种漏洞已经很难像过去那么好被利用了，攻击者想绕过防火墙发动攻击也不是那么容易的了。

而当我们发送一个钓鱼文件到客户端上，再由客户端打开这个文件，最后客户端穿过防火墙回连到我们，此时在客户端上我们就获得了一个立足点`foothold`。这样的一个过程是相对而言是较为容易的，这也是为什么要进行客户端攻击。

<!--more-->

# 0x01 如何获得客户端上的立足点

1、尽可能多的了解目标环境，即做好信息收集工作

2、创建一个虚拟机，使它与目标环境尽可能的一致，比如操作系统、使用的浏览器版本等等都需要保证严格一致

3、攻击刚刚创建的虚拟机，这会是最好的攻击目标

4、精心策划攻击方法，达到使目标认为这些攻击行为都是正常行为的效果

5、将精心制作的钓鱼文件发送给目标，比如钓鱼邮件

如果这五步都非常细致精心的去准备，那么攻击成功的概率会大幅提升。

# 0x02 系统侦察

系统侦察`System Profiler`是一个方便客户端攻击的侦察工具，这个工具将会在CS服务端上启动一个Web服务，这样当目标访问这个Web服务的时候，我们就能够看到目标使用的浏览器、操作系统等等指纹信息。

设置系统侦察需要首先在自己的VPS服务器上运行CS服务端，之后本地客户端进行连接，选择`System Profiler`功能模块，配置待跳转的URL等信息即可。

如果勾选了`Use Java Applet to get information`则可以发现目标的Java版本及内网IP地址，但是这样做被发现的风险就会提高，同时现在浏览器已经默认关闭了java执行权限，因此这个选项的作用也变得不大了。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs8-1.png)

配置完后，当用户打开配置后的链接，我们可以在三个地方进行观察

```
1、View --> Applications
2、View --> Web Log
3、Cobalt Strike --> Visualization --> Target Table
```

目标用户打开链接时，我们在CS上就能够看到目标使用的浏览器版本、系统版本等信息了，知道了版本信息，就能够进一步知道目标上可能存在什么漏洞。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs8-2.png)

值得注意的一点是如果 Cobalt Strike 的 web 服务器收到了lynx、wget 或 curl 的请求，CS会自动返回一个 404 页面，这样做是为了防御蓝队的窥探。

# 0x03 用户驱动攻击

用户驱动攻击`User-Driven Attacks`需要欺骗用户产生交互才行，但也有许多的优点。

首先用户驱动攻击不包含恶意攻击代码，所以用户系统上的安全补丁是没用的；其次无论目标使用什么版本的程序，我们都可以创建相应的功能来执行；最后因为用户驱动攻击十分可靠，也使得它很完美。

当我们采取行动来追踪并需要攻击时，它就像用户本地执行程序一样，CS为我们提供了几个用户驱动攻击的选项，分别如下：

## 用户驱动攻击包

用户驱动攻击包`User-Driven Attacks Packages`功能打开位置：`Attacks --> Packages`

### 1、HTML应用

HTML应用`HTML Application`生成(executable/VBA/powershell)这3种原理不同的VBScript实现的`evil.hta`文件。

### 2、Microsoft Office 宏文件

Microsoft Office 宏文件`Microsoft Office Document Macros`可以生成恶意宏放入office文件，非常经典的攻击手法。

### 3、Payload 生成器

Payload生成器`Payload Generator`可以生成各种语言版本的Payload，便于进行免杀。

### 4、Windows 可执行文件

Windows 可执行文件`Windows Executable` 会生成一个Windows可执行文件或DLL文件。默认x86，勾选x64表示包含x64 payload stage生成了artifactX64.exe(17kb) artifactX64.dll(17kb)

### 5、Windows 可执行文件（Stageless）

Windows 可执行文件（Stageless）`Windows Executable (Stageless)`会生成一个无进程的Windows可执行文件或DLL文件。其中的 Stageless 表示把包含payload在内的"全功能"被控端都放入生成的可执行文件beconX64.exe(313kb) beconX64.dll(313kb) becon.ps1(351kb)

## 用户驱动的Web交付攻击

用户驱动Web交付攻击`User-Driven Web Drive-by Attacks`功能打开位置：`Attacks --> Web Drive-by`

### 1、java 签名 applet 攻击

java 签名 applet 攻击`Java Signed Applet Attack`会启动一个Web服务以提供自签名Java Applet的运行环境，浏览器会要求用户授予applet运行权限，如果用户同意则实现控制，但目前该攻击方法已过时。

### 2、Java 智能 Applet 攻击

Java 智能 Applet 攻击`Java Smart Applet Attack`会自动检测Java版本并利用已知的漏洞绕过安全沙箱，但CS官方称该攻击的实现已过时，在现在的环境中无效。

### 3、脚本化 Web 交付

脚本化 Web 交付`Scripted Web Delivery` 为payload提供web服务以便于下载和执行，类似于MSF的Script Web Delivery

### 4、托管文件

托管文件`Host File`通过`Attacks --> Web Drive-by --> Host File`进行配置，攻击者可以通过这个功能将文件上传到CS服务端上，从而进行文件托管。

如果想删除上传到CS服务端上的文件，可以到`Attacks --> Web Drive-by --> Manage`下进行删除。

如果想查看谁访问了这些文件，可以到`View --> Web Log`下进行查看。

# 0x04 HTML 应用攻击演示

首先来到`Attacks --> Packages --> HTML Application`创建一个HTML应用，如果没有创建监听的话，还需要创建一个监听。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs8-3.png)

HTML应用文件生成好后，来到`Attacks --> Web Drive-by --> Host File`，选择刚才生成的文件，最后点击Launch，复制CS创建的链接，在目标主机上打开此链接。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs8-4.png)

当在目标主机上提示是否运行时，点击运行。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs8-5.png)

当该文件在目标上运行后，CS客户端上就可以看到回连的会话了。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs8-6.png)

> 参考链接：
>
> [https://xz.aliyun.com/t/3975](https://xz.aliyun.com/t/3975)
>
> [https://www.bilibili.com/video/BV16b411i7n5](https://www.bilibili.com/video/BV16b411i7n5)
>
> [https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf](https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)