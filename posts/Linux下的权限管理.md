---
title: "Linux下的权限管理"
date: 2021-01-22T17:27:21+08:00
draft: false
---
# Linux下的权限管理

**Linux 系统中为什么需要设定不同的权限，所有用户都直接使用管理员（root）身份不好吗？**
> 由于绝大多数用户使用的是个人计算机，使用者一般都是被信任的人（如家人、朋友等）。在这种情况下，大家都可以使用管理员身份直接登录。但在服务器上就不是这种情况了，往往运行的数据越重要（如游戏数据），价值越高（如电子商城数据、银行数据），则服务器中对权限的设定就要越详细，用户的分级也要越明确。  
和 Windows 系统不同，Linux 系统为每个文件都添加了很多的属性，最大的作用就是维护数据的安全。举个简单的例子，在你的 Linux 系统中，和系统服务相关的文件通常只有 root 用户才能读或写，就拿 /etc/shadow 这个文件来说，此文件记录了系统中所有用户的密码数据，非常重要，因此绝不能让任何人读取（否则密码数据会被窃取），只有 root 才可以有读取权限。
此外，如果你有一个软件开发团队，你希望团队中的每个人都可以使用某一些目录下的文件，而非团队的其他人则不予以开放。通过前面章节的学习我们知道，只需要将团队中的所有人加入新的群组，并赋予此群组读写目录的权限，即可实现要求。反之，如果你的目录权限没有做好，就很难防止其他人在你的系统中乱搞。
比如说，本来 root 用户才能做的开关机、ADSL 拨接程序，新增或删除用户等命令，一旦允许任何人拥有这些权限，系统很可能会经常莫名其妙的挂掉。而且，万一 root 用户的密码被其他人获取，他们就可以登录你的系统，从事一些只有 root 用户才能执行的操作，这是绝对不允许发生的。
因此，在服务器上，绝对不是所有的用户都使用 root 身份登录，而要根据不同的工作需要和职位需要，合理分配用户等级和权限等级。  

Linux 系统中，文件或目录的权限信息，可以使用 ls 命令查看，例如：
```bash
[root@localhost ~]# ls -al
total 156
drwxr-x---.   4    root   root     4096   Sep  8 14:06 .
drwxr-xr-x.  23    root   root     4096   Sep  8 14:21 ..
-rw-------.   1    root   root     1474   Sep  4 18:27 anaconda-ks.cfg
-rw-------.   1    root   root      199   Sep  8 17:14 .bash_history
-rw-r--r--.   1    root   root       24   Jan  6  2007 .bash_logout
```

## 1.  Linux chgrp命令：修改文件和目录的所属组

chgrp 命令用于修改文件（或目录）的所属组。
为了方便初学者记忆，可以将 chgrp 理解为是 "change group" 的缩写。

chgrp 命令的用法很简单，其基本格式为：
```bash
[root@localhost ~]# chgrp [-R] 所属组 文件名（目录名）
```

-R（注意是大写）选项长作用于更改目录的所属组，表示更改连同子目录中所有文件的所属组信息。

使用此命令需要注意的一点是，要被改变的群组名必须是真实存在的，否则命令无法正确执行，会提示 "invaild group name"。

举个例子，当以 root 身份登录 Linux 系统时，主目录中会存在一个名为 install.log 的文件，我们可以使用如下方法修改此文件的所属组：
```bash
[root@localhost ~]# groupadd group1
#新建用于测试的群组 group1
[root@localhost ~]# chgrp group1 install.log
#修改install.log文件的所属组为group1
[root@localhost ~]# ll install.log
-rw-r--r--. 1 root group1 78495 Nov 17 05:54 install.log
#修改生效
[root@localhost ~]# chgrp testgroup install.log
chgrp: invaild group name 'testgroup'
```
可以看到，在具有 group1 群组的前提下，我们成功修改了 install.log 文件的所属组，但我们再次试图将所属组修改为 testgroup 时，命令执行失败，就是因为系统的 /etc/group 文件中，没有 testgroup 群组。

## 2.  Linux chown命令：修改文件和目录的所有者和所属组

chown 命令，可以认为是 "change owner" 的缩写，主要用于修改文件（或目录）的所有者，除此之外，这个命令也可以修改文件（或目录）的所属组。

当只需要修改所有者时，可使用如下 chown 命令的基本格式：
```bash
[root@localhost ~]# chown [-R] 所有者 文件或目录
```
-R（注意大写）选项表示连同子目录中的所有文件，都更改所有者。

如果需要同时更改所有者和所属组，chown 命令的基本格式为：
```bash
[root@localhost ~]# chown [-R] 所有者:所属组 文件或目录
```
注意，在 chown 命令中，所有者和所属组中间也可以使用点（.），但会产生一个问题，如果用户在设定账号时加入了小数点（例如 zhangsan.temp），就会造成系统误判。因此，建议大家使用冒号连接所有者和所属组。

当然，chown 命令也支持单纯的修改文件或目录的所属组，例如 ```chown :group install.log``` 就表示修改 install.log 文件的所属组，但修改所属组通常使用 chgrp 命令，因此并不推荐大家使用 chown 命令。

另外需要注意的一点是，使用 chown 命令修改文件或目录的所有者（或所属者）时，要保证使用者用户（或用户组）存在，否则该命令无法正确执行，会提示 "invalid user" 或者 "invaild group"。

【例 1】  
其实，修改文件的所有者，更多时候是为了得到更高的权限，举一个实例：
```bash
[root@localhost ~]# touch file
#由root用户创建file文件
[root@localhost ~]# ll file
-rw-r--r--. 1 root root 0 Apr 17 05:12 file
#文件的所有者是root，普通用户user对这个文件拥有只读权限
[root@localhost ~]# chown user file
#修改文件的所有者
[root@localhost ~]# ll file
-rw-r--r--. 1 user root 0 Apr 17 05:12 file
#所有者变成了user用户，这时user用户对这个文件就拥有了读、写权限
```
可以看到，通过修改 file 文件的所有者，user 用户从其他人身份（只对此文件有读取权限）转变成了所有者身份，对此文件拥有读和写权限。

【例 2】  
Linux 系统中，用户等级权限的划分是非常清楚的，root 用户拥有最高权限，可以修改任何文件的权限，而普通用户只能修改自己文件的权限（所有者是自己的文件），例如：
```bash
[root@localhost ~]# cd /home/user
#进入user用户的家目录
[root@localhost user]# touch test
#由root用户新建文件test
[root@localhost user]# ll test
-rw-r--r--. 1 root root 0 Apr 17 05:37 test
#文件所有者和所属组都是root用户
[root@localhost user]# su - user
#切换为user用户
[user@localhost ~]$ chmod 755 test
chmod:更改"test"的权限：不允许的操作 #user用户不能修改test文件的权限
[user@localhost ~]$ exit
#退回到root身份
[root@localhost user]# chown user test
#由root用户把test文件的所有者改为user用户
[root@localhost user]# su - user
#切换为user用户
[user@localhost ~]$ chmod 755 test
#user用户由于是test文件的所有者，所以可以修改文件的权限
[user@localhost ~]$ ll test
-rwxr-xr-x. 1 user root 0 Apr 17 05:37 test
#查看权限
```
可以看到，user 用户无权更改所有者为 root 用户文件的权限，只有普通用户是这个文件的所有者，才可以修改文件的权限。

【例 3】  
```bash
[root@localhost ~]# chown user:group file
[root@localhost ~]# ll file
-rw-r--r--. 1 user group 0 Apr 17 05:12 file
```

## 3.  Linux权限位

Linux 系统，最常见的文件权限有 3 种，即对文件的读（用 r 表示）、写（用 w 表示）和执行（用 x 表示，针对可执行文件或目录）权限。在 Linux 系统中，每个文件都明确规定了不同身份用户的访问权限，通过 ls 命令即可看到。
除此之外，我们有时会看到 s（针对可执行文件或目录，使文件在执行阶段，临时拥有文件所有者的权限）和 t（针对目录，任何用户都可以在此目录中创建文件，但只能删除自己的文件），文件设置 s 和 t 权限，会占用 x 权限的位置。

例如，我们以 root 的身份登陆 Linux，并执行如下指令：
```bash
[root@localhost ~]# ls -al
total 156
drwxr-x---.   4    root   root     4096   Sep  8 14:06 .
drwxr-xr-x.  23    root   root     4096   Sep  8 14:21 ..
-rw-------.   1    root   root     1474   Sep  4 18:27 anaconda-ks.cfg
-rw-------.   1    root   root      199   Sep  8 17:14 .bash_history
-rw-r--r--.   1    root   root       24   Jan  6  2007 .bash_logout
...
```
可以看到，每行的第一列表示的就是各文件针对不同用户设定的权限，一共 11 位，但第 1 位用于表示文件的具体类型，最后一位此文件受 SELinux 的安全规则管理，不是本节关心的内容，放到后续章节做详细介绍。

