---
title: 【Django 学习笔记】3、视图
date: 2020-03-01 18:28:16
id: 200301-182816
tags:
- Python
- Django
- 学习笔记
categories:
- Python 学习笔记
---

* 后台管理页面搞定之后，就需要做公共页面的访问了
* 对于Django的设计框架MVT：
  * 用户在URL中请求的是视图
  * 视图接受请求后进行处理
  * 然后将处理的结果返回给请求者
* 使用视图时要进行的两步操作：
  * 定义视图
  * 配置URL

<!--more-->

# 0x00 定义视图

* 视图就是一个Python函数，被定义在应用的views.py中。
* 视图的第一个参数是 `HttpRequest`类型的对象`request`，包含了所有请求信息
* 视图必须返回`HttpResponse`对象，包含返回给请求者的响应信息。
* 需要导入`HttpResponse`模块：`from django.http import HttpResponse`
* 定义视图函数：响应字符串`OK!`给客户端

首先修改`views.py`文件，添加响应内容，修改后如下。

```python
# BookManager/Book/views.py
from django.http import HttpResponse

def index(request):
	return HttpResponse('OK!    -- By TeamsSix')
```

# 0x01、配置URL

之后再修改`urls.py`文件，添加`path('', views.index),`，完整的代码如下：

```python
# BookManager/BookManager/urls.py
from django.contrib import admin
from django.urls import path
from Book import views

urlpatterns = [
	path('admin/', admin.site.urls),
	path('', views.index),
]
```

此时，当我们访问`127.0.0.1:8000`的时候，代码运行顺序是这样的：

```python
	  HttpRequest
		↓ ↓ ↓
ROOT_URLCONF = 'BookManager.urls'	# /BookManager/settings.py
		↓ ↓ ↓
path('', views.index),	# /BookManager/urls.py
		↓ ↓ ↓
return HttpResponse('OK!    -- By TeamsSix')	# /Book/views.py
		↓ ↓ ↓
      HttpResponse
```

最终，浏览器将顺利返回我们的`HttpResponse`

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/9django_note.png)

> 更多信息欢迎关注我的微信公众号：TeamsSix
>
> 参考链接：
>
> [https://youtu.be/BXyGr9JQVcc](https://youtu.be/BXyGr9JQVcc)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)