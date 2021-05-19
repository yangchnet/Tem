---
author: "李昌"
title: "Deepin15安装Anaconda"
date: "2021-05-19"
tags: ["python", "deepin", "anaconda", "install"]
categories: ["Python"]
ShowToc: true
TocOpen: true
---

## 1. Anaconda下载地址
1. 官方下载地址：https://www.anaconda.com/distribution/#linux
2. 清华大学镜像：https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/   # 选择你想安装的版本下载

## 2. 安装
```bash
$ bash Anaconda3-2021.05-Linux-x86_64.sh  # 中间会有一些选择，按照自己的意愿选择即可 
```
关闭并重新打开你的终端来激活conda
## 3. 使用
1. 更新自己
```bash
# 更新conda
conda update conda  
conda update anaconda
```
> 更新时出现了`ValueError: check_hostname requires server_hostname`错误，经查发现是代理的问题，可尝试关闭或开启代理再次尝试

2. 对包的操作
    - 更新包
    ```bash
    conda update --all  # 更新所有包
    ```

    - 安装包
    ```bash
    conda install [包名] 
    ```

3. 对环境的操作
    - 创建环境
    ```bash
    conda create --name [环境名字]  # 使用默认的Python版本
    ```
    - 激活环境
    ```bash
    conda activate [环境名字]
    ```
    - 退出环境
    ```bash
    conda deactivate
    ```
    - 查看环境名字
    ```bash
    conda env list  # conda info -e
    ```
    - 删除环境中某个包
    ```bash
    conda remove [环境名] [包名]
    ```
    - 修改环境名字
    ```bash
    conda create -n [新环境名] --clone [旧环境名]  # 克隆旧的
    conda remove -n [旧环境名] # 删除旧的
    ```
    - 删除环境
    ```bash
    conda remove -n [环境名] --all
    ```
## 4. 配置
```bash
#1、换源,添加清华源 方法一
vim ~/.condarc #添加以下内容
channels:
  - https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
  - https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
  - defaults
show_channel_urls: true
#2、换源,添加清华源 方法二
onda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes
#换回默认源
conda config --remove-key channels
```