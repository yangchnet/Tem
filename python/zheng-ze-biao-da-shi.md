---
title: 正则表达式
date: '2021-01-22T09:27:21.000Z'
draft: false
---

# 正则表达式

## 1、什么是正则表达式

正则表达式，又称规则表达式。（英语：Regular Expression，在代码中常简写为regex、regexp或RE），计算机科学的一个概念。正则表达式通常被用来检索、替换那些符合某个模式\(规则\)的文本。（摘自百度百科）

#### 你可能熟悉文本查找,即按下 Ctrl-F,输入你要查找的词。“正则表达式”更进一步,它们让你指定要查找的“模式”。你也许不知道一家公司的准确电话号码,但如果你住在美国或加拿大,你就知道有3 位数字,然后是一个短横线,然后是 4 位数字\(有时候以 3 位区号开始\)。因此作为一个人,你看到一个电话号码就知道:415-555-1234 是电话号码,但 4,155,551,234 不是。

#### 正则表达式很有用,但如果不是程序员,很少会有人了解,它,尽管大多数现代文本编辑器和文字处理器\(诸如微软的 Word 或 OpenOffice\)都有查找和查找替换功能,可以根据正则表达式查找。正则表达式可以节约大量时间,不仅适用于软件用户,也适用于程序员。实际上,技术作家 Cory Doctorow 声称,甚至应该在教授编程之前,先教授正则表达式:

#### “知道\[正则表达式\]可能意味着用 3 步解决一个问题,而不是用 3000 步。如果你是一个技术怪侠,别忘了你用几次击键就能解决的问题,其他人需要数天的烦琐工作才能解决,而且他们容易犯错。” 1

\(摘自《Python编程快速上手—让繁琐工作自动化》\)

## 2、不用正则表达式来查找文本模式

假设你希望在字符串中查找电话号码。你知道模式:3 个数字,一个短横线,3 个数字,一个短横线,再是 4 个数字。例如:415-555-4242。 假定我们用一个名为 isPhoneNumber\(\)的函数,来检查字符串是否匹配模式,它 返回 True 或 False。

```python
def isPhoneNumber(text):
    if len(text) != 12:
        return False
    for i in range(0, 3):
        if not text[i].isdecimal():
            return False
    if text[3] != '-':
        return False
    for i in range(4, 7):
        if not text[i].isdecimal():
            return False
    if text[7] != '-':
        return False
    for i in range(8, 12):
        if not text[i].isdecimal():
            return False
    return True
print('415-555-4242 is a phone number:')
print(isPhoneNumber('415-555-4242'))
print('Moshi moshi is a phone number:')
print(isPhoneNumber('Moshi moshi'))
```

```text
415-555-4242 is a phone number:
True
Moshi moshi is a phone number:
False
```

添加更多代码,才能在更长的字符串中寻找这种文本模式

```python
message = 'Call me at 415-555-1011 tomorrow. 415-555-9999 is my office.'
for i in range(len(message)):
    chunk = message[i:i+12]
    if isPhoneNumber(chunk):
        print('Phone number found: ' + chunk)
print('Done')
```

```text
Phone number found: 415-555-1011
Phone number found: 415-555-9999
Done
```

## 3、使用正则表达式进行匹配

### 3.1、 创建正则表达式对象

```python
# 导入re模块来使用正则
import re
```

```python
# 创建正则表达式
phoneNumRegex = re.compile(r'\d\d\d-\d\d\d-\d\d\d\d')
```

### 3.2、使用正则进行查询

```python
phoneNumRegex = re.compile(r'\d\d\d-\d\d\d-\d\d\d\d')
mo = phoneNumRegex.search('My number is 415-555-4242.')
print('Phone number found: ' + mo.group())
```

```text
Phone number found: 415-555-4242
```

### 3.3、利用括号分组

假定想要将区号从电话号码中分离。添加括号将在正则表达式中创建“分组”\(\d\d\d\)-\(\d\d\d-\d\d\d\d\)。然后可以使用 group\(\)匹配对象方法,从个分组中获取匹配的文本。正则表达式字符串中的第一对括号是第 1 组。第二对括号是第 2 组。向 group\(\)匹配对象方法传入整数 1 或 2,就可以取得匹配文本的不同部分。向 group\(\)方法传入 0 或不传入参数,将返回整个匹配的文本

```python
phoneNumRegex = re.compile(r'(\d\d\d)-(\d\d\d-\d\d\d\d)')
mo = phoneNumRegex.search('My number is 415-555-4242.')
# 第一个分组
mo.group(1)

# 第二个分组
# mo.group(2)

# 全部
# mo.group(0)
# mo.group()
```

```text
'415'
```

### 3.4、用管道匹配多个分组

字符\|称为“管道”正则表达式 r'Batman\|Tina Fey'将匹配'Batman'或'Tina Fey'。如果 Batman 和 Tina Fey 都出现在被查找的字符串中,第一次出现的匹配文本,将作为 Match 对象返回

```python
heroRegex = re.compile (r'Batman|Tina Fey')
mo1 = heroRegex.search('Batman and Tina Fey.')
mo1.group()
```

```text
'Tina Fey'
```

```python
mo2 = heroRegex.search('Tina Fey and Batman.')
mo2.group()
```

```text
'Tina Fey'
```

### 3.5、 用问号实现可选匹配

有时候,想匹配的模式是可选的。就是说,不论这段文本在不在,正则表达式都会认为匹配。字符?表明它前面的分组在这个模式中是可选的。

```python
batRegex = re.compile(r'Bat(wo)?man')
mo1 = batRegex.search('The Adventures of Batman')
mo1.group()
```

```text
'Batwoman'
```

```python
mo2 = batRegex.search('The Adventures of Batwoman')
mo2.group()
```

```text
'Batwoman'
```

### 3.6、 用星号匹配零次或多次

\*\(称为星号\)意味着“匹配零次或多次”,即星号之前的分组,可以在文本中出现任意次。它可以完全不存在,或一次又一次地重复。

```python
batRegex = re.compile(r'Bat(wo)*man')
mo1 = batRegex.search('The Adventures of Batman')
mo1.group()
```

```text
'Batman'
```

```python
mo2 = batRegex.search('The Adventures of Batwoman')
mo2.group()
```

```text
'Batwoman'
```

```python
mo3 = batRegex.search('The Adventures of Batwowowowoman')
mo3.group()
```

```text
'Batwowowowoman'
```

### 3.7、 用加号匹配一次或多次

\*意味着“匹配零次或多次”,+\(加号\)则意味着“匹配一次或多次”。星号不要求分组出现在匹配的字符串中,但加号不同,加号前面的分组必须“至少出现一次”。这不是可选的。

```python
batRegex = re.compile(r'Bat(wo)+man')
mo1 = batRegex.search('The Adventures of Batwoman')
mo1.group()
```

```text
'Batwoman'
```

```python
mo2 = batRegex.search('The Adventures of Batwowowowoman')
mo2.group()
```

```text
'Batwowowowoman'
```

```python
mo3 = batRegex.search('The Adventures of Batman')
mo3 == None
```

```text
True
```

> 未完待续

