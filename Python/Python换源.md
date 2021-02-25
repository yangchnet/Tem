---
author: "李昌"
title: "Python换源"
date: "2021-02-25"
tags: ["Python"]
categories: ["Python"]
ShowToc: true
TocOpen: true
---

# Python换源

## 1. 临时换源

可以在使用pip的时候在后面加上-i参数，指定pip源
```bash
pip install scrapy -i https://pypi.tuna.tsinghua.edu.cn/simple
```

## 2. 永久换源

永久修改：
linux:
修改 ~/.pip/pip.conf (没有就创建一个)， 内容如下：
```python
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```
