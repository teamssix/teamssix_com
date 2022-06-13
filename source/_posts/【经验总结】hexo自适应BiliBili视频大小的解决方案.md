---

title: hexo自适应BiliBili视频大小的解决方案
date: 2019-06-14 11:15:12
id: 190614-111512
tags:
- BiliBili
- 自适应视频大小
- 解决方案
categories:
- 经验总结
---

# 0x00 未修改
<!--more-->
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/bilibili1.png)
```html
<iframe src="//player.bilibili.com/player.html?aid=55224540&cid=96981660&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>
```
直接将B站中的视频插入地址放入文章MarkDown中的效果如下：
<iframe src="//player.bilibili.com/player.html?aid=55224540&cid=96981660&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>  

这个视频也太小了，而且不能全屏，很难受，于是在网上看到有人用自定义调节视频的高宽。
# 0x01 自定义高宽
```html
<iframe src="//player.bilibili.com/player.html?aid=55224540&cid=96981660&page=1" width="780" height="480" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>
```
就是在原来的基础上加入了宽高，效果类似这样：
<iframe src="//player.bilibili.com/player.html?aid=55224540&cid=96981660&page=1" width="780" height="480" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>  


自定义调节视频高宽虽然可以解决这个问题，但是手机上又是另一翻景象，你看：  

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/bilibili2.png)

这就更难受了，于是在网上找遍各种自适应视频大小的方案，最终找到以下这种方案。
# 0x02 自适应视频大小
```html
<div style="position: relative; width: 100%; height: 0; padding-bottom: 75%;"><iframe src="//player.bilibili.com/player.html?aid=55224540&cid=96981660&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" style="position: absolute; width: 100%; height: 100%; left: 0; top: 0;"> </iframe></div>
```
只需要把你的视频地址和上面的视频地址替换一下就行，最后效果如下：
<div style="position: relative; width: 100%; height: 0; padding-bottom: 75%;"><iframe src="//player.bilibili.com/player.html?aid=55224540&cid=96981660&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" style="position: absolute; width: 100%; height: 100%; left: 0; top: 0;"> </iframe></div>  

手机上也能很好的自适应：   

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/bilibili3.png)

不过发现依然不能全屏播放，还是有些美中不足，所以只好也将视频播放链接放到视频下面了，如果不能全屏就只能去源链接去看了，如果你有更好的解决方法，欢迎下方留言。
# 0x03 致谢
本文中的自适应解决方案来自这篇文章：[文章地址](https://www.andyvj.com/2019/02/12/190213-01/)，里面还介绍了其他音视频平台插入的方法，在这里也谢谢这位老哥了。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)