---
author: "李昌"
title: "类型断言"
date: "2021-02-25"
tags: ["GoLang"]
categories: ["GoLang"]
ShowToc: true
TocOpen: true
---

# 类型断言

类型断言是一个**使用在接口上的操作**，语法上看起来像是x.(T)，因此被称为断言类型，这里x是接口，T是类型。一个类型断言检查它操作对象的动态类型是否和断言的类型匹配。

这里有两种可能：
1. 如果断言的类型T是一个具体类型  
类型断言检查x的动态类型是否和T相同。如果检查成功了，类型断言的结果是x的动态值，即T。换句话说，**具体类型的类型断言从它的操作对象中获得具体的值**。如果检查失败，接下来这个操作会panic。


```go
import (
    "io"
    "os"
    "bytes"
    "fmt"
)
```


```go
var w io.Writer
w = os.Stdout
f := w.(*os.File)   // 类型检查成功了，所以f的值为os.Stdout
f == os.Stdout   // true
fmt.Printf("%p", f)
```

    0xc0004560c0




    12 <nil>




```go
c := w.(*bytes.Buffer)   // 类型检查失败
```


    interface conversion: <io.Writer> is <*os.File>, not <*bytes.Buffer>


2. 断言的类型T是一个接口类型  
```go
t, ok := i.(T)
```
如果i是类型T（实现了T接口），即检查成功了，那么t将是i的原值，ok为true；如果检查失败了，t将为T类型的零值，ok为false，并且不引发panic。   
**对一个接口类型的类型断言改变了类型的表述方式，改变了可以获取的方法集合（通常更大）， 但是它保护了接口值内部的动态类型和值的部分。**


```go
var w io Wirter
w = os.Stdout
rw := w.(io.ReadWriter)
w = new(ByterCounter)
rw = w.(io.ReadWriter)
```


    repl.go:1:10: expected ';', found 'IDENT' Wirter


如果断言操作的对象是一个nil接口值，那么不论被断言的是什么类型，断言都会失败。  
对一个更少限制性的接口类型（更少的方法集合）做断言，因为它表现的就像赋值操作一样，除了对于nil接口值的情况。
