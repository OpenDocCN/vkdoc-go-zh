# 27. 使用反射

在本章中，我将描述 Go 对反射的支持，它允许应用程序处理在项目编译时未知的类型。例如，这对于创建将被其他项目使用的 API 非常有用。您可以在第 3 部分中看到反射的广泛使用，在那里我创建了一个自定义的 Web 应用程序框架。在这种情况下，应用程序框架中的代码对将要运行的应用程序定义的数据类型一无所知，必须使用反射来获取这些类型的信息，并处理由它们创建的值。

使用反射应谨慎。由于所使用的数据类型是未知的，编译器通常提供的安全保障无法使用，程序员有责任安全地检查和操作类型。反射代码通常冗长且难以阅读，并且在编写反射代码时很容易做出错误的假设，这些错误直到代码被其他开发者使用时才会暴露出来。反射代码中的错误通常会导致程序崩溃。

使用反射的代码比常规 Go 代码慢，尽管这在大多数项目中不会成为问题。除非您有特别苛刻的性能要求，否则您会发现所有 Go 代码都能以可接受的速度运行，无论它是否使用反射。有些 Go 编程任务只能通过反射完成，并且反射在整个标准库中都有使用。

这并不意味着您应该急于使用反射——因为它难以使用且容易出错——但有时它无法避免，一旦您理解了它的工作原理，谨慎地应用 Go 反射特性可以编写出灵活且适应性强的代码，正如您将在第 3 部分中看到的那样。表 27-1 将反射置于上下文中。

表 27-1：反射的上下文

| 问题 | 答案 |
| --- | --- |
| 它是什么？ | 反射允许在运行时检查类型和值，即使这些类型是在编译时未定义的。 |
| 它为什么有用？ | 当编写依赖于将来定义类型的代码时（例如，在编写将被其他项目使用的 API 时），反射非常有用。 |
| 如何使用？ | `reflect`包提供了允许反射类型和值的特性，使得可以在没有明确了解正在使用的数据类型的情况下使用它们。 |
| 有陷阱或局限性吗？ | 反射很复杂，需要高度关注细节。很容易对数据类型做出假设，这些假设直到代码在其他项目中使用时才会暴露出问题。 |
| 有替代方案吗？ | 只有在项目编译时类型未知时才需要反射。当类型预先已知时，应使用标准的 Go 语言特性。 |

表 27-2 总结了本章内容。

表 27-2：章节总结

| 问题 | 解决方案 | 清单 |
| --- | --- | --- |
| 获取反射后的类型和值 | 使用`TypeOf`和`ValueOf`函数 | 8 |
| 检查反射后的类型 | 使用`Type`接口定义的方法 | 9 |
| 检查反射后的值 | 使用`Value`结构体定义的方法 | 10 |
| 识别一个反射后的类型 | 检查其种类，并可选地检查其元素类型 | 11, 12 |
| 获取底层类型 | 使用`Interface`方法 | 13 |
| 设置反射后的值 | 使用`Set*`方法 | 14–16 |
| 比较反射后的值 | 使用`Comparable`方法和 Go 比较运算符，或使用`DeepEqual`函数 | 17–19 |
| 将反射后的值转换为不同类型 | 使用`ConvertibleTo`和`Convert`方法 | 20, 21 |
| 创建新的反射值 | 对于基本类型使用`New`类型，对于其他类型使用`Make*`方法之一 | 22 |

## 准备本章内容

为了准备本章的内容，请打开一个新的命令提示符，导航到一个方便的位置，并创建一个名为`reflection`的目录。然后在`reflection`文件夹中运行清单 27-1 所示的命令来创建一个模块文件。

