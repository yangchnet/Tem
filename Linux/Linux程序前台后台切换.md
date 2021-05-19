---
author: "李昌"
title: "Linux程序前台后台切换"
date: "2021-05-18"
tags: ["Linux", "command"]
categories: ["Linux"]
ShowToc: true
TocOpen: true
---

## 1. 前后台切换
1. 在Linux终端运行命令的时候，在命令末尾加上 & 符号，就可以让程序在后台运行
```bash
$ ./main &
```
2. 如果程序正在前台运行，可以使用 Ctrl+z 选项把程序暂停，然后用 bg %[number] 命令把这个程序放到后台运行，摁Ctrl+z，然后在最后一行加上`bg %number`

3. 对于所有运行的程序，我们可以用`jobs –l` 指令查看
```bash
$ jobs -l
```

4. 也可以用 fg %[number] 指令把一个程序掉到前台

```bash 
$ fg %1
```

5. 也可以直接终止后台运行的程序，使用 kill 命令
```bash
$ kill %1
```

## 2. fg、bg、jobs、&、nohup、ctrl+z、ctrl+c 命令

1. &
加在一个命令的最后，可以把这个命令放到后台执行，如
```bash
watch -n 10 sh test.sh & #每10s在后台执行一次test.sh脚本
```
2. ctrl + z
可以将一个正在前台执行的命令放到后台，并且处于暂停状态。

3. jobs
查看当前有多少在后台运行的命令
`jobs -l`选项可显示所有任务的PID，jobs的状态可以是running, stopped, Terminated。但是如果任务被终止了（kill），shell 从当前的shell环境已知的列表中删除任务的进程标识。

4. fg
将后台中的命令调至前台继续运行。如果后台中有多个命令，可以用`fg %jobnumber`（是命令编号，不是进程号）将选中的命令调出。

5. bg
将一个在后台暂停的命令，变成在后台继续执行。如果后台中有多个命令，可以用`bg %jobnumber`将选中的命令调出。

6. kill
- 通过jobs命令查看job号（假设为num），然后执行`kill %num`
- 通过ps命令查看job的进程号（PID，假设为pid），然后执行`kill pid`
前台进程的终止：Ctrl+c

7. nohup
如果让程序始终在后台执行，即使关闭当前的终端也执行（之前的&做不到），这时候需要`nohup`。该命令可以在你退出帐户/关闭终端之后继续运行相应的进程。关闭中断后，在另一个终端jobs已经无法看到后台跑得程序了，此时利用ps（进程查看命令）
```bash
$ ps -aux | grep “test.sh” #a:显示所有程序 u:以用户为主的格式来显示 x:显示所有程序，不以终端机来区分
```

8. 查看nohup.out的日志
```bash
$ tail -fn 50 nohup.out
```

> 转自：[Linux程序前台后台切换](https://www.cnblogs.com/geogre123/p/10643152.html)