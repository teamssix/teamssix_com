---
title: 【Python Threading 学习笔记】3、join功能
date: 2019-11-02 10:26:24
id: 191102-102624
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

# 0x00 不使用join()的结果
首先在上一节的示例基础上进行简单修改
<!--more-->
```python
import time
import threading

def thread_jobs():  # 定义要添加的线程
    print('任务1开始\n')
    for i in range(10):
        time.sleep(0.1)
    print('任务1结束\n')
    
def main():
    thread = threading.Thread(target=thread_jobs,name='任务1')  # 定义线程
    thread.start()  # 开始线程
    print('所有任务已完成\n')

if __name__ == '__main__':
    main()
```
预计输出结果：
```bash
# python 3_join.py
任务1开始
任务1结束
所有任务已完成
```
实际输出结果：
```bash
# python 3_join.py
任务1开始
所有任务已完成
任务1结束
```
可以看到在线程还没有结束的时候，程序就开始运行之后的代码了，也就是说线程和其他部分的程序都是同步进行的，如果想要避免这种情况，想要程序按照代码顺序执行的话，就需要用到join功能。
# 0x01 使用join()的结果
在源代码thread.start()下加入thread.join()即可，原来代码的main函数就变成这样：

```python
def main():
    thread = threading.Thread(target=thread_jobs,name='任务1')  # 定义线程
    thread.start()  # 开始线程
    thread.join() #加入join功能
    print('所有任务已完成\n')
```
这里就表示必须要等到任务1这个线程结束后，才能执行thread.join()之后的代码，代码运行结果如下：
```bash
# python 3_join.py
任务1开始
任务1结束
所有任务已完成
```
使用join控制多个线程的执行顺序很关键。
# 0x02 添加第2个线程
这时如果我们加入第2个线程，但是不加入join功能，看看又是什么样，第2个线程的任务量较小，因此比第1个线程会更快执行，加入的第2个线程如下：

```python
def thread_jobs2(): # 定义第2个线程
    print('任务2开始\n')
    print('任务2结束\n')
```
输出的一种结果：
```bash
# python 3_join.py
任务1开始
任务2开始
任务2结束
所有任务已完成
任务1结束
```
注意这个时候任务1和任务2都没有添加join，也就是说输出的内容是什么完全看谁执行的快，谁先执行完谁就先输出，因此这里的输出结果并不唯一，这种杂乱的输出方式是不能接收的，所以需要使用join来控制。
# 0x03 在不同位置使用join()的结果
如果在任务2开始前只对任务1加入join功能：

```python
thread.start()  # 开始线程1
thread.join()  # 对任务1加入join功能
thread2.start() # 开始线程2
print('所有任务已完成\n')
```
其输出结果如下：
```bash
# python 3_join.py
任务1开始
任务1结束
任务2开始
任务2结束
所有任务已完成
```
可以看到程序会先执行任务1再执行接下来的操作，如果在任务2开始后只对任务1加入join功能：
```python
thread.start()  # 开始线程1
thread2.start() # 开始线程2
thread.join()  # 对任务1加入join功能
print('所有任务已完成\n')
```
其输出结果如下：
```bash
# python 3_join.py
任务1开始
任务2开始
任务2结束
任务1结束
所有任务已完成
```
任务1先于任务2启动，但由于任务2的处理时间较短，因此先于任务1完成，而由于任务1加入了join，因此“所有任务已完成”也在任务1完成后再显示。
# 0x04 最终代码及输出结果
如果只对任务2加入join，同样输出结果就是要先等任务2执行完再执行其他程序，但是为了避免不必要的麻烦，推荐下面这种1221的V型排布，毕竟如果每个线程start()后就加入其join()，那就和单线程无异了。

```python
thread.start()  # 开始线程1
thread2.start() # 开始线程2
thread2.join()  # 对任务2加入join功能
thread.join()  # 对任务1加入join功能
```
最终完整代码及输出结果如下：
```python
import time
import threading

def thread_jobs():  # 定义第1个线程
    print('任务1开始\n')
    for i in range(10):
        time.sleep(0.1)
    print('任务1结束\n')

def thread_jobs2(): # 定义第2个线程
    print('任务2开始\n')
    print('任务2结束\n')

def main():
    thread = threading.Thread(target=thread_jobs,name='任务1')  # 定义线程1
    thread2 = threading.Thread(target=thread_jobs2, name='任务2')  # 定义线程2
    thread.start()  # 开始线程1
    thread2.start() # 开始线程2
    thread2.join()  # 对任务2加入join功能
    thread.join()  # 对任务1加入join功能
    print('所有任务已完成\n')

if __name__ == '__main__':
    main()
```
输出结果：
```bash
# python 3_join.py
任务1开始
任务2开始
任务2结束
任务1结束
所有任务已完成
```
>参考文章：[https://morvanzhou.github.io/tutorials/python-basic/threading](https://morvanzhou.github.io/tutorials/python-basic/threading)
>代码项目地址：[https://github.com/teamssix/Python-Threading-study-notes](https://github.com/teamssix/Python-Threading-study-notes)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)