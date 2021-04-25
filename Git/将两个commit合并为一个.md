---
author: "李昌"
title: "将两个commit合并为一个"
date: "2021-04-25"
tags: ["git", "合并commit"]
categories: ["Git"]
ShowToc: true
TocOpen: true
---

使用`git log`命令查看git日志文件，假设为如下内容
```
commit cc7b5fc7bd2ae6f8d88144cd61c8ffad15d44e41
Author: yangchnet <1048887414@qq.com>
Date:   Sun Apr 25 19:40:03 2021 +0800

    4-25

commit fbd7265095b4c8989fba830393eb32ef29cd9ee1
Merge: 3ae3c19 6a25204
Author: yangchnet <1048887414@qq.com>
Date:   Sun Apr 25 15:04:38 2021 +0800

    Merge branch 'master' of https://github.com/yangchnet/Tem

commit 6a25204187602449bfe4ca8c862c9677e65fed04
Author: yangchnet <30308940+yangchnet@users.noreply.github.com>
Date:   Thu Apr 22 21:36:05 2021 +0800

    Delete CNAME
...
```

现在想合并最后两个提交，则进行以下步骤：
1. 复制倒数第三个提交的哈希值，即：`6a25204187602449bfe4ca8c862c9677e65fed04`
2. 使用如下命令进行合并：
```sh
git rebase -i 6a25204187602449bfe4ca8c862c9677e65fed04 # 这个哈希值就是你刚才复制的
   ```
3. 若有如下提示，请进行第4步，否则直接进行第5步
   ```
不能变基：您有未暂存的变更。
请提交或为它们保存进度。
   ```
4. 使用`git stash`暂存修改
```sh
$ git stash                                             
保存工作目录和索引状态 WIP on master: cc7b5fc 4-25
HEAD 现在位于 cc7b5fc 4-25
```
5. 使用`git rebase`后，会出现如下内容
```
pick 3ae3c19 增加graphql介绍 
pick cc7b5fc 4-25 

# 变基 6a25204..cc7b5fc 到 6a25204（2 个提交） 
# 
# 命令: 
# p, pick = 使用提交 
# r, reword = 使用提交，但修改提交说明 
# e, edit = 使用提交，但停止以便进行提交修补 
# s, squash = 使用提交，但和前一个版本融合 
# f, fixup = 类似于 "squash"，但丢弃提交说明日志 
# x, exec = 使用 shell 运行命令（此行剩余部分） 
# d, drop = 删除提交 
# 
# 这些行可以被重新排序；它们会被从上至下地执行。 
# 
# 
# 如果您在这里删除一行，对应的提交将会丢失。 
# 
# 然而，如果您删除全部内容，变基操作将会终止。 
# 
# 注意空提交已被注释掉                                
```
前两行为你想要合并的commit，按照注释内容，要保留的commit开头不变，要合并到另一个的开头设为`s`，意为：`使用提交，但和前一个版本融合 `（可以按照你的需求改变），更改完成后保存。
6. 保存完成后会出现如下内容
```
# 这是一个 2 个提交的组合。 
# 这是第一个提交说明： 
 
增加graphql介绍 
 
# 这是提交说明 #2： 
 
4-25 
 
# 请为您的变更输入提交说明。以 '#' 开始的行将被忽略，而一个空的提交 
# 说明将会终止提交。
# ...
```
这是你的commit的说明，将你想保留的commit说明保留，不想要的直接删除。保存之。
7. 再次查看`git log`, 可以看到刚才的更改已经生效。
```
commit 2994a193577f4b4175b1fe7015db955df9143b89
Author: yangchnet <1048887414@qq.com>
Date:   Thu Apr 22 21:30:48 2021 +0800

    增加graphql介绍

commit 6a25204187602449bfe4ca8c862c9677e65fed04
Author: yangchnet <30308940+yangchnet@users.noreply.github.com>
Date:   Thu Apr 22 21:36:05 2021 +0800

    Delete CNAME
```
8. 若进行了第4步，则需进行本步骤
```sh
git stash pop
```


