---
author: "李昌"
title: "golang中context包的使用"
date: "2021-05-23"
tags: ["golang", "context"]
categories: ["Golang"]
ShowToc: true
TocOpen: true
---

> context包定义了Context类型，这个类型在API边界即进程中传递截止日期、同步信号，请求值等相关信息。

## 1. 对context包的介绍
在服务器的传入请求中应包含`context`，而对服务器的传出调用应接收一个`context`。它们之间的调用链必须包含`context`，或是衍生的`WithCancel`, `WithDeadline`, `WithTimeout`, `WithValue`。当一个`WithCancel Context`被“cancel”，那么当前`context`所派生的所有`context`也都将被取消。

`WithCancel`, `WithDeadline`, `WithTimeout`接收一个`Context`对象（父对象），并返回其父对象的一个携带有`cancel/deadline/timeout`的一个拷贝（子对象）。调用`CancelFunc`会取消其子对象及子对象的子对象等，删除父对象对子对象的引用，并停止所有关联的计时器。未能调用`CancelFunc`将造成父对象结束前或计时器被触发前子对象的泄露。使用`go vet`工具可以检查所有控制流路径上是否都使用了`CancelFunc`

使用`context`的程序应遵循以下规则，以使各个包之间的接口保持一致，并启用静态分析工具来检查上下文传播：    
1. 不要将`context`存储在结构类型中，而是将`context`明确传递给需要它的每个函数。`Context`应该是第一个函数，通常命名为`ctx`。
```go
func DoSomething(ctx context.Context, arg Arg) error {
    // ...use ctx...
}
```
2. 不要传递一个值为nil的`context`，即使一个函数允许这样做。如果你不确定`Context`的作用那就请传递`context.TODO`。
3. 只在进程和API间传递请求范围数据时使用`context`值，不要用于将可选参数传递给函数。
4. 同样的`Context`可以传递给运行在不同`goroutine`中的函数，`Context`是线程安全的。

## 2. Context接口
```go
type Context interface {
    Done() <-chan struct{}
    Err() error
    Deadline() (deadline time.Time, ok bool)
    Value(key interface{}) interface{}
}
```
`Context`是一个接口，其定义非常的简单，只包含4个方法：
- Done() <-chan struct{}
  `Done()`方法将一个`channel`作为取消信号返回给持有`context`的函数，当该`channel`被关闭（即`Done()`被调用），这些函数应该立即停止其工作并返回。

- Err() error
  `Err()`返回一个`Error`，说明为什么取消`context`。如果`Done()`没有被调用，那么`Err`返回nil。

- Deadline() (deadline time.Time, ok bool)
  `Deadline()`方法返回持有这个`context`的函数的预期结束时间。如果并没有设置`deadline`，那么返回的`ok`将被设置为`false`。

- Value(key interface{}) interface{}
  `Value()`方法返回与此`context`关联的`key`，如果没有与`key`对应的值那么返回nil。   
  `key`可以是任何支持比较的类型，为了避免冲突，应将`key`定义为不可导出的。   
  示例：
  ```go
  package user
  import "context"

  type User struct{...}

  type key int

  var userKey key

  func NewContext(ctx context.Context, u *User) context.Context {
      return context.WithValue(ctx, userKey, u)
  }

  func FromContext(ctx context.Context)(*User, bool){
      u, ok := ctx.Value(userKey).(*User)
      return u, ok
  }
  ```

## 3. Context构造
构造一个`context`对象有两种方法。
```go
func Background() Context 

func TODO() Context 
```
上面两个方法都会返回一个非nil，非空的`Context`对象。`Background()`一般用于构造出最初的`Context`，所有的`Context`都派生自它。`TODO()`用当传入的方法不确定是哪种类型的`Context`时，为了避免`Context`参数为nil而初始化的`Context`。

