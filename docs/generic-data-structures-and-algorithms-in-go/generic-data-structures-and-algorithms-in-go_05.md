# 1. Go 语言泛型与并发初览

本章介绍 Go 语言中泛型的语法和语义。通过大量代码示例，我们将展示这一强大新特性的应用，为全书后续使用泛型奠定基础。

本章还将回顾 Go 语言中的并发机制。通过大量代码示例与基准测试，对比使用并发算法与不使用并发算法时的性能差异，为全书后续使用并发技术打下基础。

## 1.1 Go 语言简史与概述

Go 是一种相对较新的编程语言，于 2009 年底发布，由谷歌的罗伯特·格瑞史默（Robert Griesemer，瑞士计算机科学家，曾参与谷歌 V8 JavaScript 引擎的开发）、罗布·派克（Rob Pike，加拿大计算机科学家，曾是贝尔实验室 Unix 团队成员，Limbo 编程语言的创造者）以及肯·汤普逊（Ken Thompson，Unix 操作系统和 B 编程语言的创造者）共同开发。

Go 编程语言有时也被称为 Golang。为什么？因为该语言发布时 `go.org` 域名不可用，于是便诞生了 `golang.org`（Go 与 language 的组合）。该语言的官方名称是 Go，但 Twitter 标签却是 `#golang`。别问为什么！

创建 Go 的主要目标之一是打造一种易读、强类型、静态类型，并具备垃圾回收、编译与执行速度快的语言，特别适合编写并发应用程序。

`goroutine` 是一种轻量级进程，比 Java 和 C# 等其他语言中常见的线程所需的内存开销更少。一个并发的 Go 程序可以生成成千上万个 goroutine，在数量少得多的线程上运行。`channel` 结构（将在本章后面解释）允许将信息传入和传出 goroutine，并用于同步这些并发的轻量级进程。虽然并行处理并非 goroutine 的主要目标，但它们可用于在共享内存的多处理器计算机上近似实现并行处理。

Go 是一种跨平台语言，可在包括 MacOS 在内的各种 Unix 平台上运行，也可在 MS Windows 上运行。Go 应用程序可编译为二进制可执行文件，因此可以直接分发给客户，而无需像 Python 和其他解释型语言那样打包解释器和运行时库。

与许多现代语言一样，Go 是一个公共开源项目。有许多免费工具可供下载。新包不断发布，因此该语言的许多强大功能并非来自语言本身，而是来自程序员可用的海量高质量包。从这个意义上说，Go 与 Python 相似。

可用的工具包括高质量的编辑器、调试器和 IDE。Go 要求统一的格式，因此 `gofmt` 工具通常会集成到各种代码编辑器中。拥有标准代码格式为 Go 编程团队以及检查他人代码的独立程序员提供了巨大的优势。

那么 Go 缺少什么呢？它的缺点是什么？直到最新、也可能是最重要的版本 1.18 发布之前，Go 一直缺乏泛型。随着 Go 的这个新版本的发布，这一主要缺点已不复存在。

现在，你可以构建一个算法或数据结构，而无需在底层存储的数据发生变化时每次都进行修改。数据结构和算法实现可以专注于操作数据所需的核心逻辑。与泛型相关的新语法允许程序员精确描述数据必须满足哪些要求才能存储到特定的数据结构中。这进一步增强了程序员在代码本身中表达意图的能力。下一节将介绍并说明约束和非约束泛型参数的使用。

## 1.2 泛型参数介绍

在本节中，我们将通过一系列示例来介绍和说明泛型类型参数的使用，包括非约束和约束类型。

在最初的几个代码清单中，我们展示了一组向现有学生切片中添加新学生的相关问题。首先，我们仅向现有切片添加学生姓名。接着，我们向包含 ID 号的切片添加学生 ID 号。然后，我们向现有结构体切片添加包含姓名、ID 和年龄的结构体。最后，我们引入泛型，展示使用泛型类型参数所能实现的简化效果。

### 按姓名添加新学生

考虑清单 1-1 中给出的简单 Go 应用程序。

```go
package main
import(
"fmt"
)
func addStudent(students []string, student string) []string {
return append(students, student)
}
func main() {
students := []string{} // 空切片
result := addStudent(students, "Michael")
result = addStudent(result, "Jennifer")
result = addStudent(result, "Elaine")
fmt.Println(result)
}
/*
输出:
[Michael Jennifer Elaine]
*/
清单 1-1
一个学生姓名切片
```

