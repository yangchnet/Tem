---
author: "李昌"
title: "ORM简介"
date: "2021-04-06"
tags: ["ORM", "SQL"]
categories: ["SQL"]
ShowToc: true
TocOpen: true
---

## 1. ORM是什么
面向对象编程把所有实体看成对象（object），关系型数据库则是采用实体之间的关系（relation）连接数据。很早就有人提出，关系也可以用对象表达，这样的话，就能使用面向对象编程，来操作关系型数据库。
![20210406113107](https://raw.githubusercontent.com/lich-Img/blogImg/master/img20210406113107.png)

简单的说，**ORM 就是通过实例对象的语法，完成关系型数据库的操作的技术，是"对象-关系映射"（Object/Relational Mapping） 的缩写。**  
ORM把数据库映射为对象 
> 数据库的表（table） --> 类（class）
记录（record，行数据）--> 对象（object）
字段（field）--> 对象的属性（attribute）

举例来说，下面是一行SQL语句
```SQL
SELECT id, first_name, last_name, phone, birth_date, sex
 FROM persons 
 WHERE id = 10
```
程序直接运行SQL，操作数据库的写法如下：
```python
res = db.execSql(sql)
name = res[0]["FIRST_NAME"]
```
改成ORM的写法如下：
```python
p = Person.get(10)
name = p.first_name
```

一比较就可以发现，ORM 使用对象，封装了数据库操作，因此可以不碰 SQL 语言。开发者只使用面向对象编程，与数据对象直接交互，不用关心底层数据库。   
总结起来，ORM有如下优点：  
- 数据模型都在一个地方定义，更容易更新和维护，也利于重用代码。
- ORM 有现成的工具，很多功能都可以自动完成，比如数据消毒、预处理、事务等等。
- 它迫使你使用 MVC 架构，ORM 就是天然的 Model，最终使代码更清晰。
- 基于 ORM 的业务代码比较简单，代码量少，语义性好，容易理解。
- 你不必编写性能不佳的 SQL。
但是ORM也有很突出的缺点：
- ORM 库不是轻量级工具，需要花很多精力学习和设置。
- 对于复杂的查询，ORM 要么是无法表达，要么是性能不如原生的 SQL。
- ORM 抽象掉了数据库层，开发者无法了解底层的数据库操作，也无法定制一些特殊的 SQL。

## 2. 实例
Django是一个非常著名的python web框架，其提供了简单易用的ORM。  
安装django  
```bash
pip install django
```
完成安装后，使用如下命令创建一个新的django项目
```bash
django-admin startproject newSite
```
尝试运行
```bash
cd newSite
python manage.py runserver
```
访问`https://127.0.0.1:8000`查看是否运行成功。

### 2.1 Model
完成了项目的创建后，下一步是要定义一个Model，一般来说，每个Model都映射一张数据库表。
- 每个模型都是一个 Python 的类，这些类继承 django.db.models.Model
- 模型类的每个属性都相当于一个数据库的字段。
- 利用这些，Django 提供了一个自动生成访问数据库的 API。

为django项目添加一个新的app
```bash
python manage.py startapp ormTest
```
切换到`ormTest`目录下，添加如下代码
```python
# models.py 
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
```
以上代码定义了一个 Person 模型，拥有 first_name 和 last_name:first_name 和 last_name 是模型的字段。每个字段都被指定为一个类属性，并且每个属性映射为一个数据库列。
在`/newSite/newSite/setting.py`中的`INSTALLED_APPS`添加如下代码，来告诉django你准备使用这个模型。
```python
INSTALLED_APPS = [
    #...
    'ormTest.apps.OrmtestConfig',
    #...
]
```
使用如下命令生成对应的数据库表
```bash
python manage.py makemigrations ormTest
python manage.py migrate
```
上面的`Person`模型会创建一个如下的数据库表：
```SQL
CREATE TABLE ormTest_person (
    "id" serial NOT NULL PRIMARY KEY,
    "first_name" varchar(30) NOT NULL,
    "last_name" varchar(30) NOT NULL
);
```

### 2.2 CRUD
进入python命令行
```bash
python manage.py shell
```
```shell
In [1]: from ormTest.models import Person

# Create 
In [4]: Person.objects.create(first_name='li', last_name='chang')
Out[4]: <Person: Person object (1)>

# Read
In [5]: Person.objects.all()
Out[5]: <QuerySet [<Person: Person object (1)>]>

# Update
In [6]: person1 = Person.objects.get(id=1)

In [8]: person1.first_name
Out[8]: 'li'

In [9]: person1.last_name
Out[9]: 'chang'

In [10]: person1.last_name = 'YANG'

In [11]: person1.save()

In [14]: person2 = Person.objects.get(id=1)

In [15]: person2.last_name
Out[15]: 'YANG'

# Delete
In [16]: person2.delete()
Out[16]: (1, {'ormTest.Person': 1})

In [17]: Person.objects.all()
Out[17]: <QuerySet []>
```

