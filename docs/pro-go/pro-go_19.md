# 31. 单元测试、基准测试与日志记录

在本章中，我将通过单元测试、基准测试和日志记录来介绍最实用的标准库包。Go 的日志功能虽然基础，但尚可；此外还有许多第三方包可用于将日志信息发送到不同目的地。测试和基准测试功能已集成到 `go` 命令中，但正如我接下来要解释的，我对这两项功能并不热衷。表 31-1 总结了本章内容。

表 31-1  
章节概要

| 问题 | 解决方案 | 代码清单 |
| --- | --- | --- |
| 创建单元测试 | 添加一个以 `_test.go` 结尾的文件，定义一个以 `Test` 开头且后跟大写字母的函数，并使用 `testing` 包提供的功能 | 4, 6, 7, 10, 11 |
| 运行单元测试 | 使用 `go test` 命令 | 5, 8, 9 |
| 创建基准测试 | 定义一个以 `Benchmark` 开头且后跟大写字母的函数 | 12, 14, 15 |
| 运行基准测试 | 使用带有 `-bench` 参数的 `go test` 命令 | 13 |
| 记录数据 | 使用 `log` 包提供的功能 | 16, 17 |

## 本章准备

为准备本章内容，请打开一个新的命令提示符，导航到一个方便的位置，并创建一个名为 `tests` 的目录。在 `tests` 文件夹中运行如代码清单 31-1 所示的命令以创建一个模块文件。

提示