构造出`Context`对象后，我们就可以使用`WithCancel`, `WithDeadline`, `WithTimeout`, `WithValue`来进一步的设置`Context`，构造出的`Context`都派生自`Background`或是`TODO`
![20210523145028](https://raw.githubusercontent.com/lich-Img/blogImg/master/img20210523145028.png)

## 4. context.With...
### 4.1 context.WithCancel()
```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc) 
```
`WithCancel`接收一个父`context`并返回该父`context`的一个持有`Done channel`的子`context`和一个`cancel`方法，当`cancel`方法被调用时或是父`context`的`Done channel`被关闭时，当前`context`的`Done channel`将被关闭。  

`WithCancel`常被用于通知`goroutine`退出。
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

### 4.2 context.WithValue()
```go
func WithValue(parent Context, key, val interface{}) Context 
```
`WithValue`方法接收一个父`context`，以及一个键值对，返回一个包含这个键值对的子`context`。可使用`context.Value(key)`方法取出其中保存的值。
```go
func main() {
	ctx, cancel := context.WithCancel(context.Background())

	valueCtx := context.WithValue(ctx, key, "add value")

	go watch(valueCtx)
	time.Sleep(10 * time.Second)
	cancel()

	time.Sleep(5 * time.Second)
}

func watch(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			//get value
			fmt.Println(ctx.Value(key), "is cancel")

			return
		default:
			//get value
			fmt.Println(ctx.Value(key), "int goroutine")

			time.Sleep(2 * time.Second)
		}
	}
}
```

### 4.3 context.WithDeadline()
```go
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc)
```
`WithDeadline`方法接收一个父`context`和一个截止时间`d`，返回子`context`并为其设置一个不晚于`d`的截止时间。如果父`context`的截止时间已经早于`d`, 那么`WithDeadline`在语义上与父`context`相同。当以下三种情况发生时：1.截止时间到来，2.`cancelFunc`被调用，3.父`context`的`Done channel`被关闭，当前`context`的`Done channel`将被关闭。
```go
func main() {
	d := time.Now().Add(5 * time.Second) // 5秒后到期
	ctx, cancel := context.WithDeadline(context.Background(), d)
	defer cancel() // 一个好的习惯是调用cancel()以防止goroutine泄露

	go doSomething(ctx)

	time.Sleep(10 * time.Second)
}

func doSomething(ctx context.Context) {
	for {
		select {
		case <-time.After(time.Second):
			fmt.Println("I'm doing some funny things")
		case <-ctx.Done():  // 5秒时到期
			fmt.Println(ctx.Err())
			return
		}
	}
}
```
```
I'm doing some funny things
I'm doing some funny things
I'm doing some funny things
I'm doing some funny things
context deadline exceeded
```

### 4.4 context.WithTimeout()
```go
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
	return WithDeadline(parent, time.Now().Add(timeout))
}
```
从其实现就可以看出，`WithTimeout`只是对`WithDeadline`的进一步封装，这层封装为`context`设置了一个截止时间，也就是规定了其超时时间。
```go
func main() {
	ctx, cancel := context.WithTimeout(context.Background(), time.Duration(2*time.Second)) // 超过两秒就退出，不再continue
	defer cancel() // 一个好的习惯是调用cancel()以防止goroutine泄露

	go seek(ctx)

	time.Sleep(5 * time.Second)
}

func seek(ctx context.Context) {
	for {
		select {
		case <-time.After(time.Second):
			fmt.Println("I'm looking for something")
		case <-ctx.Done():
			fmt.Println(ctx.Err())
			return
		}
	}
}
```

## 5. context包的其他函数
### 5.1 context.String()
实现`fmt.Stringer`接口，用于打印`context`
```go
func main() {
	backgroundCtx := context.Background()
	fmt.Println(backgroundCtx)

	withValueCtx := context.WithValue(backgroundCtx, "one", 1)
	fmt.Println(withValueCtx)

	withCancelCtx, cancel := context.WithCancel(withValueCtx)
	defer cancel()
	fmt.Println(withCancelCtx)

	d := time.Now().Add(2 * time.Second)
	withDeadlineCtx, cancel := context.WithDeadline(withCancelCtx, d)
	defer cancel()
	fmt.Println(withDeadlineCtx)

	withTimeoutCtx, cancel := context.WithTimeout(withDeadlineCtx, time.Duration(2 * time.Second))
	defer cancel()
	fmt.Println(withTimeoutCtx)
}
```

```
context.Background
context.Background.WithValue(type string, val <not Stringer>)
context.Background.WithValue(type string, val <not Stringer>).WithCancel
context.Background.WithValue(type string, val <not Stringer>).WithCancel.WithDeadline(2021-05-23 15:49:31.513587636 +0800 CST m=+2.000147913 [1.999897372s])
context.Background.WithValue(type string, val <not Stringer>).WithCancel.WithDeadline(2021-05-23 15:49:31.513587636 +0800 CST m=+2.000147913 [1.999881255s]).WithCancel
```

### 5.2 context.Value()
用于从`WithValue context`中根据`key`取值
```go
func main() {
	ctx := context.WithValue(context.Background(), "one", 1)
	fmt.Println(ctx.Value("one"))
}
```
值取出后`context`不会删除它，可重复取值，对一个不存在的`key`取值会返回nil。

![context](https://raw.githubusercontent.com/lich-Img/blogImg/master/context.png)



## Reference
[Package context](https://golang.org/pkg/context/)     
[Go Concurrency Patterns: Context](https://blog.golang.org/context)    
[Golang Context深入理解](https://juejin.cn/post/6844903555145400334)   
[Golang Context 原理与实战](https://segmentfault.com/a/1190000022534841)    
[Go 语言设计与实现](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-context/)    





