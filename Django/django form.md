---
author: "李昌"
title: "Django,Forms"
date: "2021-02-25"
tags: ["Django"]
categories: ["Django"]
ShowToc: true
TocOpen: true
---
# Django, Forms

# 使用forms完成了用户登录

## 1、创建model

    class User(User):
        pass

这里使用了django提供的User类，直接继承

## 2、创建UserForm类

    class SigninFrom(forms.Form):
        user_name = forms.CharField()
        user_email = forms.EmailField()
        user_password = forms.CharField()

## 3、完成模板

    <form  action="{% url 'permission:signin' %}" accept-charset="UTF-8" method="post">
                                <input name="utf8" type="hidden"
                                                                              value="&#x2713;"/>
                                {% csrf_token %}
                                <dl class="form-group mt-0">
                                    <dt class="input-label">
                                        <label class="form-label f5" for="user[login]">用户名</label>
                                    </dt>
                                            <input type="text" name="user_name" id="user_name"
                                                   class="form-control form-control-lg input-block"
                                                   placeholder="{{ default_name }}" autofocus>
    
                                </dl>
                                <dl class="form-group">
                                    <dt class="input-label">
                                        <label class="form-label f5" for="user[email]">Email</label>
                                    </dt>
                                    <dd>
                                            <input type="text" name="user_email" id="user_email"
                                                   class="form-control form-control-lg input-block js-email-notice-trigger"
                                                   placeholder="you@example.com">
                                    </dd>
                                </dl>
                                    <dl class="form-group">
                                        <dt class="input-label">
                                            <label class="form-label f5" for="user[password]">密码</label>
                                        </dt>
                                        <dd>
                                            <input type="password" name="user_password" id="user_password"
                                                   class="form-control form-control-lg input-block"
                                                   placeholder="Create a password">
                                        </dd>
                                    </dl>
                                <button class="btn-mktg btn-primary-mktg btn-large-mktg f4 btn-block" type="submit"
                                        data-ga-click="Signup, Attempt, location:teams;">在Bodydetect上注册
                                </button>
    
                            </form>

- 注意，这里三者要统一用一样的name，否则会接收不到数据

## views.py如何编写

    def fun(request):
    		if request.method == 'GET':
    				...
    				return ...
    		if request.method == 'POST':
    				...
    				return

- 使用forms.cleaned_data访问数据，返回字典类型

## 你还可以

- 从view中返回form，django会自动为你渲染，当然，十分丑陋

例如：

    <form action = "{% url 'permission:signin' %}" method="post">
                                {% csrf_token %}
                                <table>
                                    {{ form }}
                                </table>
                                <button class="btn-mktg btn-primary-mktg btn-large-mktg f4 btn-block" type="submit"
                                        data-ga-click="Signup, Attempt, location:teams;">在Bodydetect上注册
                                </button>
                            </form>

只需要为form保留位置

渲染结果：

![](_001-be5cb266-5641-43c4-a10b-20b76c49605b.png)

不使用自动渲染

![](_002-0155cf6c-ebd5-4438-9d76-57c36009cc6a.png)

## 使用ModelForm类

> views.py

    from django.db import models
    class Article(models.Model):
    		author = models.CharField(max_length = 150)
    		title = models.CharField(max_lengeth = 100)
    		content = models.CharField(max_lengeth = 10000)
    		time = models.TimeField( )

> model.py

    from django.forms import ModelForm
    class Article(ModelForm):
        class Meta:
    				model = Article
    				fields = ['author', 'title', 'content', 'time']

## save方法

- 每个ModelForm还有一个save()方法，此方法根据绑定到表单的数据创建并保存数据库对象。

    # Create a form instance with POST data.
    >>> f = AuthorForm(request.POST)
    
    # Create, but don't save the new author instance.
    >>> new_author = f.save(commit=False)
    
    # Modify the author in some way.
    >>> new_author.some_field = 'some_value'
    
    # Save the new instance.
    >>> new_author.save()
    
    # Now, save the many-to-many data for the form.
    >>> f.save_m2m()

## Reference

[Django 教程 9: 使用表单](https://developer.mozilla.org/zh-CN/docs/Learn/Server-side/Django/Forms)

[从模型创建表单 | Django documentation | Django](https://docs.djangoproject.com/zh-hans/2.0/topics/forms/modelforms/)

[The Forms API | Django documentation | Django](https://docs.djangoproject.com/zh-hans/2.0/ref/forms/api/)
