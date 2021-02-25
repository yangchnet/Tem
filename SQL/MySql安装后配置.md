---
author: "李昌"
title: "MySql安装后配置"
date: "2021-02-25"
tags: ["SQL"]
categories: ["SQL"]
ShowToc: true
TocOpen: true
---

# MySql安装后配置

## 1. 运行安全脚本

```bash
sudo mysql_secure_installation
```

这个脚本将根据你的选择改变一些不太安全的选项，并重置root密码

## 2. 调整身份验证和用户权限

进入mysql
```bash
sudo mysql
```

接下来，通过以下命令检查每个 MySQL 用户帐户使用的认证方法：
```sql
SELECT user,authentication_string,plugin,host FROM mysql.user;
```

在输出中
```
+------------------+-------------------------------------------+-----------------------+-----------+
| user             | authentication_string                     | plugin                | host      |
+------------------+-------------------------------------------+-----------------------+-----------+
| root             |                                           | auth_socket           | localhost |
| mysql.session    | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE | mysql_native_password | localhost |
| mysql.sys        | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE | mysql_native_password | localhost |
| debian-sys-maint | *E6CD266C880D217453293A0247D0142C9CF52730 | mysql_native_password | localhost |
+------------------+-------------------------------------------+-----------------------+-----------+
```
可以看出，root用户使用插件进行身份验证（进入时不需要输入密码）。如果想要root用户使用密码登陆，可使用如下命令进行配置：
```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```
> 将上面的password更改为你想要设置的密码

这条命令完成后，使用如下命令重新刷新配置：
```sql
FLUSH PRIVILEGES;
```

再次检查用户认证方法
```sql
SELECT user,authentication_string,plugin,host FROM mysql.user;
```
输出为：
```
+------------------+-------------------------------------------+-----------------------+-----------+
| user             | authentication_string                     | plugin                | host      |
+------------------+-------------------------------------------+-----------------------+-----------+
| root             | *35235E6B287036E68CD22314AFE8D80A0BB26D79 | mysql_native_password | localhost |
| mysql.session    | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE | mysql_native_password | localhost |
| mysql.sys        | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE | mysql_native_password | localhost |
| debian-sys-maint | *E6CD266C880D217453293A0247D0142C9CF52730 | mysql_native_password | localhost |
+------------------+-------------------------------------------+-----------------------+-----------+
```

## 3. 新建一个用户

新建一个用户
```sql
CREATE USER 'sammy'@'localhost' IDENTIFIED BY 'password';
```
> localhost表示只允许本地访问，如果想要允许远程访问，可使用'user'@'%'， %表示所有网络。

更改用户权限
```sql
GRANT ALL PRIVILEGES ON *.* TO 'sammy'@'localhost' WITH GRANT OPTION;
```
> 这里需要注意的一点是，我们不需要再使用`FLUSH PRIVILEGES;`来刷新配置，因为只有在grant表上进行过`INSERT`, `UPDATE`, 或者`DELETE`之后才需要用到`FLUSH PRIVILEGES;`。Because you created a new user, instead of modifying an existing one, FLUSH PRIVILEGES is unnecessary here.

退出
```sql
exit
```

## 4. Test Mysql

检查mysql服务状态
```bash
systemctl status mysql.service
```
输出为：
```
● mysql.service - MySQL Community Server
   Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2021-02-20 16:09:11 CST; 27min ago
  Process: 31328 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/run/mysqld/mysqld.pid (code=exited, status=0/SUCCESS)
  Process: 31319 ExecStartPre=/usr/share/mysql/mysql-systemd-start pre (code=exited, status=0/SUCCESS)
 Main PID: 31330 (mysqld)
    Tasks: 28 (limit: 2341)
   CGroup: /system.slice/mysql.service
           └─31330 /usr/sbin/mysqld --daemonize --pid-file=/run/mysqld/mysqld.pid
```
