---
author: "李昌"
title: "安装完virtualbox后，提示内核问题"
date: "2021-02-25"
tags: ["Linux"]
categories: ["Linux"]
ShowToc: true
TocOpen: true
---

# 安装完virtualbox后，提示内核问题

## 解决方法 

### 安装内核匹配版本
sudo pacman -S linux419-virtualbox-host-modules
### 重新加载内核模块
sudo /sbin/rcvboxdrv
然后重启即可
