---
author: "李昌"
title: "panic和recover"
date: "2021-03-27"
tags: ["golang","panic","recover"]
categories: ["Golang"]
ShowToc: true
TocOpen: true
---

## 1. Panic异常
```go
func panic(v interface{})
```
在通常情况下，函数向其调用方报告错误都是返回一个error类型，但有时会遇到致命（即会让程序崩溃）的错误时，显然无法通过返回error进行处理。这时我们使用panic函数来报告致命错误。
 
当panic异常发生时，程序会中断运行，并立即执行在该goroutine中被defer的函数。随后，程序崩溃并输出日志信息（panic value和函数调用的堆栈信息）。在Go的panic机制中，延迟函数的调用在释放堆栈信息之前.

panic的来源：
    1. 运行时panic异常
    2. 直接调用内置的panic函数
   
例子：  
```go
func main(){
	fmt.Println("main start")
	outerFunc()
	fmt.Println("main end")
}

func outerFunc(){
	fmt.Println("out start")
	innerFunc()
	fmt.Println("out end")
}

func innerFunc(){
	panic(errors.New("an intended fatal error"))
}
```
> 输出
```go
// 只有start，而没有end，因为程序崩溃了
main start
out start

panic: an intended fatal error
```
在这个程序中，当调用`innerFunc`中的panic时，`innerFunc`会立即停止执行，紧接着，`outerFunc`也会被停止，运行时panic沿着调用栈一直反方向进行传播，直至到达当前goroutine的调用栈最顶层。

## 2. recover
```go
func recover() interface{}
```
通常来说，不应该对panic异常做任何处理，但有时我们可能需要在程序崩溃前做一些操作。这时，我们可以“从异常中恢复”。 

如果在defer函数中调用了内置函数recover，并且定义该defer语句的函数发生了panic异常，recover会使程序从panic中恢复，并返回panic value。导致panic异常的函数不会继续运行，但能正常返回。在未发生panic时调用recover，recover会返回nil。

```go
// ...

// 将innerFunc修改为如下
func innerFunc(){
	defer func(){
		if p := recover(); p != nil {
			fmt.Printf("internal error:%v \n",p)
		}
	}()
	panic(errors.New("an intended fatal error"))
}
```
> 输出
```go
// 程序正常结束
main start
out start
internal error:an intended fatal error 
out end
main end
```
