---
author: "李昌"
title: "pm2的使用"
date: "2021-02-25"
tags: ["Linux"]
categories: ["Linux"]
ShowToc: true
TocOpen: true
---

# pm2的使用

PM2 Runtime是具有内置Load Balancer的Node.js应用程序的生产过程管理器。它允许您永久保持应用程序的活动，无需停机即可重新加载它们，并促进常见的Devops任务。

## 对于ipv4主机

* 用户名： lc
* 密码： Lichang1-  

登录后使用以下命令查看当前后台进程状态  
```bash
pm2 ls 
```
若status为online，则表明脚本运行正常，若为其他状态，则说明出现错误或异常，可用以下命令停止脚本运行
```bash
pm2 stop 0
```
然后使用以下命令查看日志
```bash
cat server.log
```   
从日志中可获取错误信息或其他提示。

若脚本一切正常，可直接进入数据库查看数据。

**注意：注销登录前请使用```pm2 start server.sh```命令重新启动脚本！！！**

## 对于ipv6主机 

* 用户名： ubuntu
* 密码： ahnu2019

登录后使用以下命令查看当前后台进程状态  
```bash
pm2 ls 
```
若status为online，则表明脚本运行正常，若为其他状态，则说明出现错误或异常，可用以下命令停止脚本运行
```bash
pm2 stop 1
```
然后使用以下命令查看日志
```bash
cat server.log
```   
从日志中可获取错误信息或其他提示。

若脚本一切正常，可直接进入数据库查看数据。

**注意：注销登录前请使用```pm2 start server.sh```命令重新启动脚本！！！**
