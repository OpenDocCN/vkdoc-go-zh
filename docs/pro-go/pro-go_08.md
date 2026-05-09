# 18. 数学函数与数据排序

在本章中，我将介绍两组功能。首先，我介绍对执行常见数学任务的支持，包括生成随机数。其次，我介绍对切片中的元素进行排序的功能，使其按顺序排列。表 18-1 给出了数学和排序功能的背景信息。

表 18-1  
数学函数与数据排序背景信息

| 问题 | 答案 |
| --- | --- |
| 它们是什么？ | 数学函数允许执行常见的计算。随机数是按难以预测的顺序生成的数字。排序是将一系列值按预定顺序排列的过程。 |
| 为什么它们有用？ | 这些是在整个开发过程中都会使用的功能。 |
| 如何使用它们？ | 这些功能在 `math`、`math/rand` 和 `sort` 包中提供。 |
| 是否存在任何陷阱或限制？ | 除非使用种子值初始化，否则 `math/rand` 包生成的数字不是随机的。 |
| 有替代方案吗？ | 你可以从头实现这两组功能，不过提供这些包就是为了避免这样做。 |

表 18-2 总结了本章内容。

表 18-2  
本章总结

| 问题 | 解决方案 | 代码清单 |
| --- | --- | --- |
| 执行常见计算 | 使用 `math` 包中定义的函数 | 5 |
| 生成随机数 | 使用 `math/rand` 包中的函数，注意提供种子值 | 6–9 |
| 打乱切片中的元素 | 使用 `Shuffle` 函数 | 10 |
| 对切片中的元素进行排序 | 使用 `sort` 包中定义的函数 | 11, 12, 15–20 |
| 在已排序的切片中定位元素 | 使用 `Search*` 函数 | 13, 14 |

## 本章准备工作

为了准备本章，打开一个新的命令提示符，导航到一个方便的位置，并创建一个名为 `mathandsorting` 的目录。在 `mathandsorting` 文件夹中运行代码清单 18-1 所示的命令，以创建一个模块文件。

