---
author: "李昌"
title: "主机上设置两个git账号"
date: "2021-04-25"
tags: ["git", "muti user"]
categories: ["Git"]
ShowToc: true
TocOpen: true
---
> ubuntu环境

## 1. 使用SSH连接到GitHub
使用 SSH 协议可以连接远程服务器和服务并向它们验证。 利用 SSH 密钥可以连接 GitHub，而无需在每次访问时都提供用户名和个人访问令牌。  

#### 检查现有SSH秘钥
在生成 SSH 密钥之前，您可以检查是否有任何现有的 SSH 密钥。  
```sh
$ ls -al ~/.ssh
# Lists the files in your .ssh directory, if they exist
```
如果你的主机上已有SSH公钥，则其可能是如下：
```
id_rsa.pub
id_ecdsa.pub
id_ed25519.pub
```
如果你没有现有的公钥和私钥对，或者不想使用现有的秘钥连接到github，则可以生成新的SSH秘钥。

#### 生成新SSH秘钥
输入如下命令：
```sh
ssh-keygen -t ed25519 -C "your_email@example.com"
```
会有如下输出：
```
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/lc/.ssh/id_ed25519):  # 可以直接回车使用默认位置
Enter passphrase (empty for no passphrase): # 也可直接回车
```
然后就可以在你的`~/.ssh`目录下生成新的秘钥

#### 添加秘钥到ssh-agent
启动ssh-agent
```sh
eval "$(ssh-agent -s)"
# 输出： Agent pid 59566
```
添加
```sh
ssh-add ~/.ssh/id_ed25519
```

#### 新增 SSH 密钥到 GitHub 帐户
复制你的ssh秘钥（id_ed25519文件内容），打开github
![20210425191443](https://raw.githubusercontent.com/lich-Img/blogImg/master/img20210425191443.png)
点击`new ssh key`,将你的秘钥复制到对应的位置
![20210425191558](https://raw.githubusercontent.com/lich-Img/blogImg/master/img20210425191558.png)

#### 测试你的秘钥是否生效
```sh
ssh -T git@github.com
# Hi yenian!  You've successfully authenticated, but GitHub does not provide shell access.
```

## 2. 设置两个github账户
#### 取消已有的全局配置
```sh
git config --global --unset user.name
git config --global --unset user.email
```

#### 生成两个新的ssh秘钥
输入如下命令：
```sh
ssh-keygen -t ed25519 -C "your_email@example.com"
```
会有如下输出：
```
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/lc/.ssh/id_ed25519):  # 这里要为两个秘钥定义不同的文件名
Enter passphrase (empty for no passphrase): # 也可直接回车
```
**这里要注意的是设置文件名时不能直接跳过，要为两个秘钥定义不同的文件名，比如所一个为`id_ed25519_one`, 另一个为`id_ed25519_two` ，其中`one`可以设置你的git用户名。**

#### 将两个秘钥分别加入对应的github账号
...
如上
...

#### 配置 ~/.ssh/config 文件
打开`~/.ssh/config` 文件（没有则创建）
```sh
vim ~/.ssh/config
```
文件内容如下：
```
Host git@one.github.com
HostName github.com
User one # 你的用户名
IdentityFile ~/.ssh/id_ed25519_one

Host git@two.github.com
HostName github.com
User two
IdentityFile ~/.ssh/id_ed25519_two
```

#### 测试是否设置成功
```sh
ssh -T git@one.github.com
# Hi one! You've successfully authenticated, but GitHub does not provide shell access.
```