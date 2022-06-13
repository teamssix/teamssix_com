---
title: 【漏洞笔记】Host头攻击
date: 2019-11-27 20:14:43
id: 191127-201443
tags:
- 漏洞笔记
- Host头
- 管理员设置问题
categories:
- 漏洞笔记
---
# 0x00 概述
漏洞名称：Host头攻击

风险等级：低

问题类型：管理员设置问题

# 0x01 漏洞描述
Host首部字段是HTTP/1.1新增的，旨在告诉服务器，客户端请求的主机名和端口号，主要用来实现虚拟主机技术。
<!--more-->
运用虚拟主机技术，单个主机可以运行多个站点。

例如：hacker和usagidesign两个站点都运行在同一服务器A上，不管我们请求哪个域名，最终都会被解析成服务器A的IP地址，这个时候服务器就不知道该将请求交给哪个站点处理，因此需要Host字段指定请求的主机名。

我们访问hacker域名，经DNS解析，变成了服务器A的IP，请求传达到服务器A，A接收到请求后，发现请求报文中的Host字段值为hacker，进而将请求交给hacker站点处理。

这个时候，问题就出现了。为了方便获取网站域名，开发人员一般依赖于请求包中的Host首部字段。例如，在php里用_SERVER["HTTP_HOST"]，但是这个Host字段值是不可信赖的(可通过HTTP代理工具篡改)。

# 0x02 漏洞危害
如果应用程序没有对Host字段值进行处理，就有可能造成恶意代码的传入。

# 0x03 修复建议
对Host字段进行检测

Nginx，修改ngnix.conf文件，在server中指定一个server_name名单，并添加检测。

Apache，修改httpd.conf文件，指定ServerName，并开启UseCanonicalName选项。

Tomcat，修改server.xml文件，配置Host的name属性。

>更多信息欢迎关注我的个人微信公众号：TeamsSix

>参考文章：
>[https://www.jianshu.com/p/690acbf9f321](https://www.jianshu.com/p/690acbf9f321)