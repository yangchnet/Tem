---
author: "李昌"
title: "Ubuntu18（WSL2）安装redis"
date: "2021-04-04"
tags: ["ubuntu", "install", "redis"]
categories: ["SQL"]
ShowToc: true
TocOpen: true
---

## 1. 安装并对redis进行配置
更新源并安装redis
```bash
sudo apt-get update
sudo apt-get install redis-server
```

**将redis设置为systemctl**  
```bash
sudo vim /etc/redis/redis.conf
```
找到`supervised`选项，设置为`systemd`
```bash
# If you run Redis from upstart or systemd, Redis can interact with your
# supervision tree. Options:
#   supervised no      - no supervision interaction
#   supervised upstart - signal upstart by putting Redis into SIGSTOP mode
#   supervised systemd - signal systemd by writing READY=1 to $NOTIFY_SOCKET
#   supervised auto    - detect upstart or systemd method based on
#                        UPSTART_JOB or NOTIFY_SOCKET environment variables
# Note: these supervision methods only signal "process is ready."
#       They do not enable continuous liveness pings back to your supervisor.
supervised systemd
```
开启redis服务  
```bash
sudo service redis-server start
```

## 2. 测试redis
查看redis运行状态
```bash
sudo service redis-server status
```

打开redis命令行
```bash
redis-cli
```

```bash
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> set test "It's a working"
OK
127.0.0.1:6379> get test
"It's a working"
127.0.0.1:6379>exit
```
退出后重新连接redis，再次访问`test`键
```bash
127.0.0.1:6379> get test
"It's a working"
127.0.0.1:6379>
```

## 3. 绑定到本地
一般来说，不推荐配置redis远程可访问，因此，这里只将其配置为本地可访问。   
打开Redis configuration file 
```bash
sudo vim /etc/redis/redis.conf
```
定位到`bind`,并将其配置如下
```bash
# By default, if no "bind" configuration directive is specified, Redis listens
# for connections from all the network interfaces available on the server.
# It is possible to listen to just one or multiple selected interfaces using
# the "bind" configuration directive, followed by one or more IP addresses.
#
# Examples:
#
# bind 192.168.1.100 10.0.0.1
bind 127.0.0.1 ::1   # 这里本来被注释掉，去掉注释即可
```
保存退出，重启redis
```bash
sudo service redis-server restart
```
查看此项配置是否生效
```bash
$ sudo netstat -lnp | grep redis
tcp        0      0 127.0.0.1:6379          0.0.0.0:*               LISTEN      2086/redis-server 1
tcp6       0      0 ::1:6379                :::*                    LISTEN      2086/redis-server 1
```

## 4. 为redis配置密码
打开redis配置文件，定位到`# requirepass foobared`
```bash
sudo vim /etc/redis/redis.conf
```
将`# requirepass foobared`注释去掉，把`foobared`改成你想要的密码。  
重启redis服务
```bash
sudo service redis-server restart
```
再次进入redis，尝试get
```bash
$ redis-cli
127.0.0.1:6379> get test
(error) NOAUTH Authentication required.
127.0.0.1:6379>
```
get失败，因为没有进行认证.
```bash
127.0.0.1:6379> auth your_redis_password
OK
127.0.0.1:6379> get test
"It's a working"
127.0.0.1:6379>
```


> 参考：[How To Install and Secure Redis on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-18-04)

