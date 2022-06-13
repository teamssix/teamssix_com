---
title: 【工具分享】Pigat v2.0正式发布
date: 2020-03-21 15:06:43
id: 200321-150643
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2020-03-21_14-29-32.png
tags:
- Pigat
- 工具分享
- 软件
categories:
- 工具分享
---

## 前言

Pigat（Passive Intelligence Gathering Aggregation Tool）被动信息收集聚合工具，该工具通过爬取目标URL在第三方网站比如备案查询网站、子域名查询网站的结果来对目标进行被动信息收集。

开发此工具的初衷就是平时在使用一些第三方的网站进行目标信息收集的时候，往往需要利用多个网站进行目标信息的收集，比如通过在线的Whois信息查询网站、在线的CMS信息查询网站、目标在Shodan上的信息等等，如此一来，就会增加很多重复的工作量，因此便开发了此工具，将平时常用的网站聚合到一个工具里。

该工具在2020年3月21日更新至2.0版本，该版本采用Scrapy框架开发，协程处理，运行速度更快，并且支持文件导出功能，同时修复了多个Bug，增加了多个功能。

<!--more-->

## 功能描述

* 被动收集备案信息
* 被动收集 Whois 信息
* 被动收集子域名信息
* 被动收集子域名 IP 信息
* 被动收集子域名 CMS 信息
* 被动收集子域名 Shodan 信息
* 被动收集子域名漏洞信息

## 使用方法

### 1、安装MongoDB

这一步可自行百度，如果不是在本地主机安装，则需要在安装MongoDB之后修改`setting.py`文件的第72行，将数据库的URL及端口改为自己设置的即可，默认配置如下：

```python
MONGO_DB_URI = 'mongodb://localhost:27017'
```

### 2、安装第三方模块文件

```bash
pip3 install -r requirements.txt
```

### 3、开始运行

```bash
python3 start.py -u target.com
```

## 帮助

```bash
-u ：指定要被动收集的url
-o : 导出被动收集结果，支持 CSV、JSON、XML 等格式，结果将保存到 ./output 目录下
-a : 直接被动收集所有结果，没有步骤提示信息，并默认导出 result.csv 文件
-h : 查看帮助
```

## 示例

### python3 start.py -u teamssix.com

```bash
- - - - - - - - - - - - - - - - - - - - - - - - - - -
 操作编号：

 [1] 被动收集备案信息
 [2] 被动收集 Whois 信息
 [3] 被动收集子域名信息
 [4] 被动收集子域名 IP 信息
 [5] 被动收集子域名 CMS 信息
 [6] 被动收集子域名 Shodan 信息
 [7] 被动收集子域名漏洞信息
 [8] 退出程序
 备注：第 3 步执行一次后才可执行 4、5、6 步，第 6 步执行一次后才可执行第 7 步，否则运行会出现错误提示。
- - - - - - - - - - - - - - - - - - - - - - - - - - -

[2020-03-21 13:36:35] 请输入你的操作步骤 :
```

### python3 start.py -u teamssix.com -a

