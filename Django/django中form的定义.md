---
author: "李昌"
title: "django中forms的定义"
date: "2021-02-25"
tags: ["Django"]
categories: ["Django"]
ShowToc: true
TocOpen: true
---
# django中forms的定义 

### 直接定义
```python
class ContactForm(forms.Form):
    date = DateField(widget=CalendarWidget)
    name = CharField(max_length=40, widget=OtherWidget)
```
widget参数定义了要使用的小部件，小部件选项可见[这里](https://docs.djangoproject.com/en/2.0/ref/forms/widgets/#built-in-widgets)

### 通过模型定义  
必须继承ModelForm类
```python
from django.forms import ModelForm
class BlogForm(ModelForm):
    class Meta:
        model = Blog
        fields = ['author', 'essay', 'title', 'label', 'cover']
        widgets = {
            'essay': CKEditorWidget,
            'cover': 
        }
```
通过定义内部类来生命form的属性
> 常用内部类参数说明：  
model: 说明要继承的模型  
field：说明要在表单中显示的字段，__all__表示所有  
exclude: 要从表单中排除的字段  
widgets: 设置字段的小部件  
[（详细文档）](https://docs.djangoproject.com/en/2.0/ref/forms/fields/)
