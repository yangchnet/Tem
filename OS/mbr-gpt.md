---
author: "李昌"
title: "Windows装系统遇到的问题"
date: "2021-02-25"
tags: ["Windows", "MBR", "GPT", "磁盘转换"]
categories: ["OS"]
ShowToc: true
TocOpen: true
---

# Windows装系统遇到的问题

### 1. 问题描述

windows无法安装到这个磁盘，选中的磁盘具有MBR分区表。在EFI系统上，Windows只能安装到GPT磁盘

### 2. 问题来源

这个问题的根源网上众说纷纭，暂且不管他

### 3. 解决办法

1. 首先选择U盘安装，进入安装界面

2. 按shift+F10打开命令行

3. 输入```diskpart```并回车

4. 输入```list disk```查看磁盘，一般会出现两个磁盘，一个是机器本身的磁盘，编号为0，另一个为U盘，编号为1

5. 输入```select disk x```（x为要选择的磁盘编号） cmd会提示当前选择的磁盘为x

6. 执行```clean```命令清除该磁盘上所有分区信息，并且会清空所有硬盘数据

7. 执行```convert gpt```，将该硬盘转化为GPT格式

8. 完成，继续安装系统
