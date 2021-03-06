---
author: "李昌"
title: "GOPATH的正确打开方式"
date: "2021-02-25"
tags: ["GoLang"]
categories: ["GoLang"]
ShowToc: true
TocOpen: true
---

# GOPATH的正确打开方式

> GOPATH环境变量用于指定```$GOROOT```以外的目录，其中包含Go项目及其二进制文件的源。

```GOPATH```指定了go项目的位置及其编译得到的二进制文件的位置。```GOPAHT```可以是一个列表，也就是说可以设置多个```GOPATH```。

## 1.如何查看```GOPATH```     

使用
```bash
go env
```
命令查看go相关的环境变量

## 2. ```GOPATH```的目录结构

```GOPATH```的目录结构如下：  
```
.
├── bin
├── pkg
└── src
  └── main
    └── main.go
```

**\$GOPATH/bin**  
这个目录下存储编译的二进制文件，Linux操作系统使用```\$PATH```环境变量来查找无需完整路径即可执行的二进制应用程序，我们可以将这个目录添加到```$PATH```中，这样就可以方便的执行我们的二进制文件。

**\$GOPATH/pkg**  
Go在该目录中存储了预编译的目标文件，以加快程序的后续编译速度。通常，我们不需要访问该目录。但是如果在编译时遇到问题，可以安全地删除该目录，然后Go会重建它。

**\$GOPATH/src**  
这里是我们的代码所在位置。

## 3. 我应该如何设置```GOPATH```

* 设置一个`GOPATH`  
当设置一个`GOPAHT`时，我们自行安装的一些包以及我们所有的项目文件，都会在`\$GOPATH/src`目录下，未免显得太过拥挤。不推荐。

* 设置两个`GOPATH`  
可以设置两个`GOPATH`，其中第一个作为从网络获取的包所在位置，第二个存放我们自己的项目，在从网络获取包时，将自动安装在第一个`GOPATH`下，这样就避免了我们自己的代码和网络上获取的代码挤在一起

* 每个项目设置一个`GOPATH`  
每个项目设置一个`GOPATH`，可以在建立项目的目录结构后使用
```bash
export GOPATH=`pwd`
```
来添加当前目录到`GOPATH`，但是下一次在运行这个命令时会将上一次的覆盖掉。这个方法的优点在于每个项目的目录结构变得清晰了，不再是所有项目存在于一个src文件夹下，可以建立在任意位置。但缺点在于每次切换项目都要重新export，同时还需要重新安装一些必要的包。类似于Python中的virtualenv.


```go

```
