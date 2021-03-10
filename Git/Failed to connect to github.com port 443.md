---
author: " 李昌"
title: "Failed to connect to github.com port 443"
date: "2021-03-08"
tags: ["github", "443"]
categories: ["Github"]
ShowToc: true
TocOpen: true
---
# Github 出现 Failed to connect to github.com port 443: Timed out

## 1. 问题来由
可能是由于使用了全局代理的原因

## 2. 解决
```bash
取消全局代理：
git config --global --unset http.proxy
 
git config --global --unset https.proxy
```

> 设置全局代理
```bash
git config --global http.proxy http://127.0.0.1:1080

git config --global https.proxy http://127.0.0.1:1080
```