函数 `addStudent` 接受一个表示当前学生集合的字符串切片作为第一个参数，以及一个表示要添加到集合中的新学生姓名的字符串作为第二个参数。使用 `append` 函数将新学生添加到现有切片中，并返回该结果。

### 按 ID 号添加新学生

假设我们希望使用学生的 ID 号（int 类型）而非姓名（string 类型）来指定学生切片。

我们需要修改清单 1-1，如清单 1-2 所示。

```go
package main
import(
"fmt"
)
func addStudent(students []string, student string) []string {
return append(students, student)
}
func addStudentID(students []int, student int) []int {
return append(students, student)
}
func main() {
students := []string{} // 空切片
result := addStudent(students, "Michael")
result = addStudent(result, "Jennifer")
result = addStudent(result, "Elaine")
fmt.Println(result)
students1 := []int{} // 空切片
result1 := addStudentID(students1, 155)
result1 = addStudentID(result1, 112)
result1 = addStudentID(result1, 120)
fmt.Println(result1)
}
/* 输出
[Michael Jennifer Elaine]
[155 112 120]
*/
清单 1-2
添加学生 ID
```

函数 `addStudentID` 中的逻辑与函数 `addStudent` 中的逻辑基本相同。唯一的变化是切片的基础类型从 `string` 变为了 `int`。



### 通过 `Student` 结构体添加新生

为进一步推进，假设我们定义了一个 `Student` 类型：

```
type Student struct {
    Name  string
    ID    int
    age   float64
}
```

并将清单 1-2 修改为清单 1-3 所示。

```
package main

import (
    "fmt"
)

type Student struct {
    Name  string
    ID    int
    age   float64
}

func addStudent(students []string, student string) []string {
    return append(students, student)
}

func addStudentID(students []int, student int) []int {
    return append(students, student)
}

func addStudentStruct(students []Student, student Student) []Student {
    return append(students, student)
}

func main() {
    students := []string{} // 空切片
    result := addStudent(students, "Michael")
    result = addStudent(result, "Jennifer")
    result = addStudent(result, "Elaine")
    fmt.Println(result)

    students1 := []int{} // 空切片
    result1 := addStudentID(students1, 155)
    result1 = addStudentID(result1, 112)
    result1 = addStudentID(result1, 120)
    fmt.Println(result1)

    students2 := []Student{} // 空切片
    result2 := addStudentStruct(students2, Student{"John", 213, 17.5})
    result2 = addStudentStruct(result2, Student{"James", 111, 18.75})
    result2 = addStudentStruct(result2, Student{"Marsha", 110, 16.25})
    fmt.Println(result2)
}

/* 输出
[Michael Jennifer Elaine]
[155 112 120]
[{John 213 17.5} {James 111 18.75} {Marsha 110 16.25}]
*/
清单 1-3
将 Student 类型加入混合
```

每当我们想要为各种学生集合添加新的底层数据类型时，都必须添加一个新函数，这很繁琐，也是 Go 早期版本的一个主要缺点。

### 引入泛型

进入 Go 1.18 版本，该版本引入了对泛型的支持。

这个问题的泛型解决方案如清单 1-4 所示。

```
package main

import (
    "fmt"
)

type Stringer = interface {
    String() string
}

type Integer int

func (i Integer) String() string {
    return fmt.Sprintf("%d", i)
}

type String string

func (s String) String() string {
    return string(s)
}

type Student struct {
    Name string
    ID   int
    Age  float64
}

func (s Student) String() string {
    return fmt.Sprintf("%s %d %0.2f", s.Name, s.ID, s.Age)
}

func addStudentT Stringer []T {
    return append(students, student)
}

func main() {
    students := []String{}
    result := addStudentString
    result = addStudentString
    result = addStudentString
    fmt.Println(result)

    students1 := []Integer{}
    result1 := addStudentInteger
    result1 = addStudentInteger
    result1 = addStudentInteger
    fmt.Println(result1)

    students2 := []Student{}
    result2 := addStudentStudent
    result2 = addStudentStudent
    result2 = addStudent(result2, Student{"Marsha", 110, 16.25})
    fmt.Println(result2)
}

/* 输出
[Michael Jennifer Elaine]
[45 64 78]
[John 213 17.50 James 111 18.75 Marsha 110 16.25]
*/
清单 1-4
问题的泛型解决方案
```

### `Stringer` 类型

`Stringer` 类型被定义为一个包含一个方法签名的接口：

`String() string`

任何实现了此类型（即拥有明确定义的 `String()` 方法）的实体，都可以被转换为字符串以便输出。由于我们希望能够输出各种学生集合（切片），我们将泛型函数 `addStudent` 签名中的数据类型 `T` 约束为 `Stringer` 类型。

