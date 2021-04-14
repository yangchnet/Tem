---
author: "李昌"
title: "Go中的锁"
date: "2021-04-14"
tags: ["golang", "lock", "锁", "同步"]
categories: ["Golang"]
ShowToc: true
TocOpen: true
---

## 1. sync.Mutex互斥锁
不同goroutine之间对公共资源进行访问需要使用互斥锁。例如在对银行账户的操作中，如果我们有两种操作，一个是查询余额，一个是存款。其操作如下：
```go
package bank 

// 存款余额
var balance int 

// 存款
func Deposit(amount int) { balance = balance + amount } 

// 查询
func Balance() int { return balance }
```

```go
// Alice: 
go func() { 
    bank.Deposit(200)  // A1
    fmt.Println("=", bank.Balance()) // A2
}()

// Bob: 
go bank.Deposit(100) // B
```
这其中，若把A1分为两个操作，A1r：把余额从内存中读出来；A2w：把修改后的余额写入内存。  
若执行顺序为A1r → B → A1w → A2， 正常情况下，Alice和Bob分别存入了$200，$100，因此最后的存款应该是300，但最后输出结果为200。因为A在计算时是按照A1r读出的数值进行计算，忽略了B的操作，A与B之间发生了数据竞争。
> **数据竞争**：无论任何时候，只要有两个goroutine并发访问同一变量，且至少其中的一个是写操作的时候就会发生数据竞争。

解决此问题的办法之一是使用互斥锁。
```go
import "sync"
var ( 
    mu  sync.Mutex // guards balance
    balance int
)

func Deposit(amount int) { 
    mu.Lock() 
    defer mu.Unlock()
    balance = balance + amount 
}

func Balance() int { 
    mu.Lock() 
    defer mu.Unlock() 
    return balance
}
```
每次一个goroutine访问bank变量时(这里只有balance余额变量)，它都会调用mutex的Lock方法来获取一个互斥锁。如果其它的goroutine已经获得了这个锁的话，这个操作会被阻塞直到其它goroutine调用了Unlock使该锁变回可用状态.

## 2. sync.RWMutex读写锁
由于Balance函数只需要读取变量的状态，所以我们同时让多个Balance调用并发运行事实上 是安全的，只要在运行的时候没有存款或者取款操作就行。在这种场景下我们需要一种特殊 类型的锁，其允许多个只读操作并行执行，但写操作会完全互斥。这种锁叫作“多读单写”锁 (multiple readers, single writer lock)，Go语言提供的这样的锁是sync.RWMutex：
```go
var mu sync.RWMutex 
var balance int 
func Balance() int { 
    mu.RLock() // readers lock 
    defer mu.RUnlock() 
    return balance
}
```

读写锁的规则是  

- 读锁不能阻塞读锁
- 读锁需要阻塞写锁，直到所有读锁都释放
- 写锁需要阻塞读锁，直到所有写锁都释放
- 写锁需要阻塞写锁
