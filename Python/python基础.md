---
title: "1、python与其他语言的对比（helloworld）"
date: 2021-01-22T17:27:21+08:00
draft: false
---
## 1、python与其他语言的对比（hello world） 

> C语言  

```C
include<stdio.h>
int main()
{
    printf("hello world");
    return 0;
}
```

> Java语言  

```java
public class HelloWorld{ 
    public static void main(String[] args) {  
        System.out.println("Hello World!"); 
    }
}
```

> Python  

```python
print('hello world')
```

## 2、python中的常用数据类型 

* Number  
* String
* List
* Tuple
* Dictionary


```python
# Number
a = 1
b = True
c = 3.15
d = 1.1+2.2j
```


```python
# 字符串
str1 = 'hello'
str1_1 = "hello"
str2 = "world"
print(str1==str1_1)

# 字符串连接
str3 = str1 + str2
print(str3)

# 转义字符
str4 = 'hello \n world'
print(str4)
str5 = 'hello \\n world'
print(str5)

# 格式化输出
print('str1:%s.'%str1)

# 切片
print(str1[1:4])
```

    True
    helloworld
    hello 
     world
    hello \n world
    str1:hello.
    ell



```python
# 列表
list1 = ['google', 'alibaba', 2001, 3.14]

# 通过下标访问
print(list1[0])

# 更新列表
list1[2] = 'baidu'
print(list1)

# 删除元素
del list1[3]
print(list1)

# 拼接列表
list2 = ['microsoft', 'amazon']
list3 = list1 + list2
print(list3)

# 增添列表项
list1.append('jingdong')
print(list1)
```

    google
    ['google', 'alibaba', 'baidu', 3.14]
    ['google', 'alibaba', 'baidu']
    ['google', 'alibaba', 'baidu', 'microsoft', 'amazon']
    ['google', 'alibaba', 'baidu', 'jingdong']



```python
# 元组：类似列表,是一系列元素的有序集合,但元组中的元素无法修改
tuple1 = ('google', 'alibaba', 'baidu')
tuple1[0] = 'amazon' # 不能被改变
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-23-4ed3e334c834> in <module>
          1 # 元组：类似列表,是一系列元素的有序集合,但元组中的元素无法修改
          2 tuple1 = ('google', 'alibaba', 'baidu')
    ----> 3 tuple1[0] = 'amazon'
    

    TypeError: 'tuple' object does not support item assignment



```python
# 字典
dict1 = {
         'color': 'green', 
         'points': 5
}

# 访问列表中的值
print(dict1['color'])

# 增加字典中键值对
dict1['x_pos'] = 0
dict1['y_pos']  =4
print(dict1)
```

    green
    {'color': 'green', 'points': 5, 'x_pos': 0, 'y_pos': 4}


## 3、python中的结构语句 

### 3.1、if条件语句 


```python
car = 'bmw'
if car == 'bmw':
    print(car.upper()) # 输出car的大写版本
else:
    print(car.title()) # 输出car的标题版本
```

    Bmw


现实世界中,很多情况下需要考虑的情形都超过两个。例如,来看一个根据年龄段收费的
游乐场:
* 4岁以下免费;
* 4~18岁收费5美元;
* 18岁(含)以上收费10美元。  

> 如果只使用一条 if 语句,如何确定门票价格呢?下面的代码确定一个人所属的年龄段,并打印一条包含门票价格的消息:


```python
age = 12
if age < 4:
    print("Your admission cost is $0.")
elif age < 18:
    print("Your admission cost is $5.")
else:
    print("Your admission cost is $10.")
```

    Your admission cost is $5.


### 3.2、for循环语句 


```python
fruits = ['banana', 'apple', 'mango']
for fruit in fruits:
    print('当前水果：%s'%fruit)
```

    当前水果：banana
    当前水果：apple
    当前水果：mango



```python
# range 步长为1
for i in range(0, 6):
    print(i)
```

    1
    2
    3
    4
    5



```python
# range 步长为2
for i in range(0, 6, 2):
    print(i)
```

    0
    2
    4



```python
# break 和 continue
for i in range(0, 6):
    if i == 3:
        break
    print(i)

for i in range(0, 6):
    if i == 3:
        continue
    print(i, end='')
```

    0
    1
    2
    01245

### 3.3、while循环语句


```python
# while循环
current_number = 1
while current_number <= 5:
    print(current_number)
    current_number += 1
```

    1
    2
    3
    4
    5


### 3.4、函数 


```python
def greet_user(username):
    """ 显示简单的问候语 """
    print("Hello, " + username.title() + "!")

greet_user('jesse')
greet_user('jack')
```

    Hello, Jesse!
    Hello, Jack!



```python
# 有返回值的函数
def add(a, b):
    return a+b
print('第一个函数：%d'%add(2, 3))

# 列表作为参数的函数
def add_l(mylist):
    result = 0
    for l in mylist:
        result += l
    return result
print('第二个函数：%d'%add_l([1, 2, 3, 4]))

# 有多个返回值的函数
def muti_re(mylist):
    a = max(mylist)
    b = min(mylist)
    return a, b
