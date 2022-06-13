---
title: 【CS学习笔记】7、SMBbean的作用
date: 2020-04-19 15:05:10
id: 200419-150510
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs7-4.png
summary: SMB Beacon 使用命名管道通过一个父 Beacon 进行通信。这种对等通信对同一台主机上的 Beacon 和跨网络的 Beacon 都有效。Windows 将命名管道通信封装在 SMB 协议中。因此得名 SMB Beacon。
tags:
- Cobalt Strike
- 学习笔记
- 红队
categories:
- CS 学习笔记
---

# 0x00 SMB Beacon 简介

SMB Beacon 使用命名管道通过一个父 Beacon 进行通信。这种对等通信对同一台主机上的 Beacon 和跨网络的 Beacon 都有效。Windows 将命名管道通信封装在 SMB 协议中。因此得名 SMB Beacon。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs7-1.png)

因为链接的Beacons使用Windows命名管道进行通信，此流量封装在SMB协议中，所以SMB Beacon相对隐蔽，绕防火墙时可能发挥奇效(系统防火墙默认是允许445的端口与外界通信的，其他端口可能会弹窗提醒，会导致远程命令行反弹shell失败)。

SMB Beacon监听器对“提升权限”和“横向渗透”中很有用。

# 0x01 SMB Beacon 配置

首先需要一个上线的主机，这里我使用的HTTP Beacon，具体如何上线，可以参考之前第5节《如何建立Payload处理器》学习笔记中的内容，这里不过多赘述。

主机上线后，新建一个SMB Beacon，输入监听器名称，选择Beacon SMB，管道名称可以直接默认，也可以自定义。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs7-2.png)

接下来在Beacon中直接输入`spawn SMB`，这里的`SMB`指代的是创建的SMB Beacon的监听器名称，也可以直接右击session，在Spawn选项中选择刚添加的SMB Beacon。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs7-3.png)

等待一会儿，就可以看到派生的SMB Beacon，在external中可以看到IP后有个`∞∞`字符。

接下来我这里将SMB Beacon插入到进程中，以vmtoolsed进程为例。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs7-4.png)

在vmtoolsed中插入SMB Beacon后，便能看到process为vmtoolsed.exe的派生SMB Beacon。

当上线主机较多的时候，只靠列表的方式去展现，就显得不太直观了，通过CS客户端中的透视图便能很好的展现。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs7-5.png)

在CS中，如果获取到目标的管理员权限，在用户名后会有`*`号标注，通过这个区别，可以判断出当前上线的test用户为普通权限用户，因此这里给他提升一下权限。

# 0x02 其他的一些操作

## 1、提权

> 由于下面与上面内容的笔记不是在同一天写的，因此截图中上线的主机会有所差异，这里主要是记录使用的方法。

由于CS自带的提权方式较少，因此这里就先加载一些网上的提权脚本，脚本下载地址为：[https://github.com/rsmudge/ElevateKit](https://github.com/rsmudge/ElevateKit)

下载之后，打开`Cobalt Strike --> Script Manager` ，之后点击`Load`，选择自己刚才下载的文件中的`elevate.cna`文件。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs7-6.png)

接着选择要提权的主机，右击选择`Access --> Elevate`，Listener中选择刚才新建的SMB Beacon，这里的Exploit选择了ms14-058，如果使用ms14-058不能提权，就换一个Exploit进行尝试。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs7-7.png)

顺利的情况下，就可以看到提权后的管理员权限会话了，在管理员权限的会话中，不光用户名后有个*号，其Logo也是和其他会话不同的。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs7-8.png)

## 2、连接与断开

此时如果想断开某个会话的连接，可以使用unlink命令，比如如果想断开192.168.175.144，就可以在Beacon中输入`unlink 192.168.175.144`

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs7-9.png)

如果想再次连上，就直接输入`link 192.168.175.144`，想从当前主机连到其他主机也可以使用此命令。

# 0x03 攻击载荷安全特性

1、在Beacon传输Payload到目标上执行任务时都会先验证团队服务器，以确保Beacon只接受并只运行来自其团队服务器的任务，并且结果也只能发送到其团队服务器。

2、在刚开始设置Beacon Payload时，CS会生成一个团队服务器专有的公私钥对，这个公钥嵌入在Beacon的Payload Stage中。Beacon使用团队服务器的公钥来加密传输的元数据，这个元数据中一般包含传输的进程ID、目标系统IP地址、目标主机名称等信息，这也意味着只有团队服务器才能解密这个元数据。

3、当Beacon从团队服务器下载任务或团队服务器接收Beacon输出时，团队服务器将会使用Beacon生成的会话秘钥来加密任务并解密输出。

4、值得注意的是，Payload Stagers 因为其体积很小，所以没有这些的安全特性。

> 参考链接：
>
> [https://www.bilibili.com/video/BV16b411i7n5](https://www.bilibili.com/video/BV16b411i7n5)
>
> [https://pythonpig.github.io/2018/01/17/Cobaltstrike-SMB-beacon/](https://pythonpig.github.io/2018/01/17/Cobaltstrike-SMB-beacon/)
>
> [https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf](https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)