---
title: 【Python Threading 学习笔记】4、Queue功能
date: 2019-11-03 09:22:39
id: 191103-092239
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

# 0x00 关于Queue
queue模块实现了各种【多生产者-多消费者】队列，可用于在执行的多个线程之间安全的交换信息。
<!--more-->
**queue的常用方法：**

```
q.size()：返回队列的正确大小。因为其他线程可能正在更新此队列，所以此方法的返回数字不可靠。

q.empty()：如果队列为空，返回True，否则返回False。

q.full()：如果队列已满，返回True，否则返回False。

q.put(item,block,timeout)：将item放入队列。
如果block设为True（默认值），调用者将被阻塞直到队列中出现可用的空闲位置为止。
如果block设为False，队列满时此方法将引发Full异常。

q.put_nowait(item):等价于q.put(item,False)

q.get(block,timeout):从队列中删除一项，然后返回这个项。
如果block设为True（默认值），调用者将阻塞，直到队列中出现可用的空闲为止。
如果block设为False，队列为空时将引发Empty异常。
timeout提供可选的超时值，单位为秒，如果超时，将引发Empty异常。

q.get_nowait()：等价于get(0)

q.task_done():在队列中数据的消费者用来指示对于项的处理已经结束。如果使用此方法，那么从队列中删除的每一项都应该调用一次。

q.join()：阻塞直到队列中的所有项均被删除和处理为止。一旦为队列中的每一项都调用了一次q.task_done()方法，此方法将会直接返回。
```
# 0x01 本节代码实现功能
将数据列表中的数据传入，使用三个线程处理，将结果保存在Queue中，线程执行完后，从Queue中获取存储的结果。

# 0x02导入线程,队列的标准模块
```python
import time
import threading
from queue import Queue
```
# 0x03 定义一个被多线程调用的函数
该函数的参数是一个列表lists和一个队列q，其功能是对lists列表中的元素除2取整，之后利用q.put()将结果保存在队列q中。

```python
def job(lists,q): # 被调用函数
    for i in range(len(lists)):
        lists[i] = lists[i]//2 # lists元素除2取整
    q.put(lists) # 多线程调用的函数不能用return返回值
```
# 0x04 定义一个多线程函数
在多线程函数中定义一个Queue用来保存返回值代替return，同时定义一个多线程列表，初始化一个多维数据列表用来传入上面的job()函数。

```python
def multithreading(): # 调用多线程的函数
    q = Queue() # 存放job()函数的返回值
    thread_list = []
    data = [[1],[2,3],[4,5,6]]
```
定义三个线程，启动线程并分别join三个线程到主线程
```python
for i in range(3): # 定义三个线程
    t = threading.Thread(target=job,args=(data[i],q))
    t.start()
    thread_list.append(t) # 将线程添加到thread_list列表中 
for thread in thread_list:
    thread.join()
```
定义一个空的result_list列表，将队列q中的数据添加到列表中并print
```python
result_list = []
for j in range(3): # 循环三次
    result_list.append(q.get())
    print(result_list[j])
```
# 0x05 完整的代码
```python
import time
import threading
from queue import Queue


def job(lists,q): # 被调用函数
    for i in range(len(lists)):
        lists[i] = lists[i]//2 # lists元素除2取整
    q.put(lists) # 多线程调用的函数不能用return返回值


def multithreading(): # 调用多线程的函数
    q = Queue() # 存放job()函数的返回值
    thread_list = []
    data = [[1],[2,3],[4,5,6]]

    for i in range(3): # 定义三个线程
        t = threading.Thread(target=job,args=(data[i],q))
        t.start()
        thread_list.append(t) # 将线程添加到thread_list列表中
    for thread in thread_list:
        thread.join()

    result_list = []
    for j in range(3): # 循环三次
        result_list.append(q.get())
        print(result_list[j])


if __name__ == '__main__':
    multithreading()
```
运行结果：
```bash
# python 4_queue.py
[0]
[1, 1]
[2, 2, 3]
```
>代码项目地址：[https://github.com/teamssix/Python-Threading-study-notes](https://github.com/teamssix/Python-Threading-study-notes)
>参考文章：
>1、[https://segmentfault.com/a/1190000016330288](https://segmentfault.com/a/1190000016330288)
>2、[https://morvanzhou.github.io/tutorials/python-basic/threading](https://morvanzhou.github.io/tutorials/python-basic/threading)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)