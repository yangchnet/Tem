---
author: "李昌"
title: "Reader和Writer接口"
date: "2021-03-28"
tags: ["interface", "golang", "Reader"]
categories: ["Golang"]
ShowToc: true
TocOpen: true
---

## 1. Reader接口
```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
```
> 接口说明  

Read 将 len(p) 个字节读取到 p 中。它返回读取的字节数 n（0 <= n <= len(p)） 以及任何遇到的错误。即使 Read 返回的 n < len(p)，它也会在调用过程中占用 len(p) 个字节作为暂存空间。若可读取的数据不到 len(p) 个字节，Read 会返回可用数据，而不是等待更多数据。

当 Read 在成功读取 n > 0 个字节后遇到一个错误或 EOF (end-of-file)，它会返回读取的字节数。它可能会同时在本次的调用中返回一个non-nil错误,或在下一次的调用中返回这个错误（且 n 为 0）。 一般情况下, Reader会返回一个非0字节数n, 若 n = len(p) 个字节从输入源的结尾处由 Read 返回，Read可能返回 err == EOF 或者 err == nil。并且之后的 Read() 都应该返回 (n:0, err:EOF)。

调用者在考虑错误之前应当首先处理返回的数据。这样做可以正确地处理在读取一些字节后产生的 I/O 错误，同时允许EOF的出现

> 例子
```go
func ReadFrom(reader io.Reader, num int) ([]byte, error) {
    p := make([]byte, num)
    n, err := reader.Read(p)
    if n > 0 {
        return p[:n], nil
    }
    return p, err
}
```
ReadFrom函数将io.Reader作为参数，也就是说，ReadFrom可以从任何实现了io.Reader接口的地方读取数据。
```go
// 从标准输入读取数据
data, err := ReadFrom(os.Stdin, 11)

// 从普通文件中读取， file = new(os.File)
data, err := Read From(file, 9)

// 从字符串读取
data, err := ReadFrom(strings.NewReader("from string"), 12) 
```

**io.Reader将当前io.Reader实例中的值读取到参数p中**
$$io.Reader \stackrel{data}{\rightarrow}  [\ ]byte$$

## 2. Writer接口
```go
type Writer interface {
    Write(p []byte) (n int, err error)
}
```

> 功能说明
 
Write 将 len(p) 个字节从 p 中写入到基本数据流中。它返回从 p 中被写入的字节数 n（0 <= n <= len(p)）以及任何遇到的引起写入提前停止的错误。若 Write 返回的 n < len(p)，它就必须返回一个 非nil 的错误。

> 例子

在fmt标准库中，有一组函数:Fprint/Fprintf/Fprintln,它们接收一个io.Wirter类型参数，也就是说它们将数据格式化输出到io.Writer中。那么，调用这组函数时，该如何实现这个参数呢？  
以fmt.Fprintln()为例：  
```go
func Println(a ...interface{}) (n int, err error) {
    return Fprintln(os.Stdout, a...)
}
```

**io.Writer将p中的数据写入到io.Wirter实例中**
$$[\ ]byte \stackrel{data}{\rightarrow} io.Writer$$