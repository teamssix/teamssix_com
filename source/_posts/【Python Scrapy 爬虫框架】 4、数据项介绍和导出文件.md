---
title: 【Python Scrapy 爬虫框架】 4、数据项介绍和导出文件
date: 2019-12-26 15:16:59
id: 191226-151659
tags:
- Python
- 学习笔记
- Scrapy
categories:
- Python 学习笔记
---
# 0x00 前言
通过上文的内容，已经把博客文章的标题及目录爬取下来了，接下来为了方便数据的保存，我们可以把这些文章的标题及目录给包装成一个数据项，也就是 items。

# 0x01 配置 item
先来到 items.py 文件下，对标题及目录的信息进行包装，为了对这些信息进行区别，还需要有一个 id，所以代码如下：

```python
class TeamssixItem(scrapy.Item):
    _id = scrapy.Field()
    title = scrapy.Field()
    list = scrapy.Field()
```
<!--more-->
编辑好 items.py 文件后，来到 teamssix_blog_spider.py 先把刚才编辑的内容引用进来。

```python
from teamssix.items import TeamssixItem
```
接着创建一个 item ，并抛出 item ，这时这个 item 就会进入到 item pipelines 中处理。

```python
item = TeamssixItem(_id = response.url,title = title,list = list)
yield item
```
# 0x02 运行
程序中包含 item 的好处就在于可以直接把运行结果输出到文件中，直接 -o 指定导出文件名，scrapy 支持导出 json 、jsonlines 、jl 、csv 、xml 、marshal 、pickle 这几种格式。

```
scrapy crawl blogurl -o result.json
```
另外如果发现导出文件乱码，只需要在 settings.py 文件中添加下面一行代码即可。

```python
FEED_EXPORT_ENCODING = "gb18030"
```
运行结果如下：

```
~# scrapy crawl blogurl -o result.json
~# cat result2.json
[
{"_id": "https://www.teamssix.com/year/191224-093319.html", "title": "【Python Scrapy 爬虫框架】 2、利用 Scrapy 爬取我的博客文章标题链接", "list": ["0x00 新建项目", "0x01 创建一个爬虫", "0x02
 运行爬虫", "0x03 爬取内容解析"]},
{"_id": "https://www.teamssix.com/year/191127-201447.html", "title": "【漏洞笔记】Robots.txt站点文件", "list": ["0x00 概述", "0x01 漏洞描述", "0x02 漏洞危害", "0x03 修复建议"]},
……省略……
```
可以很明显的感受到使用 scrapy 可以很方便的将数据导出到文件中，下一篇文章将介绍如何导出到 MongoDB数据库中。


>更多信息欢迎关注我的个人微信公众号：TeamsSix

>参考链接：
>[https://youtu.be/aDwAmj3VWH4](https://youtu.be/aDwAmj3VWH4)
>[http://doc.scrapy.org/en/latest/topics/architecture.html](http://doc.scrapy.org/en/latest/topics/architecture.html)
