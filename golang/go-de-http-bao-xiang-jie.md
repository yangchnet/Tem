---
title: Go的http包详解
date: '2021-01-22T09:27:21.000Z'
draft: false
---

# Go的http包详解

详细地解剖一下 http 包，看它到底是怎样实现整个过程的。  
Go 的 http 有两个核心功能：Conn、ServeMux

## Conn的goroputine

为了实现高并发和高性能，go使用了goroutine来处理Conn的读写事件，这样每个请求都能保持独立，相互不会阻塞，可以高效的相应网络事件。  
go在等待客户端请求中是这样的：

```go
c, err := srv.newConn(rw)
if err != nil {
    continue
}
go c.serve()
```

可以看到，客户端的每次请求都会创建一个Conn，这个Conn里面保存了该次请求的信息，然后再传递到相应的handler，该handler中便可以读取到相应的header信息，这样保证了每个请求的独立性。

## ServeMux的自定义

```text
它的结构如下：
```go
type ServeMux struct{
    mu sync.RWMutext  // 锁，请求涉及到并发处理，因此需要一个锁机制
    m map[string]muxEntry  // 路由规则，一个String对应一个mux实体，这里的String就是注册的一个路由表达式
    hosts bool  // 是否在任意的规则中带有host信息
}
```

下面看一下muxEntry

```go
type muxEntry struct {
    explicit bool // 是否精确匹配
    h Handler // 这个路由表达式对应哪个handler
    pattern string // 匹配字符串
}
```

在看一下Handler的定义

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)  // 路由实现器
}
```

Handler是一个接口，但是附中的`sayhelloName`函数中并没有实现ServeHTTP这个接口，为什么能添加呢？这是因为http包里面还定义了一个类型`HandlerFunc`，定义的函数`sayhelloName`就是这个HandlerFunc调用之后的结果，**这个类型默认就实现了ServeHTTP这个方法，即我们调用了HandlerFunc\(f\)，强制类型转换f成为HandlerFunc类型，这样f就拥有了ServeHTTP方法。**

```go
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, r)
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request){
    f (w, r)
}
```

路由器里面存储好了对应的路由规则之后，那么具体的请求又是怎么分发的呢？下面的代码，默认的路由器实现了`ServeHTTP`

```go
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request) {
    if r.RequestURL == "*"{
        w.Header().Set("connection", "clost")
        w.WriteHeader(StatusBadReqest)
        return
    }
    h, _ := mux.Handler(r)
    h.ServeHTTP(w, r)
}
```

路由器接收到请求后，如果是`*`那么关闭链接，否则调用`mux.Handler(r)`返回对应设置路由的处理Handler，然后执行`h.ServeHTTP(w, r)`，也就是调用对应路由的handler的ServeHTTP接口，那么mux.Handler\(r\)怎么处理的呢？

```go
func (mux *ServeMux) Handler(r *Request)(h Handler, pattern string){
    if r.Method != "CONNECT" {
        if p := cleanPath(r.URL.Path); p != r.URL.Path{
            _, pattern = mux.handler(r.Host, p)
            return RedirectHandler(p, StatusMovedPermanently), pattern
        }
    }
    return mux.handler(r.Host, r.URL.Path)
}
func (mux *ServeMux) handler(host, path string) (h Handler, pattern string) {
    mux.mu.RLock()
    defer mux.mu.RUnlock()

    // Host-specific pattern takes precedence over generic ones
    if mux.hosts {
        h, pattern = mux.match(host + path)
    }
    if h == nil {
        h, pattern = mux.match(path)
    }
    if h == nil {
        h, pattern = NotFoundHandler(), ""
    }
    return
}
```

根据用户请求的URL和路由器里面存储的map来匹配，当匹配到之后返回存储的handler，调用这个handler的ServeHTTP接口就可以执行到相应的函数。

Go 其实支持外部实现的路由器 ListenAndServe 的第二个参数就是用以配置外部路由器的，它是一个 Handler 接口，即外部路由器只要实现了 Handler 接口就可以，我们可以在自己实现的路由器的 ServeHTTP 里面实现自定义路由功能。 如下代码所示，我们自己实现了一个简易的路由器

```go
import (
    "fmt"
    "net/http"
)

type MyMux struct {
}

func (p *MyMux) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    if r.URL.Path == "/" {
        sayhelloName(w, r)
        return
    }
    http.NotFound(w, r)
    return
}

func sayhelloName(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello myroute!")
}

func main() {
    mux := &MyMux{}
    http.ListenAndServe(":9090", mux)
}
```

## GO代码的执行流程

通过对http包的分析之后，现在让我们来梳理一下整个的代码执行过程。

* 首先调用Http.HandleFunc

  按顺序做了几件事

  1. 调用了DefaultServeMux 的 HandleFunc
  2. 调用了 DefaultServeMux 的 Handle
  3. 往 DefaultServeMux 的 map \[string\] muxEntry 中增加对应的 handler 和路由规则

* 其次调用 http.ListenAndServe \(":9090", nil\)  

  按顺序做了几件事

  1. 实例化 Server
  2. 调用 Server 的 ListenAndServe \(\)
  3. 调用 net.Listen \("tcp", addr\) 监听端口
  4. 启动一个 for 循环，在循环体中 Accept 请求
  5. 对每个请求实例化一个 Conn，并且开启一个 goroutine 为这个请求进行服务 go c.serve \(\)
  6. 读取每个请求的内容 w, err := c.readRequest \(\)
  7. 判断 handler 是否为空，如果没有设置 handler（这个例子就没有设置 handler），handler 就设置为 DefaultServeMux
  8. 调用 handler 的 ServeHttp
  9. 在这个例子中，下面就进入到 DefaultServeMux.ServeHttp
  10. 根据 request 选择 handler，并且进入到这个 handler 的 ServeHTTP

      ```go
      mux.handler(r).ServeHTTP(w, r)
      ```

  11. 选择 handler：
      * 判断是否有路由能满足这个 request（循环遍历 ServeMux 的 muxEntry）
      * B 如果有路由满足，调用这个路由 handler 的 ServeHTTP
      * 如果没有路由满足，调用 NotFoundHandler 的 ServeHTTP

附：

```go
func sayHelloName(w http.ResponseWriter, r *http.Request){
    r.ParseForm()
    fmt.Println(r.Form)
    fmt.Println("path", r.URL.Path)
    fmt.Println("scheme", r.URL.Scheme)
    fmt.Println(r.Form["url_long"])
    for k, v := range r.Form{
        fmt.Println("key: ", k)
        fmt.Println("val: ", strings.Join(v, " "))
    }
    fmt.Println(w, "hello world")
}

func main() {
    http.HandleFunc("/", sayHelloName)
    err := http.ListenAndServe(":9090", nil)
    if err != nil {
        log.Fatal("listenandserver: ", err)
    }
}
```

