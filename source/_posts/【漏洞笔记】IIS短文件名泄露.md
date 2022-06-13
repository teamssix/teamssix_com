---
title: 【漏洞笔记】IIS短文件名泄露
date: 2019-11-26 21:58:04
id: 191126-215804
tags:
- 漏洞笔记
- IIS
- 信息泄露
categories:
- 漏洞笔记
---
# 0x00 概述
漏洞名称：IIS短文件名泄露

风险等级：低

问题类型：信息泄露

# 0x01 漏洞描述
此漏洞实际是由HTTP请求中旧DOS 8.3名称约定（SFN）的代字符（〜）波浪号引起的。
<!--more-->
为了兼容16位MS-DOS程序，Windows为文件名较长的文件（和文件夹）生成了对应的windows 8.3 短文件名。

Microsoft IIS 波浪号造成的信息泄露是世界网络范围内最常见的中等风险漏洞。这个问题至少从1990年开始就已经存在，但是已经证明难以发现，难以解决或容易被完全忽略。

**受影响的版本：**
IIS 1.0，Windows NT 3.51 
IIS 3.0，Windows NT 4.0 Service Pack 2 
IIS 4.0，Windows NT 4.0选项包
IIS 5.0，Windows 2000 
IIS 5.1，Windows XP Professional和Windows XP Media Center Edition 
IIS 6.0，Windows Server 2003和Windows XP Professional x64 Edition 
IIS 7.0，Windows Server 2008和Windows Vista 
IIS 7.5，Windows 7（远程启用<customErrors>或没有web.config）
IIS 7.5，Windows 2008（经典管道模式）
注意：IIS使用.Net Framework 4时不受影响

**漏洞的局限性：**
1) 只能猜解前六位，以及扩展名的前3位。
2) 名称较短的文件是没有相应的短文件名的。
3）需要IIS和.net两个条件都满足。

# 0x02 漏洞危害
**主要危害：利用“~”字符猜解暴露短文件/文件夹名**

由于短文件名的长度固定（xxxxxx~xxxx），因此黑客可直接对短文件名进行暴力破解 ，从而访问对应的文件。

举个例子，有一个数据库备份文件 backup_www.abc.com_20150101.sql ，它对应的短文件名是 backup~1.sql 。因此黑客只要暴力破解出backup~1.sql即可下载该文件，而无需破解完整的文件名。

**次要危害：.Net Framework的拒绝服务攻击 **

攻击者如果在文件夹名称中发送一个不合法的.Net文件请求，.NeFramework将递归搜索所有的根目录，消耗网站资源进而导致DOS问题。

# 0x03 修复建议
1、CMD关闭NTFS 8.3文件格式的支持

2、修改注册表禁用短文件名功能

3、关闭Web服务扩展- ASP.NET

4、升级netFramework至4.0以上版本

>更多信息欢迎关注我的个人微信公众号：TeamsSix

>参考文章：
>[https://www.freebuf.com/articles/web/172561.html](https://www.freebuf.com/articles/web/172561.html)
>[https://segmentfault.com/a/1190000006225568](https://segmentfault.com/a/1190000006225568)