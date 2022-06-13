---
title: 【Django 学习笔记】5、常用功能
date: 2020-03-04 13:22:09
id: 200304-132209
tags:
- Python
- Django
- 学习笔记
categories:
- Python 学习笔记
---

# 0x00 案例实现

* 创建的项目名称为`BookManager`，创建应用名称为`Book`，完成`图书信息`的维护
* 访问图书信息列表`127.0.0.1:8000/booklist`，并且点击每个图书能够跳转到对应图书人物信息界面

<!--more-->

# 0x01 代码实现

1、修改`templates\booklist.html`代码

```html
原来：<li><h1><a href="#">{{ book.name }}</a></h1></li>
现在：<li><h1><a href="{{ book.id }}/">{{ book.name }}</a></h1></li> #通过id值来进行对应书籍人物信息查询
```

2、在`Book\views.py`中新添`peoplelist`函数

```python
def peoplelist(request, book_id):	#book_id从url中直接传入
   book = BookInfo.objects.get(id=book_id)	#通过id查询书籍
   people_list = book.peopleinfo_set.all()	#获取书籍中的所有人物
   context = {
      'people_list': people_list
   }
   return render(request, 'peoplelist.html', context)
```

3、在`BookManager\urls.py`中新添url规则

```python
re_path(r'^booklist/(\d+)/$', views.peoplelist), #(\d+)里的内容将直接传入peoplelist函数中的book_id参数
```

如果没有导入re_path需要导入一下

```python
from django.urls import re_path
```

4、新建`templates\eoplelist.html`文件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>人物信息</title>
</head>
<body>
<ul>
    {% for people in people_list %}
    <li><h1>{{ people.name }}</h1></li>
    {% endfor%}
</ul>
</body>
</html>
```

5、最后实现功能：

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/12django_note.png.gif)

6、总结MVT流程

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/13django_note.png)

* 本次Django学习笔记更新完毕，本次项目源码已上传至我的github，源码链接：[https://github.com/teamssix/Django_study_notes](https://github.com/teamssix/Django_study_notes)



> 更多信息欢迎关注我的微信公众号：TeamsSix
>
> 参考链接：[https://youtu.be/BXyGr9JQVcc](https://youtu.be/BXyGr9JQVcc)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)