> 提示
> 您可以从[`https://github.com/apress/pro-go`](https://github.com/apress/pro-go)下载本章以及本书其他章节的示例项目。如果您在运行示例时遇到问题，请参阅第 2 章以获取帮助。

```
go mod init reflection
清单 27-1
初始化模块
```

在`reflection`文件夹中添加一个名为`printer.go`的文件，内容如清单 27-2 所示。

```
package main
import "fmt"
func Printfln(template string, values ...interface{}) {
fmt.Printf(template + "\n", values...)
}
清单 27-2
reflection 文件夹中 printer.go 文件的内容
```

在`reflection`文件夹中添加一个名为`types.go`的文件，内容如清单 27-3 所示。

```
package main
type Product struct {
Name, Category string
Price float64
}
type Customer struct {
Name, City string
}
清单 27-3
reflection 文件夹中 types.go 文件的内容
```

在`reflection`文件夹中添加一个名为`main.go`的文件，内容如清单 27-4 所示。

```
package main
func printDetails(values ...Product) {
for _, elem := range values {
Printfln("Product: Name: %v, Category: %v, Price: %v",
elem.Name, elem.Category, elem.Price)
}
}
func main() {
product := Product {
Name: "Kayak", Category: "Watersports", Price: 279,
}
printDetails(product)
}
清单 27-4
reflection 文件夹中 main.go 文件的内容
```

使用命令提示符，在`usingstrings`文件夹中运行清单 27-5 所示的命令。

```
go run .
清单 27-5
运行示例项目
```

项目中的代码将被编译并执行，产生以下结果：

```
Product: Name: Kayak, Category: Watersports, Price: 279
```



### 理解反射的必要性

Go 的类型系统执行严格，这意味着当期望的是不同类型时，你不能使用某一类型的值。清单 27-6 创建了一个 `Customer` 值并将其传递给 `printDetails` 函数，该函数定义了一个可变参数 `Product`。

```
package main
func printDetails(values ...Product) {
for _, elem := range values {
Printfln("Product: Name: %v, Category: %v, Price: %v",
elem.Name, elem.Category, elem.Price)
}
}
func main() {
product := Product {
Name: "Kayak", Category: "Watersports", Price: 279,
}
customer := Customer { Name: "Alice", City: "New York" }
printDetails(product, customer)
}
```

这段代码无法编译，因为它违反了 Go 的类型规则。当你编译项目时，会看到以下错误：

```
.\main.go:16:17: cannot use customer (type Customer) as type Product in argument to printDetails
```

在第 11 章中，我介绍了接口，它允许通过方法定义共同特征，这些方法可以被任何实现该接口的类型调用。第 11 章还介绍了空接口，它可以用于接受任何类型，如清单 27-7 所示。

```
package main
func printDetails(values ...interface{}) {
for _, elem := range values {
switch val := elem.(type) {
case Product:
Printfln("Product: Name: %v, Category: %v, Price: %v",
val.Name, val.Category, val.Price)
case Customer:
Printfln("Customer: Name: %v, City: %v", val.Name, val.City)
}
}
}
func main() {
product := Product {
Name: "Kayak", Category: "Watersports", Price: 279,
}
customer := Customer { Name: "Alice", City: "New York" }
printDetails(product, customer)
}
```

空接口允许 `printDetails` 函数接收任何类型，但由于接口未定义任何方法，因此不允许访问特定特性。需要使用类型断言将空接口缩小到特定类型，这样才能处理每个值。编译并执行代码，你将收到以下输出：

```
Product: Name: Kayak, Category: Watersports, Price: 279
Customer: Name: Alice, City: New York
```

这种方法的局限性在于 `printDetails` 函数只能处理预先已知的类型。每当我向项目添加一个类型时，就必须扩展 `printDetails` 函数来处理该类型。

许多项目处理的类型集合足够小，这不会成为问题，或者能够定义具有提供通用功能访问权限方法的接口。反射则解决了那些不满足这些条件的项目的问题，原因可能是要处理的类型数量庞大，或者无法编写接口和方法。

## 使用反射

`reflect` 包提供了 Go 的反射特性，关键函数是 `TypeOf` 和 `ValueOf`，表 27-3 简要描述了这两个函数。

**表 27-3** 关键反射函数

| 名称 | 描述 |
| --- | --- |
| `TypeOf(val)` | 此函数返回一个实现了 `Type` 接口的值，该接口描述指定值的类型。 |
| `ValueOf(val)` | 此函数返回一个 `Value` 结构体，该结构体允许检查和操作指定的值。 |

`TypeOf` 和 `ValueOf` 函数及其结果背后有大量细节，很容易忽略反射为何有用。在深入细节之前，清单 27-8 修改了 `printDetails` 函数以使用 `reflect` 包，使其能够处理任何类型，展示了应用反射所需的基本模式。

```
package main
import (
"reflect"
"strings"
"fmt"
)
func printDetails(values ...interface{}) {
for _, elem := range values {
fieldDetails := []string {}
elemType := reflect.TypeOf(elem)
elemValue := reflect.ValueOf(elem)
if elemType.Kind() == reflect.Struct {
for i := 0; i < elemType.NumField(); i++ {
fieldName := elemType.Field(i).Name
fieldVal := elemValue.Field(i)
fieldDetails = append(fieldDetails,
fmt.Sprintf("%v: %v", fieldName, fieldVal ))
}
Printfln("%v: %v", elemType.Name(), strings.Join(fieldDetails, ", "))
} else {
Printfln("%v: %v", elemType.Name(), elemValue)
}
}
}
type Payment struct {
Currency string
Amount float64
}
func main() {
product := Product {
Name: "Kayak", Category: "Watersports", Price: 279,
}
customer := Customer { Name: "Alice", City: "New York" }
payment := Payment { Currency: "USD", Amount: 100.50 }
printDetails(product, customer, payment, 10, true)
}
```

使用反射的代码可能很冗长，但一旦熟悉了基础知识，基本模式就会变得容易理解。需要记住的关键点是，反射有两个协同工作的方面：**反射类型**和**反射值**。

**反射类型**让你能够在事先不知道的情况下访问 Go 类型的详细信息。你可以探索反射类型，通过 `Type` 接口定义的方法了解其细节和特性。

**反射值**让你能够操作你所拥有的具体值。例如，当你不知道正在处理的类型时，你不能像在普通代码中那样仅通过读取结构体字段或调用方法来操作它。

反射类型和反射值的使用导致了代码冗长。例如，如果你知道正在处理一个 `Product` 结构体，你可以直接读取 `Name` 字段并获取 `string` 结果。如果你不知道正在使用什么类型，那么必须使用反射类型来确定你是否在处理一个结构体，以及它是否有 `Name` 字段。一旦确定存在这样一个字段，你就使用反射值来读取该字段并获取其值。

反射可能会令人困惑，因此我将逐步介绍清单 27-8 中的语句，并简要描述每条语句的作用，这将为接下来对 `reflect` 包的详细描述提供一些背景。

`printDetails` 函数使用空接口定义了一个可变参数，该参数通过 `range` 关键字进行枚举：

```
...
func printDetails(values ...interface{}) {
for _, elem := range values {
...
```

如前所述，空接口允许函数接受任何数据类型，但不允许访问任何特定类型的特性。`reflect` 包用于获取每个接收值的反射类型和反射值：

```
...
elemType := reflect.TypeOf(elem)
elemValue := reflect.ValueOf(elem)
...
```



`TypeOf`函数返回反射类型，该类型由`Type`接口描述。`ValueOf`函数返回反射值，该值由`Value`接口表示。

下一步是确定正在处理什么类型的类型，通过调用`Type.Kind`方法完成：

```
...
if elemType.Kind() == reflect.Struct {
...
```

`reflect`包定义了标识 Go 中不同种类类型的常量，我在表 27-5 中进行了描述。在此语句中，使用`if`语句判断反射类型是否为结构体。如果是结构体，则结合`NumField`方法使用`for`循环，该方法返回结构体定义的字段数量：

```
...
for i := 0; i < elemType.NumField(); i++ {
...
```

在`for`循环中，获取字段的名称和值：

```
...
fieldName := elemType.Field(i).Name
fieldVal := elemValue.Field(i)
...
```

对反射类型调用`Field`方法会返回一个`StructField`，它描述单个字段，包括`Name`字段。对反射值调用`Field`方法会返回一个`Value`结构体，它代表字段的值。

字段的名称和值被添加到一个字符串切片中，该切片构成输出的一部分。使用`fmt`包创建字段值的字符串表示：

```
...
fieldDetails = append(fieldDetails, fmt.Sprintf("%v: %v", fieldName, fieldVal))
...
```

处理完所有结构体字段后，写出一个包含反射类型名称的字符串，该名称通过`Name`方法获取，并为每个字段获取详细信息：

```
...
Printfln("%v: %v", elemType.Name(), strings.Join(fieldDetails, ", "))
...
```

如果反射类型不是结构体，则写出更简单的消息，包含反射类型名称和值，其格式化由`fmt`包处理：

```
...
Printfln("%v: %v", elemType.Name(), elemValue)
...
```

新代码允许`printDetails`函数接收任何数据类型，包括新定义的`Payment`结构体，以及诸如`int`和`bool`值之类的内置类型。编译并执行项目，你将看到以下输出：

```
Product: Name: Kayak, Category: Watersports, Price: 279
Customer: Name: Alice, City: New York
Payment: Currency: USD, Amount: 100.5
int: 10
bool: true
```

### 使用基本类型特性

`Type`接口通过表 27-4 中描述的方法提供类型的基本详细信息。有专门的方法用于处理特定类型的类型，例如数组，这些将在后续章节中描述，但这些是提供所有类型基本详细信息的方法。

**表 27-4：Type 接口定义的基本方法**

| 名称 | 描述 |
| --- | --- |
| `Name()` | 此方法返回类型的名称。 |
| `PkgPath()` | 此方法返回类型的包路径。对于内置类型，如`int`和`bool`，返回空字符串。 |
| `Kind()` | 此方法返回类型的种类，使用与`reflect`包定义的常量值之一匹配的值，如表 27-5 所述。 |
| `String()` | 此方法返回类型名称的字符串表示，包括包名称。 |
| `Comparable()` | 如果此类型的值可以使用标准比较运算符进行比较，则此方法返回`true`，如“比较值”部分所述。 |
| `AssignableTo(type)` | 如果此类型的值可以分配给指定反射类型的变量或字段，则此方法返回`true`。 |

`reflect`包定义了一个名为`Kind`的类型，它是`uint`的别名，并用于一系列描述不同类型种类的常量，如表 27-5 所述。

**表 27-5：Kind 常量**

| 名称 | 描述 |
| --- | --- |
| `Bool` | 此值表示`bool`。 |
| `Int`, `Int8`, `Int16`, `Int32`, `Int64` | 这些值表示不同大小的整数类型。 |
| `Uint`, `Uint8`, `Uint16`, `Uint32`, `Uint64` | 这些值表示不同大小的无符号整数类型。 |
| `Float32`, `Float64` | 这些值表示不同大小的浮点类型。 |
| `String` | 此值表示字符串。 |
| `Struct` | 此值表示结构体。 |
| `Array` | 此值表示数组。 |
| `Slice` | 此值表示切片。 |
| `Map` | 此值表示映射。 |
| `Chan` | 此值表示通道。 |
| `Func` | 此值定义一个函数。 |
| `Interface` | 此值表示接口。 |
| `Ptr` | 此值表示指针。 |
| `Uintptr` | 此值表示不安全指针，本书未对此进行描述。 |

清单 27-9 简化了示例，以显示`printDetails`函数接收的每个值的反射类型的详细信息。

```
package main
import (
    "reflect"
    // "strings"
    // "fmt"
)
func getTypePath(t reflect.Type) (path string) {
    path = t.PkgPath()
    if (path == "") {
        path = "(built-in)"
    }
    return
}
func printDetails(values ...interface{}) {
    for _, elem := range values {
        elemType := reflect.TypeOf(elem)
        Printfln("Name: %v, PkgPath: %v, Kind: %v",
            elemType.Name(), getTypePath(elemType), elemType.Kind())
    }
}
type Payment struct {
    Currency string
    Amount float64
}
func main() {
    product := Product {
        Name: "Kayak", Category: "Watersports", Price: 279,
    }
    customer := Customer { Name: "Alice", City: "New York" }
    payment := Payment { Currency: "USD", Amount: 100.50 }
    printDetails(product, customer, payment, 10, true)
}
**清单 27-9：在 reflection 文件夹的 main.go 文件中打印类型详细信息**
```

我添加了一个函数来替换空的包名称，以便更清楚地描述内置类型。编译并执行项目，你将看到以下输出：

```
Name: Product, PkgPath: main, Kind: struct
Name: Customer, PkgPath: main, Kind: struct
Name: Payment, PkgPath: main, Kind: struct
Name: int, PkgPath: (built-in), Kind: int
Name: bool, PkgPath: (built-in), Kind: bool
```

许多特定于单个类型种类（例如数组）的反射特性，如果对其他类型调用，会导致 panic，这使得`Kind`方法在使用反射时尤其重要。



### 使用基本值特性

对于每组反射类型特性，都有相应的反射值特性。`Value` 结构体定义了表 27-6 中描述的方法，这些方法提供了对基本反射特性的访问，包括访问底层值。

**表 27-6** `Value` 结构体定义的基本方法

| 名称 | 描述 |
| --- | --- |
| `Kind()` | 此方法使用表 27-5 中的某个值返回该值类型的种类。 |
| `Type()` | 此方法返回该 `Value` 对应的 `Type`。 |
| `IsNil()` | 如果该值为 `nil`，则此方法返回 `true`。如果底层值不是函数、接口、指针、切片或通道，此方法将引发 panic。 |
| `IsZero()` | 如果底层值是其类型的零值，则此方法返回 `true`。 |
| `Bool()` | 此方法返回底层的 `bool` 值。如果底层值的 `Kind` 不是 `Bool`，此方法将引发 panic。 |
| `Bytes()` | 此方法返回底层的 `[]byte` 值。如果底层值不是 `byte` 切片，此方法将引发 panic。我将在“识别字节切片”部分演示如何确定切片的类型。 |
| `Int()` | 此方法以 `int64` 形式返回底层值。如果底层值的 `Kind` 不是 `Int`、`Int8`、`Int16`、`Int32` 或 `Int64`，此方法将引发 panic。 |
| `Uint()` | 此方法以 `uint64` 形式返回底层值。如果底层值的 `Kind` 不是 `Uint`、`Uint8`、`Uint16`、`Uint32` 或 `Uint64`，此方法将引发 panic。 |
| `Float()` | 此方法以 `float64` 形式返回底层值。如果底层值的 `Kind` 不是 `Float32` 或 `Float64`，此方法将引发 panic。 |
| `String()` | 如果值的 `Kind` 是 `String`，此方法以字符串形式返回底层值。对于其他 `Kind` 值，此方法返回字符串 `<T Value>`，其中 `T` 是底层类型，例如 `<int Value>`。 |
| `Elem()` | 此方法返回指针所引用的 `Value`。此方法也可与接口一起使用，如第 29 章所述。如果底层值的 `Kind` 不是 `Ptr`，此方法将引发 panic。 |
| `IsValid()` | 如果 `Value` 是零值（例如通过 `Value{}` 创建，而不是通过 `ValueOf` 获取），则此方法返回 `false`。此方法不涉及是其反射类型零值的反射值。如果此方法返回 `false`，则所有其他 `Value` 方法都将引发 panic。 |

在使用返回底层值的方法时，务必检查 `Kind` 结果以避免 panic。清单 27-10 演示了表中描述的一些方法。

```go
package main
import (
"reflect"
// "strings"
// "fmt"
)
func printDetails(values ...interface{}) {
for _, elem := range values {
elemValue := reflect.ValueOf(elem)
switch elemValue.Kind() {
case reflect.Bool:
var val bool = elemValue.Bool()
Printfln("Bool: %v", val)
case reflect.Int:
var val int64 = elemValue.Int()
Printfln("Int: %v", val)
case reflect.Float32, reflect.Float64:
var val float64 = elemValue.Float()
Printfln("Float: %v", val)
case reflect.String:
var val string = elemValue.String()
Printfln("String: %v", val)
case reflect.Ptr:
var val reflect.Value = elemValue.Elem()
if (val.Kind() == reflect.Int) {
Printfln("Pointer to Int: %v", val.Int())
}
default:
Printfln("Other: %v", elemValue.String())
}
}
}
func main() {
product := Product {
Name: "Kayak", Category: "Watersports", Price: 279,
}
number := 100
printDetails(true, 10, 23.30, "Alice", &number, product)
}
```

**清单 27-10** 在 `reflection` 文件夹的 `main.go` 文件中使用基本值方法

此示例使用 `switch` 语句结合 `Kind` 方法的结果来确定值的类型，并调用适当的方法来获取底层值。编译并执行该项目，你将看到以下输出：

```
Bool: true
Int: 10
Float: 23.3
String: Alice
Pointer to Int: 100
Other: 
```

`String` 方法的行为与其他方法不同，当在非 `string` 类型的值上调用时不会引发 panic。相反，该方法会返回类似这样的 `string`：

```
...
Other: 
...
```

这不是 Go 标准库中其他地方常见的 `String` 方法用法，在这些地方，此方法通常返回某个值的 `string` 表示。在使用反射时，你可以使用后面章节描述的技术，或者依赖使用相同技术的 `format` 包来为你创建值的 `string` 表示。

### 识别类型

请注意，在清单 27-10 中处理指针时需要两个步骤。第一步使用 `Kind` 方法识别 `Ptr` 值，第二步使用 `Elem` 方法获取表示指针所引用数据的 `Value`：

```go
...
case reflect.Ptr:
var val reflect.Value = elemValue.Elem()
if (val.Kind() == reflect.Int) {
Printfln("Pointer to Int: %v", val.Int())
}
...
```

第一步告诉我正在处理一个指针，第二步告诉我它指向一个 `int` 值。这个过程可以通过执行反射类型的比较来简化。如果两个值具有相同的 Go 数据类型，那么将比较运算符应用于 `reflect.TypeOf` 函数的结果时，它将返回 `true`，如清单 27-11 所示。

```go
package main
import (
"reflect"
// "strings"
// "fmt"
)
var intPtrType = reflect.TypeOf((*int)(nil))
func printDetails(values ...interface{}) {
for _, elem := range values {
elemValue := reflect.ValueOf(elem)
elemType := reflect.TypeOf(elem)
if (elemType == intPtrType) {
Printfln("Pointer to Int: %v", elemValue.Elem().Int())
} else {
switch elemValue.Kind() {
case reflect.Bool:
var val bool = elemValue.Bool()
Printfln("Bool: %v", val)
case reflect.Int:
var val int64 = elemValue.Int()
Printfln("Int: %v", val)
case reflect.Float32, reflect.Float64:
var val float64 = elemValue.Float()
Printfln("Float: %v", val)
case reflect.String:
var val string = elemValue.String()
Printfln("String: %v", val)
// case reflect.Ptr:
//     var val reflect.Value = elemValue.Elem()
//     if (val.Kind() == reflect.Int) {
//         Printfln("Pointer to Int: %v", val.Int())
//     }
default:
Printfln("Other: %v", elemValue.String())
}
}
}
}
func main() {
product := Product {
Name: "Kayak", Category: "Watersports", Price: 279,
}
number := 100
printDetails(true, 10, 23.30, "Alice", &number, product)
}
```

**清单 27-11** 在 `reflection` 文件夹的 `main.go` 文件中比较类型

这种技术从一个 `nil` 值开始，将其转换为一个指向 `int` 值的指针，然后将其传递给 `TypeOf` 函数以获取一个可用于比较的 `Type`：

```go
...
var intPtrType = reflect.TypeOf((*int)(nil))
...
```

执行此操作所需的括号使其难以阅读，但这种方法避免了仅为了获取其 `Type` 而定义变量的需要。该 `Type` 可以与普通的 Go 比较运算符一起使用：

```go
...
if (elemType == intPtrType) {
Printfln("Pointer to Int: %v", elemValue.Elem().Int())
} else {
...
```

像这样比较类型可能比检查指针类型及其指向值的 `Kind` 值更简单。编译并执行代码，你将看到以下输出：

```
Bool: true
Int: 10
Float: 23.3
String: Alice
Pointer to Int: 100
Other: 
```



### 识别字节切片

使用比较运算符也是安全调用 `Bytes()` 方法的一种好方式。如果对非字节切片类型调用 `Bytes()` 方法，程序会崩溃（panic），但 `Kind()` 方法仅能指示切片类型，而无法区分其内容。清单 27-12 为 `byte` 切片定义了一个类型变量，并与比较运算符配合使用，以判断何时可以安全调用 `Bytes()` 方法。

```go
package main
import (
"reflect"
// "strings"
// "fmt"
)
var intPtrType = reflect.TypeOf((*int)(nil))
var byteSliceType = reflect.TypeOf([]byte(nil))
func printDetails(values ...interface{}) {
for _, elem := range values {
elemValue := reflect.ValueOf(elem)
elemType := reflect.TypeOf(elem)
if (elemType == intPtrType) {
Printfln("Pointer to Int: %v", elemValue.Elem().Int())
} else if (elemType == byteSliceType) {
Printfln("Byte slice: %v", elemValue.Bytes())
} else {
switch elemValue.Kind() {
case reflect.Bool:
var val bool = elemValue.Bool()
Printfln("Bool: %v", val)
case reflect.Int:
var val int64 = elemValue.Int()
Printfln("Int: %v", val)
case reflect.Float32, reflect.Float64:
var val float64 = elemValue.Float()
Printfln("Float: %v", val)
case reflect.String:
var val string = elemValue.String()
Printfln("String: %v", val)
default:
Printfln("Other: %v", elemValue.String())
}
}
}
}
func main() {
product := Product {
Name: "Kayak", Category: "Watersports", Price: 279,
}
number := 100
slice := []byte("Alice")
printDetails(true, 10, 23.30, "Alice", &number, product, slice)
}
清单 27-12
在 reflection 文件夹的 main.go 文件中识别字节切片
```

编译并执行该项目，你将看到以下输出，其中包含了对 `byte` 切片的检测：

```
Bool: true
Int: 10
Float: 23.3
String: Alice
Pointer to Int: 100
Other: 
Byte slice: [65 108 105 99 101]
```

### 获取底层值

`Value` 结构体定义了表 27-7 中描述的方法，用于获取底层值。

**表 27-7** 用于获取底层值的 `Value` 方法

| 名称 | 描述 |
| --- | --- |
| `Interface()` | 此方法通过空接口返回底层值。如果用于未导出的结构体字段，此方法会崩溃（panic）。|
| `CanInterface()` | 如果 `Interface()` 方法可以安全使用而不会崩溃，此方法返回 `true`。|

`Interface()` 方法允许你跳出反射，获取一个可由普通 Go 代码使用的值，如清单 27-13 所示。

```go
package main
import (
"reflect"
// "strings"
// "fmt"
)
func selectValue(data interface{}, index int) (result interface{}) {
dataVal := reflect.ValueOf(data)
if (dataVal.Kind() == reflect.Slice) {
result = dataVal.Index(index).Interface()
}
return
}
func main() {
names := []string {"Alice", "Bob", "Charlie"}
val := selectValue(names, 1).(string)
Printfln("Selected: %v", val)
}
清单 27-13
在 reflection 文件夹的 main.go 文件中获取底层值
```

`selectValue` 函数用于从切片中选择一个值，而无需知道切片的元素类型。该值通过 `Index()` 方法从切片中获取，该方法在第 28 章中有描述。对于本章来说，重要的是 `Index()` 方法返回一个 `Value`，这仅对使用反射的代码有用。`Interface()` 方法用于获取一个可作为函数结果的值：

```go
...
result = dataVal.Index(index).Interface()
...
```

使用反射的一个缺点是函数和方法结果的处理方式。如果结果的类型不是固定的，那么函数或方法的调用者就需要负责将结果转换为特定类型，清单 27-13 中的以下语句正是如此：

```go
...
val := selectValue(names, 1).(string)
...
```

`selectValue` 函数的结果将与切片元素具有相同的类型，但在 Go 中无法表达这一点，这就是为什么该函数使用空接口作为结果，也是 `Interface()` 方法返回空接口的原因。

这带来的问题是，调用代码需要了解函数的内部工作方式才能处理结果。当函数的行为发生变化时，这种变化必须反映在所有调用该函数的代码中，这需要一种往往难以保持的细致程度。

这并不理想——这也是应谨慎使用反射的原因之一。编译并执行该项目，你将看到以下输出：

```
Selected: Bob
```



### 使用反射设置值

`Value`（值）结构体定义了允许通过反射设置值的方法，如表 27-8 所述。

**表 27-8** 用于设置值的 Value 方法

| 名称 | 描述 |
| --- | --- |
| `CanSet()` | 如果该值可以被设置则返回 `true`，否则返回 `false`。 |
| `SetBool(val)` | 该方法将底层值设置为指定的 `bool`（布尔值）。 |
| `SetBytes(slice)` | 该方法将底层值设置为指定的 `byte`（字节）切片。 |
| `SetFloat(val)` | 该方法将底层值设置为指定的 `float64`（64 位浮点数）。 |
| `SetInt(val)` | 该方法将底层值设置为指定的 `int64`（64 位整数）。 |
| `SetUint(val)` | 该方法将底层值设置为指定的 `uint64`（64 位无符号整数）。 |
| `SetString(val)` | 该方法将底层值设置为指定的 `string`（字符串）。 |
| `Set(val)` | 该方法将底层值设置为指定 `Value` 的底层值。 |

表 27-8 中的 `Set` 方法，如果 `CanSet` 方法返回 `false`，或者被用于设置类型不符的值，则会引发恐慌（panic）。代码清单 27-14 展示了 `CanSet` 方法所解决的问题。

```
package main
import (
"reflect"
"strings"
// "fmt"
)
func incrementOrUpper(values ...interface{}) {
for _, elem := range values {
elemValue := reflect.ValueOf(elem)
if (elemValue.CanSet()) {
switch (elemValue.Kind()) {
case reflect.Int:
elemValue.SetInt(elemValue.Int() + 1)
case reflect.String:
elemValue.SetString(strings.ToUpper( elemValue.String()))
}
Printfln("Modified Value: %v", elemValue)
} else {
Printfln("Cannot set %v: %v", elemValue.Kind(), elemValue)
}
}
}
func main() {
name := "Alice"
price := 279
city := "London"
incrementOrUpper(name, price, city)
for _, val := range []interface{} { name, price, city }  {
Printfln("Value: %v", val)
}
}
代码清单 27-14
在 reflection 文件夹中的 main.go 文件中创建不可设置的值
```

`incrementOrUpper` 函数递增 `int` 值，并将字符串值转换为大写。编译并执行代码，你将看到以下输出，显示传递给 `incrementOrUpper` 函数的所有值都无法被设置：

```
Cannot set string: Alice
Cannot set int: 279
Cannot set string: London
Value: Alice
Value: 279
Value: London
```

`CanSet` 方法可能会引起混淆，但请记住，当值作为函数和方法的参数时，它们会被复制。当这些值被传递给 `incrementOrUpper` 时，它们被复制了：

```
...
incrementOrUpper(name, price, city)
...
```

这阻止了这些值被修改，因为这些值已被复制供函数内部使用。代码清单 27-15 通过使用指针解决了这个问题。

```
package main
import (
"reflect"
"strings"
// "fmt"
)
func incrementOrUpper(values ...interface{}) {
for _, elem := range values {
elemValue := reflect.ValueOf(elem)
if (elemValue.Kind() == reflect.Ptr) {
elemValue = elemValue.Elem()
}
if (elemValue.CanSet()) {
switch (elemValue.Kind()) {
case reflect.Int:
elemValue.SetInt(elemValue.Int() + 1)
case reflect.String:
elemValue.SetString(strings.ToUpper( elemValue.String()))
}
Printfln("Modified Value: %v", elemValue)
} else {
Printfln("Cannot set %v: %v", elemValue.Kind(), elemValue)
}
}
}
func main() {
name := "Alice"
price := 279
city := "London"
incrementOrUpper(&name, &price, &city)
for _, val := range []interface{} { name, price, city }  {
Printfln("Value: %v", val)
}
}
代码清单 27-15
在 reflection 文件夹中的 main.go 文件中设置值
```

因此，就像常规代码一样，反射只有在能够访问原始存储时才能修改一个值。在代码清单 27-15 中，使用指针来调用 `incrementOrUpper` 函数，这需要修改反射代码以检测指针，并在找到指针时使用 `Elem` 方法沿着指针找到其值。编译并执行该项目，你将看到以下输出：

```
Modified Value: ALICE
Modified Value: 280
Modified Value: LONDON
Value: ALICE
Value: 280
Value: LONDON
```

## 使用一个值设置另一个值

`Set` 方法允许使用一个 `Value` 来设置另一个 `Value`，这是一种使用通过反射接收到的值来修改值的便捷方式，如代码清单 27-16 所示。

```
package main
import (
"reflect"
//"strings"
// "fmt"
)
func setAll(src interface{}, targets ...interface{}) {
srcVal := reflect.ValueOf(src)
for _, target := range  targets {
targetVal := reflect.ValueOf(target)
if (targetVal.Kind() == reflect.Ptr &&
targetVal.Elem().Type() == srcVal.Type() &&
targetVal.Elem().CanSet()) {
targetVal.Elem().Set(srcVal)
}
}
}
func main() {
name := "Alice"
price := 279
city := "London"
setAll("New String", &name, &price, &city)
setAll(10, &name, &price, &city)
for _, val := range []interface{} { name, price, city }  {
Printfln("Value: %v", val)
}
}
代码清单 27-16
在 reflection 文件夹中的 main.go 文件中使用一个值设置另一个值
```

`setAll` 函数使用 `for` 循环处理其可变参数，并查找那些指向与 `src` 参数类型相同值的指针。当找到匹配的指针时，它所引用的值会通过 `Set` 方法被更改。`setAll` 函数中的大部分代码负责检查值的兼容性和可设置性，但结果就是，使用一个 `string` 作为第一个参数会设置所有后续的 `string` 参数，而使用一个 `int` 则会设置所有后续的 `int` 值。编译并执行代码，你将收到以下输出：

```
Value: New String
Value: 10
Value: New String
```



### 比较值

并非所有数据类型都能使用 Go 语言的关系运算符进行比较，这很容易在反射代码中引发恐慌（panic），如清单 27-17 所示。

```
package main
import (
"reflect"
//"strings"
// "fmt"
)
func contains(slice interface{}, target interface{}) (found bool) {
sliceVal := reflect.ValueOf(slice)
if (sliceVal.Kind() == reflect.Slice) {
for i := 0; i < sliceVal.Len(); i++ {
if sliceVal.Index(i).Interface() == target {
found = true
}
}
}
return
}
func main() {
// name := "Alice"
// price := 279
city := "London"
citiesSlice := []string { "Paris", "Rome", "London"}
Printfln("Found #1: %v", contains(citiesSlice, city))
sliceOfSlices := [][]string {
citiesSlice,  { "First", "Second", "Third"}}
Printfln("Found #2: %v", contains(sliceOfSlices, citiesSlice))
}
Listing 27-17
Comparing Value in the main.go File in the reflection Folder
```

`contains` 函数接受一个切片，如果其中包含指定值则返回 `true`。该函数使用 `Len` 和 `Index` 方法枚举切片，这两个方法将在第 28 章中介绍，但本节中重要的语句是这一行：

```
...
if sliceVal.Index(i).Interface() == target {
...
```

该语句将关系运算符应用于切片中特定索引处的值和目标值。但由于 `contains` 函数接受任何类型，如果函数接收到的类型无法进行比较，应用程序就会发生恐慌。编译并执行该项目，你将看到以下输出：

```
Found #1: true
panic: runtime error: comparing uncomparable type []string
goroutine 1 [running]:
main.contains(0x243640, 0xc000114078, 0x243f00, 0xc000153f60, 0xc000153f40)
C:/reflection/main.go:13 +0x1a5
main.main()
C:/reflection/main.go:33 +0x2e5
exit status 2
```

在清单 27-17 中，`main` 函数调用了两次 `contains` 函数。第一次调用成功，因为切片包含字符串值，字符串值可以使用关系运算符。第二次调用失败，因为切片包含其他切片，而关系运算符不能应用于切片。为了帮助避免这个问题，`Type` 接口定义了表 27-9 中描述的方法。

表 27-9

用于判断类型是否可比较的 Type 方法

| 名称 | 描述 |
| --- | --- |
| `Comparable()` | 如果反射类型可以与 Go 关系运算符一起使用，则此方法返回 `true`，否则返回 `false`。 |

清单 27-18 展示了如何使用 `Comparable` 方法来避免执行会导致恐慌的比较操作。

```
...
func contains(slice interface{}, target interface{}) (found bool) {
sliceVal := reflect.ValueOf(slice)
targetType := reflect.TypeOf(target)
if (sliceVal.Kind() == reflect.Slice &&
sliceVal.Type().Elem().Comparable() &&
targetType.Comparable()) {
for i := 0; i < sliceVal.Len(); i++ {
if sliceVal.Index(i).Interface() == target {
found = true
}
}
}
return
}
...
Listing 27-18
Safely Comparing Values in the main.go File in the reflection Folder
```

这些更改确保关系运算符仅应用于那些类型可比较的值。编译并执行该项目，你将看到以下输出：

```
Found #1: true
Found #2: false
```

## 使用比较便捷函数

`reflect` 包定义了一个函数，该函数提供了标准 Go 关系运算符的替代方案，如表 27-10 所述。

表 27-10

用于比较值的 reflect 包函数

| 名称 | 描述 |
| --- | --- |
| `DeepEqual(val, val)` | 此函数比较任意两个值，如果它们相同则返回 `true`。 |

`DeepEqual` 函数不会引发恐慌，并且可以执行 `==` 运算符无法进行的额外比较。此函数的所有比较规则都列在 [`https://pkg.go.dev/reflect@go1.17.1#DeepEqual`](https://pkg.go.dev/reflect%2540go1.17.1%2523DeepEqual) 上，但总的来说，`DeepEqual` 函数通过递归检查值的所有字段或元素来进行比较。这种比较最有用的一个方面是，如果切片的所有值都相等，则切片相等，这解决了标准关系运算符最常遇到的限制之一，如清单 27-19 所示。

```
package main
import (
"reflect"
//"strings"
// "fmt"
)
func contains(slice interface{}, target interface{}) (found bool) {
sliceVal := reflect.ValueOf(slice)
if (sliceVal.Kind() == reflect.Slice) {
for i := 0; i < sliceVal.Len(); i++ {
if reflect.DeepEqual(sliceVal.Index(i).Interface(), target) {
found = true
}
}
}
return
}
func main() {
// name := "Alice"
// price := 279
city := "London"
citiesSlice := []string { "Paris", "Rome", "London"}
Printfln("Found #1: %v", contains(citiesSlice, city))
sliceOfSlices := [][]string {
citiesSlice,  { "First", "Second", "Third"}}
Printfln("Found #2: %v", contains(sliceOfSlices, citiesSlice))
}
Listing 27-19
Performing a Comparison in the main.go File in the reflection Folder
```

这种简化版的 `contains` 函数无需检查类型是否可比较，并且在编译和执行项目时会产生以下输出：

```
Found #1: true
Found #2: true
```

这个例子可以比较切片，因此对 `contains` 函数的两次调用都产生 `true` 结果。

## 转换值

如第 3 部分所述，Go 语言支持类型转换，允许将定义为一个类型的值以不同的类型表示。`Type` 接口定义了表 27-11 中描述的方法，用于判断反射类型是否可以转换。

表 27-11

用于评估类型转换的 Type 方法

| 名称 | 描述 |
| --- | --- |
| `ConvertibleTo(type)` | 如果调用该方法的 `Type` 可以转换为指定的 `Type`，则此方法返回 `true`。 |

`Type` 接口定义的方法允许检查类型的可转换性。表 27-12 描述了 `Value` 结构体定义的用于执行转换的方法。

表 27-12

用于转换类型的 Value 方法

| 名称 | 描述 |
| --- | --- |
| `Convert(type)` | 此方法执行类型转换，并返回一个包含新类型和原始值的 `Value`。 |

清单 27-20 演示了使用反射执行的简单类型转换。

```
package main
import (
"reflect"
//"strings"
// "fmt"
)
func convert(src, target interface{}) (result interface{}, assigned bool) {
srcVal := reflect.ValueOf(src)
targetVal := reflect.ValueOf(target)
if (srcVal.Type().ConvertibleTo(targetVal.Type())) {
result = srcVal.Convert(targetVal.Type()).Interface()
assigned = true
} else {
result = src
}
return
}
func main() {
name := "Alice"
price := 279
//city := "London"
newVal, ok := convert(price, 100.00)
Printfln("Converted %v: %v, %T", ok, newVal, newVal)
newVal, ok = convert(name, 100.00)
Printfln("Converted %v: %v, %T", ok, newVal, newVal)
}
Listing 27-20
Performing a Type Conversion in the main.go File in the reflection Folder
```

`convert` 函数尝试将一个值转换为另一个值的类型，这是通过 `ConvertibleTo` 和 `Convert` 方法完成的。第一次调用 `convert` 函数尝试将 `int` 值转换为 `float64`，成功了；第二次调用尝试将 `string` 转换为 `float64`，失败了。编译并执行该项目，你将看到以下输出：

```
Converted true: 279, float64
Converted false: Alice, string
```



### 转换数值类型

`Value` 结构体定义了表 27-13 所示的方法，用于检查某个值在转换为目标类型时是否会导致溢出。这些方法在将一种数值类型转换为另一种数值类型时非常有用。

表 27-13  
用于检查溢出的 `Value` 方法

| 名称 | 描述 |
| --- | --- |
| `OverflowFloat(val)` | 如果指定的 `float64` 值转换为调用该方法的 `Value` 的类型会导致溢出，则此方法返回 `true`。除非 `Value.Kind` 方法返回 `Float32` 或 `Float64`，否则此方法将引发 panic。 |
| `OverflowInt(val)` | 如果指定的 `int64` 值转换为调用该方法的 `Value` 的类型会导致溢出，则此方法返回 `true`。除非 `Value.Kind` 方法返回有符号整数类型之一，否则此方法将引发 panic。 |
| `OverflowUint(val)` | 如果指定的 `uint64` 值转换为调用该方法的 `Value` 的类型会导致溢出，则此方法返回 `true`。除非 `Value.Kind` 方法返回无符号整数类型之一，否则此方法将引发 panic。 |

如第 5 章所述，Go 数值在溢出时会进行回绕（wrap）。表 27-13 中描述的方法可用于判断转换是否会导致溢出，如清单 27-21 所示，这可能会产生意外结果。

```
package main
import (
    "reflect"
    //"strings"
    // "fmt"
)
func IsInt(v reflect.Value) bool {
    switch v.Kind() {
        case reflect.Int, reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64:
        return true
    }
    return false
}
func IsFloat(v reflect.Value) bool {
    switch v.Kind() {
        case reflect.Float32, reflect.Float64:
        return true
    }
    return false
}
func convert(src, target interface{}) (result interface{}, assigned bool) {
    srcVal := reflect.ValueOf(src)
    targetVal := reflect.ValueOf(target)
    if (srcVal.Type().ConvertibleTo(targetVal.Type())) {
        if (IsInt(targetVal) && IsInt(srcVal)) &&
            targetVal.OverflowInt(srcVal.Int()) {
                Printfln("Int overflow")
                return src, false
        } else if (IsFloat(targetVal) && IsFloat(srcVal) &&
            targetVal.OverflowFloat(srcVal.Float())) {
            Printfln("Float overflow")
            return src, false
        }
        result = srcVal.Convert(targetVal.Type()).Interface()
        assigned = true
    } else {
        result = src
    }
    return
}
func main() {
    name := "Alice"
    price := 279
    //city := "London"
    newVal, ok := convert(price, 100.00)
    Printfln("Converted %v: %v, %T", ok, newVal, newVal)
    newVal, ok = convert(name, 100.00)
    Printfln("Converted %v: %v, %T", ok, newVal, newVal)
    newVal, ok = convert(5000, int8(100))
    Printfln("Converted %v: %v, %T", ok, newVal, newVal)
}
清单 27-21
reflection 文件夹中 main.go 文件中的溢出预防
```

清单 27-21 中的新代码增加了在将一种整数类型转换为另一种整数类型，以及将一种浮点值转换为另一种浮点值时的溢出保护。编译并执行该项目，你将看到以下输出：

```
Converted true: 279, float64
Converted false: Alice, string
Int overflow
Converted false: 5000, int
```

清单 27-21 中对 `convert` 函数的最后一次调用尝试将值 `5000` 转换为 `int8`，这将导致溢出。`OverflowInt` 方法返回 `true`，因此不会执行转换。

## 创建新值

`reflect` 包定义了用于创建新值的函数，这些函数在表 27-14 中进行了描述。我将在后续章节中演示特定于特定数据结构（例如切片和映射）的函数。

表 27-14  
用于创建新值的函数

| 名称 | 描述 |
| --- | --- |
| `New(type)` | 此函数创建一个 `Value`，它指向指定类型的一个值，并初始化为该类型的零值。 |
| `Zero(type)` | 此函数创建一个 `Value`，它表示指定类型的零值。 |
| `MakeMap(type)` | 此函数创建一个新的映射，如第 28 章所述。 |
| `MakeMapWithSize(type, size)` | 此函数创建一个具有指定大小的新映射，如第 28 章所述。 |
| `MakeSlice(type, capacity)` | 此函数创建一个新的切片，如第 28 章所述。 |
| `MakeFunc(type, args, results)` | 此函数创建一个具有指定参数和结果的新函数，如第 29 章所述。 |
| `MakeChan(type, buffer)` | 此函数创建一个具有指定缓冲区大小的新通道，如第 29 章所述。 |

使用 `New` 函数时必须小心，因为它返回一个指向指定类型新值的指针，这意味着很容易创建指向指针的指针。清单 27-22 使用 `New` 函数在交换其参数的函数中创建一个临时值。

```
package main
import (
    "reflect"
    //"strings"
    // "fmt"
)
func swap(first interface{}, second interface{}) {
    firstValue, secondValue := reflect.ValueOf(first), reflect.ValueOf(second)
    if firstValue.Type() == secondValue.Type() &&
        firstValue.Kind() == reflect.Ptr &&
        firstValue.Elem().CanSet() && secondValue.Elem().CanSet() {
        temp :=  reflect.New(firstValue.Elem().Type())
        temp.Elem().Set(firstValue.Elem())
        firstValue.Elem().Set(secondValue.Elem())
        secondValue.Elem().Set(temp.Elem())
    }
}
func main() {
    name := "Alice"
    price := 279
    city := "London"
    swap(&name, &city)
    for _, val := range []interface{} { name, price, city }  {
        Printfln("Value: %v", val)
    }
}
清单 27-22
reflection 文件夹中 main.go 文件中的值创建
```

执行交换需要一个新值，该值通过 `New` 函数创建：

```
...
temp :=  reflect.New(firstValue.Elem().Type())
...
```

传递给 `New` 函数的 `Type` 是从某个参数值的 `Elem` 结果中获取的，这避免了创建指向指针的指针。`Set` 方法用于设置临时值并执行交换。编译并执行该项目，你将收到以下输出，显示 `name` 和 `city` 变量的值已被交换：

```
Value: London
Value: 279
Value: Alice
```

## 本章小结

在本章中，我介绍了 Go 反射的基本特性并演示了它们的用法。我解释了如何获取反射的类型和值、如何确定被反射的类型种类、如何设置反射值，以及如何使用 `reflect` 包提供的便捷功能。在下一章中，我将继续描述反射，向你展示如何处理指针、切片、映射和结构体。



