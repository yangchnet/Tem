---
title: "WordPress安装踩坑"
date: 2021-01-22T17:27:21+08:00
draft: false
---
# WordPress安装踩坑

## 1. 第一个坑，忘了安装PHP。。。

## 2. 第二个坑，访问页面not found

发现是因为同时开了apache2和nginx,导致冲突了，把nginx关掉就好了

## 3. 第三个坑，打开页面全是源代码

1. 打开/etc/apache2/apache2.conf，将以下内容添加到文件的底部：
```conf
<FilesMatch \ .php $> 
SetHandler application / x-httpd-php 
</ FilesMatch>
```

2. 为了使PHP正常运行，您必须禁用mpm_event模块并启用mpm_prefork和php7模块。为此，请返回您的终端窗口并发出命令：
```bash
sudo a2dismod mpm_event && sudo a2enmod mpm_prefork && sudo a2enmod php7.0
```

## 4. 在执行上面的命令时，遇到了第四个坑

```bash
ERROR: Module php7.0 does not exist!
```

**解决办法**
```bash
sudo apt-get install libapache2-mod-php7.0
```

## 5. 您的PHP似乎没有安装运行WordPress所必需的MySQL扩展。

```bash
sudo apt-get install php-mysql
```

爬出来了。。
