---
title: 【Python 学习笔记】多进程爬虫
date: 2019-12-20 16:15:33
id: 191220-161533
tags:
- Python
- 多进程
- 分布式
categories:
- Python 学习笔记
---
# 0x00 前言

前段时间学习了多线程，但在实际的情况中对于多线程的速度实在不满意，所以今天就来学学多进程分布式爬虫，在这里感谢莫烦的Python教程。

# 0x01 什么是多进程爬虫

在讲述多进程之前，先来回顾一下之前学习的多线程。
<!--more-->
对于多线程可以简单的理解成运输快递的货车，虽然在整个运输快递的途中有很多货车参与运输，但快递到你手中的时间并不会因为货车的数量增加而变化多少，甚至可能会因为参与运输的货车数量过多，导致送货时间变慢，因为货物在不断的上货卸货。
当然现实中可不会有人这么干，然而在计算机的世界里，有时却会犯这种错误，这也就是说多线程并不是越多越好。

如果有操作系统的基础，则对于线程与进程的理解会更深刻些，这里继续参照上面的例子，对于线程可以简单的理解成一个线程就是一个货车，而一个进程则是一整条快递运输线路上的货车集合，也就是说一个进程包含了多个线程。

如果在只有一个快递需要运输的时候，使用线程与进程的区别或许不大，但是如果有十件快递、百件快递，使用多进程无疑能够极大的提高效率。

# 0x02 准备工作

在开始学习多进程之前，先来理一下爬虫思路，这里拿爬取我的博客文章举例，首先先用 requests 访问 temassix.com，之后利用 BeautifulSoup 解析出我博客中的文章链接，接着再利用 requests 访问文章，便完成了一个简单的爬虫。

接下来需要用到的模块：
```Python
    import time    #测试爬取时间
    import requests
    import threading
    from multiprocessing import Process   #多进程模块
    from bs4 import BeautifulSoup
```
接下来需要用到的一些子函数：
```Python
    def req_url(url):
        r = requests.get(url)    #访问url
        return(r.text)    #返回html
    
    def soup_url(html):
    	url_list = []
    	soup = BeautifulSoup(html, 'html.parser')    #解析返回的html
    	for i in soup.select('.post-title'):
    		url_list.append('https://www.teamssix.com{}'.format(i['href']))    #拼接博客文章的url
    	return (url_list)    #返回博客文章url数组
```
# 0x03 测试普通爬取方法

这里先使用普通爬取的方法，也就是单线程测试一下，为了方便，下面提到的单线程处理方法，准确的来说是单进程单线程，同样的，下面提到的多进程准确的说法是多进程单线程，多线程准确的说则是单进程多线程。

值得注意的是爬取耗时根据自己的网络情况而定，即使碰到多进程耗时几百秒而单线程耗时几十秒也是正常的，这种情况是因为网络环境较差造成的，所以碰到结果出入很大的时候，可以多试几次，排除偶然性，下面就来上代码。
```Python
    if __name__ == '__main__':
    	# 开始单线程
    	start_time = time.time()
    	url = 'https://www.teamssix.com'
    	html = req_url(url)
    	home_page = soup_url(html)
    	for i in home_page:
    		req_url(i)
    	end_time = time.time()
    	print('\n单线程：',end_time - start_time)
```
最终运行结果如下：
```
    单线程： 29.181440114974976
```
单线程花费了 29 秒的时间，接下来使用多进程测试一下

# 0x04 测试多进程爬取方法

通过学习发现多进程的用法和多线程还是挺相似的，所以就直接放代码吧，感兴趣的可以看看参考文章。
```Python
    if __name__ == '__main__':
    	# 开始多进程
    	start_time = time.time()
    	url = 'https://www.teamssix.com'
    	pool = Pool(4)
    	home_page = soup_url(req_url(url))
    	for i in home_page:
    		pool.apply_async(req_url, args=(i,))
    	pool.close()
    	pool.join()
    	end_time = time.time()
    	print('\n多进程：',end_time - start_time)
```
最终运行结果如下：
```
    多进程： 12.674117088317871
```
多进程仅用了 12 秒就完成了任务，经过多次测试，发现使用多进程基本上能比单线程快2倍以上。

为了看到多线程与多进程的差距，这里使用多线程处理了一下上面的操作，代码如下：
```Python
    if __name__ == '__main__':
    	#开始多线程
    	start_time = time.time()
    	url = 'https://www.teamssix.com'
    	thread_list = []
    	home_page = soup_url(req_url(url))
    	for i in home_page:
    		t = threading.Thread(target = req_url, args=(i,))
    		thread_list.append(t)
    	for i in thread_list:
    		i.start()
    	for i in thread_list:
    		i.join()
    	end_time = time.time()
    	print('\n多线程：', end_time - start_time)
```
最终运行结果如下：
```
    多线程： 11.685778141021729
```
看到这里可能会觉着，这多线程和多进程爬虫的时间也差不多呀，然而事实并非那么简单。

由于爬虫的大多数时间都耗在了请求等待响应中，所以在爬虫的时候使用多线程好像快了不少，但我以前写过一个笔记：[不一定有效率GIL](https://www.teamssix.com/year/191104-101112.html)
在这篇文章里演示了如果使用单线程和多线程处理密集计算任务，有时多线程反而会比单线程慢了不少，所以接下来就看看多进程处理密集计算任务的表现。

# 0x05 处理密集计算任务耗时对比

直接上代码：
```Python
    import time  # 测试爬取时间
    import threading
    from multiprocessing import Pool
    
    def math(i):
    	result2 = 2 ** i    #执行幂运算
    
    
    if __name__ == '__main__':
    	#开始单线程
    	start_time = time.time()
    	for i in range(0, 1000000001, 250000000):
    		math(i)
    	end_time = time.time()
    	print('\n单线程：',end_time - start_time)
    
    	#开始多进程
    	start_time = time.time()
    	pool = Pool(4)
    	for i in range(0, 1000000001, 250000000):
    		pool.apply_async(math, args=(i,))
    	pool.close()
    	pool.join()
    	end_time = time.time()
    	print('\n多进程：',end_time - start_time)
    
    	# 开始多线程
    	start_time = time.time()
    	thread_list = []
    	for i in range(0, 1000000001, 250000000):
    		t = threading.Thread(target = math, args=(i,))
    		thread_list.append(t)
    	for i in thread_list:
    		i.start()
    	for i in thread_list:
    		i.join()
    	end_time = time.time()
    	print('\n多线程：', end_time - start_time)
```
最终运行结果如下：
```
    单线程： 20.495169162750244
    
    多进程： 11.645867347717285
    
    多线程： 22.07299304008484
```
通过运行结果可以很明显看出，单线程与多线程的耗时差距不大，但是多进程的耗时与之相比几乎快了一倍，所以平时为了提高效率是使用多线程还是多进程，也就很清楚了。

但如果平时想提高爬虫效率是用多线程还是多进程呢？毕竟他们效率都差不多，那么协程了解一下🧐

> 更多信息欢迎关注我的个人微信公众号：TeamsSix
> 参考文章：
> https://morvanzhou.github.io/tutorials/data-manipulation/scraping/4-01-distributed-scraping/
> https://www.liaoxuefeng.com/wiki/1016959663602400/1017628290184064