### 受约束的泛型类型

我们单个 `addStudent` 函数的泛型签名变为：

`func addStudentT Stringer []T`

定义了 `Integer`、`String` 和 `Student` 类型以及各自的 `String()` 方法定义，这样我们就可以使用这些 `Stringer` 类型来调用泛型函数 `addStudent`。

### 实现接口

在 Go 中，通过实现接口定义中指定的函数来隐式实现接口。在这种情况下，任何实现了 `String()` 函数的类型都可以被认为是 `Stringer` 类型。

### 实例化泛型类型

在 `main` 函数中，将 `students` 声明为 `String` 类型的空切片（而非字符串切片）后，我们调用 `addStudentString`。

被约束为 `Stringer` 类型的泛型参数 `T` 被实际的实例化类型 `String` 替换，我们知道 `String` 属于 `Stringer` 类型，因为它实现了 `Stringer` 接口（该接口有具体的 `String()` 方法定义）。

接下来，我们使用 `Integer` 作为 `Stringer` 类型来调用 `addStudent`。最后，我们使用 `Student` 作为 `Stringer` 类型来调用 `addStudent`。

### 无约束的泛型类型 `any`

如果我们用无约束类型 `any` 替换受约束的泛型类型 `[T Stringer]`，Go 编译器可以进行类型推断。

清单 1-5 展示了一个更简单、更简洁的 `addStudent` 泛型实现，以及一个驱动函数 `main`，它执行这个泛型函数。

```
package main

import (
    "fmt"
)

type Student struct {
    Name string
    ID   int
    Age  float64
}

func addStudentT any []T {
    return append(students, student)
}

func main() {
    students := []string{}
    result := addStudentstring
    result = addStudentstring
    result = addStudentstring
    fmt.Println(result)

    students1 := []int{}
    result1 := addStudentint
    result1 = addStudentint
    result1 = addStudentint
    fmt.Println(result1)

    students2 := []Student{}
    result2 := addStudentStudent
    result2 = addStudentStudent
    result2 = addStudent(result2, Student{"Marsha", 110, 16.25})
    fmt.Println(result2)
}

/* 输出
[Michael Jennifer Elaine]
[45 64 78]
[John 213 17.50 James 111 18.75 Marsha 110 16.25]
*/
清单 1-5
更简单的泛型函数 addStudent
```

利用类型推断，编译器使用 `string`、`int` 和 `Student` 的默认转换，通过将每种类型转换为字符串来允许程序输出。

### 泛型的优势

在清单 1-5 中，泛型的优势显而易见。将新生（第二个参数）追加到现有学生集合的简单算法，其表达方式与用于表示学生的具体类型无关。

假设我们希望在输出每个学生集合之前对其进行排序。我们可以使用下一节讨论的 `sort` 包来实现这一点。

### 使用 Go 的 sort 包

Go 的 `sort` 包中的 `Sort` 函数 `sort.Sort` 要求被排序切片中的类型必须实现以下三个方法：

1.  `Len`
2.  `Less`
3.  `Swap`

我们演示如何对实现为切片的泛型集合进行排序。

我们按如下方式定义 `OrderedSlice`，并提供所需的 `Len`、`Less` 和 `Swap` 方法组。

```
// 确保 OrderedSlice 可以被排序的函数组
type OrderedSlice[T Ordered] []T // T 必须实现

func (s OrderedSlice[T]) Len() int {
    return len(s)
}

func (s OrderedSlice[T]) Less(i, j int) bool {
    return s[i] < s[j]
}

func (s OrderedSlice[T]) Swap(i, j int) {
    s[i], s[j] = s[j], s[i]
}
// OrderedSice 的函数组结束
```



### Sort Type

我们引入另一个类型 `SortType`，以及必需的 `Len`、`Less` 和 `Swap` 函数组。

```
// Group of functions that ensure that SortType can be sorted
type SortType[T any] struct {
	slice   []T
	compare func(T, T) bool
}
func (s SortType[T]) Len() int {
	return len(s.slice)
}
func (s SortType[T]) Less(i, j int) bool {
	return s.compare(s.slice[i], s.slice[j])
}
func (s SortType[T]) Swap(i, j int) {
	s.slice[i], s.slice[j] = s.slice[j], s.slice[i]
}
// end of group for SortType
```

最后，我们定义一个函数 `PerformSort`，它使用 `SortType` 如下：

