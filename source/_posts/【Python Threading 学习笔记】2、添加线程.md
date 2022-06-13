---
title: 【Python Threading 学习笔记】2、添加线程
date: 2019-11-01 11:20:15
id: 191101-112015
tags:
- Python
- 多线程
- 学习笔记
categories:
- Python 学习笔记
---
往期内容：
[1、什么是多线程？](https://www.teamssix.com/year/1901031-202253.html)

这一节主要学习Threading模块的一些基本操作，如获取线程数，添加线程等。
<!--more-->
首先导入Threading模块

```python
import threading
```
获取已激活的线程数

```python
threading.active_count()
```
查看所有线程信息

```python
threading.enumerate()
```
查看现在正在运行的线程

```python
threading.current_thread()
```
添加线程，threading.Thread()接收参数target代表这个线程要完成的任务，需自行定义

```python
import threading
def thread_jobs():  # 定义要添加的线程
    print('已激活的线程数： %s' % threading.active_count())
    print('所有线程信息： %s' % threading.enumerate())
    print('正在运行的线程： %s' % threading.current_thread())
def main():
    thread = threading.Thread(target=thread_jobs, )  # 定义线程
    thread.start()  # 开始线程

if __name__ == '__main__':
    main()
```
运行结果：
```bash
# python 2_add_thread.py
已激活的线程数： 2
所有线程信息： [<_MainThread(MainThread, stopped 16800)>, <Thread(Thread-1, started 20512)>]
正在运行的线程 <Thread(Thread-1, started 20512)>
```
>代码项目地址：[https://github.com/teamssix/Python-Threading-study-notes](https://github.com/teamssix/Python-Threading-study-notes)
>参考文章：[https://morvanzhou.github.io/tutorials/python-basic/threading](https://morvanzhou.github.io/tutorials/python-basic/threading)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)