---
title: 【Django 学习笔记】1、基础概念和MVT架构
date: 2020-02-29 22:07:46
id: 200229-220746
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/3django_note.png
tags:
- Python
- Django
- 学习笔记
categories:
- Python 学习笔记
---

# 0x00 Django 简介

* Django是Python写的开源Web开发框架，主要目的是做一个简便、快速的开发数据库驱动的网站
* Django遵循MVC设计模式，在Django中有个专有名词，叫做MVT
  * 设计模式就是前辈们在开发过程中总结出来的经验和套路
  * MVC是一种设计模式，在这种设计模式下衍生出了MVT
* Django中文说明文档：[https://yiyibooks.cn/xx/django_182/index.html](https://yiyibooks.cn/xx/django_182/index.html)

<!--more-->

# 0x01 MVC 简介

* 全拼：`Model View Controller`
* MVC 核心思想：**解耦**
  - 让不同的模块之间降低耦合, 增强代码的可扩展性和可移植性, 实现更好的向后续版本的兼容
  - 开发原则 : **高内聚, 低耦合**
* MVC 解析
  - `M`全拼为`Model`, 主要封装对数据库层的访问, 内嵌ORM框架, 实现面向对象的编程来操作数据库.

  - `V`全拼为`View`, 用于封装结果, 内嵌了模板引擎, 实现动态展示数据.

  - `C`全拼为`Controller`, 用于接收GET或POST请求, 处理业务逻辑, 与Model和View交互, 返回结果.

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/1django_note.png)

# 0x02 MVT 简介

* MVT全拼为`Model-View-Template`
* MVT核心思想： **解耦**（按照模块间的职能进行划分，然后做解耦）
* MVT解析
  - `M (模型)`全拼为`Model`, 与MVC中的M功能相同, 负责数据处理, 内嵌了ORM框架.
  - `V (视图)`全拼为`View`, 与MVC中的C功能相同, 接收HttpRequest, 业务处理，返回HttpResponse.
  - `T (模板)`全拼为`Template`, 与MVC中的V功能相同, 负责封装构造要返回的html, 内嵌了模板引擎.
* MVT 和 MVC 差异就在于黑箭头标识出来的部分.

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/2django_note.png)

**学习 Django, 重点就是研究 `Model-View-Template` 三个模块间如何协同工作及各自模块的代码如何编写。**

# 0x03 Django安装及项目创建

## 1、安装Django

```
pip install django
```

## 2、创建Django项目

以书籍管理系统为例，这里创建的项目名为"book"

```bash
> django-admin startproject BookManager #创建项目
> cd BookManager
BookManager/ > python3 manage.py startapp Book #创建应用
BookManager/ > tree
.
├── Book
│   ├── __init__.py	#表示文件Book可以被当作包使用
│   ├── admin.py	#后台的站点管理注册文件
│   ├── apps.py
│   ├── migrations	#做模型迁移
│   │   └── __init__.py
│   ├── models.py	#MVT中的M
│   ├── tests.py	#做测试用
│   └── views.py	#MVT中的V
├── BookManager
│   ├── __init__.py	#表示文件BookManager可以被当作包使用
│   ├── __pycache__
│   │   ├── __init__.cpython-37.pyc
│   │   └── settings.cpython-37.pyc
│   ├── asgi.py
│   ├── settings.py	#项目的整体配置文件
│   ├── urls.py		#项目的URL配置文件
│   └── wsgi.py		#项目与WSGI兼容的Web服务器入口
└── manage.py		#项目运行的入口, 指定配置文件路径

4 directories, 15 files
```

创建之后，使用PyChram打开，在`setting.py`的第39行下方添加`'Book',`即将INSTALLED_APPS修改成如下所示：

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'Book',	#添加此行以安装应用
]
```

## 3、运行项目

在项目目录下，执行`python3 manage.py runserver`即可运行

```
python3 manage.py runserver
```

运行结果：

```bash
BookManager/ > python3 manage.py runserver
Watching for file changes with StatReloader
Performing system checks...
System check identified no issues (0 silenced).
You have 17 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
February 28, 2020 - 19:08:25
Django version 3.0.3, using settings 'BookManager.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
```

此时，浏览器访问`http://127.0.0.1:8000/`，出现以下界面，说明项目已经成功创建了。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/3django_note.png)



> 参考链接：
>
> [https://youtu.be/BXyGr9JQVcc](https://youtu.be/BXyGr9JQVcc)
>
> [https://www.cnblogs.com/Demon-Mx/p/8385318.html](https://www.cnblogs.com/Demon-Mx/p/8385318.html)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)