```
func PerformSortT any bool) {
	sort.Sort(SortType[T]{slice, compare})
}
```

`PerformSort` 的用户必须提供一个用于比较两个类型为 `T` 的元素的函数。

清单 1-6 将此功能集成到实现泛型 `addStudent` 函数的代码中，以允许我们使用导入的 `sort` 包及其 `Sort` 函数。

```
package main

import (
	"fmt"
	"sort"
)

type Ordered interface {
	~int | ~float64 | ~string
}

type Student struct {
	Name string
	ID   int
	Age  float64
}

func addStudentT any []T {
	return append(students, student)
}

// Group of functions that ensure that an OrderedSlice can be sorted
type OrderedSlice[T Ordered] []T // T must implement 

func (s OrderedSlice[T]) Len() int {
	return len(s)
}

func (s OrderedSlice[T]) Less(i, j int) bool {
	return s[i] < s[j]
}

func (s OrderedSlice[T]) Swap(i, j int) {
	s[i], s[j] = s[j], s[i]
}
// end group for OrderedSice

// Group of functions that ensure that SortType can be sorted
type SortType[T any] struct {
	slice   []T
	compare func(T, T) bool
}

func (s SortType[T]) Len() int {
	return len(s.slice)
}

func (s SortType[T]) Less(i, j int) bool {
	return s.compare(s.slice[i], s.slice[j])
}

func (s SortType[T]) Swap(i, j int) {
	s.slice[i], s.slice[j] = s.slice[j], s.slice[i]
}
// end of group for SortType

func PerformSortT any bool) {
	sort.Sort(SortType[T]{slice, compare})
}

func main() {
	students := []string{}
	result := addStudentstring
	result = addStudentstring
	result = addStudentstring
	sort.Sort(OrderedSlicestring)
	fmt.Println(result)

	students1 := []int{}
	result1 := addStudentint
	result1 = addStudentint
	result1 = addStudentint
	sort.Sort(OrderedSliceint)
	fmt.Println(result1)

	students2 := []Student{}
	result2 := addStudentStudent
	result2 = addStudentStudent
	result2 = addStudent(result2,  Student{"Marsha", 110, 16.25} )
	PerformSortStudent bool {
		return s1.Age < s2.Age // comparing two Student values
	})
	fmt.Println(result2)
}
/* Output
[Elaine Jennifer Michael]
[45 64 78]
[{Marsha 110 16.25} {John 213 17.5} {James 111 18.75}]
*/
Listing 1-6
Building and sorting slices of students
```

### Map Functions

Go 语言中的 Map 函数很常见，它对切片执行转换，生成一个包含转换结果的新切片。考虑以下示例：

```
func MyMap(input []int, f func(int) int) []int {
	result := make([]int, len(input))
	for index, value := range input {
		result[index] = f(value)
	}
	return result
}

func main() {
	slice := []int{1, 5, 2, 7, 4}
	result := MyMap(slice, func(i int) int {
		return i * i
	})
	fmt.Println(result)
}
/* Output
[1 25 4 49 16]
*/
```

`MyMap` 函数生成一个输出切片，其中包含输入切片中整数的平方。在将 `result` 声明为长度为 `len(slice)` 的整数切片后，它遍历 `input` 中的值范围，根据传递给 `MyMap` 的函数 `f` 转换每个值。

### 使 MyMap 成为泛型

`MyMap` 可以按如下方式泛型化：

```
func GenericMapT1, T2 any T2) []T2 {
	result := make([]T2, len(input))
	for index, value := range input {
		result[index] = f(value)
	}
	return result
}
```

函数 `GenericMap` 接受两个泛型参数 `T1` 和 `T2`。使用传入的函数 `f`，它将 `input` 切片中的数据转换为类型 `T2`。这里，`T1` 和 `T2` 不受约束，它们都属于 `any` 类型。

### Filter Functions

Go 语言中的 Filter 函数也很常见，它基于传入的函数对输入切片执行过滤操作。考虑以下示例：

```
func MyFilter(input []float64, f func(float64) bool) []float64 {
	var result []float64
	for _, value := range input {
		if f(value) == true {
			result = append(result, value)
		}
	}
	return result
}

func main() {
	input := []float64{17.3, 11.1, 9.9, 4.3, 12.6}
	res := MyFilter(input, func(num float64) bool {
		if num <= 10.0 {
			return true
		}
		return false
	})
	fmt.Println(res)
}
/* Output
[9.9 4.3]
*/
```

在这里，输入切片中小于或等于 `10.0` 的任何值都会被保留，而所有其他值则被过滤掉。

