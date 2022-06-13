---
title: 【Django 学习笔记】2、模型
date: 2020-02-29 22:34:31
id: 200229-223431
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/8django_note.png
tags:
- Python
- Django
- 学习笔记
categories:
- Python 学习笔记
---

# 0x00 模型

* 当前项目的开发, 都是数据驱动的。

* 使用Django进行数据库开发的提示 ：
  * `MVT`设计模式中的`Model`, 专门负责和数据库交互.对应`(models.py)`
  * 由于`Model`中内嵌了`ORM框架`, 所以不需要直接面向数据库编程.
  * 而是定义模型类, 通过`模型类和对象`完成数据库表的`增删改查`.
  * `ORM框架`就是把数据库表的行与相应的对象建立关联, 互相转换.使得数据库的操作面向对象.

* 使用Django进行数据库开发的步骤 ：

  1. 定义模型类
  2. 模型迁移
  3. 操作数据库

<!--more-->

## 1、定义模型类

在这之前需要先设计数据库的表什么的，这里就不详细的说了（主要是我太懒了，手动狗头），感兴趣的可以看本文的参考链接，下面直接贴定义模型类的代码.


```Python
# BookManager/Book/models.py
from django.db import models

class BookInfo(models.Model):  # 定义数据信息类模型
	name = models.CharField(max_length=10)  # 设计name属性
	
class PeopleInfo(models.Model):  # 定义人物信息类模型
	name = models.CharField(max_length=10)
	gender = models.BooleanField()
	book = models.ForeignKey(BookInfo)
```

## 2、模型迁移

由两步完成，首先生成迁移文件，根据模型类生成创建表的语句；接下来执行迁移，根据第一步生成的语句在数据库中创建表。分别由以下两句完成。

```bash
python3 manage.py makemigrations
python3 manage.py migrate
```

运行结果：

```bash
BookManager/ > python3 manage.py makemigrations
Traceback (most recent call last):
  File "manage.py", line 21, in <module>
    main()
………内容太多，此处省略………
TypeError: __init__() missing 1 required positional argument: 'on_delete'
```

在运行第一个命令的时候报错了，此时只需要修改定义外键的那行代码即可。

```python
# 原来的
book = models.ForeignKey(BookInfo)
#修改后
book = models.ForeignKey(BookInfo,on_delete=models.CASCADE)
```

发生这个错误的原因是由于我看的教程使用的是1.8版本的Django，而我安装的是3.0，Django在2.0版本后，如果定义外键就需要加上`on_delete`选项了，OK，接下来，继续运行这两个代码。

```bash
BookManager/ > python3 manage.py makemigrations
Migrations for 'Book':
  Book\migrations\0001_initial.py
    - Create model BookInfo
    - Create model PeopleInfo

BookManager/ > python3 manage.py migrate
Operations to perform:
  Apply all migrations: Book, admin, auth, contenttypes, sessions
Running migrations:
  Applying Book.0001_initial... OK
………内容太多，此处省略………
  Applying sessions.0001_initial... OK
```

到此，将主目录下生成的`db.sqlite3`文件拖拽到Database窗口中即可，如果没有Database的窗口，可以用Pycharm专业版试试。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/4django_note.png)

# 0x01 站点管理

* 站点分为内容发布和公共访问两部分。
* 使用django站点管理模块步骤：
  * 管理界面本地化
  * 创建管理员
  * 注册模型类
  * 自定义站点管理界面



## 1、管理界面本地化

 将语言，时间设置为本地的语言时间，大陆使用的`简体中文`，时区使用`亚洲/上海`时区，修改`settings.py`文件。

> ps：为什么是上海时区，而不是北京时区？可能老外感觉上海才是国际大都市，北京只是二三线城市，毕竟老外对中国的印象都是陆家嘴而不是天安门（道听途说，不要当真，嘿嘿）

```python
# BookManager/BookManager/settings.py
LANGUAGE_CODE = 'zh-Hans'
TIME_ZONE = 'Asia/Shanghai'
```

## 2、创建管理员

```bash
python3 manage.py createsuperuser
```

 运行命令

```bash
BookManager/ > python3 manage.py createsuperuser
用户名 (leave blank to use 'dora'): test
电子邮件地址: test@test.com
Password:
Password (again):
Superuser created successfully.
```

运行服务

```
BookManager/ > python3 manage.py runserver
Watching for file changes with StatReloader
Performing system checks...
System check identified no issues (0 silenced).
February 29, 2020 - 20:52:43
Django version 3.0.3, using settings 'BookManager.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
```

运行之后，在浏览器打开 http://127.0.0.1:8000/admin，使用刚才创建的用户名密码登陆。

## 3、注册模型类

刚打开管理员界面的时候，只能看到`认证和授权`管理栏，这时候就需要将模型类注册进去。

修改`admin.py`代码

```python
# BookManager/Book/admin.py
from django.contrib import admin
from Book.models import BookInfo,PeopleInfo
admin.site.register(BookInfo)
admin.site.register(PeopleInfo)
```

刷新浏览器页面，即可看到刚添加的两个模型类。如果页面无法加载，可以看看是不是服务出现异常，如果出现异常，重新启动服务即可。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/5django_note.png)

## 4、自定义站点管理界面

在管理页面中，随便添加点数据，之后会发现书籍的名称都显示成了`BookInfo object`

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/6django_note.png)

此时，只需要在`model.py`里的class里添加以下内容即可。

```python
def __str__(self):
	return self.name
```

`model.py`完整的代码就是这个样子：

```python
# BookManager/Book/models.py
from django.db import models

class BookInfo(models.Model):  # 定义数据信息类模型
	name = models.CharField(max_length=10)  # 设计name属性
	def __str__(self):
		return self.name

class PeopleInfo(models.Model):  # 定义人物信息类模型
	name = models.CharField(max_length=10)
	gender = models.BooleanField()
	book = models.ForeignKey(BookInfo,on_delete=models.CASCADE)
	def __str__(self):
		return self.name
```

此时，再刷新页面，就可以看到显示正常了，同样 people info 界面也是正常的了。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/7_1django_note.png)

但是，这样显示还是不够直观，所以就需要自定义站点管理界面了，接下来修改`admin.py`页面，添加以下语句：

```python
class PeopleInfoAdmin(admin.ModelAdmin):
	list_display = ['id', 'name', 'gender', 'book']
```

`admin.py`完成的代码如下：

```python
# BookManager/Book/admin.py
from django.contrib import admin
from Book.models import BookInfo, PeopleInfo

class PeopleInfoAdmin(admin.ModelAdmin):
	list_display = ['id', 'name', 'gender', 'book']

admin.site.register(BookInfo)
admin.site.register(PeopleInfo, PeopleInfoAdmin)#注意此处添加PeopleInfoAdmin以注册
```

再来刷新一下页面，就舒服很多了。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/8django_note.png)



> 参考链接：
>
> https://youtu.be/BXyGr9JQVcc
>
> https://www.cnblogs.com/Demon-Mx/p/8385318.html
>
> https://blog.csdn.net/qq_35965090/article/details/81663941
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)