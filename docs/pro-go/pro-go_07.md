# 17. 格式化与扫描字符串

在本章中，我介绍用于格式化和扫描字符串的标准库功能。格式化是从一个或多个数据值组合成新字符串的过程，而扫描是从字符串中解析出值的过程。表 17-1 将格式化与扫描字符串放在上下文中进行说明。

表 17-1 - 格式化与扫描字符串的背景说明

| 问题 | 答案 |
| --- | --- |
| 它们是什么？ | 格式化是将值组合成字符串的过程。扫描是从字符串中解析出其所含值的过程。 |
| 它们有什么用？ | 格式化字符串是一种常见需求，用于生成从日志记录和调试到向用户展示信息等各种目的的字符串。扫描对于从字符串（如 HTTP 请求或用户输入）中提取数据非常有用。 |
| 如何使用它们？ | 这两个功能集都通过 `fmt` 包中定义的函数提供。 |
| 是否存在任何陷阱或限制？ | 用于格式化字符串的模板可能难以阅读，并且没有内置函数可以自动在生成的格式化字符串末尾追加换行符。 |
| 是否有其他替代方案？ | 可以使用第 23 章中描述的模板功能来生成大量的文本和 HTML 内容。 |

表 17-2 总结了本章内容。

表 17-2 - 本章总结

| 问题 | 解决方案 | 代码清单 |
| --- | --- | --- |
| 将数据值组合成字符串 | 使用 `fmt` 包提供的基本格式化函数 | 5, 6 |
| 指定字符串的结构 | 使用使用格式化模板的 `fmt` 函数，并使用格式化动词 | 7–9, 11–18 |
| 更改自定义数据类型的表示方式 | 实现 `Stringer` 接口 | 10 |
| 解析字符串以获取其中包含的数据值 | 使用 `fmt` 包提供的扫描函数 | 19–22 |

## 本章准备工作

为准备本章内容，请打开一个新的命令提示符，导航到一个方便的位置，并创建一个名为 `usingstrings` 的目录。运行代码清单 17-1 中所示的命令以创建一个模块文件。

提示

你可以从 `https://github.com/apress/pro-go` 下载本章以及本书所有其他章节的示例项目。如果在运行示例时遇到问题，请参阅第 2 章以获取帮助。

```go
go mod init usingstrings
```

代码清单 17-1 - 初始化模块

在 `usingstrings` 文件夹中添加一个名为 `product.go` 的文件，内容如代码清单 17-2 所示。

```go
package main
type Product struct {
Name, Category string
Price float64
}
var Kayak = Product {
Name: "Kayak",
Category: "Watersports",
Price: 275,
}
var Products = []Product {
{ "Kayak", "Watersports", 279 },
{ "Lifejacket", "Watersports", 49.95 },
{ "Soccer Ball", "Soccer", 19.50 },
{ "Corner Flags", "Soccer", 34.95 },
{ "Stadium", "Soccer", 79500 },
{ "Thinking Cap", "Chess", 16 },
{ "Unsteady Chair", "Chess", 75 },
{ "Bling-Bling King", "Chess", 1200 },
}
```

代码清单 17-2 - `usingstrings` 文件夹中 `product.go` 文件的内容

在 `usingstrings` 文件夹中添加一个名为 `main.go` 的文件，内容如代码清单 17-3 所示。

```go
package main
import "fmt"
func main() {
fmt.Println("Product:", Kayak.Name, "Price:", Kayak.Price)
}
```

代码清单 17-3 - `usingstrings` 文件夹中 `main.go` 文件的内容

使用命令提示符在 `usingstrings` 文件夹中运行代码清单 17-4 中所示的命令。

```go
go run .
```

代码清单 17-4 - 运行示例项目

代码将被编译并执行，产生以下输出：

```
Product: Kayak Price: 275
```

## 编写字符串

`fmt` 包提供了组合和编写字符串的函数。基本函数在表 17-3 中进行了描述。其中一些函数使用写入器，这是 Go 对输入/输出支持的一部分，将在第 20 章中描述。

表 17-3 - 用于组合和编写字符串的基本 `fmt` 函数

