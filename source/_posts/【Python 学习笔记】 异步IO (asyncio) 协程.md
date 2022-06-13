---
title: 【Python 学习笔记】 异步IO (asyncio) 协程
date: 2019-12-20 16:17:45
id: 191220-161745
tags:
- Python
- 协程
- 异步IO
categories:
- Python 学习笔记
---
# 0x00 前言

之前对协程早有耳闻，但一直没有去学习，今天就来学习一下协程，再次感谢莫烦的教程。

可以交给asyncio执行的任务被称为协程， asyncio 即异步的意思，在 Python3 中这是一个仅使用单线程就能达到多线程、多进程效果的工具。

在单线程中使用异步发起 IO 操作的时候，不需要等待 IO 的结束，在等待 IO 操作结束的这个空当儿可以继续做其他事情，结束的时候就会得到通知，所以能够很有效的利用等待下载的这段时间。

今天就来看看协程能不能干掉多线程和多进程。
<!--more-->
# 0x01 基本用法

Python 的在 3.4 中引入了协程的概念，3.5 则确定了协程的语法，所以想使用协程处理 IO ，需要Python3.5 及以上的版本，下面是一个简单示例代码。
```Python
    import time
    import asyncio
    
    async def job(t):
        print('开始第', t,'个任务')
        await asyncio.sleep(t)  #等待t秒
        print('第', t, '个任务执行了', t, '秒')
        
    
    async def main(loop):
        tasks = [loop.create_task(job(t)) for t in range(1, 4)]     #创建多个任务
        await asyncio.wait(tasks)    #运行刚才创建的那些任务
    
    if __name__ == '__main__':
        start_time = time.time()
        loop = asyncio.get_event_loop()    #创建事件循环
        loop.run_until_complete(main(loop))    #运行刚才创建的事件循环
        loop.close()
        print("所有总共耗时", time.time() - start_time)
```
运行结果如下：
```
    开始第 1 个任务
    开始第 2 个任务
    开始第 3 个任务
    第 1 个任务执行了 1 秒
    第 2 个任务执行了 2 秒
    第 3 个任务执行了 3 秒
    所有总共耗时 3.0029773712158203
```
这里运行了三个任务，三个任务的执行时间加在一起是6秒，但是最后总共耗时是3秒，接下来就看看协程在爬虫中的使用。

# 0x02 aiohttp的使用

使用 aiohttp 模块可以将 requests 替换成一个异步的 requests ，下面先来看看一般的 requests 的使用，下面的运行结果耗时是我运行了三次，然后取平均数的结果。
```Python
    import time
    import requests
    
    def normal():
        for i in range(3):
            r = requests.get(URL)
    
    if __name__ == '__main__':
    		t1 = time.time()
    		URL = 'https://www.teamssix.com/'
        normal()
        print("正常访问 3 次博客耗费时间", time.time()-t1)
```
运行结果如下：

    正常访问 3 次博客耗费时间 12.872265259424845

正常情况下，花费了近 13 秒，接下来使用 aiohttp 看看耗时多少。
```Python
    import time
    import asyncio
    import aiohttp
    
    async def job(session):
       response = await session.get('https://www.teamssix.com/')       # 等待并切换
       return str(response.url)
    
    async def main(loop):
       async with aiohttp.ClientSession() as session:      # 官网推荐建立 Session 的形式
           tasks = [loop.create_task(job(session)) for _ in range(3)]
           finished, unfinished = await asyncio.wait(tasks)
    if __name__ == '__main__':
    	t1 = time.time()
    	loop = asyncio.get_event_loop()
    	loop.run_until_complete(main(loop))
    	loop.close()
    	print("异步访问 3 次博客耗费时间", time.time() - t1)
```
运行结果如下：
```
    异步访问 3 次博客耗费时间 4.055158615112305
```
从运行结果上来看使用 aiohttp 还是很给力的，接下来，看看多线程运行的时间。
```Python
    import time
    import threading
    import requests
    
    def thread_test():
        r = requests.get(URL)
    
    if __name__ == '__main__':
        t1 = time.time()
        URL = 'https://www.teamssix.com/'
        thread_list = []
        for i in range(3):
            t = threading.Thread(target=thread_test)
            thread_list.append(t)
        for i in thread_list:
            i.start()
        for i in thread_list:
            i.join()
        print("多线程访问 3 次博客耗费时间", time.time()-t1)
```
运行结果如下：
```
    5.449431339899699
```
可以看到 aiohttp 的速度还是要略快于多线程的，这里只是简单介绍了一下 aiohttp ，详细的可以参阅[官方文档](https://docs.python.org/zh-cn/3/library/asyncio.html)，想要使用的熟练还是需要大量练习，任重道远。

> 更多信息欢迎关注我的个人微信公众号：TeamsSix
> 参考文章：
> https://www.jianshu.com/p/b5e347b3a17c
> https://segmentfault.com/a/1190000008814676
> https://www.lylinux.net/article/2019/6/9/57.html
> https://morvanzhou.github.io/tutorials/data-manipulation/scraping/4-02-asyncio/