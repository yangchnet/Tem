---
title: "golang中json数据的处理"
date: 2021-01-22T17:27:21+08:00
draft: false
---
# golang中json数据的处理

go中有关json处理的包为：```encoding/json```

## json的编码

要编码JSON数据，我们使用Marshal函数。
```go
func Marshal(v interface{}) ([]byte, error)
```

给定go结构体Message：


```go
type Message struct{
    Name string
    Body string
    Time int64
}
```

和一个实例：


```go
m := Message{"Alice", "hello", 1294706395881547000}
```

那么可以使用如下命令编码json数据


```go
import "encoding/json"
b, err := json.Marshal(m)
```


```go
,
```