| 名称 | 描述 |
| --- | --- |
| `Print(...vals)` | 此函数接受可变数量的参数，并将其值写入标准输出。在非字符串值之间会添加空格。 |
| `Println(...vals)` | 此函数接受可变数量的参数，并将其值写入标准输出，值之间用空格分隔，末尾追加换行符。 |
| `Fprint(writer, ...vals)` | 此函数将可变数量的参数写入指定的写入器（我将在第 20 章中描述）。在非字符串值之间会添加空格。 |
| `Fprintln(writer, ...vals)` | 此函数将可变数量的参数写入指定的写入器（我将在第 20 章中描述），末尾追加换行符。所有值之间都会添加空格。 |

注意

Go 标准库包含一个模板包（在第 23 章中描述），可用于生成大量的文本和 HTML 内容。

表 17-3 中描述的函数会在其生成的字符串中的值之间添加空格，但添加方式不一致。`Println` 和 `Fprintln` 函数在所有值之间都添加空格，但 `Print` 和 `Fprint` 函数仅在不属于字符串的值之间添加空格。这意味着表 17-3 中的函数对之间的差异不仅仅是多了一个换行符，如代码清单 17-5 所示。

```go
package main
import "fmt"
func main() {
fmt.Println("Product:", Kayak.Name, "Price:", Kayak.Price)
fmt.Print("Product:", Kayak.Name, "Price:", Kayak.Price, "\n")
}
```

代码清单 17-5 - 在 `usingstrings` 文件夹的 `main.go` 文件中编写字符串

在许多编程语言中，代码清单 17-5 中的语句产生的字符串不会有任何区别，因为我向 `Print` 函数传递的参数中添加了一个换行符。但是，由于 `Print` 函数仅在成对的非字符串值之间添加空格，因此结果是不同的。编译并执行代码，你将看到以下输出：

```
Product: Kayak Price: 275
Product:KayakPrice:275
```



### 格式化字符串

在前面的章节中，我一直使用 `fmt.Println` 函数来输出内容。之所以用这个函数，是因为它很简单，但它无法控制输出的格式，这意味着它适用于简单的调试，但不适合生成复杂的字符串或格式化要展示给用户的值。`fmt` 包中其他提供格式化控制功能的函数如清单 17-6 所示。

```
package main
import "fmt"
func main() {
fmt.Printf("Product: %v, Price: $%4.2f", Kayak.Name, Kayak.Price)
}
```

`Printf` 函数接受一个模板字符串和一系列值。模板中会扫描*动词*，动词由百分号（`%` 字符）后跟格式说明符表示。清单 17-6 的模板中有两个动词：

```
...
fmt.Printf("Product: %v, Price: $%4.2f", Kayak.Name, Kayak.Price)
...
```

第一个动词是 `%v`，它指定类型的默认表示形式。例如，对于 `string` 类型的值，`%v` 只是将字符串包含在输出中。`%4.2f` 动词指定浮点值的格式，小数点前有 4 位数字，小数点后有 2 位数字。模板动词的值取自其余参数，并按照指定的顺序使用。就本例而言，这意味着 `%v` 动词用于格式化 `Product.Name` 值，`%4.2f` 动词用于格式化 `Product.Price` 值。这些值被格式化后，插入到模板字符串中，并输出到控制台，你可以通过编译和执行代码看到结果：

```
Product: Kayak, Price: $275.00
```

表 17-4 描述了 `fmt` 包提供的用于格式化字符串的函数。我将在“理解格式化动词”一节中介绍格式化动词。

**表 17-4.** `fmt` 函数：用于格式化字符串

| 名称 | 描述 |
| --- | --- |
| `Sprintf(t, ...vals)` | 此函数返回一个字符串，该字符串通过处理模板 `t` 创建。其余参数用作模板动词的值。 |
| `Printf(t, ...vals)` | 此函数通过处理模板 `t` 创建一个字符串。其余参数用作模板动词的值。该字符串被写入标准输出。 |
| `Fprintf(writer, t, ...vals)` | 此函数通过处理模板 `t` 创建一个字符串。其余参数用作模板动词的值。该字符串被写入一个 `Writer`，这将在第 20 章中介绍。 |
| `Errorf(t, ...values)` | 此函数通过处理模板 `t` 创建一个 `error`。其余参数用作模板动词的值。结果是一个 `error` 值，其 `Error` 方法返回格式化后的字符串。 |

在清单 17-7 中，我定义了一个函数，它使用 `Sprintf` 格式化字符串结果，并使用 `Errorf` 创建一个错误。

