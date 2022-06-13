---
title: 【Python Threading 学习笔记】6、锁lock
date: 2019-11-05 12:10:11
id: 191105-121011
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

[5、不一定有效率GIL](https://www.teamssix.com/year/191104-101112.html)

# 0x00 关于线程锁lock
多线程和多进程最大的不同在于，多进程中，同一个变量，各自有一份拷贝存在于每个进程中，互不影响，而多线程中，所有变量都由所有线程共享，所以，任何一个变量都可以被任何一个线程修改，因此，线程之间共享数据最大的危险在于多个线程同时改一个变量，把内容给改乱了。

而使用lock就可以在不同线程使用同一共享内存时，能够确保线程之间互不影响。
<!--more-->
# 0x01 不使用lock锁的情况
job1：全局变量A的值每次加1，循环7次并打印
```python
def job1(): # 全局变量A的值每次加1，循环7次并打印
   global A
   for i in range(7):
      A += 1
      print('job1',A)
```
job2：全局变量A的值每次加10，循环7次并打印
```python
def job2():# 全局变量A的值每次加10，循环7次并打印
   global A
   for i in range(7):
      A += 10
      print('job2',A)
```
main：定义两个线程并执行job1和job2
```python
def main(): # 定义两个线程并执行job1和job2
   t1 = threading.Thread(target=job1)
   t2 = threading.Thread(target=job2)
   t1.start()
   t2.start()
   t1.join()
   t2.join()
```
完整代码：
```python
import threading


def job1(): # 全局变量A的值每次加1，循环7次并打印
   global A
   for i in range(7):
      A += 1
      print('job1',A)


def job2():# 全局变量A的值每次加10，循环7次并打印
   global A
   for i in range(7):
      A += 10
      print('job2',A)


def main(): # 定义两个线程并执行job1和job2
   t1 = threading.Thread(target=job1)
   t2 = threading.Thread(target=job2)
   t1.start()
   t2.start()
   t1.join()
   t2.join()


if __name__ == '__main__':
   A = 0
   main()
```
运行结果：
```bash
# python 6_lock.py
job1 1
job1 2
job1 3
job1 4
job1 5job2 15
job2 
job1 2625
job2
job1 36 37
job2 
47
job2 57
job2 67
job2 77
```
可以看到不使用lock的时候，打印的结果很混乱。
# 0x02 使用lock的情况
使用lock的方法是， 在每个线程执行运算修改共享内存之前，执行lock.acquire()将共享内存上锁， 确保当前线程执行时，内存不会被其他线程访问，执行运算完毕后，使用lock.release()将锁打开， 保证其他的线程可以使用该共享内存。

为job1和job2加锁：
```python
def job1(): # 全局变量A的值每次加1，循环7次并打印
   global A,lock
   lock.acquire() # 上锁
   for i in range(7):
      A += 1
      print('job1',A)
   lock.release() # 开锁


def job2():# 全局变量A的值每次加10，循环7次并打印
   global A,lock
   lock.acquire() # 上锁
   for i in range(7):
      A += 10
      print('job2',A)
   lock.release() # 开锁
```
在程序入口处定义一个lock
```python
if __name__ == '__main__':
   lock = threading.Lock()
   A = 0
   main()
```
完整代码：
```python
import threading


def job1(): # 全局变量A的值每次加1，循环7次并打印
   global A,lock
   lock.acquire()
   for i in range(7):
      A += 1
      print('job1',A)
   lock.release()


def job2():# 全局变量A的值每次加10，循环7次并打印
   global A,lock
   lock.acquire()
   for i in range(7):
      A += 10
      print('job2',A)
   lock.release()


def main(): # 定义两个线程并执行job1和job2
   t1 = threading.Thread(target=job1)
   t2 = threading.Thread(target=job2)
   t1.start()
   t2.start()
   t1.join()
   t2.join()


if __name__ == '__main__':
   lock = threading.Lock()
   A = 0
   main()
```
运行结果：
```bash
# python 6_lock.py
job1 1
job1 2
job1 3
job1 4
job1 5
job1 6
job1 7
job2 17
job2 27
job2 37
job2 47
job2 57
job2 67
job2 77
```
从运行结果来看，使用lock后，一个线程一个线程的执行完，两个线程之间互不影响。
至此，整个【Python Threading 学习笔记】系列更新完毕。

>代码项目地址：[https://github.com/teamssix/Python-Threading-study-notes](https://github.com/teamssix/Python-Threading-study-notes)
>参考文章：
>1、[https://www.jianshu.com/p/05b6a6f6fdac](https://www.jianshu.com/p/05b6a6f6fdac)
>2、[https://morvanzhou.github.io/tutorials/python-basic/threading](https://morvanzhou.github.io/tutorials/python-basic/threading)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)