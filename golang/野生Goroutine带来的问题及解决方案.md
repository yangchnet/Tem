---
author: "李昌"
title: "野生Goroutine带来的问题及对应解决方案"
date: "2021-05-19"
tags: ["golang", "goroutine"]
categories: ["Golang"]
ShowToc: true
TocOpen: true
---

## 1. goroutine中panic无法恢复
正常的函数中panic的recover
```go
import (
    "fmt"
)

func main(){
    defer func(){
        if err := recover(); err != nil{
            fmt.Println(err)
        }
    }()
    var bar = []int{1}
    fmt.Println(bar[1])
}
```

```
reflect: slice index out of range
```

goroutine中panic的恢复
```go
func main(){
	defer func(){
		if err := recover(); err != nil {  // 在这里使用recover(),不能捕获panic
			fmt.Println("catch you, bad guy")
		}
	}()

	go func(){
		fmt.Println("I'm in a goroutine")
		panic("come to catch me")
	}()
	time.Sleep(10 * time.Second) // 主进程做其他事
}
```

```
I'm in a goroutine
panic: come to catch me

goroutine 6 [running]:
main.foo.func2()
        /home/lc/Workspace/lab/main/main.go:22 +0x95
created by main.foo
        /home/lc/Workspace/lab/main/main.go:15 +0x5b

Process finished with the exit code 2
```
上面的程序在`goroutine`中触发了一个`panic`，现在假设`foo()`主进程是我们的服务，我们在服务的`main()`中使用`recover()`尝试捕获程序中发生的panic，但程序运行后发现，goroutine中触发的`panic`没有被捕获，程序直接退出，代表我们的服务挂掉了。显然在正式环境中，我们不可能容忍这样的事情发生，因此需要对`goroutine`中触发的panic进行捕获处理。

显然最简单的方法是在`goroutine`中添加recover(), 进行异常捕获，如下：
```go
func main(){
	go func(){
        defer func(){
            if err := recover(); err != nil {  // 在这里使用recover(),不能捕获panic
                fmt.Println("catch you, bad guy")
            }
	    }()
		fmt.Println("I'm in a goroutine")
		panic("come to catch me")
	}()
	time.Sleep(10 * time.Second) // 主进程做其他事
}
```

```
I'm in a goroutine
catch you, bad guy

Process finished with the exit code 0

```
这时程序正常退出，服务没有挂掉。但每次使用`goroutine`都要加`recover()`稍显麻烦。我们可以使用函数式编程的思想来对`goroutine`进行一下封装：
```go
func Go(f func()){
	go func(){
		defer func(){
			if err := recover(); err != nil{
				fmt.Println("catch you, bad guy")
			}
		}()
		f()
	}()
}

func main(){
	Go(func(){
		fmt.Printf("I'm in a goroutine")
		panic("come to catch me")
	})
	time.Sleep(10 * time.Second) // 主进程做其他事
}
```

```
I'm in a goroutine
catch you, bad guy

Process finished with the exit code 0
```
显然，这样封装过后我们捕获到了panic，但**实际上我们仍然是在`goroutine`内部捕获的panic**，**A goroutine cannot recover from a panic in another goroutine**.    

## 2. goroutine的泄露问题

### 2.1 什么是goroutine泄露
如果你启动了一个goroutine，但并没有符合预期的退出，而是直到程序结束，此goroutine才退出，这种情况就是goroutine泄露。当goroutine泄露发生时，该goroutine所占用的内存不能得到释放，可用内存持续减少，最终将导致系统崩溃。
```go
package main

import "fmt"

func main(){
	for{
		go sayHello()
	}
}

func sayHello(){
	for{
		fmt.Println("hello")
	}
}
```
以上代码是一个最简单的goroutine泄露场景，所有的goroutine都没有退出，而是一直在后台执行。在我的电脑上执行以上代码后，内存占用达到98%。

### 2.2 goroutine泄露发生的原因

