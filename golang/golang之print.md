---
author: "李昌"
title: "golang中的print系函数详解"
date: "2021-02-25"
tags: ["GoLang"]
categories: ["GoLang"]
ShowToc: true
TocOpen: true
---

# golang中的print系函数详解

pirnt系函数来自fmt包，主要用于做各种格式的输出
这些函数主要有

1. [fmt.Fprintf](#fmt.Fprintf)
2. [fmt.Printf](#fmt.Printf)
3. [fmt.Sprintf](#fmt.Sprintf)
4. [fmt.Fprint](#fmt.Fprint)
5. [fmt.Print](#fmt.Print)
6. [fmt.Sprint](#fmt.Sprint)
7. [fmt.Fprintln](#fmt.Fprintln)
8. [fmt.Println](#fmt.Println)
9. [fmt.Sprintln](#fmt.Sprintln)  
10. [总结](#总结)

下面来逐个分析


```go
import (
    "fmt"
    "os"
    "io"
)
```

## fmt.Fprintf

* 函数原型：    
```go
Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error)
```

* 官方注释    
Fprintf formats according to a format specifier and writes to w.It returns the number of bytes written and any write error encountered.

* Arguement  
```fmt.Fprintf()``` 依据指定的格式向第一个参数内写入字符串，第一参数必须实现了 ```io.Writer``` 接口。```Fprintf()``` 能够写入任何类型，只要其实现了 ```Write``` 方法，包括 ```os.Stdout```,文件（例如 ```os.File```），管道，网络连接，通道等等，同样的也可以使用 ```bufio``` 包中缓冲写入。bufio 包中定义了 ```type Writer struct{...}```。```Fprintf```：来格式化并输出到 io.Writers 而不是 os.Stdout。


```go
// example
func ExampleFprintf() {
	const name, age = "Kim", 22
	n, err := fmt.Fprintf(os.Stdout, "%s is %d years old.\n", name, age)

	// The n and err return values from Fprintf are
	// those returned by the underlying io.Writer.
	if err != nil {
		fmt.Fprintf(os.Stderr, "Fprintf: %v\n", err)
	}
	fmt.Printf("%d bytes written.\n", n)

	// Output:
	// Kim is 22 years old.
	// 21 bytes written.
}
ExampleFprintf()
```

    Kim is 22 years old.
    21 bytes written.
    

## fmt.Printf

* 函数原型  
```go
Printf(format string, a ...interface{}) (n int, err error)
```

* 官方注释  
Printf formats according to a format specifier and writes to standard output.It returns the number of bytes written and any write error encountered.

* Arguement  
只可以打印出格式化的字符串。只可以直接输出字符串类型的变量（不可以输出整形变量和整形 等），其实这个函数是```fmt.Fprintf```的调用，其调用语句```return Fprintf(os.Stdout, format, a...)```，输出对象为标准输出。

* optional  

|optional|description|  
|----|----|    
|%d|十进制整数|  
|%x, %o, %b| 十六进制，八进制，二进制整数|  
|%f, %g, %e| 浮点数，3.141593 3.141592653589793 3.141593e+00|  
|%t| 布尔 ture或false|  
|%c|字符（rune）（Uniode码点）|  
|%s|字符串|  
|%q|带双引号的字符串"abc"或带单引号的字符'c'|  
|%v|变量的自然形式|  
|%T|变量的类型|  
|%%|字面上的百分号标志（无操作数）|



```go
fmt.Printf("整数：%d\t浮点数：%v\t字符串：%s\n", 1, 3.14, "hello world")
func ExamplePrintf() {
	const name, age = "Kim", 22
	fmt.Printf("%s is %d years old.\n", name, age)

	// It is conventional not to worry about any
	// error returned by Printf.

	// Output:
	// Kim is 22 years old.
}
ExamplePrintf()
```

    整数：1	浮点数：3.14	字符串：hello world
    Kim is 22 years old.
    

## fmt.Sprintf

* 函数原型  
```go
func Sprintf(format string, a ...interface{}) string
```  

* 官方注释  
Sprintf formats according to a format specifier and returns the resulting string.

* Argument  
格式化并返回一个字符串而不带任何输出。可用于给其他函数传递格式化参数。


```go
func ExampleSprintf() {
	const name, age = "Kim", 22
	s := fmt.Sprintf("%s is %d years old.\n", name, age)

	io.WriteString(os.Stdout, s) // Ignoring error for simplicity.

	// Output:
	// Kim is 22 years old.
}
ExampleSprintf()
```

    Kim is 22 years old.
    

## fmt.Fprint

* 函数原型  
```go
Fprint(w io.Writer, a ...interface{}) (n int, err error)
```  

* 官方注释  
Fprint formats using the default formats for its operands and writes to w.Spaces are added between operands when neither is a string.It returns the number of bytes written and any write error encountered.

* Argument  
向```io.Writer```对象中输入字符，第一个参数为输入对象，后面的参数为字符串，直接输出


```go
func ExampleFprint() {
	const name, age = "Kim", 22
	n, err := fmt.Fprint(os.Stdout, name, " is ", age, " years old.\n")

	// The n and err return values from Fprint are
	// those returned by the underlying io.Writer.
	if err != nil {
		fmt.Fprintf(os.Stderr, "Fprint: %v\n", err)
	}
	fmt.Print(n, " bytes written.\n")

	// Output:
	// Kim is 22 years old.
	// 21 bytes written.
}
ExampleFprint()
```

    Kim is 22 years old.
    21 bytes written.
    

## fmt.Print

* 函数原型  
```go
func Print(a ...interface{}) (n int, err error) {
	return Fprint(os.Stdout, a...)
}
```  

* 官方注释  
Print formats using the default formats for its operands and writes to standard output.Spaces are added between operands when neither is a string.It returns the number of bytes written and any write error encountered.  

* Arguement  
显然，这个函数是对```fmt.Fprint```的调用，只是把输入对象默认为标准输出。


```go
func ExamplePrint() {
	const name, age = "Kim", 22
	fmt.Print(name, " is ", age, " years old.\n")

	// It is conventional not to worry about any
	// error returned by Print.

	// Output:
	// Kim is 22 years old.
}
ExamplePrint()
```

    Kim is 22 years old.
    

## fmt.Sprint

* 函数原型  
```go
Sprint(a ...interface{}) string  
```

* 官方注释  
Sprint formats using the default formats for its operands and returns the resulting string.Spaces are added between operands when neither is a string.

* Arguement  
格式化并返回一个字符串而不带任何输出


```go
func ExampleSprint() {
	const name, age = "Kim", 22
	s := fmt.Sprint(name, " is ", age, " years old.\n")

	io.WriteString(os.Stdout, s) // Ignoring error for simplicity.

	// Output:
	// Kim is 22 years old.
}
ExampleSprint()
```

    Kim is 22 years old.
    

## fmt.Fprintln

* 函数原型  
```go
Fprintln(w io.Writer, a ...interface{}) (n int, err error)
```

* 官方注释  
Fprintln formats using the default formats for its operands and writes to w.Spaces are always added between operands and a newline is appended.It returns the number of bytes written and any write error encountered.

* Arguement  
输入字符串到```io.Writer```,并且带有换行


```go
func ExampleFprintln() {
	const name, age = "Kim", 22
	n, err := fmt.Fprintln(os.Stdout, name, "is", age, "years old.")

	// The n and err return values from Fprintln are
	// those returned by the underlying io.Writer.
	if err != nil {
		fmt.Fprintf(os.Stderr, "Fprintln: %v\n", err)
	}
	fmt.Println(n, "bytes written.")

	// Output:
	// Kim is 22 years old.
	// 21 bytes written.
}
ExampleFprintln()
```

    Kim is 22 years old.
    21 bytes written.
    

## fmt.Println

* 函数原型  
```go
func Println(a ...interface{}) (n int, err error) {
	return Fprintln(os.Stdout, a...)
}
```

* 官方注释  
Println formats using the default formats for its operands and writes to standard output.Spaces are always added between operands and a newline is appended.It returns the number of bytes written and any write error encountered.

* Arguement  
显然这个函数是对```fmt.Fprintln```的调用，输入到```os.Stdout```


```go
func ExamplePrintln() {
	const name, age = "Kim", 22
	fmt.Println(name, "is", age, "years old.")

	// It is conventional not to worry about any
	// error returned by Println.

	// Output:
	// Kim is 22 years old.
}
ExamplePrintln()
```

    Kim is 22 years old.
    

## fmt.Sprintln

* 函数原型  
```go
Sprintln(a ...interface{}) string
```

* 官方注释 
Sprintln formats using the default formats for its operands and returns the resulting string.Spaces are always added between operands and a newline is appended.  

* Arguement  
返回一个字符串，不做任何输出


```go
func ExampleSprintln() {
	const name, age = "Kim", 22
	s := fmt.Sprintln(name, "is", age, "years old.")

	io.WriteString(os.Stdout, s) // Ignoring error for simplicity.

	// Output:
	// Kim is 22 years old.
}
ExampleSprintln()
```

    Kim is 22 years old.
    

## 总结

所有的函数名都是```_print_```格式的，其中```_```代表某个标志，标志及其含义如下：  

|标志|含义|  
|-----|-----|  
|_f|格式化|  
|_ln|换行|  
|F_|输入到某个对象|  
|S_|只返回而不做输出|  

且，  
```fmt.Fprintf```和```fmt.Printf```是一对    
```fmt.Fprint```和```fmt.Print```是一对  
```fmt.Fprintln```和```fmt.Println```是一对    
不带```F```前缀的函数均是对带```F```前缀函数的调用，其输入对象均设置为标准输出
