---
title: 【CS学习笔记】23、malleable命令
date: 2020-04-19 15:07:04
id: 200419-150704
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs23-1.png
summary: 这节课将来看看如何优化 CS 的攻击载荷，从而使它更方便、隐蔽些。
tags:
- Cobalt Strike
- 学习笔记
- 红队
categories:
- CS 学习笔记
---

# 0x00 前言

这节课将来看看如何优化 CS 的攻击载荷，从而使它更方便、隐蔽些。

# 0x01 malleable 命令和控制

malleable 是一种针对特定领域的语言，主要用来控制 Cobalt Strike Beacon

在开启 teamserver 时，在其命令后指定配置文件即可调用，比如：

```
./teamserver [ip address] [password] [profile]
```

# 0x02 编写配置文件

## 1、定义事务指标

```
http-get {
	# 指标
}
http-post {
	# 指标
}
```

## 2、控制客户端和服务端指标

```
http-get {
	client {
		# 指标
	}
	server {
		# 指标
	}
}
```

## 3、set  操作

set 语句是给一个选项赋值的方法，以分号结束。

```
set useragent "Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 5.1)";
```

malleable 给了我们很多选项，比如：

```
jitter		# 控制 beacon 默认回连的抖动因子
maxdns		# 控制最大 DNS 请求，限制最大数量可以使 DNS Beacon 发送数据看起来正常些
sleeptime	# 控制 beacon 的全部睡眠时间
spawnto
uri
useragent	# 控制每次发送请求的 useragent
```

`sleeptime` 和 `jitter` 两个选项是很重要的

## 4、添加任意 headers

```
header "Accept" "text/html,application/xhtml";
header "Referer" "https://www.google.com";
header "Progma" "no-cache";
header "Cache-Control" "no-cache";
```

## 5、其他指标

```
header "header" "value";
parameter "key" "value";
```

## 6、转换/存储数据

```
metadata {
    netbios;
    append "-.jpg";
    uri-append;
}
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs23-1.png)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs23-2.png)

## 7、数据转换语言

```
append "string"
base64
netbios
netbiosu
prepend "string"
```

> 参考链接：
> 
>[https://www.bilibili.com/video/BV16b411i7n5](https://www.bilibili.com/video/BV16b411i7n5)
> 
>[https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf](https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf)
> 
>更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)