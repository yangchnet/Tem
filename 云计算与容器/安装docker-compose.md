---
title: "安装docker-compose"
date: 2021-01-22T17:27:21+08:00
draft: false
---
# 安装docker-compose 

1. 下载最新版本的docker-compose
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```  
2. 对二进制文件应用可执行权限
```bash
sudo chmod +x /usr/local/bin/docker-compose
```
3. 测试安装完成
```bash
docker-compose --version
```
