---
title: Docker基本命令
date: '2021-01-22T09:27:21.000Z'
draft: false
---

# Docker基本命令

* 启动docker服务  

  sudo systemctl start docker

* 查看本地镜像  

  sudo docker images

* 查看正在运行的镜像  

  sudo docker ps   

* 查看所有镜像  

  sudo docker ps -a 

* 停止正在运行的镜像  

  sudo docker stop container\_name

* 开始运行某个镜像  

  sudo docker start container\_name

* 删除某个镜像  

  sudo docker rmi container\_name

* 进入某个正在运行的镜像  

  sudo docker attach container\_name

* 导出容器  

  sudo docker export container\_id &gt; name.tar

* 导入容器  

  cat name.tar \| sudo docker import -test/buntu:v1.0

* 从网络导入  

  sudo docker import [http://example.com/exampleimage.tgz](http://example.com/exampleimage.tgz) example/imagerepo

```python

```

