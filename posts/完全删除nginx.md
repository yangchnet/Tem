---
title: "Ubuntu完全删除nginx"
date: 2021-01-22T17:27:21+08:00
draft: false
---
# Ubuntu完全删除nginx

## 1. 卸载nginx及相关软件

* 卸载nginx
```bash
sudo apt-get --purge remove nginx
```


* 移除全部无用包 
```bash
sudo apt-get autoremove
```


* 列出与nginx相关的软件
```bash
dpkg --get-selections | grep nginx
```


* 删除之
```bash
sudo apt-get --purge remove nginx-common
sudo apt-get --purge remove nginx-core
```

## 2. 停止所有与nginx有关的进程

* 查看相关进程
```bash
ps -ef | grep nginx
```


* 停止这些进程
```bash
sudo kill -9 {process_id}
```
> 00:00:00 grep --color=auto nginx 这个不是


## 3. 查找主机中与nginx相关的文件
使用命令：
```bash
sudo  find  /  -name  nginx*
```

删除之
```bash
sudo rm -rf {dir}
```

## 4. 现在可以重新安装nginx

```bash
sudo apt-get update
sudo apt-get install nginx
```
