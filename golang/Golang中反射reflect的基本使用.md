---
author: "李昌"
title: "Golang中反射reflect的基本使用"
date: "2021-05-22"
tags: ["reflect", "language", "golang"]
categories: ["Golang"]
ShowToc: true
TocOpen: true
---

> 在计算机领域，反射是指一类应用，它们能够自描述和自控制。也即是说，这类应用通过采用某种机制来实现对自己行为的描述和监测，并能根据自身行为的状态和结果，调整或修改应用所描述行为的状态和相关的语义。
> 反射（reflect）让我们能在运行期探知对象的类型信息和内存结构，这从一定程度上弥(mi)补了静态语言在动态行为上的不足。
> 反射（reflect）是在计算机程序运行时，访问，检查，修改它自身的一种能力，是元编程的一种形式。
> Go语音提供了一种机制在运行时更新变量和检查它们的值、调用它们的方法和它们支持的内在操作，但是在编译时并不知道这些变量的具体类型。这种机制被称为反射。反射也可以让我们将类型本身作为第一类的值类型处理。

## 1. 为何我们需要反射？
`fmt.Fprintf`函数提供字符串格式化处理逻辑，它可以对任意类型的值格式化并打印，甚至支持用户自定义的类型。  
让我们也来尝试实现一个类似功能的函数。为了简单起见，我们的函数只接收一个参数，然后返回和fmt.Sprint类似的格式化后的字符串。我们实现的函数名也叫Sprint。
这里我们使用switch类型分支来对不同的类型进行处理。
```go
func Sprint(x interface{}) string {
    type stringer interface {
        String() string
    }
    switch x := x.(type) {
    case stringer:
        return x.String()
    case string:
        return x
    case int:
        return strconv.Itoa(x)
        // ...similar cases for int16, uint32, and so on...
    case bool:
        if x {
            return "true"
        }
        return "false"
    default:
            // array, chan, func, map, pointer, slice, struct
            return "???"
    }
}
```
但是我们如何处理其它类似`[]float64、map[string][]string`等类型呢？我们当然可以添加更多的测试分支，但是这些组合类型的数目基本是无穷的。还有如何处理`url.Values`等命名的类型呢？虽然类型分支可以识别出底层的基础类型是`map[string][]string`，但是它并不匹配`url.Values`类型，因为它们是两种不同的类型，而且switch类型分支也不可能包含每个类似`url.Values`的类型，这会导致对这些库的循环依赖。   
没有一种方法来检查未知类型的表示方式，我们被卡住了，这就是我们为何需要反射的原因。

