---
title: Docker搭建Java环境
date: 2021-01-22T09:27:21.000Z
draft: false
---

# Docker 搭建Java环境

## 1. 拉取Ubuntu镜像

```bash
docker pull ubuntu
```

## 2. 运行docker镜像，并进入

```bash
docker run -it -v /home/hadoop/build:/root/build --name ubuntu ubuntu
```

> -i 表示Interaction，交互，开启交互模式  
> -t 表示分配一个tty，即控制台  
> -v 分配一个共享目录，  
> --name 为镜像命名

## 3. 基本操作

* 首先更新源，并安装vim

  ```bash
  apt-get update
  apt-get install vim
  ```

* 为保证后面的软件安装速度，进行换源  
  打开`/etc/apt/sources.list`，将其中的内容替换为：

  ```text
  deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
  ```

然后进行源更新

```bash
apt-get update
apt-get upgrade
```

## 4. 安装jdk

```bash
apt-get install openjdk-8-jdk
```

设置环境变量`vim ~/.bashrc`

```text
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/
export PATH=$PATH:$JAVA_HOME/bin
```

```text
source ~/.bashrc
```

## 5. 安装mysql

```bash
apt-get update
apt-get install mysql-server
```

安装完成后并不知道MySQL的初始密码，可在`/etc/mysql/debian.cnf`寻找

```text
user     = debian-sys-maint
password = xXI6LBYyXzeWAwYj
```

这就是mysql初始的用户与密码  
使用

```text
mysql -u debian-sys-maint -p
```

登录，其密码为xXI6LBYyXzeWAwYj

若出现`ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (2)`，则可能是mysql服务尚未打开，可使用`/etc/init.d/mysql start`打开，同时可将`/etc/init.d/mysql start`添加到`~/.bashrc`，以开机启动。

为mysql添加root用户

```sql
mysql>set password for 'root'@'localhost' = password('123456');
Query OK, 0 rows affected (0.00 sec)
```

更改root用户的访问位置

```text
use mysql;
update user set host = '%' where user = 'root';
```

改为%代表可在任何位置，原来是localhost，也就是只能本机访问

## 6. 安装tomcat

将下载好的tomcat安装包通过共享文件夹保存到docker容器的`/root/build`  
解压到`/usr/local`

```bash
tar -zxvf apache-tomcat-7.0.77.tar.gz -C /usr/local
cd /usr/local
mv apache-tomcat-7.0.77.tar.gz tomcat7
```

设置环境变量

```text
CATALINA_HOME=/usr/local/tomcat7
export CATALINA_HOME
```

设置tomcat

```text
vim  /usr/local/tomcat7/bin/catalina.sh
```

在其中增加

```bash
CATALINA_HOME=/usr/local/tomcat7
JAVA_HOME=/usr/local/java/jdk1.8.0_121  # 这个要根据自己的来
```

启动tomcat

```text
sudo ./bin/startup.sh
```

可将其写入~/.bashrc开机启动

