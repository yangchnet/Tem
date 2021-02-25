---
title: python与其他语言的对比（helloworld）
date: '2021-01-22T09:27:21.000Z'
draft: false
---

# python基础

## python与其他语言的对比（hello world）

> C语言

```c
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

## python中的常用数据类型

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

```text
True
helloworld
hello 
 world
hello \n world
str1:hello.
ell
```

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

```text
google
['google', 'alibaba', 'baidu', 3.14]
['google', 'alibaba', 'baidu']
['google', 'alibaba', 'baidu', 'microsoft', 'amazon']
['google', 'alibaba', 'baidu', 'jingdong']
```

```python
# 元组：类似列表,是一系列元素的有序集合,但元组中的元素无法修改
tuple1 = ('google', 'alibaba', 'baidu')
tuple1[0] = 'amazon' # 不能被改变
```

```text
TypeError                                 Traceback (most recent call last)

<ipython-input-23-4ed3e334c834> in <module>
      1 # 元组：类似列表,是一系列元素的有序集合,但元组中的元素无法修改
      2 tuple1 = ('google', 'alibaba', 'baidu')
----> 3 tuple1[0] = 'amazon'


TypeError: 'tuple' object does not support item assignment
```

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

```text
green
{'color': 'green', 'points': 5, 'x_pos': 0, 'y_pos': 4}
```

## python中的结构语句

### if条件语句

```python
car = 'bmw'
if car == 'bmw':
    print(car.upper()) # 输出car的大写版本
else:
    print(car.title()) # 输出car的标题版本
```

```text
Bmw
```

现实世界中,很多情况下需要考虑的情形都超过两个。例如,来看一个根据年龄段收费的 游乐场:

* 4岁以下免费;
* 4~18岁收费5美元;
* 18岁\(含\)以上收费10美元。  

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

```text
Your admission cost is $5.
```

### for循环语句

```python
fruits = ['banana', 'apple', 'mango']
for fruit in fruits:
    print('当前水果：%s'%fruit)
```

```text
当前水果：banana
当前水果：apple
当前水果：mango
```

```python
# range 步长为1
for i in range(0, 6):
    print(i)
```

```text
1
2
3
4
5
```

```python
# range 步长为2
for i in range(0, 6, 2):
    print(i)
```

```text
0
2
4
```

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

```text
0
1
2
01245
```

```python
# while循环
current_number = 1
while current_number <= 5:
    print(current_number)
    current_number += 1
```

```text
1
2
3
4
5
```

### 函数

```python
def greet_user(username):
    """ 显示简单的问候语 """
    print("Hello, " + username.title() + "!")

greet_user('jesse')
greet_user('jack')
```

```text
Hello, Jesse!
Hello, Jack!
```

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

```text
第一个函数：5
第二个函数：10
第三个函数, 最大值：4, 最小值：1
```

