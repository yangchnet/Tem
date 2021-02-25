---
title: "PV操作"
date: 2021-01-22T17:27:21+08:00
draft: false
---
# PV操作

## 1.生产者消费者问题 

### 1.1. 基本生产者消费者问题 

**问题描述**   
假设“生产者”进程不断向共享缓冲区写人数据(即生产数据),而“消费者”进程不断从共享缓冲区读出数据(即消费数据);共享缓冲区共有n个;任何时刻只能有一个进程可对共享缓冲区进行操作。所有生产者和消费者之间要协调,以完成对共享缓冲区的操作。  


**分析**  
我们可把共享缓冲区中的n个缓冲块视为共享资源,生产者写人数据的缓冲块成为消费者可用资源,而消费者读出数据后的缓冲块成为生产者的可用资源。为此,可设置三个信号量: full、 empty和mutex。其中: full表示有数据的缓冲块数目,初值是0; empty表示空的缓冲块数初值是n; mutex用于访问缓冲区时的互斥,初值是1。实际上, full和empty间存在如下关系:full+ empty = N

```C
buffer: array [0..k-1]of integer;
in,out: 0..k-1;//in记录第一个空缓冲区,out记录第一个不空的缓冲区
empty,full,mutex: semaphore;
//empty控制缓冲区不满,full控制缓冲区不空,mutex保护临界区;
//初始化empty=k,full=0,mutex=1
cobegin
    procedure producer:
    while true then
        begin
        produce(&item);
        p(empty);
        p(mutex);
        buffer[in]:=item;
        in:=(in+1) mod k;
        v(mutex);
        v(full);
        end
        
     procedure consumer:
        while true then
        begin
        p(full);
        p(mutex);
        item:=buffer[out];
        out:=(out+1) mod k;
        v(mutex);
        v(empty);
        consume(&item);
        end
coend
```

### 1.2 生产者消费者问题的扩展

**问题描述**  
设P,Q,R共享一个缓冲区,P,Q构成一对生产者-消费者,R既为生产者又为消费者。使用P,V 实现其同步。  

```C
var mutex,full,empty: semaphore;
full:=1;
empty:=0;
mutex:=1;
cobegin
Procedure P
    begin
    while true
        p(empty);
        p(mutex);
        Product one;
        v(mutex);
        v(full);
        end
        
Procedure Q
    begin
    while true
        p(full);
        p(mutex);
        consume one
        v(mutex);
        v(empty);
        end
        
Procedure R
    begin
        if empty:=1 then
            begin
            p(empty);
            P(mutex);
            product;
            v(mutex);
            v(full);
            end
        if full:=1 then
            begin
            p(full);
            p(mutex);
            消费一个产品;
            v(mutex);
            v(empty);
            end
coend
```

## 2. 读者写者问题

### 2.1 基本读者写者问题

**问题描述**  
有一个许多进程共享的数据区,这个数据区可以是一个文件或者主存的一块空间;有一些只读取这个数据区的进程(Reader)和一些只往数据区写数据的进程(Writer),此外还需要满足以下条件:  
1. 任意多个读进程可以同时读这个文件;  
2. 一次只有一个写进程可以往文件中写;  
3. 如果一个写进程正在进行操作,禁止任何读进程度文件。  

**分两种情况实现该问题：**
* 读优先: 要求指一个读者试图进行读操作时,如果这时正有其他读者在进行操作,他可直接开始读操作,而不需要等待。
```C++
//读者优先算法
rwmutex   //用于写者与其他读者/写者互斥的访问共享数据
rmutex //用于读者互斥的访问
readcount //读者计数器
var rwmutex, rmutex : semaphore := 1, 1 ;
int readcount = 0;
    cobegin
        procedure reader_i
            begin    // i=1,2,....
            P(rmutex);
            Readcount + +;
            if (readcount = = 1) 
                P(rwmutex);
            V(rmutex);
            读数据;
            P(rmutex);
            Readcount - -;
            if (readcount = = 0) 
                V(rwmutex);
            V(rmutex);
            end
        procedure Writer_j
            begin    // j = 1,2,....
            P(rwmutex);
            写更新;
            V(rwmutex);
            end
    Coend
```   
---
* 一个读者试图进行读操作时,如果有其他写者在等待进行写操作或正在进行写操作,他要等待该写者完成写操作后才开始读操作  

```C
//写者优先算法
/*
（1）多个读者可以同时进行读
（2）写者必须互斥(只允许一个写者写,也不能读者写者同时进行)
（3）写者优先于读者(一旦有写者,则后续读者必须等待,唤醒时优先考虑写者)
    如果读者数是固定的,我们可采用下面的算法:
*/
rwmutex:用于写者与其他读者/写者互斥的访问共享数据
rmutex: 该信号量初始值设为10,表示最多允许10个读者进程同时进行读操作
var rwmutex, rmutex : semaphore := 1, 10 ;
cobegin
    procedure reader_i
        begin
        // i=1,2,....
        P(rwmutex); //读者、写者互斥
        P(rmutex);
        V(rwmutex); // 释放读写互斥信号量,允许其它读、写进程访问资源
        读数据;
        V(rmutex);
        end
    procedure Writer_j
        begin
        // j = 1,2,....
        P(rwmutex);
        for (i = 1;i <= 10;i + +) 
            P(rmutex);
        //禁止新读者,并等待已进入的读者退出写更新;
        for (i = 1;i <= 10;i + +) 
            V(rmutex);
        // 恢复允许rmutex 值为10
        V(rwmutex);
        end
Coend
```