### 使 MyFilter 成为泛型

`MyFilter` 可以按如下方式泛型化：

```
func GenericFilterT any bool) []T {
	var result []T
	for _, val := range input {
		if f(val) {
			result = append(result, val)
		}
	}
	return result
}
```

在清单 1-7 中，我们演练了泛型 map 和 filter 函数。

```
package main

import (
	"fmt"
)

func GenericMapT1, T2 any T2) []T2 {
	result := make([]T2, len(input))
	for index, value := range input {
		result[index] = f(value)
	}
	return result
}

func GenericFilterT any bool) []T {
	var result []T
	for _, val := range input {
		if f(val) {
			result = append(result, val)
		}
	}
	return result
}

func main() {
	input := []float64{-5.0, -2.0, 4.0, 8.0}
	result1 := GenericMapfloat64, float64 float64 {
		return n * n
	})
	fmt.Println(result1)

	greaterThanFive := GenericFilterint bool {
			return i > 5
		})
	fmt.Println(greaterThanFive)

	oddNumbers := GenericFilterint bool {
			return i % 2 == 1
		})
	fmt.Println(oddNumbers)

	lengthGreaterThan3 := GenericFilterstring bool {
		return len(s) > 3
	})
	fmt.Println(lengthGreaterThan3)
}
/* Output
[25 4 16 64]
[6 20 7]
[5 1 7]
[hello maybe]
*/
Listing 1-7
Using generic map and filter functions
```

我们现在将注意力转向并发性的使用。

## 1.3 并发

并发性允许程序同时处理多个任务（并行处理，每个任务被分配给一个单独的处理器），或者看似同时处理多个任务（任务被复用，使得所有任务随时间推进）。如果复用速度非常快，那么并发进程看起来像是在同时运行，但实际上它们是在重叠的时间段内运行的。

在大多数支持并发处理的语言中，使用 `thread` 结构来支持并发性。线程有相关的内存开销，因此同一时间可以生成的线程数量是有限的。



### Goroutine

在开发 Go 语言时，Google 引入了一种名为 **goroutine** 的轻量级进程，它比线程所需的内存开销更少。其动机是能够同时处理来自多个网络浏览器的大量 HTML 网页。

Goroutine 是与其他函数并发运行的函数。当调用一个普通函数时，该函数完成其工作后，其下方的代码才会执行。当调用一个 goroutine 函数时，由于其与下方的代码并发运行，因此紧接其下的代码会立即执行。

我们在清单 1-8 中对此进行了说明。

```go
package main
import (
"fmt"
"time"
)
func regularFunction() {
fmt.Println("Just entered regularFunction()")
time.Sleep(5 * time.Second)
}
func goroutineFunction() {
fmt.Println("Just entered goroutineFunction()")
time.Sleep(10 * time.Second)
fmt.Println("goroutineFunction finished its work")
}
func main() {
go goroutineFunction()
fmt.Println("In main one line below goroutineFunction()")
regularFunction()
fmt.Println("In main one line below regularFunction()")
}
/* Output
In main, one line below goroutineFunction()
Just entered regularFunction()
Just entered goroutineFunction()
In main one line below regularFunction()
*/
Listing 1-8
Simple goroutine running concurrent with main
```

当 `goroutineFunction` 通过 `go goroutineFunction()` 作为 goroutine 启动时，它会与 `main` 函数（本身也是一个 goroutine）并发运行。第一行输出会立即出现，即使 `goroutineFunction` 需要十秒才能完成其工作。当接下来调用 `regularFunction()` 时，需要经过五秒才会输出“In main, one line below regularFunction()”。`main` 函数在输出此行后立即终止，导致程序在 `goroutineFunction` 完成其工作前结束。当程序结束时，`goroutineFunction` 会被中断并终止。

如果将时间延迟互换，使得 `goroutineFunction` 有五秒延迟，而 `regularFunction` 有十秒延迟，输出将变为：

```
In main one line below goroutineFunction()
Just entered regularFunction()
Just entered goroutineFunction()
goroutineFunction finished its work
In main, one line below regularFunction()
```

现在，与 `main` 并发运行的 goroutine 在 `regularFunction` 和 `main` goroutine 退出前就完成了其工作。

### WaitGroup

Go 提供了一种机制，允许在 `main` 退出之前，让多个 goroutine 全部完成其工作，同时终止未完成的 goroutine。

我们引入 `sync` 包和 `WaitGroup` 结构，并在清单 1-9 中说明其用法。