**提示**
你可以从 [`https://github.com/apress/pro-go`](https://github.com/apress/pro-go) 下载本章以及本书所有其他章节的示例项目。如果在运行示例时遇到问题，请参阅第 2 章了解如何获取帮助。

```
go mod init mathandsorting
代码清单 18-1
初始化模块
```

向 `mathandsorting` 文件夹添加一个名为 `printer.go` 的文件，其内容如代码清单 18-2 所示。

```
package main
import "fmt"
func Printfln(template string, values ...interface{}) {
fmt.Printf(template + "\n", values...)
}
代码清单 18-2
mathandsorting 文件夹中 printer.go 文件的内容
```

向 `mathandsorting` 文件夹添加一个名为 `main.go` 的文件，其内容如代码清单 18-3 所示。

```
package main
func main() {
Printfln("Hello, Math and Sorting")
}
代码清单 18-3
mathandsorting 文件夹中 main.go 文件的内容
```

使用命令提示符在 `mathandsorting` 文件夹中运行代码清单 18-4 所示的命令。

```
go run .
代码清单 18-4
运行示例项目
```

该代码将编译并执行，生成以下输出：

```
Hello, Math and Sorting
```



## 使用数字

正如我在第 4 章中解释的，Go 语言支持一组可应用于数值的算术运算符，允许执行加法和乘法等基本任务。对于更高级的操作，Go 标准库包含 `math` 包，它提供了大量的函数。表 18-3 描述了在典型项目中最常用的函数。有关完整的函数集，包括对三角学等更特定领域的支持，请参阅 [`https://golang.org/pkg/math`](https://golang.org/pkg/math) 上的包文档。

**表 18-3：math 包中的实用函数**

| 名称 | 描述 |
| --- | --- |
| `Abs(val)` | 此函数返回 `float64` 值的绝对值，即不考虑方向时与零的距离。 |
| `Ceil(val)` | 此函数返回大于或等于指定 `float64` 值的最小整数。结果也是一个 `float64` 值，即使它代表一个整数。 |
| `Copysign(x, y)` | 此函数返回一个 `float64` 值，它是 `x` 的绝对值并带有 `y` 的符号。 |
| `Floor(val)` | 此函数返回小于或等于指定 `float64` 值的最大整数。结果也是一个 `float64` 值，即使它代表一个整数。 |
| `Max(x, y)` | 此函数返回指定的 `float64` 值中的最大值。 |
| `Min(x, y)` | 此函数返回指定的 `float64` 值中的最小值。 |
| `Mod(x, y)` | 此函数返回 `x` / `y` 的余数。 |
| `Pow(x, y)` | 此函数返回 `x` 的 `y` 次幂。 |
| `Round(val)` | 此函数将指定值四舍五入到最接近的整数，将 .5 的情况向上舍入。结果是一个 `float64` 值，即使它代表一个整数。 |
| `RoundToEven(val)` | 此函数将指定值舍入到最接近的整数，将 .5 的情况舍入到最接近的偶数。结果是一个 `float64` 值，即使它代表一个整数。 |

这些函数都作用于 `float64` 值并产生 `float64` 结果，这意味着你必须显式地与其他类型进行转换。清单 18-5 演示了表 18-3 中所描述的函数的用法。

```
package main
import "math"
func main() {
val1 := 279.00
val2 := 48.95
Printfln("Abs: %v", math.Abs(val1))
Printfln("Ceil: %v", math.Ceil(val2))
Printfln("Copysign: %v", math.Copysign(val1, -5))
Printfln("Floor: %v", math.Floor(val2))
Printfln("Max: %v", math.Max(val1, val2))
Printfln("Min: %v", math.Min(val1, val2))
Printfln("Mod: %v", math.Mod(val1, val2))
Printfln("Pow: %v", math.Pow(val1, 2))
Printfln("Round: %v", math.Round(val2))
Printfln("RoundToEven: %v", math.RoundToEven(val2))
}
清单 18-5：在 mathandsorting 文件夹的 main.go 文件中使用 math 包中的函数
```

编译并执行该项目，你将看到以下输出：

```
Abs: 279
Ceil: 49
Copysign: -279
Floor: 48
Max: 279
Min: 48.95
Mod: 34.249999999999986
Pow: 77841
Round: 49
RoundToEven: 49
```

`math` 包还为数值数据类型的限制提供了一组常量，如表 18-4 所述。

**表 18-4：限制常量**

| 名称 | 描述 |
| --- | --- |
| `MaxInt8` / `MinInt8` | 这些常量表示可以使用 `int8` 存储的最大值和最小值。 |
| `MaxInt16` / `MinInt16` | 这些常量表示可以使用 `int16` 存储的最大值和最小值。 |
| `MaxInt32` / `MinInt32` | 这些常量表示可以使用 `int32` 存储的最大值和最小值。 |
| `MaxInt64` / `MinInt64` | 这些常量表示可以使用 `int64` 存储的最大值和最小值。 |
| `MaxUint8` | 此常量表示可以使用 `uint8` 表示的最大值。最小值是零。 |
| `MaxUint16` | 此常量表示可以使用 `uint16` 表示的最大值。最小值是零。 |
| `MaxUint32` | 此常量表示可以使用 `uint32` 表示的最大值。最小值是零。 |
| `MaxUint64` | 此常量表示可以使用 `uint64` 表示的最大值。最小值是零。 |
| `MaxFloat32` / `MaxFloat64` | 这些常量表示可以使用 `float32` 和 `float64` 值表示的最大值。 |
| `SmallestNonzeroFloat32` / `SmallestNonzeroFloat64` | 这些常量表示可以使用 `float32` 和 `float64` 值表示的最小非零值。 |

### 生成随机数

`math/rand` 包为生成随机数提供了支持。最有用的函数在表 18-5 中进行了描述。（尽管我在本节中使用了术语 *随机*，但 `math/rand` 包产生的数字是伪随机的，这意味着它们不应用于随机性至关重要的场景，例如生成加密密钥。）

**表 18-5：实用的 math/rand 函数**

| 名称 | 描述 |
| --- | --- |
| `Seed(s)` | 此函数使用指定的 `int64` 值设置种子值。 |
| `Float32()` | 此函数生成一个介于 0 和 1 之间的随机 `float32` 值。 |
| `Float64()` | 此函数生成一个介于 0 和 1 之间的随机 `float64` 值。 |
| `Int()` | 此函数生成一个随机的 `int` 值。 |
| `Intn(max)` | 此函数生成一个小于指定值的随机 `int`。 |
| `UInt32()` | 此函数生成一个随机的 `uint32` 值。 |
| `UInt64()` | 此函数生成一个随机的 `uint64` 值。 |
| `Shuffle(count, func)` | 此函数用于随机化元素的顺序。 |

`math/rand` 包的一个奇怪之处在于，默认情况下它返回一组可预测的值，如清单 18-6 所示。

```
package main
import "math/rand"
func main() {
for i := 0; i < 5; i++ {
Printfln("Value %v : %v", i, rand.Int())
}
}
清单 18-6：在 mathandsorting 文件夹的 main.go 文件中生成可预测的值
```

此示例调用 `Int` 函数并写出该值。编译并执行代码，你将看到以下输出：

```
Value 0 : 5577006791947779410
Value 1 : 8674665223082153551
Value 2 : 6129484611666145821
Value 3 : 4037200794235010051
Value 4 : 3916589616287113937
```

清单 18-6 中的代码将始终生成相同的数字集，这是因为初始种子值始终相同。为了避免生成相同的数字序列，必须使用一个非固定的值调用 `Seed` 函数，如清单 18-7 所示。

```
package main
import (
"math/rand"
"time"
)
func main() {
rand.Seed(time.Now().UnixNano())
for i := 0; i < 5; i++ {
Printfln("Value %v : %v", i, rand.Int())
}
}
清单 18-7：在 mathandsorting 文件夹的 main.go 文件中设置种子值
```

通常的做法是使用当前时间作为种子值，这是通过调用 `time` 包提供的 `Now` 函数，并对结果调用 `UnixNano` 方法来实现的，该方法提供一个可以传递给 `Seed` 函数的 `int64` 值。（我将在第 19 章中描述 `time` 包。）编译并执行该项目，你将看到一系列每次执行程序时都会变化的数字。以下是我收到的输出：

```
Value 0 : 8113726196145714527
Value 1 : 3479565125812279859
Value 2 : 8074476402089812953
Value 3 : 3916870404047362448
Value 4 : 8226545715271170755
```



### 生成指定范围内的随机数

`Intn`函数可用于生成具有指定最大值的随机数，如代码清单[18-8]所示。

```
package main
import (
"math/rand"
"time"
)
func main() {
rand.Seed(time.Now().UnixNano())
for i := 0; i < 5; i++ {
Printfln("Value %v : %v", i, rand.Intn(10))
}
}
代码清单 18-8
在 mathandsorting 文件夹的 main.go 文件中指定最大值
```

该语句指定所有随机数都应小于 10。编译并执行代码，你将看到类似如下的输出，但具体的随机值会不同：

```
Value 0 : 7
Value 1 : 5
Value 2 : 4
Value 3 : 0
Value 4 : 7
```

虽然没有专门指定最小值的函数，但可以轻松地将 `Intn` 函数生成的值偏移到特定范围内，如代码清单[18-9]所示。

```
package main
import (
"math/rand"
"time"
)
func IntRange(min, max int) int {
return rand.Intn(max - min) + min
}
func main() {
rand.Seed(time.Now().UnixNano())
for i := 0; i < 5; i++ {
Printfln("Value %v : %v", i, IntRange(10, 20))
}
}
代码清单 18-9
在 mathandsorting 文件夹的 main.go 文件中指定下限
```

`IntRange` 函数返回指定范围内的一个随机数。编译并执行该项目，你将得到一系列介于 10 到 19 之间的数字，类似于：

```
Value 0 : 10
Value 1 : 19
Value 2 : 11
Value 3 : 10
Value 4 : 17
```

### 打乱元素顺序

`Shuffle` 函数用于随机重新排列元素，它通过一个自定义函数来实现，如代码清单[18-10]所示。

```
package main
import (
"math/rand"
"time"
)
var names = []string { "Alice", "Bob", "Charlie", "Dora", "Edith"}
func main() {
rand.Seed(time.Now().UnixNano())
rand.Shuffle(len(names), func (first, second int) {
names[first], names[second] = names[second], names[first]
})
for i, name := range names {
Printfln("Index %v: Name: %v", i, name)
}
}
代码清单 18-10
在 mathandsorting 文件夹的 main.go 文件中打乱元素顺序
```

`Shuffle` 函数的参数是元素数量和一个用于交换两个元素的函数，这两个元素通过索引标识。该函数被调用来随机交换元素。在代码清单[18-10]中，匿名函数交换了 `names` 切片中的两个元素，这意味着使用 `Shuffle` 函数起到了打乱 `names` 值顺序的效果。编译并执行该项目，输出将显示 `names` 切片中元素被打乱后的顺序，类似于：

```
Index 0: Name: Edith
Index 1: Name: Dora
Index 2: Name: Charlie
Index 3: Name: Alice
Index 4: Name: Bob
```

## 排序数据

前面的示例展示了如何打乱切片中的元素，但更常见的需求是将元素排列成更可预测的顺序，这便是 `sort` 包提供的函数所负责的工作。在接下来的章节中，我将介绍该包提供的内置排序功能，并演示它们的用法。

### 对数字和字符串切片进行排序

表 18-6 中描述的函数用于对包含 `int`、`float64` 或 `string` 值的切片进行排序。

**表 18-6：** 用于排序的基本函数

| 名称 | 描述 |
| --- | --- |
| `Float64s(slice)` | 此函数对 `float64` 值的切片进行排序。元素会被原地排序。 |
| `Float64sAreSorted(slice)` | 如果指定的 `float64` 切片中的元素已排序，则此函数返回 `true`。 |
| `Ints(slice)` | 此函数对 `int` 值的切片进行排序。元素会被原地排序。 |
| `IntsAreSorted(slice)` | 如果指定的 `int` 切片中的元素已排序，则此函数返回 `true`。 |
| `Strings(slice)` | 此函数对 `string` 值的切片进行排序。元素会被原地排序。 |
| `StringsAreSorted(slice)` | 如果指定的 `string` 切片中的元素已排序，则此函数返回 `true`。 |

每种数据类型都有一组自己的函数，用于对数据进行排序或判断数据是否已排序，如代码清单[18-11]所示。

```
package main
import (
//"math/rand"
//"time"
"sort"
)
func main() {
ints := []int { 9, 4, 2, -1, 10}
Printfln("Ints: %v", ints)
sort.Ints(ints)
Printfln("Ints Sorted: %v", ints)
floats := []float64 { 279, 48.95, 19.50 }
Printfln("Floats: %v", floats)
sort.Float64s(floats)
Printfln("Floats Sorted: %v", floats)
strings := []string { "Kayak", "Lifejacket", "Stadium" }
Printfln("Strings: %v", strings)
if (!sort.StringsAreSorted(strings)) {
sort.Strings(strings)
Printfln("Strings Sorted: %v", strings)
} else {
Printfln("Strings Already Sorted: %v", strings)
}
}
代码清单 18-11
在 mathandsorting 文件夹的 main.go 文件中排序切片
```

此示例对包含 `int` 和 `float64` 值的切片进行排序。还有一个 `string` 切片，使用 `StringsAreSorted` 函数对其进行了检查，以避免对已排序的数据进行排序。编译并执行该项目，你将看到如下输出：

```
Ints: [9 4 2 -1 10]
Ints Sorted: [-1 2 4 9 10]
Floats: [279 48.95 19.5]
Floats Sorted: [19.5 48.95 279]
Strings: [Kayak Lifejacket Stadium]
Strings Already Sorted: [Kayak Lifejacket Stadium]
```

请注意，代码清单[18-11]中的函数是原地对元素进行排序，而不是创建新的切片。如果你想创建一个已排序的新切片，则必须使用内置的 `make` 和 `copy` 函数，如代码清单[18-12]所示。这些函数在第[7]章中介绍过。

```
package main
import (
"sort"
)
func main() {
ints := []int { 9, 4, 2, -1, 10}
sortedInts := make([]int, len(ints))
copy(sortedInts, ints)
sort.Ints(sortedInts)
Printfln("Ints: %v", ints)
Printfln("Ints Sorted: %v", sortedInts)
}
代码清单 18-12
在 mathandsorting 文件夹的 main.go 文件中创建切片的已排序副本
```

编译并执行该项目，你将看到如下输出：

```
Ints: [9 4 2 -1 10]
Ints Sorted: [-1 2 4 9 10]
```



### 排序已排序数据

`sort` 包定义了表 18-7 中描述的函数，用于在已排序数据中搜索特定值。

**表 18-7** 用于搜索已排序数据的函数

| 名称 | 描述 |
| --- | --- |
| `SearchInts(slice, val)` | 此函数在已排序切片中搜索指定的 `int` 值。结果是该指定值的索引，如果未找到该值，则结果为在保持排序顺序的情况下可插入该值的索引。 |
| `SearchFloat64s(slice, val)` | 此函数在已排序切片中搜索指定的 `float64` 值。结果是该指定值的索引，如果未找到该值，则结果为在保持排序顺序的情况下可插入该值的索引。 |
| `SearchStrings(slice, val)` | 此函数在已排序切片中搜索指定的 `string` 值。结果是该指定值的索引，如果未找到该值，则结果为在保持排序顺序的情况下可插入该值的索引。 |
| `Search(count, testFunc)` | 此函数对指定数量的元素调用测试函数。结果是函数返回 `true` 的索引。如果没有匹配项，则结果为可插入指定值以保持排序顺序的索引。 |

表 18-7 中描述的函数使用起来略显笨拙。当找到值时，这些函数会返回其在切片中的位置。但不同寻常的是，如果未找到该值，则结果是在保持排序顺序的情况下可插入该值的位置，如清单 18-13 所示。

```
package main
import (
"sort"
)
func main() {
ints := []int { 9, 4, 2, -1, 10}
sortedInts := make([]int, len(ints))
copy(sortedInts, ints)
sort.Ints(sortedInts)
Printfln("Ints: %v", ints)
Printfln("Ints Sorted: %v", sortedInts)
indexOf4:= sort.SearchInts(sortedInts, 4)
indexOf3 := sort.SearchInts(sortedInts, 3)
Printfln("Index of 4: %v", indexOf4)
Printfln("Index of 3: %v", indexOf3)
}
清单 18-13
在 mathandsorting 文件夹中的 main.go 文件中搜索已排序数据
```

编译并执行代码，你会看到搜索切片中存在的值与搜索不存在的值得到了相同的结果：

```
Ints: [9 4 2 -1 10]
Ints Sorted: [-1 2 4 9 10]
Index of 4: 2
Index of 3: 2
```

这些函数需要一个额外的测试来检查这些函数返回位置上的值是否是所搜索的值，如清单 18-14 所示。

```
package main
import (
"sort"
)
func main() {
ints := []int { 9, 4, 2, -1, 10}
sortedInts := make([]int, len(ints))
copy(sortedInts, ints)
sort.Ints(sortedInts)
Printfln("Ints: %v", ints)
Printfln("Ints Sorted: %v", sortedInts)
indexOf4:= sort.SearchInts(sortedInts, 4)
indexOf3 := sort.SearchInts(sortedInts, 3)
Printfln("Index of 4: %v (present: %v)", indexOf4, sortedInts[indexOf4] == 4)
Printfln("Index of 3: %v (present: %v)", indexOf3, sortedInts[indexOf3] == 3)
}
清单 18-14
在 mathandsorting 文件夹中的 main.go 文件中消除搜索结果歧义
```

编译并执行项目，你将得到以下结果：

```
Ints: [9 4 2 -1 10]
Ints Sorted: [-1 2 4 9 10]
Index of 4: 2 (present: true)
Index of 3: 2 (present: false)
```

### 排序自定义数据类型

为了对自定义数据类型进行排序，`sort` 包定义了一个命名令人困惑的接口 `Interface`，该接口指定了表 18-8 中描述的方法。

**表 18-8** `sort.Interface` 接口定义的方法

| 名称 | 描述 |
| --- | --- |
| `Len()` | 此方法返回将要排序的项数。 |
| `Less(i, j)` | 如果索引 `i` 处的元素应出现在排序序列中索引 `j` 的元素之前，则此方法返回 `true`。如果 `Less(i,j)` 和 `Less(j, i)` 都为 `false`，则这些元素被视为相等。 |
| `Swap(i, j)` | 此方法交换指定索引处的元素。 |

当一个类型定义了表 18-8 中描述的方法时，就可以使用表 18-9 中描述的函数对其进行排序，这些函数由 `sort` 包定义。

**表 18-9** 用于排序实现 `Interface` 的类型的函数

| 名称 | 描述 |
| --- | --- |
| `Sort(data)` | 此函数使用表 18-8 中描述的方法对指定数据进行排序。 |
| `Stable(data)` | 此函数使用表 18-8 中描述的方法对指定数据进行排序，而不改变相等值元素的顺序。 |
| `IsSorted(data)` | 如果数据已按顺序排序，则此函数返回 `true`。 |
| `Reverse(data)` | 此函数反转数据的顺序。 |

表 18-8 中定义的方法应用于要排序的数据项集合，这意味着需要引入一个类型别名和用于转换的函数，以便调用表 18-9 中定义的函数。为演示，在 `mathandsorting` 文件夹中添加一个名为 `productsort.go` 的文件，代码如清单 18-15 所示。

```
package main
import "sort"
type Product struct {
Name string
Price float64
}
type ProductSlice []Product
func ProductSlices(p []Product) {
sort.Sort(ProductSlice(p))
}
func ProductSlicesAreSorted(p []Product) {
sort.IsSorted(ProductSlice(p))
}
func (products ProductSlice) Len() int {
return len(products)
}
func (products ProductSlice) Less(i, j int) bool {
return products[i].Price < products[j].Price
}
func (products ProductSlice) Swap(i, j int) {
products[i], products[j] = products[j], products[i]
}
清单 18-15
在 mathandsorting 文件夹中的 productsort.go 文件内容
```

`ProductSlice` 类型是 `Product` 切片的别名，并且是在该类型上实现了接口方法。除了这些方法，我还定义了一个 `ProductSlices` 函数，它接收一个 `Product` 切片，将其转换为 `ProductSlice` 类型，并将其作为参数传递给 `Sort` 函数。还有一个 `ProductSlicesAreSorted` 函数，它调用 `IsSorted` 函数。这些函数的命名遵循 `sort` 包建立的惯例，即在别名类型名后加上字母 `s`。清单 18-16 使用这些函数对 `Product` 值的切片进行排序。

```
package main
import (
//"sort"
)
func main() {
products := []Product {
{ "Kayak", 279} ,
{ "Lifejacket", 49.95 },
{ "Soccer Ball",  19.50 },
}
ProductSlices(products)
for _, p := range products {
Printfln("Name: %v, Price: %.2f", p.Name, p.Price)
}
}
清单 18-16
在 mathandsorting 文件夹中的 main.go 文件中排序切片
```

编译并执行项目，你将看到输出显示 `Product` 值按 `Price` 字段的升序排序：

```
Name: Soccer Ball, Price: 19.50
Name: Lifejacket, Price: 49.95
Name: Kayak, Price: 279.00
```



### 使用不同字段进行排序

类型组合可用于支持对同一结构体类型使用不同字段进行排序，如代码清单 18-17 所示。

```go
package main
import "sort"
type Product struct {
Name string
Price float64
}
type ProductSlice []Product
func ProductSlices(p []Product) {
sort.Sort(ProductSlice(p))
}
func ProductSlicesAreSorted(p []Product) {
sort.IsSorted(ProductSlice(p))
}
func (products ProductSlice) Len() int {
return len(products)
}
func (products ProductSlice) Less(i, j int) bool {
return products[i].Price < products[j].Price
}
func (products ProductSlice) Swap(i, j int) {
products[i], products[j] = products[j], products[i]
}
type ProductSliceName struct { ProductSlice }
func ProductSlicesByName(p []Product) {
sort.Sort(ProductSliceName{ p })
}
func (p ProductSliceName) Less(i, j int) bool {
return p.ProductSlice[i].Name < p.ProductSlice[j].Name
}
```

**代码清单 18-17** `mathandsorting` 文件夹中 `productsort.go` 文件内对不同字段进行排序

为每个需要排序的结构体字段定义一个结构体类型，其中嵌入一个 `ProductSlice` 字段，如下所示：

```go
...
type ProductSliceName struct { ProductSlice }
...
```

类型组合特性意味着为 `ProductSlice` 类型定义的方法会被提升到外层类型。为外层类型定义一个新的 `Less` 方法，该方法将用于使用不同字段对数据进行排序，如下所示：

```go
...
func (p ProductSliceName) Less(i, j int) bool {
return p.ProductSlice[i].Name <= p.ProductSlice[j].Name
}
...
```

最后一步是定义一个函数，执行从 `Product` 切片到新类型的转换，并调用 `Sort` 函数：

```go
...
func ProductSlicesByName(p []Product) {
sort.Sort(ProductSliceName{ p })
}
...
```

代码清单 18-17 中添加内容的结果是，`Product` 值的切片可以根据其 `Name` 字段的值进行排序，如代码清单 18-18 所示。

```go
package main
import (
//"sort"
)
func main() {
products := []Product {
{ "Kayak", 279} ,
{ "Lifejacket", 49.95 },
{ "Soccer Ball",  19.50 },
}
ProductSlicesByName(products)
for _, p := range products {
Printfln("Name: %v, Price: %.2f", p.Name, p.Price)
}
}
```

**代码清单 18-18** `mathandsorting` 文件夹中 `main.go` 文件内按附加字段排序

编译并执行该项目，你会看到 `Product` 值按其 `Name` 字段排序，结果如下：

```
Name: Kayak, Price: 279.00
Name: Lifejacket, Price: 49.95
Name: Soccer Ball, Price: 19.50
```

### 指定比较函数

另一种方法是在 `sort` 函数外部指定用于比较元素的表达式，如代码清单 18-19 所示。

```go
package main
import "sort"
type Product struct {
Name string
Price float64
}
type ProductSlice []Product
// ...为简洁起见省略了类型和函数...
type ProductComparison func(p1, p2 Product) bool
type ProductSliceFlex struct {
ProductSlice
ProductComparison
}
func (flex ProductSliceFlex) Less(i, j int) bool {
return flex.ProductComparison(flex.ProductSlice[i], flex.ProductSlice[j])
}
func SortWith(prods []Product, f ProductComparison) {
sort.Sort(ProductSliceFlex{ prods, f})
}
```

**代码清单 18-19** `mathandsorting` 文件夹中 `productsort.go` 文件内使用外部比较函数

创建了一个名为 `ProductSliceFlex` 的新类型，它结合了数据和比较函数，这将使此方法能够适应 `sort` 包所定义函数的结构。为 `ProductSliceFlex` 类型定义了一个 `Less` 方法，该方法会调用比较函数。最后一部分是 `SortWith` 函数，它将数据和函数组合成一个 `ProductSliceFlex` 值，并将其传递给 `sort.Sort` 函数。代码清单 18-20 演示了通过指定比较函数来对数据进行排序。

```go
package main
import (
//"sort"
)
func main() {
products := []Product {
{ "Kayak", 279} ,
{ "Lifejacket", 49.95 },
{ "Soccer Ball",  19.50 },
}
SortWith(products, func (p1, p2 Product) bool {
return p1.Name < p2.Name
})
for _, p := range products {
Printfln("Name: %v, Price: %.2f",  p.Name, p.Price)
}
}
```

**代码清单 18-20** `mathandsorting` 文件夹中 `main.go` 文件内使用比较函数进行排序

数据通过比较 `Name` 字段进行排序，编译并执行该项目后，代码产生以下输出：

```
Name: Kayak, Price: 279.00
Name: Lifejacket, Price: 49.95
Name: Soccer Ball, Price: 19.50
```

## 本章小结

在本章中，我介绍了用于生成随机数和打乱切片元素的功能。我还介绍了与之相反的功能，即对切片中的元素进行排序。在下一章中，我将介绍关于时间、日期和时段的标准库功能。

## 19. 日期、时间和时段

在本章中，我将介绍 `time` 包提供的功能，该包是标准库的一部分，负责表示时间点和时段。表 19-1 将这些功能置于上下文中。

**表 19-1** 日期、时间和时段背景概览

| 问题 | 答案 |
| --- | --- |
| 这些是什么？ | `time` 包提供的功能用于表示特定的时间点以及时间间隔或时段。 |
| 为什么它们有用？ | 这些功能在任何需要处理日历或闹钟的应用程序中，以及在任何需要未来延迟或通知的功能开发中都非常有用。 |
| 如何使用它们？ | `time` 包定义了表示日期和单个时间单位的数据类型，以及用于操作它们的函数。此外，还有一些功能集成到了 Go 语言的通道系统中。 |
| 是否存在任何陷阱或限制？ | 日期可能很复杂，必须小心处理日历和时区问题。 |
| 有没有替代方案？ | 这些是可选功能，并非必须使用。 |

**表 19-2** 本章内容总结

| 问题 | 解决方案 | 代码清单 |
| --- | --- | --- |
| 表示时间、日期或时段 | 使用 `time` 包定义的函数和类型 | 5, 13–16 |
| 将日期和时间格式化为字符串 | 使用 `Format` 函数和布局模板 | 6–7 |
| 从字符串解析日期和时间 | 使用 `Parse` 函数 | 8–12 |
| 从字符串解析时段 | 使用 `ParseDuration` 函数 | 17 |
| 暂停 goroutine 的执行 | 使用 `Sleep` 函数 | 18 |
| 延迟执行函数 | 使用 `AfterFunc` 函数 | 19 |
| 接收周期性通知 | 使用 `After` 函数 | 20–24 |



## 本章准备

为准备本章内容，请打开新的命令提示符，导航至方便的位置，创建一个名为 `datesandtimes` 的目录。在 `datesandtimes` 文件夹中运行清单 19-1 所示的命令，以创建一个模块文件。

**提示**

你可以从 [`github.com/apress/pro-go`](https://github.com/apress/pro-go) 下载本章及本书所有其他章节的示例项目。如果在运行示例时遇到问题，请参阅第 2 章了解如何获取帮助。

```
go mod init datesandtimes
```

*清单 19-1 初始化模块*

在 `datesandtimes` 文件夹中添加一个名为 `printer.go` 的文件，内容如清单 19-2 所示。

```
package main
import "fmt"
func Printfln(template string, values ...interface{}) {
fmt.Printf(template + "\n", values...)
}
```

*清单 19-2 datesandtimes 文件夹中 printer.go 文件的内容*

在 `datesandtimes` 文件夹中添加一个名为 `main.go` 的文件，内容如清单 19-3 所示。

```
package main
func main() {
Printfln("Hello, Dates and Times")
}
```

*清单 19-3 datesandtimes 文件夹中 main.go 文件的内容*

使用命令提示符在 `datesandtimes` 文件夹中运行清单 19-4 所示的命令。

```
go run .
```

*清单 19-4 运行示例项目*

代码将被编译并执行，产生以下输出：

```
Hello, Dates and Times
```

### 处理日期和时间

`time` 包提供了测量时间间隔以及表示日期和时间的功能。在接下来的章节中，我将介绍其中最有用的功能。

## 表示日期和时间

`time` 包提供了 `Time` 类型，用于表示特定的时间点。表 19-3 中描述的函数用于创建 `Time` 值。

**表 19-3 time 包中用于创建 Time 值的函数**

| 名称 | 描述 |
| --- | --- |
| `Now()` | 此函数创建一个表示当前时刻的 `Time`。 |
| `Date(y, m, d, h, min, sec, nsec, loc)` | 此函数创建一个表示指定时刻的 `Time`，该时刻由年、月、日、时、分、秒、纳秒和 `Location` 参数表示。（`Location` 类型在“从字符串解析时间值”部分中描述。） |
| `Unix(sec, nsec)` | 此函数根据自协调世界时 (UTC) 1970 年 1 月 1 日以来的秒数和纳秒数创建一个 `Time` 值，通常称为 Unix 时间。 |

`Time` 的各个组件可通过表 19-4 中描述的方法进行访问。

**表 19-4 用于访问 Time 组件的方法**

| 名称 | 描述 |
| --- | --- |
| `Date()` | 此方法返回年、月和日组件。年和日以 `int` 值返回，月以 `Month` 值返回。 |
| `Clock()` | 此方法返回 `Time` 的时、分、秒组件。 |
| `Year()` | 此方法返回年组件，以 `int` 形式表示。 |
| `YearDay()` | 此方法返回一年中的第几天，以 `int` 形式表示，范围在 1 到 366 之间（以容纳闰年）。 |
| `Month()` | 此方法返回月组件，使用 `Month` 类型表示。 |
| `Day()` | 此方法返回月份中的第几天，以 `int` 形式表示。 |
| `Weekday()` | 此方法返回星期几，以 `Weekday` 类型表示。 |
| `Hour()` | 此方法返回一天中的小时，以 `int` 形式表示，范围在 0 到 23 之间。 |
| `Minute()` | 此方法返回当前小时已过去的分钟数，以 `int` 形式表示，范围在 0 到 59 之间。 |
| `Second()` | 此方法返回当前分钟已过去的秒数，以 `int` 形式表示，范围在 0 到 59 之间。 |
| `Nanosecond()` | 此方法返回当前秒已过去的纳秒数，以 `int` 形式表示，范围在 0 到 999,999,999 之间。 |

定义了两种类型来帮助描述 `Time` 值的组件，如表 19-5 所示。

**表 19-5 用于描述时间组件的类型**

| 名称 | 描述 |
| --- | --- |
| `Month` | 此类型表示月份，`time` 包为英文月份名称定义了常量值：`January`、`February` 等。`Month` 类型定义了一个 `String` 方法，在格式化字符串时使用这些名称。 |
| `Weekday` | 此类型表示星期几，`time` 包为英文星期名定义了常量值：`Sunday`、`Monday` 等。`Weekday` 类型定义了一个 `String` 方法，在格式化字符串时使用这些名称。 |

使用表 19-3 至 19-5 中描述的类型和方法，清单 19-5 演示了如何创建 `Time` 值并访问其组件。

```
package main
import "time"
func PrintTime(label string, t *time.Time) {
Printfln("%s: Day: %v: Month: %v Year: %v",
label, t.Day(), t.Month(), t.Year())
}
func main() {
current := time.Now()
specific := time.Date(1995, time.June, 9, 0, 0, 0, 0, time.Local)
unix := time.Unix(1433228090, 0)
PrintTime("Current", &current)
PrintTime("Specific", &specific)
PrintTime("UNIX", &unix)
}
```

*清单 19-5 在 datesandtimes 文件夹的 main.go 文件中创建 Time 值*

`main` 函数中的语句使用表 19-3 中描述的函数创建了三个不同的 `Time` 值。使用常量值 `June` 创建了其中一个 `Time` 值，这说明了表 19-5 中描述的一种类型的使用方法。这些 `Time` 值被传递给 `PrintTime` 函数，该函数使用表 19-4 中的方法访问日、月和年组件，以写出描述每个 `Time` 的消息。编译并执行该项目，你将看到类似下面的输出，其中 `Now` 函数返回的时间会有所不同：

```
Current: Day: 2: Month: June Year: 2021
Specific: Day: 9: Month: June Year: 1995
UNIX: Day: 2: Month: June Year: 2015
```

`Date` 函数的最后一个参数是 `Location`，它指定了该 `Time` 值将使用的时区对应的位置。在清单 19-5 中，我使用了 `time` 包定义的 `Local` 常量，它提供了系统时区的 `Location`。我将在本章后面的“从字符串解析时间值”部分解释如何创建不由系统配置决定的 `Location` 值。



### 将时间格式化为字符串

`Format`方法用于从`Time`值创建格式化字符串。字符串的格式通过提供一个布局字符串来指定，该字符串显示了`Time`的哪些组件是必需的，以及它们应该被表达的顺序和精度。表 19-6 描述了`Format`方法以供快速参考。

表 19-6
创建格式化字符串的`Time`方法

| 名称 | 描述 |
| --- | --- |
| `Format(layout)` | 此方法返回一个格式化字符串，该字符串使用指定的布局创建。 |

布局字符串使用一个参考时间，即 MST 时区（比格林威治标准时间晚 7 小时）2006 年 1 月 2 日星期一 15:04:05（即下午 3 点过 4 分 5 秒）。清单 19-6 演示了如何使用参考时间创建格式化字符串。

```
package main
import (
"time"
"fmt"
)
func PrintTime(label string, t *time.Time) {
layout := "Day: 02 Month: Jan Year: 2006"
fmt.Println(label, t.Format(layout))
}
func main() {
current := time.Now()
specific := time.Date(1995, time.June, 9, 0, 0, 0, 0, time.Local)
unix := time.Unix(1433228090, 0)
PrintTime("Current", &current)
PrintTime("Specific", &specific)
PrintTime("UNIX", &unix)
}
清单 19-6
datesandtimes 文件夹中的 main.go 文件：格式化时间值
```

布局可以混合日期组件和固定字符串，在示例中，我使用了一个布局来重新创建早期示例中使用的格式，该格式使用参考日期指定。编译并执行项目，您将看到以下输出：

```
Current Day: 03 Month: Jun Year: 2021
Specific Day: 09 Month: Jun Year: 1995
UNIX Day: 02 Month: Jun Year: 2015
```

`time`包定义了一组常用时间和日期格式的常量，如表 19-7 所示。

表 19-7
`time`包定义的布局常量

| 名称 | 参考日期格式 |
| --- | --- |
| `ANSIC` | `Mon Jan _2 15:04:05 2006` |
| `UnixDate` | `Mon Jan _2 15:04:05 MST 2006` |
| `RubyDate` | `Mon Jan 02 15:04:05 -0700 2006` |
| `RFC822` | `02 Jan 06 15:04 MST` |
| `RFC822Z` | `02 Jan 06 15:04 -0700` |
| `RFC850` | `Monday, 02-Jan-06 15:04:05 MST` |
| `RFC1123` | `Mon, 02 Jan 2006 15:04:05 MST` |
| `RFC1123Z` | `Mon, 02 Jan 2006 15:04:05 -0700` |
| `RFC3339` | `2006-01-02T15:04:05Z07:00` |
| `RFC3339Nano` | `2006-01-02T15:04:05.999999999Z07:00` |
| `Kitchen` | `3:04PM` |
| `Stamp` | `Jan _2 15:04:05` |
| `StampMilli` | `Jan _2 15:04:05.000` |
| `StampMicro` | `Jan _2 15:04:05.000000` |
| `StampNano` | `Jan _2 15:04:05.000000000` |

这些常量可以代替自定义布局使用，如清单 19-7 所示。

```
package main
import (
"time"
"fmt"
)
func PrintTime(label string, t *time.Time) {
//layout := "Day: 02 Month: Jan Year: 2006"
fmt.Println(label, t.Format(time.RFC822Z))
}
func main() {
current := time.Now()
specific := time.Date(1995, time.June, 9, 0, 0, 0, 0, time.Local)
unix := time.Unix(1433228090, 0)
PrintTime("Current", &current)
PrintTime("Specific", &specific)
PrintTime("UNIX", &unix)
}
清单 19-7
datesandtimes 文件夹中的 main.go 文件：使用预定义布局
```

自定义布局已被`RFC822Z`布局替换，当项目被编译和执行时，会产生以下输出：

```
Current 03 Jun 21 08:04 +0100
Specific 09 Jun 95 00:00 +0100
UNIX 02 Jun 15 07:54 +0100
```

### 从字符串解析时间值

`time`包提供了从字符串创建`Time`值的支持，如表 19-8 所述。

表 19-8
`time`包用于将字符串解析为时间值的函数

| 名称 | 描述 |
| --- | --- |
| `Parse(layout, str)` | 此函数使用指定的布局解析字符串以创建`Time`值。返回一个`error`来指示解析字符串时的问题。 |
| `ParseInLocation(layout, str, location)` | 此函数使用指定的布局解析字符串，如果字符串中不包含时区，则使用`Location`。返回一个`error`来指示解析字符串时的问题。 |

表 19-8 中描述的函数使用一个参考时间，该时间用于指定要解析的字符串的格式。这个参考时间是 MST 时区（比 GMT 晚七小时）2006 年 1 月 2 日星期一 15:04:05（即下午 3 点过 4 分 5 秒）。

参考日期的组件被排列起来，以指定要解析的日期字符串的布局，如清单 19-8 所示。

```
package main
import (
"time"
"fmt"
)
func PrintTime(label string, t *time.Time) {
//layout := "Day: 02 Month: Jan Year: 2006"
fmt.Println(label, t.Format(time.RFC822Z))
}
func main() {
layout := "2006-Jan-02"
dates := []string {
"1995-Jun-09",
"2015-Jun-02",
}
for _, d := range dates {
time, err := time.Parse(layout, d)
if (err == nil) {
PrintTime("Parsed", &time)
} else {
Printfln("Error: %s", err.Error())
}
}
}
清单 19-8
datesandtimes 文件夹中的 main.go 文件：解析日期字符串
```

此示例中使用的布局包括四位数的年份、三个字母的月份和两位数的日期，全部用连字符分隔。布局与要解析的字符串一起传递给`Parse`函数，该函数返回一个时间值和一个错误，该错误将详细说明任何解析问题。编译并执行项目，您将收到以下输出，尽管您可能会看到不同的时区偏移（我稍后会解释）：

```
Parsed 09 Jun 95 00:00 +0000
Parsed 02 Jun 15 00:00 +0000
```

## 使用预定义日期布局

表 19-7 中描述的布局常量可用于解析日期，如清单 19-9 所示。

```
package main
import (
"time"
"fmt"
)
func PrintTime(label string, t *time.Time) {
//layout := "Day: 02 Month: Jan Year: 2006"
fmt.Println(label, t.Format(time.RFC822Z))
}
func main() {
//layout := "2006-Jan-02"
dates := []string {
"09 Jun 95 00:00 GMT",
"02 Jun 15 00:00 GMT",
}
for _, d := range dates {
time, err := time.Parse(time.RFC822, d)
if (err == nil) {
PrintTime("Parsed", &time)
} else {
Printfln("Error: %s", err.Error())
}
}
}
清单 19-9
datesandtimes 文件夹中的 main.go 文件：使用预定义布局
```

此示例使用`RFC822`常量解析日期字符串，并产生以下输出，尽管您可能会看到不同的时区偏移：

```
Parsed 09 Jun 95 01:00 +0100
Parsed 02 Jun 15 01:00 +0100
```



### 指定解析位置

`Parse` 函数假定未指定时区的日期和时间以协调世界时（UTC）定义。可以使用 `ParseInLocation` 方法来指定当未指定时区时使用的位置，如清单 19-10 所示。

```
package main
import (
"time"
"fmt"
)
func PrintTime(label string, t *time.Time) {
//layout := "Day: 02 Month: Jan Year: 2006"
fmt.Println(label, t.Format(time.RFC822Z))
}
func main() {
layout := "02 Jan 06 15:04"
date := "09 Jun 95 19:30"
london, lonerr := time.LoadLocation("Europe/London")
newyork, nycerr := time.LoadLocation("America/New_York")
if (lonerr == nil && nycerr == nil) {
nolocation, _ := time.Parse(layout, date)
londonTime, _ := time.ParseInLocation(layout, date, london)
newyorkTime, _ := time.ParseInLocation(layout, date, newyork)
PrintTime("No location:", &nolocation)
PrintTime("London:", &londonTime)
PrintTime("New York:", &newyorkTime)
} else {
fmt.Println(lonerr.Error(), nycerr.Error())
}
}
Listing 19-10
在 datesandtimes 文件夹的 main.go 文件中指定位置
```

`ParseInLocation` 接受一个 `time.Location` 参数，该参数指定一个位置，当解析的字符串中不含时区时，将使用该位置的时区。`Location` 值可以使用表 19-9 中描述的函数创建。

**表 19-9** 创建位置的函数

| 名称 | 描述 |
| --- | --- |
| `LoadLocation(name)` | 该函数返回指定名称的 `*Location` 和一个指示任何问题的 `error`。 |
| `LoadLocationFromTZData(name, data)` | 该函数从一个包含格式化时区数据库的 `byte` 切片返回 `*Location`。 |
| `FixedZone(name, offset)` | 该函数返回一个始终使用指定名称和 UTC 偏移量的 `*Location`。 |

当向 `LoadLocation` 函数传递一个地点时，返回的 `Location` 包含该地点所使用的时区的详细信息。地点名称定义在 IANA 时区数据库 [`www.iana.org/time-zones`](https://www.iana.org/time-zones) 中，并列出在 [`en.wikipedia.org/wiki/List_of_tz_database_time_zones`](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) 上。清单 19-10 中的示例指定了 `Europe/London` 和 `America/New_York`，分别生成了伦敦和纽约的 `Location` 值。编译并执行代码，你将看到以下输出：

```
No location: 09 Jun 95 19:30 +0000
London: 09 Jun 95 19:30 +0100
New York: 09 Jun 95 19:30 -0400
```

这三个日期显示了如何使用不同的时区解析字符串。当使用 `Parse` 方法时，时区被假定为 UTC，其偏移量为零（输出中的 `+0000` 部分）。当使用伦敦位置时，时间被假定为比 UTC 早一小时，因为解析字符串中的日期处于英国使用的夏令时期间。类似地，当使用纽约位置时，偏移量比 UTC 晚四小时。

### 嵌入时区数据库

用于创建 `Location` 值的时区数据库与 Go 工具一起安装，这意味着在部署编译后的应用程序时，该数据库可能不可用。`time/tzdata` 包包含一个嵌入版本的数据库，该数据库通过包初始化函数加载（如第 12 章所述）。为了确保时区数据始终可用，请像这样声明对该包的依赖：

```
...
import (
"fmt"
"time"
_ "time/tzdata"
)
...
```

该包中没有导出的特性，因此必须使用空白标识符来声明依赖关系，而不会产生编译器错误。

### 使用本地位置

如果用于创建 `Location` 的地点名称是 `Local`，则将使用运行应用程序的计算机的时区设置，如清单 19-11 所示。

```
package main
import (
"time"
"fmt"
)
func PrintTime(label string, t *time.Time) {
//layout := "Day: 02 Month: Jan Year: 2006"
fmt.Println(label, t.Format(time.RFC822Z))
}
func main() {
layout := "02 Jan 06 15:04"
date := "09 Jun 95 19:30"
london, lonerr := time.LoadLocation("Europe/London")
newyork, nycerr := time.LoadLocation("America/New_York")
local, _ := time.LoadLocation("Local")
if (lonerr == nil && nycerr == nil) {
nolocation, _ := time.Parse(layout, date)
londonTime, _ := time.ParseInLocation(layout, date, london)
newyorkTime, _ := time.ParseInLocation(layout, date, newyork)
localTime, _ := time.ParseInLocation(layout, date, local)
PrintTime("No location:", &nolocation)
PrintTime("London:", &londonTime)
PrintTime("New York:", &newyorkTime)
PrintTime("Local:", &localTime)
} else {
fmt.Println(lonerr.Error(), nycerr.Error())
}
}
Listing 19-11
在 datesandtimes 文件夹的 main.go 文件中使用本地时区
```

此示例产生的输出将因你的位置而异。我住在英国，这意味着在我的本地时区，夏令时期间比 UTC 早一小时，产生以下输出：

```
No location: 09 Jun 95 19:30 +0000
London: 09 Jun 95 19:30 +0100
New York: 09 Jun 95 19:30 -0400
Local: 09 Jun 95 19:30 +0100
```

### 直接指定时区

使用地点名称是确保日期正确解析的最可靠方法，因为夏令时会自动应用。`FixedZone` 函数可用于创建具有固定时区的 `Location`，如清单 19-12 所示。

```
package main
import (
"time"
"fmt"
)
func PrintTime(label string, t *time.Time) {
//layout := "Day: 02 Month: Jan Year: 2006"
fmt.Println(label, t.Format(time.RFC822Z))
}
func main() {
layout := "02 Jan 06 15:04"
date := "09 Jun 95 19:30"
london := time.FixedZone("BST", 1 * 60 * 60)
newyork := time.FixedZone("EDT", -4 * 60 * 60)
local := time.FixedZone("Local", 0)
//if (lonerr == nil && nycerr == nil) {
nolocation, _ := time.Parse(layout, date)
londonTime, _ := time.ParseInLocation(layout, date, london)
newyorkTime, _ := time.ParseInLocation(layout, date, newyork)
localTime, _ := time.ParseInLocation(layout, date, local)
PrintTime("No location:", &nolocation)
PrintTime("London:", &londonTime)
PrintTime("New York:", &newyorkTime)
PrintTime("Local:", &localTime)
// } else {
//     fmt.Println(lonerr.Error(), nycerr.Error())
// }
}
Listing 19-12
在 datesandtimes 文件夹的 main.go 文件中指定时区
```

`FixedZone` 函数的参数是一个名称和距 UTC 的偏移秒数。此示例创建了三个固定时区，其中一个比 UTC 早一小时，其中一个晚四小时，还有一个没有偏移。编译并执行该项目，你将看到以下输出：

```
No location: 09 Jun 95 19:30 +0000
London: 09 Jun 95 19:30 +0100
New York: 09 Jun 95 19:30 -0400
Local: 09 Jun 95 19:30 +0000
```



### 操作时间值

`time` 包中定义了用于处理 `Time` 值的方法，如表 19-10 所述。其中一些方法依赖于 `Duration` 类型，我将在下一节中对此进行描述。

**表 19-10** 用于处理时间值的方法

| 名称 | 描述 |
| --- | --- |
| `Add(duration)` | 此方法将指定的 `Duration` 添加到 `Time` 中，并返回结果。 |
| `Sub(time)` | 此方法返回一个 `Duration`，该值表示调用该方法的 `Time` 与作为参数提供的 `Time` 之间的差值。 |
| `AddDate(y, m, d)` | 此方法将指定的年、月、日添加到 `Time` 中，并返回结果。 |
| `After(time)` | 如果调用该方法的 `Time` 晚于作为参数提供的 `Time`，则此方法返回 `true`。 |
| `Before(time)` | 如果调用该方法的 `Time` 早于作为参数提供的 `Time`，则此方法返回 `true`。 |
| `Equal(time)` | 如果调用该方法的 `Time` 等于作为参数提供的 `Time`，则此方法返回 `true`。 |
| `IsZero()` | 如果调用该方法的 `Time` 表示零时间点（即 `公元 1 年 1 月 1 日, 00:00:00 UTC`），则此方法返回 `true`。 |
| `In(loc)` | 此方法返回以指定的 `Location` 表示的 `Time` 值。 |
| `Location()` | 此方法返回与 `Time` 相关联的 `Location`，从而有效地允许时间在不同的时区表示。 |
| `Round(duration)` | 此方法将 `Time` 舍入到由 `Duration` 值表示的最近间隔。 |
| `Truncate(duration)` | 此方法将 `Time` 向下舍入到由 `Duration` 值表示的最近间隔。 |

清单 19-13 从一个字符串解析出 `Time`，并使用表中描述的一些方法。

```
package main
import (
"time"
"fmt"
)
func main() {
t, err := time.Parse(time.RFC822, "09 Jun 95 04:59 BST")
if (err == nil) {
Printfln("After: %v", t.After(time.Now()))
Printfln("Round: %v", t.Round(time.Hour))
Printfln("Truncate: %v", t.Truncate(time.Hour))
} else {
fmt.Println(err.Error())
}
}
清单 19-13
在 datesandtimes 文件夹中的 main.go 文件里处理 Time 值
```

编译并执行该项目，你会收到以下输出，日期格式可能略有不同：

```
After: false
Round: 1995-06-09 05:00:00 +0100 BST
Truncate: 1995-06-09 04:00:00 +0100 BST
```

`Time` 值可以使用 `Equal` 函数进行比较，该函数会考虑时区差异，如清单 19-14 所示。

```
package main
import (
//"fmt"
"time"
)
func main() {
t1, _ := time.Parse(time.RFC822Z, "09 Jun 95 04:59 +0100")
t2, _ := time.Parse(time.RFC822Z, "08 Jun 95 23:59 -0400")
Printfln("Equal Method: %v", t1.Equal(t2))
Printfln("Equality Operator: %v", t1 == t2)
}
清单 19-14
在 datesandtimes 文件夹中的 main.go 文件里比较 Time 值
```

此示例中的 `Time` 值表示不同时区中的同一时刻。`Equal` 函数考虑了时区的影响，而使用标准相等运算符时则不会。编译并执行该项目，你会收到以下输出：

```
Equal Method: true
Equality Operator: false
```

### 表示持续时间

`Duration` 类型是 `int64` 类型的别名，用于表示特定的毫秒数。自定义的 `Duration` 值由 `time` 包中定义的 `Duration` 常量组成，如表 19-11 所述。

**表 19-11** time 包中的 Duration 常量

| 名称 | 描述 |
| --- | --- |
| `Hour` | 此常量表示 1 小时。 |
| `Minute` | 此常量表示 1 分钟。 |
| `Second` | 此常量表示 1 秒。 |
| `Millisecond` | 此常量表示 1 毫秒。 |
| `Microsecond` | 此常量表示 1 微秒。 |
| `Nanosecond` | 此常量表示 1 纳秒。 |

创建 `Duration` 后，可以使用表 19-12 中描述的方法进行检查。

**表 19-12** Duration 方法

| 名称 | 描述 |
| --- | --- |
| `Hours()` | 此方法返回一个 `float64`，表示以小时为单位的 `Duration`。 |
| `Minutes()` | 此方法返回一个 `float64`，表示以分钟为单位的 `Duration`。 |
| `Seconds()` | 此方法返回一个 `float64`，表示以秒为单位的 `Duration`。 |
| `Milliseconds()` | 此方法返回一个 `int64`，表示以毫秒为单位的 `Duration`。 |
| `Microseconds()` | 此方法返回一个 `int64`，表示以微秒为单位的 `Duration`。 |
| `Nanoseconds()` | 此方法返回一个 `int64`，表示以纳秒为单位的 `Duration`。 |
| `Round(duration)` | 此方法返回一个 `Duration`，该值被舍入到指定 `Duration` 的最近倍数。 |
| `Truncate(duration)` | 此方法返回一个 `Duration`，该值被向下舍入到指定 `Duration` 的最近倍数。 |

清单 19-15 演示了如何使用这些常量创建一个 `Duration`，并使用了表 19-12 中的一些方法。

```
package main
import (
//"fmt"
"time"
)
func main() {
var d time.Duration = time.Hour + (30 * time.Minute)
Printfln("Hours: %v", d.Hours())
Printfln("Mins: %v", d.Minutes())
Printfln("Seconds: %v", d.Seconds())
Printfln("Millseconds: %v", d.Milliseconds())
rounded := d.Round(time.Hour)
Printfln("Rounded Hours: %v", rounded.Hours())
Printfln("Rounded Mins: %v", rounded.Minutes())
trunc := d.Truncate(time.Hour)
Printfln("Truncated  Hours: %v", trunc.Hours())
Printfln("Rounded Mins: %v", trunc.Minutes())
}
清单 19-15
在 datesandtimes 文件夹中的 main.go 文件里创建和检查 Duration
```

`Duration` 被设置为 90 分钟，然后使用 `Hours()`、`Minutes()`、`Seconds()` 和 `Milliseconds()` 方法生成输出。使用 `Round` 和 `Truncate` 方法来创建新的 `Duration` 值，并以小时和分钟的形式输出。编译并执行该项目，你会收到以下输出：

```
Hours: 1.5
Mins: 90
Seconds: 5400
Millseconds: 5400000
Rounded Hours: 2
Rounded Mins: 120
Truncated  Hours: 1
Rounded Mins: 60
```

注意，表 19-12 中的方法返回的是以特定单位（如小时或分钟）表示的整个持续时间。这与 `Time` 类型定义的名称相似的方法不同，后者只返回日期/时间的某一部分。



### 创建相对于某个时间的持续时间

`time`包定义了两个函数，可用于创建表示特定`Time`与当前`Time`之间时间量的`Duration`值，如表 19-13 所述。

**表 19-13**  
用于创建相对于某个`Time`的`Duration`值的`time`函数

| 名称 | 描述 |
| --- | --- |
| `Since(time)` | 此函数返回一个`Duration`，表示自指定的`Time`值以来经过的时间。 |
| `Until(time)` | 此函数返回一个`Duration`，表示直到指定的`Time`值所经过的时间。 |

清单 19-18 演示了这些函数的用法。

```
package main
import (
//"fmt"
"time"
)
func main() {
toYears := func(d time.Duration) int {
return int( d.Hours() / (24 * 365))
}
future := time.Date(2051, 0, 0, 0, 0, 0, 0, time.Local)
past := time.Date(1965, 0, 0, 0, 0, 0, 0, time.Local)
Printfln("Future: %v", toYears(time.Until(future)))
Printfln("Past: %v", toYears(time.Since(past)))
}
```

**清单 19-16**  
在`datesandtimes`文件夹中的`main.go`文件里创建相对于时间的`Duration`

该示例使用`Until`和`Since`方法来计算到 2051 年还有多少年，以及自 1965 年以来过去了多少年。清单 19-16 中的代码在编译后会产生以下输出，不过根据你运行示例的时间，你可能会看到不同的结果：

```
Future: 29
Past: 56
```

### 从字符串创建持续时间

`time.ParseDuration`函数解析字符串以创建`Duration`值。为快速参考，表 19-14 描述了此函数。

**表 19-14**  
用于将字符串解析为`Duration`值的函数

| 名称 | 描述 |
| --- | --- |
| `ParseDuration(str)` | 此函数返回一个`Duration`和一个`error`，指示解析指定字符串时是否存在问题。 |

`ParseDuration`函数支持的字符串格式是一系列数值后面跟着表 19-15 中描述的单位指示符。

**表 19-15**  
`Duration`字符串单位指示符

| 单位 | 描述 |
| --- | --- |
| `h` | 此单位表示小时。 |
| `m` | 此单位表示分钟。 |
| `s` | 此单位表示秒。 |
| `ms` | 此单位表示毫秒。 |
| `us` 或 `μs` | 这些单位表示微秒。 |
| `ns` | 此单位表示纳秒。 |

数值之间不允许有空格，可以指定为整数或浮点数。清单 19-17 演示了如何从字符串创建`Duration`。

```
package main
import (
"fmt"
"time"
)
func main() {
d, err := time.ParseDuration("1h30m")
if (err == nil) {
Printfln("Hours: %v", d.Hours())
Printfln("Mins: %v", d.Minutes())
Printfln("Seconds: %v", d.Seconds())
Printfln("Millseconds: %v", d.Milliseconds())
} else {
fmt.Println(err.Error())
}
}
```

**清单 19-17**  
在`datesandtimes`文件夹中的`main.go`文件里解析字符串

该字符串指定了 1 小时 30 分钟。编译并执行该项目，将产生以下输出：

```
Hours: 1.5
Mins: 90
Seconds: 5400
Millseconds: 5400000
```

## 在协程和通道中使用时间特性

`time`包提供了一小组函数，用于处理协程和通道，如表 19-16 所述。

**表 19-16**  
`time`包函数

| 名称 | 描述 |
| --- | --- |
| `Sleep(duration)` | 此函数将当前协程暂停至少指定的持续时间。 |
| `AfterFunc(duration, func)` | 此函数在指定的持续时间后，在自己的协程中执行指定的函数。结果是一个`*Timer`，其`Stop`方法可用于在持续时间结束前取消函数的执行。 |
| `After(duration)` | 此函数返回一个通道，该通道会阻塞指定的持续时间，然后产生一个`Time`值。详情请参阅“接收定时通知”一节。 |
| `Tick(duration)` | 此函数返回一个通道，该通道会定期发送一个`Time`值，周期由持续时间指定。 |

尽管这些函数都定义在同一个包中，但它们的用途不同，接下来的几节将进行演示。

### 让协程休眠

`Sleep`函数将当前协程的执行暂停指定的持续时间，如清单 19-18 所示。

```
package main
import (
//"fmt"
"time"
)
func writeToChannel(channel chan <- string) {
names := []string { "Alice", "Bob", "Charlie", "Dora" }
for _, name := range names {
channel <- name
time.Sleep(time.Second * 1)
}
close(channel)
}
func main() {
nameChannel := make (chan string)
go writeToChannel(nameChannel)
for name := range nameChannel {
Printfln("Read name: %v", name)
}
}
```

**清单 19-18**  
在`datesandtimes`文件夹中的`main.go`文件里暂停协程

`Sleep`函数指定的持续时间是协程被暂停的最短时间，你不应依赖精确的时间段，尤其是对于较短的持续时间。请记住，`Sleep`函数会暂停调用它的协程，这意味着它也会暂停`main`协程，这可能会让应用程序看起来像被锁住了。（如果发生这种情况，你不小心调用了`Sleep`函数的迹象是自动死锁检测不会引发恐慌。）编译并执行该项目，你会看到以下输出，每个名字之间会有短暂延迟：

```
Read name: Alice
Read name: Bob
Read name: Charlie
Read name: Dora
```

### 延迟执行函数

`AfterFunc`函数用于将函数的执行延迟指定的时间，如清单 19-19 所示。

```
package main
import (
//"fmt"
"time"
)
func writeToChannel(channel chan <- string) {
names := []string { "Alice", "Bob", "Charlie", "Dora" }
for _, name := range names {
channel <- name
//time.Sleep(time.Second * 1)
}
close(channel)
}
func main() {
nameChannel := make (chan string)
time.AfterFunc(time.Second * 5, func () {
writeToChannel(nameChannel)
})
for name := range nameChannel {
Printfln("Read name: %v", name)
}
}
```

**清单 19-19**  
在`datesandtimes`文件夹中的`main.go`文件里延迟执行函数

`AfterFunc`的第一个参数是延迟时间，本例中为五秒。第二个参数是要执行的函数。在这个例子中，我想执行`writeToChannel`函数，但`AfterFunc`只接受没有参数和返回值的函数，所以我必须使用一个简单的包装器。编译并执行该项目，你会看到以下结果，这些结果会在五秒延迟后写出：

```
Read name: Alice
Read name: Bob
Read name: Charlie
Read name: Dora
```



### 接收定时通知

`After` 函数会等待指定的持续时间，然后将一个 `Time` 值发送到一个通道，这是一种利用通道在指定的未来时间接收通知的有用方式，如代码清单 19-20 所示。

```
package main
import (
//"fmt"
"time"
)
func writeToChannel(channel chan <- string) {
Printfln("等待初始时长...")
_ = <- time.After(time.Second * 2)
Printfln("初始时长已过。")
names := []string { "Alice", "Bob", "Charlie", "Dora" }
for _, name := range names {
channel <- name
time.Sleep(time.Second * 1)
}
close(channel)
}
func main() {
nameChannel := make (chan string)
go writeToChannel(nameChannel)
for name := range nameChannel {
Printfln("读取姓名：%v", name)
}
}
代码清单 19-20
在 datesandtimes 文件夹的 main.go 文件中接收未来通知
```

`After` 函数返回一个通道，该通道携带 `Time` 值。该通道会阻塞指定的持续时间，当发送一个 `Time` 值时，表示持续时间已过。在此示例中，通过通道发送的值仅作为信号使用，并未直接使用，这就是为什么它被赋值给空白标识符，如下所示：

```
...
_ = <- time.After(time.Second * 2)
...
```

这种使用 `After` 函数的方式在 `writeToChannel` 函数中引入了初始延迟。编译并执行该项目，您将看到以下输出：

```
等待初始时长...
初始时长已过。
读取姓名：Alice
读取姓名：Bob
读取姓名：Charlie
读取姓名：Dora
```

在此示例中，其效果与使用 `Sleep` 函数相同，但区别在于 `After` 函数返回一个通道，该通道在读取值之前不会阻塞，这意味着可以先指定方向，执行额外的工作，然后再执行通道读取，这样通道只会阻塞持续时间的剩余部分。

### 在 Select 语句中将通知用作超时

`After` 函数可以与 `select` 语句一起使用来提供超时，如代码清单 19-21 所示。

```
package main
import (
//"fmt"
"time"
)
func writeToChannel(channel chan <- string) {
Printfln("等待初始时长...")
_ = <- time.After(time.Second * 2)
Printfln("初始时长已过。")
names := []string { "Alice", "Bob", "Charlie", "Dora" }
for _, name := range names {
channel <- name
time.Sleep(time.Second * 3)
}
close(channel)
}
func main() {
nameChannel := make (chan string)
go writeToChannel(nameChannel)
channelOpen := true
for channelOpen {
Printfln("开始读取通道")
select {
case name, ok := <- nameChannel:
if (!ok) {
channelOpen = false
break
} else {
Printfln("读取姓名：%v", name)
}
case <- time.After(time.Second * 2):
Printfln("超时")
}
}
}
代码清单 19-21
在 datesandtimes 文件夹的 main.go 文件中使用 Select 语句中的超时
```

`select` 语句会阻塞，直到其中一个通道就绪或计时器到期。这是因为 `select` 语句会阻塞直到其某个通道就绪，并且 `After` 函数创建了一个在指定时间段内阻塞的通道。编译并执行该项目，您将看到以下输出：

```
等待初始时长...
初始时长已过。
超时
读取姓名：Alice
超时
读取姓名：Bob
超时
读取姓名：Charlie
超时
读取姓名：Dora
超时
```

### 停止和重置计时器

当您确定始终需要定时通知时，`After` 函数非常有用。如果您需要取消通知的选项，则可以使用表 19-17 中描述的函数。

**表 19-17**  
用于创建计时器的 `time` 函数

| 名称 | 描述 |
| --- | --- |
| `NewTimer(duration)` | 此函数返回一个带有指定周期的 `*Timer`。 |

`NewTimer` 函数的结果是指向一个 `Timer` 结构体的指针，该结构体定义了表 19-18 中描述的方法。

**表 19-18**  
`Timer` 结构体定义的方法

| 名称 | 描述 |
| --- | --- |
| `C` | 此字段返回 `Time` 将通过其发送 `Time` 值的通道。 |
| `Stop()` | 此方法停止计时器。结果是一个 `bool`，如果计时器已停止则为 `true`，如果计时器已发送其消息则为 `false`。 |
| `Reset(duration)` | 此方法停止计时器并重置它，使其间隔为指定的 `Duration`。 |

代码清单 19-22 使用 `NewTimer` 函数创建一个 `Timer`，该计时器在指定持续时间过去之前被重置。

> **警告**  
> 停止计时器时要小心。计时器的通道不会关闭，这意味着即使计时器停止后，从通道读取仍会继续阻塞。

```
package main
import (
//"fmt"
"time"
)
func writeToChannel(channel chan <- string) {
timer := time.NewTimer(time.Minute * 10)
go func () {
time.Sleep(time.Second * 2)
Printfln("重置计时器")
timer.Reset(time.Second)
}()
Printfln("等待初始时长...")
<- timer.C
Printfln("初始时长已过。")
names := []string { "Alice", "Bob", "Charlie", "Dora" }
for _, name := range names {
channel <- name
//time.Sleep(time.Second * 3)
}
close(channel)
}
func main() {
nameChannel := make (chan string)
go writeToChannel(nameChannel)
for name := range nameChannel {
Printfln("读取姓名：%v", name)
}
}
代码清单 19-22
在 datesandtimes 文件夹的 main.go 文件中重置计时器
```

此示例中的 `Timer` 创建时持续时间为十分钟。一个 goroutine 休眠两秒，然后重置计时器，使其持续时间为两秒。编译并执行该项目，您将看到以下输出：

```
等待初始时长...
重置计时器
初始时长已过。
读取姓名：Alice
读取姓名：Bob
读取姓名：Charlie
读取姓名：Dora
```



### 接收定期通知

`Tick` 函数返回一个通道，该通道会按指定时间间隔发送 `Time` 值，如代码清单 19-23 所示。

```
package main
import (
//"fmt"
"time"
)
func writeToChannel(nameChannel chan <- string) {
names := []string { "Alice", "Bob", "Charlie", "Dora" }
tickChannel := time.Tick(time.Second)
index := 0
for {
<- tickChannel
nameChannel <- names[index]
index++
if (index == len(names)) {
index = 0
}
}
}
func main() {
nameChannel := make (chan string)
go writeToChannel(nameChannel)
for name := range nameChannel {
Printfln("Read name: %v", name)
}
}
代码清单 19-23
在 datesandtimes 文件夹的 main.go 文件中接收定期通知
```

同样，`Tick` 函数创建的通道的价值并非在于其上发送的 `Time` 值，而在于发送这些值的周期性。在此示例中，`Tick` 函数用于创建一个通道，该通道每秒发送一次值。当没有值可读时，该通道会阻塞，这使得 `Tick` 函数创建的通道能够控制 `writeToChannel` 函数生成值的速率。编译并执行该项目，你将看到以下输出，该输出会重复直到程序终止：

```
Read name: Alice
Read name: Bob
Read name: Charlie
Read name: Dora
Read name: Alice
Read name: Bob
...
```

当需要无限序列的信号时，`Tick` 函数非常有用。如果需要固定序列的值，则可以使用表 19-19 中描述的函数。

**表 19-19** 用于创建 Ticker 的 `time` 函数

| 名称 | 描述 |
| --- | --- |
| `NewTicker(duration)` | 此函数返回一个具有指定周期的 `*Ticker`。 |

`NewTicker` 函数的结果是一个指向 `Ticker` 结构体的指针，该结构体定义了表 19-20 中描述的字段和方法。

**表 19-20** `Ticker` 结构体定义的字段和方法

| 名称 | 描述 |
| --- | --- |
| `C` | 此字段返回 `Ticker` 用于发送其 `Time` 值的通道。 |
| `Stop()` | 此方法停止 ticker（但不会关闭 `C` 字段返回的通道）。 |
| `Reset(duration)` | 此方法停止一个 ticker 并重置它，使其时间间隔变为指定的 `Duration`。 |

代码清单 19-24 使用 `NewTicker` 函数创建一个 `Ticker`，并在不再需要时将其停止。

```
package main
import (
//"fmt"
"time"
)
func writeToChannel(nameChannel chan <- string) {
names := []string { "Alice", "Bob", "Charlie", "Dora" }
ticker := time.NewTicker(time.Second / 10)
index := 0
for {
<- ticker.C
nameChannel <- names[index]
index++
if (index == len(names)) {
ticker.Stop()
close(nameChannel)
break
}
}
}
func main() {
nameChannel := make (chan string)
go writeToChannel(nameChannel)
for name := range nameChannel {
Printfln("Read name: %v", name)
}
}
代码清单 19-24
在 datesandtimes 文件夹的 main.go 文件中创建一个 Ticker
```

当应用程序需要创建多个 ticker，同时又不希望那些不再需要的 ticker 继续发送消息时，这种方法非常有用。编译并执行该项目，你将看到以下输出：

```
Read name: Alice
Read name: Bob
Read name: Charlie
Read name: Dora
```

## 总结

在本章中，我描述了 Go 标准库中用于处理时间、日期和持续时间的特性，包括对通道和 goroutine 的内置支持。在下一章中，我将介绍 reader 和 writer，它们是 Go 中用于读写数据的机制。

