---
author: "李昌"
title: "ubuntu中增加用户"
date: "2021-02-25"
tags: ["Linux"]
categories: ["Linux"]
ShowToc: true
TocOpen: true
---

# ubuntu中增加用户 

```bash
sudo useradd -m [username] -s /bin/bash #创建账户，使用/bin/bash作为shell
sudo passwd [username] #设置密码
sudo adduser [username] sudo #添加管理员权限
su [username]#切换用户
```