## 2. reflect.Type和reflect.Values
![20210522163144](https://raw.githubusercontent.com/lich-Img/blogImg/master/img20210522163144.png)

### 2.1 interface{}和反射
**接口值**    
概念上讲一个接口的值，由两部分组成，一个具体的类型和那个类型的值, 它们被称为接口的动态类型和动态值。在Go的概念模型中，一些提供每个类型信息的值被称为`类型描述符`，比如类型的名称和方法。**在一个接口值中，类型部分代表与之相关类型的描述符**。  
下面4个语句中，变量w得到了3个不同的值（第一个和最后一个是相同的）
```go
var w io.Writer
w = os.Stdout
w = new(bytes.Buffer)
w = nil
```

进一步观察在每一个语句后的w变量的值和动态行为。
1. 第一个语句定义了变量w:
```go
var w io.Writer
```
在Go语言中，变量总是被一个定义明确的值初始化，即使接口类型也不例外。对于一个接口的零值就是它的类型和值的部分都是nil.如下图：
![20210522143522](https://raw.githubusercontent.com/lich-Img/blogImg/master/img20210522143522.png)
一个接口值基于它的动态类型被描述为空或非空，所以这是一个空的接口值。

2. 第二个语句将一个`*os.File`类型的值赋给变量`w`
```go
w = os.Stdout
```
这个赋值过程调用了一个具体类型到接口类型的隐式转换，这和显式的使用`io.Writer(os.Stdout)`是等价的.这个接口值的动态类型被设为`*os.Stdout`指针的类型描述符，它的动态值持有`os.Stdout`的拷贝；这是一个代表处理标准输出的`os.File`类型变量的指针
![20210522144134](https://raw.githubusercontent.com/lich-Img/blogImg/master/img20210522144134.png)
调用一个包含`*os.File`类型指针的接口值的`Write`方法，使得`(*os.File).Write`方法被调用。这个调用输出“hello”。   
```go
w.Write([]byte("hello"))
```
通常在编译期，我们不知道接口值的动态类型是什么，所以一个接口上的调用必须使用动态分配(即运行时分配)。因为不是直接进行调用，所以编译器必须把代码生成在类型描述符的方法Write上，然后间接调用那个地址。这个调用的接收者是一个接口动态值的拷贝：`os.Stdout`(参照上图)。效果和下面这个直接调用一样：
```go
os.Stdout.Write([]byte("hello")) // "hello"
```

3. 第三个语句给接口值赋了一个`*bytes.Buffer`类型的值
```go
w = new(bytes.Buffer)
```
现在动态类型是`*bytes.Buffer`并且动态值是一个指向新分配的缓冲区的指针。
![20210522145148](https://raw.githubusercontent.com/lich-Img/blogImg/master/img20210522145148.png)
`Write`方法的调用也使用了和之前一样的机制。
```go
w.Write([]byte("hello"))  // write "hello" to the bytes.Buffers
```
这次类型描述符是`*bytes.Buffer`，所以调用了`(*bytes.Buffer).Write`方法，并且接收者是该缓冲区的地址。这个调用把字符串“hello”添加到缓冲区中。

4. 最后，第四个语句将nil赋给了接口值
```go
w = nil
```
这个重置将它所有的部分都设为nil值，把变量w恢复到和它之前定义时相同的状态图。
![20210522143522](https://raw.githubusercontent.com/lich-Img/blogImg/master/img20210522143522.png)

interface及其`动态类型，动态值`的存在，是Golang中实现反射的前提，理解了接口的动态类型的动态值，就更容易理解反射。反射就是用来检测存储在接口变量内部动态类型，动态值的一种机制。

### 2.2 类型（Type）
一个Type表示一个Go类型，它是一个接口:reflect.Type()。   
函数`reflect.TypeOf`接受任意的`interface{}`类型，并返回对应动态类型的`reflect.Type`:
```go
t := reflect.TypeOf(3)  // a reflect.Type
fmt.Println(t.String()) // int
fmt.Println(t) // int
```
`TypeOf(3)`调用将3作为`interface{}`类型参数传入。按[2.1节](### 2.1 interface{}和反射)所述，将一个具体的值转为接口类型会有一个隐式的接口转换操作，它会创建一个包含两个信息的接口值：操作数的动态类型（这里是int）和它的动态的值（这里是3）。

因为`reflect.TypeOf`返回的是一个接口的动态类型值，它总是返回具体的类型，因此下面的代码将打印“*os.File”而不是“io.Writer”.
```go
var w io.Writer = os.Stdout
fmt.Println(reflect.TypeOf(w)) // "*os.File"
```

`reflect.Type`接口是满足`fmt.Stringer`接口的。因为打印动态类型值对于调试和日志是很有帮助的，`fmt.Printf`提供了一个简短的`%T`标志参数，内部使用`reflect.TypeOf`的结果输出。
```go
fmt.Printf("%T\n", 3) // "int"
```

### 2.3 值（Value）
一个`reflect.Value`可以持有一个任意类型的值，函数`reflect.ValueOf`接受任意的`interface{}`类型，并返回对应动态类型的`reflect。Value`。   
与`reflect.TypeOf`类似，`reflect.ValueOf`返回的结果也是对于具体的类型，但是`reflect.Value`也可以持有一个接口值。
```go
v := reflect.ValueOf(3) // a reflect.Value
fmt.Println(v) // "3"
fmt.Printf("%v\n", v) // "3"
fmt.Println(v.String()) // NOTE: "<int Value>
```
和`reflect.Type` 类似, `reflect.Value` 也满足 `fmt.Stringer` 接口, 但是除非 Value 持有的是字符串,否则 String 只是返回具体的类型. 相同, 使用 `fmt` 包的 `%v` 标志参数, 将使用 `reflect.Values` 的结果格式化.

调用`Value`的`Type`方法将返回具体类型所对应的`reflect.Type`
```go
t := v.Type()
fmt.Println(t.String()) // int
```
逆操作是调用 reflect.ValueOf 对应的 reflect.Value.Interface 方法. 它返回一个 interface{} 类型
表示 reflect.Value 对应类型的具体值:
```go
v := reflect.ValueOf(3) // a reflect.Value
x := v.Interface() // an interface{}
i := x.(int) // an int
fmt.Printf("%d\n", i) // "3"
```
一个 `reflect.Value` 和 `interface{}` 都能保存任意的值. 所不同的是, 一个空的接口隐藏了值对应的表示方式和所有的公开的方法, 因此只有我们知道具体的动态类型才能使用类型断言来访问内部的值(就像上面那样), 对于内部值并没有特别可做的事情. 相比之下, 一个 `Value` 则有很多方法来检查其内容, 无论它的具体类型是什么.

### 2.4 再次尝试format.Any
我们使用 `reflect.Value` 的 `Kind` 方法来替代之前的类型switch. 虽然还是有无穷多的类型, 但是它们的`kinds`类型却是有限的: Bool, String 和 所有数字类型的基础类型; Array 和 Struct 对应的聚合类型; Chan, Func, Ptr, Slice, 和 Map 对应的引用类似; 接口类型; 还有表示空值的无效类型. (空的 reflect.Value 对应 Invalid 无效类型.   
`reflect.Value.Kind()`返回`reflect.Value`的基类型。即对应类型的底层表示。
```go
var f func(string)int
v := reflect.TypeOf(f)
fmt.Println(v, v.Kind())  // "func(string)int"  "int"
v.Kind() == reflect.Func  // true
```


```go
package format

import (
	"reflect"
	"strconv"
)

// Any formats any value as a string.
func Any(value interface{}) string {
	return formatAtom(reflect.ValueOf(value))
}

// formatAtom formats a value without inspecting its internal structure.
func formatAtom(v reflect.Value) string {
	switch v.Kind() {
	case reflect.Invalid:
		return "invalid"
	case reflect.Int, reflect.Int8, reflect.Int16,
		reflect.Int32, reflect.Int64:
		return strconv.FormatInt(v.Int(), 10)
	case reflect.Uint, reflect.Uint8, reflect.Uint16,
		reflect.Uint32, reflect.Uint64, reflect.Uintptr:
		return strconv.FormatUint(v.Uint(), 10)
		// ...floating-point and complex cases omitted for brevity...
	case reflect.Bool:
		return strconv.FormatBool(v.Bool())
	case reflect.String:
		return strconv.Quote(v.String())
	case reflect.Chan, reflect.Func, reflect.Ptr, reflect.Slice, reflect.Map:
		return v.Type().String() + " 0x" +
			strconv.FormatUint(uint64(v.Pointer()), 16)
	default: // reflect.Array, reflect.Struct, reflect.Interface
		return v.Type().String() + " value"
	}
}
```

## 3. 三大法则
### 3.1 从`interface{}`变量可以反射出反射对象
当我们执行 `reflect.ValueOf(1)` 时，虽然看起来是获取了基本类型 int 对应的反射类型，但是由于 `reflect.TypeOf`、`reflect.ValueOf` 两个方法的入参都是 `interface{}` 类型，所以在方法执行的过程中发生了类型转换。

因为Go 语言的函数调用都是值传递的，所以变量会在函数调用时进行类型转换。基本类型 `int` 会转换成 `interface{}` 类型，这也就是为什么第一条法则是从接口到反射对象。

上面提到的 `reflect.TypeOf` 和 `reflect.ValueOf` 函数就能完成这里的转换，如果我们认为 Go 语言的类型和反射类型处于两个不同的世界，那么这两个函数就是连接这两个世界的桥梁。
![20210522164336](https://raw.githubusercontent.com/lich-Img/blogImg/master/img20210522164336.png)
请看以下例子：
```go
author := "Bob"
fmt.Println("TypeOf author:", reflect.TypeOf(author))  // TypeOf author: string

fmt.Println("ValueOf author:", reflect.ValueOf(author)) // ValueOf author: Bob
```
有了变量的类型之后，我们可以通过 Method 方法获得类型实现的方法，通过 Field 获取类型包含的全部字段。对于不同的类型，我们也可以调用不同的方法获取相关信息：

- 结构体：获取字段的数量并通过下标和字段名获取字段 StructField；
- 哈希表：获取哈希表的 Key 类型；
- 函数或方法：获取入参和返回值的类型；
- …
总而言之，使用 `reflect.TypeOf`和 `reflect.ValueOf` 能够获取 Go 语言中的变量对应的反射对象。一旦获取了反射对象，我们就能得到跟当前类型相关数据和操作，并可以使用这些运行时获取的结构执行方法。

### 3.2 从反射对象可以获取`interface{}`变量
既然能够将接口类型的变量转换成反射对象，那么一定需要其他方法将反射对象还原成接口类型的变量，`reflect`包中的`reflect.Value.Interface`就能完成这项工作：
```go
v := reflect.ValueOf(3) // a reflect.Value
x := v.Interface() // an interface{}
i := x.(int) // an int
fmt.Printf("%d\n", i) // "3"
```
![20210522165432](https://raw.githubusercontent.com/lich-Img/blogImg/master/img20210522165432.png)

`reflect.ValueOf`、`reflect.TypeOf`与`Interface`方法可以说是一个互逆的过程。`reflect.ValueOf`、`reflect.TypeOf`将`interface{}`转化为reflect对象，而`Interface`将一个reflect对象重新转化为一个接口值。
![20210522170413](https://raw.githubusercontent.com/lich-Img/blogImg/master/img20210522170413.png)


### 3.3 要修改反射对象，其值必须可设置
假如我们想要更新一个`reflect.Value`,那么它持有的值一定是可以被更新的，假设有如下代码：
```go
i := 1
v := reflect.ValueOf(i)
v.SetInt(10)
fmt.Println(i) // panic: reflect: reflect.flag.mustBeAssignable using unaddressable value
```
运行上述代码会导致程序崩溃并报出 “reflect: reflect.flag.mustBe Assignableusing unaddressable value” 错误，仔细思考一下就能够发现出错的原因：由于 Go 语言的函数调用都是传值的，所以我们得到的反射对象跟最开始的变量没有任何关系，那么直接修改反射对象无法改变原始变量，程序为了防止错误就会崩溃。

想要修改原变量只能使用如下方法：
```go
i := 1
v := reflect.ValueOf(&i)
v.Elem().SetInt(10)
fmt.Println(i)
```
`reflect.Value.Elem()`方法获取指针指向的变量

## 4. reflect场景实践

1. 动态调用函数（无参数）
```go
type T struct {}

func main() {
    name := "Do"
    t := &T{}
    reflect.ValueOf(t).MethodByName(name).Call(nil)
}

func (t *T) Do() {
    fmt.Println("hello")
}
```

2. 动态调用函数（有参数）
```go
type T struct{}

func main() {
    name := "Do"
    t := &T{}
    a := reflect.ValueOf(1111)
    b := reflect.ValueOf("world")
    in := []reflect.Value{a, b}
    reflect.ValueOf(t).MethodByName(name).Call(in)
}

func (t *T) Do(a int, b string) {
    fmt.Println("hello" + b, a)
}
```
3. 处理返回值中的错误
返回值也是 Value 类型，对于错误，可以转为 interface 之后断言
```go
type T struct{}

func main() {
    name := "Do"
    t := &T{}
    ret := reflect.ValueOf(t).MethodByName(name).Call(nil)
    fmt.Printf("strValue: %[1]v\nerrValue: %[2]v\nstrType: %[1]T\nerrType: %[2]T", ret[0], ret[1].Interface().(error))
}

func (t *T) Do() (string, error) {
    return "hello", errors.New("new error")
}
```

4. struct tag 解析
```go
type T struct {
    A int    `json:"aaa" test:"testaaa"`
    B string `json:"bbb" test:"testbbb"`
}

func main() {
    t := T{
        A: 123,
        B: "hello",
    }
    tt := reflect.TypeOf(t)
    for i := 0; i < tt.NumField(); i++ {
        field := tt.Field(i)
        if json, ok := field.Tag.Lookup("json"); ok {
            fmt.Println(json)
        }
        test := field.Tag.Get("test")
        fmt.Println(test)
    }
}
```

5. 类型转换和赋值
```go
type T struct {
    A int    `newT:"AA"`
    B string `newT:"BB"`
}

type newT struct {
    AA int
    BB string
}

func main() {
    t := T{
        A: 123,
        B: "hello",
    }
    tt := reflect.TypeOf(t)
    tv := reflect.ValueOf(t)

    newT := &newT{}
    newTValue := reflect.ValueOf(newT)

    for i := 0; i < tt.NumField(); i++ {
        field := tt.Field(i)
        newTTag := field.Tag.Get("newT")
        tValue := tv.Field(i)
        newTValue.Elem().FieldByName(newTTag).Set(tValue)
    }

    fmt.Println(newT)
}
```

6. 通过 kind（）处理不同分支
```go
func main() {
    a := 1
    t := reflect.TypeOf(a)
    switch t.Kind() {
    case reflect.Int:
        fmt.Println("int")
    case reflect.String:
        fmt.Println("string")
    }
}
```

7. 判断实例是否实现了某接口
```go
type IT interface {
    test1()
}

type T struct {
    A string
}

func (t *T) test1() {}

func main() {
    t := &T{}
    ITF := reflect.TypeOf((*IT)(nil)).Elem()
    tv := reflect.TypeOf(t)
    fmt.Println(tv.Implements(ITF))
}
```

## Reference
《The Go Programmer Language》   
[Golang的反射reflect深入理解和示例](https://juejin.cn/post/6844903559335526407)    
[Go 语言设计与实现](https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-reflect/)    
[《Go学习笔记 . 雨痕》反射](https://www.cnblogs.com/52php/p/6340487.html)    
[Go Reflect 高级实践](https://segmentfault.com/a/1190000016230264)     
[Package reflect](https://golang.org/pkg/reflect)     









