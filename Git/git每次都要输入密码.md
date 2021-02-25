---
author: "李昌"
title: "解决github每次push都要输入密码"
date: "2021-02-25"
tags: ["Git"]
categories: ["Git"]
ShowToc: true
TocOpen: true
---

# 解决github每次push都要输入密码

进入你的本地仓库目录
输入：
```bash
git config --global credential.helper store
```
然后再运行一遍```git pull```或```git push```就可以了