```
package main
import "fmt"
func getProductName(index int) (name string, err error) {
if (len(Products) > index) {
name = fmt.Sprintf("Name of product: %v", Products[index].Name)
} else {
err = fmt.Errorf("Error for index %v", index)
}
return
}
func main() {
name, _ := getProductName(1)
fmt.Println(name)
_, err := getProductName(10)
fmt.Println(err.Error())
}
```

本例中的两个格式化字符串都使用了 `%v` 值，它将以默认形式输出值。编译并执行该项目，你将看到如下结果和一个错误：

```
Name of product: Lifejacket
Error for index 10
```

## 理解格式化动词

表 17-4 中描述的函数在其模板中支持多种格式化动词。在接下来的章节中，我将介绍最常用的动词。我先介绍那些可用于任何数据类型的动词，然后再介绍更具体的动词。

### 使用通用格式化动词

通用动词可用于显示任何值，如表 17-5 所述。

**表 17-5.** 适用于任何值的格式化动词

| 动词 | 描述 |
| --- | --- |
| `%v` | 此动词显示值的默认格式。在动词后加上加号（`%+v`）会在输出结构体值时包含字段名。 |
| `%#v` | 此动词以一种可用于在 Go 代码文件中重新创建该值的格式来显示值。 |
| `%T` | 此动词显示值的 Go 类型。 |

在清单 17-8 中，我定义了一个自定义结构体类型，并使用表中所示的动词来格式化该类型的一个值。

```
package main
import "fmt"
func Printfln(template string, values ...interface{}) {
fmt.Printf(template + "\n", values...)
}
func main() {
Printfln("Value: %v", Kayak)
Printfln("Go syntax: %#v", Kayak)
Printfln("Type: %T", Kayak)
}
```

`Printf` 函数不会像 `Println` 函数那样在其输出后追加换行符，因此我定义了一个 `Printfln` 函数，该函数在调用 `Printf` 函数之前会在模板后追加换行符。`main` 函数中的语句定义了包含表 17-5 中动词的简单字符串模板。编译并执行代码，你将看到以下输出：

```
Value: {Kayak Watersports 275}
Go syntax: main.Product{Name:"Kayak", Category:"Watersports", Price:275}
Type: main.Product
```



### 控制结构体格式化

Go 为所有数据类型都提供了一种默认格式，由 `%v` 动词使用。对于结构体，默认值会在花括号内列出字段值。可以通过加号修改默认动词，以便在输出中包含字段名称，如代码清单 17-9 所示。

```
package main
import "fmt"
func Printfln(template string, values ...interface{}) {
fmt.Printf(template + "\n", values...)
}
func main() {
Printfln("Value: %v", Kayak)
Printfln("Value with fields: %+v", Kayak)
}
```

编译并执行该项目，你将看到同一个 `Product` 值分别以包含和不包含字段名称的格式输出：

```
Value: {Kayak Watersports 275}
Value with fields: {Name:Kayak Category:Watersports Price:275}
```

`fmt` 包通过一个名为 `Stringer` 的接口支持自定义结构体格式化，该接口定义如下：

```
type Stringer interface {
String() string
}
```

`Stringer` 接口指定的 `String` 方法将用于获取任何定义了该方法的类型的字符串表示形式，从而可以指定自定义格式，如代码清单 17-10 所示。

```
package main
import "fmt"
type Product struct {
Name, Category string
Price float64
}
// ...为简洁起见，变量已省略...
func (p Product) String() string {
return fmt.Sprintf("Product: %v, Price: $%4.2f", p.Name, p.Price)
}
```

当需要 `Product` 值的字符串表示形式时，`String` 方法将被自动调用。编译并执行代码，输出将使用自定义格式：

```
Value: Product: Kayak, Price: $275.00
Value with fields: Product: Kayak, Price: $275.00
```

请注意，当修改 `%v` 动词以显示结构体字段时，也会使用自定义格式。

**提示**：如果你定义了一个返回字符串的 `GoString` 方法，那么你的类型将符合 `GoStringer` 接口，该接口允许为 `%#v` 动词提供自定义格式。

### 格式化数组、切片和映射

当数组和切片被表示为字符串时，输出是一组方括号，其中包含各个元素，如下所示：

```
...
[Kayak Lifejacket Paddle]
...
```

请注意，元素之间没有逗号分隔。当映射被表示为字符串时，键值对显示在方括号内，并以 `map` 关键字开头，如下所示：

```
...
map[1:Kayak 2:Lifejacket 3:Paddle]
...
```

