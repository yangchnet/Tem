---
author: "李昌"
title: "goget没反应"
date: "2021-02-25"
tags: ["GoLang"]
categories: ["GoLang"]
ShowToc: true
TocOpen: true
---

# go get 没反应

**修改hosts，然后reboot**
添加
```bash
192.30.253.112 github.com
151.101.185.194 github.global.ssl.fastly.net
```
到```/etc/hosts```
然后reboot

## go get golang.org

在使用```go get golang.org/...```时，总是```time out```（就算fp也一样，fp之后可以访问```golang.org```），不知道为啥。

幸好github上存在golang.org的镜像  
例如
```bash
go get -u golang.org/x/net
```
那么这个包的位置在github上就是```github.com/golang/net```, 所以，我们可以手动建立```golang.org/x/```目录，并切换到该目录下，然后使用
```bash
git clone https://github.com/golang/net.git
```
**注意：**要使用```git clone```命令，直接下载下来复制到目录下会提示找不到版本号。
