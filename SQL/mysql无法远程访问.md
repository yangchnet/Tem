---
author: "李昌"
title: "Mysql无法远程访问"
date: "2021-02-25"
tags: ["SQL"]
categories: ["SQL"]
ShowToc: true
TocOpen: true
---

#  Mysql无法远程访问

在使用navicat远程连接阿里云的时候，出现“2003 can t connect to mysql server on 10061”错误

经过艰难的谷歌百度stackflow后，发现是3306端口没有监听外部连接，只接收内部ip访问。

### 解决方案
1. 首先保证阿里云服务器3306端口开放
2. 使用netstat -ntpl |grep 3306命令查看3306端口状态
 tcp        0      0 127.0.0.1:22                  0.0.0.0:*                   LISTEN      -   
 可看出只接收内部访问  
3. 打开/etc/mysql/mysql.conf.d/mysqld.cnf(网上大部分说是:/etc/mysql/my.cnf)
将bind-address  = 127.0.0.1改成bind-address  = 0.0.0.0
4. 再次使用netstat -ntpl |grep 3306命令查看
tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN      -  
此时3306端口开始监听所有网络访问
**如果是ipv6主机，则改为 bind-address  = :: ,表示监听所有网络**

# 主机'xxx.xx.xxx.xxx'不允许连接到此MySQL服务器

在进行连接ipv6主机的时候出现了如下问题：django.db.utils.InternalError: (1130, "Host '2409:8930:1450:316:6179:c54:5901:2f2b' is not allowed to connect to this MySQL server")
解决方法如下：
```mysql
mysql> CREATE USER 'monty'@'localhost' IDENTIFIED BY 'some_pass';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'monty'@'localhost'
    ->     WITH GRANT OPTION;
mysql> CREATE USER 'monty'@'%' IDENTIFIED BY 'some_pass';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'monty'@'%'
    ->     WITH GRANT OPTION;
```

'user'@'%'后面的百分号代表所有网络

### 有时候用root用户还不行，要重新建一个用户