`Stringer` 接口可用于更改数组、切片或映射中包含的自定义数据类型的格式。但是，除非你使用类型别名，否则无法更改默认格式，因为方法必须与它们所应用的类型定义在同一个包中。

### 使用整数格式化动词

表 17-6 描述了整数值的格式化动词，无论其大小如何。

**表 17-6**：整数值的格式化动词

| 动词 | 描述 |
| --- | --- |
| `%b` | 此动词将整数值显示为二进制字符串。 |
| `%d` | 此动词将整数值显示为十进制字符串。这是整数值的默认格式，在使用 `%v` 动词时应用。 |
| `%o`, `%O` | 这些动词将整数值显示为八进制字符串。`%O` 动词会添加 `0o` 前缀。 |
| `%x`, `%X` | 这些动词将整数值显示为十六进制字符串。字母 A–F 由 `%x` 动词以小写形式显示，由 `%X` 动词以大写形式显示。 |

代码清单 17-11 将表 17-6 中描述的动词应用于一个整数值。

```
package main
import "fmt"
func Printfln(template string, values ...interface{}) {
fmt.Printf(template + "\n", values...)
}
func main() {
number := 250
Printfln("Binary: %b", number)
Printfln("Decimal: %d", number)
Printfln("Octal: %o, %O", number, number)
Printfln("Hexadecimal: %x, %X", number, number)
}
```

编译并执行该项目，你将收到以下输出：

```
Binary: 11111010
Decimal: 250
Octal: 372, 0o372
Hexadecimal: fa, FA
```


好的，作为高级文档工程师和翻译员，我将遵循您提供的注意事项和示例，将给定的英文文本翻译成中文。


### 使用浮点数格式化动词

表 17-7 描述了用于浮点数值的格式化动词，这些动词可以应用于 `float32` 和 `float64` 两种类型的值。

**表 17-7**  
浮点数值的格式化动词

| 动词 | 描述 |
| --- | --- |
| `%b` | 此动词显示带有指数但无小数位的浮点数值。 |
| `%e, %E` | 这些动词显示带有指数和小数位的浮点数值。`%e` 使用小写指数指示符，而 `%E` 使用大写指示符。 |
| `%f, %F` | 这些动词显示带有小数位但没有指数的浮点数值。`%f` 和 `%F` 动词产生相同的输出。 |
| `%g` | 此动词会根据其显示的值进行调整。对于指数较大的值，使用 `%e` 格式；否则使用 `%f` 格式。这是默认格式，在使用 `%v` 动词时会应用此格式。 |
| `%G` | 此动词会根据其显示的值进行调整。对于指数较大的值，使用 `%E` 格式；否则使用 `%f` 格式。 |
| `%x, %X` | 这些动词以十六进制表示法显示浮点数值，使用小写字母 (`%x`) 或大写字母 (`%X`)。 |

清单 17-12 将表 17-7 中描述的动词应用到一个浮点数值上。

```
package main
import "fmt"
func Printfln(template string, values ...interface{}) {
fmt.Printf(template + "\n", values...)
}
func main() {
number := 279.00
Printfln("Decimalless with exponent: %b", number)
Printfln("Decimal with exponent: %e", number)
Printfln("Decimal without exponent: %f", number)
Printfln("Hexadecimal: %x, %X", number, number)
}
```

编译并执行该项目，您将看到以下输出：

```
Decimalless with exponent: 4908219906392064p-44
Decimal with exponent: 2.790000e+02
Decimal without exponent: 279.000000
Hexadecimal: 0x1.17p+08, 0X1.17P+08
```

浮点数值的格式可以通过修改动词来指定宽度（用于表示值的字符数）和精度（小数点后的位数）来控制，如清单 17-13 所示。

```
package main
import "fmt"
func Printfln(template string, values ...interface{}) {
fmt.Printf(template + "\n", values...)
}
func main() {
number := 279.00
Printfln("Decimal without exponent: >>%8.2f<<", number)
}
```

宽度在百分号后指定，后跟句点，再跟精度，最后是动词的其余部分。在清单 17-13 中，宽度是 8 个字符，精度是两位，当代码被编译并执行时，会产生以下输出：

```
Decimal without exponent: >>  279.00<<
```

我在清单 17-13 中为格式化后的值添加了尖括号，以演示当指定的宽度大于显示值所需的字符数时，会使用空格进行填充。

如果您只关心精度，可以省略宽度，如清单 17-14 所示。

