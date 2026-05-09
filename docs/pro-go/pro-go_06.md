# 16. 字符串处理与正则表达式

在本章中，我将介绍用于处理 `string` 值的标准库特性，这些特性几乎是每个项目都需要的，并且许多语言将其作为内建类型的方法提供。但是，尽管 Go 在标准库中定义了这些特性，但仍提供了一整套函数，以及对正则表达式工作的良好支持。表 16-1 将这些特性置于上下文中。

**表 16-1** -  将字符串处理与正则表达式置于上下文中

| 问题 | 答案 |
| --- | --- |
| 它们是什么？ | 字符串处理包括从修剪空白到将字符串拆分为组件的广泛操作。正则表达式是允许简洁定义字符串匹配规则的模式。 |
| 为什么它们有用？ | 当应用程序需要处理 `string` 值时，这些操作非常有用。一个常见的例子是处理 HTTP 请求。 |
| 如何使用它们？ | 这些特性包含在属于标准库的 `strings` 和 `regexp` 包中。 |
| 是否有任何陷阱或限制？ | 某些操作的执行方式存在一些特性，但它们大多符合预期行为。 |
| 是否有任何替代方案？ | 使用这些包是可选的，并非必须使用。话虽如此，自己实现这些特性几乎没有意义，因为标准库编写良好且经过了彻底测试。 |

**表 16-2** - 本章小结

| 问题 | 解决方案 | 代码清单 |
| --- | --- | --- |
| 比较字符串 | 使用 `strings` 包中的 `Contains`、`EqualFold` 或 `Has*` 函数 | 4 |
| 转换字符串大小写 | 使用 `strings` 包中的 `ToLower`、`ToUpper`、`Title` 或 `ToTitle` 函数 | 5, 6 |
| 检查或更改字符大小写 | 使用 `unicode` 包提供的函数 | 7 |
| 在字符串中查找内容 | 使用 `strings` 或 `regexp` 包提供的函数 | 8, 9, 24–27, 29–32 |
| 拆分字符串 | 使用 `strings` 和 `regexp` 包中的 `Fields` 或 `Split*` 函数 | 10–14, 28 |
| 连接字符串 | 使用 `strings` 包中的 `Join` 或 `Repeat` 函数 | 22 |
| 从字符串修剪字符 | 使用 `strings` 包中的 `Trim*` 函数 | 15–18 |
| 执行替换 | 使用 `strings` 包中的 `Replace*` 或 `Map` 函数，使用 `Replacer`，或使用 `regexp` 包中的 `Replace*` 函数 | 19–21, 33 |
| 高效构建字符串 | 使用 `strings` 包中的 `Builder` 类型 | 23 |



## 本章准备

为准备本章内容，请打开一个新的命令提示符窗口，导航至一个便捷位置，并创建一个名为 `stringsandregexp` 的目录。运行清单 16-1 所示的命令来创建一个模块文件。

**提示**

