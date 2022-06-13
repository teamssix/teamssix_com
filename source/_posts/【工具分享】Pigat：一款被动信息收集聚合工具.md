---
title: Pigat：一款被动信息收集聚合工具
date: 2019-11-26 21:57:59
id: 191126-215759
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/pigat1.png
tags:
- 被动信息收集
- pigat
- 聚合工具
categories:
- 工具分享
---
# 0x00 前言
Pigat即Passive Intelligence Gathering Aggregation Tool，翻译过来就是被动信息收集聚合工具，既然叫聚合工具，也就是说该工具将多款被动信息收集工具结合在了一起，进而提高了平时信息收集的效率。

早在一个月前便萌生了开发这个工具的想法，但是一直没有时间，正好最近有时间了，就简单写一下。
<!--more-->
因为我没有太多的开发经验，所以这款工具难免存在需要改进的地方，因此希望各位大佬能够多多反馈这款工具存在的问题，一起完善这个工具。

# 0x01 工具原理及功能概述
这款工具的原理很简单，用户输入目标url，再利用爬虫获取相关被动信息收集网站关于该url的信息，最后回显出来。

目前该工具具备8个功能，原该工具具备7个功能，分别为收集目标的资产信息、CMS信息、DNS信息、备案信息、IP地址、子域名信息、whois信息，现加入第8个功能：如果在程序中两次IP查询目标URL的结果一致，那么查询该IP的端口，即端口查询功能。

# 0x02 工具简单上手使用
## 1、查看帮助信息
```
# python pigat.py -h
```
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/pigat1.png)

## 2、指定url进行信息获取
如果只指定url这一个参数，没有指定其他参数，则默认获取该url的所有信息
```
# python pigat.py -u teamssix.com
```
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/pigat2.png)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/pigat3.png)

## 3、指定url进行单项信息获取
```
# python pigat.py -u baidu.com --assert
```
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/pigat4.png)

## 4、指定url进行多项信息获取
```
# python pigat.py -u teamssix.com --ip --cms
```
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/pigat5.png)

# 0x03 工具获取
关于此工具的下载地址可在我的个人公众号（TeamsSix）回复"pigat"获取。

# 0x04 声明
1、本文在FreeBuf首发，原文地址在文章尾部

2、由于我的个人疏忽，导致在FreeBuf文中获取工具的方式存在错误的地方，正确的获取方式应是回复"pigat"，而不是"pigta"，这就导致不少人及时回复了关键词也没有获取到工具地址，在这里表示深刻歉意，现在公众号后台规则已经更新，上述两个关键词均可以获取到工具地址。

>原文地址：[https://www.freebuf.com/sectool/219681.html](https://www.freebuf.com/sectool/219681.html)