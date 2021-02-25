---
author: "李昌"
title: "删除WordPress"
date: "2021-02-25"
tags: ["Linux"]
categories: ["Linux"]
ShowToc: true
TocOpen: true
---

# 删除WordPress

1. 删除网络文件：
```bash
rm -Rf /var/www/html/*
```

2. 删除数据库。首先获取mysql的root密码（通过ssh登录时显示在MOTD中）。
```sql
mysql -uroot -p
```
输入密码后使用语句：
```sql
DROP DATABASE wordpress;
```
删除WordPress数据库

```sql
exit;
```
