---
author: "李昌"
title: "CentOS安装Python环境"
date: "2021-02-25"
tags: ["Linux"]
categories: ["Linux"]
ShowToc: true
TocOpen: true
---

# CentOS安装Python环境

吐槽：网上一堆从官网获取安装包然后自己编译的，慢不说，还容易出错

可使用以下命令安装Python3环境：  

1. 
```bash
yum install rh-python36
```
使用这条命令，安装Python3.6，但是安装后找不到，输入Python3后还是找不到命令

2. scl enable rh-python36 bash  
上面的命令是调用/opt/rh/rh-python36/enable更改shell环境变量的脚本。  
如果再次检查Python版本，你会发现Python 3.6现在是当前shell中的默认版本。
**需要指出的是，Python 3.6仅在此shell会话中设置为默认的Python版本。如果退出会话或从另一个终端打开一个新会话，Python 2.7将是默认的Python版本。**

3. 可使用当前shell窗口建立一个Python3虚拟环境，这样就可以使用Python3

```bash
#首先，创建项目目录并切换到它：
mkdir ~/my_new_project
cd ~/my_new_project
#使用该scl工具激活Python 3.6 ：
sl enable rh-python36 bash
# 从项目根目录内部运行以下命令以创建名为的虚拟环境my_project_venv：
python -m venv my_project_venv
#要首先使用虚拟环境，我们需要输入以下命令来激活它：
source my_project_venv/bin/activate
#激活环境后，shell提示符将以环境名称作为前缀：
(my_project_venv) user@host:~/my_new_project$
```
