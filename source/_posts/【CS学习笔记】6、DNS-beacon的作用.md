---
title: 【CS学习笔记】6、DNS_beacon的作用
date: 2020-04-19 15:05:02
id: 200419-150502
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs6-4.png
summary: DNS Beacon，顾名思义就是使用DNS请求将Beacon返回。这些 DNS 请求用于解析由你的 CS 团队服务器作为权威 DNS 服务器的域名。
tags:
- Cobalt Strike
- 学习笔记
- 红队
categories:
- CS 学习笔记
---

由于笔者在学习CS过程中，所看的教程使用的是3.x版本的CS，而我使用的是4.0版本的CS。因此域名配置实操部分是自己参考网上大量文章后自己多次尝试后的结果，所以难免出现错误之处，要是表哥发现文中错误的地方，欢迎留言指正。

# 0x00 DNS Beacon 的工作原理

DNS Beacon，顾名思义就是使用DNS请求将Beacon返回。这些 DNS 请求用于解析由你的 CS 团队服务器作为权威 DNS 服务器的域名。DNS 响应告诉 Beacon 休眠或是连接到团队服务器来下载任务。DNS 响应也告诉 Beacon 如何从你的团队服务器下载任务。

在CS 4.0及之后的版本中，DNS Beacon是一个仅DNS的Payload，在这个Payload中没有HTTP通信模式，这是与之前不同的地方。

> 以上内容摘自 A-TEAM 团队的 CS 4.0 用户手册

DNS Beacon的工作流程具体如下：

首先，CS服务器向目标发起攻击，将DNS Beacon传输器嵌入到目标主机内存中，然后在目标主机上的DNS Beacon传输器回连下载CS服务器上的DNS Beacon传输体，当DNS Beacon在内存中启动后就开始回连CS服务器，然后执行来自CS服务器的各种任务请求。

原本DNS Beacon可以使用两种方式进行传输，一种是使用HTTP来下载Payload，一种是使用DNS TXT记录来下载Payload，不过现在4.0版本中，已经没有了HTTP方式，CS4.0以及未来版本都只有DNS TXT记录这一种选择了，所以接下来重点学习使用DNS TXT记录的方式。

根据作者的介绍，DNS Beacon拥有更高的隐蔽性，但是速度相对于HTTP Beacon什么的会更慢。

# 0x01 域名配置

既然是配置域名，所以就需要先有个域名，这里就用我的博客域名作为示例：添加一条A记录指向CS服务器的公网IP，再添加几条ns记录指向A记录域名即可。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs6-1.png)

添加一个监听器，DNS Hosts填写NS记录和A记录对应的名称，DNS Host填写A记录对应的名称

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs6-2.png)

根据上一章的方法创建一个攻击脚本，放到目标主机中运行后，在CS客户端可以看到一个小黑框

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs6-3.png)

然后经过漫长的等待，就可以发现已经上线了

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs6-4.png)

> 参考链接：
>
> [https://www.bilibili.com/video/BV16b411i7n5](https://www.bilibili.com/video/BV16b411i7n5)
>
> [https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf](https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)