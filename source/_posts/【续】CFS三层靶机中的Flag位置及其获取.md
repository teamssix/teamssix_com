---
title: 【续】CFS三层靶机中的Flag位置及其获取
date: 2019-10-21 21:38:28
id: 191021-213828
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/flag13.png
tags:
- 内网渗透
- CFS
- 比赛
categories:
- 实例演示
---
# 0x00 前言
最近写了一篇《CFS三层靶机搭建及其内网渗透》的文章，里面满满的干货，本篇文章需要结合《CFS三层靶机搭建及其内网渗透》一起看，这篇文章可以在本文底部找到阅读链接。
<!--more-->
# 0x01 Target1
1、系统根目录下

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/flag1.png)

2、网站根目录下

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/flag2.png)

3、网站robots.txt文件中

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/flag3.png)

# 0x02 Target2
1、系统根目录下

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/flag4.png)

2、日志文件中

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/flag5.png)

3、passwd文件中

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/flag6.png)

4、crontab文件中

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/flag7.png)

5、网站根目录下

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/flag8.png)

6、管理后台中

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/flag9.png)

# 0x03 Target3
1、通过meterpreter上传Everything工具并安装

```bash
meterpreter > upload ./Everything_64.exe C:
```
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/flag10.png)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/flag11.png)

2、利用Everything，直接搜索flag文件

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/flag12.png)

3、找到两处flag，继续找寻发现计划任务中存在第三处flag

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/flag13.png)

4、最后一处在事件日志的注册表中被找到

```
HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Eventlog
```
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/flag14.png)

>《CFS三层靶机搭建及其内网渗透》文章地址：
>1、首发地址：https://www.anquanke.com/post/id/187908
>2、博客地址：https://www.teamssix.com/year/191021-211425.html

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)