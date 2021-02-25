---
author: "李昌"
title: "mysql中无法插入中文解决办法"
date: "2021-02-25"
tags: ["SQL"]
categories: ["SQL"]
ShowToc: true
TocOpen: true
---

# mysql 中无法插入中文解决办法

## alert database tuanplus character set utf8；
使用此方法可更改整个数据库，但是数据可能会丢失

##  alter table address convert to character set utf8；
使用此方法可更改某个表的编码