a, b = muti_re([1, 2, 3, 4])
print('第三个函数, 最大值：%d, 最小值：%d'%(a,b))
```

    第一个函数：5
    第二个函数：10
    第三个函数, 最大值：4, 最小值：1


有时候,你预先不知道函数需要接受多少个实参, 但是Python允许函数从调用语句中收集任意数量的实参。  
> 例如,来看一个制作比萨的函数,它需要接受很多配料,但你无法预先确定顾客要多少种配
料。下面的函数只有一个形参 *toppings ,但不管调用语句提供了多少实参,这个形参都将它们
统统收入囊中:


```python
def make_pizza(*toppings):
    print(toppings)
make_pizza('pepperoni')
make_pizza('mushrooms', 'green peppers', 'extra cheese')
```

    ('pepperoni',)
    ('mushrooms', 'green peppers', 'extra cheese')


### 3.5、类 

  面向对象编程是最有效的软件编写方法之一。在面向对象编程中,
你编写表示现实世界中的事物和情景的类,并基于这些类来创建对象。
编写类时,你定义一大类对象都有的通用行为。基于类创建对象时,
每个对象都自动具备这种通用行为,然后可根据需要赋予每个对象独
特的个性。使用面向对象编程可模拟现实情景,其逼真程度达到了令
你惊讶的地步。
  根据类来创建对象被称为实例化,这让你能够使用类的实例。在
本章中,你将编写一些类并创建其实例。你将指定可在实例中存储什
么信息,定义可对这些实例执行哪些操作。你还将编写一些类来扩展
既有类的功能,让相似的类能够高效地共享代码。你将把自己编写的类存储在模块中,并在
自己的程序文件中导入其他程序员编写的类

使用类几乎可以模拟任何东西。下面来编写一个表示小狗的简单类 Dog ——它表示的不是特
定的小狗,而是任何小狗。对于大多数宠物狗,我们都知道些什么呢?它们都有名字和年龄;我
们还知道,大多数小狗还会蹲下和打滚。由于大多数小狗都具备上述两项信息(名字和年龄)和
两种行为(蹲下和打滚),我们的 Dog 类将包含它们。这个类让Python知道如何创建表示小狗的对
象。编写这个类后,我们将使用它来创建表示特定小狗的实例。 


```python
# 创建类
class Dog():
    def __init__(self, name, age):
        self.name = name
        self.age = age 
        
    def sit(self):
        print(self.name.title() + " is now sitting.") 
        
    def roll_over(self):
        print(self.name.title() + " rolled over!")对字符串的大小写进行更改
```

类中的函数称为方法;你前面学到的有关函数的一切都适用于方法,就目前而言,唯一重要
的差别是调用方法的方式。方法 __init__() 是一个特殊的方法,每当你根据 Dog 类创建新实
例时,Python都会自动运行它。在这个方法的名称中,开头和末尾各有两个下划线,这是一种约
定,旨在避免Python默认方法与普通方法发生名称冲突。


```python
# 根据类创建实例
my_dog = Dog('willie', 6)
print("My dog's name is " + my_dog.name.title() + ".")
print("My dog is " + str(my_dog.age) + " years old.")
```

    My dog's name is Willie.
    My dog is 6 years old.



```python
# 访问类的属性
my_dog.name
```




    'willie'




```python
# 调用类的方法
my_dog.sit()
my_dog.roll_over()
```

    Willie is now sitting.
    Willie rolled over!


## 4、对字符串的一些操作 

### 4.1、对字符串的大小写进行更改


```python
# 对字符串的大小写进行更改
name = "ada lovelace"
print(name.title())

name = "Ada Lovelace"
print(name.upper())
print(name.lower())
```

    Ada Lovelace
    ADA LOVELACE
    ada lovelace


### 4.2、合并/拼接字符串


```python
first_name = "ada"
last_name = "lovelace"
full_name = first_name + " " + last_name
print(full_name)

```

    ada lovelace


### 4.3、使用制表符或换行符来添加空白


```python
print("\tPython")
print("Languages:\nPython\nC\nJavaScript")
```

    	Python
    Languages:
    Python
    C
    JavaScript


### 4.4、删除空白 


```python
favorite_language = ' python '
# 删除末尾的空白
favorite_language.rstrip()
# 删除开头的空白
favorite_language.lstrip()
```




    'python '



### 4.5、字符串分割 


```python
my_languages = 'python,C,JAVA,PHP,MATLAB'
languages_list = my_languages.split(',')
print(languages_list)
print(languages_list[0])
print(languages_list[0][3])
```

    ['python', 'C', 'JAVA', 'PHP', 'MATLAB']
    python
    h


##  习题

### 1、打印出99乘法表 

### 2、判断是否是闰年

### 3、编写Python程序计算斐波那契数列，要求：输入序号n，输出数列的第n项，数列的计算用函数实现

### 4、使用面向对象的思想， 创建一个名为 User 的类,其中包含属性 first_name 和 last_name ,还有用户简介通常会存储的其他几个属性。在类 User 中定义一个名为 describe_user() 的方法,它打印用户信息摘要;再定义一个名为 greet_user() 的方法,它向用户发出个性化的问候。
### 创建多个表示不同用户的实例,并对每个实例都调用上述两个方法。
