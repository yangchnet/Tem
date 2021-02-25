---
title: 'Django,Forms'
date: '2021-01-22T09:27:21.000Z'
draft: false
---

# django form

## Django, Forms

Column 1: Jan 26, 2019 11:41 PM

## 使用forms完成了用户登录

### 1、创建model

```text
class User(User):
    pass
```

这里使用了django提供的User类，直接继承

### 2、创建UserForm类

```text
class SigninFrom(forms.Form):
    user_name = forms.CharField()
    user_email = forms.EmailField()
    user_password = forms.CharField()
```

### 3、完成模板

  {% csrf\_token %} 用户名  Email  密码  在Bodydetect上注册

* 注意，这里三者要统一用一样的name，否则会接收不到数据

### views.py如何编写

```text
def fun(request):
        if request.method == 'GET':
                ...
                return ...
        if request.method == 'POST':
                ...
                return
```

* 使用forms.cleaned\_data访问数据，返回字典类型

### 你还可以

* 从view中返回form，django会自动为你渲染，当然，十分丑陋

例如：

 {% csrf\_token %}

|  {{ form }} |
| :--- |


 在Bodydetect上注册

只需要为form保留位置

渲染结果：

![](https://github.com/yangchnet/Tem/tree/73198dc3a08ab7039d303be561dda4e51ef6c3b5/Django/_001-be5cb266-5641-43c4-a10b-20b76c49605b.png)

不使用自动渲染

![](https://github.com/yangchnet/Tem/tree/73198dc3a08ab7039d303be561dda4e51ef6c3b5/Django/_002-0155cf6c-ebd5-4438-9d76-57c36009cc6a.png)

### 使用ModelForm类

> views.py

```text
from django.db import models
class Article(models.Model):
        author = models.CharField(max_length = 150)
        title = models.CharField(max_lengeth = 100)
        content = models.CharField(max_lengeth = 10000)
        time = models.TimeField( )
```

> model.py

```text
from django.forms import ModelForm
class Article(ModelForm):
    class Meta:
                model = Article
                fields = ['author', 'title', 'content', 'time']
```

### save方法

* 每个ModelForm还有一个save\(\)方法，此方法根据绑定到表单的数据创建并保存数据库对象。

  **Create a form instance with POST data.**

  > > > f = AuthorForm\(request.POST\)

  **Create, but don't save the new author instance.**

  > > > new\_author = f.save\(commit=False\)

  **Modify the author in some way.**

  > > > new\_author.some\_field = 'some\_value'

  **Save the new instance.**

  > > > new\_author.save\(\)

  **Now, save the many-to-many data for the form.**

  > > > f.save\_m2m\(\)

### Reference

[Django 教程 9: 使用表单](https://developer.mozilla.org/zh-CN/docs/Learn/Server-side/Django/Forms)

[从模型创建表单 \| Django documentation \| Django](https://docs.djangoproject.com/zh-hans/2.0/topics/forms/modelforms/)

[The Forms API \| Django documentation \| Django](https://docs.djangoproject.com/zh-hans/2.0/ref/forms/api/)

