---
title: 【Python Scrapy 爬虫框架】 6、继续爬虫、终止和重启任务
date: 2019-12-26 15:17:07
id: 191226-151707
tags:
- Python
- 学习笔记
- Scrapy
categories:
- Python 学习笔记
---
# 0x00 前言
有时候我们不想只爬一个页面的，比如之前我只爬了主页，但是现在想把其他页面的也爬下来，这就是本文的任务。

# 0x01 修改代码
在之前的基础上，修改 teamssix_blog_spider.py 文件，首先添加 start_urls

```python
start_urls = [
   'https://www.teamssix.com',
   'https://www.teamssix.com/page/2/',
   'https://www.teamssix.com/page/3/',
   'https://www.teamssix.com/page/4/',
   'https://www.teamssix.com/page/5/'
]
```
<!--more-->
接下来在 sub_article 函数尾部添加 parse 函数的全部代码

```python
soup = BeautifulSoup(response.text, 'html.parser')
for i in soup.select('.post-title'):
   url = 'https://www.teamssix.com{}'.format(i['href'])
   yield scrapy.Request(url, callback=self.sub_article)
```
所以 sub_article 函数的完整代码就是这个样子：

```python
def sub_article(self,response):
   soup = BeautifulSoup(response.text,'html.parser')
   title = self.article_title(soup)
   list = self.article_list(soup)
   print(title)
   item = TeamssixItem(_id = response.url,title = title,list = list)
   yield item

   soup = BeautifulSoup(response.text, 'html.parser')
   for i in soup.select('.post-title'):
      url = 'https://www.teamssix.com{}'.format(i['href'])
      yield scrapy.Request(url, callback=self.sub_article)
```
从最后一行 callback=self.sub_article 这里不难看出这里其实就是一个循环， sub_article 函数第一遍执行完，又会调用继续执行第二遍，直到 start_urls 被执行完。

# 0x02 运行
代码修改的就这些，接下来直接 scrapy crawl blogurl 运行代码，来到 robo 3T 看看爬取到的数据。

![图片](https://uploader.shimo.im/f/4C5P0BNBAy8DfwVP.png!thumbnail)

最终在这些 start_urls 中爬取下来了 43 篇文章，Emm，还行。

这次的 Scrapy 学习笔记就更新到这里，这个项目的代码已经放在了我的 GitHub 里，项目链接已经放在了下面。

>更多信息欢迎关注我的个人微信公众号：TeamsSix
>项目地址：[https://github.com/teamssix/scrapy_study_notes](https://github.com/teamssix/scrapy_study_notes)

>参考链接：
>[https://youtu.be/aDwAmj3VWH4](https://youtu.be/aDwAmj3VWH4)
>[http://doc.scrapy.org/en/latest/topics/architecture.html](http://doc.scrapy.org/en/latest/topics/architecture.html)
