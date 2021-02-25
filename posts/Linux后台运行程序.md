---
title: "Linux后台运行程序"
date: 2021-01-22T17:27:21+08:00
draft: false
---
# Linux后台运行程序

使用screen

## screen介绍 

Screen是一个控制台应用程序，允许您在一个窗口中使用多个终端会话。该程序在shell会话中运行，并充当其他终端会话的容器和管理器，类似于窗口管理器管理窗口的方式。

在许多情况下，创建多个终端窗口是不可能或不理想的。您可能需要在没有运行X服务器的情况下管理多个控制台会话，您可能需要轻松访问许多远程云服务器，或者您可能需要在处理其他任务时监视正在运行的程序的输出。所有需求都可以通过屏幕的强大功能轻松解决。

## 安装srceen 

ubuntu下   
```bash
sudo apt-get install screen
```  
manjaro下   
```bash
sudo pacman -S screen
```

## 基本使用方法 

使用```screen```命令打开一个新的窗口，在其中运行你想运行的脚本。   
开始运行后， 按```ctrl+ad```退出窗口   
使用```screen -r```重新进入窗口   

## 附：重定向 

|命令	|说明|  
|---|---|
|command > file|	将输出重定向到 file。|  
|command < file	|将输入重定向到 file。|  
|command >> file	|将输出以追加的方式重定向到 file。|  
|n > file	|将文件描述符为 n 的文件重定向到 file。|  
|n >> file	|将文件描述符为 n 的文件以追加的方式重定向到 file。|  
|n >& m	|将输出文件 m 和 n 合并。|  
|n <& m	|将输入文件 m 和 n 合并。|  
|<< tag	|将开始标记 tag 和结束标记 tag 之间的内容作为输入。|  

Edited by Li Chang