## 3. 银行排队问题 

**问题描述**  
银行有n个柜员,每个顾客进入银行后先取一个号,并且等着叫号,当一个柜员空闲后,就叫下一个号.  
**问题分析**  
将顾客号码排成一个队列,顾客进入银行领取号码后,将号码由队尾插入;柜员空闲时,从队首取得顾客号码,并且为这个顾客服务,由于队列为若干进程共享, 所以需要互斥.柜员空闲时,若有顾客,就叫下一个顾客为之服务.因此,需要设置一个信号量来记录等待服务的顾客数.  

```C
begin
    var mutex=1,customer_count=0:semaphore;
        cobegin
            process customer
                begin
                repeat
                取号码;
                p(mutex);
                进入队列;
                v(mutex);
                v(customer_count);
                end
            process serversi(i=1,...,n)
                begin
                repeat
                p(customer_count);
                p(mutex);
                从队列中取下一个号码;
                v(mutex);
                为该号码持有者服务;
                end
        coend
end
```

## 4. 机房上机问题 

**问题描述**  
某高校计算机系开设网络课并安排上机实习,假设机房共有2m台机器,有2n名学生选课(m,n均大于等于1),规定:
* 每两个学生组成一组,各占一台及其协同完成上机实习;  
* 只有一组两个学生到齐,并且此时机房有空闲机器时,该组学生才能进入机房;  
* 上机实习由一名教师检查,检查完毕,一组学生同时离开机房  

**注意**  
本题目隐含一个进程(Guard )。  

```C 
var stu,computer,enter,finish,test:semaphore;
ste:=2N;
computer:=2M;
enter:=0;
finish:=0;
test:=0;

cobegin
    Procedure Student
        begin
        p(computer);
        p(stu);
        Start computer;
        v(finish);
        v(test);
        v(computer);
        end
    Procedure Teacher
        begin
        p(finish);
        Test the work;
        v(test);
        v(test);
        end
    Procedure Guard
        begin
        p(stu);
        p(stu);
        Enter;
        v(enter);
        v(enter);
        end
coend
```

## 5. 保管员问题 

**问题描述**  
有一材料保管员,他保管纸和笔若干。有A 、 B 两组学生, A 组学生每人都备有纸, B组学生每人都备有笔.任一学生只要能得到其他一种材料就可以写信。有一个可以放一张纸或一支笔的小盒,当小盒中无物品时,保管员就可任意放一张纸或一支笔供学生取用,每次允许一个学生从中取出自己所需的材料,当学生从盒中取走材料后允许保管员再存放一件材料,请用信号量与P 、v 操作。  

```scala
var
    s, Sa.Sb, mutexa, mutexb: semaphore;
    s: = mutexa :=mutexb: = 1;
    sa: = sb: = 0;
    box: (PaPer, Pen);
cobegin
    process 保管员
        begin
        repeat
        P(S);
        take a material intobox ;
        if (box)=Paper then V(Sa);
        else V(Sb);
        untile false ;
        end

    Process A组学生
        begin
        repeat
        P(Sa);
        P(mutexa);
        take the pen from box ;
        V(mutexa);
        V(S);
        write a letter;
        untile false ;
        end

    Process B组学生
        begin
        repeat
        P(Sb);
        P(mutexb);
        take the paper from box ;
        V(mutexb);
        V(S);
        wnte a letter ;
        untile false ;
        end
Coend.
```


## 6. 批处理系统问题 

**问题描述**  
设某个批处理系统中,有三个进程:卡片输入进程、作业调度进程、作业控制进程。他们之间的合作关系是:
* 只要卡片输入机上有作业信息输入,进程把作业逐个输入至输出井并为每个作业建立一个JCB块并把它插入至后备作业队列(JCB链表)中  
* 当内存中无作业运行时,作业调度进程从JCB中选一个作业,把该作业装入内存。  
* 作业控制进程负责处理已调入内存的作业  

(1)P,V写出输入和调度进程的同步。  
(2)用消息缓冲痛惜,写出调度进程与作业控制进程间的同步算法。

```C
procedure 输入:
    begin
    L1:如果有卡片 then L2
        等待卡片;
    L2:把作业输入至输出井并建立JCB块;
    p(s);
    把JCB插入链中;
    v(mutex);
    v(s);
    Goto L1;
    end
procedure 调度:
    begin
    M:P(s);
    p(mutex);
    查JCB;
    v(s);
    send();//向控制进程发信息
    receive();//接受信息
    Goto M;
    end
procedure 作业控制:
    begin
    N:receive();
    处理;
    send();//向调度发信息
    Goto N;
    end
```
