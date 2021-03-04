---
author: "李昌"
title: "安装与配置clash"
date: "2021-03-04"
tags: ["clash", "Linux", "install"]
categories: ["Linux"]
ShowToc: true
TocOpen: true
---

# 安装与配置clash
> 参考了[这位老哥的博客](https://wh1te.fun/2020/11/02/%E9%98%BF%E9%87%8C%E4%BA%91Linux%E4%BD%BF%E7%94%A8Clash%E8%BF%9B%E8%A1%8C%E5%A4%96%E7%BD%91%E4%BB%A3%E7%90%86%E5%8A%A0%E9%80%9F/)

## 0. 来由
阿里云与腾讯云git太慢了。。想快点

## 1. 下载安装clash
地址在[这里](https://github.com/Dreamacro/clash)，找到对应自己系统的版本，可以先下载到自己本地主机后再用FileZilla上传到云服务器（虽然蛮麻烦，但是它快呀）

## 2. 安装clash
将下载的clash上传到自己的服务器之后，解压之：
```bash
gunzip clash-linux-amd64-v1.4.1.gz
```
解压结果就是一个可执行文件
重命名：
```bash
mv clash-linux-amd64 clash
```
赋予运行权限：
```bash
chmod +x clash
```
移动到bin目录下：
```bash
mv clash /usr/bin
```

## 3. 编辑config.yaml文件
config.yaml文件的内容来自你购买的vpn提供商
```bash
mkdir /etc/clash
touch /etc/clash/config.yaml
···
# 编辑你的config.yaml
```

## 4. 将clash添加为系统服务
```bash
cd /etc/systemd/system/
vim clash.service
```
clash.service的内容为：
```
[Unit]
Description=clash proxy
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/clash -d /etc/clash

[Install]
WantedBy=multi-user.target
```
> 要想深入了解systemctl服务，可前往[阮一峰大佬的教程](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)

编辑完成后，重载systemctl
```bash
systemctl daemon-reload
```
开启clash服务
```bash
systemctl start clash
```

## 5. 开启http代理和socks5代理
- 手动开启代理
  ```bash
  export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7891
  ```
- 保持启动，修改~/.bashrc
  ```bash
  echo "export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7891" >> ~/.bashrc
  ```

## 6. 测试是否成功
```bash
curl -I https://www.google.com/
```
返回：
```
HTTP/1.1 200 Connection established

HTTP/2 200
content-type: text/html; charset=ISO-8859-1
p3p: CP="This is not a P3P policy! See g.co/p3phelp for more info."
date: Thu, 04 Mar 2021 09:08:48 GMT
server: gws
x-xss-protection: 0
x-frame-options: SAMEORIGIN
expires: Thu, 04 Mar 2021 09:08:48 GMT
cache-control: private
set-cookie: 1P_JAR=2021-03-04-09; expires=Sat, 03-Apr-2021 09:08:48 GMT; path=/; domain=.google.com; Secure
set-cookie: NID=210=wzyYoXKPgWcDE4ptLfGQOeOE1w5kehSHm4-gycNKB2bUaeueNyGXgDUm-p5KR6X0aTtmcXjxsamD0xM3YJVRqJnw74uXjiOwcJpRiwsQ-jdUc4y4JG8ObdBOGQJQcddFf_d8fyJ1nVEeEWWZanQUDJToFA6b0T05oCegNpeM70Q; expires=Fri, 03-Sep-2021 09:08:48 GMT; path=/; domain=.google.com; HttpOnly
alt-svc: h3-29=":443"; ma=2592000,h3-T051=":443"; ma=2592000,h3-Q050=":443"; ma=2592000,h3-Q046=":443"; ma=2592000,h3-Q043=":443"; ma=2592000,quic=":443"; ma=2592000; v="46,43"
```