```
package main
import "fmt"
func Printfln(template string, values ...interface{}) {
fmt.Printf(template + "\n", values...)
}
func main() {
number := 279.00
Printfln("Decimal without exponent: >>%.2f<<", number)
}
```

宽度值被省略，但句点仍然是必需的。清单 17-7 中指定的格式在编译并执行时会产生以下输出：

```
Decimal without exponent: >>279.00<<
```

表 17-7 中动词的输出可以使用表 17-8 中描述的修饰符进行更改。

**表 17-8**  
格式化动词修饰符

| 修饰符 | 描述 |
| --- | --- |
| `+` | 此修饰符（加号）始终为数值打印符号，无论是正号还是负号。 |
| `0` | 当宽度大于显示值所需的字符数时，此修饰符使用零（而不是空格）进行填充。 |
| `-` | 此修饰符（减号）将填充添加到数字的右侧，而不是左侧。 |

清单 17-15 应用了这些修饰符来更改整数值的格式。

```
package main
import "fmt"
func Printfln(template string, values ...interface{}) {
fmt.Printf(template + "\n", values...)
}
func main() {
number := 279.00
Printfln("Sign: >>%+.2f>%010.2f>%-8.2f<<", number)
}
```

编译并执行该项目，您将看到这些修饰符对格式化输出的影响：

```
Sign: >>+279.00>0000279.00>279.00  <<
```

### 使用字符串和字符格式化动词

表 17-9 描述了用于字符串和符文（rune）的格式化动词。

**表 17-9**  
字符串和符文的格式化动词

| 动词 | 描述 |
| --- | --- |
| `%s` | 此动词显示一个字符串。这是默认格式，在使用 `%v` 动词时会应用此格式。 |
| `%c` | 此动词显示一个字符。必须小心避免将字符串分割成单个字节，如下表后的文本所述。 |
| `%U` | 此动词以 Unicode 格式显示一个字符，因此输出以 `U+` 开头，后跟十六进制字符代码。 |

字符串很容易格式化，但在格式化单个字符时必须小心。正如我在第 7 章中解释的，某些字符使用多个字节表示，您必须确保不会尝试仅格式化该字符的部分字节。清单 17-16 演示了表 17-9 中描述的动词的使用。

```
package main
import "fmt"
func Printfln(template string, values ...interface{}) {
fmt.Printf(template + "\n", values...)
}
func main() {
name := "Kayak"
Printfln("String: %s", name)
Printfln("Character: %c", []rune(name)[0])
Printfln("Unicode: %U", []rune(name)[0])
}
```

编译并执行该项目，您将看到以下格式化输出：

```
String: Kayak
Character: K
Unicode: U+004B
```

### 使用布尔格式化动词

表 17-10 描述了用于格式化 `bool` 值的动词。这是默认的 `bool` 格式，这意味着 `%v` 动词将使用它。

**表 17-10**  
bool 格式化动词

| 动词 | 描述 |
| --- | --- |
| `%t` | 此动词格式化 `bool` 值，并显示 `true` 或 `false`。 |

清单 17-17 展示了 `bool` 格式化动词的使用。

```
package main
import "fmt"
func Printfln(template string, values ...interface{}) {
fmt.Printf(template + "\n", values...)
}
func main() {
name := "Kayak"
Printfln("Bool: %t", len(name) > 1)
Printfln("Bool: %t", len(name) > 100)
}
```

编译并执行该项目，您将看到格式化输出：

```
Bool: true
Bool: false
```



### 使用指针格式化动词

表 17-11 中描述的动词适用于指针。

**表 17-11：** 指针格式化动词

| 动词 | 描述 |
| --- | --- |
| `%p` | 此动词显示指针存储位置的十六进制表示。 |

清单 17-18 演示了指针动词的使用。

```
package main
import "fmt"
func Printfln(template string, values ...interface{}) {
fmt.Printf(template + "\n", values...)
}
func main() {
name := "Kayak"
Printfln("Pointer: %p", &name)
}
清单 17-18
在 usingstrings 文件夹的 main.go 文件中格式化指针
```

编译并执行代码，您将看到类似如下的输出，但显示的地址可能不同：

```
Pointer: 0xc00004a240
```

## 扫描字符串

`fmt` 包提供了用于扫描字符串的函数，即解析包含空格分隔值的字符串的过程。表 17-12 描述了这些函数，其中一些函数与后续章节中介绍的功能一起使用。

