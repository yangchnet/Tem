---
title: "Flag包的基本用法"
date: 2021-01-22T17:27:21+08:00
draft: false
---
# Flag包的基本用法

> flag包用于处理golang命令行程序中的参数

## 1. 使用flag包的基本流程

使用flag包涉及三个步骤：  
1. 定义变量以捕获标志值
2. 定义Go应用程序将使用的标志
3. 在执行时解析提供给应用程序的标志。 

flag软件包中的大多数功能都与定义标志并将其绑定到定义的变量有关。解析阶段由Parse()函数处理。

### 一个例子

创建一个程序，该程序定义一个布尔标志，该标志会更改将打印到标准输出的消息。如果-color提供了一个标志，程序将以蓝色打印一条消息。如果未提供标志，则消息将被打印为没有任何颜色。

```go
// boolean.go
import (
    "flag"
    "fmt"
)

type Color string   // 定义变量以捕获标志值

const (
    ColorBlack  Color = "\u001b[30m"
    ColorRed          = "\u001b[31m"
    ColorGreen        = "\u001b[32m"
    ColorYellow       = "\u001b[33m"
    ColorBlue         = "\u001b[34m"
    ColorReset        = "\u001b[0m"
)

func colorize(color Color, message string) {
    fmt.Println(string(color), message, string(ColorReset))
}

func main() {
    useColor := flag.Bool("color", false, "display colorized output")  // 定义Go应用程序将使用的标志
    flag.Parse()  // 解析标志

    if *useColor {
        colorize(ColorBlue, "Hello, DigitalOcean!")
        return
    }
    fmt.Println("Hello, DigitalOcean!")
}
```

在```main```函数中，使用```flag.Bool```定义了一个布尔型的标志```color```，其默认值为第二个参数```false```，在未提供```color```参数时将默认使用此值。最后一个参数是关于此标志用法的说明文档。

```flag.Bool```返回值是一个```bool```指针类型，```flag.Parse```使用此```bool```指针根据用户传递的标志来设置变量。然后我们就可以通过引用这个指针来检查这个变量，通过变量的值控制程序的行为。

## 2. 处理位置参数

通常，命令带有一定的参数，并且这些参数是命令工作的焦点所在，例如：```python main.py```,这里的```main.py```就是一个位置参数，是```python```这个命令的第一个参数。

假如我们有个命令```head```，它打印一个文件的开始几行，用法为：```head {inputfile}```。  
通常，```Parse()```函数解析一个命令直到其检查到没有标志参数的存在。```flag```包通过```Args```和```Arg```函数来解决位置参数的问题。

```go
// head.go
import (
    "bufio"
    "flag"
    "fmt"
    "io"
    "os"
)

func main() {
    var count int
    flag.IntVar(&count, "n", 5, "number of lines to read from the file")
    flag.Parse()

    var in io.Reader
    if filename := flag.Arg(0); filename != "" {
        f, err := os.Open(filename)
        if err != nil {
            fmt.Println("error opening file: err:", err)
            os.Exit(1)
        }
        defer f.Close()

        in = f
    } else {
        in = os.Stdin
    }

    buf := bufio.NewScanner(in)

    for i := 0; i < count; i++ {
        if !buf.Scan() {
            break
        }
        fmt.Println(buf.Text())
    }

    if err := buf.Err(); err != nil {
        fmt.Fprintln(os.Stderr, "error reading: err:", err)
    }
}
```

首先，我们定义了一个```count```变量来标记程序需要读取文件的前几行。通过```flag.IntVar```定义了```-n```参数。这个函数允许我们传递一个指针作为标记而不是像没有```Var```后缀的函数那样返回一个指针。```flag.IntVar```和```flag.Int```之间除了这一点不同外，其余参数的意义都相同。在使用```flag.Parse```解析后，接下来是根据参数来控制程序逻辑。   

在```if```部分，使用```flag.Arg```来访问在所有标志参数之后的第一个位置参数。
