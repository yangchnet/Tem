---
title: "Didyouinstallmysqlclient?"
date: 2021-01-22T17:27:21+08:00
draft: false
---
# Did you install mysqlclient?


在进行新的django项目时出现了这个错误

## 解决方法

1. 确保pymysql和mysqlcient都安装
2. 在和setting.py同级的init.py中加入  

```python
import pymysql

pymysql.install_as_MySQLdb()
```
