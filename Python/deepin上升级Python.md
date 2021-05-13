---
author: "李昌"
title: "deepin上升级Python"
date: "2021-05-13"
tags: ["python", "deepin", "升级"]
categories: ["Python"]
ShowToc: true
TocOpen: true
---

## 1. 安装高版本的Python
> 这里要说明，不能删除原来的python2以及python3.5，因为系统是依赖于这两个python版本的，当然你也可以试试，后果自负...

1. 去官网下载最新的Python
   我这里下载的是源码，因为没有对应的安装包。（[Python3.9](https://www.python.org/ftp/python/3.9.5/Python-3.9.5.tar.xz)）
2. 下载完成后解压到本地
   ```bash
   sudo tar -xvf Python-3.9.5.tar.xz -C /opt/python
   ```
3. 编译安装
   ```bash
   cd /opt/python
   mv Python-3.9.5 python3.9
   sudo ./configure --enable-optimizations  # 默认安装到/usr/local/bin, 可用--prefix指定安装目录
   make -j8 && sudo make altinstall
   sudo make clean
   ```
4. 验证安装成功
   ```bash
    /usr/local/bin/python3.9
   ```
   ```
       Python 3.9.5 (default, May 13 2021, 09:51:10) 
    [GCC 6.3.0 20170516] on linux
    Type "help", "copyright", "credits" or "license" for more information.
    >>> exit()
    ```
## 2. 设置默认Python版本

### 2.1 用户级修改
```bash
vim ~/.bashrc
```
增加一行：`alias python='/usr/local/bin/python3.9`
```bash
source ~/.bashrc
```

### 2.2 系统级修改
```bash
sudo rm /usr/bin/python   # 删除原来默认的Python软链接
sudo ln -s /usr/local/bin/python3.9 /usr/bin/python  # 设置新的软连接 从python3.9指向python
```

### 2.3 基于update-alternatives
列出所有可用的python版本替代信息
```
update-alternatives --list python
```
若出现`update-alternatives: 错误: 无 python 的候选项`，则说明update-alternatives没有添加Python的替代版本，需要手动添加  
```bash
sudo update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1

sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.5 2

sudo update-alternatives --install /usr/bin/python python /usr/local/bin/python3.9 3
```
`update-alternatives`的`--install`参数后跟4个参数，分别是`link name path priority`， `link` is the generic name for the master link, `name` is the name of its symlink in the alternatives directory, and `path` is the alternative being introduced for the master link。最后一个是优先级，数字越高优先级越高
再次使用`update-alternatives --list python`查看可用的版本替代信息

开始切换：
```
update-alternatives --config python
```

```
有 3 个候选项可用于替换 python (提供 /usr/bin/python)。

  选择       路径                    优先级  状态
------------------------------------------------------------
* 0            /usr/local/bin/python3.9   3         自动模式
  1            /usr/bin/python2.7         1         手动模式
  2            /usr/bin/python3.5         2         手动模式
  3            /usr/local/bin/python3.9   3         手动模式

要维持当前值[*]请按<回车键>，或者键入选择的编号：
```
输入3，选择Python3.9为默认值

## 3. pip错误
更换版本后，pip可能会出现错误。
在我的电脑上出现了`subprocess.CalledProcessError: Command '('lsb_release', '-a')' returned non-zero exit status 1.`错误，
解决方法为：删除`/usr/bin/lsb_release`
```bash
sudo rm /usr/bin/lsb_release
```

