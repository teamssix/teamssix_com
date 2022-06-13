---
title: 【Python Threading 学习笔记】5、不一定有效率GIL
date: 2019-11-04 10:11:12
id: 191104-101112
tags:
- Python
- 多线程
- 学习笔记
categories:
- Python 学习笔记
---
往期内容：

[1、什么是多线程？](https://www.teamssix.com/year/1901031-202253.html)

[2、添加线程](https://www.teamssix.com/year/191101-112015.html)

[3、join功能](https://www.teamssix.com/year/191102-102624.html)

[4、Queue功能](https://www.teamssix.com/year/191103-092239.html)

# 0x00 关于GIL
GIL的全称是Global Interpreter Lock(全局解释器锁)，来源是python设计之初的考虑，为了数据安全所做的决定。
<!--more-->
每个CPU在同一时间只能执行一个线程（在单核CPU下的多线程其实都只是并发，不是并行，并发和并行从宏观上来讲都是同时处理多路请求的概念。但并发和并行又有区别，并行是指两个或者多个事件在同一时刻发生；而并发是指两个或多个事件在同一时间间隔内发生。）

在Python多线程下，每个线程的执行方式如下：

1.获取GIL

2.执行代码直到sleep或者是python虚拟机将其挂起。

3.释放GIL

可见，某个线程想要执行，必须先拿到GIL，我们可以把GIL看作是“通行证”，并且在一个python进程中，GIL只有一个。拿不到通行证的线程，就不允许进入CPU执行。

也就是说尽管Python支持多线程，但是因为GIL的存在，使得Python还是一次性只能处理一个东西，那是不是说Python中的多线程就完全没用了呢，当然不是的。

GIL往往只会影响到那些严重依赖CPU的程序，比如各种循环处理、计数等这种CPU密集型的程序；如果程序中大部分只会涉及到I/O，比如文件处理、网络爬虫等这种IO密集型的程序，那么多线程就能够有效的提高效率，因为在爬虫的时候大部分时间都在等待。

实际上，你完全可以放心的创建几千个Python线程， 现代操作系统运行这么多线程没有任何压力，没啥可担心的。

# 0x01 测试GIL
```python
import copy
import time
import requests
import threading
from queue import Queue

def job(lists,q):
   res = sum(lists)
   q.put(res)


def multithreading(lists):
   q = Queue()
   threads_list = []

   for i in range(4):
      t = threading.Thread(target=job,args=(copy.copy(lists),q),name = '任务 %i' % i)
      t.start()
      threads_list.append(t)
   for t in threads_list:
      t.join()

   total = 0
   for _ in range(4):
      total += q.get()
   print('使用线程运算结果:',total)


def normal(lists):
   total = sum(lists)
   print('不使用线程运算结果:',total)


def req_job(i):
   requests.get(i)


def req_multithreading(req_lists):
   threads_list = []

   for i in range(4):
      t = threading.Thread(target=req_job,args=(req_lists[i],),name='爬虫任务 %i' % i)
      t.start()
      threads_list.append(t)
   for t in threads_list:
      t.join()


def req_normal(req_lists):
   for i in req_lists:
      requests.get(i)


if __name__ == '__main__':
   lists = list(range(1000000)) # 完成一个较大的计算
   req_lists = ['https://www.teamssix.com','https://github.com/teamssix','https://me.csdn.net/qq_37683287','https://space.bilibili.com/148389186']
   start_time = time.time()
   multithreading(lists)
   print('计算使用线程耗时:', time.time() - start_time,'\n')

   start_time = time.time()
   normal(lists * 4)
   print('计算不使用线程耗时:', time.time() - start_time,'\n')
   start_time = time.time()
   req_multithreading(req_lists)
   print('爬虫使用线程耗时:', time.time() - start_time)
   start_time = time.time()
   req_normal(req_lists)
   print('爬虫不使用线程耗时:', time.time() - start_time)
```
运行结果：
```bash
# python 5_GIL.py
使用线程运算结果: 1999998000000
计算使用线程耗时: 0.39594030380249023 

不使用线程运算结果: 1999998000000
计算不使用线程耗时: 0.3919515609741211

爬虫使用线程耗时: 2.2410056591033936
爬虫不使用线程耗时: 7.1159656047821045
```
可以看到在计算程序的代码中不使用线程和使用线程的运算结果是相同的，说明不使用线程和使用线程的程序都进行了一样多次的运算，但是很明显可以看到计算的耗时并没有少很多，按照预期我们使用了4个线程，应该会快近4倍才对，这就是因为GIL在作怪。
与此同时，可以看到在使用request对一个url发起get请求的时候，使用线程比不使用线程快了3倍多，也进一步的反映出在使用Python进行爬虫的时候，多线程确实可以很大程度上提高效率，但是在进行密集计算任务的时候，多线程就显得很鸡肋了。

>代码项目地址：[https://github.com/teamssix/Python-Threading-study-notes](https://github.com/teamssix/Python-Threading-study-notes)
>参考文章：
>1、[https://zhuanlan.zhihu.com/p/20953544](https://zhuanlan.zhihu.com/p/20953544)
>2、[https://morvanzhou.github.io/tutorials/python-basic/threading](https://morvanzhou.github.io/tutorials/python-basic/threading)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)