因此，为文件设定不同用户的读、写和执行权限，仅涉及到 9 位字符，以 ls 命令输出信息中的 .bash_logout 文件为例，设定不同用户的访问权限是 rw-r--r--，Linux 将访问文件的用户分为 3 类，分别是文件的所有者，所属组（也就是文件所属的群组）以及其他人。除了所有者，以及所属群组中的用户可以访问文件外，其他用户（其他群组中的用户）也可以访问文件，这部分用户都归为其他人范畴.![](http://c.biancheng.net/uploads/allimg/190417/2-1Z41G11439421.gif)  
很显然，Linux 系统为 3 种不同的用户身份，分别规定了是否对文件有读、写和执行权限。拿图 1 来说，文件所有者拥有对文件的读和写权限，但是没有执行权限；所属群组中的用户只拥有读权限，也就是说，这部分用户只能访问文件，无法修改文件；其他人也是只能访问文件。

Linux 系统中，多数文件的文件所有者和所属群组都是 root（都是 root 账户创建的），这也就是为什么，root 用户是超级管理员，权限足够大的原因。

## 4.  Linux读写执行权限（-r、-w、-x）的真正含义

**rwx 权限对文件的作用**  
文件，是系统中用来存储数据的，包括普通的文本文件、数据库文件、二进制可执行文件，等等。不同的权限对文件的含义如表 1 所示。  

|rwx权限|对文件的作用|  
|-----|-----|  
|读权限（r）|表示可读取此文件中的实际内容，例如，可以对文件执行 cat、more、less、head、tail 等文件查看命令。|  
|写权限（w）	|表示可以编辑、新增或者修改文件中的内容，例如，可以对文件执行 vim、echo 等修改文件数据的命令。注意，无权限不赋予用户删除文件的权利，除非用户对文件的上级目录拥有写权限才可以。| 
|执行权限（x）	|表示该文件具有被系统执行的权限。Window系统中查看一个文件是否为可执行文件，是通过扩展名（.exe、.bat 等），但在 Linux 系统中，文件是否能被执行，是通过看此文件是否具有 x 权限来决定的。也就是说，只要文件拥有 x 权限，则此文件就是可执行文件。但是，文件到底能够正确运行，还要看文件中的代码是否正确。|  

**rwx 权限对目录的作用**
目录，主要用来记录文件名列表，不同的权限对目录的作用如表 2 所示。    

|rwx权限|对目录的作用|  
|-----|-----|  
|读权限（r）|表示具有读取目录结构列表的权限，也就是说，可以看到目录中有哪些文件和子目录。一旦对目录拥有 r 权限，就可以在此目录下执行 ls 命令，查看目录中的内容。|  
|写权限（w）| 对于目录来说，w 权限是最高权限。对目录拥有 w 权限，表示可以对目录做以下操作：            
* 在此目录中建立新的文件或子目录；
* 删除已存在的文件和目录（无论子文件或子目录的权限是怎样的）；
* 对已存在的文件或目录做更名操作；
* 移动此目录下的文件和目录的位置.
一旦对目录拥有 w 权限，就可以在目录下执行 touch、rm、cp、mv 等命令。|  
|执行权限（x）|目录是不能直接运行的，对目录赋予 x 权限，代表用户可以进入目录，也就是说，赋予 x 权限的用户或群组可以使用 cd 命令。|  

对目录来说，如果只赋予 r 权限，则此目录是无法使用的。很简单，只有 r 权限的目录，用户只能查看目录结构，根本无法进入目录（需要用 x 权限），更不用说使用了。

因此，对于目录来说，常用来设定目录的权限其实只有 0（---）、5（r-x）、7（rwx）这 3 种。  
#### 【例 1】
某目录的权限如下所示：
```bash
drwxr--r--.  3  root  root  4096  Jun 25 08:35   .ssh
```
**系统有个账号名称为 vbird，此账户并不包含在 root 群组中，请问 vbird 对这个目录有何权限？是否可切换到此目录中？**
> 答案：vbird 对此目录仅具有 r 的权限，因此 vbird 可以查询此目录下的文件名列表。因为 vbird 不具有 x 的权限，因此 vbird 并不能切换到此目录内。


#### 【例 2】
**假设有个账号名称为dmtsai，他的家目录在/home/dmtsai/，dmtsai对此目录具有[rwx]的权限。若在此目录下有个名为 the_root.data 的文件，该文件的权限如下：**
```
-rwx------. 1 root  root  4365 Sep 19 23:20  the_root.data
```
**请问 dmtsai 对此文件的权限为何？可否删除此文件？**
> 答案：由于 dmtsai 对此文件来说是其他人的身份，因此这个文件他无法读、无法编辑也无法执行，也就是说，他无法变动这个文件的内容就是了。但是由于这个文件在他的家目录下，他在此目录下具有 rwx 的完整权限，因此对于 the_root.data 这个文件来说，是能够删除的。

## 5.  Linux chmod命令：修改文件或目录的权限

**既然我们已经知道文件权限对于一个系统的重要性，也知道每个文件都设定了针对不同用户的访问权限，那么，是否可以手动修改文件的访问权限呢？**

可以，通过 chmod 命令即可。chmod 命令设定文件权限的方式有 2 种，分别可以使用数字或者符号来进行权限的变更。
chmod命令使用数字修改文件权限
Linux 系统中，文件的基本权限由 9 个字符组成，以 rwxrw-r-x 为例，我们可以使用数字来代表各个权限，各个权限与数字的对应关系如下：
* r --> 4
* w --> 2
* x --> 1

由于这 9 个字符分属 3 类用户，因此每种用户身份包含 3 个权限（r、w、x），通过将 3 个权限对应的数字累加，最终得到的值即可作为每种用户所具有的权限。

拿 rwxrw-r-x 来说，所有者、所属组和其他人分别对应的权限值为：
* 所有者 = rwx = 4+2+1 = 7
* 所属组 = rw- = 4+2 = 6
* 其他人 = r-x = 4+1 = 5

所以，此权限对应的权限值就是 765。

使用数字修改文件权限的 chmod 命令基本格式为：
```bash
[root@localhost ~]# chmod [-R] 权限值 文件名
```
-R（注意是大写）选项表示连同子目录中的所有文件，也都修改设定的权限。

例如，使用如下命令，即可完成对 .bashrc 目录文件的权限修改：
```bash
[root@localhost ~]# ls -al .bashrc
-rw-r--r--. 1 root root 176 Sep 22 2004 .bashrc
[root@localhost ~]# chmod 777 .bashrc
[root@localhost ~]# ls -al .bashrc
-rwxrwxrwx. 1 root root 176 Sep 22 2004 .bashrc
```

再举个例子，通常我们以 Vim 编辑 Shell 文件批处理文件后，文件权限通常是 rw-rw-r--（644），那么，如果要将该文件变成可执行文件，并且不让其他人修改此文件，则只需将此文件的权限该为 ```rwxr-xr-x（755）```即可。
chmod命令使用字母修改文件权限
既然文件的基本权限就是 3 种用户身份（所有者、所属组和其他人）搭配 3 种权限（rwx），chmod 命令中用 u、g、o 分别代表 3 种身份，还用 a 表示全部的身份（all 的缩写）。另外，chmod 命令仍使用 r、w、x 分别表示读、写、执行权限。

使用字母修改文件权限的 chmod 命令，其基本格式如图 1 所示。![](http://c.biancheng.net/uploads/allimg/190417/2-1Z41G31209649.gif)

chmod 命令基本格式
图 1 chmod 命令基本格式

例如，如果我们要设定 .bashrc 文件的权限为 rwxr-xr-x，则可执行如下命令：
```bash
[root@localhost ~]# chmod u=rwx,go=rx .bashrc
[root@localhost ~]# ls -al .bashrc
-rwxr-xr-x. 1 root root 176 Sep 22 2004 .bashrc
```

再举个例子，如果想要增加 .bashrc 文件的每种用户都可做写操作的权限，可以使用如下命令：
```bash
[root@localhost ~]# ls -al .bashrc
-rwxr-xr-x. 1 root root 176 Sep 22 2004 .bashrc
[root@localhost ~]# chmod a+w .bashrc
[root@localhost ~]# ls -al .bashrc
-rwxrwxrwx. 1 root root 176 Sep 22 2004 .bashrc
```

## 6.  Linux umask详解：令新建文件和目录拥有默认权限

Linux 是注重安全性的操作系统，而安全的基础在于对权限的设定，不仅所有已存在的文件和目录要设定必要的访问权限，创建新的文件和目录时，也要设定必要的初始权限。

Windows 系统中，新建的文件和目录时通过继承上级目录的权限获得的初始权限，而 Linux 不同，它是通过使用 umask 默认权限来给所有新建的文件和目录赋予初始权限的。

那么，我们如何得知 umask 默认权限的值呢？直接通过 umask 命令即可：
```
[root@localhost ~]# umask
0022
#root用户默认是0022，普通用户默认是 0002
```

大家可能会问，不应该只有 3 个数字（分别对应 3 种用户身份）吗，为什么有 4 个？ umask 默认权限确实由 4 个八进制数组成，但第 1 个数代表的是文件所具有的特殊权限（SetUID、SetGID、Sticky BIT），此部分内容放到后续章节中讲解，现在先不讨论。也就是说，后 3 位数字 "022" 才是本节真正要用到的 umask 权限值，将其转变为字母形式为 ----w--w-。

注意，虽然 umask 默认权限是用来设定文件或目录的初始权限，但并不是直接将 umask 默认权限作为文件或目录的初始权限，还要对其进行 "再加工"。

文件和目录的真正初始权限，可通过以下的计算得到：
文件（或目录）的初始权限 = 文件（或目录）的最大默认权限 - umask权限

如果按照官方的标准算法，需要将 umask 默认权限使用二进制并经过逻辑与和逻辑非运算后，才能得到最终文件或目录的初始权限，计算过程比较复杂，且容易出错，因此本节给大家介绍了更简单的计算方式。

显然，如果想最终得到文件或目录的初始权限值，我们还需要了解文件和目录的最大默认权限值。在 Linux 系统中，文件和目录的最大默认权限是不一样的：
对文件来讲，其可拥有的最大默认权限是 666，即 rw-rw-rw-。也就是说，使用文件的任何用户都没有执行（x）权限。原因很简单，执行权限是文件的最高权限，赋予时绝对要慎重，因此绝不能在新建文件的时候就默认赋予，只能通过用户手工赋予。
对目录来讲，其可拥有的最大默认权限是 777，即 rwxrwxrwx。

接下来，我们利用字母权限的方式计算文件或目录的初始权限。以 umask 值为 022 为例，分别计算新建文件和目录的初始权限：
文件的最大默认权限是 666，换算成字母就是 "-rw-rw-rw-"，umask 的值是 022，换算成字母为 "-----w--w-"。把两个字母权限相减，得到 (-rw-rw-rw-) - (-----w--w-) = (-rw-r--r--)，这就是新建文件的初始权限。我们测试一下：
```bash
[root@localhost ~]# umask
0022
#默认umask的值是0022
[root@localhost ~]# touch file  <--新建file空文件
[root@localhost ~]# ll -d file
-rw-r--r--. 1 root root 0 Apr 18 02:36 file
```
目录的默认权限最大可以是 777，换算成字母就是 "drwxrwxrwx"，umask 的值是 022，也就是 "-----w--w-"。把两个字母权限相减，得到的就是新建目录的默认权限，即 (drwxrwxrwx) - (-----w--w-) = (drwxr-xr-x)。我们再来测试一下：
```bash
[root@localhost ~]# umask
0022
[root@localhost ~]# mkdir catalog  <--新建catalog目录
[root@localhost ~]# ll -d catalog
drwxr-xr-x. 2 root root 4096 Apr 18 02:36 catalog
```

注意，在计算文件或目录的初始权限时，不能直接使用最大默认权限和 umask 权限的数字形式做减法，这是不对的。例如，若 umask 默认权限的值为 033，按照数字形式计算文件的初始权限，666-033=633，但我们按照字母的形式计算会得到 （rw-rw-rw-) - (----wx-wx) = (rw-r--r--)，换算成数字形式是 644。
这里的减法，其实是“遮盖”的意思，也就是说，最大默认权限中和 umask 权限公共的部分，通过减法运算会被遮盖掉，最终剩下的“最大默认权限”，才是最终赋予文件或目录的初始权限。

umask默认权限的修改方法
umask 权限值可以通过如下命令直接修改：
```bash
[root@localhost ~]# umask 002
[root@localhost ~]# umask
0002
[root@localhost ~]# umask 033
[root@localhost ~]# umask
0033
```

不过，这种方式修改的 umask 只是临时有效，一旦重启或重新登陆系统，就会失效。如果想让修改永久生效，则需要修改对应的环境变量配置文件 /etc/profile。例如：
```bash
[root@localhost ~]# vim /etc/profile
...省略部分内容...
if [ $UID -gt 199]&&[ "'id -gn'" = "'id -un'" ]; then
    umask 002
    #如果UID大于199（普通用户），则使用此umask值
else
    umask 022
    #如果UID小于199（超级用户），则使用此umask值
fi
...省略部分内容...
```
这是一段 Shell 脚本程序，不懂也没关系，大家只需要知道，普通用户的 umask 由 if 语句的第一段定义，而超级用户 root 的 umask 值由 else 语句定义即可。 修改此文件，则 umask 值就会永久生效。

## 7.  ACL权限是什么，Linux ACL访问控制权限（包含开启方式）

Linux 系统传统的权限控制方式，无非是利用 3 种身份（文件所有者，所属群组，其他用户），并分别搭配 3 种权限（读 r，写 w，访问 x）。比如，我们可以通过 ls -l 命令查看当前目录中所有文件的详细信息，其中就包含对各文件的权限设置：
```bash
[root@localhost ~]# ls -l
total 36
drwxr-xr-x. 2 root root 4096 Apr 15 16:33 Desktop
drwxr-xr-x. 2 root root 4096 Apr 15 16:33 Documents
...
-rwxr-xr-x. 2 root root 4096 Apr 15 16:33 post-install
...
```

以上输出信息中，“rwxr-xr-x”就指明了不同用户访问文件的权限，即文件所有者拥有对文件的读、写、访问权限（rwx），文件所属群组拥有对文件的读、访问权限（r-x），其他用户拥有对文件的读、访问权限（r-x）。
权限前的字符，表示文件的具体类型，比如 d 表示目录，- 表示普通文件，l 表示连接文件，b 表示设备文件，等等。

但在实际应用中，以上这 3 种身份根本不够用，给大家举个例子。
![](http://c.biancheng.net/uploads/allimg/181011/2-1Q011153333H0.jpg)
ACL控制权限
图 1 ACL 访问控制权限

图 1 的根目录中有一个 /project 目录，这是班级的项目目录。班级中的每个学员都可以访问和修改这个目录，老师需要拥有对该目录的最高权限，其他班级的学员当然不能访问这个目录。

需要怎么规划这个目录的权限呢？应该这样，老师使用 root 用户，作为这个目录的属主，权限为 rwx；班级所有的学员都加入 tgroup 组，使 tgroup 组作为 /project 目录的属组，权限是 rwx；其他人的权限设定为 0（也就是 ---）。这样一来，访问此目录的权限就符合我们的要求了。  

有一天，班里来了一位试听的学员 st，她必须能够访问 /project 目录，所以必须对这个目录拥有 r 和 x 权限；但是她又没有学习过以前的课程，所以不能赋予她 w 权限，怕她改错了目录中的内容，所以学员 st 的权限就是 r-x。可是如何分配她的身份呢？变为属主？当然不行，要不 root 该放哪里？加入 tgroup 组？也不行，因为 tgroup 组的权限是 rwx，而我们要求学员 st 的权限是 r-x。如果把其他人的权限改为 r-x 呢？这样一来，其他班级的所有学员都可以访问 /project 目录了。  

显然，普通权限的三种身份不够用了，无法实现对某个单独的用户设定访问权限，这种情况下，就需要使用 ACL 访问控制权限。  

ACL，是 Access Control List（访问控制列表）的缩写，在 Linux 系统中， ACL 可实现对单一用户设定访问文件的权限。也可以这么说，设定文件的访问权限，除了用传统方式（3 种身份搭配 3 种权限），还可以使用 ACL 进行设定。拿本例中的 st 学员来说，既然赋予它传统的 3 种身份，无法解决问题，就可以考虑使用 ACL 权限控制的方式，直接对 st 用户设定访问文件的 r-x 权限。
开启 ACL 权限
CentOS 6.x 系统中，ACL 权限默认处于开启状态，无需手工开启。但如果你的操作系统不是 CentOS 6.x，可以通过如下方式查看ACL权限是否开启：  
```bash
[root@localhost ~]# mount
/dev/sda1 on /boot type ext4 (rw)
/dev/sda3 on I type ext4 (rw)
…省略部分输出…
#使用mount命令可以看到系统中已经挂载的分区，但是并没有看到ACL权限的设置
[root@localhost ~]# dumpe2fs -h /dev/sda3
#dumpe2fs是查询指定分区文件系统详细信息的命令
…省略部分输出…
Default mount options: user_xattr acl
…省略部分输出…
``
其中，dumpe2fs 命令的 -h 选项表示仅显示超级块中的信息，而不显示磁盘块组的详细信息；

使用 mount 命令可以查看到系统中已经挂载的分区，而使用 dumpe2fs 命令可以查看到这个分区文件系统的详细信息。大家可以看到，我们的 ACL 权限是 /dev/sda3 分区的默认挂载选项，所以不需要手工挂载。
如果 Linux 系统如果没有默认挂载，可以执行如下命令实现手动挂载：  
```bash
[root@localhost ~]# mount -o remount,acl /  
#重新挂载根分区，并加入ACL权限  
```
使用 mount 命令重新挂载，并加入 ACL 权限。但使用此命令只是临时生效，要想永久生效，需要修改 /etc/fstab 文件，修改方法如下：
```bash
[root@localhost ~]#vi /etc/fstab
UUID=c2ca6f57-b15c-43ea-bca0-f239083d8bd2 /ext4 defaults,acl 1 1
#加入ACL权限
[root@localhost ~]# mount -o remount /
#重新挂载文件系统或重启系统，使修改生效
```
在你需要开启 ACL 权限的分区行上（也就是说 ACL 权限针对的是分区），手工在 defaults 后面加入 "，acl" 即可永久在此分区中开启 ACL 权限。

## 8.  Linux ACL权限设置（setfacl和getfacl）

通过上一节的学习，我们知道了什么是 ACL 权限，也了解了如何配置 Linux 系统使其开启 ACL 权限，本节来学习 ACL 设定文件访问权限的具体方法。

设定 ACl 权限，常用命令有 2 个，分别是 setfacl 和 getfacl 命令，前者用于给指定文件或目录设定 ACL 权限，后者用于查看是否配置成功。

getfacl 命令用于查看文件或目录当前设定的 ACL 权限信息。该命令的基本格式为：
```bash
[root@localhost ~]# getfacl 文件名
```
getfacl 命令的使用非常简单，且常和 setfacl 命令一起搭配使用。

setfacl 命令可直接设定用户或群组对指定文件的访问权限。此命令的基本格式为：
```bash
[root@localhost ~]# setfacl 选项 文件名
```
表 1 罗列出了该命令可以使用的所用选项及功能。

表 1 setfacl 命令选项及用法

|选项	|功能|  
|-----|-----|  
|-m 参数	|设定 ACL 权限。如果是给予用户 ACL 权限，参数则使用 "u:用户名:权限" 的格式，例如 setfacl -m u:st:rx /project 表示设定 st 用户对 project 目录具有 rx 权限；如果是给予组 ACL 权限，参数则使用 "g:组名:权限" 格式，例如 setfacl -m g:tgroup:rx /project 表示设定群组 tgroup 对 project 目录具有 rx 权限。|  
|-x 参数	|删除指定用户（参数使用 u:用户名）或群组（参数使用 g:群组名）的 ACL 权限，例如 setfacl -x u:st /project 表示删除 st 用户对 project 目录的 ACL 权限。|  
|-b	|删除所有的 ACL 权限，例如 setfacl -b /project 表示删除有关 project 目录的所有 ACL 权限。|  
|-d	|设定默认 ACL 权限，命令格式为 "setfacl -m d:u:用户名:权限 文件名"（如果是群组，则使用 d:g:群组名:权限），只对目录生效，指目录中新建立的文件拥有此默认权限，例如 setfacl -m d:u:st:rx /project 表示 st 用户对 project 目录中新建立的文件拥有 rx 权限。|  
|-R	|递归设定 ACL 权限，指设定的 ACL 权限会对目录下的所有子文件生效，命令格式为 "setfacl -m u:用户名:权限 -R 文件名"（群组使用 g:群组名:权限），例如 setfacl -m u:st:rx -R /project 表示 st 用户对已存在于 project 目录中的子文件和子目录拥有 rx 权限。|  
|-k	|删除默认 ACL 权限。|   

#### setfacl -m：给用户或群组添加 ACL 权限
回归上一节案例，解决方案如下：
* 老师使用 root 用户，并作为 /project 的所有者，对 project 目录拥有 rwx 权限；
* 新建 tgroup 群组，并作为 project 目录的所属组，包含本班所有的班级学员（假定只有 zhangsan 和 lisi），拥有对 project 的 rwx 权限；
* 将其他用户访问 project 目录的权限设定为 0（也就是 ---）。
* 对于试听学员 st 来说，我们对其设定 ACL 权限，令该用户对 project 拥有 rx 权限。

具体的设置命令如下：
```bash
[root@localhost ~]# useradd zhangsan
[root@localhost ~]# useradd lisi
[root@localhost ~]# useradd st
[root@localhost ~]# groupadd tgroup <-- 添加需要试验的用户和用户组，省略设定密码的过程
[root@localhost ~]# mkdir /project <-- 建立需要分配权限的目录
[root@localhost ~]# chown root:tgroup /project <-- 改变/project目录的所有者和所属组
[root@localhost ~]# chmod 770 /project  <-- 指定/project目录的权限
[root@localhost ~]# ll -d /project
drwxrwx---. 2 root tgroup 4096 Apr 16 12:55 /project
#这时st学员来试听了，如何给她分配权限
[root@localhost ~]# setfacl -m u:st:rx /project
#给用户st赋予r-x权限，使用"u:用户名：权限" 格式
[root@localhost /]# cd /
[root@localhost /]# ll -d /project
drwxrwx---+ 2 root tgroup 4096 Apr 16 12:55 /project
#如果查询时会发现，在权限位后面多了一个"+"，表示此目录拥有ACL权限
[root@localhost /]# getfacl project
#查看/prpject目录的ACL权限
#file:project <--文件名
#owner:root <--文件的所有者
#group:tgroup <--文件的所属组
user::rwx <--用户名栏是空的，说明是所有者的权限
user:st:r-x <--用户st的权限
group::rwx <--组名栏是空的，说明是所属组的权限
mask::rwx <--mask权限
other::--- <--其他人的权限
```
可以看到，通过设定 ACL 权限，我们可以单独给 st 用户分配 r-x 权限，而无需给 st 用户设定任何身份。

同样的道理，也可以给用户组设定 ACL 权限，例如：
```bash
[root@localhost /]# groupadd tgroup2
#添加新群组
[root@localhost /]# setfacl -m g:tgroup2:rwx project
#为组tgroup2纷配ACL权限
[root@localhost /]# ll -d project
drwxrwx---+ 2 root tgroup 4096 1月19 04:21 project
#属组并没有更改
[root@localhost /]# getfacl project
#file: project
#owner: root
#group: tgroup
user::rwx
user:st:r-x
group::rwx
group:tgroup2:rwx <-用户组tgroup2拥有了rwx权限
mask::rwx
other::---
```
#### setfacl -d：设定默认 ACL 权限
既然已经对 project 目录设定了 ACL 权限，那么，如果在这个目录中新建一些子文件和子目录，这些文件是否会继承父目录的 ACL 权限呢？执行以下命令进行验证：
```bash
[root@localhost /]# cd project
[root@localhost project]# touch abc
[root@localhost project]# mkdir d1
#在/project目录中新建了abc文件和d1目录
[root@localhost project]#ll
总用量4
-rw-r--r-- 1 root root 01月19 05:20 abc
drwxr-xr-x 2 root root 4096 1月19 05:20 d1
```
可以看到，这两个新建立的文件权限位后面并没有 "+"，表示它们没有继承 ACL 权限。这说明，后建立的子文件或子目录，并不会继承父目录的 ACL 权限。

当然，我们可以手工给这两个文件分配 ACL 权限，但是如果在目录中再新建文件，都要手工指定，则显得过于麻烦。这时就需要用到默认 ACL 权限。

默认 ACL 权限的作用是，如果给父目录设定了默认 ACL 权限，那么父目录中所有新建的子文件都会继承父目录的 ACL 权限。需要注意的是，默认 ACL 权限只对目录生效。

例如，给 project 文件设定 st 用户访问 rx 的默认 ACL 权限，可执行如下指令：
```bash
[root@localhost /]# setfacl -m d:u:st:rx project
[root@localhost project]# getfacl project
# file: project
# owner: root
# group: tgroup
user:: rwx
user:st:r-x
group::rwx
group:tgroup2:rwx
mask::rwx
other::---
default:user::rwx <--多出了default字段
default:user:st:r-x
default:group::rwx
default:mask::rwx
default:other::---
[root@localhost /]# cd project
[root@localhost project]# touch bcd
[root@localhost project]# mkdir d2
#新建子文件和子目录
[root@localhost project]# ll 总用量8
-rw-r--r-- 1 root root 01月19 05:20 abc
-rw-rw----+ 1 root root 01月19 05:33 bcd
drwxr-xr-x 2 root root 4096 1月19 05:20 d1
drwxrwx---+ 2 root root 4096 1月19 05:33 d2
#新建的bcd和d2已经继承了父目录的ACL权限
```
大家发现了吗？原先的 abc 和 d1 还是没有 ACL 权限，因为默认 ACL 权限是针对新建立的文件生效的。

对目录设定的默认 ACL 权限，可直接使用 setfacl -k 命令删除。例如：
```bash
[root@localhost /]# setfacl -k project
```
通过此命令，即可删除 project 目录的默认 ACL 权限，读者可自行通过 getfacl 命令查看。
setfacl -R：设定递归 ACL 权限
递归 ACL 权限指的是父目录在设定 ACL 权限时，所有的子文件和子目录也会拥有相同的 ACL 权限。

例如，给 project 目录设定 st 用户访问权限为 rx 的递归 ACL 权限，执行命令如下：
```
[root@localhost project]# setfacl -m u:st:rx -R project
[root@localhost project]# ll
总用量 8
-rw-r-xr--+ 1 root root 01月19 05:20 abc
-rw-rwx--+ 1 root root 01月19 05:33 bcd
drwxr-xr-x+ 2 root root 4096 1月19 05:20 d1
drwxrwx---+ 2 root root 4096 1月19 05:33 d2
#abc和d1也拥有了ACL权限
```
注意，默认 ACL 权限指的是针对父目录中后续建立的文件和目录会继承父目录的 ACL 权限；递归 ACL 权限指的是针对父目录中已经存在的所有子文件和子目录会继承父目录的 ACL 权限。
setfacl -x：删除指定的 ACL 权限
使用 setfacl -x 命令，可以删除指定的 ACL 权限，例如，删除前面建立的 st 用户对 project 目录的 ACL 权限，执行命令如下：
```bash
[root@localhost /]# setfacl -x u:st project
#删除指定用户和用户组的ACL权限
[root@localhost /]# getfacl project
# file:project
# owner: root
# group: tgroup
user::rwx
group::rwx
group:tgroup2:rwx
mask::rwx
other::---
#st用户的权限已被删除
```
#### setfacl -b：删除指定文件的所有 ACL 权限
此命令可删除所有与指定文件或目录相关的 ACL 权限。例如，现在我们删除一切与 project 目录相关的 ACL 权限，执行命令如下：
```bash
[root@localhost /]# setfacl -b project
#会删除文件的所有ACL权限
[root@localhost /]# getfacl project
#file: project
#owner: root
# group: tgroup
user::rwx
group::rwx
other::---
#所有ACL权限已被删除
```

## 9. Linux mask有效权限详解

前面，我们已经学习如何使用 setfacl 和 getfacl 为用户或群组添加针对某目录或文件的 ACL 权限。例如：
```bash
[root@localhost /]# getfacl project
#file: project <-文件名
#owner: root <-文件的属主
#group: tgroup <-文件的属组
user::rwx <-用户名栏是空的，说明是所有者的权限
group::rwx <-组名栏是空的，说明是所属组的权限
other::--- <-其他人的权限
[root@localhost ~]# setfacl -m u:st:rx /project
#给用户st设定针对project目录的rx权限
[root@localhost /]# getfacl project
#file: project 
#owner: root
#group: tgroup 
user::rwx 
user:st:r-x <-用户 st 的权限
group::rwx
mask::rwx <-mask 权限
other::---
```

对比添加 ACL 权限前后 getfacl 命令的输出信息，后者多了 2 行信息，一行是我们对 st 用户设定的 r-x 权限，另一行就是 mask 权限。

mask 权限，指的是用户或群组能拥有的最大 ACL 权限，也就是说，给用户或群组设定的 ACL 权限不能超过 mask 规定的权限范围，超出部分做无效处理。

举个例子，如果像上面命令那样，给 st 用户赋予访问 project 目录的 r-x 权限，此时并不能说明 st 用户就拥有了对该目录的读和访问权限，还需要和 mask 权限对比，r-x 确实是在 rwx 范围内，这时才能说 st 用户拥有 r-x 权限。
需要注意的是，这里将权限进行对比的过程，实则是将两权限做“按位相与”运算，最终得出的值，即为 st 用户有效的 ACL 权限。这里以读（r）权限为例，做相与操作的结果如表 1 所示：


表 1 读权限做相与操作  

|A	|B	|and|  
|-----|-----|-----|  
|r	|r	|r|  
|r	|-	|-|  
|-	|r	|-|  
|-	|-	|-|  

但是，如果把 mask 权限改为 r--，再和 st 用户的权限 r-x 比对（r-- 和 r-w 做与运算），由于 r-w 超出 r-- 的权限范围，因此 st 用户最终只有 r 权限，手动赋予的 w 权限无效。这就是在设定 ACL 权限时 mask 权限的作用。

大家可以这样理解 mask 权限的功能，它将用户或群组所设定的 ACL 权限限制在 mask 规定的范围内，超出部分直接失效。

mask 权限可以使用 setfacl 命令手动更改，比如，更改 project 目录 mask 权限值为 r-x，可执行如下命令：
```bash
[root@localhost ~]# setfacl -m m:rx /project
#设定mask权限为r-x，使用"m:权限"格式
[root@localhost ~]# getfacl /project
#file：project
#owner：root
#group：tgroup
user::rwx
group::rwx
mask::r-x  <--mask权限变为r-x
other::---
```
不过，我们一般不更改 mask 权限，只要赋予 mask 最大权限（也就是 rwx），则给用户或群组设定的 ACL 权限本身就是有效的。

## 10.  Linux SetUID（SUID）文件特殊权限用法详解

在讲解《权限位》一节时提到过，其实除了 rwx 权限，还会用到 s 权限，例如：
```bash
[root@localhost ~]# ls -l /usr/bin/passwd
-rwsr-xr-x. 1 root root 22984 Jan  7  2007 /usr/bin/passwd
```
可以看到，原本表示文件所有者权限中的 x 权限位，却出现了 s 权限，此种权限通常称为 SetUID，简称 SUID 特殊权限。

SUID 特殊权限仅适用于可执行文件，所具有的功能是，只要用户对设有 SUID 的文件有执行权限，那么当用户执行此文件时，会以文件所有者的身份去执行此文件，一旦文件执行结束，身份的切换也随之消失。

举一个例子，我们都知道，Linux 系统中所有用户的密码数据都记录在 /etc/shadow 这个文件中，通过 ll /etc/shadow 命令可以看到，此文件的权限是 0（---------），也就是说，普通用户对此文件没有任何操作权限。

这就会产生一个问题，为什么普通用户可以使用 passwd 命令修改自己的密码呢？

本节开头已经显示了 passwd 命令的权限配置，可以看到，此命令拥有 SUID 特殊权限，而且其他人对此文件也有执行权限，这就意味着，任何一个用户都可以用文件所有者，也就是 root 的身份去执行 passwd 命令。
Linux 系统中，绝对多数命令的文件所有者默认都是 root。

换句话说，当普通用户使用 passwd 命令尝试更改自己的密码时，实际上是在以 root 的身份执行passwd命令，正因为 root 可以将密码写入 /etc/shadow 文件，所以普通用户也能做到。只不过，一旦命令执行完成，普通用户所具有的 root身份也随之消失。

如果我们手动将 /usr/bin/passwd 文件的 SUID 权限取消，会发生什么呢？观察如下命令的执行过程：
```bash
[root@localhost ~]# chmod u-s /usr/bin/passwd
#属主取消SetUID权限
[root@localhost ~]# ll /usr/bin/passwd
-rwxr-xr-x. 1 root root 30768 Feb 22 2012 /usr/bin/passwd
[root@localhost ~]# su - lamp
[lamp@localhost ~]$ passwd
Changing password for user lamp.
Changing password for user.
(current) UNIX password:
#看起来没有什么问题
New passwor:
Retype new password:
password:Authentication token manipulation error  <--鉴定令牌操作错误
#最后密码没有生效
```
显然，虽然用户有执行 passwd 命令的权限，但无修改 /etc/shadow 文件的权限，因此最终密码修改失败。
注意，实验完成后，一定要再把 /usr/bin/passwd 文件的 SetUID 权限加上。

那么，普通用户可以使用 cat 命令查看 /etc/shadow 文件吗？答案的否定的，因为 cat 不具有 SUID 权限，因此普通用户在执行 cat /etc/shadow 命令时，无法以  root 的身份，只能以普通用户的身份，因此无法成功读取。

我们可以使用下面这张图来描述上述过程：
![](http://c.biancheng.net/uploads/allimg/181011/2-1Q0111H312U8.jpg)
SUID示意图
图 1 SUID示意图

由此，我们可以总结出，SUID 特殊权限具有如下特点：
只有可执行文件才能设定 SetUID 权限，对目录设定 SUID，是无效的。
用户要对该文件拥有 x（执行）权限。
用户在执行该文件时，会以文件所有者的身份执行。
SetUID 权限只在文件执行过程中有效，一旦执行完毕，身份的切换也随之消失。

## 11. SetUID（SUID）千万不要胡乱使用！

SetUID权限设置不当，会给 Linux 系统造成重大安全隐患。

前面的例子中，我们试验了将 passwd 命令取消 SUID 权限，这会导致 passwd 命令的功能失效。那么，如果我们手动给默认无 SetUID 权限的系统命令赋予 SetUID 权限，会出现什么情况呢？

比如说，我们尝试给 Vim 赋予 SetUID 权限：
```bash
[root@localhost ~]# chmod u+s /usr/bin/vim
[root@localhost ~]# ll /usr/bin/vim
-rwsr-xr-x. 1 root root 1847752 Apr 5 2012 /usr/bin/vim
```
此时你会发现，即便是普通用户使用 vim 命令，都会暂时获得 root 的身份和权限，例如，很多原本普通用户不能查看和修改的文件，竟然可以查看了，以 /etc/passwd 和 /etc/shadow 文件为例，普通用户也可以将自己的 UID 手动修改为 0，这意味着，此用户升级成为了超级用户。除此之外，普通用户还可以修改例如 /etc/inittab 和 /etc/fstab 这样重要的系统文件，可以轻易地使系统瘫痪。

其实，任何只有管理员可以执行的命令，如果被赋予了 SetUID 权限，那么后果都是灾难性的。普通用户可以随时重启服务器、随时关闭看得不顺眼的服务、随时添加其他普通用户的服务器，可以想象是什么样子。所以，SetUID 权限不能随便设置。

有读者可能会问，如何防止他人（例如黑客）对 SetUID 权限的恶意篡改呢？这里，给大家提供以下几点建议：
关键目录要严格控制写权限，比如 "/"、"/usr" 等。
用户的密码设置要严格遵守密码规范。
对系统中默认应该有 SetUID 权限的文件制作一张列表，定时检査有没有列表之外的文件被设置了 SetUID 权限。

前面 2 点不再做过多解释，这里就最后一点，给大家提供一个脚本，仅供参考。

首先，在服务器第一次安装完成后，马上查找系统中所有拥有 SetUID 和 SetGID 权限的文件，把它们记录下来，作为扫描的参考模板。如果某次扫描的结果和本次保存下来的模板不一致，就说明有文件被修改了 SetUID 和 SetGID 权限。命令如下：
```bash
[root@localhost ~]# find / -perm -4000 -o -perm -2000 > /root/suid.list
#-perm安装权限査找。-4000对应的是SetUID权限，-2000对应的是SetGID权限
#-o是逻辑或"or"的意思。并把命令搜索的结果放在/root/suid.list文件中
```
接下来，只要定时扫描系统，然后和模板文件比对就可以了。脚本如下：
```bash
[root@localhost ~]#vi suidcheck.sh
#!/bin/bash
find / -perm -4000 -o -perm -2000 > /tmp/setuid.check
#搜索系统中所有拥有SetUID和SetGID权限的文件，并保存到临时目录中
for i in $(cat /tmp/setuid.check)
#循环，每次循环都取出临时文件中的文件名
do
    grep $i /root/suid.list > /dev/null
    #比对这个文件名是否在模板文件中
    if ["$?"!="o"]
    #检测测上一条命令的返回值，如果不为0，则证明上一条命令报错
    then
        echo "$i isn't in listfile! " >>/root/suid_log_$(date +%F)
        #如果文件名不在模板文件中，则输出错误信息，并把报错写入日志中
    fi
done
rm -rf/tmp/setuid.check
#删除临时文件
[root@localhost ~]# chmod u+s /bin/vi
#手工给vi加入SetUID权限
[root@localhost ~]# ./suidcheck.sh
#执行检测脚本
[root@localhost ~]# cat suid_log_2013-01-20
/bin/vi isn't in listfile!
#报错了，vi不在模板文件中。代表vi被修改了SetUID权限
```
这个脚本成功的关键在于模板文件是否正常。所以一定要安装完系统就马上建立模板文件，并保证模板文件的安全。

注意，除非特殊情况，否则不要手工修改 SetUID 和 SetGID 权限，这样做非常不安全。而且就算我们做实验修改了 SetUID 和 SetGID 权限，也要马上修改回来，以免造成安全隐患。
SetGID 特殊权限的相关内容，下节会做详细介绍。

## 12.  Linux SetGID（SGID）文件特殊权限用法详解

前面学习了 SetUID，那么，什么是 SetGID 呢？很简单，当 s 权限位于所属组的 x 权限位时，就被称为 SetGID，简称 SGID 特殊权限。例如：
```bash
[root@localhost ~]# ll /usr/bin/locate
-rwx--s--x. 1 root slocate 35612 8月24 2010 /usr/bin/locate
```
与 SUID 不同的是，SGID 既可以对文件进行配置，也可以对目录进行配置。
SetGID（SGID）对文件的作用
同 SUID 类似，对于文件来说，SGID 具有如下几个特点：
SGID 只针对可执行文件有效，换句话说，只有可执行文件才可以被赋予 SGID 权限，普通文件赋予 SGID 没有意义。
用户需要对此可执行文件有 x 权限；
用户在执行具有 SGID 权限的可执行文件时，用户的群组身份会变为文件所属群组；
SGID 权限赋予用户改变组身份的效果，只在可执行文件运行过程中有效；
其实，SGID 和 SUID 的不同之处就在于，SUID 赋予用户的是文件所有者的权限，而 SGID 赋予用户的是文件所属组的权限，就这么简单。

就以本节开头的 locate 命令为例，可以看到，/usr/bin/locate 文件被赋予了 SGID 的特殊权限，这就意味着，当普通用户使用 locate 命令时，该用户的所属组会直接变为 locate 命令的所属组，也就是 slocate。

我们知道，locate 命令是用于在系统中按照文件名查找符合条件的文件的，当执行搜索操作时，它会通过搜索 /var/lib/mlocate/mlocate.db 这个数据库中的数据找到答案，我们来看看此数据库的权限：
```bash
[root@localhost ~]# ll /var/lib/mlocate/mlocate.db
-rw-r-----. 1 root slocate 1838850 1月20 04:29 /var/lib/mlocate/mlocate.db
```
可以看到，mlocate.db 文件的所属组为 slocate，虽然对文件只拥有 r 权限，但对于普通用户执行 locate 命令来说，已经足够了。一方面，普通用户对 locate命令拥有执行权限，其次，locate 命令拥有 SGID 权限，这使得普通用户在执行 locate 命令时，所属组身份会变为 slocate，而 slocate 对 mlocate.db 数据库文件拥有 r 权限，所以即便是普通用户，也可以成功执行 locate 命令。
再次强调，无论是 SUID，还是 SGID，它们对用户身份的转换，只有在命令执行的过程中有效，一旦命令执行完毕，身份转换也随之失效。

SetGID（SGID）对目录的作用
事实上，SGID 也能作用于目录，且这种用法很常见。

当一个目录被赋予 SGID 权限后，进入此目录的普通用户，其有效群组会变为该目录的所属组，会就使得用户在创建文件（或目录）时，该文件（或目录）的所属组将不再是用户的所属组，而使用的是目录的所属组。

也就是说，只有当普通用户对具有 SGID 权限的目录有 rwx 权限时，SGID 的功能才能完全发挥。比如说，如果用户对该目录仅有 rx 权限，则用户进入此目录后，虽然其有效群组变为此目录的所属组，但由于没有 x 权限，用户无法在目录中创建文件或目录，SGID 权限也就无法发挥它的作用。

举个例子：
```bash
[root@localhost ~]# cd /tmp
#进入临时目录做此实验。因为只有临时目录才允许普通用户修改
[root@localhost tmp]# mkdir dtest
#建立测试目录
[root@localhost tmp]# chmod g+s dtest
#给测试目录赋予SetGID权限
[root@localhost tmp]# ll -d dtest
drwxr-sr-x 2 root root 4096 Apr 19 06:04 dtest
#SetGID权限已经生效
[root@localhost tmp]# chmod 777 dtest
#给测试目录赋予777权限，让普通用户可以写
[root@localhost tmp]# su - lamp
[lamp@localhost ~]# grep lamp /etc/passwd /etc/group
/etc/passwd:lamp:x:501:501::/home/lamp:/bin/bash
/etc/group:lamp:x:501:
#切换成普通用户lamp，此用户的所属群组为 lamp
[lamp@localhost ~]$ cd /tmp/dtest/
#普通用户进入测试目录
[lamp@localhost dtest]$ touch abc
[lamp@localhost dtest]$ mkdir zimulu
#在此目录中创建新的文件 abc 和子目录 zimulu
[lamp@localhost dtest]$ ll
total 0
-rw-rw-r--. 1 lamp root 0 Apr 19 06:07 abc
drwxrwsr-x. 2 lamp root 40 Apr 19 06:07 zimulu
```
可以看到，虽然是 lamp 用户创建的 abc 文件和 zimulu 目录，但它们的所属组都不是 lamp（lamp 用户的所属组），而是 root（dtest 目录的所属组）。

## 13. Linux Stick BIT（SBIT）文件特殊权限用法详解

Sticky BIT，简称 SBIT 特殊权限，可意为粘着位、粘滞位、防删除位等。

SBIT 权限仅对目录有效，一旦目录设定了 SBIT 权限，则用户在此目录下创建的文件或目录，就只有自己和 root 才有权利修改或删除该文件。

也就是说，当甲用户以目录所属组或其他人的身份进入 A 目录时，如果甲对该目录有 w 权限，则表示对于 A 目录中任何用户创建的文件或子目录，甲都可以进行修改甚至删除等操作。但是，如果 A 目录设定有 SBIT 权限，那就大不一样啦，甲用户只能操作自己创建的文件或目录，而无法修改甚至删除其他用户创建的文件或目录。

举个例子，Linux 系统中，存储临时文件的 /tmp 目录就设定有 SBIT 权限：
```bash
[root@localhost ~]# ll -d /tmp
drwxrwxrwt. 4 root root 4096 Apr 19 06:17 /tmp

可以看到，在其他人身份的权限设定中，原来的 x 权限位被 t 权限占用了，这就表示此目录拥有 SBIT 权限。通过下面一系列的命令操作，我们来具体看看 SBIT 权限对 /tmp 目录的作用。
[root@localhost ~]# useradd lamp
[root@localhost ~]# useradd lamp1
#建立测试用户lamp和lamp1，省略设置密码过程
[root@localhost ~]# su -lamp
#切换为lamp用户
[lamp@localhost ~]$ cd /tmp
[lamp@localhost tmp]$ touch ftest
#建立测试文件
[lamp@localhost tmp]$ll ftest
-rw-rw-r-- 1 lamp lamp Apr 19 06:36 ftest
[lamp@localhost tmp]$ su - lamp1
#省略输入lamp1用户密码的过程，切换成lamp1用户
[lamp1 @localhost ~]$ cd /tmp/
[lamp1 @localhost tmp]$ rm -rf ftest
rm:cannot remove 'ftest':Operation not permitted
```
可以看到，虽然 /tmp 目录的权限设定是 777，但由于其具有 SBIT 权限，因此 lamp 用户在此目录创建的文件 ftest，lamp1 用户删除失败。

## 14.  Linux文件特殊权限（SUID、SGID和SBIT）设置详解

前面已经学习 SUID、SGID、SBIT 特殊权限，以及各自的含义和功能，那么，如何给文件或目录手动设定这些特殊权限呢？

还是要依赖 chmod 命令。我们知道，使用 chmod 命令给文件或目录设定权限，有 2 种方式，分别是使用数字形式和字母形式。例如：
```bash
#数字形式
[root@localhost ~]# chmod 755 ftest
#字母形式
[root@localhost ~]# chmod u=rwx,go=rx ftest
```
给文件或目录设定 SUID、SGID 和 SBIT 特殊权限，也可以使用这 2 种形式。


我们知道，给 chmod 命令传递 3 个数字，即可实现给文件或目录设定普通权限。比如说，"755" 表示所有者拥有 rwx 权限，所属组拥有 rx 权限，其他人拥有 tx 权限。

给文件或目录设定特殊权限，只需在这 3 个数字之前增加一个数字位，用来放置给文件或目录设定的特殊权限，就这么简单。

因此，我们有必要知道 SUID、SGID、SBIT 分别对应的数字，如下所示：
* 4 --> SUID
* 2 --> SGID
* 1 --> SBIT

举个例子，如果要将一个文件权限设置为 -rwsr-xr-x，怎么办呢？此文件的普通权限为 755，另外，此文件还有 SUID 权限，因此只需在 755 的前面，加上 SUID 对应的数字 4 即可。也就是说，只需执行chmod 4755 文件名命令，就完成了-rwsr-xr-x 权限的设定。
关于 -rwsr-xr-x 的普通权限是 755，你可以这样理解，标记有 s 和 t 的权限位，隐藏有 x 权限，对此，本节后续会给出更详细的解释。

同样的道理，如果某文件拥有 SUID 和 SGID 权限，则只需要给 chmod 命令传递 6---（- 表示数字）即可；如果某目录拥有 SGID 和 SBIT，只需要给 chmod 命令传递 3--- 即可。

注意，不同的特殊权限，作用的对象是不同的，SUID 只对可执行文件有效；SGID 对可执行文件和目录都有效；SBIT 只对目录有效。当然，你也可以给文件设置 7---，也就是将 SUID、SGID、SBIT赋予一个文件或目录，例如：
```bash
[root@localhost ~]# chmod 7777 ftest
#一次赋予SetUID、SetGID和SBIT权限
[root@localhost ~]# ll ftest
-rwsrwsrwt. 1 root root Apr 19 23:54 ftest
```
执行过程虽然没有报错，但这样做，没有任何实际意义。

除了赋予 chmod 命令 4 个数字设定特殊权限，还可以使用字母的形式。例如，可以通过 "u+s" 给文件赋予 SUID 权限；通过 "g+s" 给文件或目录赋予 SGID 权限；通过 "o+t" 给目录赋予 SBIT 权限。

举一个例子：
```bash
[root@localhost ~]#chmod u+s, g+s, o+t ftest
#设置特殊权限
[root@localhost ~]# ll ftest
-rwsr-sr-t. 1 root root Apr 19 23:54 ftest
[root@localhost ~]# chmod u-s, g-s, o-t ftest
#取消特殊权限
[root@localhost ~]# ll ftest
-rwxr-xr-x. 1 root root Apr 19 23:54 ftest
```
例子中，通过字母的形式成功给 ftest 文件赋予了 3 种特殊权限，此做法仅为验证字母形式的可行性，对 ftest 文件来说，并无实际意义。

细心的读者可能发现这样一个问题，使用 chmod 命令给文件或目录赋予特殊权限时，原文件或目录中存在的 x 权限会被替换成 s 或 t，而当我们使用 chmod 命令消除文件或目录的特殊权限时，原本消失的 x 权限又会显现出来。

这是因为，无论是 SUID、SGID 还是 SBIT，它们只针对具有 x 权限的文件或目录有效。没有 x 权限的文件或目录，即便赋予特殊权限，也无法发挥它们的功能，没有任何意义。

例如，我们就是要给不具有 x 权限的文件或目录赋予特殊权限，看看有什么效果：
```bash
[root@localhost ~]# chmod 7666 ftest
[root@localhost ~]# ll ftest
-rwSrwSrwT. 1 root root Apr 23:54 ftest
```
可以看到，相应的权限位会被标记为 S（大写）和 T（大写），指的就是设置的 SUID、SGID 和 SBIT 权限没有意义。

## 15.  Linux chattr命令详解：修改文件系统的权限属性

管理 Linux 系统中的文件和目录，除了可以设定普通权限和特殊权限外，还可以利用文件和目录具有的一些隐藏属性。

chattr 命令，专门用来修改文件或目录的隐藏属性，只有 root 用户可以使用。该命令的基本格式为：
```bash
[root@localhost ~]# chattr [+-=] [属性] 文件或目录名
```
+ 表示给文件或目录添加属性，- 表示移除文件或目录拥有的某些属性，= 表示给文件或目录设定一些属性。

表 1 列出了常用的一些属性及功能。
表 1 chattr 命令常用的属性选项及功能   

|属性选项	|功能|  
|-----|-----|  
|i|	如果对文件设置 i 属性，那么不允许对文件进行删除、改名，也不能添加和修改数据；如果对目录设置 i 属性，那么只能修改目录下文件中的数据，但不允许建立和删除文件；|
|a|	如果对文件设置 a 属性，那么只能在文件中増加数据，但是不能删除和修改数据； 如果对目录设置 a 属性，那么只允许在目录中建立和修改文件，但是不允许删除文件；|  
|u	|设置此属性的文件或目录，在删除时，其内容会被保存，以保证后期能够恢复，常用来防止意外删除文件或目录。|  
|s	|和 u 相反，删除文件或目录时，会被彻底删除（直接从硬盘上删除，然后用 0 填充所占用的区域），不可恢复。|  

【例 1】 给文件赋予 i 属性。
```bash
[root@localhost ~]# touch ftest
#建立测试文件
[root@localhost ~]# chattr +i ftest
[root@localhost ~]# rm -rf ftest
rm:cannot remove 'ftest':Operation not permitted
#无法删除"ftesr"，操作不允许
#被赋予i属性后，root不能删除
[root@localhost ~]# echo 111>>ftest
bash:ftest:Permission denied
#权限不够，不能修改文件中的数据
```
可以看到，设置有 i 属性的文件，即便是 root 用户，也无法删除和修改数据。

【例 2】为目录赋予 i 属性。
```bash
[root@localhost ~]# mkdir dtest
#建立测试目录
[root@localhost dtest]# touch dtest/abc
#再建立一个测试文件abc
[root@localhost ~]# chattr +i dtest
#给目录赋予i属性
[root@localhost ~]# cd dtest
[root@localhost dtest]# touch bed
touch: cannot touch 'bed':Permission denied
#无法创建"bcd"，权限不够，dtest目录不能新建文件
[root@localhost dtest]# echo 11>>abc
[root@localhost dtest]# cat abc
11
#可以修改文件内容
[root@localhost dtest]# rm -rf abc
rm: cannot remove 'abc': Permission denied
#无法删除"abc"，权限不够
```
一旦给目录设置 i 属性，即使是 root 用户，也无法在目录内部新建或删除文件，但可以修改文件内容。
给设置有 i 属性的文件删除此属性也很简单，只需将 chattr 命令中 + 改为 - 即可。


【例 3】演示 a 属性的作用。
假设有这样一种应用，我们每天自动实现把服务器的日志备份到指定目录，备份目录可设置 a 属性，变为只可创建文件而不可删除。命令如下：
```bash
[root@localhost ~]# mkdir -p /back/log
#建立备份目录
[root@localhost ~]# chattr +a /back/log
#赋予a属性
[root@localhost ~]# cp /var/log/messages /back/log
#可以复制文件和新建文件到指定目录中
[root@localhost ~]# rm -rf /back/log/messages
rm: cannot remove '/back/log/messages': Permission denied
#无法删除 /back/log/messages，操作不允许
```

注意，通常情况下，不要使用 chattr 命令修改 /、/dev/、/tmp/、/var/ 等目录的隐藏属性，很容易导致系统无法启动。另外，chatrr 命令常与 lsattr 命令合用，前者修改文件或目录的隐藏属性，后者用于查看是否修改成功。有关 lsattr 命令，放到下节讲解。

## 16.  Linux lsattr命令：查看文件系统属性

使用 chattr 命令配置文件或目录的隐藏属性后，可以使用 lsattr 命令查看。

lsattr 命令，用于显示文件或目录的隐藏属性，其基本格式如下：
```bash
[root@localhost ~]# lsattr [选项] 文件或目录名
```

常用选项有以下 3 种：
* -a：后面不带文件或目录名，表示显示所有文件和目录（包括隐藏文件和目录）
* -d：如果目标是目录，只会列出目录本身的隐藏属性，而不会列出所含文件或子目录的隐藏属性信息；
* -R：和 -d 恰好相反，作用于目录时，会连同子目录的隐藏信息数据也一并显示出来。

【例 1】
```bash
[root@localhost ~]# touch attrtest
-----------e- attrtest
[root@localhost ~]# chattr +aij attrtest
[root@localhost ~]# lsattr attrtest
----ia---j-e- attrtest
```
注意，不使用任何选项，仅用于显示文件的隐藏信息，不适用于目录。

【例 2】
```baSH
[root@localhost ~]#lsattr -a
-----------e- ./.
------------- ./..
-----------e- ./.gconfd
-----------e- ./.bashrc
```


【例 3】
```bash
[root@localhost ~]#lsattr -d /back/log
-----a------e- /back/log
#查看/back/log目录，其拥有a和e属性
```

## 17.  Linux sudo命令用法详解：系统权限管理

我们知道，使用 su 命令可以让普通用户切换到 root 身份去执行某些特权命令，但存在一些问题，比如说：
仅仅为了一个特权操作就直接赋予普通用户控制系统的完整权限；
当多人使用同一台主机时，如果大家都要使用 su 命令切换到 root 身份，那势必就需要 root 的密码，这就导致很多人都知道 root 的密码；

考虑到使用 su 命令可能对系统安装造成的隐患，最常见的解决方法是使用 sudo 命令，此命令也可以让你切换至其他用户的身份去执行命令。

相对于使用 su 命令还需要新切换用户的密码，sudo 命令的运行只需要知道自己的密码即可，甚至于，我们可以通过手动修改 sudo 的配置文件，使其无需任何密码即可运行。

sudo 命令默认只有 root 用户可以运行，该命令的基本格式为：
```bash
[root@localhost ~]# sudo [-b] [-u 新使用者账号] 要执行的命令
```
常用的选项与参数：
* -b  ：将后续的命令放到背景中让系统自行运行，不对当前的 shell 环境产生影响。
* -u  ：后面可以接欲切换的用户名，若无此项则代表切换身份为 root 。
* -l：此选项的用法为 sudo -l，用于显示当前用户可以用 sudo 执行那些命令。

【例 1】
```bash
[root@localhost ~]#  grep sshd /etc/passwd
sshd:x:74:74:privilege-separated SSH:/var/empty/sshd:/sbin.nologin
[root@localhost ~]#  sudo -u sshd touch /tmp/mysshd
[root@localhost ~]#  ll /tmp/mysshd
-rw-r--r-- 1 sshd sshd 0 Feb 28 17:42 /tmp/mysshd
```
本例中，无法使用 su - sshd 的方式成功切换到 sshd 账户中，因为此用户的默认 Shell 是 /sbin/nologin。这时就显现出 sudo 的优势，我们可以使用 sudo 以 sshd 的身份在 /tmp 目录下创建 mysshd 文件，可以看到，新创建的 mysshd 文件的所有者确实是 sshd。

【例 2】
```bash
[root@localhost ~]#  sudo -u vbird1 sh -c "mkdir ~vbird1/www; cd ~vbird1/www; \
>  echo 'This is index.html file' > index.html"
[root@localhost ~]#  ll -a ~vbird1/www
drwxr-xr-x 2 vbird1 vbird1 4096 Feb 28 17:51 .
drwx------ 5 vbird1 vbird1 4096 Feb 28 17:51 ..
-rw-r--r-- 1 vbird1 vbird1   24 Feb 28 17:51 index.html
```
这个例子中，使用 sudo 命令切换至 vbird1 身份，并运行 sh -c 的方式来运行一连串的命令。

前面说过，默认情况下 sudo 命令，只有 root 身份可以使用，那么，如何让普通用户也能使用它呢？

解决这个问题之前，先给大家分析一下 sudo 命令的执行过程。sudo命令的运行，需经历如下几步：
当用户运行 sudo 命令时，系统会先通过 /etc/sudoers 文件，验证该用户是否有运行 sudo 的权限；
确定用户具有使用 sudo 命令的权限后，还要让用户输入自己的密码进行确认。出于对系统安全性的考虑，如果用户在默认时间内（默认是 5 分钟）不使用 sudo 命令，此后使用时需要再次输入密码；
密码输入成功后，才会执行 sudo 命令后接的命令。

显然，能否使用 sudo 命令，取决于对 /etc/sudoers 文件的配置（默认情况下，此文件中只配置有 root 用户）。所以接下来，我们学习对 /etc/sudoers 文件进行合理的修改。
sudo命令的配置文件/etc/sudoers
修改 /etc/sudoers，不建议直接使用 vim，而是使用 visudo。因为修改 /etc/sudoers 文件需遵循一定的语法规则，使用 visudo 的好处就在于，当修改完毕 /etc/sudoers 文件，离开修改页面时，系统会自行检验 /etc/sudoers 文件的语法。

因此，修改 /etc/sudoers 文件的命令如下：
```bash
[root@localhost ~]# visudo
…省略部分输出…
root ALL=(ALL) ALL  <--大约 76 行的位置
# %wheel ALL=(ALL) ALL   <--大约84行的位置
#这两行是系统为我们提供的模板，我们参照它写自己的就可以了
…省略部分输出…
```
通过 visudo 命令，我们就打开了 /etc/sudoers 文件，可以看到如上显示的 2 行信息，这是系统给我们提供的 2 个模板，分别用于添加用户和群组，使其能够使用 sudo 命令。

这两行模板的含义分为是：
```bash
root ALL=(ALL) ALL
#用户名 被管理主机的地址=(可使用的身份) 授权命令(绝对路径)
#%wheel ALL=(ALL) ALL
#%组名 被管理主机的地址=(可使用的身份) 授权命令(绝对路径)
```
表 1 对以上 2 个模板的各部分进行详细的说明。
表 1 /etc/sudoers 用户和群组模板的含义  

|模块	|含义|  
|---|---|
|用户名或群组名	|表示系统中的那个用户或群组，可以使用 sudo 这个命令。|  
|被管理主机的地址	|用户可以管理指定 IP 地址的服务器。这里如果写 ALL，则代表用户可以管理任何主机；如果写固定 IP，则代表用户可以管理指定的服务器。如果我们在这里写本机的 IP 地址，不代表只允许本机的用户使用指定命令，而是代表指定的用户可以从任何 IP 地址来管理当前服务器。|  
|可使用的身份	|就是把来源用户切换成什么身份使用，（ALL）代表可以切换成任意身份。这个字段可以省略。|   
|授权命令	|表示 root 把什么命令命令授权给用户，换句话说，可以用切换的身份执行什么命令。需要注意的是，此命令必须使用绝对路径写。默认值是 ALL，表示可以执行任何命令。|  

【例 3】
授权用户 lamp 可以重启服务器，由 root 用户添加，可以在 /etc/sudoers 模板下添加如下语句：
```bash
[root@localhost ~]# visudo
lamp ALL=/sbin/shutdown -r now
```
注意，这里也可以写多个授权命令，之间用逗号分隔。用户 lamp 可以使用 sudo -l 查看授权的命令列表：
```bash
[root@localhost ~]# su - lamp
#切换成lamp用户
[lamp@localhost ~]$ sudo -l
[sudo] password for lamp:
#需要输入lamp用户的密码
User lamp may run the following commands on this host:
(root) /sbin/shutdown -r now
```
可以看到，lamp 用户拥有了 shutdown -r now 的权限。这时，lamp 用户就可以使用 sudo 执行如下命令重启服务器：
```bash
[lamp@localhost ~]$ sudo /sbin/shutdown -r now
```
再次强调，授权命令要使用绝对路径（或者把 /sbin 路径导入普通用户 PATH 路径中，不推荐使用此方式），否则无法执行。

【例 4】
假设现在有 pro1，pro2，pro3 这 3 个用户，还有一个 group 群组，我们可以通过在 /etc/sudoers 文件配置 wheel 群组信息，令这 3 个用户同时拥有管理系统的权限。

首先，向 /etc/sudoers 文件中添加群组配置信息：
```bash
[root@localhost ~]# visudo
....(前面省略)....
%group     ALL=(ALL)    ALL
#在 84 行#wheel这一行后面写入
```
此配置信息表示，group 这个群组中的所有用户都能够使用 sudo 切换任何身份，执行任何命令。接下来，我们使用 usermod 命令将 pro1 加入 group 群组，看看有什么效果：
```bash
[root@localhost ~]# usermod -a -G group pro1
[pro1@localhost ~]# sudo tail -n 1 /etc/shadow <==注意身份是 pro1
....(前面省略)....
Password:  <==输入 pro1 的口令喔！
pro3:$1$GfinyJgZ$9J8IdrBXXMwZIauANg7tW0:14302:0:99999:7:::
[pro2@localhost ~]# sudo tail -n 1 /etc/shadow <==注意身份是 pro2
Password:
pro2 is not in the sudoers file.  This incident will be reported.
#此错误信息表示 pro2 不在 /etc/sudoers 的配置中。
```
可以看到，由于 pro1 加入到了 group 群组，因此 pro1 就可以使用 sudo 命令，而 pro2 不行。同样的道理，如果我们想让 pro3 也可以使用 sudo 命令，不用再修改 /etc/sudoers 文件，只需要将 pro3 加入 group 群组即可。
