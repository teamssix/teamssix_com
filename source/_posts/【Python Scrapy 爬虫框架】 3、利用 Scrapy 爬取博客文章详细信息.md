---
title: 【Python Scrapy 爬虫框架】 3、利用 Scrapy 爬取博客文章详细信息
date: 2019-12-26 15:16:52
id: 191226-151652
tags:
- Python
- 学习笔记
- Scrapy
categories:
- Python 学习笔记
---

# 0x00 写在前面
在之前的文章中，会发现如果直接使用爬取命令，终端会回显很多调试信息，这样输出的内容就会显得很乱，所以就可以使用下面的命令：

```
scrapy crawl blogurl  -s LOG_FILE=all.log
```
<!--more-->
也就是在原来的基础上加上一个 -s 参数，这样调试信息就会保存到参数指定的文件中，不过也可以在 class 下添加下面的代码，这样只会显示调试出现错误的信息，所以这种方式就不用加 -s 了，至于选择哪一个，就需要​视情况而定。​

```
custom_settings = {'LOG_LEVEL':'ERROR'}
```
# 0x01 编写子页面爬取代码
先来看一行关键代码

```python
yield scrapy.Request(url,callback=self.sub_article)
```
上面这行代码中，使用 yield 返回利用 scrapy 请求 url 所获得的数据，并将数据通过 callback 传递到 sub_article 函数中。

其实对于 yield 和 return 都可以返回数据，但是利用 yield 返回数据后，还可以继续运行下面的代码，而使用 return 后，接下来的代码就不会再运行了，在 scrapy 中，如果使用 return 返回数据再用 list 存储数据，会造成不少的内存消耗，而使用 yield 则可以减少这些不必要的内存浪费。

所以接下来在 sub_article 函数中写上我们爬取子页面的代码即可，这里就爬取每个文章的标题和目录作为示例了。

```python
def sub_article(self,response):
   soup = BeautifulSoup(response.text,'html.parser')
   print('\n',soup.select('.title')[0].text)
   for i in soup.select('.toc-text'):
      print('\t',i.text)
```
运行结果如下：

```
~# scrapy crawl blogurl  -s LOG_FILE=all.log
【漏洞笔记】Robots.txt站点文件
         0x00 概述
         0x01 漏洞描述
         0x02 漏洞危害
         0x03 修复建议
【经验总结】常见的HTTP方法
         0x00 概述
         0x01 GET
         0x02 HEAD
         0x03 POST
         0x04 PUT
         0x05 DELETE
         0x06 CONNECT
         0x07 OPTIONS
         0x08 TRACE
         0x09 PATCH
【漏洞笔记】Host头攻击
         0x00 概述
         0x01 漏洞描述
         0x02 漏洞危害
         0x03 修复建议
【直播笔记】白帽子的成长之路
【Python 学习笔记】 异步IO (asyncio) 协程
         0x00 前言
         0x01 基本用法
……省略……
```
# 0x02 完整代码
```python
import scrapy
from bs4 import BeautifulSoup

class BlogSpider(scrapy.Spider):
   name = 'blogurl'
   start_urls = ['https://www.teamssix.com']

   def parse(self,response):
      soup = BeautifulSoup(response.text,'html.parser')
      for i in soup.select('.post-title'):
         url = 'https://www.teamssix.com{}'.format(i['href'])
         yield scrapy.Request(url,callback=self.sub_article)

   def sub_article(self,response):
      soup = BeautifulSoup(response.text,'html.parser')
      title = self.article_title(soup)
      list = self.article_list(soup)
      print(title)
      for i in list:
         print('\t',i)

   def article_title(self,soup):
      return soup.select('.title')[0].text

   def article_list(self,soup):
      list = []
      for i in soup.select('.toc-text'):
         list.append(i.text)
      return list
```
>更多信息欢迎关注我的个人微信公众号：TeamsSix

>参考链接：
>[https://youtu.be/aDwAmj3VWH4](https://youtu.be/aDwAmj3VWH4)
>[https://blog.csdn.net/DEREK_D/article/details/84239813](https://blog.csdn.net/DEREK_D/article/details/84239813)
>[http://doc.scrapy.org/en/latest/topics/architecture.html](http://doc.scrapy.org/en/latest/topics/architecture.html)
