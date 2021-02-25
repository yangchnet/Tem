---
title: 如何在manjaro中安装MySQL
date: 2021-01-22T09:27:21.000Z
draft: false
---

# 如何在manjaro中安装MySQL

```bash
# 安装MySQL
pacman -S mysql
# 初始化MySQL，记住输出的root密码
mysqld --initialize --user=mysql --basedir=/usr --datadir=/var/lib/mysql
# 设置开机启动MySQL服务
systemctl enable mysqld.service
systemctl daemon-reload
systemctl start mysqld.service
# 使用MySQL前必须修改root密码，MySQL 8.0.15不能使用set password修改密码
mysql -u root -p
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';
```