#### 2.2.1 channel导致的泄露
**1. 只读不发**
```go
func leak() {
	ch := make(chan int)

	go func() {
		val := <-ch
		fmt.Println("We received a value:", val)
	}()
}
```
在这个函数中，leak创建了一个goroutine，其在从ch中读取一个值之前一直阻塞，但leak并没有往ch中发送值，因此goroutine将一直阻塞，在函数结束前不能得到释放。

> 解决方案 

记得发送
```go
func unleak() {
	ch := make(chan int)

	go func() {
		val := <-ch
		fmt.Println("We received a value:", val)
	}()

	ch <- 1
}
```

**2. 只发不读**
```go
func gen(nums ...int) <-chan int {
	out := make(chan int)
	go func() {
		for _, n := range nums {
			out <- n
		}
		close(out)
	}()
	return out
}

func main() {
	defer func() {
		time.Sleep(time.Second)
		fmt.Println("the number of goroutines: ", runtime.NumGoroutine())
	}()
	
	_ = gen(2, 3)

}
```
```
the number of goroutines:  2
```

在这段代码中，gen向channel中发送数据，但由于没有对channel做接收，因此这个channel将会被阻塞，从而goroutine也不能正常退出，发生了goroutine泄露。

> 解决方案

利用channel向接收者发送停止消息,goroutine中使用select多路复用接收退出信号。但如果我们想要退出多个goroutine怎么办呢。这时一种可行的方法是使用channel的广播机制向所有goroutine广播消息。  
当一个被关闭的channel中已经发送的数据都被成功接收后，后续的接收操作将不再阻塞，它们会立即返回一个零值。将这个机制拓展一下，不要向channel发送值，而是用关闭一个channel来进行广播。
```go
func gen(done chan struct{}, nums ...int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for _, n := range nums {
			select{
			case out <- n:
			case <- done:
				return
			}
		}
	}()
	return out
}

func main() {
	defer func() {
		time.Sleep(time.Second)
		fmt.Println("the number of goroutines: ", runtime.NumGoroutine())
	}()

	done := make(chan struct{})
	defer close(done)
	gen(done, 2, 3)
	gen(done, 2, 3) // 调用两次，代表创建多个goroutine
}
```

```
the number of goroutines:  1
```

**3. select操作在所有case上阻塞**
```go
func fibonacci(c, quit chan int)  {
	x, y := 0, 1
	for{
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	defer func() {
		time.Sleep(time.Second)
		fmt.Println("the number of goroutines: ", runtime.NumGoroutine())
	}()
	c := make(chan int)
	quit := make(chan int)

	go fibonacci(c, quit)

	for i := 0; i < 10; i++{
		fmt.Println(<- c)
	}

	// close(quit)
}
```

```
the number of goroutines:  2
```
在上面的代码中，我们用一个独立的goroutine作为斐波那契数列的生成器，但是若在主函数的最后忘记向quit发送或关闭quit，那么显然fibonacci将一直运行到main退出，造成goroutine泄露。

> 解决方案
- 使用defer或在程序的最后往quit中发送或关闭quit
- 使用context包
```go
func fibonacci(c chan int, ctx context.Context)  {
	x, y := 0, 1
	for{
		select {
		case c <- x:
			x, y = y, x+y
		case <-ctx.Done():
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	defer func() {
		time.Sleep(time.Second)
		fmt.Println("the number of goroutines: ", runtime.NumGoroutine())
	}()
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	c := make(chan int)

	go fibonacci(c, ctx)

	for i := 0; i < 10; i++{
		fmt.Println(<- c)
	}
}
```
```
the number of goroutines: 1 
```
`context.WithCancel`接收一个`context.Context`作为参数，返回一个附带Done channel的`concext.Context`的拷贝和一个`cancel`函数，当`cancel`被调用时，附带的Done channel就会被关闭。goroutine中通过检查`ctx.Done()`，就可以得知是否应该退出。

