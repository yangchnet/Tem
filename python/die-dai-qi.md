---
title: 迭代器相关内容
date: '2021-01-22T09:27:21.000Z'
draft: false
---

# 迭代器相关内容

## 1. 代理迭代

对于一个自定义对象，若想实现在这个自定义对象上的迭代，只需要定义一个`__iter__()`方法，将迭代操作代理到容器内部得对象上去。如：

```python
class Node: 
    def __init__(self, value): 
        self._value = value 
        self._children = []

    #---

    def __iter__(self): 
        return iter(self._children)
```

