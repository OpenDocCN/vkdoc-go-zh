# 20. 读写数据

在本章中，我将描述标准库中最重要的两个接口：`Reader` 接口和 `Writer` 接口。这些接口无论在哪里读写数据都会用到，这意味着任何数据的源或目标都可以用大致相同的方式处理。例如，将数据写入文件与将数据写入网络连接是完全一样的。表 20-1 介绍了本章描述的特性所处的上下文。

**表 20-1** Reader 和 Writer 的上下文

| 问题 | 答案 |
| --- | --- |
| 它们是什么？ | 这些接口定义了读写数据所需的基本方法。 |
| 它们有什么用？ | 这种方法意味着几乎任何数据源都可以用相同的方式使用，同时仍然允许使用第 13 章中描述的组合特性来定义专门的特性。 |
| 如何使用它们？ | `io` 包定义了这些接口，但其实现在许多其他包中可用，其中一些将在后续章节中详细描述。 |
| 是否有任何陷阱或限制？ | 这些接口并不能完全隐藏数据源或目标的具体细节，并且通常需要额外的由基于 `Reader` 和 `Writer` 构建的接口提供的方法。 |
| 是否有替代方案？ | 使用这些接口是可选的，但很难避免，因为它们在标准库中广泛使用。 |

**表 20-2** 章节总结

| 问题 | 解决方案 | 代码清单 |
| --- | --- | --- |
| 读取数据 | 使用 `Reader` 接口的实现 | 6 |
| 写入数据 | 使用 `Writer` 接口的实现 | 7 |
| 简化读写数据的过程 | 使用工具函数 | 8 |
| 组合读取器或写入器 | 使用专门的实现 | 9–16 |
| 缓冲读取和写入 | 使用 `bufio` 包提供的特性 | 17–23 |
| 使用 reader 和 writer 扫描和格式化数据 | 使用接受 `Reader` 或 `Writer` 参数的 `fmt` 包中的函数 | 24–27 |



## 本章准备

要开始本章的学习，请打开一个新的命令提示符，导航到一个方便的位置，并创建一个名为 `readersandwriters` 的目录。运行代码清单 20-1 所示的命令来创建一个模块文件。