**4. nil channel**
向`nil channel`发送和接收数据都将会导致阻塞。这种情况可能在我们定义 channel 时忘记初始化的时候发生。
```go
func main() {
	defer func() {
		time.Sleep(time.Second)
		fmt.Println("the number of goroutines: ", runtime.NumGoroutine())
	}()

	var ch chan int
	go func() {
		<-ch
		// ch <- 1
	}()
}
```
```
the number of goroutines: 2
```
无论是向nil channel中发送还是接受，都会导致阻塞

> 解决方案

不要忘记初始化

**5. 死循环**
同[2.1小节](### 2.1 什么是goroutine泄露)中的情况


#### 2.2.2 同步机制导致的泄露
**1. 锁导致的goroutine泄露**
```go
func main() {
	total := 0

	defer func() {
		time.Sleep(time.Second)
		fmt.Println("the number of goroutines: ", runtime.NumGoroutine())
	}()

	var mutex sync.Mutex
	for i := 0; i < 2; i++ {
		go func() {
			mutex.Lock()
			total += 1
		}()
	}
}
```
在这段代码中，创建了两个goroutine，两个goroutine都要对`total`进行独占访问，但由于第一个goroutine没有解锁，导致第二个goroutine一直阻塞。

> 解决方案

Lock()之后马上`defer Unlock()`

**2. WaitGroup导致的内存泄露**
```go
func handle() {
	var wg sync.WaitGroup

	wg.Add(4)

	go func() {
		wg.Done()
	}()

	go func() {
		wg.Done()
	}()

	go func() {
		wg.Done()
	}()

	wg.Wait()
}

func main() {
	defer func() {
		time.Sleep(time.Second * 2)
		fmt.Println("the number of goroutines: ", runtime.NumGoroutine())
	}()

	go handle()
}
```

```
the number of goroutines:  2
```
在以上代码中，设置并发任务数为4，但其实只有3个任务，所以`wg.Wait`永远不可能满足。handle将一直被阻塞。

> 解决方案

建议不要一次性设置任务数量，尽量在任务启动时通过`wg.Add(1)`的方式来增加任务数。


**未完待续**

### 2.3 goroutine泄露的排查
https://ms2008.github.io/2019/06/02/golang-goroutine-leak/
https://lessisbetter.site/2019/05/18/go-goroutine-leak/

## 3. 控制goroutine的退出

## 4. 对goroutine的监视




1. 野生goroutine中的panic的捕获
2. goroutine泄露
3. 控制goroutine退出
4. goroutine执行超时



# 参考
> 线程池
[Java线程池实现原理及其在美团业务中的实践](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)
[新手一看就懂的线程池！](https://segmentfault.com/a/1190000038220635)
[线程池学习看这篇就够了,万字总结线程池!!!](https://segmentfault.com/a/1190000039097003)

> goroutine泄露
[Golang中的goroutine泄漏问题](https://blog.haohtml.com/archives/19308)
[跟读者聊 Goroutine 泄露的 N 种方法，真刺激！](https://mp.weixin.qq.com/s/ql01K1nOnEZpdbp--6EDYw)
[如何防止 goroutine 泄露](https://juejin.cn/post/6844903891935461383)
[技术解析系列 | PouchContainer Goroutine Leak 检测实践](https://blog.51cto.com/u_13778063/2152654)
[goroutine泄露：原理、场景、检测和防范](https://segmentfault.com/a/1190000019644257)

> 其他
[Go语言野生goroutine的处理](https://blog.csdn.net/liumingzhuo/article/details/115402058)
[goroutine 到底该怎么用](https://studygolang.com/articles/32206)
[如何查看golang程序中有哪些goroutine 正在执行](https://blog.csdn.net/lanyang123456/article/details/106984623)
[Go 语言踩坑记——panic 与 recover](https://xiaomi-info.github.io/2020/01/20/go-trample-panic-recover/)
[Handling panics in go routines](https://stackoverflow.com/questions/50409011/handling-panics-in-go-routines)