```go
package main
import (
"fmt"
"time"
"math/rand"
"sync"
)
var wg sync.WaitGroup
func outputStrings() {
defer wg.Done()
strings := [5]string{"One", "Two", "Three", "Four", "Five"}
for i := 0; i < 5; i++ {
delay := 1 + rand.Intn(3)
time.Sleep(time.Duration(delay) * time.Second)
fmt.Println(strings[i])
}
}
func outputInts() {
defer wg.Done()
for i := 0; i < 5; i++ {
delay := 1 + rand.Intn(3)
time.Sleep(time.Duration(delay) * time.Second)
fmt.Println(i)
}
}
func outputFloats() {
defer wg.Done()
for i := 0; i < 5; i++ {
delay := 1 + rand.Intn(3)
time.Sleep(time.Duration(delay) * time.Second)
fmt.Println(float64(i * i) + 0.5)
}
}
func main() {
wg.Add(3)
go outputStrings()
go outputInts()
go outputFloats()
wg.Wait()
}
/* Output
One
0.5

Two
1.5

4.5

Three
Four

9.5
Five
16.5
*/
Listing 1-9
The sync package and WaitGroup
```

这个程序除了说明 `WaitGroup` 结构并展示三个 goroutine 并发运行外，没有做任何有用的事情。

声明了一个类型为 `sync.WaitGroup` 的全局变量 `wg`。

在 `main` 中，我们调用 `wg.Add(3)`。`main` 中的最后一行代码是 `wg.Wait()`。这会导致 `main` 暂停，直到 `wg` 的值变为零。这确保所有三个 goroutine 在程序终止前完成其工作。

在每个 goroutine 中，第一行代码 `defer wg.Done()` 会在 goroutine 完成其工作时，使全局变量 `wg` 的值递减。当 `wg` 的值达到零时，`main` 函数才被允许退出。

每次运行程序时生成的随机数序列是相同的，但输出序列每次运行都会变化。这是因为时间多路复用器每次运行程序时，都会为每个并发 goroutine 分配不同的执行时间片。程序第二次运行后的输出是：

```
One

0.5
1.5
Two

4.5

Three
9.5
16.5
Four

Five

```

### Channel

我们经常希望能够同步 goroutine 的执行顺序，并让它们相互通信。我们引入强大的 **channel** 结构来实现这一点。

考虑清单 1-10 中的 goroutine。

```go
package main
import (
"fmt"
"time"
"sync"
)
var wg sync.WaitGroup
func pingGenerator(c chan string) {
defer wg.Done()
for i := 0; i < 5; i++ {
c <- "ping"
time.Sleep(time.Second * 1)
}
}
func output(c chan string) {
defer wg.Done()
for {
value := <- c
fmt.Println(value)
}
}
func main() {
c := make(chan string)
wg.Add(2)
go pingGenerator(c)
go output(c)
wg.Wait()
}
/* Output
ping
ping
ping
ping
ping
fatal error: all goroutines are asleep - deadlock!
*/
Listing 1-10
Deadlock
```

`main` 中的第一行代码初始化了一个 channel `c`。Channel 必须使用 `make` 语句进行初始化后才能使用。

与上一个清单类似，我们创建了一个初始值为 2 的 `WaitGroup` 变量 `wg`。

接下来，我们启动两个 goroutine，`pingGenerator` 和 `output`，并将 channel 变量 `c` 传递给它们。

`pingGenerator` goroutine 每秒将字符串“ping”分配给 channel 变量 `c`，共执行五次。从值“ping”到变量 `c` 的左箭头表示将“ping”值分配给 `c`。

要完成此分配，channel 必须为空。在 `output` goroutine 中，使用 `value := <- c` 对 `value` 进行赋值，只要 `pingGenerator` 中一赋值给 channel 变量 `c`，`output` 就会立即取走它。这每秒发生一次。在 `pingGenerator` 两次分配“ping”之间的时间内，`value` 的赋值会被阻塞。也就是说，在 `output` goroutine 中，执行会暂停，直到 channel 中有信息可供赋值给 `value`。因此，两个 goroutine 都受到它们共有的 channel 变量 `c` 的影响。

这里有一个问题。当 `pingGenerator` 发出其五个“ping”赋值（每个赋值都通过 `output` goroutine 显示在控制台上）后，它会一直阻塞，等待第六次 channel 赋值。但这种情况永远不会发生。程序崩溃并显示前面展示的错误信息。发生了死锁。程序无法终止。

### Select 语句

我们可以通过修改 `output` goroutine 并使用 **select** 语句来解决这个问题。