**表 17-12：** 用于扫描字符串的 `fmt` 函数

| 名称 | 描述 |
| --- | --- |
| `Scan(...vals)` | 此函数从标准输入读取文本，并将空格分隔的值存储到指定的参数中。换行符被视为空格，该函数会一直读取，直到为其所有参数接收到值为止。其结果是已读取值的数量以及描述任何问题的 `error`。 |
| `Scanln(...vals)` | 此函数的工作方式与 `Scan` 相同，但在遇到换行符时会停止读取。 |
| `Scanf(template, ...vals)` | 此函数的工作方式与 `Scan` 相同，但使用模板字符串从其收到的输入中选择值。 |
| `Fscan(reader, ...vals)` | 此函数从指定的读取器（详见第 20 章）读取空格分隔的值。换行符被视为空格，该函数返回已读取值的数量以及描述任何问题的错误。 |
| `Fscanln(reader, ...vals)` | 此函数的工作方式与 `Fscan` 相同，但在遇到换行符时会停止读取。 |
| `Fscanf(reader, template, ...vals)` | 此函数的工作方式与 `Fscan` 相同，但使用模板从其收到的输入中选择值。 |
| `Sscan(str, ...vals)` | 此函数扫描指定字符串中由空格分隔的值，这些值被分配给其余参数。其结果是扫描值的数量以及描述任何问题的错误。 |
| `Sscanf(str, template, ...vals)` | 此函数的工作方式与 `Sscan` 相同，但使用模板从字符串中选择值。 |
| `Sscanln(str, template, ...vals)` | 此函数的工作方式与 `Sscanf` 相同，但一旦遇到换行符就会停止扫描字符串。 |

选择使用哪个扫描函数取决于要扫描的字符串来源、如何处理换行符以及是否应使用模板。清单 17-19 展示了 `Scan` 函数的基本用法，这是一个很好的入门示例。

```
package main
import "fmt"
func Printfln(template string, values ...interface{}) {
fmt.Printf(template + "\n", values...)
}
func main() {
var name string
var category string
var price float64
fmt.Print("Enter text to scan: ")
n, err := fmt.Scan(&name, &category, &price)
if (err == nil) {
Printfln("Scanned %v values", n)
Printfln("Name: %v, Category: %v, Price: %.2f", name, category, price)
} else {
Printfln("Error: %v", err.Error())
}
}
清单 17-19
在 usingstrings 文件夹的 main.go 文件中扫描字符串
```

`Scan` 函数从标准输入读取字符串，并扫描其中由空格分隔的值。从字符串解析出的值按其定义的顺序分配给参数。为了让 `Scan` 函数能够赋值，其参数是指针。

在清单 17-19 中，我定义了 `name`、`category` 和 `price` 变量，并将它们用作 `Scan` 函数的参数：

```
...
n, err := fmt.Scan(&name, &category, &price)
...
```

当被调用时，`Scan` 函数将读取一个字符串，提取三个空格分隔的值，并将它们分配给这些变量。编译并执行该项目，系统将提示您输入文本，如下所示：

```
...
Enter text to scan:
...
```

输入 `Kayak Watersports 279`，即单词 `Kayak`，后跟空格，再跟单词 `Watersports`，后跟空格，再跟数字 `279`。按下回车键，字符串将被扫描，产生如下输出：

```
Scanned 3 values
Name: Kayak, Category: Watersports, Price: 279.00
```

`Scan` 函数必须将接收到的子字符串转换为 Go 值，如果字符串无法处理，则会报告错误。再次运行代码，但输入 `Kayak Watersports Zero`，您将收到以下错误：

```
Error: strconv.ParseFloat: parsing "": invalid syntax
```

字符串 `Zero` 无法转换为 Go 的 `float64` 值，而 `float64` 正是 `Price` 参数的类型。

### 扫描到切片

如果需要扫描一系列相同类型的值，很自然的做法是扫描到切片或数组中，如下所示：

```
...
vals := make([]string, 3)
fmt.Print("Enter text to scan: ")
fmt.Scan(vals...)
Printfln("Name: %v", vals)
...
```

这段代码无法编译，因为字符串切片无法正确分解以用于可变参数。还需要一个额外的步骤，如下所示：

```
...
vals := make([]string, 3)
ivals := make([]interface{}, 3)
for i := 0; i < len(vals); i++ {
ivals[i] = &vals[i]
}
fmt.Print("Enter text to scan: ")
fmt.Scan(ivals...)
Printfln("Name: %v", vals)
...
```

