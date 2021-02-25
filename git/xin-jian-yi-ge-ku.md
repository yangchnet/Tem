---
title: 怎样从本地现有代码新建一个库
date: '2021-01-22T09:27:21.000Z'
draft: false
---

# 怎样从本地现有代码新建一个库

1. 在本地代码目录初始化git

   ```bash
   git init
   ```

2. 然后将代码添加到仓库中

   ```bash
   git add .    #这里是添加了当前目录所有的文件
   ```

3. 提交代码到本地库中

   ```bash
   git commit -m "这里是提交的注释"
   ```

4. 接下来我们需要在github上新建一个库，比如说新库的名字为hello
5. 将本地的仓库关联到github上

   ```bash
   git remote add origin https://github.com/yourname/hello
   ```

6. 上传之前，先要拉取远程的相关信息

   ```bash
   git pull origin master
   ```

7. 最后上传

   ```bash
   git push -u origin master
   ```

## 遇到的问题

> 来自 [https://github.com/yangchnet/Mygolang](https://github.com/yangchnet/Mygolang)
>
> * branch            master     -&gt; FETCH\_HEAD
>
>   fatal: 拒绝合并无关的历史

解决方案 首先将远程仓库和本地仓库关联起来：

```bash
git branch --set-upstream-to=origin/master master
```

然后使用git pull整合远程仓库和本地仓库，

```bash
git pull --allow-unrelated-histories    (忽略版本不同造成的影响)
```