```bash
[2020-03-21 13:45:50] 正在被动收集 freebuf.com 的备案信息……
[2020-03-21 13:45:50] 备案网站名称：黑客与极客
[2020-03-21 13:45:50] 备案单位类型：企业
[2020-03-21 13:45:50] 备案单位名称：上海斗象信息科技有限公司
[2020-03-21 13:45:50] 备案网站首页：www.freebuf.com
[2020-03-21 13:45:50] 备案许可证号：沪ICP备13033796号-1
[2020-03-21 13:45:50] 备案审核时间：2019/7/17

[2020-03-21 13:45:56] 正在被动收集 freebuf.com 的 Whois 信息……
[2020-03-21 13:45:56] 注册商：Alibaba Cloud Computing (Beijing) Co., Ltd.
[2020-03-21 13:45:56] 注册邮箱：DomainAbuse@service.aliyun.com
[2020-03-21 13:45:56] 注册电话：+86.95187
[2020-03-21 13:45:56] 注册网址：http://www.net.cn
[2020-03-21 13:45:56] 注册商 Whois 服务器：grs-whois.hichina.com
[2020-03-21 13:45:56] DNS 解析服务器：['F1G1NS1.DNSPOD.NET', 'F1G1NS2.DNSPOD.NET']
[2020-03-21 13:45:56] 注册日期：2010-08-21T15:24:37Z
[2020-03-21 13:45:56] 到期日期：2021-08-21T15:24:37Z
[2020-03-21 13:45:56] 更新日期：2020-02-27T02:03:14Z

[2020-03-21 13:46:04] 正在被动收集 freebuf.com 的子域信息……
[2020-03-21 13:46:04] 搜索"freebuf.com"查询结果为:29条。响应时间：5.204秒，搜索结果共 2 页
[2020-03-21 13:46:04] zhuanlan.freebuf.com      专栏 - FreeBuf 互联网安全新媒体平台 | 关注黑客与极客    200
[2020-03-21 13:46:04] you3enter.freebuf.com     空
[2020-03-21 13:46:04] www.freebuf.com   FreeBuf互联网安全新媒体平台     200
……此处内容太多，省略……
[2020-03-21 13:46:06] ai.freebuf.com    空
[2020-03-21 13:46:06] freebuf.com       FreeBuf互联网安全新媒体平台     200

[2020-03-21 13:46:09] 正在被动收集 freebuf.com 的子域 IP 信息……
[2020-03-21 13:46:09] 提示：如果出现大量无法判断 IP 的情况，请尝试更换自己的IP
[2020-03-21 13:46:09] freebuf.com       wittest.freebuf.com             123.57.248.123
[2020-03-21 13:46:09] freebuf.com       www.freebuf.com         123.57.145.69
……此处内容太多，省略……
[2020-03-21 13:46:12] freebuf.com       ai.freebuf.com          无法判断 IP
[2020-03-21 13:46:14] freebuf.com       static.freebuf.com              39.96.250.248

[2020-03-21 13:46:17] 正在被动收集 freebuf.com 的子域 CMS 信息……
[2020-03-21 13:46:18] freebuf.com       static.freebuf.com                      Apache 2.2.21
[2020-03-21 13:46:19] freebuf.com       wittest.freebuf.com     WitAwards 2017互联网安全年度评选                源
……此处内容太多，省略……
[2020-03-21 13:46:28] freebuf.com       bar.freebuf.com FreeBuf小酒馆           Apache 2.2.21
[2020-03-21 13:46:29] freebuf.com       company.freebuf.com     FreeBuf.COM | 企业空间          jQuery 2.0.3    Twitter Bootstrap       Tengine PHP

[2020-03-21 13:47:23] 正在被动收集 freebuf.com 的子域 shodan 信息……
[2020-03-21 13:47:23] 如果您的 api 输入错误，可到数据库中修改
[2020-03-21 13:47:26] freebuf.com       123.57.145.69   ['www.freebuf.com', 'search.freebuf.com']       China   Aliyun Computing Co.    None    [80]

[2020-03-21 13:47:26] freebuf.com       182.254.150.199 ['freebuf.com', 'freebuf.com']  China   Tencent cloud computing None    [443]   ['CVE-2016-6291', 'CVE-2016-6290', 'CVE-2016-6292', 'CVE-2016-4473', 'CVE-2016-6294', 'CVE-2016-6297',
 'CVE-2016-5767', 'CVE-2016-5768', 'CVE-2016-5769', 'CVE-2018-10547', 'CVE-2018-10546', 'CVE-2019-9641', 'CVE-2016-6295', 'CVE-2018-10548', 'CVE-2018-19520', 'CVE-2018-19396', 'CVE-2016-7478', 'CVE-2016-5766', 'CVE-2018-19935', 'CVE-2016-
6296', 'CVE-2018-17082', 'CVE-2018-10545', 'CVE-2019-9639', 'CVE-2019-9638', 'CVE-2019-9637', 'CVE-2015-8994', 'CVE-2018-14883', 'CVE-2016-5773', 'CVE-2016-5772', 'CVE-2016-5771', 'CVE-2016-5770', 'CVE-2016-6289', 'CVE-2018-19395', 'CVE-2
019-6977', 'CVE-2018-20783', 'CVE-2016-7568', 'CVE-2018-19518', 'CVE-2016-5399', 'CVE-2019-9023', 'CVE-2019-9020', 'CVE-2019-9021', 'CVE-2017-16642', 'CVE-2019-9024', 'CVE-2018-15132', 'CVE-2018-10549']

[2020-03-21 13:47:46] freebuf.com       123.57.248.123  ['wittest.freebuf.com', 'wittest.freebuf.com']  China   Aliyun Computing Co.    None    [80, 443, 2222] ['CVE-2012-0021', 'CVE-2017-15906', 'CVE-2011-4317', 'CVE-2017-7679', 'CVE-201
6-0777', 'CVE-2018-1312', 'CVE-2011-3368', 'CVE-2012-3499', 'CVE-2012-4558', 'CVE-2013-1896', 'CVE-2011-5000', 'CVE-2016-8612', 'CVE-2014-1692', 'CVE-2012-4557', 'CVE-2014-0098', 'CVE-2017-7668', 'CVE-2012-0814', 'CVE-2013-6438', 'CVE-201
2-2687', 'CVE-2011-4415', 'CVE-2012-0031', 'CVE-2010-5107', 'CVE-2013-2249', 'CVE-2016-10708', 'CVE-2011-3607', 'CVE-2017-3167', 'CVE-2011-4327', 'CVE-2012-0053', 'CVE-2012-0883', 'CVE-2017-3169', 'CVE-2014-0231', 'CVE-2013-1862', 'CVE-20
16-4975', 'CVE-2010-4478', 'CVE-2010-4755']

[2020-03-21 13:47:54] freebuf.com       39.96.250.248   ['zhuanlan.freebuf.com', 'shop.freebuf.com', 'wit.freebuf.com', 'my.freebuf.com', 'open.freebuf.com', 'static.freebuf.com', 'live.freebuf.com', 'job.freebuf.com', 'prize.freebuf.com'
, 'www.freebuf.com', 'fit.freebuf.com', 'api.freebuf.com', 'chat.freebuf.com', 'bar.freebuf.com', 'company.freebuf.com', 'wit.freebuf.com', 'prize.freebuf.com', 'zhuanlan.freebuf.com', 'open.freebuf.com', 'live.freebuf.com', 'job.freebuf.
com', 'fit.freebuf.com', 'my.freebuf.com', 'shop.freebuf.com', 'api.freebuf.com', 'chat.freebuf.com', 'company.freebuf.com', 'bar.freebuf.com', 'static.freebuf.com']   China   Aliyun Computing Co.    None    [5000, 6666, 8334, 8080, 8081,
 7443, 8086, 8087, 8090, 10000, 9201, 9000, 8106, 9003, 53, 443, 8000, 8001, 8002, 8009, 81, 82, 83, 7001, 4443, 8800, 7777, 1000, 9200, 6001, 8181, 8443, 9002]

[2020-03-21 13:47:57] 正在被动收集 freebuf.com 的漏洞详情信息……
[2020-03-21 13:47:58] freebuf.com       CVE-2016-6291   高危    PHPext/exif/exif.c文件安全漏洞  182.254.150.199 ['freebuf.com'] http://www.cnnvd.org.cn/web/xxk/ldxqById.tag?CNNVD=CNNVD-201607-929

[2020-03-21 13:47:58] freebuf.com       CVE-2016-6292   中危    PHPext/exif/exif.c文件安全漏洞  182.254.150.199 ['freebuf.com'] http://www.cnnvd.org.cn/web/xxk/ldxqById.tag?CNNVD=CNNVD-201607-930

[2020-03-21 13:47:58] freebuf.com       CVE-2016-4473   高危    PHP安全漏洞     182.254.150.199 ['freebuf.com'] http://www.cnnvd.org.cn/web/xxk/ldxqById.tag?CNNVD=CNNVD-201610-062

[2020-03-21 13:47:58] freebuf.com       CVE-2016-5768   高危    PHPmbstring扩展整数溢出漏洞     182.254.150.199 ['freebuf.com'] http://www.cnnvd.org.cn/web/xxk/ldxqById.tag?CNNVD=CNNVD-201606-559

[2020-03-21 13:47:58] freebuf.com       CVE-2016-6297   中危    PHPext/zip/zip_stream.c文件整数溢出漏洞 182.254.150.199 ['freebuf.com'] http://www.cnnvd.org.cn/web/xxk/ldxqById.tag?CNNVD=CNNVD-201607-935

……此处内容太多，省略……
```

## 运行截图

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2020-03-21_14-29-32.png)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2020-03-21_14-30-07.png)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2020-03-21_14-30-21.png)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2020-03-21_14-30-33.png)

## 下载地址

```
https://github.com/teamssix/pigat/releases
```

[点击此处进入下载页面](https://github.com/teamssix/pigat/releases)

## 写在最后

因为我没有太多的开发经验，因此该工具难免存在有问题以及不恰当的地方，希望各位大佬在使用的过程中碰到问题能够多多反馈。

开发不易，还望大佬们走过路过顺手给个star，小弟将不胜感激。

> 更多信息欢迎关注我的个人微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)