这是一个笨拙的过程，但可以将其封装在一个实用函数中，这样就不必每次都创建并填充 `interface` 切片。

### 处理换行符

默认情况下，扫描将换行符视为空格，作为值之间的分隔符。要查看此行为，请执行该项目，并在提示输入时，输入 `Kayak`，后跟空格，再跟 `Watersports`，然后按回车键，输入 `279`，再按一次回车键。此序列将产生以下输出：

```
Scanned 3 values
Name: Kayak, Category: Watersports, Price: 279.00
```

`Scan` 函数在接收到期望数量的值之前不会停止查找值，第一次按下回车键被视为分隔符，而不是输入的终止。表 17-12 中名称以 `ln` 结尾的函数（例如 `Scanln`）改变了此行为。清单 17-20 使用了 `Scanln` 函数。

```
package main
import "fmt"
func Printfln(template string, values ...interface{}) {
fmt.Printf(template + "\n", values...)
}
func main() {
var name string
var category string
var price float64
fmt.Print("Enter text to scan: ")
n, err := fmt.Scanln(&name, &category, &price)
if (err == nil) {
Printfln("Scanned %v values", n)
Printfln("Name: %v, Category: %v, Price: %.2f", name, category, price)
} else {
Printfln("Error: %v", err.Error())
}
}
清单 17-20
在 usingstrings 文件夹的 main.go 文件中使用 Scanln 函数
```

编译并执行项目，重复输入序列。当您第一次按下回车键时，换行符将终止输入，导致 `Scanln` 函数接收到的值少于所需数量，产生如下输出：

```
Error: unexpected newline
```



### 使用不同的字符串源

表 17-12 中描述的函数从三种来源扫描字符串：标准输入、读取器（在第 20 章中描述）以及作为参数提供的值。将字符串作为参数提供最为灵活，因为这意味着字符串可以来自任何地方。在代码清单 17-21 中，我用 `Sscan` 函数替换了 `Scanln` 函数，这样就可以扫描一个字符串变量了。

```
package main
import "fmt"
func Printfln(template string, values ...interface{}) {
fmt.Printf(template + "\n", values...)
}
func main() {
var name string
var category string
var price float64
source := "Lifejacket Watersports 48.95"
n, err := fmt.Sscan(source, &name, &category, &price)
if (err == nil) {
Printfln("Scanned %v values", n)
Printfln("Name: %v, Category: %v, Price: %.2f", name, category, price)
} else {
Printfln("Error: %v", err.Error())
}
}
代码清单 17-21
在 usingstrings 文件夹的 main.go 文件中扫描变量
```

`Sscan` 函数的第一个参数是要扫描的字符串，但在其他所有方面，扫描过程是相同的。编译并执行该项目，你将看到以下输出：

```
Scanned 3 values
Name: Lifejacket, Category: Watersports, Price: 48.95
```

### 使用扫描模板

可以使用模板扫描包含不需要的字符的字符串中的值，如代码清单 17-22 所示。

```
package main
import "fmt"
func Printfln(template string, values ...interface{}) {
fmt.Printf(template + "\n", values...)
}
func main() {
var name string
var category string
var price float64
source := "Product Lifejacket Watersports 48.95"
template := "Product %s %s %f"
n, err := fmt.Sscanf(source, template, &name, &category, &price)
if (err == nil) {
Printfln("Scanned %v values", n)
Printfln("Name: %v, Category: %v, Price: %.2f", name, category, price)
} else {
Printfln("Error: %v", err.Error())
}
}
代码清单 17-22
在 usingstrings 文件夹的 main.go 文件中使用模板
```

在代码清单 17-22 中使用的模板忽略了术语 `Product`，跳过了字符串的该部分，并允许从下一个术语开始扫描。编译并执行该项目，你将看到以下输出：

```
Scanned 3 values
Name: Lifejacket, Category: Watersports, Price: 48.95
```

使用模板进行扫描不如使用正则表达式灵活，因为扫描的字符串只能包含由空格分隔的值。但如果你只想要字符串中的某些值，并且不想定义复杂的匹配规则，使用模板仍然很有用。

## 本章小结

在本章中，我描述了标准库中用于格式化和扫描字符串的功能，这些功能都由 `fmt` 包提供。在下一章中，我将描述标准库提供的数学函数和切片排序功能。

