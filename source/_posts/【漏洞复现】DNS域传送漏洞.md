---
title: 【漏洞复现】DNS域传送漏洞
date: 2019-12-06 17:29:01
id: 191206-172901
tags:
- 漏洞复现
- DNS
- 域传送
categories:
- 漏洞复现
---
注：本文中使用的域名是不存在DNS域传送漏洞的，本文仅用作技术交流学习用途，严禁将该文内容用于违法行为。
<!--more-->
# 0x00 漏洞描述
DNS: 网域名称系统（英文：Domain Name System，缩写：DNS）是互联网的一项服务。

它作为将域名和IP地址相互映射的一个分布式数据库，能够使人更方便地访问互联网。

DNS使用TCP和UDP端口53，当前，对于每一级域名长度的限制是63个字符，域名总长度则不能超过253个字符。

常用的DNS记录有以下几类：

```
主机记录(A记录)：
A记录是用于名称解析的重要记录，它将特定的主机名映射到对应主机的IP地址上。

IPv6主机记录(AAAA记录)：
与A记录对应，用于将特定的主机名映射到一个主机的IPv6地址。 

别名(CNAME记录)：
CNAME记录用于将某个别名指向到某个A记录上，这样就不需要再为某个新名字另外创建一条新的A记录。

电子邮件交换记录（MX记录)：
记录一个邮件域名对应的IP地址

域名服务器记录 (NS记录)：
记录该域名由哪台域名服务器解析

反向记录(PTR记录):
也即从IP地址到域名的一条记录

TXT记录：
记录域名的相关文本信息
```
DNS服务器分为主服务器，备份服务器，缓存服务器。

备份服务器需要利用“域传送”从主服务器上复制数据，然后更新自身的数据库，以达到数据同步的目的，这样是为了增加冗余，万一主服务器挂了还有备份服务器顶着。

而“域传送”漏洞则是由于dns配置不当，本来只有备份服务器能获得主服务器的数据，由于漏洞导致任意client都能通过“域传送”获得主服务器的数据（zone数据库信息）。

这样，攻击者就能获得某个域的所有记录，甚至整个网络拓扑都暴露无遗，凭借这份网络蓝图，攻击者可以节省很多扫描时间以及信息收集的时间，还提升了准确度等等。

大的互联网厂商通常将内部网络与外部互联网隔离开，一个重要的手段是使用 Private DNS。如果内部 DNS 泄露，将造成极大的安全风险。风险控制不当甚至造成整个内部网络沦陷。

# 0x01 漏洞利用
## 1、Windows下使用nslookup
nslookup命令以两种方式运行：非交互式和交互式。

1、非交互式模式下，查看对应主机域的域名服务器

```
~# nslookup -type=ns teamssix.com

服务器:  ns-gg.online.ny.cn
Address:  118.192.13.5
非权威应答:
teamssix.com    nameserver = clint.ns.cloudflare.com
teamssix.com    nameserver = isla.ns.cloudflare.com
```
2、进入交互模式，指定域名服务器，列出域名信息
```
~# nslookup

默认服务器:  ns-gg.online.ny.cn
Address:  118.192.13.5

> server clint.ns.cloudflare.com

默认服务器:  clint.ns.cloudflare.com
Addresses:  2400:cb00:2049:1::adf5:3b5a
          2606:4700:58::adf5:3b5a
          173.245.59.90

> ls teamssix.com

ls: connect: No such file or directory
*** 无法列出域 teamssix.com: Unspecified error
DNS 服务器拒绝将区域 teamssix.com 传送到你的计算机。如果这不正确，
请检查 IP 地址 2400:cb00:2049:1::adf5:3b5a 的 DNS 服务器上 teamssix.com 的
区域传送安全设置。
```
如果提示无法列出域，那就说明此域名不存在域传送漏洞。
## 2、Kali下使用dig、dnsenum、dnswalk
### a、dig
这里涉及dig 一个重要的命令axfr，axfr 是q-type类型的一种，axfr类型是Authoritative Transfer的缩写，指请求传送某个区域的全部记录。

我们只要欺骗dns服务器发送一个axfr请求过去，如果该dns服务器上存在该漏洞，就会返回所有的解析记录值。

dig的整体利用步骤基本和nslookup一致。

