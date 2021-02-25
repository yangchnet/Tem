---
title: Go语言中的字面量
date: '2021-01-22T09:27:21.000Z'
draft: false
---

# Go语言中的字面量

* [什么是字面量](go-yu-yan-zhong-de-zi-mian-liang.md#什么是字面量)
* [整型和浮点型的字面值](go-yu-yan-zhong-de-zi-mian-liang.md#整型和浮点型的字面值)
* [字符串的字面值](go-yu-yan-zhong-de-zi-mian-liang.md#字符串的字面值)
* [常量的字面值](go-yu-yan-zhong-de-zi-mian-liang.md#常量的字面值)
* [数组的字面值](go-yu-yan-zhong-de-zi-mian-liang.md#数组的字面值)
* [Slice的字面值](go-yu-yan-zhong-de-zi-mian-liang.md#Slice的字面值)
* [Map的字面值](go-yu-yan-zhong-de-zi-mian-liang.md#Map的字面值)
* [结构体的字面值](go-yu-yan-zhong-de-zi-mian-liang.md#结构体的字面值)

## 什么是字面量

在计算机科学中，字面量（literal）是用于表达源代码中一个固定值的表示法（notation）。  
简单的说，字面量或者说字面值就是一个变量的值。

## 整型和浮点型的字面值

```python
var i int = 1
var f float64 = 3.14159
```

## 字符串的字面值

字符串值也可以用字符串面值方式编写，只要将一系列字节序列包含在双引号即可：

```python
"Hello 世界"
`世界`
```

```text
世界
```

一个原生的字符串面值形式是

```go
`...`
```

使用反引号代替双引号。在原生的字符串面值中，没有转义操作；全部的内容都是字面的意思，包含退格和换行，因此一个程序中的原生字符串面值可能跨越多行（译注：在原生字符串面值内部是无法直接写`````````字符的，可以用八进制或十六进制转义或`````+"\`"\`\`\`链接字符串常量完成）。唯一的特殊处理是会删除回车以保证在所有平台上的值都是一样的，包括那些把回车也放入文本文件的系统（注：Windows系统会把回车和换行一起放入文本文件中）。

原生字符串面值用于编写正则表达式会很方便，因为正则表达式往往会包含很多反斜杠。原生字符串面值同时被广泛应用于HTML模板、JSON面值、命令行提示信息以及那些需要扩展到多行的场景。

```python
const GoUsage = `Go is a tool for managing Go source code.

Usage:
    go command [arguments]
...`
GoUsage
```

```text
Go is a tool for managing Go source code.

Usage:
    go command [arguments]
...
```

## 常量的字面值

对于常量面值，不同的写法可能会对应不同的类型。例如0、0.0、0i和\u0000虽然有着相同的常量值，但是它们分别对应无类型的整数、无类型的浮点数、无类型的复数和无类型的字符等不同的常量类型。同样，true和false也是无类型的布尔类型，字符串面值常量是无类型的字符串类型。

## 数组的字面值

使用数组字面值语法用一组值来初始化数组：

```python
var q [3]int = [3]int{1, 2, 3}
var r [3]int = [3]int{1, 2}
r[2]
```

```text
0
```

在数组字面值中，如果在数组的长度位置出现的是“...”省略号，则表示数组的长度是根据初始化值的个数来计算。因此，上面q数组的定义可以简化为

```python
import "fmt"
q := [...]int{1, 2, 3}
fmt.Printf("%T\n", q) // "[3]int"
```

```text
[3]int





7 <nil>
```

数组、slice、map和结构体字面值的写法都很相似。上面的形式是直接提供顺序初始化值序列，但是也可以指定一个索引和对应值列表的方式初始化，就像下面这样：

```python
type Currency int

const (
    USD Currency = iota // 美元
    EUR                 // 欧元
    GBP                 // 英镑
    RMB                 // 人民币
)

//这个看起来和字典很相似，但其实USD等标识是index下标
symbol := [...]string{USD: "$", EUR: "€", GBP: "￡", RMB: "￥"}

fmt.Println(RMB, symbol[RMB]) // "3 ￥"
symbol[EUR] == symbol[1]
```

```text
3 ￥





true
```

## Slice的字面值

slice和数组的字面值语法很类似，它们都是用花括弧包含一系列的初始化元素，但是对于slice并没有指明序列的长度。这会隐式地创建一个合适大小的数组，然后slice的指针指向底层的数组。就像数组字面值一样，slice的字面值也可以按顺序指定初始化值序列，或者是通过索引和元素值指定，或者两种风格的混合语法初始化。

```python
s := []int{0, 1, 2, 3, 4, 5}
s
```

```text
[0 1 2 3 4 5]
```

```python
a := []string{"one", "two", 4:"three"}
a[4]
```

```text
three
```

## Map的字面值

用map字面值的语法创建map，同时还可以指定一些最初的key/value：

```python
ages := map[string]int{
    "alice":   31,
    "charlie": 34,
}
```

这相当于

```python
ages := make(map[string]int)
ages["alice"] = 31
ages["charlie"] = 34
```

## 结构体的字面值

第一种写法

```python
type Point struct{ X, Y int }

p := Point{1, 2}
```

更常用的是第二种写法，以成员名字和相应的值来初始化，可以包含部分或全部的成员

```python
anim := gif.GIF{LoopCount: nframes}
```

```text
repl.go:1:9: undefined "gif" in gif.GIF <*ast.SelectorExpr>
```

两种写法不能混用

