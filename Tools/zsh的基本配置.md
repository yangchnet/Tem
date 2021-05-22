---
author: "李昌"
title: "zsh的基本配置"
date: "2021-05-20"
tags: ["zsh", "terminal", "美化", "tools", "效率工具"]
categories: ["Tools"]
ShowToc: true
TocOpen: true
---

## 1. 按照Oh my zsh
```bash
$ sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## 2. 配置Oh my zsh
1. 将zsh设置为默认Shell
    ```bash
    chsh -s /bin/zsh # 不需要使用root权限
    ```
2. 更换主题
   ```bash
   vim ~/.zshrc
   ```
   找到`ZSH_THEME='robbyrussell'`, 更换为你想要使用的主题，可以在[这里](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)找到你想要的主题
3. 安装插件
   ```bash
   vim ~/.zshrc
   ```
   找到`plugins=()`, 添加插件名称，我这里添加的插件有：
   ```
   plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
   ```
4. 完成
   ```bash
   source ~/.zshrc # 启动zsh
   ```
> 推荐主题`powerlevel10k`