1、查看对应主机域的域名服务器

```
~# dig teamssix.com ns

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> teamssix.com ns
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 16945
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;teamssix.com.                  IN      NS


;; ANSWER SECTION:
teamssix.com.           5       IN      NS      clint.ns.cloudflare.com.
teamssix.com.           5       IN      NS      isla.ns.cloudflare.com.


;; Query time: 6 msec
;; SERVER: 10.18.37.7#54(10.18.37.7)
;; WHEN: Fri Dec 06 03:30:37 EST 2019
;; MSG SIZE  rcvd: 83
```
2、向该域名发送axfr 请求

```
~# dig axfr @clint.ns.cloudflare.com teamssix.com

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> axfr @clint.ns.cloudflare.com teamssix.com
; (3 servers found)
;; global options: +cmd
; Transfer failed.
```

### b、dnsenum
这个工具相较于之前的方法要为简单，一行命令即可

```
~# dnsenum -enum teamssix.com

Smartmatch is experimental at /usr/bin/dnsenum line 698.
Smartmatch is experimental at /usr/bin/dnsenum line 698.
dnsenum VERSION:1.2.4
Warning: can't load Net::Whois::IP module, whois queries disabled.
Warning: can't load WWW::Mechanize module, Google scraping desabled.

-----   teamssix.com   -----

Host's addresses:
__________________
teamssix.com.                            5        IN    A        104.28.22.70
teamssix.com.                            5        IN    A        104.28.23.70

Name Servers:
______________

isla.ns.cloudflare.com.                  5        IN    A        173.245.58.119
clint.ns.cloudflare.com.                 5        IN    A        173.245.59.90
clint.ns.cloudflare.com.                 5        IN    RRSIG             (

Mail (MX) Servers:
___________________

Trying Zone Transfers and getting Bind Versions:
_________________________________________________

Trying Zone Transfer for teamssix.com on isla.ns.cloudflare.com ...
AXFR record query failed: FORMERR

Trying Zone Transfer for teamssix.com on clint.ns.cloudflare.com ...
AXFR record query failed: FORMERR

brute force file not specified, bay.
```
### c、dnswalk
dnswalk的使用同样一条命令，但是注意在域名最后加上一个点

```
~# dnswalk teamssix.com.

Checking teamssix.com.
Getting zone transfer of teamssix.com. from clint.ns.cloudflare.com...failed
FAIL: Zone transfer of teamssix.com. from clint.ns.cloudflare.com failed: FORMERR
Getting zone transfer of teamssix.com. from isla.ns.cloudflare.com...failed
FAIL: Zone transfer of teamssix.com. from isla.ns.cloudflare.com failed: FORMERR
BAD: All zone transfer attempts of teamssix.com. failed!
2 failures, 0 warnings, 1 errors.
```
# 0x02 修复建议
区域传送是 DNS 常用的功能，为保证使用安全，应严格限制允许区域传送的主机，例如一个主 DNS 服务器应该只允许它的备用 DNS 服务器执行区域传送功能。

在相应的 zone、options 中添加 allow-transfer，对执行此操作的服务器进行限制。如：

* 严格限制允许进行区域传送的客户端的 IP
```
allow-transfer ｛1.1.1.1; 2.2.2.2;｝
```

* 设置 TSIG key
```
allow-transfer ｛key "dns1-slave1"; key "dns1-slave2";｝
```
# 0x03 总结
最后感谢前辈们的文章与辛勤奉献，DNS域传送漏洞除了本文中讨论的方法外，也可以使用Python脚本或者万能的nmap，这里就不做讨论啦。

想写这篇文章已经想一周了，今天终于有时间给整理整理，另外前文发布的Pigat工具，最近也会修复一些bug提交到Github更新哒。

>更多信息欢迎关注我的个人公众号：TeamsSix

>参考文章：
>http://sunu11.com/2017/03/16/8/
>https://www.jianshu.com/p/d2af08e6f8fb
>http://www.lijiejie.com/dns-zone-transfer-1/
>https://www.alibabacloud.com/help/zh/faq-detail/37529.htm
>https://www.lsablog.com/networksec/awd/dns-zone-transfer/
>https://larry.ngrep.me/2015/09/02/DNS-zone-transfer-studying/