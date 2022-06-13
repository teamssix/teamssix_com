---
title: 【工具分享】一个爬取各大SRC厂家公告的小工具
date: 2020-11-15 19:48:50
id: 201115-194850
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2020-11-15_18-35-17.png
summary: 一个爬取各大SRC厂家公告的小工具，没啥技术含量，都是体力活。
tags:
- 批量工具
- Python
- SRC
categories:
- 工具分享
---

# 前言

平时在挖洞的时候，往往会关注其有没有活动之类的公告，但是国内那么多家厂商，一个个的去关注难免效率比较低，因此这里写了一个爬取各大SRC厂家公告的小工具，没啥技术含量，都是体力活。

工具下载地址：[https://github.com/teamssix/src_notice](https://github.com/teamssix/src_notice)

# 介绍

✔ 爬取各个SRC平台的公告通知

✔ 对2020年发布的活动通知进行红色高亮显示

✔ 【进阶版】将当天发布的公告推送到微信上，结合系统定时任务可实现SRC平台公告监测

> **在我的微信公众号“TeamsSix”后台回复“SRC”即可获得进阶版下载地址**

✔ 支持的SRC平台【当前共计27家】：

360、58、阿里、阿里本地生活、爱奇艺、百度、贝壳、哔哩哔哩、菜鸟、滴滴出行、度小满、瓜子、京东、蚂蚁金服、美团、陌陌、OPPO、平安、水滴互助、顺丰、腾讯、vivo、网易、唯品会、WIFI万能钥匙、中通、字节跳动

# 安装

```
git clone https://github.com/teamssix/src_notice.git
cd src_notice
pip3 install -r requirements.txt
python3 src_notice.py
```

# 使用

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2020-11-15_18-35-17.png)

# 进阶版

进阶版比普通版多了一个微信通知的功能，结合系统定时任务可实现SRC平台公告监测。**在我的微信公众号“TeamsSix”后台回复“SRC”即可获得进阶版下载地址**

### 如何实现微信通知？

进阶版下载好后，在 503 行代码中添加上你自己的 Server酱 key 就行了， Server酱 key 的申请地址为：[http://sc.ftqq.com/](http://sc.ftqq.com/)

### 如何实现公告监测？

首先在 vps 上下载安装该工具，之后设置定时任务即可。比如我想在每天的上午 9 点获取一下各大 SRC 有没有新的公告：

1、输入`crontab -e`

2、在打开的界面中输入`00 9 * * * python3 /root/src_notice/src_notice.py`即可。

### 注意

1、如果之前没有编辑过定时任务，第一次打开会提示选择编辑器，根据自己喜好选择即可

2、上面命令中的 `python3 /root/src_notice/src_notice.py`要修改成你自己的绝对路径，我这里是在 root 目录下的

# 关于

> 更多信息欢迎关注我的个人微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)