---
author: "李昌"
title: "docker常用操作"
date: "2021-02-25"
tags: ["docker"]
categories: ["云计算与容器"]
ShowToc: true
TocOpen: true
---
# docker常用操作
* 启动docker服务
sudo systemctl start docker
* 查看本地镜像
sudo docker images
* 查看正在运行的镜像
sudo docker ps 
* 查看所有镜像
sudo docker ps -a 
* 停止正在运行的镜像
sudo docker stop container_name
* 开始运行某个镜像
sudo docker start container_name
* 删除某个镜像
sudo docker rmi container_name
* 进入某个正在运行的镜像
sudo docker attach container_name
* 导出容器
sudo docker export container_id > name.tar
* 导入容器
cat name.tar | sudo docker import -test/buntu:v1.0
* 从网络导入
sudo docker import http://example.com/exampleimage.tgz example/imagerepo
