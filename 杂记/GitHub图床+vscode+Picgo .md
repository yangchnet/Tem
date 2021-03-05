---
author: "李昌"
title: "GitHub图床+vscode+Picgo "
date: "2021-03-05"
tags: ["图床", "github", "vscode", "Picgo", "tools"]
categories: ["杂记"]
ShowToc: true
TocOpen: true
---
# GitHub图床+vscode+Picgo 

## 0. 来由
用markdown写博客的时候，图片往哪里存地干活？图床里存···

## 1. GitHub配置
1. **创建图床仓库**
为了不污染我原来的git账号，我决定新建一个git账号，专门用作图床账号。
新建账号之后，new一个repo，啥都不用点，直接create。

2. **生成token**
   点击你GitHub页面右上角的头像，点击settings
   ![20210305101427](https://raw.githubusercontent.com/lich-Img/blogImg/master/img/20210305101427.png)
   在页面左侧找到`Developer settings`，选择之，再找到`Personal access tokens`，再选择之，然后`generate new tokens`
   ![20210305101744](https://raw.githubusercontent.com/lich-Img/blogImg/master/img/20210305101744.png)
   在新弹出的页面中填写note，并选择repo，
   ![20210305101942](https://raw.githubusercontent.com/lich-Img/blogImg/master/img/20210305101942.png)
   然后直接到最下面，`Generate token`
   这样GitHub会为你生成一个token（只会出现这一次），复制它留用。
   ![20210305102210](https://raw.githubusercontent.com/lich-Img/blogImg/master/img/20210305102210.png)

## 2. 配置VScode中的Picgo插件
在vscode的插件商店中直接搜索Picgo，然后点击安装
![20210305102533](https://raw.githubusercontent.com/lich-Img/blogImg/master/img/20210305102533.png)
安装完成后，再来配置你的Picgo
File>Preferences>settings>Entensions>Picgo找到配置picgo的位置，填写必要的信息
![20210305102916](https://raw.githubusercontent.com/lich-Img/blogImg/master/img/20210305102916.png)
```json
    "picgo.picBed.current": "github",
    "picgo.picBed.github.branch": "master",
    "picgo.picBed.github.path": "",    # 你想要图片存储的路径
    "picgo.picBed.github.repo": "",    # 你的用户名以及repo名，user/REPO_name
    "picgo.picBed.github.token": ""    # 刚才复制的token，粘贴到这里
```

## 3. 使用picgo上传图片
截个图并复制到剪贴板，在vscode里按下"CTRL+ALT+u"，图片就可以十分迅速的上传到你配置的GitHub仓库并为你返回图片链接
（￣︶￣）↗。

END

