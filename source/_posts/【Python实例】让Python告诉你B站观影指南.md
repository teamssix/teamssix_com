---
title: 【Python实例】让Python告诉你B站观影指南
date: 2019-06-19 20:27:02
id: 190619-202702
tags:
- Python
- 电影
- 爬虫
categories:
- 实例
---

# 0x00 前言
hello大家好，这里是TeamsSix，昨天晚上11点多的突然想在B站看电影了，但是又不知道那个电影值得看，于是首先想到的是去各大影评UP主的频道里面看看，转了几圈后发现他们讲解的电影B站很多都没有，这个时候又想到了一种方法，就是在B站搜索：“在B站值得看的电影”，没想到以前还真有UP主统计过：
<!--more-->
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/bilibili_movie1.png)
点进去之后发现UP主居然手动统计了160多部电影，最后他做成了一个表格，看完了之后立刻给了三连，因为在我看来每一位付出了努力与汗水的UP主都值得被尊重（疯狂暗示），事后就想到能不能用我这三脚猫的Python水平统计一下B站最值得看的电影呢？
有了想法，立刻从床上爬了起来，在夜黑风高的晚上开始垒起了代码，终于经历了一个通宵的时间之后完成了这个想法。
# 0x01 代码运行
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/bilibili_movie2.png)
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/bilibili_movie3.png)
>具体代码见文章底部链接
# 0x02 运行结果
通过刚才的运行结果，可以看到，播放数量最高的是《你的名字》，足足有一千八百多万的播放量
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/bilibili_movie4.png)
弹幕数量最高的还是《你的名字》，有高达98万条弹幕
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/bilibili_movie5.png)
硬币数量最多的依然是《你的名字》，硬币数量达到了39万个
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/bilibili_movie6.png)
追剧人数最高的《命运之夜--天之杯：恶兆之花》
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/bilibili_movie7.png)
B站评分最高的不详，因为评分最高9.9的视频比较多，所以B站评分没有统计到视频最后的汇总里
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/bilibili_movie8.png)
在B站的电影中豆瓣评分最高的是《武林外传》，高达9.5分
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/bilibili_movie9.png)
B站评分与豆瓣评分差最大的是《深夜食堂》，两个平台差了7分，这部电影我也看过，表示还阔以，不明白为什么豆瓣评分那么低。
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/bilibili_movie10.png)
ok，最后我们再来总结一下，B站的电影中播放数量、弹幕数量、硬币数量最高的均为《你的名字》，可以说是B站电影区当之无愧的C位，其余《刀剑神域：序列之争》《声之形》《白蛇：缘起》也都经常出现在前三之中。
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/bilibili_movie11.png)
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/bilibili_movie12.png)

# 0x03 结语
在视频的最后再简单说两句，这些数据都是可以导出为表格的，另外在写代码的中间有个小插曲，就是在获取豆瓣搜索结果中电影评分的时候，发现电影数据都是被加密的，就像这个样子
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/bilibili_movie13.png)
最后通过SergioJune在Github上提供的代码得以解决，在这里也向他表示感谢
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/bilibili_movie14.png)

>文章代码：https://github.com/teamssix/bilibili-movie

演示视频：
<div style="position: relative; width: 100%; height: 0; padding-bottom: 75%;"><iframe src="//player.bilibili.com/player.html?aid=56117996&cid=98086981&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" style="position: absolute; width: 100%; height: 100%; left: 0; top: 0;"> </iframe></div>

如果视频不能全屏播放，请点击[源链接](https://www.bilibili.com/video/av56117996/ "源链接")观看。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)