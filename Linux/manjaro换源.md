---
author: "李昌"
title: "manjaro换源"
date: "2021-02-25"
tags: ["Linux"]
categories: ["Linux"]
ShowToc: true
TocOpen: true
---

# manjaro换源 

有关manjaro换源的文件：  
/etc/pacman.d/mirrorlist

网上教程：  
```bash
sudo pacman-mirrors -gb testing -c China  //选择中国源并更新
sudo pacman -Syyu  //更新系统
```

manjaro更新  

```bash
pacman -Sc //清空并且下载新数据
pacman-mirrors -gb testing -c China //更新源
or
pacman-mirrors -c China -g //更新源
pacman -Syu //更新
pacman -Syy //更新源数据库
pacman -Syyu //安装更新
```