```go
func output(c chan string) {
for {
select {
case value := <- c:
fmt.Println(value)
case <-time.After(3 * time.Second):
fmt.Println("Program timed out.")
wg.Done()
}
}
}
```

在 **select** 语句中，首先发生的 case 会被执行。如果两个或多个 case 都准备就绪，系统会随机选择一个。由于 channel `c` 每秒都会被赋值给 `value`（赋值之间会阻塞），因此程序会输出五个“ping”赋值。与之前发生死锁不同，在第五次也是最后一次“ping”被赋值给 `value` 三秒后，第二个 case 会被执行。



### 使用`quit`通道避免使用`WaitGroup`

我们可以使用`quit`通道来阻止`main`函数退出，从而避免使用`WaitGroup`，如代码清单 1-11 所示。

```
package main
import (
"fmt"
"time"
)
var quit chan bool
func pingGenerator(c chan string) {
for i := 0; i < 5; i++ {
c <- "ping"
time.Sleep(time.Second * 1)
}
}
func output(c chan string) {
for {
select {
case value := <- c:
fmt.Println(value)
case <-time.After(3 * time.Second):
fmt.Println("Program timed out.")
quit <- true
}
}
}
func main() {
quit = make(chan bool)
c := make(chan string)
go pingGenerator(c)
go output(c)
<- quit
}
/* Output
ping
ping
ping
ping
ping
Program timed out.
*/
代码清单 1-11
使用 quit 通道代替 WaitGroup
```

`quit`通道在`main`函数的第一行代码中被初始化。`main`函数的最后一行代码`<- quit`会阻塞`main`函数，使其无法结束，直到一个布尔值被赋给`quit`。这个赋值操作发生在协程`output`的第二个`case`语句中。

这种控制程序结束的机制比使用`WaitGroup`更简单，也更少负担。

我们将不可避免的`pongGenerator`添加到这个程序中。

### 通道方向

可以为协程签名添加通道方向，如代码清单 1-12 所示。如`pingGenerator`和`pongGenerator`的签名所示，从右侧指向`chan`的箭头要求协程向通道赋值（即生成器）。`chan`左侧指向通道变量的箭头要求协程只能从通道中消费值。

如果尝试向指定为消费者的通道发送信息，或者尝试从指定为生成器的通道中读取信息，则会发生编译错误。

```
package main
import (
"fmt"
"time"
)
var quit chan bool
func pingGenerator(c chan<- string) {
// 通道只能被发送 - 一个生成器
for i := 0; i < 5; i++ {
c <- "ping"
}
}
func pongGenerator(c chan<- string) {
// 信息只能被发送到通道 - 一个生成器
for i := 0; i < 5; i++ {
c <- "pong"
}
}
func output(c <- chan string) {
// 信息只能从通道接收 - 一个消费者
for {
time.Sleep(time.Second * 1)
select {
case value := <- c:
fmt.Println(value)
case <-time.After(3 * time.Second):
fmt.Println("Program timed out.")
quit <- true
}
}
}
func main() {
quit = make(chan bool)
c := make(chan string)
go pingGenerator(c)
go pongGenerator(c)
go output(c)
<- quit
}
/* Output
ping
pong
ping
pong
ping
pong
ping
pong
ping
pong
Program timed out.
*/
代码清单 1-12
在协程签名中使用方向通道实现乒乓球效果
```

我们将一秒钟的时间延迟移到了`output`协程中。这使得`ping`和`pong`生成器可以交替执行，因为每次对通道的赋值都会交替阻塞，直到通道被消费协程`output`读取。

### 竞态条件

使用并发时的一个普遍问题是竞态条件。当两个或更多的协程修改同一份共享数据时，就会出现这个问题。

代码清单 1-13 给出了一个简单的例子。

```
package main
import (
"fmt"
"sync"
)
const (
number = 1000
)
var countValue int
func main() {
var wg sync.WaitGroup
wg.Add(number)
for i := 0; i < number; i++ {
go func() {
countValue++
wg.Done()
}()
}
wg.Wait()
fmt.Printf("\ncountValue = %d", countValue)
}
代码清单 1-13
竞态条件示例
```

在`main`函数的 for 循环中，生成了 1000 个协程。每个协程将`countValue`恰好增加一次。因此，人们会期望程序的输出是 1000。

每次运行程序，输出都不同。这是因为多个协程试图在几乎同一时间修改全局变量`countValue`而产生了冲突。系统不会生成错误消息，但输出结果是不正确的。