**提示**  
你可以从 [`https://github.com/apress/pro-go`](https://github.com/apress/pro-go) 下载本章以及本书所有其他章节的示例项目。如果在运行示例时遇到问题，请参阅第 2 章了解如何获取帮助。

```
go mod init readersandwriters
代码清单 20-1
初始化模块
```

在 `readersandwriters` 文件夹中添加一个名为 `printer.go` 的文件，内容如代码清单 20-2 所示。

```
package main
import "fmt"
func Printfln(template string, values ...interface{}) {
fmt.Printf(template + "\n", values...)
}
代码清单 20-2
readersandwriters 文件夹中 printer.go 文件的内容
```

在 `readersandwriters` 文件夹中添加一个名为 `product.go` 的文件，内容如代码清单 20-3 所示。

```
package main
type Product struct {
Name, Category string
Price float64
}
var Kayak = Product {
Name: "Kayak",
Category: "Watersports",
Price: 279,
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
代码清单 20-3
readersandwriters 文件夹中 product.go 文件的内容
```

在 `readersandwriters` 文件夹中添加一个名为 `main.go` 的文件，内容如代码清单 20-4 所示。

```
package main
func main() {
Printfln("Product: %v, Price : %v", Kayak.Name, Kayak.Price)
}
代码清单 20-4
readersandwriters 文件夹中 main.go 文件的内容
```

在命令提示符中，进入 `readersandwriters` 文件夹并运行代码清单 20-5 所示的命令。

```
go run .
代码清单 20-5
运行示例项目
```

代码将被编译并执行，产生以下输出：

```
Product: Kayak Price: 275
```

### 理解读取器与写入器

`Reader` 和 `Writer` 接口由 `io` 包定义，提供了读写数据的抽象方式，而不与数据的来源或去向绑定。在接下来的章节中，我将介绍这些接口并演示它们的用法。

## 理解读取器

`Reader` 接口定义了一个单一方法，如表 20-3 所述。

**表 20-3**  
**Reader 接口**

| 名称 | 描述 |
| --- | --- |
| `Read(byteSlice)` | 此方法将数据读入指定的 `[]byte`。该方法返回已读取的字节数（以 `int` 类型表示）和一个 `error`。 |

`Reader` 接口不包含关于数据来自何处或如何获取的任何细节——它仅定义了 `Read` 方法。具体细节由实现该接口的类型决定，标准库中有针对不同数据源的读取器实现。其中最简单的读取器之一使用 `string` 作为数据源，如代码清单 20-6 所示。

```
package main
import (
"io"
"strings"
)
func processData(reader io.Reader) {
b := make([]byte, 2)
for {
count, err := reader.Read(b);
if (count > 0) {
Printfln("读取了 %v 字节: %v", count, string(b[0:count]))
}
if err == io.EOF {
break
}
}
}
func main() {
r := strings.NewReader("Kayak")
processData(r)
}
代码清单 20-6
在 readersandwriters 文件夹的 main.go 文件中使用读取器
```

每种类型的 `Reader` 的创建方式都不同，我将在本章及后续章节中演示。要创建一个基于 `string` 的读取器，`strings` 包提供了 `NewReader` 构造函数，它接受一个 `string` 作为参数：

```
...
r := strings.NewReader("Kayak")
...
```

为了强调接口的使用，我将 `NewReader` 函数的结果作为参数传递给一个接受 `io.Reader` 的函数。在函数内部，我使用 `Read` 方法来读取字节数据。我通过设置传递给 `Read` 函数的 `byte` 切片的大小来指定想要接收的最大字节数。`Read` 函数的返回值指示已读取了多少字节数据以及是否发生了错误。

`io` 包定义了一个名为 `EOF` 的特殊错误，用于指示 `Reader` 已到达数据末尾。如果 `Read` 函数返回的 `error` 结果等于 `EOF` 错误，则我会跳出正在从 `Reader` 读取数据的 `for` 循环：

```
...
if err == io.EOF {
break
}
...
```

其效果是，`for` 循环每次调用 `Read` 函数最多获取两个字节并将其写出。当到达字符串末尾时，`Read` 函数返回 `EOF` 错误，导致 `for` 循环终止。编译并执行代码，你将看到以下输出：

```
读取了 2 字节: Ka
读取了 2 字节: ya
读取了 1 字节: k
```

## 理解写入器

`Writer` 接口定义了表 20-4 所述的方法。

**表 20-4**  
**Writer 接口**

| 名称 | 描述 |
| --- | --- |
| `Write(byteSlice)` | 此方法写入指定 `byte` 切片中的数据。该方法返回已写入的字节数和一个 `error`。如果写入的字节数小于切片的长度，则 `error` 将不为 `nil`。 |

`Writer` 接口不包含关于写入数据如何存储、传输或处理的任何细节，这些都由实现该接口的类型决定。在代码清单 20-7 中，我创建了一个 `Writer`，它根据接收到的数据构建一个字符串。

```
package main
import (
"io"
"strings"
)
func processData(reader io.Reader, writer io.Writer) {
b := make([]byte, 2)
for {
count, err := reader.Read(b);
if (count > 0) {
writer.Write(b[0:count])
Printfln("读取了 %v 字节: %v", count, string(b[0:count]))
}
if err == io.EOF {
break
}
}
}
func main() {
r := strings.NewReader("Kayak")
var builder strings.Builder
processData(r, &builder)
Printfln("字符串构建器内容: %s", builder.String())
}
代码清单 20-7
在 readersandwriters 文件夹的 main.go 文件中使用写入器
```

我在第 16 章中描述的 `strings.Builder` 结构体实现了 `io.Writer` 接口，这意味着我可以将字节写入 `Builder`，然后调用其 `String` 方法从这些字节创建字符串。

如果写入器无法写入切片中的所有数据，它将返回一个 `error`。在代码清单 20-7 中，我检查了错误结果，如果返回错误，则跳出 `for` 循环。然而，由于此示例中的 `Writer` 是在内存中构建字符串，因此几乎不会发生错误。

请注意，我使用了地址运算符将指向 `Builder` 的指针传递给 `processData` 函数，如下所示：

```
...
processData(r, &builder)
...
```

通常，`Reader` 和 `Writer` 的方法是为指针实现的，这样将 `Reader` 或 `Writer` 传递给函数时不会创建副本。在代码清单 20-7 中，我不必对 `Reader` 使用地址运算符，因为 `strings.NewReader` 函数返回的结果本身就是一个指针。

编译并执行项目，你将看到以下输出，表明字节从一个字符串中读取并用于创建另一个字符串：

```
读取了 2 字节: Ka
读取了 2 字节: ya
读取了 1 字节: k
字符串构建器内容: Kayak
```




### 为读写器使用便捷函数

`io`包包含一组函数，它们提供了读写数据的额外方式，如表 20-5 所述。

**表 20-5** `io`包中用于读写数据的函数

| 名称 | 描述 |
| --- | --- |
| `Copy(w, r)` | 此函数将数据从`Reader`复制到`Writer`，直到返回`EOF`或遇到其他错误。返回值是复制的字节数和一个用于描述问题的`error`。 |
| `CopyBuffer(w, r, buffer)` | 此函数执行与`Copy`相同的任务，但在将数据传递到`Writer`之前，会先将数据读入指定的`buffer`。 |
| `CopyN(w, r, count)` | 此函数将`count`个字节从`Reader`复制到`Writer`。返回值是复制的字节数和一个用于描述问题的`error`。 |
| `ReadAll(r)` | 此函数从指定的`Reader`读取数据，直到到达`EOF`。返回值是一个包含已读取数据的`byte`切片和一个用于描述问题的`error`。 |
| `ReadAtLeast(r, byteSlice, min)` | 此函数从读取器读取至少指定数量的字节，并将它们放入`byte`切片中。如果读取的字节少于指定数量，则会报告错误。 |
| `ReadFull(r, byteSlice)` | 此函数用数据填充指定的`byte`切片。返回值是读取的`bytes`数和一个`error`。如果在读取足够填充切片的字节之前遇到`EOF`，则会报告错误。 |
| `WriteString(w, str)` | 此函数将指定的`string`写入写入器。 |

表 20-5 中的函数使用了由`Reader`和`Writer`接口定义的`Read`和`Write`方法，但方式更为便捷，避免了在需要处理数据时定义`for`循环。在清单 20-8 中，我使用了`Copy`函数将示例`string`中的字节从`Reader`复制到`Writer`。

```
package main
import (
"io"
"strings"
)
func processData(reader io.Reader, writer io.Writer) {
count, err := io.Copy(writer, reader)
if (err == nil) {
Printfln("Read %v bytes", count)
} else {
Printfln("Error: %v", err.Error())
}
}
func main() {
r := strings.NewReader("Kayak")
var builder strings.Builder
processData(r, &builder)
Printfln("String builder contents: %s", builder.String())
}
Listing 20-8
Copying Data in the main.go File in the readersandwriters Folder
```

使用`Copy`函数实现了与之前示例相同的结果，但更简洁。编译并执行代码，您将收到以下输出：

```
Read 5 bytes
String builder contents: Kayak
```

## 使用专门的读写器

除了基本的`Reader`和`Writer`接口之外，`io`包还提供了一些专门的实现，如表 20-6 所述，并在随后的章节中进行了演示。

**表 20-6** 用于专用读写器的`io`包函数

| 名称 | 描述 |
| --- | --- |
| `Pipe()` | 此函数返回一个`PipeReader`和一个`PipeWriter`，可用于连接需要`Reader`和`Writer`的函数，如“使用管道”部分所述。 |
| `MultiReader(...readers)` | 此函数定义了一个可变参数，允许指定任意数量的`Reader`值。结果是一个`Reader`，它会按其参数定义的顺序传递来自每个参数的内容，如“连接多个读取器”部分所述。 |
| `MultiWriter(...writers)` | 此函数定义了一个可变参数，允许指定任意数量的`Writer`值。结果是一个`Writer`，它会将相同的数据发送到所有指定的写入器，如“组合多个写入器”部分所述。 |
| `LimitReader(r, limit)` | 此函数创建一个`Reader`，它在读取了指定数量的字节后将返回`EOF`，如“限制读取数据”部分所述。 |




### 使用管道

管道用于连接通过 `Reader` 消费数据的代码和通过 `Writer` 产生数据的代码。在 `readersandwriters` 文件夹中添加一个名为 `data.go` 的文件，内容如代码清单 20-9 所示。

```go
package main
import "io"
func GenerateData(writer io.Writer) {
data := []byte("Kayak, Lifejacket")
writeSize := 4
for i := 0; i < len(data); i += writeSize {
end := i + writeSize
if (end > len(data)) {
end = len(data)
}
count, err := writer.Write(data[i: end])
Printfln("Wrote %v byte(s): %v", count, string(data[i: end]))
if (err != nil)  {
Printfln("Error: %v", err.Error())
}
}
}
func ConsumeData(reader io.Reader) {
data := make([]byte, 0, 10)
slice := make([]byte, 2)
for {
count, err := reader.Read(slice)
if (count > 0) {
Printfln("Read data: %v", string(slice[0:count]))
data = append(data, slice[0:count]...)
}
if (err == io.EOF) {
break
}
}
Printfln("Read data: %v", string(data))
}
```

*代码清单 20-9：readersandwriters 文件夹中 data.go 文件的内容*

`GenerateData` 函数定义了一个 `Writer` 参数，它使用该参数从一个字符串写入字节。`ConsumeData` 函数定义了一个 `Reader` 参数，它使用该参数读取字节数据，然后将这些数据用于创建一个字符串。

实际项目中无需从一个字符串读取字节再创建另一个字符串，但这样做能很好地展示管道的工作原理，如代码清单 20-10 所示。

```go
package main
import (
"io"
//"strings"
)
// func processData(reader io.Reader, writer io.Writer) {
//     count, err := io.Copy(writer, reader)
//     if (err == nil) {
//         Printfln("Read %v bytes", count)
//     } else {
//         Printfln("Error: %v", err.Error())
//     }
// }
func main() {
pipeReader, pipeWriter := io.Pipe()
go func() {
GenerateData(pipeWriter)
pipeWriter.Close()
}()
ConsumeData(pipeReader)
}
```

*代码清单 20-10：readersandwriters 文件夹中 main.go 文件里使用管道*

`io.Pipe` 函数返回一个 `PipeReader` 和一个 `PipeWriter`。`PipeReader` 和 `PipeWriter` 结构体实现了 `Closer` 接口，该接口定义了表 20-7 所示的方法。

**表 20-7：Closer 方法**

| 名称 | 描述 |
| --- | --- |
| `Close()` | 此方法关闭读取器或写入器。具体细节取决于实现，但通常来说，后续对已关闭 `Reader` 的任何读取都将返回零字节和 `EOF` 错误，而对已关闭 `Writer` 的任何后续写入都将返回一个错误。 |

由于 `PipeWriter` 实现了 `Writer` 接口，我可以将其作为参数传递给 `GenerateData` 函数，然后在函数完成后调用 `Close` 方法，以便读取器能够接收到 `EOF`，如下所示：

```go
...
GenerateData(pipeWriter)
pipeWriter.Close()
...
```

管道是同步的，这意味着 `PipeWriter.Write` 方法会阻塞，直到数据从管道中被读取。因此，`PipeWriter` 需要在与读取器不同的 goroutine 中使用，以防止应用程序死锁：

```go
...
go func() {
GenerateData(pipeWriter)
pipeWriter.Close()
}()
...
```

注意此语句末尾的括号。为匿名函数创建 goroutine 时需要这些括号，但很容易忘记添加。

`PipeReader` 结构体实现了 `Reader` 接口，这意味着我可以将其作为参数传递给 `ConsumeData` 函数。`ConsumeData` 函数在 `main` goroutine 中执行，这意味着在该函数完成之前，应用程序不会退出。

其效果是，数据通过 `PipeWriter` 写入管道，并通过 `PipeReader` 从管道读取。当 `GenerateData` 函数完成后，会在 `PipeWriter` 上调用 `Close` 方法，这将导致 `PipeReader` 的下一次读取产生 `EOF`。编译并执行该项目，你将看到以下输出：

```
Read data: Ka
Wrote 4 byte(s): Kaya
Read data: ya
Read data: k,
Wrote 4 byte(s): k, L
Read data:  L
Read data: if
Wrote 4 byte(s): ifej
Read data: ej
Read data: ac
Wrote 4 byte(s): acke
Read data: ke
Wrote 1 byte(s): t
Read data: t
Read data: Kayak, Lifejacket
```

输出结果突显了管道是同步的这一事实。`GenerateData` 函数调用写入器的 `Write` 方法，然后阻塞，直到数据被读取。这就是为什么输出中的第一条消息来自读取器：读取器每次消费两个字节，这意味着在用于发送四个字节的 `Write` 方法的初始调用完成之前，需要两次读取操作，然后才会显示来自 `GenerateData` 函数的消息。

#### 改进示例

在代码清单 20-10 中，我在执行 `GenerateData` 函数的 goroutine 中调用了 `PipeWriter` 上的 `Close` 方法。这确实可行，但我更倾向于在生成数据的代码中检查 `Writer` 是否实现了 `Closer` 接口，如代码清单 20-11 所示。

```go
...
func GenerateData(writer io.Writer) {
data := []byte("Kayak, Lifejacket")
writeSize := 4
for i := 0; i < len(data); i += writeSize {
end := i + writeSize
if (end > len(data)) {
end = len(data)
}
count, err := writer.Write(data[i: end])
Printfln("Wrote %v byte(s): %v", count, string(data[i: end]))
if (err != nil)  {
Printfln("Error: %v", err.Error())
}
}
if closer, ok := writer.(io.Closer); ok {
closer.Close()
}
}
...
```

*代码清单 20-11：readersandwriters 文件夹中 data.go 文件里关闭一个写入器*

这种方法为定义了 `Close` 方法的 `Writer` 提供了一致的处理方式，其中包含后续章节中描述的一些最常用的类型。它还使我能够更改 goroutine，以便无需匿名函数即可执行 `GenerateData` 函数，如代码清单 20-12 所示。

```go
package main
import (
"io"
//"strings"
)
func main() {
pipeReader, pipeWriter := io.Pipe()
go GenerateData(pipeWriter)
ConsumeData(pipeReader)
}
```

*代码清单 20-12：readersandwriters 文件夹中 main.go 文件里简化代码*

此示例产生的输出与代码清单 20-10 中的代码相同。

### 连接多个读取器

`MultiReader` 函数将多个读取器的输入串联起来，以便可以按顺序处理它们，如代码清单 20-13 所示。

```go
package main
import (
"io"
"strings"
)
func main() {
r1 := strings.NewReader("Kayak")
r2 := strings.NewReader("Lifejacket")
r3 := strings.NewReader("Canoe")
concatReader := io.MultiReader(r1, r2, r3)
ConsumeData(concatReader)
}
```

*代码清单 20-13：readersandwriters 文件夹中 main.go 文件里连接读取器*

由 `MultiReader` 函数返回的 `Reader` 在响应 `Read` 方法时，会从底层某个 `Reader` 值中获取内容。当第一个 `Reader` 返回 `EOF` 时，则会从第二个 `Reader` 读取内容。此过程一直持续，直到最后一个底层 `Reader` 返回 `EOF`。编译并执行代码，你将看到以下输出：

```
Read data: Ka
Read data: ya
Read data: k
Read data: Li
Read data: fe
Read data: ja
Read data: ck
Read data: et
Read data: Ca
Read data: no
Read data: e
Read data: KayakLifejacketCanoe
```



### 组合多个写入器

`MultiWriter` 函数用于组合多个写入器，使数据同时发送至所有写入器，如代码清单 20-14 所示。

```
package main
import (
"io"
"strings"
)
func main() {
var w1 strings.Builder
var w2 strings.Builder
var w3 strings.Builder
combinedWriter := io.MultiWriter(&w1, &w2, &w3)
GenerateData(combinedWriter)
Printfln("写入器 #1: %v", w1.String())
Printfln("写入器 #2: %v", w2.String())
Printfln("写入器 #3: %v", w3.String())
}
代码清单 20-14
在 readersandwriters 文件夹的 main.go 文件中组合写入器
```

此示例中的写入器是 `string.Builder` 类型的值，该类在第 16 章中已有描述，并实现了 `Writer` 接口。`MultiWriter` 函数用于创建一个 `Writer`，调用其 `Write` 方法会导致同一数据被写入这三个独立的写入器。编译并执行项目，你将看到如下输出：

```
写入 4 字节: Kaya
写入 4 字节: k, L
写入 4 字节: ifej
写入 4 字节: acke
写入 1 字节: t
写入器 #1: Kayak, Lifejacket
写入器 #2: Kayak, Lifejacket
写入器 #3: Kayak, Lifejacket
```

### 将读取数据回显到写入器

`TeeReader` 函数返回一个 `Reader`，该读取器会将接收到的数据回显至一个 `Writer`，如代码清单 20-15 所示。

```
package main
import (
"io"
"strings"
)
func main() {
r1 := strings.NewReader("Kayak")
r2 := strings.NewReader("Lifejacket")
r3 := strings.NewReader("Canoe")
concatReader := io.MultiReader(r1, r2, r3)
var writer strings.Builder
teeReader := io.TeeReader(concatReader, &writer);
ConsumeData(teeReader)
Printfln("回显数据: %v", writer.String())
}
代码清单 20-15
在 readersandwriters 文件夹的 main.go 文件中回显数据
```

`TeeReader` 函数用于创建一个 `Reader`，该读取器会将数据回显至一个 `strings.Builder`（该类在第 16 章中已有描述，并实现了 `Writer` 接口）。编译并执行项目，你将看到如下输出，其中包含回显的数据：

```
读取数据: Ka
读取数据: ya
读取数据: k
读取数据: Li
读取数据: fe
读取数据: ja
读取数据: ck
读取数据: et
读取数据: Ca
读取数据: no
读取数据: e
读取数据: KayakLifejacketCanoe
回显数据: KayakLifejacketCanoe
```

### 限制读取数据量

`LimitReader` 函数用于限制从 `Reader` 中获取的数据量，如代码清单 20-16 所示。

```
package main
import (
"io"
"strings"
)
func main() {
r1 := strings.NewReader("Kayak")
r2 := strings.NewReader("Lifejacket")
r3 := strings.NewReader("Canoe")
concatReader := io.MultiReader(r1, r2, r3)
limited := io.LimitReader(concatReader, 5)
ConsumeData(limited)
}
代码清单 20-16
在 readersandwriters 文件夹的 main.go 文件中限制数据量
```

`LimitReader` 函数的第一个参数是提供数据的 `Reader`，第二个参数是可读取的最大字节数。当达到限制时，`LimitReader` 函数返回的 `Reader` 将发送 `EOF`——除非底层读取器先发送 `EOF`。在代码清单 20-16 中，我将限制设置为 5 字节，编译并执行项目后，将产生如下输出：

```
读取数据: Ka
读取数据: ya
读取数据: k
读取数据: Kayak
```

## 数据缓冲

`bufio` 包提供了为读取器和写入器添加缓冲的功能。要了解无缓冲时数据是如何处理的，请在 `readersandwriters` 文件夹中添加一个名为 `custom.go` 的文件，其内容如代码清单 20-17 所示。

```
package main
import "io"
type CustomReader struct {
reader io.Reader
readCount int
}
func NewCustomReader(reader io.Reader) *CustomReader {
return &CustomReader { reader, 0 }
}
func (cr *CustomReader) Read(slice []byte) (count int, err error) {
count, err = cr.reader.Read(slice)
cr.readCount++
Printfln("自定义读取器: %v 字节", count)
if (err == io.EOF) {
Printfln("总读取次数: %v", cr.readCount)
}
return
}
代码清单 20-17
readersandwriters 文件夹中 custom.go 文件的内容
```

代码清单 20-17 中的代码定义了一个名为 `CustomReader` 的结构体类型，它作为 `Reader` 的包装器。`Read` 方法的实现会生成输出，报告读取了多少数据以及总共执行了多少次读取操作。代码清单 20-18 将新类型用作基于字符串的 `Reader` 的包装器。

```
package main
import (
"io"
"strings"
)
func main() {
text := "It was a boat. A small boat."
var reader io.Reader = NewCustomReader(strings.NewReader(text))
var writer strings.Builder
slice := make([]byte, 5)
for {
count, err := reader.Read(slice)
if (count > 0) {
writer.Write(slice[0:count])
}
if (err != nil) {
break
}
}
Printfln("读取数据: %v", writer.String())
}
代码清单 20-18
在 readersandwriters 文件夹的 main.go 文件中使用读取器包装器
```

`NewCustomReader` 函数用于创建一个 `CustomReader`，它从字符串中读取数据，并使用 `for` 循环通过 `byte` 切片来消费数据。编译并执行项目，你将看到数据是如何被读取的：

```
自定义读取器: 5 字节
自定义读取器: 5 字节
自定义读取器: 5 字节
自定义读取器: 5 字节
自定义读取器: 5 字节
自定义读取器: 3 字节
自定义读取器: 0 字节
总读取次数: 7
读取数据: It was a boat. A small boat.
```

传递给 `Read` 函数的 `byte` 切片的大小决定了数据如何被消费。在本例中，切片大小为 5，这意味着每次调用 `Read` 函数最多读取 5 字节。有两次读取未获得 5 字节数据。倒数第二次读取产生了 3 字节，因为源数据不能被 5 整除，剩余了 3 字节数据。最后一次读取返回了 0 字节，但收到了 `EOF` 错误，表明已到达数据末尾。

总共读取 28 字节需要 7 次读取。（我选择的源数据中所有字符都只需单字节表示；但如果修改示例，引入需要多字节表示的字符，你可能会看到不同的读取次数。）

当每次操作的开销较大时，少量读取数据可能会成为问题。对于内存中存储的字符串，这不是问题，但从文件等其他数据源读取数据时，开销可能更大，因此最好进行次数较少但每次读取量更大的操作。为此，可以引入一个缓冲区，将大量数据读入其中，以满足多次较小的数据请求。表 20-8 描述了 `bufio` 包提供的用于创建带缓冲读取器的函数。

表 20-8

用于创建带缓冲读取器的 `bufio` 函数

| 名称 | 描述 |
| --- | --- |
| `NewReader(r)` | 此函数返回一个具有默认缓冲区大小（撰写本文时为 4,096 字节）的带缓冲 `Reader`。 |
| `NewReaderSize(r, size)` | 此函数返回一个具有指定缓冲区大小的带缓冲 `Reader`。 |



`NewReader`和`NewReaderSize`产生的结果实现了`Reader`接口，但引入了缓冲区，这可以减少对底层数据源执行的读取操作次数。清单 20-19 演示了在示例中引入缓冲区。

```go
package main
import (
"io"
"strings"
"bufio"
)
func main() {
text := "It was a boat. A small boat."
var reader io.Reader = NewCustomReader(strings.NewReader(text))
var writer strings.Builder
slice := make([]byte, 5)
reader = bufio.NewReader(reader)
for {
count, err := reader.Read(slice)
if (count > 0) {
writer.Write(slice[0:count])
}
if (err != nil) {
break
}
}
Printfln("Read data: %v", writer.String())
}
```
清单 20-19：在 `readersandwriters` 文件夹的 `main.go` 文件中使用缓冲区

我使用了`NewReader`函数，它会创建一个具有默认缓冲区大小的`Reader`。带缓冲的`Reader`会填充其缓冲区，并使用其中包含的数据来响应`Read`方法的调用。编译并执行项目，查看引入缓冲区的效果：

```
Custom Reader: 28 bytes
Custom Reader: 0 bytes
Total Reads: 2
Read data: It was a boat. A small boat.
```

默认缓冲区大小为 4,096 字节，这意味着带缓冲的读取器能够通过一次读取操作读取所有数据，外加一次额外的读取来产生`EOF`结果。引入缓冲区可以减少与读取操作相关的开销，但代价是消耗了用于缓冲数据的内存。

### 使用额外的带缓冲读取器方法

`NewReader`和`NewReaderSize`函数返回`bufio.Reader`值，这些值实现了`io.Reader`接口，并且可以作为其他类型`Reader`方法的即插即用包装器，无缝地引入读取缓冲区。

`bufio.Reader`结构体定义了直接使用缓冲区的额外方法，如表 20-9 所述。

表 20-9：带缓冲读取器定义的方法

| 名称 | 描述 |
| --- | --- |
| `Buffered()` | 此方法返回一个`int`，表示可以从缓冲区读取的字节数。 |
| `Discard(count)` | 此方法丢弃指定数量的字节。 |
| `Peek(count)` | 此方法返回指定数量的字节，而不将其从缓冲区中移除，这意味着它们将在后续对`Read`方法的调用中被返回。 |
| `Reset(reader)` | 此方法丢弃缓冲区中的数据，并从指定的`Reader`执行后续读取。 |
| `Size()` | 此方法返回缓冲区的大小，以`int`表示。 |

清单 20-20 展示了使用`Size`和`Buffered`方法来报告缓冲区的大小及其包含的数据量。

```go
package main
import (
"io"
"strings"
"bufio"
)
func main() {
text := "It was a boat. A small boat."
var reader io.Reader = NewCustomReader(strings.NewReader(text))
var writer strings.Builder
slice := make([]byte, 5)
buffered := bufio.NewReader(reader)
for {
count, err := buffered.Read(slice)
if (count > 0) {
Printfln("Buffer size: %v, buffered: %v",
buffered.Size(), buffered.Buffered())
writer.Write(slice[0:count])
}
if (err != nil) {
break
}
}
Printfln("Read data: %v", writer.String())
}
```
清单 20-20：在 `readersandwriters` 文件夹的 `main.go` 文件中操作缓冲区

编译并执行项目，你会看到每次读取操作都会消耗部分缓冲数据：

```
Custom Reader: 28 bytes
Buffer size: 4096, buffered: 23
Buffer size: 4096, buffered: 18
Buffer size: 4096, buffered: 13
Buffer size: 4096, buffered: 8
Buffer size: 4096, buffered: 3
Buffer size: 4096, buffered: 0
Custom Reader: 0 bytes
Total Reads: 2
Read data: It was a boat. A small boat.
```

### 执行带缓冲写入

`bufio`包也提供了创建使用缓冲区的写入器的支持，使用的函数如表 20-10 所述。

表 20-10：用于创建带缓冲写入器的 `bufio` 函数

| 名称 | 描述 |
| --- | --- |
| `NewWriter(w)` | 此函数返回一个具有默认缓冲区大小（撰写本文时为 4,096 字节）的带缓冲`Writer`。 |
| `NewWriterSize(w, size)` | 此函数返回一个具有指定缓冲区大小的带缓冲`Writer`。 |

表 20-10 中描述的函数的产生结果实现了`Writer`接口，这意味着它们可以无缝地引入写入缓冲区。这些函数返回的具体数据类型是`bufio.Writer`，它定义了表 20-11 中描述的用于管理缓冲区及其内容的方法。

表 20-11：`bufio.Writer` 结构体定义的方法

| 名称 | 描述 |
| --- | --- |
| `Available()` | 此方法返回缓冲区中的可用字节数。 |
| `Buffered()` | 此方法返回已写入缓冲区的字节数。 |
| `Flush()` | 此方法将缓冲区的内容写入到底层`Writer`。 |
| `Reset(writer)` | 此方法丢弃缓冲区中的数据，并将后续写入操作定向到指定的`Writer`。 |
| `Size()` | 此方法返回缓冲区的容量（以字节为单位）。 |

清单 20-21 定义了一个自定义`Writer`，它会报告其操作，并展示缓冲区的效果。这是上一节中创建的`Reader`的对应部分。

```go
package main
import "io"
// ...为了简洁起见，省略了读取器类型和函数...
type CustomWriter struct {
writer io.Writer
writeCount int
}
func NewCustomWriter(writer io.Writer) * CustomWriter {
return &CustomWriter{ writer, 0}
}
func (cw *CustomWriter) Write(slice []byte) (count int, err error) {
count, err = cw.writer.Write(slice)
cw.writeCount++
Printfln("Custom Writer: %v bytes", count)
return
}
func (cw *CustomWriter) Close() (err error) {
if closer, ok := cw.writer.(io.Closer); ok {
closer.Close()
}
Printfln("Total Writes: %v", cw.writeCount)
return
}
```
清单 20-21：在 `readersandwriters` 文件夹的 `custom.go` 文件中定义自定义写入器

`NewCustomWriter`构造函数使用`CustomWriter`结构体包装了一个`Writer`，该结构体会报告其写入操作。清单 20-22 演示了不带缓冲执行写入操作的方式。

```go
package main
import (
//"io"
"strings"
//"bufio"
)
func main() {
text := "It was a boat. A small boat."
var builder strings.Builder
var writer = NewCustomWriter(&builder)
for i := 0; true; {
end := i + 5
if (end >= len(text)) {
writer.Write([]byte(text[i:]))
break
}
writer.Write([]byte(text[i:end]))
i = end
}
Printfln("Written data: %v", builder.String())
}
```
清单 20-22：在 `readersandwriters` 文件夹的 `main.go` 文件中执行无缓冲写入

该示例每次向`Writer`写入五个字节，该`Writer`由`strings`包中的`Builder`支持。编译并执行项目，你可以看到每次调用`Write`方法的效果：

```
Custom Writer: 5 bytes
Custom Writer: 5 bytes
Custom Writer: 5 bytes
Custom Writer: 5 bytes
Custom Writer: 5 bytes
Custom Writer: 3 bytes
Written data: It was a boat. A small boat.
```

带缓冲的`Writer`将数据保留在缓冲区中，并且仅当缓冲区已满或调用`Flush`方法时，才将其传递到底层`Writer`。清单 20-23 在示例中引入了缓冲区。



```
package main
import (
//"io"
"strings"
"bufio"
)
func main() {
text := "It was a boat. A small boat."
var builder strings.Builder
var writer = bufio.NewWriterSize(NewCustomWriter(&builder), 20)
for i := 0; true; {
end := i + 5
if (end >= len(text)) {
writer.Write([]byte(text[i:]))
writer.Flush()
break
}
writer.Write([]byte(text[i:end]))
i = end
}
Printfln("Written data: %v", builder.String())
}
Listing 20-23
Using a Buffered Writer in the main.go File in the readersandwriters Folder
```

使用缓冲的 `Writer` 并非完全无缝衔接，因为调用 `Flush` 方法确保所有数据被写出至关重要。我在清单 20-23 中选择的缓冲区大小为 20 字节，远小于默认缓冲区（在实际项目中太小而无法发挥作用），但它非常适合演示引入缓冲区如何减少示例中的写操作次数。编译并执行该项目，你将看到以下输出：

```
Custom Writer: 20 bytes
Custom Writer: 8 bytes
Written data: It was a boat. A small boat.
```

## 使用读写器进行格式化和扫描

在第十七章中，我描述了 `fmt` 包提供的格式化和扫描功能，并演示了它们如何与字符串一起使用。正如该章所述，`fmt` 包提供了将这些功能应用于 `Reader` 和 `Writer` 的支持，具体内容如下节所述。我还描述了如何使用 `strings` 包中的功能与 `Writer` 配合使用。

### 从读取器中扫描值

`fmt` 包提供了从 `Reader` 中扫描值并将其转换为不同类型的函数，如清单 20-24 所示。（使用函数扫描值并非必需，此处仅为强调扫描过程适用于任何 `Reader`。）

```
package main
import (
"io"
"strings"
//"bufio"
"fmt"
)
func scanFromReader(reader io.Reader, template string,
vals ...interface{}) (int, error) {
return fmt.Fscanf(reader, template, vals...)
}
func main() {
reader := strings.NewReader("Kayak Watersports $279.00")
var name, category string
var price float64
scanTemplate := "%s %s $%f"
_, err := scanFromReader(reader, scanTemplate, &name, &category, &price)
if (err != nil) {
Printfln("Error: %v", err.Error())
} else {
Printfln("Name: %v", name)
Printfln("Category: %v", category)
Printfln("Price: %.2f", price)
}
}
Listing 20-24
Scanning from a Reader in the main.go File in the readersandwriters Folder
```

扫描过程从 `Reader` 中读取字节，并使用扫描模板解析接收到的数据。清单 20-24 中的扫描模板包含两个字符串和一个 `float64` 值，编译并执行代码将产生以下输出：

```
Name: Kayak
Category: Watersports
Price: 279.00
```

使用 `Reader` 时的一个实用技巧是通过循环逐步扫描数据，如清单 20-25 所示。这种方法在字节随时间到达时效果很好，例如从 HTTP 连接读取时（我将在第二十五章中描述）。

```
package main
import (
"io"
"strings"
//"bufio"
"fmt"
)
func scanFromReader(reader io.Reader, template string,
vals ...interface{}) (int, error) {
return fmt.Fscanf(reader, template, vals...)
}
func scanSingle(reader io.Reader, val interface{}) (int, error) {
return fmt.Fscan(reader, val)
}
func main() {
reader := strings.NewReader("Kayak Watersports $279.00")
for {
var str string
_, err := scanSingle(reader, &str)
if (err != nil) {
if (err != io.EOF) {
Printfln("Error: %v", err.Error())
}
break
}
Printfln("Value: %v", str)
}
}
Listing 20-25
Scanning Gradually in the main.go File in the readersandwriters Folder
```

`for` 循环调用 `scanSingle` 函数，该函数使用 `Fscan` 从 `Reader` 中读取一个 `string`。值会一直被读取到返回 `EOF` 为止，此时循环终止。编译并执行该项目，你将得到以下输出：

```
Value: Kayak
Value: Watersports
Value: $279.00
```

### 将格式化字符串写入写入器

`fmt` 包还提供了将格式化字符串写入 `Writer` 的函数，如清单 20-26 所示。（使用函数格式化字符串并非必需，此处仅为强调格式化适用于任何 `Reader`。）

```
package main
import (
"io"
"strings"
//"bufio"
"fmt"
)
// func scanFromReader(reader io.Reader, template string,
//         vals ...interface{}) (int, error) {
//     return fmt.Fscanf(reader, template, vals...)
// }
// func scanSingle(reader io.Reader, val interface{}) (int, error) {
//     return fmt.Fscan(reader, val)
// }
func writeFormatted(writer io.Writer, template string, vals ...interface{}) {
fmt.Fprintf(writer, template, vals...)
}
func main() {
var writer strings.Builder
template := "Name: %s, Category: %s, Price: $%.2f"
writeFormatted(&writer, template, "Kayak", "Watersports", float64(279))
fmt.Println(writer.String())
}
Listing 20-26
Writing a Formatted String in the main.go File in the readersandwriters Folder
```

`writeFormatted` 函数使用 `fmt.Fprintf` 函数将按模板格式化的字符串写入 `Writer`。编译并执行该项目，你将看到以下输出：

```
Name: Kayak, Category: Watersports, Price: $279.00
```

### 将替换器与写入器配合使用

`strings.Replacer` 结构体可用于对 `string` 执行替换，并将修改后的结果输出到 `Writer`，如清单 20-27 所示。

```
package main
import (
"io"
"strings"
//"bufio"
"fmt"
)
func writeReplaced(writer io.Writer, str string, subs ...string) {
replacer := strings.NewReplacer(subs...)
replacer.WriteString(writer, str)
}
func main() {
text := "It was a boat. A small boat."
subs := []string { "boat", "kayak", "small", "huge" }
var writer strings.Builder
writeReplaced(&writer, text, subs...)
fmt.Println(writer.String())
}
Listing 20-27
Using a Replacer in the main.go File in the readersandwriters Folder
```

`WriteString` 方法执行替换并写出修改后的字符串。编译并执行代码，你将得到以下输出：

```
It was a kayak. A huge kayak.
```

## 总结

在本章中，我介绍了 `Reader` 和 `Writer` 接口，它们在整个标准库中用于数据的读取和写入。我描述了这些接口定义的方法，解释了可用的专用实现的用法，并向你展示了如何实现缓冲、格式化和扫描。下一章，我将介绍处理 JSON 数据的支持，其中会用到本章描述的功能。



