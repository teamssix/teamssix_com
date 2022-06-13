---
title: 【Python Scrapy 爬虫框架】 2、利用 Scrapy 爬取我的博客文章标题链接
date: 2019-12-24 09:33:19
id: 191224-093319
tags:
- Python
- 学习笔记
- Scrapy
categories:
- Python 学习笔记
---
# 0x00 新建项目
在终端中即可直接新建项目，这里我创建一个名称为 teamssix 的项目，命令如下：

```
scrapy startproject teamssix
```
命令运行后，会自动在当前目录下生成许多文件，如下所示：

```
teamssix
    │  scrapy.cfg  #scrapy的配置文件
    └─teamssix  #项目的Python模块，在这里写自己的代码
        │  items.py  #项目定义文件
        │  middlewares.py  #项目中间件文件
        │  pipelines.py  #项目管道文件，用来处理数据的写入存储等操作
        │  settings.py  #项目设置文件
        │  __init__.py
        ├─spiders  #在这里写爬虫代码
        └─ __init__.py
```
<!--more-->
接下来使用 Pycharm 打开我们刚才新建的项目。

# 0x01 创建一个爬虫
首先，在 spiders 文件下 new 一个 python file，这里我新建了一个名为 teamssix_blog_spider 的 py 文件。

在新建的文件中写入自己的代码，这里我写的代码如下：

```Python
import scrapy
class BlogSpider(scrapy.Spider):  #创建 Spider 类
   name = 'blogurl'，  #爬虫名称，必填
   start_urls = ['https://www.teamssix.com']  #待爬取的 url ，必填
   def parse(self,response):  #定义 parse 函数，以解析爬到的东西
      print(response.url)
      print(response.text)
```
# 0x02 运行爬虫
之后运行我们刚新建的 blogurl 项目，运行命令如下：

```
scrapy crawl blogurl
```
之后输出结果如下：

```
2019-12-23 18:33:45 [scrapy.utils.log] INFO: Scrapy 1.8.0 started (bot: teamssix)
2019-12-23 18:33:45 [scrapy.utils.log] INFO: Versions: lxml 4.2.5.0, libxml2 2.9.8, cssselect 1.1.0, parsel 1.5.2, w3lib 1.21.0, Twisted 19.10.0, Python 3.7e'
……省略……
https://www.teamssix.com
<!DOCTYPE html><html lang="zh-CN"><head><meta name="generator" content="Hexo 3.8.0"><meta charset="UTF-8"><meta http-equiv="X-UA-Compatible" content="IE=edge"><meta name="vi                                      
tent><meta name="keywords" content><meta name="author" content="Teams Six,undefined"><meta name="copyright" content="Teams Six"><title>【Teams Six】</title><link rel="styles                                      
css"><link rel="icon" href="/favicon.ico"><!-- script(src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML")--><script src="/js/mathjax/mathjax
    tex2jax: {inlineMath: [['$', '$'], ['\\(', '\\)']]}
});
……省略……
```
不难看出，我们想要的内容已经被打印出来了，但这还远远不够，我们还需要对其进行简单的解析，这里就用到了 BeautifulSoup ，有过爬虫经验的对这个库应该是不陌生了。

# 0x03 爬取内容解析
接下来，想要获取到每个文章的链接，只需要对 parse 的内容进行修改，修改也很简单，基本之前写的多线程里的代码一致。

```Python
def parse(self,response):
   soup = BeautifulSoup(response.text,'html.parser')
   for i in soup.select('.post-title'):
      print('https://www.teamssix.com{}'.format(i['href']))
```
很简单的一个小爬虫，然后将爬虫运行一下

```
~# scrapy crawl blogurl  #运行命令
2019-12-23 19:02:01 [scrapy.utils.log] INFO: Scrapy 1.8.0 started (bot: teamssix)
……省略……
https://www.teamssix.com/year/191222-192227.html
https://www.teamssix.com/year/191220-161745.html
……省略……
2019-12-23 19:02:04 [scrapy.core.engine] INFO: Spider closed (finished)
```

此时就能够将我们想要的东西爬下来了，但这实现的功能还是比较简单，接下来将介绍如何使用 Scrapy 爬取每个子页面中的详细信息。

>更多信息欢迎关注我的个人微信公众号：TeamsSix
>原文链接：[https://www.temassix.com/year/191224-093319.html](https://www.temassix.com/year/191224-093319.html)

>参考链接：
>[https://youtu.be/aDwAmj3VWH4](https://youtu.be/aDwAmj3VWH4)
>[http://doc.scrapy.org/en/latest/intro/tutorial.html](http://doc.scrapy.org/en/latest/intro/tutorial.html)