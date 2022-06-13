---
title: 【Django 学习笔记】4、模板
date: 2020-03-01 18:28:31
id: 200301-182831
tags:
- Python
- Django
- 学习笔记
categories:
- Python 学习笔记
---
1、在项目根目录下，创建`templates`目录，在`templates`下新建`index.html`文件，PyCharm将自动生成html的文件内容格式。

```bash
.
├── Book
├── BookManager
└── templates
    └── index.html
```

<!--more-->

2、编辑`setting.py`文件第58行，修改`TEMPLATES`内容如下，目的是添加`templates`路径，好让接下来程序能够找到`index.html`文件。

```python
'DIRS': [os.path.join(BASE_DIR,'templates')],
```

3、修改`view.py`文件如下。

```python
from django.shortcuts import render
from django.http import HttpResponse

def index(request):
	context={
		'H1':'OK!',
		'H2':'-- By TeamsSix'
	}
	return render(request,'index.html',context)
```

4、修改`index.html`文件如下。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Home</title>
</head>
<body>
<H1 style="color:red;">{{ H1 }}</H1>
<H2 style="color:limegreen;">{{ H2 }}</H2>
</body>
</html>
```

5、此时，刷新浏览器。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/10django_note.png)

6、视图与`templates`的总结

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/11django_note.png)



> 更多信息欢迎关注我的微信公众号：TeamsSix
>
> 参考链接：
>
> [https://youtu.be/BXyGr9JQVcc](https://youtu.be/BXyGr9JQVcc)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)