### 互斥锁

我们可以使用互斥锁来修正竞态条件问题。这会在每个协程修改其值时锁定全局的`countValue`，从而保护这份共享数据免遭破坏。

代码清单 1-14 展示了使用`mutex`消除竞态条件的方法。

```
package main
import (
"fmt"
"sync"
)
const number = 1000
var countValue int
var m sync.Mutex
func main() {
var wg sync.WaitGroup
wg.Add(number)
for i := 0; i < number; i++ {
go func() {
m.Lock()
countValue++
m.Unlock()
wg.Done()
}()
}
wg.Wait()
fmt.Printf("\ncountValue = %d", countValue)
}
代码清单 1-14
使用互斥锁避免竞态条件
```

每个协程中的`m.Lock()`代码保护了全局变量`countValue`，使其不会被调用该锁的协程之外的协程修改。在调用`m.Unlock()`之前，没有其他协程可以更改`countValue`。

使用`mutex`会减慢程序执行速度，但可以保护程序免受代码清单 1-13 中所示的竞态条件影响。

## 使用协程下棋

代码清单 1-15 模拟了两位棋手使用协程轮流下棋的顺序。

```
// 本示例程序演示如何使用无缓冲通道来模拟两个协程之间的象棋移动。
package main
import (
"fmt"
"math/rand"
"time"
)
var quit chan bool
func main() {
rand.Seed(time.Now().UnixNano())
move := make(chan int)
quit = make(chan bool)
// 启动两位玩家。
go player("鲍比·费舍尔", move)
go player("鲍里斯·斯帕斯基", move)
// 开始移动
move <- 1
<-quit
}
// player 模拟一个下棋的人。
func player(name string, move chan int) {
for {
// 等待轮到自己的回合。
turn := <-move
// // 选择一个随机数，看看玩家是否输掉。
if turn := <-move; turn > 0 && turn < 100 {
// ... (游戏逻辑)
}
if turn = rand.Intn(100); turn == 5 {
fmt.Printf("玩家 %s 被将杀，输掉了比赛！", name)
quit <- true
return
}
// 显示总移动次数并将其加一。
fmt.Printf("玩家 %s 已经移动。回合 %d。\n", name, turn)
turn++
// 将回合让给对手
time.Sleep(1 * time.Second)
move <- turn
}
}
/*
玩家 鲍里斯·斯帕斯基 已经移动。回合 1。
玩家 鲍比·费舍尔 已经移动。回合 2。
玩家 鲍里斯·斯帕斯基 已经移动。回合 3。
玩家 鲍比·费舍尔 已经移动。回合 4。
玩家 鲍里斯·斯帕斯基 已经移动。回合 5。
玩家 鲍比·费舍尔 已经移动。回合 6。
玩家 鲍里斯·斯帕斯基 已经移动。回合 7。
玩家 鲍比·费舍尔 已经移动。回合 8。
玩家 鲍里斯·斯帕斯基 已经移动。回合 9。
玩家 鲍比·费舍尔 已经移动。回合 10。
玩家 鲍里斯·斯帕斯基 已经移动。回合 11。
玩家 鲍比·费舍尔 已经移动。回合 12。
玩家 鲍里斯·斯帕斯基 已经移动。回合 13。
玩家 鲍比·费舍尔 已经移动。回合 14。
玩家 鲍里斯·斯帕斯基 已经移动。回合 15。
玩家 鲍比·费舍尔 已经移动。回合 16。
玩家 鲍里斯·斯帕斯基 已经移动。回合 17。
玩家 鲍比·费舍尔 已经移动。回合 18。
玩家 鲍里斯·斯帕斯基 已经移动。回合 19。
玩家 鲍比·费舍尔 已经移动。回合 20。
玩家 鲍里斯·斯帕斯基 已经移动。回合 21。
玩家 鲍比·费舍尔 已经移动。回合 22。
玩家 鲍里斯·斯帕斯基 已经移动。回合 23。
玩家 鲍比·费舍尔 已经移动。回合 24。
玩家 鲍里斯·斯帕斯基 已经移动。回合 25。
玩家 鲍比·费舍尔 被将杀，输掉了比赛！
*/
代码清单 1-15
象棋中的并发移动
```

协程`player`的 for 循环中的第一行代码会阻塞，直到从`move`通道中取出一个 int 值。有 5%的概率该玩家会输掉比赛，并将`quit`设置为 true。

在输出玩家已移动并公布其回合数之后，它会将一个 int 值放回`move`通道，从而释放另一位玩家，使其可以移动。



