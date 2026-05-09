# 2\. Go 语言概述

编程语言层出不穷。有些高度专精，另一些则相当通用，而第三类语言旨在填补广泛但某种程度上细分的领域。Go 语言诞生于 2007 年，并于 2009 年公开发布。它被设计为一种系统编程语言，用于增强（或替代）C++ 以及其他静态编译语言，以构建生产环境下的网络和多处理系统。

Go 加入了一系列现代语言的行列，包括 Rust、Swift、Julia 等。Go 的独特之处在于其简洁的语法、基于“结构”类型化对多个程序单元进行快速编译的能力，当然，还有从 C、C++ 和 Java 等大规模程序开发中汲取的经验教训。

在 2022 年第一季度的语言流行度排行榜中，例如 TIOBE 指数（参见 [`http://www.tiobe.com/tiobe-index/`](http://www.tiobe.com/tiobe-index/)），将 Go 排在第 13 位。PYPL 指数（参见 [`http://pypl.github.io/PYPL.html`](http://pypl.github.io/PYPL.html)）也将其列为第 13 位。与之并列的是 Java、Python、C、C++、JavaScript 等已有 20 多年历史的语言。

本书假设你是一位经验丰富的程序员，对 Go 语言有一定程度或深入的了解。这些知识可能来源于入门书籍，例如 Caleb Doxsey 的 *Introducing Go*（O'Reilly 出版）或 Karl Seguin 的 *The Little Go Book*，或者通过阅读更正式的文档，例如 [`https://go.dev/ref/spec`](https://go.dev/ref/spec) 上的 Go 语言规范。

如果你是一位经验丰富的程序员，可以跳过本章。如果不是，本章将指出本书中使用的 Go 相关要点，但你仍需从其他途径获取必要的基础知识。Go 网站 [`https://go.dev`](https://go.dev) 上有多个教程，例如：

*   入门指南 – [`https://go.dev/learn/`](https://go.dev/learn/)
*   Go 编程语言教程 – [`https://go.dev/doc/tutorial/getting-started`](https://go.dev/doc/tutorial/getting-started)
*   在线互动教程 – [`https://go.dev/tour/list`](https://go.dev/tour/list)
*   Effective Go – [`https://go.dev/doc/effective_go`](https://go.dev/doc/effective_go)

安装 Go 的最佳方式是从 Go 编程语言官网进行。本书中的示例将在 Go 1.18 版本下运行。本书第一版使用的是 1.8 版本。核心语言和库基本保持不变，但包管理和工具链在持续改进。本书的主要目标是在你的程序中实现网络概念，而非追求“完美”的 Go 代码。我们将力求与时俱进，确保你不仅具备创建代码的能力，也能审查他人的代码。

实际上，你无需安装 Go 即可测试程序。Go 有一个可从主页面访问的“Playground”，可用于运行代码（[`https://go.dev/play/`](https://go.dev/play/)）。还有一些 REPL（读取-求值-打印循环）环境，但这些是第三方提供的。不过，你通常无法运行与网络相关的代码。这是出于安全考虑；Playground 限制了你的操作范围。但它对于学习非网络代码以及分享代码仍然很有价值。

本书主要使用 Go 标准库（[`https://pkg.go.dev/std`](https://pkg.go.dev/std)）中的库和包。Go 团队还构建了另一组包作为“子仓库”，但这些包通常得不到与标准库同等级别的支持。本书会偶尔使用它们。这些包需要使用 `go get` 命令进行安装。它们的包名中包含“x”，例如 `golang.org/x/net/ipv4`。

在本修订版中，我们将扩展 Go 语言被广泛使用的相关网络生态系统；示例包括常见的 Go 网络中间件和微服务工具（例如 gRPC）。

## 类型

Go 预定义了布尔型、数值型和字符串型类型。数值类型包括 `uint32`、`int32`、`float32` 以及其他不同大小的数值类型，还有字节（`uint8`）和符文（rune）。由于国际化问题在分布式程序中可能很重要，因此在第 6 章中会详细论述符文（`int32` 的别名）和字符串。

还有一些更复杂的类型，将在下面讨论。



### 切片与数组

`数组`是单一类型元素的序列。`切片`是底层数组的片段。在 Go 语言中，处理`切片`通常更为便捷。`切片`允许开发者创建数组的多个视图，理论上可以节省内存。

数组可以静态创建：

```
var x [128]int
```

或以指针形式动态创建：

```
xp := new([128]int)
```

切片可以与其底层数组一同创建：

```
x := make([]int, 50, 100)
```

或者

```
x := new([100]int)[0:50]
```

最后这两种方式的类型都是 `[]int`（如 `reflect.TypeOf(x)` 所示），容量为 100，长度为 50。

数组或切片的元素通过其索引访问：

```
x[1]
```

索引范围是从 0 到 `len(x)-1`。

可以通过使用数组或切片的起始索引（包含）和结束索引（不包含）来获取数组或切片的切片：

-   引用底层数组
-   切片视图在数组中的长度
-   控制我们能看到的底层数组更多部分的能力字段

```
a := [5]int{-1, -2, -3, -4, -5}
s := a[1:4]  // s 现在是 [-2, -3, -4]
切片类似于结构体对象，其中包含三个关键信息片段。
```

在前面的示例中，`s` 是一个切片，而 `a` 是一个数组。数组具有固定的类型和大小，这些方面不能改变。因此 `a` 的类型是 `[5]int`，而 `s` 的类型是 `[]int`。当我们看到切片 `s` 的类型时，我们看不到它的长度。要获取长度或容量，我们有以下内置函数。

```
$ go doc builtin.len
package builtin // import "builtin"
func len(v Type) int
内置函数 len 返回 v 的长度，根据其类型：
数组：v 中元素的数量。
指向数组的指针：*v 中元素的数量（即使 v 为 nil）。
切片或映射：v 中元素的数量；如果 v 为 nil，则 len(v) 为 0。
字符串：v 中的字节数。
通道：通道缓冲区中排队（未读）的元素数量；
如果 v 为 nil，则 len(v) 为 0。
对于某些参数，例如字符串字面量或简单的数组表达式，
结果可能是一个常量。详情请参阅 Go 语言规范中的“长度
和容量”部分。
$ go doc builtin.cap
package builtin // import "builtin"
func cap(v Type) int
内置函数 cap 返回 v 的容量，根据其类型：
数组：v 中元素的数量（与 len(v) 相同）。
指向数组的指针：*v 中元素的数量（与 len(v) 相同）。
切片：当重新切片时，切片可以达到的最大长度；
如果 v 为 nil，则 cap(v) 为 0。
通道：通道缓冲区的容量，以元素为单位；
如果 v 为 nil，则 cap(v) 为 0。
对于某些参数，例如简单的数组表达式，结果可能是一个
常量。详情请参阅 Go 语言规范中的“长度和容量”部分
以获取详细信息。
```

我们现在可以查看每个的长度和容量。还提到了其他类型，但我们这里专注于数组和切片。

```
fmt.Println(len(a), cap(a)) // 5 5
fmt.Println(len(s), cap(s)) // 3 4
```

切片的一个目标是我们可以对同一数组数据拥有许多不同的视图。例如，`a[2:]` 创建一个长度和容量均为 3 的切片。当长度和容量相等时，这意味着在不修改底层数据结构的情况下，切片无法增长。例如，如果我们对切片执行 `append()` 操作，它会插入到底层数组的相对位置。如果 `cap() – len() == 0`，那么底层数组的大小就需要增加。在这种情况下，这种增加确实会发生，但可能是出乎意料的。关于此过程以及切片如何共享数组（如果追加操作数据量过大则可能不共享）的更多细节，可以在此处找到：[`https://go.dev/blog/slices-intro`](https://go.dev/blog/slices-intro)。更多关于切片/数组关系（包括内存细节）的详细信息可以在这里找到：[`https://go.dev/blog/slices-intro`](https://go.dev/blog/slices-intro)。

### 映射

*映射*是一种类型元素的无序集合，由另一种类型的键进行索引。本书中我们不常使用映射，但有一个地方是在第 [10](https://go.dev/blog/slices-intro) 章中，其中 HTTP 请求字段的值可以通过使用字段名作为键，通过映射来访问。

```
myval := 10
x := map[string]int{"mykey": myval}
```

上述赋值有一些值得讨论的点。

-   `map[string]int` 声明了我们映射的类型。
-   `{"mykey": myval}` 使用字面量语法来初始化我们指定类型的映射。
-   最终赋值给 `x`。

根据前一节，我们可以通过内置函数 `len` 来检索键的数量。

```
len(x) // 1
```

自 1.12 版本起，可以按键排序的顺序遍历映射（例如，打印出键和值）。

删除映射的键可以通过另一个名为 `delete` 的内置函数来完成。

```
delete(x, "mykey")
```

长度为 0 的映射与 nil 映射不同。

### 指针

指针的行为与其他编程语言的指针类似。`*` 操作符解引用指针，而 `&` 操作符获取变量的地址。Go 简化了指针的使用，使得大多数时候你无需担心它们。在本书中，我们最多只是检查指针值是否为 `nil`，这通常表示一个错误，或者相反地，检查可能出现的错误值是否不为 `nil`，如下一节所述。

```
// 返回指向存有 int 零值地址的新指针
x := new(int)
// 在指针指向的地址位置设置整数值
*x = 10
// 指向、指针的地址、我们指向位置的值
fmt.Println(x, &x, *x)
// 0xc000094010 0xc000096018 10
//每个变量都有一个地址，即其值
y := 12
fmt.Println(&y, y)
// 0xc000094018 12
```

需要注意的一点是，在声明函数参数时，指针和数组不是一回事。如果你打算传递一个数组，通常你会想要传递其地址；这是因为 Go 是按值传递的。

### 函数

函数使用 Go 语言特有的符号定义。为什么没有使用熟悉的 C 语法（或其他任何语法）在 Go 的声明语法博客（参见 [`https://go.dev/blog/declaration-syntax`](https://go.dev/blog/declaration-syntax)）中进行了解释。我们将语法的详细解释留给教科书。

每个 Go 程序都必须有一个声明如下的 `main` 函数：

```
func main() { ... }
```

我们将经常使用一个定义如下的函数 `checkError`：

```
func checkError(err error) {
    if err != nil {
        fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
        os.Exit(1)
    }
}
```

它接受一个参数并且没有返回值。它以一个小写字母开头，因此它对声明它的包来说是本地的。

返回值的函数通常会返回一个错误状态以及一个实质性值，如下面这个来自第 [3](https://go.dev/blog/slices-intro) 章的函数：

```
func readFully(conn net.Conn) ([]byte, error) { ... }
```

它接受 `net.Conn` 作为参数，并返回一个字节数组和一个错误状态（如果没有发生错误则为 `nil`）。

在本书中，不会使用比这更复杂的定义。

### 结构体

结构体与其他编程语言中的类似。在第 [4](https://go.dev/blog/slices-intro) 章中，我们考虑数据的序列化，并使用以下结构体的示例：

```
type Name struct {
    Family   string
    Personal string
}
type Email struct {
    Kind    string
    Address string
}
type Person struct {
    Name  Name
    Email []Email
}
```

一个复合结构体可以声明如下：

```
person := Person{
    Name: Name{Family: "Newmarch", Personal: "Jan"},
    Email: []Email{Email{Kind: "home", Address: "jan@newmarch.name"},
        Email{Kind: "work", Address: "j.newmarch@boxhill.edu.au"}}}
```



## 方法

Go 语言并不像 Java 这类语言拥有类（classes）。然而，*类型*可以关联*方法*，其行为类似于更标准的面向对象语言中的方法。

我们将大量使用为各种网络类型定义的方法。从下一章的第一个程序开始就会用到。例如，`IPMask` 类型被定义为一个字节数组：

```
type IPMask []byte
```

该类型定义了许多函数，例如：

```
func (m IPMask) Size() (ones, bits int)
```

一个 `IPMask` 类型的变量可以像下面这样应用 `Size()` 方法：

```
var m IPMask
...
ones, bits := m.Size()
```

学习如何使用网络相关类型的方法，是本书的主要目的。

本书中我们不会过多地自定义方法，这是因为为了演示 Go 的库，我们不需要太多自己的复杂类型。一个典型的用法是美化打印之前定义的类型，比如 `Person` 类型：

```
func (p Person) String() string {
    s := p.Name.Personal + " " + p.Name.Family
    for _, v := range p.Email {
        s += "\n" + v.Kind + ": " + v.Address
    }
    return s
}
```

更具体地说，触发上述方法的方式如下：

```
p := Person{}
details := p.String() // String 是一个方法
len(details) // len 是一个函数
```

在第 10 章中有更广泛的应用，其中使用了多种类型及其上的方法。这是因为当构建更真实的系统时，我们*确实*需要自己的类型。

Go 还支持一等函数和高阶函数。一等函数意味着我们可以将函数（或方法）的引用存储在变量中。高阶函数意味着函数（或方法）可以接受或返回一个函数。以下是一等函数的示例：

```
m := p.String
m() // 触发 p.String() 方法
```

以下是高阶函数的示例：

```
package main
import (
    "fmt"
)
func f1(f func(string) int, data string) {
    fmt.Println(f(data))
}
func main() {
    f1(func(s string) int { return len(s) }, "testing")
}
$ go run prog.go
```

## 多线程

Go 语言提供了一种简单的机制，通过 `go` 命令启动额外的线程。本书中，我们只需要用到这些。这里不需要执行诸如同步多个线程之类的复杂任务。

## 包

Go 程序由链接的包构建而成。任何代码块所使用的包，都必须通过代码文件顶部的 `import` 语句导入。我们自己的程序被声明为属于 `main` 包。

除了第 10 章外，本书中几乎所有的程序都在 `main` 包中。

大多数包都是从标准库导入的。也有一些是从子仓库导入的，例如 `golang.org/x/net/ipv4`。

结构体字段的*可见性*由字段名的首字母大小写控制。如果首字母大写，则在声明它的包外部可见；如果小写，则不可见。在前面的例子中，所有结构体的所有字段都是可见的。

## 模块

Go 模块追踪包及其版本。模块允许追踪直接和间接依赖关系。

虽然模块是在 Go 中分组和共享代码（包）的正确方式，但用户仍然经常仅使用包的方式。

当单个包无法满足需求时，我们将使用模块。即使使用了模块，我们的意图也是让示例专注于网络代码。在这个例子中，生成了一个名为 `go.mod` 的文件，Go 工具使用它来管理我们的依赖关系：

```
//示例
$ mkdir myapp; cd myapp
$ go mod init example.com
$ // 创建程序
$ go mod tidy
$ go run prog.go
```

你可以在这里了解更多关于模块的信息：[`https://go.dev/blog/using-go-modules`](https://go.dev/blog/using-go-modules)。

在 Go 1.18 中，包含了一个帮助管理跨模块本地开发的新特性；该特性称为工作区。本书中没有用到这个特性，但你可以在这里了解更多信息：[`https://go.dev/doc/tutorial/workspaces`](https://go.dev/doc/tutorial/workspaces)。

## 类型转换

在本书中，我们首先需要关注的类型转换是字符串与字节数组之间的相互转换。要将字符串转换为字节数组，可以这样做：

```
var b []byte
b = []byte("string")
```

要将整个数组/切片转换为字符串，请使用：

```
var s string
s = string(b[:])
```

我们需要重点关注的第二种类型转换称为函数适配器，其最常见的形式如下：

```
...
http.Handle("/", http.HandlerFunc(func (w ResponseWriter, r *Request) {fmt.Fprintf(w,"hi")}))
...
$ go doc --src http.HandlerFunc
package http // import "net/http"
// HandlerFunc 类型是一个适配器，允许将普通函数用作 HTTP 处理程序。
// 如果 f 是一个具有适当签名的函数，则 HandlerFunc(f) 是一个
// 会调用 f 的 Handler。
type HandlerFunc func(ResponseWriter, *Request)
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request)
```

通过使用这种类型，对 `HandlerFunc.ServeHTTP` 的调用将触发我们传入的函数。

## 语句

一个函数或方法由一组语句组成。这些语句包括赋值、`if` 和 `switch` 语句、`for` 和 `while` 循环以及其他几种。

除了语法之外，这些语句与其他编程语言中的含义基本相同。几乎所有类型的语句都将在后面的章节中使用。

## GOPATH

有两种组织项目工作区的方式：将所有项目放在一个共享工作区，或者为每个项目设置独立的工作区。

Go 工具通过环境变量 `GOPATH` 支持这两种方式。该变量可以设置为一个目录列表（在 Linux/UNIX 中是用 `:` 分隔的列表，在 Windows 中是用 `;` 分隔的列表，在 Plan9 中是一个列表）。如果未设置，则默认为用户主目录下的 `go` 目录。

对于 `GOPATH` 中的每个目录，都会有三个子目录——`src`、`pkg` 和 `bin`。`src` 目录通常每个包名包含一个目录，其下是该包的源文件。例如，在第 10 章中，我们有一个完整的 web 服务器，它使用了我们定义的包：`dictionary` 和 `flashcards`。`src/flashcards` 目录包含文件 `FlashCards.go`。

即使在使用模块时，`GOPATH` 也用作下载依赖项的中心位置。将 `GOPATH` 设置为其他位置是有原因的，但这些原因大多在模块和工作区出现之前。

## 运行 Go 程序

一个 Go 程序必须有一个定义 `main` 包的文件。本书中的大多数程序都定义在单个文件中，例如第 3 章中的程序 `IP.go`。运行它的最简单方法是从包含该文件的目录执行：

```
go run IP.go
```

或者，你可以先构建一个可执行文件，然后运行它：

```
go build IP.go
./IP 
```

## 标准库

Go 拥有广泛的标准库集合。虽然不如 C、Java 或 C++ 那么大，但那些语言已经存在了相当*长*的时间。Go 包的文档在 [`https://pkg.go.dev/std`](https://pkg.go.dev/std)。我们将在本书中广泛使用这些库，特别是 `net`、`crypto` 和 `encoding` 包。

此外，在同一个页面上还有一个子仓库包组。这些包不太稳定，但有时包含有用的包，我们会偶尔使用。

除了这些，还有大量用户贡献的包。本书正文内容涉及的是原理，因此不会使用这些包，但在实践中，你可能会发现它们中的许多非常有用。一些会在最后一章中讨论。



## 错误值

上一章讨论过，分布式编程与本地编程的一个主要区别在于执行过程中出现错误的概率显著增加。本地函数调用可能因简单的编程错误（如除以零）而失败；也可能出现更微妙的错误（如内存不足），但它们的发生概率通常是可预见的。

另一方面，几乎所有利用网络的函数都可能因应用程序无法控制的原因而失败。因此，网络程序布满了错误检查。这虽然繁琐，但却是必要的。就像操作系统内核代码总是进行错误检查一样——错误需要被管理。

在本书中，对于客户端，我们通常会在出现错误时打印相应信息并退出程序；对于服务器，我们会尝试通过断开有问题的连接并继续运行来恢复。

像 C 语言这样的语言通常通过返回“非法”值（如负整数和空指针）或引发信号来指示错误。像 Java 这样的语言会抛出异常，这可能导致代码混乱且通常较慢。标准的 Go 函数则通过函数调用返回的额外参数来提供错误信息。

例如，在下一章中，我们会讨论`net`包中的函数：

```go
func ResolveIPAddr(net, addr string) (*IPAddr, error)
```

处理该函数的典型代码如下：

```go
addr, err := net.ResolveIPAddr("ip", name)
if err != nil {
    ...
}
```

## 结论

本书假定读者已具备 Go 编程语言的知识。本章仅重点介绍后续章节所需的部分内容。