你可以从 [`https://github.com/apress/pro-go`](https://github.com/apress/pro-go) 下载本章以及本书所有其他章节的示例项目。如果在运行示例时遇到问题，请参阅第 2 章了解如何获取帮助。

```
go mod init stringsandregexp
清单 16-1
初始化模块
```

在 `stringsandregexp` 文件夹中添加一个名为 `main.go` 的文件，内容如清单 16-2 所示。

```
package main
import (
"fmt"
)
func main() {
product := "Kayak"
fmt.Println("Product:", product)
}
清单 16-2
stringsandregexp 文件夹中 main.go 文件的内容
```

使用命令提示符在 `stringsandregexp` 文件夹中运行清单 16-3 所示的命令。

```
go run .
清单 16-3
运行示例项目
```

代码将被编译并执行，产生如下输出：

```
Product: Kayak
```

### 处理字符串

`strings` 包提供了一组用于处理字符串的函数。在接下来的小节中，我将介绍 `strings` 包中最有用的功能，并演示其用法。

## 比较字符串

`strings` 包提供了比较函数，如表 16-3 所述。这些函数可以作为相等运算符（`==` 和 `!=`）的补充使用。

**表 16-3** `strings` 包的字符串比较函数

| 函数 | 描述 |
| --- | --- |
| `Contains(s, substr)` | 如果字符串 `s` 包含 `substr`，则返回 `true`；否则返回 `false`。 |
| `ContainsAny(s, substr)` | 如果字符串 `s` 包含字符串 `substr` 中的任意字符，则返回 `true`。 |
| `ContainsRune(s, rune)` | 如果字符串 `s` 包含特定的 `rune`，则返回 `true`。 |
| `EqualFold(s1, s2)` | 该函数执行不区分大小写的比较，如果字符串 `s1` 和 `s2` 相同，则返回 `true`。 |
| `HasPrefix(s, prefix)` | 如果字符串 `s` 以字符串 `prefix` 开头，则返回 `true`。 |
| `HasSuffix(s, suffix)` | 如果字符串 `s` 以字符串 `suffix` 结尾，则返回 `true`。 |

清单 16-4 演示了表 16-3 中所述函数的用法。

```
package main
import (
"fmt"
"strings"
)
func main() {
product := "Kayak"
fmt.Println("Contains:", strings.Contains(product, "yak"))
fmt.Println("ContainsAny:", strings.ContainsAny(product, "abc"))
fmt.Println("ContainsRune:", strings.ContainsRune(product, 'K'))
fmt.Println("EqualFold:", strings.EqualFold(product, "KAYAK"))
fmt.Println("HasPrefix:", strings.HasPrefix(product, "Ka"))
fmt.Println("HasSuffix:", strings.HasSuffix(product, "yak"))
}
清单 16-4
在 stringsandregexp 文件夹的 main.go 文件中比较字符串
```

表 16-3 中的函数执行区分大小写的比较，但 `EqualFold` 函数除外。（折叠是 Unicode 处理字符大小写的方式，其中字符可以有不同的小写、大写和首字母大写表示形式。）清单 16-4 中的代码执行后将产生如下输出：

```
Contains: true
ContainsAny: true
ContainsRune: true
HasPrefix: true
HasSuffix: true
EqualFold: true
```

## 使用面向字节的函数

对于 `strings` 包中所有操作字符的函数，`bytes` 包中都有一个对应的函数来操作 `byte` 切片，如下所示：

```
package main
import (
"fmt"
"strings"
"bytes"
)
func main() {
price := "€100"
fmt.Println("Strings Prefix:", strings.HasPrefix(price, "€"))
fmt.Println("Bytes Prefix:", bytes.HasPrefix([]byte(price),
[]byte { 226, 130 }))
}
```

此示例展示了两个包提供的 `HasPrefix` 函数的用法。`strings` 版本对字符进行操作，无论字符占用多少个字节，都会检查前缀。这让我能够判断 `price` 字符串是否以欧元货币符号开头。`bytes` 版本的函数让我能够判断 `price` 变量是否以特定的字节序列开头，无论这些字节与字符的关系如何。在本章中，我将使用 `strings` 包中的函数，因为它们是最常用的。在第 25 章中，我将使用 `bytes.Buffer` 结构，这是一种在内存中存储二进制数据的有用方法。



### 转换字符串大小写

`strings` 包提供了表 16-4 中描述的用于转换字符串大小写的函数。

**表 16-4** `strings` 包中的大小写函数

| 函数 | 描述 |
| --- | --- |
| `ToLower(str)` | 此函数返回一个新字符串，其中包含将指定字符串中的字符映射为小写形式的结果。 |
| `ToUpper(str)` | 此函数返回一个新字符串，其中包含将指定字符串中的字符映射为大写形式的结果。 |
| `Title(str)` | 此函数转换指定字符串，使每个单词的首字符大写，其余字符小写。 |
| `ToTitle(str)` | 此函数返回一个新字符串，其中包含将指定字符串中的字符映射为首字母大写形式的结果。 |

必须小心使用 `Title` 和 `ToTitle` 函数，它们的工作方式可能与你预期的不同。`Title` 函数返回一个适合用作标题的字符串，但它对所有单词一视同仁，如代码清单 16-5 所示。

```
package main
import (
"fmt"
"strings"
)
func main() {
description := "A boat for sailing"
fmt.Println("Original:", description)
fmt.Println("Title:", strings.Title(description))
}
```

按照惯例，标题大小写不会大写冠词、短介词和连词，这意味着转换这个字符串：

```
A boat for sailing
```

按惯例应转换如下：

```
A Boat for Sailing
```

单词 *for* 并未大写，但其他单词都大写了。然而这些规则很复杂，存在多种解释，且因语言而异，因此 Go 采取了一种更简单的方法，即大写所有单词。编译并运行代码清单 16-5 中的代码可以看到效果，其输出如下：

```
Original: A boat for sailing
Title: A Boat For Sailing
```

在某些语言中，有些字符在标题中使用时其外观会发生变化。Unicode 为每个字符定义了三种状态——小写、大写和标题大小写——而 `ToTitle` 函数返回一个仅包含标题大小写字符的字符串。对于英语，其效果与 `ToUpper` 函数相同，但在其他语言中可能会产生不同的结果，如代码清单 16-6 所示。

```
package main
import (
"fmt"
"strings"
)
func main() {
specialChar := "\u01c9"
fmt.Println("Original:", specialChar, []byte(specialChar))
upperChar := strings.ToUpper(specialChar)
fmt.Println("Upper:", upperChar, []byte(upperChar))
titleChar := strings.ToTitle(specialChar)
fmt.Println("Title:", titleChar, []byte(titleChar))
}
```

我有限的语言技能尚未掌握需要不同标题大小写的语言，因此我使用了一个 Unicode 转义序列来选择字符。（我是从 Unicode 规范中获得该字符代码的。）编译并执行代码清单 16-6 中的代码后，会输出该字符的小写、大写和标题大小写版本，以及用于表示它的字节：

```
Original: ǉ [199 137]
Upper: Ǉ [199 135]
Title: ǈ [199 136]
```

你可能能看到字符外观上的差异，但即使看不到，也能看到大写和标题大小写使用了不同的字节值组合。

**本地化：要么不做，要么做好**

产品本地化需要时间、精力和资源——并且需要由了解目标国家或地区语言、文化和货币惯例的人来完成。如果没有正确本地化，其结果可能比根本不本地化更糟糕。

正是由于这个原因，我在本书（以及我的任何一本书中）都不会详细描述本地化特性。在脱离使用情境的情况下描述特性，感觉就像是在为读者埋下自酿苦果的陷阱。至少，如果产品没有本地化，用户知道自己的处境，不必试图弄清楚你只是忘了更改货币代码，还是那些价格实际上就是美元。（这是我在英国生活时经常看到的问题。）

你*应该*本地化你的产品。你的用户*应该*能够以他们理解的方式进行交易或执行其他操作。但你*必须*认真对待，并投入所需的时间和精力来正确地完成它。

### 处理字符大小写

`unicode` 包提供了可用于判断或转换单个字符大小写的函数，如表 16-5 所述。

**表 16-5** `unicode` 包中用于字符大小写的函数

| 函数 | 描述 |
| --- | --- |
| `IsLower(rune)` | 如果指定的 rune 是小写，则此函数返回 `true`。 |
| `ToLower(rune)` | 此函数返回与指定 rune 关联的小写 rune。 |
| `IsUpper(rune)` | 如果指定的 rune 是大写，则此函数返回 `true`。 |
| `ToUpper(rune)` | 此函数返回与指定 rune 关联的大写 rune。 |
| `IsTitle(rune)` | 如果指定的 rune 是标题大小写，则此函数返回 `true`。 |
| `ToTitle(rune)` | 此函数返回与指定 rune 关联的标题大小写 rune。 |

代码清单 16-7 使用了表 16-5 中描述的函数来检查和转换一个 rune 的大小写。

```
package main
import (
"fmt"
//"strings"
"unicode"
)
func main() {
product := "Kayak"
for _, char := range product {
fmt.Println(string(char), "Upper case:", unicode.IsUpper(char))
}
}
```

代码清单 16-7 中的代码枚举了 `product` 字符串中的字符，以判断它们是否为大写。编译并执行此代码会产生以下输出：

```
K Upper case: true
a Upper case: false
y Upper case: false
a Upper case: false
k Upper case: false
```



### 检查字符串

表 16-6 中的函数由 `strings` 包提供，用于检查字符串。

**表 16-6**  
`strings` 包中用于检查字符串的函数

| 函数 | 描述 |
| --- | --- |
| `Count(s, sub)` | 该函数返回一个 `int` 值，报告指定子串在字符串 `s` 中出现的次数。 |
| `Index(s, sub)`, `LastIndex(s, sub)` | 这些函数返回指定子串在字符串 `s` 中第一次或最后一次出现的索引，如果未出现则返回 `-1`。 |
| `IndexAny(s, chars)`, `LastIndexAny(s, chars)` | 这些函数返回指定字符串中任意字符在字符串 `s` 中第一次或最后一次出现的索引，如果未出现则返回 `-1`。 |
| `IndexByte(s, b)`, `LastIndexByte(s, b)` | 这些函数返回指定 `byte` 在字符串 `s` 中第一次或最后一次出现的索引，如果未出现则返回 `-1`。 |
| `IndexFunc(s, func)`, `LastIndexFunc(s, func)` | 这些函数返回字符串 `s` 中使指定函数返回 `true` 的字符第一次或最后一次出现的索引，详见“使用自定义函数检查字符串”部分。 |

清单 16-8 演示了表 16-6 中描述的函数，其中一些通常用于根据内容对字符串进行切片。

```
package main
import (
"fmt"
"strings"
//"unicode"
)
func main() {
description := "A boat for one person"
fmt.Println("Count:", strings.Count(description, "o"))
fmt.Println("Index:", strings.Index(description, "o"))
fmt.Println("LastIndex:", strings.LastIndex(description, "o"))
fmt.Println("IndexAny:", strings.IndexAny(description, "abcd"))
fmt.Println("LastIndex:", strings.LastIndex(description, "o"))
fmt.Println("LastIndexAny:", strings.LastIndexAny(description, "abcd"))
}
```

*清单 16-8：在 `stringsandregexp` 文件夹的 `main.go` 文件中检查字符串*

这些函数执行的比较是区分大小写的，这意味着清单 16-8 中用于测试的字符串包含 `person`，但不包含 `Person`。要进行不区分大小写的比较，请将表 16-6 中描述的函数与表 16-4 和 16-5 中的函数结合使用。编译并执行清单 16-8 中的代码，将产生以下输出：

```
Count: 4
Index: 3
LastIndex: 19
IndexAny: 2
LastIndex: 19
LastIndexAny: 4
```

#### 使用自定义函数检查字符串

`IndexFunc` 和 `LastIndexFunc` 函数使用自定义函数来检查字符串，如清单 16-9 所示。

```
package main
import (
"fmt"
"strings"
)
func main() {
description := "A boat for one person"
isLetterB := func (r rune) bool {
return r == 'B' || r == 'b'
}
fmt.Println("IndexFunc:", strings.IndexFunc(description, isLetterB))
}
```

*清单 16-9：在 `stringsandregexp` 文件夹的 `main.go` 文件中使用自定义函数检查字符串*

自定义函数接收一个 `rune` 并返回一个 `bool` 结果，指示该字符是否满足所需条件。`IndexFunc` 函数会为字符串中的每个字符调用自定义函数，直到获得 `true` 结果，此时返回该索引。

`isLetterB` 变量被赋予一个自定义函数，该函数接收一个 `rune`，如果该 rune 是大写或小写的 `B`，则返回 `true`。该自定义函数被传递给 `strings.IndexFunc` 函数，在编译并执行代码时产生以下输出：

```
IndexFunc: 2
```

### 操作字符串

`strings` 包提供了用于编辑字符串的有用函数，包括替换部分或全部字符或删除空格的功能。

#### 分割字符串

第一组函数用于分割字符串，如表 16-7 所述。（还有一个使用正则表达式分割字符串的有用功能，在本章后面的“使用正则表达式”部分中介绍。）

**表 16-7**  
`strings` 包中用于分割字符串的函数

| 函数 | 描述 |
| --- | --- |
| `Fields(s)` | 该函数根据空白字符分割字符串，并返回一个包含字符串 `s` 中非空白部分的切片。 |
| `FieldsFunc(s, func)` | 该函数根据自定义函数返回 `true` 的字符分割字符串 `s`，并返回包含字符串剩余部分的切片。 |
| `Split(s, sub)` | 该函数在指定子串的每次出现处分割字符串 `s`，返回一个 `string` 切片。如果分隔符是空字符串，则切片将包含每个字符的字符串。 |
| `SplitN(s, sub, max)` | 此函数与 `Split` 类似，但接受一个额外的 `int` 参数，指定要返回的最大子串数量。结果切片中的最后一个子串将包含源字符串未分割的部分。 |
| `SplitAfter(s, sub)` | 此函数与 `Split` 类似，但会将用于分割的子串包含在结果中。详见表格后的演示。 |
| `SplitAfterN(s, sub, max)` | 此函数与 `SplitAfter` 类似，但接受一个额外的 `int` 参数，指定要返回的最大子串数量。 |

表 16-7 中描述的函数执行相同的基本任务。`Split` 和 `SplitAfter` 函数之间的区别在于，`Split` 函数不会将用于分割的子串包含在结果中，如清单 16-10 所示。

```
package main
import (
"fmt"
"strings"
)
func main() {
description := "A boat for one person"
splits := strings.Split(description, " ")
for _, x := range splits {
fmt.Println("Split >>" + x + ">" + x + "<<")
}
}
```

*清单 16-10：在 `stringsandregexp` 文件夹的 `main.go` 文件中分割字符串*

为了突出显示区别，清单 16-10 中的代码使用 `Split` 和 `SplitAfter` 函数分割了同一个字符串。两个函数的结果都使用 `for` 循环进行枚举，循环写入的消息将结果括在尖括号中，结果前后没有空格。编译并执行代码，您将看到以下结果：

```
Split >>A>boat>for>one>person>A >boat >for >one >person<<
```

这些字符串是根据空格字符进行分割的。结果所示，空格字符不包含在 `Split` 函数产生的结果中，但包含在 `SplitAfter` 函数产生的结果中。

#### 限制结果数量

`SplitN` 和 `SplitAfterN` 函数接受一个 `int` 参数，用于指定结果中应包含的最大结果数量，如清单 16-11 所示。

```
package main
import (
"fmt"
"strings"
)
func main() {
description := "A boat for one person"
splits := strings.SplitN(description, " ", 3)
for _, x := range splits {
fmt.Println("Split >>" + x + ">" + x + "<<")
}
}
```

*清单 16-11：在 `stringsandregexp` 文件夹的 `main.go` 文件中限制结果数量*

如果字符串可以分割成的子串数量超过指定数量，则结果切片中的最后一个元素将是字符串未分割的剩余部分。清单 16-11 指定了最多三个结果，这意味着切片中的前两个元素将正常分割，第三个元素将是字符串的剩余部分。编译并执行代码，您将看到以下输出：

```
Split >>A>boat>for one person<<
```



#### 按空白字符分割

`Split`、`SplitN`、`SplitAfter` 和 `SplitAfterN` 函数的一个局限在于它们无法处理重复字符序列，这在按空白字符分割字符串时可能成为问题，如清单 16-12 所示。

```
package main
import (
"fmt"
"strings"
)
func main() {
description := "This  is  double  spaced"
splits := strings.SplitN(description, " ", 3)
for _, x := range splits {
fmt.Println("Split >>" + x + "<<")
}
}
```

源字符串中的单词是双空格间隔的，但 `SplitN` 函数仅根据第一个空格字符进行分割，产生了奇怪的结果。编译并执行代码，你将看到以下输出：

```
Split >>This>>is  double  spaced<<
```

结果切片中的第二个元素是一个空格字符。为了处理重复的空白字符，`Fields` 函数可根据任意空白字符来分割字符串，如清单 16-13 所示。

```
package main
import (
"fmt"
"strings"
)
func main() {
description := "This  is  double  spaced"
splits := strings.Fields(description)
for _, x := range splits {
fmt.Println("Field >>" + x + "<<")
}
}
```

`Fields` 函数不支持限制结果数量，但能正确处理双空格。编译并执行该项目，你将看到以下输出：

```
Field >>This>is>double>spaced<<
```

#### 使用自定义函数分割字符串

`FieldsFunc` 函数通过将每个字符传递给自定义函数，并在该函数返回 `true` 时进行分割，如清单 16-14 所示。

```
package main
import (
"fmt"
"strings"
)
func main() {
description := "This  is  double  spaced"
splitter := func(r rune) bool {
return r == ' '
}
splits := strings.FieldsFunc(description, splitter)
for _, x := range splits {
fmt.Println("Field >>" + x + "<<")
}
}
```

自定义函数接收一个 `rune` 类型参数，如果该字符应导致字符串被分割，则返回 `true`。`FieldsFunc` 函数足够智能，能够处理重复字符，例如清单 16-14 中的双空格。

注意

在清单 16-14 中，我指定了空格字符，以强调 `FieldsFunc` 函数能够处理重复字符。`Fields` 函数有更优的方法，即根据 `unicode` 包中 `IsSpace` 函数返回 `true` 的任意字符进行分割。

编译并执行该项目，你将看到以下输出：

```
Field >>This>is>double>spaced<<
```

### 修剪字符串

修剪过程会移除字符串开头和结尾的字符，最常用于去除空白字符。表 16-8 描述了 `strings` 包提供的用于修剪的函数。

**表 16-8** strings 包中用于修剪字符串的函数

| 函数 | 描述 |
| --- | --- |
| `TrimSpace(s)` | 该函数返回字符串 `s`，并移除开头或结尾的空白字符。 |
| `Trim(s, set)` | 该函数返回从字符串 `s` 中移除了包含在字符串 `set` 中的任何开头或结尾字符后的结果。 |
| `TrimLeft(s, set)` | 该函数返回字符串 `s`，并移除任何包含在字符串 `set` 中的开头字符。此函数匹配指定的任一字符——若要移除完整的子串，请使用 `TrimPrefix` 函数。 |
| `TrimRight(s, set)` | 该函数返回字符串 `s`，并移除任何包含在字符串 `set` 中的结尾字符。此函数匹配指定的任一字符——若要移除完整的子串，请使用 `TrimSuffix` 函数。 |
| `TrimPrefix(s, prefix)` | 该函数返回移除指定前缀字符串后的字符串 `s`。此函数会移除完整的 `prefix` 字符串——若要移除字符集中的字符，请使用 `TrimLeft` 函数。 |
| `TrimSuffix(s, suffix)` | 该函数返回移除指定后缀字符串后的字符串 `s`。此函数会移除完整的 `suffix` 字符串——若要移除字符集中的字符，请使用 `TrimRight` 函数。 |
| `TrimFunc(s, func)` | 该函数返回字符串 `s`，并移除自定义函数返回 `true` 的任何开头或结尾字符。 |
| `TrimLeftFunc(s, func)` | 该函数返回字符串 `s`，并移除自定义函数返回 `true` 的任何开头字符。 |
| `TrimRightFunc(s, func)` | 该函数返回字符串 `s`，并移除自定义函数返回 `true` 的任何结尾字符。 |

#### 修剪空白字符

`TrimSpace` 函数执行最常见的修剪任务，即移除任何开头或结尾的空白字符。这在处理用户输入时尤其有用，例如输入用户名时，用户可能无意中引入空格，若不加以清除则会造成混淆，如清单 16-15 所示。

```
package main
import (
"fmt"
"strings"
)
func main() {
username := " Alice"
trimmed := strings.TrimSpace(username)
fmt.Println("Trimmed:", ">>" + trimmed + "<<")
}
```

用户可能在输入姓名时并未意识到自己按下了空格键，通过在使用前修剪输入的姓名，可以避免用户混淆。编译并执行示例项目，你将看到修剪后的姓名显示如下：

```
Trimmed: >>Alice<<
```

#### 修剪字符集

`Trim`、`TrimLeft` 和 `TrimRight` 函数会匹配指定字符串中的任意字符。清单 16-16 展示了 `Trim` 函数的用法。其他函数的工作方式与此相同，但仅修剪字符串的开头或结尾。

```
package main
import (
"fmt"
"strings"
)
func main() {
description := "A boat for one person"
trimmed := strings.Trim(description, "Asno ")
fmt.Println("Trimmed:", trimmed)
}
```

在清单 16-16 中，调用 `Trim` 函数时，我指定了字母 `A`、`s`、`n`、`o` 和空格字符。该函数使用字符集中的任一字符进行区分大小写的匹配，并从结果中移除所有匹配的字符。一旦找到不在字符集中的字符，匹配即停止。对前缀的匹配从字符串开头进行，对后缀的匹配从字符串结尾进行。如果字符串中不包含字符集中的任何字符，则 `Trim` 函数将原封不动地返回该字符串。

对于本例，这意味着字符串开头的字母 `A` 和空格将被修剪，字符串结尾的字母 `s`、`o` 和 `n` 也将被修剪。编译并执行该项目，输出将显示修剪后的字符串：

```
Trimmed: boat for one per
```



#### 修剪子字符串

`TrimPrefix`和`TrimSuffix`函数用于修剪子字符串，而不是从字符集中修剪字符，如清单 16-17 所示。

```go
package main
import (
"fmt"
"strings"
)
func main() {
description := "A boat for one person"
prefixTrimmed := strings.TrimPrefix(description, "A boat ")
wrongPrefix := strings.TrimPrefix(description, "A hat ")
fmt.Println("Trimmed:", prefixTrimmed)
fmt.Println("Not trimmed:", wrongPrefix)
}
```

目标字符串的开头或结尾必须与指定的前缀或后缀完全匹配；否则，修剪函数的返回结果将是原始字符串。在清单 16-17 中，我使用了两次`TrimPrefix`函数，但只有一次使用了与字符串开头匹配的前缀，编译并执行代码时会产生以下结果：

```
Trimmed: for one person
Not trimmed: A boat for one person
```

#### 使用自定义函数进行修剪

`TrimFunc`、`TrimLeftFunc`和`TrimRightFunc`函数使用自定义函数修剪字符串，如清单 16-18 所示。

```go
package main
import (
"fmt"
"strings"
)
func main() {
description := "A boat for one person"
trimmer := func(r rune) bool {
return r == 'A' || r == 'n'
}
trimmed := strings.TrimFunc(description, trimmer)
fmt.Println("Trimmed:", trimmed)
}
```

自定义函数会对字符串开头和结尾的字符进行调用，并且会一直修剪字符直到函数返回`false`。编译并执行该示例，您将收到以下输出，其中字符串的第一个和最后一个字符已被修剪：

```
Trimmed:  boat for one perso
```

### 修改字符串

表 16-9 中描述的函数由`strings`包提供，用于修改字符串的内容。

**表 16-9** `strings`包中用于修改字符串的函数

| 函数 | 描述 |
| --- | --- |
| `Replace(s, old, new, n)` | 此函数通过将字符串`old`的出现替换为字符串`new`来修改字符串`s`。可替换的最大次数由`int`参数`n`指定。 |
| `ReplaceAll(s, old, new)` | 此函数通过将字符串`old`的所有出现替换为字符串`new`来修改字符串`s`。与`Replace`函数不同，它对替换次数没有限制。 |
| `Map(func, s)` | 此函数通过对字符串`s`中的每个字符调用自定义函数并连接结果来生成一个新字符串。如果函数产生负值，则当前字符被丢弃而不进行替换。 |

`Replace`和`ReplaceAll`函数用于定位子字符串并替换它们。`Replace`函数允许指定最大更改次数，而`ReplaceAll`函数将替换其找到的所有子字符串，如清单 16-19 所示。

```go
package main
import (
"fmt"
"strings"
)
func main() {
text := "It was a boat. A small boat."
replace := strings.Replace(text, "boat", "canoe", 1)
replaceAll := strings.ReplaceAll(text, "boat", "truck")
fmt.Println("Replace:", replace)
fmt.Println("Replace All:", replaceAll)
}
```

在清单 16-19 中，`Replace`函数被用来替换单词`boat`的一个实例，而`ReplaceAll`函数用来替换每一个实例。编译并执行代码，您将看到以下输出：

```
Replace: It was a canoe. A small boat.
Replace All: It was a truck. A small truck.
```

#### 使用映射函数修改字符串

`Map`函数通过对每个字符调用一个函数并将结果组合成新字符串来修改字符串，如清单 16-20 所示。

```go
package main
import (
"fmt"
"strings"
)
func main() {
text := "It was a boat. A small boat."
mapper := func(r rune) rune {
if r == 'b' {
return 'c'
}
return r
}
mapped := strings.Map(mapper, text)
fmt.Println("Mapped:", mapped)
}
```

清单 16-20 中的映射函数将字符`b`替换为字符`c`，并原封不动地传递所有其他字符。编译并执行该项目，您将看到以下结果：

```
Mapped: It was a coat. A small coat.
```

#### 使用字符串替换器

`strings`包导出了一个名为`Replacer`的结构体类型，用于替换字符串，提供了一种替代表 16-10 中所述函数的方法。清单 16-21 演示了`Replacer`的使用。

**表 16-10** `Replacer`方法

| 名称 | 描述 |
| --- | --- |
| `Replace(s)` | 此方法返回一个字符串，其中构造函数指定的所有替换都已在字符串`s`上执行。 |
| `WriteString(writer, s)` | 此方法用于执行构造函数指定的替换，并将结果写入`io.Writer`，该接口在第 20 章中有描述。 |

```go
package main
import (
"fmt"
"strings"
)
func main() {
text := "It was a boat. A small boat."
replacer := strings.NewReplacer("boat", "kayak", "small", "huge")
replaced := replacer.Replace(text)
fmt.Println("Replaced:", replaced)
}
```

名为`NewReplacer`的构造函数函数用于创建`Replacer`，它接受成对的参数来指定子字符串及其替换内容。表 16-10 描述了为`Replacer`类型定义的方法。

用于创建清单 16-21 中`Replacer`的构造函数指定了应将`boat`的实例替换为`kayak`，并将`small`的实例替换为`huge`。调用`Replace`方法来执行替换，编译并执行代码时会产生以下输出：

```
Replaced: It was a kayak. A huge kayak.
```



### 构建与生成字符串

`strings` 包提供了两个用于生成字符串的函数，以及一个结构体类型，该类型的结构体方法可用于高效地逐步构建字符串。表 16-11 描述了这些函数。

**表 16-11**  
用于生成字符串的 `strings` 函数

| 函数 | 描述 |
| --- | --- |
| `Join(slice, sep)` | 此函数将指定字符串切片中的元素进行组合，并在元素之间插入指定的分隔符字符串。 |
| `Repeat(s, count)` | 此函数通过将字符串 `s` 重复指定次数来生成一个字符串。 |

在这两个函数中，`Join` 最为有用，因为它可用于重新组合已被拆分的字符串，如清单 16-22 所示。

```
package main
import (
"fmt"
"strings"
)
func main() {
text := "It was a boat. A small boat."
elements := strings.Fields(text)
joined := strings.Join(elements, "--")
fmt.Println("Joined:", joined)
}
```
*清单 16-22*  
在 `stringsandregexp` 文件夹的 `main.go` 文件中拆分和连接字符串

此示例使用 `Fields` 函数按空白字符拆分字符串，并使用两个连字符作为分隔符来连接各元素。编译并执行该项目，你将看到以下输出：

```
Joined: It--was--a--boat.--A--small--boat.
```

### 构建字符串

`strings` 包提供了 `Builder` 类型，该类型没有导出字段，但提供了一组方法，可用于高效地逐步构建字符串，如表 16-12 所述。

**表 16-12**  
`strings.Builder` 方法

| 名称 | 描述 |
| --- | --- |
| `WriteString(s)` | 此方法将字符串 `s` 追加到正在构建的字符串中。 |
| `WriteRune(r)` | 此方法将字符 `r` 追加到正在构建的字符串中。 |
| `WriteByte(b)` | 此方法将字节 `b` 追加到正在构建的字符串中。 |
| `String()` | 此方法返回由构建器创建的字符串。 |
| `Reset()` | 此方法重置由构建器创建的字符串。 |
| `Len()` | 此方法返回用于存储构建器所创建字符串的字节数。 |
| `Cap()` | 此方法返回由构建器分配的字节数。 |
| `Grow(size)` | 此方法增加由构建器分配用于存储正在构建的字符串的字节数。 |

通用模式是创建一个 `Builder`；使用 `WriteString`、`WriteRune` 和 `WriteByte` 函数构建一个字符串；然后通过 `String` 方法获取已构建的字符串，如清单 16-23 所示。

```
package main
import (
"fmt"
"strings"
)
func main() {
text := "It was a boat. A small boat."
var builder strings.Builder
for _, sub := range strings.Fields(text) {
if (sub == "small") {
builder.WriteString("very ")
}
builder.WriteString(sub)
builder.WriteRune(' ')
}
fmt.Println("String:", builder.String())
}
```
*清单 16-23*  
在 `stringsandregexp` 文件夹的 `main.go` 文件中构建字符串

使用 `Builder` 创建字符串比在常规 `string` 值上使用连接运算符更高效，尤其是在使用 `Grow` 方法预先分配存储空间的情况下。

**注意**  
在向函数和方法传递 `Builder` 值或从中返回时，务必使用指针；否则，当 `Builder` 被复制时，效率优势将丧失。

编译并执行该项目，你将看到以下输出：

```
String: It was a boat. A very small boat.
```

## 使用正则表达式

`regexp` 包提供了对正则表达式的支持，允许在字符串中查找复杂模式。表 16-13 描述了基本的正则表达式函数。

**表 16-13**  
`regexp` 包提供的基本函数

| 函数 | 描述 |
| --- | --- |
| `Match(pattern, b)` | 此函数返回一个 `bool` 值，指示字节切片 `b` 是否匹配给定的模式。 |
| `MatchString(pattern, s)` | 此函数返回一个 `bool` 值，指示字符串 `s` 是否匹配给定的模式。 |
| `Compile(pattern)` | 此函数返回一个 `RegExp`，可用于使用指定模式执行重复模式匹配，如“编译与重用模式”部分所述。 |
| `MustCompile(pattern)` | 此函数提供与 `Compile` 相同的功能，但如果指定的模式无法编译，则会引发 panic，如第 15 章所述。 |

**注意**  
本节中使用的正则表达式执行基本匹配，但 `regexp` 包支持广泛的模式语法，该语法在 [`https://pkg.go.dev/regexp/syntax@go1.17.1`](https://pkg.go.dev/regexp/syntax%40go1.17.1) 中有描述。

`MatchString` 方法是判断字符串是否匹配正则表达式的最简单方式，如清单 16-24 所示。

```
package main
import (
"fmt"
//"strings"
"regexp"
)
func main() {
description := "A boat for one person"
match, err := regexp.MatchString("[A-z]oat", description)
if (err == nil) {
fmt.Println("Match:", match)
} else {
fmt.Println("Error:", err)
}
}
```
*清单 16-24*  
在 `stringsandregexp` 文件夹的 `main.go` 文件中使用正则表达式

`MatchString` 函数接受一个正则表达式模式以及要搜索的字符串。`MatchString` 函数返回一个 `bool` 值（如果匹配则为 `true`）和一个错误（如果匹配过程没有问题则为 `nil`）。正则表达式的错误通常出现在模式无法处理时。

清单 16-24 中使用的模式将匹配任何大写或小写字母 A–Z，后跟小写的 `oat`。该模式将匹配 `description` 字符串中的单词 *boat*，当代码编译并执行时将产生以下输出：

```
Match: true
```



### 编译与复用模式

`MatchString` 函数简单便捷，但正则表达式的全部威力需要通过 `Compile` 函数来发挥。该函数会编译一个正则表达式模式，以便可以重复使用，如代码清单 16-25 所示。

```
package main
import (
"fmt"
"regexp"
)
func main() {
pattern, compileErr := regexp.Compile("[A-z]oat")
description := "A boat for one person"
question := "Is that a goat?"
preference := "I like oats"
if (compileErr == nil) {
fmt.Println("Description:", pattern.MatchString(description))
fmt.Println("Question:", pattern.MatchString(question))
fmt.Println("Preference:", pattern.MatchString(preference))
} else {
fmt.Println("Error:", compileErr)
}
}
```

这样做效率更高，因为模式只需编译一次。`Compile` 函数的结果是 `RegExp` 类型的一个实例，该类型定义了 `MatchString` 函数。编译并执行代码清单 16-25 中的代码，将产生如下输出：

```
Description: true
Question: true
Preference: false
```

编译模式还能让你访问使用正则表达式功能的方法，其中最实用的方法在表 16-14 中描述。本章介绍的方法作用于字符串，但 `RegExp` 类型也提供了用于处理字节切片的方法，以及处理读取器的方法。读取器是 Go 语言对 I/O 支持的一部分，将在第 20 章中介绍。

**表 16-14** 有用的基本 Regexp 方法

| 函数 | 描述 |
| --- | --- |
| `MatchString(s)` | 如果字符串 `s` 与编译后的模式匹配，则此方法返回 `true`。 |
| `FindStringIndex(s)` | 此方法返回一个包含 `int` 值的切片，该切片表示编译后的模式在字符串 `s` 中最左侧匹配的位置。如果结果为 `nil`，则表示未找到匹配项。 |
| `FindAllStringIndex(s, max)` | 此方法返回一个包含 `int` 切片切片的切片，这些切片表示编译后的模式在字符串 `s` 中所有匹配的位置。如果结果为 `nil`，则表示未找到匹配项。 |
| `FindString(s)` | 此方法返回一个字符串，其中包含编译后的模式在字符串 `s` 中最左侧的匹配。如果没有匹配项，则返回空字符串。 |
| `FindAllString(s, max)` | 此方法返回一个字符串切片，其中包含编译后的模式在字符串 `s` 中的所有匹配。整数参数 `max` 指定最大匹配数，`-1` 表示不限制数量。如果没有匹配项，则返回 `nil`。 |
| `Split(s, max)` | 此方法使用编译后的模式匹配结果作为分隔符来拆分字符串 `s`，并返回一个包含拆分后子字符串的切片。 |

`MatchString` 方法是表 16-3 中所述函数的一个替代方案，用于确认一个字符串是否与某个模式匹配。

`FindStringIndex` 和 `FindAllStringIndex` 方法提供匹配项的索引位置，然后可以结合数组/切片范围表示法来提取字符串的子区域，如代码清单 16-26 所示。（范围表示法已在第 7 章中介绍。）

```
package main
import (
"fmt"
"regexp"
)
func getSubstring(s string, indices []int) string {
return string(s[indices[0]:indices[1]])
}
func main() {
pattern := regexp.MustCompile("K[a-z]{4}|[A-z]oat")
description := "Kayak. A boat for one person."
firstIndex := pattern.FindStringIndex(description)
allIndices := pattern.FindAllStringIndex(description, -1)
fmt.Println("First index", firstIndex[0], "-", firstIndex[1],
"=", getSubstring(description, firstIndex))
for i, idx := range allIndices {
fmt.Println("Index", i, "=", idx[0], "-",
idx[1], "=", getSubstring(description, idx))
}
}
```

代码清单 16-26 中的正则表达式将对 `description` 字符串进行两次匹配。`FindStringIndex` 方法只返回从左到右的第一个匹配。匹配结果以一个 `int` 切片表示，其中第一个值表示匹配在字符串中的起始位置，第二个数字表示匹配的字符数。

`FindAllStringIndex` 方法返回多个匹配，在代码清单 16-26 中是使用参数 `-1` 调用的，表示应返回所有匹配。匹配结果在一个 `int` 切片的切片中返回（即结果切片中的每个值都是一个 `int` 值的切片），每个切片描述一个单独的匹配。在代码清单 16-26 中，这些索引被用于通过一个名为 `getSubstring` 的函数从字符串中提取子区域，编译并执行后将产生如下结果：

```
First index 0 - 5 = Kayak
Index 0 = 0 - 5 = Kayak
Index 1 = 9 - 13 = boat
```

如果你不需要知道匹配的位置，那么 `FindString` 和 `FindAllString` 方法会更实用，因为它们的结果直接是与正则表达式匹配的子字符串，如代码清单 16-27 所示。

```
package main
import (
"fmt"
"regexp"
)
// func getSubstring(s string, indices []int) string {
//     return string(s[indices[0]:indices[1]])
// }
func main() {
pattern := regexp.MustCompile("K[a-z]{4}|[A-z]oat")
description := "Kayak. A boat for one person."
firstMatch := pattern.FindString(description)
allMatches := pattern.FindAllString(description, -1)
fmt.Println("First match:", firstMatch)
for i, m := range allMatches {
fmt.Println("Match", i, "=", m)
}
}
```

编译并执行该项目，你将看到以下输出：

```
First match: Kayak
Match 0 = Kayak
Match 1 = boat
```

### 使用正则表达式拆分字符串

`Split` 方法使用正则表达式的匹配结果来拆分字符串，这为本章前面介绍的拆分函数提供了一种更灵活的替代方案，如代码清单 16-28 所示。

```
package main
import (
"fmt"
"regexp"
)
func main() {
pattern := regexp.MustCompile(" |boat|one")
description := "Kayak. A boat for one person."
split := pattern.Split(description, -1)
for _, s := range split {
if s != "" {
fmt.Println("Substring:", s)
}
}
}
```

本例中的正则表达式匹配空格字符或字符串 `boat` 和 `one`。每当表达式匹配时，`description` 字符串就会被拆分。`Split` 方法有一个奇特之处，即它会在匹配点周围的结果中引入空字符串，这就是我在示例中从结果切片里过滤掉这些值的原因。编译并执行代码，你将看到以下结果：

```
Substring: Kayak.
Substring: A
Substring: for
Substring: person.
```



### 使用子表达式

子表达式允许访问正则表达式的部分内容，这可以更容易地从匹配区域中提取子字符串。`清单 16-29` 提供了一个子表达式何时有用的示例。

```
package main
import (
"fmt"
"regexp"
)
func main() {
pattern := regexp.MustCompile("A [A-z]* for [A-z]* person")
description := "Kayak. A boat for one person."
str := pattern.FindString(description)
fmt.Println("Match:", str)
}
```

此示例中的模式匹配一个特定的句子结构，这使我能够匹配字符串中感兴趣的部分。但是，句子结构的大部分是静态的，而模式中的两个可变部分包含了我想要的内容。在这种情况下，`FindString` 方法是一种生硬的工具，因为它匹配整个模式，包括静态部分。编译并执行代码，您将看到以下输出：

```
Match: A boat for one person
```

我可以添加子表达式来识别模式中重要的内容区域，如 `清单 16-30` 所示。

```
package main
import (
"fmt"
"regexp"
)
func main() {
pattern := regexp.MustCompile("A ([A-z]*) for ([A-z]*) person")
description := "Kayak. A boat for one person."
subs := pattern.FindStringSubmatch(description)
for _, s := range subs {
fmt.Println("Match:", s)
}
}
```

子表达式用括号表示。在 `清单 16-30` 中，我定义了两个子表达式，每个子表达式都包围模式中的一个可变部分。`FindStringSubmatch` 方法执行与 `FindString` 相同的任务，但其结果中还包含由这些表达式匹配的子字符串。编译并执行代码，您将看到以下输出：

```
Match: A boat for one person
Match: boat
Match: one
```

`表 16-15` 描述了用于处理子表达式的 `RegExp` 方法。

**表 16-15 子表达式相关的 Regexp 方法**

| 名称 | 描述 |
| --- | --- |
| `FindStringSubmatch(s)` | 此方法返回一个切片，其中包含模式匹配到的第一个结果以及模式定义的子表达式对应的文本。 |
| `FindAllStringSubmatch(s, max)` | 此方法返回一个切片，其中包含所有匹配结果及子表达式对应的文本。`int` 参数用于指定最大匹配次数。值为 `-1` 表示匹配所有。 |
| `FindStringSubmatchIndex(s)` | 此方法等同于 `FindStringSubmatch`，但返回的是索引而非子字符串。 |
| `FindAllStringSubmatchIndex(s, max)` | 此方法等同于 `FindAllStringSubmatch`，但返回的是索引而非子字符串。 |
| `NumSubexp()` | 此方法返回子表达式的数量。 |
| `SubexpIndex(name)` | 此方法返回具有指定名称的子表达式的索引，如果没有该子表达式则返回 `-1`。 |
| `SubexpNames()` | 此方法返回子表达式的名称，按定义的顺序排列。 |

#### 使用命名子表达式

子表达式可以被赋予名称，这会使正则表达式更难理解，但使结果更易于处理。`清单 16-31` 展示了命名子表达式的用法。

```
package main
import (
"fmt"
"regexp"
)
func main() {
pattern := regexp.MustCompile(
"A (?P<type>[A-z]*) for (?P<capacity>[A-z]*) person")
description := "Kayak. A boat for one person."
subs := pattern.FindStringSubmatch(description)
for _, name := range []string { "type", "capacity" } {
fmt.Println(name, "=", subs[pattern.SubexpIndex(name)])
}
}
```

为子表达式分配名称的语法有些别扭：在括号内，使用一个问号，后跟一个大写字母 *P*，再后跟尖括号内的名称。`清单 16-31` 中的模式定义了两个命名子表达式：

```
...
pattern := regexp.MustCompile("A (?P<type>[A-z]*) for (?P<capacity>[A-z]*) person")
...
```

这两个子表达式分别被命名为 `type` 和 `capacity`。`SubexpIndex` 方法返回命名子表达式在结果中的位置，这使我能够获取由 `type` 和 `capacity` 子表达式匹配的子字符串。编译并执行该示例，您将看到以下输出：

```
type = boat
capacity = one
```

### 使用正则表达式替换子字符串

最后一组 `RegExp` 方法用于替换正则表达式匹配到的子字符串，如 `表 16-16` 所述。

**表 16-16 替换子字符串相关的 Regexp 方法**

| 名称 | 描述 |
| --- | --- |
| `ReplaceAllString(s, template)` | 此方法使用指定的模板替换字符串 `s` 中匹配的部分，该模板在包含到结果中之前会进行扩展以纳入子表达式。 |
| `ReplaceAllLiteralString(s, sub)` | 此方法使用指定的内容替换字符串 `s` 中匹配的部分，该内容会直接包含到结果中，而不会为子表达式进行扩展。 |
| `ReplaceAllStringFunc(s, func)` | 此方法使用指定函数产生的结果替换字符串 `s` 中匹配的部分。 |

`ReplaceAllString` 方法用于将正则表达式匹配到的字符串部分替换为一个模板，该模板可以引用子表达式，如 `清单 16-32` 所示。

```
package main
import (
"fmt"
"regexp"
)
func main() {
pattern := regexp.MustCompile(
"A (?P<type>[A-z]*) for (?P<capacity>[A-z]*) person")
description := "Kayak. A boat for one person."
template := "(type: ${type}, capacity: ${capacity})"
replaced := pattern.ReplaceAllString(description, template)
fmt.Println(replaced)
}
```

`ReplaceAllString` 方法返回一个包含替换后内容的字符串。模板可以通过名称（如 `${type}`）或位置（如 `${1}`）来引用子表达式的匹配结果。在清单中，`description` 字符串中被模式匹配到的部分将被替换为一个模板，该模板包含 `type` 和 `capacity` 子表达式的匹配结果。编译并执行代码，您将看到以下输出：

```
Kayak. (type: boat, capacity: one).
```

请注意，在 `清单 16-32` 中，模板只负责 `ReplaceAllString` 方法结果的一部分。`description` 字符串的前面部分——单词 `Kayak`，后跟一个句点和空格——没有被正则表达式匹配到，因此未做修改就直接包含在结果中。

**提示：** 如果你想要替换内容，并且不希望新的子字符串被解释为子表达式，请使用 `ReplaceAllLiteralString` 方法。



#### 用函数替换匹配的内容

`ReplaceAllStringFunc` 方法会用函数生成的内容替换字符串中匹配的部分，如代码清单 16-33 所示。

```go
package main
import (
"fmt"
"regexp"
)
func main() {
pattern := regexp.MustCompile(
"A (?P[A-z]*) for (?P[A-z]*) person")
description := "Kayak. A boat for one person."
replaced := pattern.ReplaceAllStringFunc(description, func(s string) string {
return "This is the replacement content"
})
fmt.Println(replaced)
}
```

代码清单 16-33 - 在 `stringsandregexp` 文件夹的 `main.go` 文件中使用函数替换内容

函数返回的结果不会针对子表达式引用进行处理，你可以在代码编译并执行时产生的输出中看到这一点：

```
Kayak. This is the replacement content.
```

## 本章总结

在本章中，我介绍了用于处理 `string` 值和应用正则表达式的标准库功能，这些功能由 `strings`、`unicode` 和 `regexp` 包提供。在下一章中，我将介绍相关的功能，这些功能允许对字符串进行格式化和扫描。

