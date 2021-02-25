---
author: "李昌"
title: "http/template"
date: "2021-02-25"
tags: ["GoLang"]
categories: ["GoLang"]
ShowToc: true
TocOpen: true
---

# http/template

## 什么是模板

模板是一种常见的视图，通过它我们可以传递数据以使该视图有意义。可以以任何方式对其进行自定义以获取任何可能的输出。

## 模板包

Go中的模板附带两个包text/template和html/template。文本包允许我们使用模板插入文本，而HTML模板通过提供安全的HTML代码来帮助我们。

## Part of template

### 1. 模板动作

模板动作是主要的控制流程，数据评估功能。这些动作控制最终输出将如何显示
```template
{{ /* a comment isside template */ }}
```

### 2. 控制结构

控制结构确定模板的控制流程，有助于产生结构化的输出，以下是模板中的一些控制结构  
**if语句**  
```html
{{ if .condition }} {{ else }} {{ end }}
```
循环块  
```html
{{ range .Items }} {{ end }}
```

### 3. 功能

函数也可以在模板内部使用，可以使用管道符```|```来使用预定义的函数

> 如何预定义函数

下面的代码创建并分析上面定义的模板templ。注意方法调用链的顺序:template.New先创建并返回一个模板;Funcs方法将daysAgo等自定义函数注册到模板中,并返回模板;最后调用Parse函数分析模板。

```go
report, err := template.New("report").Funcs(template.FuncMap{"daysAgo": daysAgo}).Parse(templ)
if err != nil {
    log.Fatal(err)
}
```

## 在Go中解析模板

现在，我们来解析一些文本和HTML模板

### 1. 访问数据

要访问传递的数据，使用点```.```，如下所示：  
```
{{ .data }}
```

### 2. 解析文本模板

现在，来解析一个文本模板  
```go
package main
 
import (
    "os"
    "text/template"
)
 
type User struct {
    Name    string
    Bio     string
}
 
func main() {
    u := User{"John", "a regular user"}
 
    ut, err := template.New("users").Parse("The user is {{ .Name }} and he is {{ .Bio }}.")
     
    if err != nil {
        panic(err)
    }
 
    err = ut.Execute(os.Stdout, u)
 
    if err != nil {
        panic(err)
    }
}
```

其输出如图所示：  
![](https://cdn.golangdocs.com/wp-content/uploads/2020/01/string-template-parse.png)

### 3. 解析HTML模板

> hello.html   

```html
<h1>Go templates</h1>
<p>The user is {{ .Name }}</p>
<h2>Skills:</h2>
{{ range .Skills }} 
    <p>{{ . }}</p>
{{ end }}
```

> main.go

```go
package main
 
import (
    "os"
    "html/template"
)
 
func main() {
    t, err := template.ParseFiles("templates/hello.gohtml")
 
    if err != nil {
        panic(err)
    }
 
    data := struct {
        Name string
        Skills []string
    }{
        Name: "John Doe",
        Skills: []string{
            "C++",
            "Java",
            "Python",
        },
    }
 
    err = t.Execute(os.Stdout, data)
 
    if err != nil {
        panic(err)
    }
}
```


则结果：  
![](https://cdn.golangdocs.com/wp-content/uploads/2020/01/html-template-parsing-1.png)

## Go中的模板验证

为了验证模板是否有效，我们使用template.Must()函数。它有助于在解析过程中验证模板。因为模板通常在编译时就测试好了,如果模板解析失败将是一个致命的错误template.Must
辅助函数可以简化这个致命错误的处理:它接受一个模板和一个error类型的参数,检测error是否为nil(如果不是nil则发出panic异常),然后返回传入的模板
```go
var report = template.Must(template.New("issuelist").
Funcs(template.FuncMap{"daysAgo": daysAgo}).
Parse(templ))
func main() {
    result, err := github.SearchIssues(os.Args[1:])
    if err != nil {
        log.Fatal(err)
}
if err := report.Execute(os.Stdout, result); err != nil {
        log.Fatal(err)
    }
}
```