你可以从 [`https://github.com/apress/pro-go`](https://github.com/apress/pro-go) 下载本章——以及本书所有其他章节——的示例项目。如果在运行示例时遇到问题，请参阅第 2 章了解如何获取帮助。

```
go mod init tests
```
代码清单 31-1  
初始化模块

在 `tests` 文件夹中添加一个名为 `main.go` 的文件，其内容如代码清单 31-2 所示。

```
package main
import (
"sort"
"fmt"
)
func sortAndTotal(vals []int) (sorted []int, total int) {
sorted = make([]int, len(vals))
copy(sorted, vals)
sort.Ints(sorted)
for _, val := range sorted {
total += val
total++
}
return
}
func main() {
nums := []int { 100, 20, 1, 7, 84 }
sorted, total := sortAndTotal(nums)
fmt.Println("Sorted Data:", sorted)
fmt.Println("Total:", total)
}
```
代码清单 31-2  
`tests` 文件夹中 `main.go` 文件的内容

`sortAndTotal` 函数包含一个故意引入的错误，这将有助于演示下一节中的测试功能。在 `tests` 文件夹中运行如代码清单 31-3 所示的命令以编译并执行该项目。

```
go run .
```
代码清单 31-3  
编译并运行项目

该命令将产生以下输出：

```
Sorted Data: [1 7 20 84 100]
Total: 217
```

## 使用测试

单元测试定义在文件名以 `_test.go` 结尾的文件中。若要创建一个简单的测试，请在 `tests` 文件夹中添加一个名为 `simple_test.go` 的文件，其内容如代码清单 31-4 所示。

```
package main
import "testing"
func TestSum(t *testing.T) {
testValues := []int{ 10, 20, 30 }
_, sum := sortAndTotal(testValues)
expected := 60
if (sum != expected) {
t.Fatalf("Expected %v, Got %v", expected, sum)
}
}
```
代码清单 31-4  
`tests` 文件夹中 `simple_test.go` 文件的内容

Go 标准库通过 `testing` 包为编写单元测试提供支持。单元测试被表示为以 `Test` 开头的函数，后跟一个以大写字母开头的术语，例如 `TestSum`。（大写字母很重要，因为测试工具不会将以 `Testsum` 这样的函数名识别为单元测试。）

### 决定是否使用测试工具

我喜欢集成测试的理念，但我发现自己并不常用 Go 的测试功能；即使使用，也并非按预期的方式使用。

我喜欢单元测试，但我只在尝试理清具有复杂问题的代码时，或者在编写我知道很难一次写对的功能时，才会编写测试。这可能只是我思考测试的方式，或者是我习惯了经典的 arrange/act/assert 测试工具模式，但 Go 的测试工具总有一些让我不太喜欢的地方。

我最终使用测试是为了能创建简单的入口点来检查特定包是否正常工作。但即便如此，我也只是创建一个简单的测试，用于实例化包中的类型，从而让我能够访问它们定义的字段、函数和方法，而不必修改我的 `main` 函数。这些测试中的代码总是杂乱无章的，我使用 `println` 语句输出结果，而不是使用表 31-2 中描述的方法。一旦确认代码工作正常，我就会删除测试文件。

我完全承认这是我的一个缺点，但我对于 Go 的测试工具就是提不起热情。这并不意味着你不会觉得它们有用——也许你比我更勤奋。但如果你发现本节描述的功能并不能激发你编写测试的动力，那么请记住，你并不孤单。

单元测试函数接收一个指向 `T` 结构体的指针，该结构体定义了管理测试和报告测试结果的方法。Go 测试不依赖断言，而是使用常规代码语句编写。测试工具只关心测试是否失败，失败信息通过表 31-2 中描述的方法来报告。

表 31-2  
用于报告测试结果的 `T` 方法

| 名称 | 描述 |
| --- | --- |
| `Log(...vals)` | 此方法将指定的值写入测试错误日志。 |
| `Logf(template, ...vals)` | 此方法使用指定的模板和值向测试错误日志写入一条消息。 |
| `Fail()` | 调用此方法将测试标记为失败，但继续执行测试。 |
| `FailNow()` | 调用此方法将测试标记为失败，并停止执行测试。 |
| `Failed()` | 如果测试失败，此方法返回 `true`。 |
| `Error(...errs)` | 调用此方法等效于先调用 `Log` 方法，再调用 `Fail` 方法。 |
| `Errorf(template, ...vals)` | 调用此方法等效于先调用 `Logf` 方法，再调用 `Fail` 方法。 |
| `Fatal(...vals)` | 调用此方法等效于先调用 `Log` 方法，再调用 `FailNow` 方法。 |
| `Fatalf(template, ...vals)` | 调用此方法等效于先调用 `Logf` 方法，再调用 `FailNow` 方法。 |

代码清单 31-4 中的测试调用 `sumAndTotal` 函数，传入一组值，并使用标准 Go 比较运算符将结果与预期结果进行比较。如果结果不等于预期值，则调用 `Fatalf` 方法，该方法报告测试失败并阻止执行单元测试中的任何剩余语句（尽管此示例中没有剩余语句）。

### 理解测试包访问权限

代码清单 31-4 中的测试文件使用 `package` 关键字指定了 `main` 包。由于测试是用标准 Go 编写的，这意味着此文件中的测试可以访问 `main` 包中定义的所有功能，包括那些未在包外导出的功能。

如果你希望编写仅能访问导出功能的测试，则可以使用 `package` 语句指定 `main_test` 包。`_test` 后缀不会导致编译器问题，并且允许编写的测试仅能访问被测试包中导出的功能。



### 运行单元测试

要发现并运行项目中的单元测试，请在 `tests` 文件夹中运行如代码清单 31-5 所示的命令。

#### 为单元测试编写 Mock 对象

创建单元测试 mock 实现的唯一方法是创建接口实现，这样可以定义自定义方法来产生测试所需的结果。如果要在单元测试中使用 mock，则应编写接受接口类型的 API。

尽管 mock 的使用仅限于接口，但通常仍然可以创建结构体值，为其字段分配特定的值用于测试。这有时可能有点麻烦，但大多数函数和方法都可以通过某种方式进行测试，即使需要一些持久化的技巧来弄清楚细节。

```
go test
代码清单 31-5
执行单元测试
```

如前所述，代码清单 31-2 中定义的代码存在错误，导致单元测试失败：

```
tests > go test
--- FAIL: TestSum (0.00s)
simple_test.go:10: Expected 60, Got 63
FAIL
exit status 1
FAIL    tests   0.090s
```

测试输出报告了错误以及测试运行的整体结果。代码清单 31-6 修复了 `sortAndTotal` 函数中的错误。

```
...
func sortAndTotal(vals []int) (sorted []int, total int) {
sorted = make([]int, len(vals))
copy(sorted, vals)
sort.Ints(sorted)
for _, val := range sorted {
total += val
//total++
}
return
}
...
代码清单 31-6
修复 tests 文件夹中 main.go 文件中的错误
```

保存更改并运行 `go test` 命令，输出将显示测试通过：

```
PASS
ok      tests   0.102s
```

一个测试文件可以包含多个测试，它们会被自动发现并执行。代码清单 31-7 在 `simple_test.go` 文件中添加了第二个测试函数。

```
package main
import (
"testing"
"sort"
)
func TestSum(t *testing.T) {
testValues := []int{ 10, 20, 30 }
_, sum := sortAndTotal(testValues)
expected := 60
if (sum != expected) {
t.Fatalf("Expected %v, Got %v", expected, sum)
}
}
func TestSort(t *testing.T) {
testValues := []int{ 1, 279, 48, 12, 3}
sorted, _ := sortAndTotal(testValues)
if (!sort.IntsAreSorted(sorted)) {
t.Fatalf("Unsorted data %v", sorted)
}
}
代码清单 31-7
在 tests 文件夹的 simple_test.go 文件中定义测试
```

`TestSort` 测试验证 `sortAndTotal` 函数是否对数据进行了排序。请注意，在单元测试中我可以依赖 Go 标准库提供的特性，并使用 `sort.IntsAreSorted` 函数来执行测试。运行 `go test` 命令，你将看到以下结果：

```
ok      tests   0.087s
```

`go test` 命令默认不报告任何详细信息，但可以通过在 `tests` 文件夹中运行代码清单 31-8 所示的命令来生成更多信息。

```
go test  -v
代码清单 31-8
执行详细测试
```

`-v` 参数启用详细模式，该模式会报告每个测试的结果：

```
=== RUN   TestSum
--- PASS: TestSum (0.00s)
=== RUN   TestSort
--- PASS: TestSort (0.00s)
PASS
ok      tests   0.164s
```

#### 运行特定测试

`go test` 命令可用于按名称选择运行测试。在 `tests` 文件夹中运行代码清单 31-9 所示的命令。

```
go test -v -run "um"
代码清单 31-9
在 tests 文件夹的 main.go 文件中选择测试
```

测试通过正则表达式进行选择，代码清单 31-9 中的命令选择函数名包含 `um` 的测试（无需包含函数名中的 `Test` 部分）。唯一名称匹配该表达式的测试是 `TestSum`，该命令产生以下输出：

```
=== RUN   TestSum
--- PASS: TestSum (0.00s)
PASS
ok      tests   0.123s
```

### 管理测试执行

`T` 结构体还提供了一组用于管理测试执行的方法，如表 31-3 所述。

**表 31-3** 用于管理测试执行的 T 方法

| 名称 | 描述 |
| --- | --- |
| `Run(name, func)` | 调用此方法将指定的函数作为子测试执行。在测试于其自身的 goroutine 中执行时，该方法会阻塞，并返回一个 `bool` 值，指示测试是否成功。 |
| `SkipNow()` | 调用此方法将停止执行测试并将其标记为跳过。 |
| `Skip(...args)` | 此方法等同于先调用 `Log` 方法，再调用 `SkipNow` 方法。 |
| `Skipf(template, ...args)` | 此方法等同于先调用 `Logf` 方法，再调用 `SkipNow` 方法。 |
| `Skipped()` | 如果测试已被跳过，此方法返回 `true`。 |

`Run` 方法用于执行子测试，这是从单个函数运行一系列相关测试的便捷方式，如代码清单 31-10 所示。

```
package main
import (
"testing"
"sort"
"fmt"
)
func TestSum(t *testing.T) {
testValues := []int{ 10, 20, 30 }
_, sum := sortAndTotal(testValues)
expected := 60
if (sum != expected) {
t.Fatalf("Expected %v, Got %v", expected, sum)
}
}
func TestSort(t *testing.T) {
slices := [][]int {
{ 1, 279, 48, 12, 3 },
{ -10, 0, -10 },
{ 1, 2, 3, 4, 5, 6, 7 },
{ 1 },
}
for index, data := range slices {
t.Run(fmt.Sprintf("Sort #%v", index), func(subT *testing.T) {
sorted, _ := sortAndTotal(data)
if (!sort.IntsAreSorted(sorted)) {
subT.Fatalf("Unsorted data %v", sorted)
}
})
}
}
代码清单 31-10
在 tests 文件夹的 simple_test.go 文件中运行子测试
```

`Run` 方法的参数是测试的名称以及一个接受 `T` 结构体并执行测试的函数。在代码清单 31-10 中，`Run` 方法用于测试一组不同的 `int` 切片是否被正确排序。使用 `go test -v` 命令运行测试以显示详细输出，你将看到以下输出：

```
=== RUN   TestSum
--- PASS: TestSum (0.00s)
=== RUN   TestSort
=== RUN   TestSort/Sort_#0
=== RUN   TestSort/Sort_#1
=== RUN   TestSort/Sort_#2
=== RUN   TestSort/Sort_#3
--- PASS: TestSort (0.00s)
--- PASS: TestSort/Sort_#0 (0.00s)
--- PASS: TestSort/Sort_#1 (0.00s)
--- PASS: TestSort/Sort_#2 (0.00s)
--- PASS: TestSort/Sort_#3 (0.00s)
PASS
ok      tests   0.112s
```



#### 跳过测试

可以使用表 31-3 中描述的方法跳过测试，当某个测试失败导致执行相关测试意义不大时，此方法非常有用，如清单 31-11 所示。

```
package main
import (
"testing"
"sort"
"fmt"
)
type SumTest struct {
testValues []int
expectedResult int
}
func TestSum(t *testing.T) {
testVals := []SumTest {
{ testValues: []int{10, 20, 30}, expectedResult:  10},
{ testValues: []int{ -10, 0, -10 }, expectedResult:  -20},
{ testValues: []int{ -10, 0, -10 }, expectedResult:  -20},
}
for index, testVal := range testVals {
t.Run(fmt.Sprintf("Sum #%v", index), func(subT *testing.T) {
if (t.Failed()) {
subT.SkipNow()
}
_, sum := sortAndTotal(testVal.testValues)
if (sum != testVal.expectedResult) {
subT.Fatalf("Expected %v, Got %v", testVal.expectedResult, sum)
}
})
}
}
func TestSort(t *testing.T) {
slices := [][]int {
{ 1, 279, 48, 12, 3 },
{ -10, 0, -10 },
{ 1, 2, 3, 4, 5, 6, 7 },
{ 1 },
}
for index, data := range slices {
t.Run(fmt.Sprintf("Sort #%v", index), func(subT *testing.T) {
sorted, _ := sortAndTotal(data)
if (!sort.IntsAreSorted(sorted)) {
subT.Fatalf("Unsorted data %v", sorted)
}
})
}
}
清单 31-11
在 tests 文件夹的 simple_test.go 文件中跳过测试
```

`TestSum`函数已被重写以运行子测试。使用子测试时，如果任何单个测试失败，则整个测试也会失败。在清单 31-11 中，我通过调用整体测试的`T`结构上的`Failed`方法，并使用`SkipNow`方法在发生故障后跳过子测试来依赖此行为。`TestSum`执行的第一个子测试定义的预期结果不正确，导致测试失败，当使用`go test -v`命令时会产生以下输出：

```
=== RUN   TestSum
=== RUN   TestSum/Sum_#0
simple_test.go:27: Expected 10, Got 60
=== RUN   TestSum/Sum_#1
=== RUN   TestSum/Sum_#2
--- FAIL: TestSum (0.00s)
--- FAIL: TestSum/Sum_#0 (0.00s)
--- SKIP: TestSum/Sum_#1 (0.00s)
--- SKIP: TestSum/Sum_#2 (0.00s)
=== RUN   TestSort
=== RUN   TestSort/Sort_#0
=== RUN   TestSort/Sort_#1
=== RUN   TestSort/Sort_#2
=== RUN   TestSort/Sort_#3
--- PASS: TestSort (0.00s)
--- PASS: TestSort/Sort_#0 (0.00s)
--- PASS: TestSort/Sort_#1 (0.00s)
--- PASS: TestSort/Sort_#2 (0.00s)
--- PASS: TestSort/Sort_#3 (0.00s)
FAIL
exit status 1
FAIL    tests   0.138s
```

## 基准测试代码

名称以`Benchmark`开头、后跟以大写字母开头的单词（如`Sort`）的函数是基准测试，其执行将被计时。基准测试函数接收一个指向`testing.B`结构体的指针，该结构体定义了表 31-4 中描述的字段。

表 31-4  
由 `B` 结构体定义的字段

| 名称 | 描述 |
| --- | --- |
| `N` | 这个`int`字段指定基准测试函数应执行待测代码的次数。 |

`N`的值用于基准测试函数内部的`for`循环中，以重复执行待测代码。基准测试工具可能会使用不同的`N`值重复调用基准测试函数，以建立稳定的测量结果。在`tests`文件夹中添加一个名为`benchmark_test.go`的文件，内容如清单 31-12 所示。

决定何时进行基准测试

性能调优代码就像性能调优汽车：可能很有趣，通常很昂贵，而且几乎每次都会引发比解决的问题更多的问题。

任何项目中最昂贵的部分都是程序员时间，无论是在初始开发阶段还是在维护阶段。性能调优不仅会占用本可用于完成项目的时间，而且通常会导致代码更难理解，这将在未来消耗更多时间，因为其他开发人员试图理解你巧妙的优化。

我愿意接受有些项目具有特定的性能要求，但你的项目很可能不是其中之一。不过别担心，因为我的项目也没有那些要求。对于普通项目来说，购买更多服务器或存储容量比雇佣昂贵的开发人员进行调优更便宜。

基准测试可以具有教育意义，通过理解代码的执行方式，你可以了解很多关于项目的信息。但进行教育性基准测试的时间窗口很短暂，介于部署和第一个缺陷报告到来之间，否则这段时间你可能只能用来按颜色整理打印纸。在此之前，我的建议是专注于编写易于理解和易于维护的代码。

```
package main
import (
"testing"
"math/rand"
"time"
)
func BenchmarkSort(b *testing.B) {
rand.Seed(time.Now().UnixNano())
size := 250
data := make([]int, size)
for i := 0; i < b.N; i++ {
for j := 0; j < size; j++ {
data[j] = rand.Int()
}
sortAndTotal(data)
}
}
清单 31-12
tests 文件夹中 benchmark_test.go 文件的内容
```

`BenchmarkSort`函数创建一个包含随机数据的切片，并将其传递给在清单 31-2 中定义的`sortAndTotal`函数。要执行基准测试，请在`tests`文件夹中运行清单 31-13 所示的命令。

```
go test -bench . -run notest
清单 31-13
执行基准测试
```

`-bench`参数后的句点会让`go test`工具发现的所有基准测试都被执行。句点可以替换为正则表达式来选择特定的基准测试。默认情况下，单元测试也会被执行，但由于我在清单 31-12 的`TestSum`函数中引入了一个故意错误，我使用了`-run`参数指定了一个不会与项目中任何测试函数名称匹配的值，结果将只执行基准测试。

清单 31-13 中的命令会查找并执行`BenchmarkSort`函数，并产生类似如下的输出，具体内容因系统而异：

```
goos: windows
goarch: amd64
pkg: tests
BenchmarkSort-12           23853             42642 ns/op
PASS
ok      tests   1.577s
```

基准测试函数的名称后面跟着 CPU 或核心数，在我的系统上是 12，但这不会对测试结果产生影响，因为代码没有使用 goroutine：

```
...
BenchmarkSort-12           23853             42642 ns/op
...
```

下一个字段报告了传递给基准测试函数以生成这些结果的`N`值：

```
...
BenchmarkSort-12           23853             42642 ns/op
...
```

在我的系统上，测试工具以`N`值为 23853 运行了`BenchmarkSort`函数。这个数字会因测试和系统而异。最后一个值报告了基准循环每次迭代所花费的持续时间（以纳秒为单位）：

```
...
BenchmarkSort-12           23853             42642 ns/op
...
```

对于这次测试运行，基准测试完成了 42,642 纳秒。



### 从基准测试中移除准备工作

在`for`循环的每次迭代中，`BenchmarkSort`函数都必须生成随机数据，而生成这些数据所花费的时间被计入基准测试结果中。`B`结构体定义了表 31-5 中描述的方法，这些方法用于控制测试所用的计时器。

**表 31-5**
*用于计时控制的 B 方法*

| 名称 | 描述 |
| --- | --- |
| `StopTimer()` | 此方法停止计时器。 |
| `StartTimer()` | 此方法启动计时器。 |
| `ResetTimer()` | 此方法重置计时器。 |

当一个基准测试需要一些初始设置时，`ResetTimer`方法非常有用；而其他方法则在每次被测活动都有额外开销时发挥作用。清单 31-14 使用了这些方法，将准备工作从基准测试结果中排除。

```
package main
import (
"testing"
"math/rand"
"time"
)
func BenchmarkSort(b *testing.B) {
rand.Seed(time.Now().UnixNano())
size := 250
data := make([]int, size)
b.ResetTimer()
for i := 0; i < b.N; i++ {
b.StopTimer()
for j := 0; j < size; j++ {
data[j] = rand.Int()
}
b.StartTimer()
sortAndTotal(data)
}
}
```

**清单 31-14**
*在 tests 文件夹下的 benchmark_test.go 文件中控制计时器*

在设置了随机数种子并初始化了切片之后，计时器被重置。在`for`循环内部，使用`StopTimer`方法在切片填充随机数据前停止计时器，并使用`StartTimer`方法在调用`sortAndTotal`函数前启动计时器。在`tests`文件夹下运行清单 31-14 所示的命令，修改后的基准测试将被执行。在我的系统上，产生了以下结果：

```
goos: windows
goarch: amd64
pkg: tests
BenchmarkSort-12           35088             32095 ns/op
PASS
ok      tests   4.133s
```

排除了为基准测试做准备所需的工作后，对执行`sortAndTotal`函数所花费时间的评估更加准确了。

### 执行子基准测试

基准测试函数可以执行子基准测试，就像测试函数可以运行子测试一样。为方便参考，表 31-6 描述了用于运行子基准测试的方法。

**表 31-6**
*用于运行子基准测试的 B 方法*

| 名称 | 描述 |
| --- | --- |
| `Run(name, func)` | 调用此方法会将指定的函数作为子基准测试执行。该方法在基准测试执行期间会阻塞。 |

清单 31-15 更新了`BenchmarkSort`函数，使其对不同的数组大小执行一系列基准测试。

```
package main
import (
"testing"
"math/rand"
"time"
"fmt"
)
func BenchmarkSort(b *testing.B) {
rand.Seed(time.Now().UnixNano())
sizes := []int { 10, 100, 250 }
for _, size := range sizes {
b.Run(fmt.Sprintf("Array Size %v", size), func(subB *testing.B) {
data := make([]int, size)
subB.ResetTimer()
for i := 0; i < subB.N; i++ {
subB.StopTimer()
for j := 0; j < size; j++ {
data[j] = rand.Int()
}
subB.StartTimer()
sortAndTotal(data)
}
})
}
}
```

**清单 31-15**
*在 tests 文件夹下的 benchmarks_test.go 文件中执行子基准测试*

这些基准测试可能需要一些时间才能完成。以下是我系统上的结果，使用清单 31-13 所示的命令生成：

```
goos: windows
goarch: amd64
pkg: tests
BenchmarkSort/Array_Size_10-12            753120              1984 ns/op
BenchmarkSort/Array_Size_100-12           110248             10953 ns/op
BenchmarkSort/Array_Size_250-12            34369             31717 ns/op
PASS
ok      tests   61.453s
```

### 记录数据

`log`包提供了一个简单的日志记录 API，用于创建日志条目并将其发送到`io.Writer`，从而允许应用程序生成日志数据而不需要知道这些数据将存储在哪里。`log`包定义的最有用的函数如表 31-7 所述。

**表 31-7**
*有用的 log 函数*

| 名称 | 描述 |
| --- | --- |
| `Output()` | 此函数返回将接收日志消息的`Writer`。默认情况下，日志消息写入标准输出。 |
| `SetOutput(writer)` | 此函数使用指定的`Writer`进行日志记录。 |
| `Flags()` | 此函数返回用于格式化日志消息的标志。 |
| `SetFlags(flags)` | 此函数使用指定的标志来格式化日志消息。 |
| `Prefix()` | 此函数返回应用于日志消息的前缀。默认没有前缀。 |
| `SetPrefix(prefix)` | 此函数使用指定的`string`作为日志消息的前缀。 |
| `Output(depth, message)` | 此函数将指定的消息写入由`Output`函数返回的`Writer`，并带有指定的调用深度，默认为`2`。调用深度用于控制代码文件的选择，通常不会更改。 |
| `Print(...vals)` | 此函数通过调用`fmt.Sprint`并将结果传递给`Output`函数来创建日志消息。 |
| `Printf(template, ...vals)` | 此函数通过调用`fmt.Sprintf`并将结果传递给`Output`函数来创建日志消息。 |
| `Fatal(...vals)` | 此函数通过调用`fmt.Sprint`创建日志消息，将结果传递给`Output`函数，然后终止应用程序。 |
| `Fatalf(template, ...vals)` | 此函数通过调用`fmt.Sprintf`创建日志消息，将结果传递给`Output`函数，然后终止应用程序。 |
| `Panic(...vals)` | 此函数通过调用`fmt.Sprint`创建日志消息，然后将结果传递给`Output`函数，接着调用`panic`函数。 |
| `Panicf(template, ...vals)` | 此函数通过调用`fmt.Sprintf`创建日志消息，然后将结果传递给`Output`函数，接着调用`panic`函数。 |

日志消息的格式通过`SetFlags`函数控制，`log`包为它定义了表 31-8 中描述的常量。

**表 31-8**
*log 包常量*

| 名称 | 描述 |
| --- | --- |
| `Ldate` | 选择此标志会在日志输出中包含日期。 |
| `Ltime` | 选择此标志会在日志输出中包含时间。 |
| `Lmicroseconds` | 选择此标志会在时间中包含微秒。 |
| `Llongfile` | 选择此标志会包含记录消息的代码文件名（含目录）和行号。 |
| `Lshortfile` | 选择此标志会包含记录消息的代码文件名（不含目录）和行号。 |
| `LUTC` | 选择此标志会使用 UTC 表示日期和时间，而不是本地时区。 |
| `Lmsgprefix` | 选择此标志会将前缀从其默认位置（日志消息的开头）移动到紧靠传递给`Output`函数的字符串之前。 |
| `LstdFlags` | 此常量表示默认格式，即选择`Ldate`和`Ltime`。 |

清单 31-16 使用了表 31-7 中的函数来执行简单的日志记录。


```markdown

## 代码示例与日志记录

```go
package main
import (
"sort"
//"fmt"
"log"
)
func sortAndTotal(vals []int) (sorted []int, total int) {
sorted = make([]int, len(vals))
copy(sorted, vals)
sort.Ints(sorted)
for _, val := range sorted {
total += val
//total++
}
return
}
func main() {
nums := []int { 100, 20, 1, 7, 84 }
sorted, total := sortAndTotal(nums)
log.Print("Sorted Data: ", sorted)
log.Print("Total: ", total)
}
func init() {
log.SetFlags(log.Lshortfile | log.Ltime)
}
```

清单 31-16：在 `tests` 文件夹的 `main.go` 文件中记录日志消息

初始化函数使用 `SetFlags` 函数选择 `Lshortfile` 和 `Ltime` 标志，这些标志将在日志输出中包含文件名和时间。在 `main` 函数中，使用 `Print` 函数创建日志消息。使用 `go run .` 命令编译并执行项目，您将看到类似于以下的输出：

```
08:51:25 main.go:26: Sorted Data: [1 7 20 84 100]
08:51:25 main.go:27: Total: 212
```

### 创建自定义日志记录器

`log` 包可用于设置不同的日志选项，以便应用程序的不同部分可以写入不同目标或使用不同格式选项。表 31-9 描述了用于创建自定义日志目标的函数。

**表 31-9：** 用于自定义日志记录的 `log` 包函数

| 名称 | 描述 |
| --- | --- |
| `New(writer, prefix, flags)` | 此函数返回一个 `Logger`，它将消息写入指定的写入器，并配置了指定的前缀和标志。 |

`New` 函数的结果是一个 `Logger`，它是一个结构体，定义了与表 31-7 中描述的函数相对应的方法。表 31-7 中的函数只是调用默认记录器上同名的方法。清单 31-17 使用 `New` 函数创建一个 `Logger`。

```go
package main
import (
"sort"
//"fmt"
"log"
)
func sortAndTotal(vals []int) (sorted []int, total int) {
var logger = log.New(log.Writer(), "sortAndTotal: ",
log.Flags() | log.Lmsgprefix)
logger.Printf("Invoked with %v values", len(vals))
sorted = make([]int, len(vals))
copy(sorted, vals)
sort.Ints(sorted)
logger.Printf("Sorted data: %v", sorted)
for _, val := range sorted {
total += val
//total++
}
logger.Printf("Total: %v", total)
return
}
func main() {
nums := []int { 100, 20, 1, 7, 84 }
sorted, total := sortAndTotal(nums)
log.Print("Sorted Data: ", sorted)
log.Print("Total: ", total)
}
func init() {
log.SetFlags(log.Lshortfile | log.Ltime)
}
```

清单 31-17：在 `tests` 文件夹的 `main.go` 文件中创建自定义日志记录器

`Logger` 结构体使用新的前缀和添加的 `Lmsgprefix` 标志创建，并使用从表 31-7 中描述的 `Output` 函数获得的 `Writer`。结果是日志消息仍然写入同一目标，但带有一个额外的前缀，表示来自 `sortAndTotal` 函数的消息。编译并执行项目，您将看到额外的日志消息：

```
09:12:37 main.go:11: sortAndTotal: Invoked with 5 values
09:12:37 main.go:15: sortAndTotal: Sorted data: [1 7 20 84 100]
09:12:37 main.go:20: sortAndTotal: Total: 212
09:12:37 main.go:27: Sorted Data: [1 7 20 84 100]
09:12:37 main.go:28: Total: 212
```

## 总结

在本章中，我通过单元测试、基准测试和日志记录完成了对最有用的标准库包的描述。正如我所解释的，我觉得测试功能不够吸引人，并且对基准测试有强烈保留意见，但这两组功能都很好地集成到了 Go 工具中，这使得它们更容易使用，如果您在这些主题上的观点与我不一致的话。日志记录功能争议较少，我在第三部分中创建的自定义 Web 应用程序平台中使用了它们。

# 第三部分：应用 Go

# 32. 创建 Web 平台

在本章中，我开始开发一个自定义 Web 应用程序平台，并在第 33 章和第 34 章中继续此开发。在第 35 章至第 38 章中，我使用该平台创建一个名为 SportsStore 的应用程序，该应用程序以某种形式出现在我的几乎所有书籍中。

本部分书籍的目的是展示 Go 被用于解决实际开发项目中出现的各种问题。对于 Web 应用程序平台，这意味着创建日志记录、会话、HTML 模板、授权等功能。对于 SportsStore 应用程序，这意味着使用产品数据库、跟踪用户的产品选择、验证用户输入以及结账。

请记住，这些章节中的代码是专门为本书编写的，并且仅测试到后续章节中的功能按预期工作的程度。有一些优秀的第三方包提供了本部分中创建的部分或全部功能，这些是开始您项目的良好起点。我推荐 Gorilla Web Toolkit（[`www.gorillatoolkit.org`](http://www.gorillatoolkit.org)），它提供了一些有用的包（我在第 34 章中使用了其中一个包）。

> **注意：** 这些章节内容高级且复杂，请务必完全按照所示示例进行操作。如果您遇到困难，应首先检查本书在 GitHub 仓库（[`https://github.com/apress/pro-go`](https://github.com/apress/pro-go)）中的勘误表，我将在其中列出任何出现问题的解决方案。

## 创建项目

打开命令提示符，导航到合适的位置，并创建一个名为 `platform` 的新目录。导航到 `platform` 目录并运行清单 32-1 中所示的命令。

> **提示：** 您可以从 [`https://github.com/apress/pro-go`](https://github.com/apress/pro-go) 下载本章以及本书所有其他章节的示例项目。有关如何在运行示例时遇到问题获得帮助，请参见第 2 章。

```
go mod init platform
```

清单 32-1：初始化项目

在 `platform` 文件夹中添加一个名为 `main.go` 的文件，内容如清单 32-2 所示。

```go
package main
import (
"fmt"
)
func writeMessage() {
fmt.Println("Hello, Platform")
}
func main() {
writeMessage()
}
```

清单 32-2：`platform` 文件夹中 `main.go` 文件的内容

在 `platform` 文件夹中运行清单 32-3 中所示的命令。

```
go run .
```

清单 32-3：编译并执行项目

项目将被编译并执行，并产生以下输出：

```
Hello, Platform
```

## 创建一些基本平台功能

首先，我将定义一些基本服务，这些服务将为运行 Web 应用程序提供基础。

```


### 创建日志系统

需要实现的第一个服务器特性是日志记录。Go 标准库中的`log`包为创建日志提供了一套不错的基础功能，但还需要一些额外特性来筛选消息的详细程度。请创建`platform/logging`文件夹，并在其中添加一个名为`logging.go`的文件，文件内容如清单 32-4 所示。

```
package logging

type LogLevel int

const (
    Trace LogLevel = iota
    Debug
    Information
    Warning
    Fatal
    None
)

type Logger interface {
    Trace(string)
    Tracef(string, ...interface{})
    Debug(string)
    Debugf(string, ...interface{})
    Info(string)
    Infof(string, ...interface{})
    Warn(string)
    Warnf(string, ...interface{})
    Panic(string)
    Panicf(string, ...interface{})
}
```

该文件定义了`Logger`接口，该接口指定了用于记录不同严重级别消息的方法，严重级别通过`LogLevel`值设置，其范围从`Trace`到`Fatal`。此外还有一个`None`级别，表示不输出任何日志。对于每个严重级别，`Logger`接口定义了一个接受简单字符串的方法和一个接受模板字符串及占位符值的方法。

我为平台提供的所有功能定义了接口，并使用这些接口提供默认实现。这将允许应用程序在需要时替换默认实现，并且还能以服务的形式向应用程序提供功能，我将在本章后面部分进行描述。

为了创建`Logger`接口的默认实现，请在`logging`文件夹中添加一个名为`logger_default.go`的文件，文件内容如清单 32-5 所示。

```
package logging

import (
    "log"
    "fmt"
)

type DefaultLogger struct {
    minLevel    LogLevel
    loggers     map[LogLevel]*log.Logger
    triggerPanic bool
}

func (l *DefaultLogger) MinLogLevel() LogLevel {
    return l.minLevel
}

func (l *DefaultLogger) write(level LogLevel, message string) {
    if (l.minLevel <= level) {
        l.loggers[level].Output(2, message)
    }
}

func (l *DefaultLogger) Trace(msg string) {
    l.write(Trace, msg)
}

func (l *DefaultLogger) Tracef(template string, vals ...interface{}) {
    l.write(Trace, fmt.Sprintf(template, vals...))
}

func (l *DefaultLogger) Debug(msg string) {
    l.write(Debug, msg)
}

func (l *DefaultLogger) Debugf(template string, vals ...interface{}) {
    l.write(Debug, fmt.Sprintf(template, vals...))
}

func (l *DefaultLogger) Info(msg string) {
    l.write(Information, msg)
}

func (l *DefaultLogger) Infof(template string, vals ...interface{}) {
    l.write(Information, fmt.Sprintf(template, vals...))
}

func (l *DefaultLogger) Warn(msg string) {
    l.write(Warning, msg)
}

func (l *DefaultLogger) Warnf(template string, vals ...interface{}) {
    l.write(Warning, fmt.Sprintf(template, vals...))
}

func (l *DefaultLogger) Panic(msg string) {
    l.write(Fatal, msg)
    if (l.triggerPanic) {
        panic(msg)
    }
}

func (l *DefaultLogger) Panicf(template string, vals ...interface{}) {
    formattedMsg := fmt.Sprintf(template, vals...)
    l.write(Fatal, formattedMsg)
    if (l.triggerPanic) {
        panic(formattedMsg)
    }
}
```

`DefaultLogger`结构体使用标准库中`log`包提供的功能实现了`Logger`接口，该`log`包在第 31 章中有所描述。每个严重级别都分配了一个`log.Logger`，这意味着消息可以发送到不同的目的地或以不同的方式格式化。请将名为`default_create.go`的文件添加到`logging`文件夹，其代码如清单 32-6 所示。

```
package logging

import (
    "log"
    "os"
)

func NewDefaultLogger(level LogLevel) Logger {
    flags := log.Lmsgprefix | log.Ltime
    return &DefaultLogger {
        minLevel: level,
        loggers: map[LogLevel]*log.Logger {
            Trace:       log.New(os.Stdout, "TRACE ",  flags),
            Debug:       log.New(os.Stdout, "DEBUG ",  flags),
            Information: log.New(os.Stdout, "INFO ",  flags),
            Warning:     log.New(os.Stdout, "WARN ",  flags),
            Fatal:       log.New(os.Stdout, "FATAL ",  flags),
        },
        triggerPanic: true,
    }
}
```

`NewDefaultLogger`函数创建了一个`DefaultLogger`，它包含一个最低严重级别以及将消息写入标准输出的`log.Loggers`。作为简单测试，清单 32-7 修改了`main`函数，使其使用日志功能来输出消息。

```
package main

import (
    //"fmt"
    "platform/logging"
)

func writeMessage(logger logging.Logger) {
    logger.Info("Hello, Platform")
}

func main() {
    var logger logging.Logger = logging.NewDefaultLogger(logging.Information)
    writeMessage(logger)
}
```

`NewDefaultLogger`创建的日志记录器的最低严重级别设置为`Information`，这意味着严重级别较低的消息（`Trace`和`Debug`）将被丢弃。编译并执行该项目，你将看到以下输出，尽管时间戳可能不同：

```
18:28:46 INFO Hello, Platform
```

### 创建配置系统

下一步是添加配置应用程序的能力，这样就不必在代码文件中定义设置。请创建`platform/config`文件夹，并在其中添加一个名为`config.go`的文件，文件内容如清单 32-8 所示。

```
package config

type Configuration interface {
    GetString(name string) (configValue string, found bool)
    GetInt(name string) (configValue int, found bool)
    GetBool(name string) (configValue bool, found bool)
    GetFloat(name string) (configValue float64, found bool)
    GetStringDefault(name, defVal string) (configValue string)
    GetIntDefault(name string, defVal int) (configValue int)
    GetBoolDefault(name string, defVal bool) (configValue bool)
    GetFloatDefault(name string, defVal float64) (configValue float64)
    GetSection(sectionName string) (section Configuration, found bool)
}
```

`Configuration`接口定义了用于获取配置设置的方法，支持获取`string`、`int`、`float64`和`bool`类型的值。还有一组方法允许提供默认值。配置数据将支持嵌套的配置节，可以通过`GetSection`方法获取。

#### 定义配置文件

如果你能看到我将要使用的配置文件类型，那么理解配置系统的实现会更容易。请将名为`config.json`的文件添加到`platform`文件夹，其内容如清单 32-9 所示。

```
{
    "logging" : {
        "level": "debug"
    },
    "main" : {
        "message" : "Hello from the config file"
    }
}
```

该配置文件定义了两个配置节，分别名为`logging`和`main`。`logging`节包含一个名为`level`的`string`类型配置设置。`main`节包含一个名为`message`的`string`类型配置设置。随着我为平台添加功能以及开始处理 SportsStore 应用程序时，我会添加更多的配置设置，但该文件展示了配置文件使用的基本结构。添加配置设置时，请特别注意引号和逗号，JSON 格式对这两者都有要求，但很容易遗漏。



### 实现配置接口

要创建`Configuration`接口的实现，请在`config`文件夹中添加一个名为`config_default.go`的文件，其内容如清单 32-10 所示。

```go
package config
import "strings"
type DefaultConfig struct {
configData map[string]interface{}
}
func (c *DefaultConfig) get(name string) (result interface{}, found bool) {
data := c.configData
for _, key := range strings.Split(name, ":") {
result, found = data[key]
if newSection, ok := result.(map[string]interface{}); ok && found {
data = newSection
} else {
return
}
}
return
}
func (c *DefaultConfig) GetSection(name string) (section Configuration, found bool) {
value, found := c.get(name)
if (found) {
if sectionData, ok := value.(map[string]interface{}) ; ok {
section = &DefaultConfig { configData: sectionData }
}
}
return
}
func (c *DefaultConfig) GetString(name string) (result string, found bool) {
value, found := c.get(name)
if (found) { result = value.(string) }
return
}
func (c *DefaultConfig) GetInt(name string) (result int, found bool) {
value, found := c.get(name)
if (found) { result =  int(value.(float64)) }
return
}
func (c *DefaultConfig) GetBool(name string) (result bool, found bool) {
value, found := c.get(name)
if (found) { result = value.(bool) }
return
}
func (c *DefaultConfig) GetFloat(name string) (result float64, found bool) {
value, found := c.get(name)
if (found) { result = value.(float64) }
return
}
```

`DefaultConfig`结构体使用`map`实现了`Configuration`接口。嵌套的配置节也以映射形式表示。通过用冒号分隔节名与设置名，可以请求单个配置项（如`logging:level`），也可以使用节名（如`logging`）请求包含所有设置的映射。为了定义接受默认值的方法，请在`config`文件夹中添加一个名为`config_default_fallback.go`的文件，其内容如清单 32-11 所示。

```go
package config
func (c *DefaultConfig) GetStringDefault(name, val string) (result string) {
result, ok := c.GetString(name)
if !ok {
result = val
}
return
}
func (c *DefaultConfig) GetIntDefault(name string, val int) (result int) {
result, ok := c.GetInt(name)
if !ok {
result = val
}
return
}
func (c *DefaultConfig) GetBoolDefault(name string, val bool) (result bool) {
result, ok := c.GetBool(name)
if !ok {
result = val
}
return
}
func (c *DefaultConfig) GetFloatDefault(name string, val float64) (result float64) {
result, ok := c.GetFloat(name)
if !ok {
result = val
}
return
}
```

为了定义从配置文件中加载数据的函数，请在`config`文件夹中添加一个名为`config_json.go`的文件，其内容如清单 32-12 所示。

```go
package config
import (
"os"
"strings"
"encoding/json"
)
func Load(fileName string) (config Configuration,  err error) {
var data []byte
data, err = os.ReadFile(fileName)
if (err == nil) {
decoder := json.NewDecoder(strings.NewReader(string(data)))
m := map[string]interface{} {}
err = decoder.Decode(&m)
if (err == nil) {
config = &DefaultConfig{ configData: m }
}
}
return
}
```

`Load`函数读取文件内容，将其中的 JSON 解码为一个映射，然后使用该映射创建一个`DefaultConfig`值。

### 使用配置系统

为了从配置系统中获取日志级别，请对`logging`文件夹中的`default_create.go`文件进行清单 32-13 所示的修改。

```go
package logging
import (
"log"
"os"
"strings"
"platform/config"
)
func NewDefaultLogger(cfg config.Configuration) Logger {
var level LogLevel = Debug
if configLevelString, found := cfg.GetString("logging:level"); found {
level = LogLevelFromString(configLevelString)
}
flags := log.Lmsgprefix | log.Ltime
return &DefaultLogger {
minLevel: level,
loggers: map[LogLevel]*log.Logger {
Trace: log.New(os.Stdout, "TRACE ",  flags),
Debug: log.New(os.Stdout, "DEBUG ",  flags),
Information: log.New(os.Stdout, "INFO ",  flags),
Warning: log.New(os.Stdout, "WARN ",  flags),
Fatal: log.New(os.Stdout, "FATAL ",  flags),
},
triggerPanic: true,
}
}
func LogLevelFromString(val string) (level LogLevel) {
switch strings.ToLower(val) {
case "debug":
level = Debug
case "information":
level = Information
case "warning":
level = Warning
case "fatal":
level = Fatal
case "none":
level = None
default:
level = Debug
}
return
}
```

在 JSON 中无法很好地表示`iota`值，因此我使用了字符串，并定义了`LogLevelFromString`函数来将配置设置转换为`LogLevel`值。清单 32-14 更新了`main`函数，以加载并应用配置数据，并使用配置系统读取其写出的消息。

```go
package main
import (
//"fmt"
"platform/config"
"platform/logging"
)
func writeMessage(logger logging.Logger, cfg config.Configuration) {
section, ok := cfg.GetSection("main")
if (ok) {
message, ok := section.GetString("message")
if (ok) {
logger.Info(message)
} else {
logger.Panic("Cannot find configuration setting")
}
} else {
logger.Panic("Config section not found")
}
}
func main() {
var cfg config.Configuration
var err error
cfg, err = config.Load("config.json")
if (err != nil) {
panic(err)
}
var logger logging.Logger = logging.NewDefaultLogger(cfg)
writeMessage(logger, cfg)
}
```

配置从`config.json`文件中加载，然后将`Configuration`实现传递给`NewDefaultLogger`函数，该函数使用它来读取日志级别设置。

`writeMessage`函数演示了如何使用配置节，这可以很好地为组件提供其所需的设置，尤其当需要多个具有不同设置的实例时，每个实例都可以在其自己的节中定义。

编译并执行清单 32-14 中的代码，会产生以下输出：

```
18:49:12 INFO Hello from the config file
```



### 使用依赖注入管理服务

为了获取 `Logger` 和 `Configuration` 接口的实现，`main` 函数中的代码需要知道如何创建实现这些接口的结构体实例：

```
...
cfg, err = config.Load("config.json")
...
var logger logging.Logger = logging.NewDefaultLogger(cfg)
...
```

这是一种可行的方法，但它违背了定义接口的初衷，需要谨慎确保实例创建的一致性，并使替换接口实现的过程复杂化。

我更推荐的方法是使用**依赖注入 (DI)**，在这种模式下，依赖接口的代码无需选择底层类型或直接创建实例就能获取实现。我将先从**服务定位**说起，这为后续更高级的功能奠定基础。

在应用程序启动时，应用定义的接口将连同创建实现结构体实例的工厂函数一起被添加到一个注册器中。例如，`platform.logger.Logger` 接口将注册一个调用 `NewDefaultLogger` 函数的工厂函数。当一个接口被添加到注册器时，它就被称为一个**服务**。

在程序执行期间，需要服务所描述功能的应用程序组件会访问注册器，请求它们需要的接口。注册器调用工厂函数并返回创建的结构体，这使得应用程序组件能够使用接口的功能，而无需知道或指定使用哪个实现结构体以及如何创建它。如果这一点现在还不太清楚，别担心——这可能是一个难以理解的话题，在实际应用中会变得更容易理解。

## 定义服务生命周期

服务注册时带有生命周期，它指定了何时调用工厂函数来创建新的结构体值。我将使用三种服务生命周期，如表 32-1 所述。

**表 32-1** 服务生命周期

| 生命周期 | 描述 |
| --- | --- |
| `Transient` | 对于此生命周期，每次服务请求都会调用工厂函数。 |
| `Singleton` | 对于此生命周期，工厂函数只调用一次，所有请求都接收同一个结构体实例。 |
| `Scoped` | 对于此生命周期，工厂函数在作用域内的第一个请求时被调用一次，该作用域内的所有请求都接收同一个结构体实例。 |

创建 `platform/services` 文件夹，并在其中添加一个名为 `lifecycles.go` 的文件，内容如清单 32-15 所示。

```
package services
type lifecycle int
const (
Transient lifecycle = iota
Singleton
Scoped
)
Listing 32-15
The Contents of the lifecycles.go File in the services Folder
```

我将使用标准库中的 `context` 包来实现 `Scoped` 生命周期，该包在第 30 章中有描述。服务器收到的每个 HTTP 请求都会自动创建一个 `Context`，这意味着处理该请求的所有请求处理代码都可以共享同一组服务。例如，一个提供会话信息的单一结构体可以在整个给定请求的处理过程中被使用。

为了更容易地使用上下文，在 `services` 文件夹中添加一个名为 `context.go` 的文件，内容如清单 32-16 所示。

```
package services
import (
"context"
"reflect"
)
const ServiceKey = "services"
type serviceMap map[reflect.Type]reflect.Value
func NewServiceContext(c context.Context) context.Context {
if (c.Value(ServiceKey) == nil) {
return context.WithValue(c, ServiceKey, make(serviceMap))
} else {
return c
}
}
Listing 32-16
The Contents of the context.go File in the services Folder
```

`NewServiceContext` 函数使用 `WithValue` 函数派生一个上下文，并添加一个存储已解析服务的映射。有关派生上下文的不同方式的详细信息，请参见第 30 章。



### 定义内部服务函数

我将通过检查一个工厂函数并使用其结果来确定它处理的接口，来处理服务注册。以下是注册新服务时将使用的工厂函数类型示例：

```go
...
func ConfigurationFactory() config.Configuration {
// TODO 创建实现 Configuration 接口的结构体
}
...
```

此函数的结果类型是 `config.Configuration`。使用反射检查该函数，我可以获取结果类型并确定其对应的工厂接口。

某些工厂函数会依赖其他服务。以下是另一个工厂函数示例：

```go
...
func Loggerfactory(cfg config.Configuration) logging.Logger {
// TODO 创建实现 Logger 接口的结构体
}
...
```

这个工厂函数解析对 `Logger` 接口的请求，但它依赖于 `Configuration` 接口的实现。这意味着必须先解析 `Configuration` 接口，以便提供解析 `Logger` 接口所需的参数。这是一个*依赖注入*的示例，其中工厂函数所依赖的参数被解析，以便能够调用该函数。

**注意**：定义依赖其他服务的工厂函数可能会改变嵌套服务的生命周期。例如，如果你定义了一个依赖瞬态服务的单例服务，那么嵌套服务只会在单例首次实例化时被解析一次。在大多数项目中这不是问题，但需要谨记这一点。

在 `services` 文件夹中添加一个名为 `core.go` 的文件，内容如代码清单 32-17 所示。

```go
package services
import (
"reflect"
"context"
"fmt"
)
type BindingMap struct {
factoryFunc reflect.Value
lifecycle
}
var services = make(map[reflect.Type]BindingMap)
func addService(life lifecycle, factoryFunc interface{}) (err error) {
factoryFuncType := reflect.TypeOf(factoryFunc)
if factoryFuncType.Kind() == reflect.Func && factoryFuncType.NumOut() == 1 {
services[factoryFuncType.Out(0)] = BindingMap{
factoryFunc: reflect.ValueOf(factoryFunc),
lifecycle: life,
}
} else {
err = fmt.Errorf("类型不能用作服务: %v", factoryFuncType)
}
return
}
var contextReference = (*context.Context)(nil)
var contextReferenceType = reflect.TypeOf(contextReference).Elem()
func resolveServiceFromValue(c context.Context, val reflect.Value) (err error ){
serviceType := val.Elem().Type()
if serviceType == contextReferenceType {
val.Elem().Set(reflect.ValueOf(c))
} else if binding, found := services[serviceType]; found {
if (binding.lifecycle == Scoped) {
resolveScopedService(c, val, binding)
} else {
val.Elem().Set(invokeFunction(c, binding.factoryFunc)[0])
}
} else {
err = fmt.Errorf("找不到服务 %v", serviceType)
}
return
}
func resolveScopedService(c context.Context, val reflect.Value,
binding BindingMap) (err error) {
sMap, ok := c.Value(ServiceKey).(serviceMap)
if (ok) {
serviceVal, ok := sMap[val.Type()]
if (!ok) {
serviceVal = invokeFunction(c, binding.factoryFunc)[0]
sMap[val.Type()] = serviceVal
}
val.Elem().Set(serviceVal)
} else {
val.Elem().Set(invokeFunction(c, binding.factoryFunc)[0])
}
return
}
func resolveFunctionArguments(c context.Context,  f reflect.Value,
otherArgs ...interface{}) []reflect.Value {
params := make([]reflect.Value, f.Type().NumIn())
i := 0
if (otherArgs != nil) {
for ; i < len(otherArgs); i++ {
params[i] = reflect.ValueOf(otherArgs[i])
}
}
for ; i < len(params); i++ {
pType := f.Type().In(i)
pVal := reflect.New(pType)
err := resolveServiceFromValue(c, pVal)
if err != nil {
panic(err)
}
params[i] = pVal.Elem()
}
return params
}
func invokeFunction(c context.Context, f reflect.Value,
otherArgs ...interface{}) []reflect.Value {
return f.Call(resolveFunctionArguments(c, f, otherArgs...))
}
```

*代码清单 32-17：`services` 文件夹中 `core.go` 文件的内容*

`BindingMap` 结构体表示一个工厂函数（以 `reflect.Value` 形式表示）与一个生命周期的组合。`addService` 函数用于注册服务，它通过创建 `BindingMap` 并将其添加到分配给 services 变量的映射中来实现。

调用 `resolveServiceFromValue` 函数来解析服务，其参数是一个 `Context` 和一个 `Value`，该 `Value` 是指向变量的指针，变量的类型是要解析的接口（当你看到服务解析的实际过程时，这将更有意义）。为了解析服务，`resolveServiceFromValue` 函数会使用请求的类型作为键，检查 services 映射中是否存在 `BindingMap`。如果存在 `BindingMap`，则调用其工厂函数，并通过指针赋值。

`invokeFunction` 函数负责调用工厂函数，它使用 `resolveFunctionArguments` 函数来检查工厂函数的参数并解析每个参数。这些函数接受可选的附加参数，当需要以服务和常规值参数的混合方式调用函数时，会使用这些参数（在这种情况下，需要常规值的参数必须先定义）。

作用域服务需要特殊处理。`resolveScopedService` 函数会检查 `Context` 是否包含之前请求解析该服务的值。如果没有，则会解析该服务并将其添加到 `Context` 中，以便在同一作用域内复用。

### 定义服务注册函数

代码清单 32-17 中定义的函数均未导出。为了创建应用程序其余部分中用于注册服务的函数，请在 `services` 文件夹中添加一个名为 `registration.go` 的文件，内容如代码清单 32-18 所示。

```go
package services
import (
"reflect"
"sync"
)
func AddTransient(factoryFunc interface{}) (err error) {
return addService(Transient, factoryFunc)
}
func AddScoped(factoryFunc interface{}) (err error) {
return addService(Scoped, factoryFunc)
}
func AddSingleton(factoryFunc interface{}) (err error) {
factoryFuncVal := reflect.ValueOf(factoryFunc)
if factoryFuncVal.Kind() == reflect.Func && factoryFuncVal.Type().NumOut() == 1 {
var results []reflect.Value
once := sync.Once{}
wrapper := reflect.MakeFunc(factoryFuncVal.Type(),
func ([]reflect.Value) []reflect.Value {
once.Do(func() {
results = invokeFunction(nil, factoryFuncVal)
})
return results
})
err = addService(Singleton, wrapper.Interface())
}
return
}
```

*代码清单 32-18：`services` 文件夹中 `registration.go` 文件的内容*

`AddTransient` 和 `AddScoped` 函数只是将工厂函数传递给 `addService` 函数。单例生命周期需要稍多一些工作，`AddSingleton` 函数创建了一个围绕工厂函数的包装器，确保该工厂函数只在首次请求解析服务时执行一次。这保证了只会创建实现结构体的一个实例，并且直到首次需要它时才会创建。



### 定义服务解析函数

接下来的一组功能包括允许解析服务的函数。在 `services` 文件夹中添加一个名为 `resolution.go` 的文件，其内容如代码清单 32-19 所示。

```
package services
import (
"reflect"
"errors"
"context"
)
func GetService(target interface{}) error {
return GetServiceForContext(context.Background(), target)
}
func GetServiceForContext(c context.Context, target interface{}) (err error) {
targetValue := reflect.ValueOf(target)
if targetValue.Kind() == reflect.Ptr &&
targetValue.Elem().CanSet() {
err = resolveServiceFromValue(c, targetValue)
} else {
err = errors.New("Type cannot be used as target")
}
return
}
```

`GetServiceForContext` 接受一个上下文和一个指向可使用反射设置值的指针。为方便起见，`GetService` 函数使用后台上下文来解析服务。

### 注册和使用服务

基本服务功能已就绪，这意味着可以注册服务然后解析它们。在 `services` 文件夹中添加一个名为 `services_default.go` 的文件，其内容如代码清单 32-20 所示。

```
package services
import (
"platform/logging"
"platform/config"
)
func RegisterDefaultServices() {
err := AddSingleton(func() (c config.Configuration) {
c, loadErr :=  config.Load("config.json")
if (loadErr != nil) {
panic(loadErr)
}
return
})
err = AddSingleton(func(appconfig config.Configuration) logging.Logger {
return logging.NewDefaultLogger(appconfig)
})
if (err != nil) {
panic(err)
}
}
```

`RegisterDefaultServices` 创建了 `Configuration` 和 `Logger` 服务。这些服务是使用 `AddSingleton` 函数创建的，这意味着实现每个接口的结构体的单个实例将由整个应用程序共享。代码清单 32-21 更新了 `main` 函数以使用服务，而不是直接实例化结构体。

```
package main
import (
//"fmt"
"platform/config"
"platform/logging"
"platform/services"
)
func writeMessage(logger logging.Logger, cfg config.Configuration) {
section, ok := cfg.GetSection("main")
if (ok) {
message, ok := section.GetString("message")
if (ok) {
logger.Info(message)
} else {
logger.Panic("Cannot find configuration setting")
}
} else {
logger.Panic("Config section not found")
}
}
func main() {
services.RegisterDefaultServices()
var cfg config.Configuration
services.GetService(&cfg)
var logger logging.Logger
services.GetService(&logger)
writeMessage(logger, cfg)
}
```

解析服务是通过传递一个指向接口类型变量的指针来完成的。在代码清单 32-21 中，使用 `GetService` 函数获取 `Repository` 和 `Logger` 接口的实现，无需知道将使用哪个结构体类型、创建它的过程或服务生命周期。

解析服务需要两个步骤：创建变量和传递指针。编译并执行项目，您将看到以下输出：

```
19:17:06 INFO Hello from the config file
```

#### 添加对调用函数的支持

一旦基本服务功能就绪，就很容易创建增强功能，使服务解析更简单、更容易。为了添加对直接执行函数的支持，在 `services` 文件夹中添加一个名为 `functions.go` 的文件，其内容如代码清单 32-22 所示。

```
package services
import (
"reflect"
"errors"
"context"
)
func Call(target interface{}, otherArgs ...interface{}) ([]interface{}, error) {
return CallForContext(context.Background(), target, otherArgs...)
}
func CallForContext(c context.Context, target interface{}, otherArgs ...interface{}) (results []interface{}, err error) {
targetValue := reflect.ValueOf(target)
if (targetValue.Kind() == reflect.Func) {
resultVals := invokeFunction(c, targetValue, otherArgs...)
results = make([]interface{}, len(resultVals))
for i := 0; i < len(resultVals); i++ {
results[i] = resultVals[i].Interface()
}
} else {
err = errors.New("Only functions can be invoked")
}
return
}
```

`CallForContext` 函数接收一个函数，并使用服务来生成用作函数调用参数的值。`Call` 函数是一个便利函数，用于当没有 `Context` 可用时。此功能的实现依赖于代码清单 32-22 中用于调用工厂函数的代码。代码清单 32-23 演示了直接调用函数如何简化服务的使用。

```
package main
import (
//"fmt"
"platform/config"
"platform/logging"
"platform/services"
)
func writeMessage(logger logging.Logger, cfg config.Configuration) {
section, ok := cfg.GetSection("main")
if (ok) {
message, ok := section.GetString("message")
if (ok) {
logger.Info(message)
} else {
logger.Panic("Cannot find configuration setting")
}
} else {
logger.Panic("Config section not found")
}
}
func main() {
services.RegisterDefaultServices()
// var cfg config.Configuration
// services.GetService(&cfg)
// var logger logging.Logger
// services.GetService(&logger)
services.Call(writeMessage)
}
```

函数被传递给 `Call`，后者将检查其参数并使用服务解析它们。（请注意，函数名后面没有括号，因为那样会调用函数，而不是将其传递给 `services.Call`。）不再需要直接请求服务，可以依赖 `services` 包来处理细节。编译并执行代码，您将看到以下输出：

```
19:19:08 INFO Hello from the config file
```



#### 为结构体字段添加依赖解析支持

我计划为`service`包添加的最后一个功能，是解析对结构体字段的依赖关系。在`services`文件夹中新建一个名为`structs.go`的文件，内容如代码清单 32-24 所示。

```go
package services
import (
"reflect"
"errors"
"context"
)
func Populate(target interface{}) error {
return PopulateForContext(context.Background(), target)
}
func PopulateForContext(c context.Context, target interface{}) (err error) {
return PopulateForContextWithExtras(c, target,
make(map[reflect.Type]reflect.Value))
}
func PopulateForContextWithExtras(c context.Context, target interface{},
extras map[reflect.Type]reflect.Value) (err error) {
targetValue := reflect.ValueOf(target)
if targetValue.Kind() == reflect.Ptr &&
targetValue.Elem().Kind() == reflect.Struct {
targetValue = targetValue.Elem()
for i := 0; i < targetValue.Type().NumField(); i++ {
fieldVal := targetValue.Field(i)
if fieldVal.CanSet() {
if extra, ok := extras[fieldVal.Type()]; ok {
fieldVal.Set(extra)
} else {
resolveServiceFromValue(c,  fieldVal.Addr() )
}
}
}
} else {
err = errors.New("Type cannot be used as target")
}
return
}
```

*代码清单 32-24：services 文件夹中 structs.go 文件的内容*

这些函数会检查结构体定义的字段，并尝试使用已定义的服务来解析它们。任何类型不是接口或没有对应服务的字段都会被跳过。`PopulateForContextWithExtras`函数允许为结构体字段提供额外的值。

代码清单 32-25 定义了一个结构体，其字段声明了对服务的依赖关系。

```go
package main
import (
//"fmt"
"platform/config"
"platform/logging"
"platform/services"
)
func writeMessage(logger logging.Logger, cfg config.Configuration) {
section, ok := cfg.GetSection("main")
if (ok) {
message, ok := section.GetString("message")
if (ok) {
logger.Info(message)
} else {
logger.Panic("Cannot find configuration setting")
}
} else {
logger.Panic("Config section not found")
}
}
func main() {
services.RegisterDefaultServices()
services.Call(writeMessage)
val := struct {
message string
logging.Logger
}{
message: "Hello from the struct",
}
services.Populate(&val)
val.Logger.Debug(val.message)
}
```

*代码清单 32-25：在 platform 文件夹的 main.go 文件中注入结构体依赖*

`main`函数定义了一个匿名结构体，并通过将指针传递给`Populate`函数来解析其所需的服务。结果是，嵌入的`Logger`字段通过服务进行了填充。`Populate`函数跳过了`message`字段，但在结构体初始化时已为其定义了值。编译并执行项目，你将看到以下输出：

```
19:21:43 INFO Hello from the config file
19:21:43 DEBUG Hello from the struct
```

## 本章小结

在本章中，我开始了自定义 Web 应用平台的开发。我创建了日志记录和配置功能，并添加了对服务和依赖注入的支持。在下一章中，我将继续开发，创建请求处理管道和自定义模板系统。

