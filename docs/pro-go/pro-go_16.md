# 28. 使用反射，第二部分

除了前一章介绍的基本功能外，`reflect` 包还提供了处理特定类型（如映射或结构体）时有用的额外功能。在后续各节中，我将介绍这些功能并演示其用法。部分描述的方法和函数可用于多种类型，为了便于快速查阅，我会多次列出它们。表 28-1 总结了本章内容。

**表 28-1.** 本章小结

| 问题 | 解决方案 | 代码清单 |
| --- | --- | --- |
| 创建或跟踪指针类型 | 使用 `PtrTo` 和 `Elem` 方法 | 3 |
| 创建或跟踪指针值 | 使用 `Addr` 和 `Elem` 方法 | 4 |
| 检查或创建切片 | 使用针对切片的 `Type` 和 `Value` 方法 | 5–8 |
| 创建、复制和追加切片 | 使用针对切片的 `reflect` 函数 | 9 |
| 检查或创建映射 | 使用针对映射的 `Type` 和 `Value` 方法 | 10–14 |
| 检查或创建结构体 | 使用针对结构体的 `reflect` 函数 | 15–17, 19–21 |
| 检查结构体标签 | 使用 `StructTag` 定义的方法 | 18 |

## 本章准备

在本章中，我将继续使用第 27 章创建的 `reflection` 项目。为准备本章，请将清单 28-1 中所示的类型添加到 `reflection` 文件夹中的 `types.go` 文件中。

**提示** 你可以从 [`https://github.com/apress/pro-go`](https://github.com/apress/pro-go) 下载本章（以及本书所有其他章节）的示例项目。如果在运行示例时遇到问题，请参阅第 2 章了解如何获取帮助。

```
package main

type Product struct {
    Name, Category string
    Price          float64
}

type Customer struct {
    Name, City string
}

type Purchase struct {
    Customer
    Product
    Total   float64
    taxRate float64
}
```

**清单 28-1.** 在 `reflection` 文件夹的 `types.go` 文件中定义类型

在 `reflection` 文件夹中运行清单 28-2 所示的命令，以编译并执行项目。

```
go run .
```

**清单 28-2.** 编译并执行项目

此命令将产生以下输出：

```
Value: London
Value: 279
Value: Alice
```

## 使用指针

`reflect` 包提供了表 28-2 所示的函数和方法，用于处理指针类型。

**表 28-2.** `reflect` 包中用于指针的函数和方法

| 名称 | 描述 |
| --- | --- |
| `PtrTo(type)` | 此函数返回一个 `Type`，该 `Type` 是指向作为参数传入的 `Type` 的指针。 |
| `Elem()` | 此方法在指针类型上调用，返回底层的 `Type`。在非指针类型上使用此方法会导致恐慌。 |

`PtrTo` 函数创建指针类型，而 `Elem` 方法返回被指向的类型，如清单 28-3 所示。

```
package main

import (
    "reflect"
    //"strings"
    // "fmt"
)

func createPointerType(t reflect.Type) reflect.Type {
    return reflect.PtrTo(t)
}

func followPointerType(t reflect.Type) reflect.Type {
    if t.Kind() == reflect.Ptr {
        return t.Elem()
    }
    return t
}

func main() {
    name := "Alice"
    t := reflect.TypeOf(name)
    Printfln("Original Type: %v", t)
    pt := createPointerType(t)
    Printfln("Pointer Type: %v", pt)
    Printfln("Follow pointer type: %v", followPointerType(pt))
}
```

**清单 28-3.** 在 `reflection` 文件夹的 `main.go` 文件中使用指针类型

`PtrTo` 函数从 `reflect` 包中导出。它可以在任何类型（包括指针类型）上调用，结果是一个指向原始类型的类型，因此 `string` 类型产生 `*string` 类型，而 `*string` 产生 `**string`。

`Elem` 是 `Type` 接口定义的一个方法，只能用于指针类型，这就是为什么清单 28-3 中的 `followPointerType` 函数在调用 `Elem` 方法之前会检查 `Kind` 方法的结果。编译并执行项目，你将看到以下输出：

```
Original Type: string
Pointer Type: *string
Follow pointer type: string
```

### 使用指针值

`Value` 结构体定义了表 28-3 所示的方法，用于处理指针值（区别于上一节描述的类型）。

**表 28-3.** 用于处理指针类型的 `Value` 方法

| 名称 | 描述 |
| --- | --- |
| `Addr()` | 此方法返回一个 `Value`，该 `Value` 是指向调用它的 `Value` 的指针。如果 `CanAddr` 方法返回 `false`，此方法会引发恐慌。 |
| `CanAddr()` | 如果 `Value` 可以与 `Addr` 方法一起使用，此方法返回 `true`。 |
| `Elem()` | 此方法跟踪指针并返回其 `Value`。如果在非指针值上调用此方法，会引发恐慌。 |

`Elem` 方法用于跟踪指针以获取其底层值，如清单 28-4 所示。其他方法在处理结构体字段时最有用，如“设置结构体字段值”一节所述。

```
package main

import (
    "reflect"
    "strings"
    // "fmt"
)

var stringPtrType = reflect.TypeOf((*string)(nil))

func transformString(val interface{}) {
    elemValue := reflect.ValueOf(val)
    if (elemValue.Type() == stringPtrType) {
        upperStr := strings.ToUpper(elemValue.Elem().String())
        if (elemValue.Elem().CanSet()) {
            elemValue.Elem().SetString(upperStr)
        }
    }
}

func main() {
    name := "Alice"
    transformString(&name)
    Printfln("Follow pointer value: %v", name)
}
```

**清单 28-4.** 在 `reflection` 文件夹的 `main.go` 文件中跟踪指针

`transformString` 函数识别 `*string` 值，并使用 `Elem` 方法获取 `string` 值，以便将其传递给 `strings.ToUpper` 函数。编译并执行项目，你将看到以下输出：

```
Follow pointer value: ALICE
```



## 处理数组与切片类型

`Type` 结构体定义了一些可用于检查数组和切片类型的方法，详见表 28-4。

**表 28-4** 数组与切片的 `Type` 方法

| 名称 | 描述 |
| --- | --- |
| `Elem()` | 此方法返回数组或切片元素的 `Type`。 |
| `Len()` | 此方法返回数组类型的长度。如果对包括切片在内的其他类型调用此方法，将引发 panic。 |

除了这些方法，`reflect` 包还提供了表 28-5 中描述的用于创建数组和切片类型的函数。

**表 28-5** 用于创建数组和切片类型的 `reflect` 函数

| 名称 | 描述 |
| --- | --- |
| `ArrayOf(len, type)` | 此函数返回一个 `Type`，描述一个具有指定长度和元素类型的数组。 |
| `SliceOf(type)` | 此函数返回一个 `Type`，描述一个具有指定元素类型的数组。 |

清单 28-5 使用 `Elem` 方法来检查数组和切片的类型。

```
package main
import (
"reflect"
//"strings"
// "fmt"
)
func checkElemType(val interface{}, arrOrSlice interface{}) bool {
elemType := reflect.TypeOf(val)
arrOrSliceType := reflect.TypeOf(arrOrSlice)
return (arrOrSliceType.Kind() == reflect.Array ||
arrOrSliceType.Kind() == reflect.Slice) &&
arrOrSliceType.Elem() == elemType
}
func main() {
name := "Alice"
city := "London"
hobby := "Running"
slice := []string { name, city, hobby }
array := [3]string { name, city, hobby}
Printfln("Slice (string): %v",  checkElemType("testString", slice))
Printfln("Array (string): %v",  checkElemType("testString", array))
Printfln("Array (int): %v",  checkElemType(10, array))
}
清单 28-5
检查 `reflection` 文件夹中 `main.go` 文件里的数组与切片类型
```

`checkElemType` 函数使用 `Kind` 方法来识别数组和切片，并使用 `Elem` 方法来获取元素的 `Type`。然后将这些与第一个参数的类型进行比较，以确定该值能否作为元素添加。编译并运行项目，你会看到如下结果：

```
Slice (string): true
Array (string): true
Array (int): false
```

## 处理数组与切片值

`Value` 接口定义了表 28-6 中描述的方法，用于处理数组和切片的值。

**表 28-6** 用于处理数组与切片的 `Value` 方法

| 名称 | 描述 |
| --- | --- |
| `Index(index)` | 此方法返回一个 `Value`，表示指定索引处的元素。 |
| `Len()` | 此方法返回数组或切片的长度。 |
| `Cap()` | 此方法返回数组或切片的容量。 |
| `SetLen()` | 此方法设置切片的长度。不能在数组上使用。 |
| `SetCap()` | 此方法设置切片的容量。不能在数组上使用。 |
| `Slice(lo, hi)` | 此方法使用指定的低值和高值创建一个新的切片。 |
| `Slice3(lo, hi, max)` | 此方法使用指定的低值、高值和最大值创建一个新的切片。 |

`Index` 方法返回一个 `Value`，它可以与第 27 章中描述的 `Set` 方法一起使用，以更改切片或数组中的值，如清单 28-6 所示。

```
package main
import (
"reflect"
//"strings"
// "fmt"
)
func setValue(arrayOrSlice interface{}, index int, replacement interface{}) {
arrayOrSliceVal := reflect.ValueOf(arrayOrSlice)
replacementVal := reflect.ValueOf(replacement)
if (arrayOrSliceVal.Kind() == reflect.Slice) {
elemVal := arrayOrSliceVal.Index(index)
if (elemVal.CanSet()) {
elemVal.Set(replacementVal)
}
} else if (arrayOrSliceVal.Kind() == reflect.Ptr &&
arrayOrSliceVal.Elem().Kind() == reflect.Array &&
arrayOrSliceVal.Elem().CanSet()) {
arrayOrSliceVal.Elem().Index(index).Set(replacementVal)
}
}
func main() {
name := "Alice"
city := "London"
hobby := "Running"
slice := []string { name, city, hobby }
array := [3]string { name, city, hobby}
Printfln("Original slice: %v", slice)
newCity := "Paris"
setValue(slice, 1, newCity)
Printfln("Modified slice: %v", slice)
Printfln("Original slice: %v", array)
newCity = "Rome"
setValue(&array, 1, newCity)
Printfln("Modified slice: %v", array)
}
清单 28-6
更改 `reflection` 文件夹中 `main.go` 文件里的切片元素
```

`setValue` 函数更改切片或数组中某个元素的值，但每种类型都必须以不同方式处理。切片是最容易处理的，可以作为值传递，就像这样：

```
...
setValue(slice, 1, newCity)
...
```

正如我在第 7 章中解释的，切片是引用，当它们用作函数参数时不会被复制。在清单 28-6 中，`setValue` 方法使用 `Kind` 方法来检测切片，使用 `Index` 方法获取指定位置元素的 `Value`，并使用 `Set` 方法来更改元素的值。数组必须作为指针传递，就像这样：

```
...
setValue(&array, 1, newCity)
...
```

如果不使用指针，那么你将无法设置新值，并且 `CanSet` 方法将返回 `false`。`Kind` 方法用于检测指针，`Elem` 方法用于确认它指向一个数组：

```
...
} else if (arrayOrSliceVal.Kind() == reflect.Ptr &&
arrayOrSliceVal.Elem().Kind() == reflect.Array &&
arrayOrSliceVal.Elem().CanSet()) {
...
```

要设置元素值，通过 `Elem` 方法跟随指针以获取反射后的 `Value`，使用 `Index` 方法获取指定索引元素的 `Value`，再使用 `Set` 方法分配新值：

```
...
arrayOrSliceVal.Elem().Index(index).Set(replacementVal)
...
```

总体效果是 `setValue` 函数可以操作切片和数组，而无需知道具体使用什么类型。编译并执行项目，你会看到以下输出：

```
Original slice: [Alice London Running]
Modified slice: [Alice Paris Running]
Original slice: [Alice London Running]
Modified slice: [Alice Rome Running]
```



### 枚举切片与数组

使用 `Len` 方法可以在 `for` 循环中设定枚举数组或切片元素的限制，如代码清单 28-7 所示。

```go
package main
import (
"reflect"
//"strings"
// "fmt"
)
func enumerateStrings(arrayOrSlice interface{}) {
arrayOrSliceVal := reflect.ValueOf(arrayOrSlice)
if (arrayOrSliceVal.Kind() == reflect.Array ||
arrayOrSliceVal.Kind() == reflect.Slice) &&
arrayOrSliceVal.Type().Elem().Kind() == reflect.String {
for i := 0; i < arrayOrSliceVal.Len(); i++ {
Printfln("Element: %v, Value: %v", i, arrayOrSliceVal.Index(i).String())
}
}
}
func main() {
name := "Alice"
city := "London"
hobby := "Running"
slice := []string { name, city, hobby }
array := [3]string { name, city, hobby}
enumerateStrings(slice)
enumerateStrings(array)
}
代码清单 28-7
在 reflection 文件夹的 main.go 文件中枚举数组和切片
```

`enumerateStrings` 函数检查 `Kind` 结果，以确保处理的是字符串数组或字符串切片。在此过程中，很容易对使用了哪个 `Elem` 方法感到困惑，因为 `Type` 和 `Value` 都定义了 `Kind` 和 `Elem` 方法。`Kind` 方法执行相同的任务，但在切片或数组的 `Value` 上调用 `Elem` 方法会导致 panic，而在切片或数组的 `Type` 上调用 `Elem` 方法则会返回元素的 `Type`：

```go
...
arrayOrSliceVal.Type().Elem().Kind() == reflect.String {
...
```

一旦函数确认处理的是字符串数组或字符串切片，就会使用 `for` 循环，其限制由 `Len` 方法的结果设定：

```go
...
for i := 0; i < arrayOrSliceVal.Len(); i++ {
...
```

在 `for` 循环内部使用 `Index` 方法获取当前索引处的元素，并使用 `String` 方法获取其值：

```go
...
Printfln("Element: %v, Value: %v", i, arrayOrSliceVal.Index(i).String())
...
```

注意，枚举其内容时不需要使用指针引用数组。这仅在进行修改时才需要。编译并执行项目，你将看到以下输出，这是对切片和数组的枚举结果：

```
Element: 0, Value: Alice
Element: 1, Value: London
Element: 2, Value: Running
Element: 0, Value: Alice
Element: 1, Value: London
Element: 2, Value: Running
```

### 从现有切片创建新切片

`Slice` 方法用于从一个切片创建另一个切片，如代码清单 28-8 所示。

```go
package main
import (
"reflect"
//"strings"
// "fmt"
)
func findAndSplit(slice interface{}, target interface{}) interface{} {
sliceVal := reflect.ValueOf(slice)
targetType := reflect.TypeOf(target)
if (sliceVal.Kind() == reflect.Slice && sliceVal.Type().Elem() == targetType) {
for i := 0; i < sliceVal.Len(); i++ {
if sliceVal.Index(i).Interface() == target {
return sliceVal.Slice(0, i +1)
}
}
}
return slice
}
func main() {
name := "Alice"
city := "London"
hobby := "Running"
slice := []string { name, city, hobby }
//array := [3]string { name, city, hobby}
Printfln("Strings: %v", findAndSplit(slice, "London"))
numbers := []int {1, 3, 4, 5, 7}
Printfln("Numbers: %v", findAndSplit(numbers, 4))
}
代码清单 28-8
在 reflection 文件夹的 main.go 文件中创建新切片
```

`findAndSplit` 函数枚举切片，查找指定元素，这通过 `Interface` 方法完成，该方法允许比较切片元素而无需处理特定类型。一旦找到目标元素，就使用 `Slice` 方法创建并返回一个新切片。编译并执行项目，你将看到以下输出：

```
Strings: [Alice London]
Numbers: [1 3 4]
```

### 创建、复制和追加元素到切片

`reflect` 包定义了表 28-7 中描述的函数，这些函数允许将值复制和追加到切片，而无需处理底层类型。

**表 28-7** 用于追加元素到切片的函数

| 名称 | 描述 |
| --- | --- |
| `MakeSlice(type, len, cap)` | 此函数创建一个反映新切片的 `Value`，使用 `Type` 表示元素类型，并具有指定的长度和容量。 |
| `Append(sliceVal, ...val)` | 此函数将一个或多个值追加到指定切片，所有这些值都使用 `Value` 接口表示。结果是修改后的切片。如果对切片之外的任何类型使用该函数，或者值的类型与切片元素类型不匹配，则该函数会引发 panic。 |
| `AppendSlice(sliceVal, sliceVal)` | 此函数将一个切片追加到另一个切片。如果任何一个 `Value` 不表示切片，或者切片类型不兼容，则该函数会引发 panic。 |
| `Copy(dst, src)` | 此函数将 `src Value` 反映的切片或数组中的元素复制到 `dst Value` 反映的切片或数组中。元素会被复制，直到目标切片已满或所有源元素都被复制完毕。源和目标必须具有相同的元素类型。 |

这些函数接受 `Type` 或 `Value` 参数，这可能违反直觉并需要准备。`MakeSlice` 函数接受一个 `Type` 参数，指定切片类型，并返回一个反映新切片的 `Value`。其他函数对 `Value` 参数进行操作，如代码清单 28-9 所示。

```go
package main
import (
"reflect"
//"strings"
//"fmt"
)
func pickValues(slice interface{}, indices ...int) interface{} {
sliceVal := reflect.ValueOf(slice)
if (sliceVal.Kind() == reflect.Slice) {
newSlice := reflect.MakeSlice(sliceVal.Type(), 0, 10)
for _, index := range indices {
newSlice = reflect.Append(newSlice, sliceVal.Index(index))
}
return newSlice
}
return nil
}
func main() {
name := "Alice"
city := "London"
hobby := "Running"
slice := []string { name, city, hobby, "Bob", "Paris", "Soccer" }
picked := pickValues(slice, 0, 3, 5)
Printfln("Picked values: %v", picked)
}
代码清单 28-9
在 reflection 文件夹的 main.go 文件中创建新切片
```

`pickValues` 函数使用从现有切片反映出的 `Type` 创建一个新切片，并使用 `Append` 函数将值添加到新切片。编译并执行项目，你将看到以下输出：

```
Picked values: [Alice Bob Soccer]
```



### 使用映射类型

`Type` 结构体定义了一些可用于检查映射类型的方法，如表 28-8 所述。

**表 28-8** 用于映射的 `Type` 方法

| 名称 | 描述 |
| --- | --- |
| `Key()` | 此方法返回映射键的 `Type`。 |
| `Elem()` | 此方法返回映射值的 `Type`。 |

除了这些方法之外，`reflect` 包还提供了表 28-9 中描述的函数，用于创建映射类型。

**表 28-9** 用于创建映射类型的 `reflect` 函数

| 名称 | 描述 |
| --- | --- |
| `MapOf(keyType, valType)` | 此函数返回一个新的 `Type`，该 `Type` 反映具有指定键类型和值类型的映射类型，这两种类型均使用 `Type` 描述。 |

清单 28-10 定义了一个接收映射并报告其类型的函数。

> **注意：** 描述映射的反射很困难，因为术语 *value* 既用于指代映射中包含的键值对，也用于指代由 `Value` 接口表示的反射值。我尽量保持一致性，但您可能会发现，您需要反复阅读本节中的某些部分才能理解它们。

```go
package main
import (
"reflect"
//"strings"
//"fmt"
)
func describeMap(m interface{}) {
mapType := reflect.TypeOf(m)
if (mapType.Kind() == reflect.Map) {
Printfln("Key type: %v, Val type: %v", mapType.Key(), mapType.Elem())
} else {
Printfln("Not a map")
}
}
func main() {
pricesMap := map[string]float64 {
"Kayak": 279, "Lifejacket": 48.95, "Soccer Ball": 19.50,
}
describeMap(pricesMap)
}
```

*清单 28-10：在 `reflection` 文件夹的 `main.go` 文件中处理映射类型*

`Kind` 方法用于确认 `describeMap` 函数接收到了一个映射，而 `Key` 和 `Elem` 方法用于写出键和值的类型。编译并执行该项目，您将看到以下输出：

```
Key type: string, Val type: float64
```

### 使用映射值

`Value` 接口定义了表 28-10 中描述的方法，用于处理映射值。

**表 28-10** 用于处理映射的 `Value` 方法

| 名称 | 描述 |
| --- | --- |
| `MapKeys()` | 此方法返回一个 `[]Value`，其中包含映射的所有键。 |
| `MapIndex(key)` | 此方法返回与指定键对应的 `Value`，该键也作为 `Value` 表达。如果映射不包含指定键，则返回零值，这可以通过调用 `IsValid` 方法检测到，该方法将返回 `false`，如第 27 章所述。 |
| `MapRange()` | 此方法返回一个 `*MapIter`，允许迭代映射内容，如下表所述。 |
| `SetMapIndex(key, val)` | 此方法设置指定的键和值，两者都使用 `Value` 接口表达。 |
| `Len()` | 此方法返回映射中包含的键值对数量。 |

`reflect` 包提供了两种不同的方式来枚举映射的内容。第一种是使用 `MapKeys` 方法获取一个包含反射键值的切片，然后使用 `MapIndex` 方法获取每个反射的映射值，如清单 28-11 所示。

```go
package main
import (
"reflect"
//"strings"
//"fmt"
)
func printMapContents(m interface{}) {
mapValue := reflect.ValueOf(m)
if (mapValue.Kind() == reflect.Map) {
for _, keyVal := range mapValue.MapKeys() {
reflectedVal := mapValue.MapIndex(keyVal)
Printfln("Map Key: %v, Value: %v", keyVal, reflectedVal)
}
} else {
Printfln("Not a map")
}
}
func main() {
pricesMap := map[string]float64 {
"Kayak": 279, "Lifejacket": 48.95, "Soccer Ball": 19.50,
}
printMapContents(pricesMap)
}
```

*清单 28-11：在 `reflection` 文件夹的 `main.go` 文件中迭代映射内容*

使用 `MapRange` 方法可以达到同样的效果，该方法返回一个指向 `MapIter` 值的指针，该值定义了表 28-11 中描述的方法。

**表 28-11** `MapIter` 结构体定义的方法

| 名称 | 描述 |
| --- | --- |
| `Next()` | 此方法前进到映射中的下一个键值对。此方法的结果是一个 `bool` 值，指示是否还有更多的键值对可读取。在调用 `Key` 或 `Value` 方法之前，必须先调用此方法。 |
| `Key()` | 此方法返回代表当前位置映射键的 `Value`。 |
| `Value()` | 此方法返回代表当前位置映射值的 `Value`。 |

`MapIter` 结构体提供了一种基于游标的方法来枚举映射，其中 `Next` 方法遍历映射内容，而 `Key` 和 `Value` 方法提供对当前位置键和值的访问。`Next` 方法的结果指示是否还有剩余的值可读取，这使得它便于与 `for` 循环一起使用，如清单 28-12 所示。

```go
package main
import (
"reflect"
//"strings"
//"fmt"
)
func printMapContents(m interface{}) {
mapValue := reflect.ValueOf(m)
if (mapValue.Kind() == reflect.Map) {
iter := mapValue.MapRange()
for iter.Next() {
Printfln("Map Key: %v, Value: %v", iter.Key(), iter.Value())
}
} else {
Printfln("Not a map")
}
}
func main() {
pricesMap := map[string]float64 {
"Kayak": 279, "Lifejacket": 48.95, "Soccer Ball": 19.50,
}
printMapContents(pricesMap)
}
```

*清单 28-12：在 `reflection` 文件夹的 `main.go` 文件中使用 `MapIter`*

重要的是，要在调用 `Key` 和 `Value` 方法之前调用 `Next` 方法，并且当 `Next` 方法返回 `false` 时，避免调用这些方法。编译并执行清单 28-11 和清单 28-12 会生成以下输出：

```
Map Key: Kayak, Value: 279
Map Key: Lifejacket, Value: 48.95
Map Key: Soccer Ball, Value: 19.5
```



#### 设置与移除 Map 中的值

`SetMapIndex` 方法用于在 map 中添加、修改或移除键值对。清单 28-13 定义了用于修改 map 的函数。

```
package main
import (
"reflect"
//"strings"
//"fmt"
)
func setMap(m interface{}, key interface{}, val interface{}) {
	mapValue := reflect.ValueOf(m)
	keyValue := reflect.ValueOf(key)
	valValue := reflect.ValueOf(val)
	if (mapValue.Kind() == reflect.Map &&
		mapValue.Type().Key() == keyValue.Type() &&
		mapValue.Type().Elem() == valValue.Type()) {
		mapValue.SetMapIndex(keyValue, valValue)
	} else {
		Printfln("Not a map or mismatched types")
	}
}
func removeFromMap(m interface{}, key interface{}) {
	mapValue := reflect.ValueOf(m)
	keyValue := reflect.ValueOf(key)
	if (mapValue.Kind() == reflect.Map &&
		mapValue.Type().Key() == keyValue.Type()) {
		mapValue.SetMapIndex(keyValue, reflect.Value{})
	}
}
func main() {
	pricesMap := map[string]float64 {
		"Kayak": 279, "Lifejacket": 48.95, "Soccer Ball": 19.50,
	}
	setMap(pricesMap, "Kayak", 100.00)
	setMap(pricesMap, "Hat", 10.00)
	removeFromMap(pricesMap, "Lifejacket")
	for k, v := range pricesMap {
		Printfln("Key: %v, Value: %v", k, v)
	}
}
清单 28-13
reflection 文件夹中 main.go 文件内修改 map 的代码
```

如第 7 章所述，map 作为参数传递时不会被复制，因此修改 map 内容无需使用指针。`setMap` 函数会检查接收到的值，确认其是一个 map，并且键和值参数具有预期的类型，然后才通过 `SetMapIndex` 方法设置值。

如果值参数是 map 值类型的零值，`SetMapIndex` 方法会从 map 中移除该键。这在处理内置类型（如 `int` 和 `float64`）时会带来问题，因为这些类型的零值也是有效的 map 条目。为了防止 `SetMapIndex` 将值设置为零值，`removeFromMap` 函数创建了一个 `Value` 结构体的实例，代码如下：

```
...
mapValue.SetMapIndex(keyValue, reflect.Value{})
...
```

这是一个便捷的小技巧，可确保将 `float64` 值从 map 中移除。编译并执行该项目，你会看到以下输出：

```
Key: Kayak, Value: 100
Key: Soccer Ball, Value: 19.5
Key: Hat, Value: 10
```

#### 创建新的 Map

`reflect` 包定义了表 28-12 中描述的函数，用于通过反射类型创建新的 map。

表 28-12

用于创建 Map 的函数

| 名称 | 描述 |
| --- | --- |
| `MakeMap(type)` | 该函数返回一个 `Value`，它反映了一个使用指定 `Type` 创建的 map。 |
| `MakeMapWithSize(type, size)` | 该函数返回一个 `Value`，它反映了一个使用指定 `Type` 和大小创建的 map。 |

创建 map 时，可以使用表 28-9 中描述的 `MapOf` 函数来创建 `Type` 值，如清单 28-14 所示。

```
package main
import (
"reflect"
"strings"
//"fmt"
)
func createMap(slice interface{}, op func(interface{}) interface{}) interface{} {
	sliceVal := reflect.ValueOf(slice)
	if (sliceVal.Kind() == reflect.Slice) {
		mapType := reflect.MapOf(sliceVal.Type().Elem(), sliceVal.Type().Elem())
		mapVal := reflect.MakeMap(mapType)
		for i := 0; i < sliceVal.Len(); i++ {
			elemVal := sliceVal.Index(i)
			mapVal.SetMapIndex(elemVal, reflect.ValueOf(op(elemVal.Interface())))
		}
		return mapVal.Interface()
	}
	return nil
}
func main() {
	names := []string { "Alice", "Bob", "Charlie"}
	reverse := func(val interface{}) interface{} {
		if str, ok := val.(string); ok {
			return strings.ToUpper(str)
		}
		return val
	}
	namesMap := createMap(names, reverse).(map[string]string)
	for k, v := range namesMap {
		Printfln("Key: %v, Value:%v", k, v)
	}
}
清单 28-14
reflection 文件夹中 main.go 文件内创建 map 的代码
```

`createMap` 函数接收一个值切片和一个函数。它会枚举切片，对每个元素调用该函数，然后使用原始值和转换后的值来填充一个 map，并将该 map 作为函数结果返回。

调用代码必须对 `createMap` 的结果进行类型断言，以缩小具体的 map 类型（本例中为 `map[string]string`）。本例中的转换函数必须编写为能接收和返回空接口，以便 `createMap` 函数能够使用它。我将在第 29 章中解释如何使用反射来改进对函数的处理。编译并执行该项目，你会看到以下输出：

```
Key: Alice, Value:ALICE
Key: Bob, Value:BOB
Key: Charlie, Value:CHARLIE
```



### 使用结构体类型

`Type` 结构体定义了可用于检查结构体类型的方法，如表 28-13 所述。

| 名称 | 描述 |
| --- | --- |
| `NumField()` | 该方法返回结构体类型定义的字段数量。 |
| `Field(index)` | 该方法返回指定索引处的字段，以 `StructField` 表示。 |
| `FieldByIndex(indices)` | 该方法接受一个 `int` 切片，用于定位嵌套字段，并以 `StructField` 表示。 |
| `FieldByName(name)` | 该方法返回具有指定名称的字段，以 `StructField` 表示。返回结果是一个表示该字段的 `StructField` 和一个指示是否找到匹配项的 `bool`。 |
| `FieldByNameFunc(func)` | 该方法将每个字段的名称（包括嵌套字段）传递给指定函数，并返回第一个使函数返回 `true` 的字段。返回结果是一个表示该字段的 `StructField` 和一个指示是否找到匹配项的 `bool`。 |

`reflect` 包使用 `StructField` 结构体表示反射字段，该结构体定义了表 28-14 所述的字段。

| 名称 | 描述 |
| --- | --- |
| `Name` | 该字段存储反射字段的名称。 |
| `PkgPath` | 该字段返回包名，用于确定字段是否已导出。对于已导出的反射字段，该字段返回空字符串。对于未导出的反射字段，该字段返回包名，这是该字段唯一可用的包。 |
| `Type` | 该字段返回反射字段的反射类型，使用 `Type` 描述。 |
| `Tag` | 该字段返回与反射字段关联的结构体标签，如"检查结构体标签"一节所述。 |
| `Index` | 该字段返回一个 `int` 切片，表示表 28-13 中 `FieldByIndex` 方法使用的字段索引。 |
| `Anonymous` | 如果反射字段是嵌入字段，该字段返回 `true`，否则返回 `false`。 |

清单 28-15 使用了表 28-13 和表 28-14 所述的方法和字段来检查结构体类型。

```
package main
import (
"reflect"
// "strings"
// "fmt"
)
func inspectStructs(structs ...interface{}) {
for _, s := range structs {
structType := reflect.TypeOf(s)
if (structType.Kind() == reflect.Struct) {
inspectStructType(structType)
}
}
}
func inspectStructType(structType reflect.Type) {
Printfln("--- Struct Type: %v", structType)
for i := 0; i < structType.NumField(); i++ {
field := structType.Field(i)
Printfln("Field %v: Name: %v, Type: %v, Exported: %v",
field.Index, field.Name, field.Type, field.PkgPath == "")
}
Printfln("--- End Struct Type: %v", structType)
}
func main() {
inspectStructs( Purchase{} )
}
清单 28-15
在 reflection 文件夹中的 main.go 文件中检查结构体类型
```

`inspectStructs` 函数定义了一个可变参数，通过该参数接收值。使用 `TypeOf` 函数获取反射类型，并使用 `Kind` 方法确认每个类型都是结构体。反射的 `Type` 被传递给 `inspectStructType` 函数，在该函数中，`NumField` 方法在 `for` 循环中使用，从而可以使用 `Field` 方法枚举结构体字段。编译并执行项目，你将看到 `Purchase` 结构体类型的详细信息：

```
--- Struct Type: main.Purchase
Field [0]: Name: Customer, Type: main.Customer, Exported: true
Field [1]: Name: Product, Type: main.Product, Exported: true
Field [2]: Name: Total, Type: float64, Exported: true
Field [3]: Name: taxRate, Type: float64, Exported: false
--- End Struct Type: main.Purchase
```

## 处理嵌套字段

清单 28-15 的输出中包含 `StructField.Index` 字段，用于标识结构体类型定义的每个字段的位置，如下所示：

```
...
Field [2]: Name: Total, Type: float64, Exported: true
...
```

`Total` 字段的索引为 2。字段的索引由它们在源代码中定义的顺序决定，这意味着改变字段的顺序会在反射结构体类型时改变它们的索引。

当检查嵌套结构体字段时，标识字段会变得更加复杂，如清单 28-16 所示。

```
package main
import (
"reflect"
// "strings"
// "fmt"
)
func inspectStructs(structs ...interface{}) {
for _, s := range structs {
structType := reflect.TypeOf(s)
if (structType.Kind() == reflect.Struct) {
inspectStructType([]int {}, structType)
}
}
}
func inspectStructType(baseIndex []int, structType reflect.Type) {
Printfln("--- Struct Type: %v", structType)
for i := 0; i < structType.NumField(); i++ {
fieldIndex := append(baseIndex, i)
field := structType.Field(i)
Printfln("Field %v: Name: %v, Type: %v, Exported: %v",
fieldIndex, field.Name, field.Type, field.PkgPath == "")
if (field.Type.Kind() == reflect.Struct) {
field := structType.FieldByIndex(fieldIndex)
inspectStructType(fieldIndex, field.Type)
}
}
Printfln("--- End Struct Type: %v", structType)
}
func main() {
inspectStructs( Purchase{} )
}
清单 28-16
在 reflection 文件夹中的 main.go 文件中检查嵌套结构体字段
```

新代码检测结构体字段，并通过递归调用 `inspectStructType` 函数进行处理。

**提示** 使用 `Type.Elem` 方法获取指针引用的类型后，相同的方法可用于检查指向结构体类型的指针字段。

编译并执行项目，你将看到以下输出，我添加了缩进以使字段之间的关系更加明显：

```
--- Struct Type: main.Purchase
Field [0]: Name: Customer, Type: main.Customer, Exported: true
--- Struct Type: main.Customer
Field [0 0]: Name: Name, Type: string, Exported: true
Field [0 1]: Name: City, Type: string, Exported: true
--- End Struct Type: main.Customer
Field [1]: Name: Product, Type: main.Product, Exported: true
--- Struct Type: main.Product
Field [1 0]: Name: Name, Type: string, Exported: true
Field [1 1]: Name: Category, Type: string, Exported: true
Field [1 2]: Name: Price, Type: float64, Exported: true
--- End Struct Type: main.Product
Field [2]: Name: Total, Type: float64, Exported: true
Field [3]: Name: taxRate, Type: float64, Exported: false
--- End Struct Type: main.Purchase
```

你可以看到对 `Purchase` 结构体类型的探索现在包含了嵌套的 `Product` 和 `Customer` 字段，并显示了这些嵌套类型定义的字段。你会注意到输出通过在其定义类型和父类型中的索引来标识每个字段，如下所示：

```
...
Field [1 2]: Name: Price, Type: float64, Exported: true
...
```

`Price` 字段在其所属的 `Product` 结构体中索引为 2，而该结构体在所属的 `Purchase` 结构体中索引为 1。

`reflect` 包在处理嵌套结构体字段的方式上存在不一致性。`FieldByIndex` 方法用于定位嵌套字段，因此如果我已知索引序列，就可以直接请求字段，例如通过传递 `[]int {1, 2}` 给 `FieldByIndex` 方法直接获取 `Price` 字段。问题在于，`FieldByIndex` 方法返回的 `StructField` 的 `Index` 字段仅返回一个元素，只反映在所属结构体中的索引。



### 这意味着从 `FieldByIndex()` 方法返回的结果，不能方便地用于后续对该方法的调用。正因如此，我需要使用自己创建的 `int` 切片来跟踪索引，并将其作为清单 28-16 中 `FieldByIndex()` 方法的参数：

```
...
fieldIndex := append(baseIndex, i)
...
field := structType.FieldByIndex(fieldIndex)
...
```

这个问题使得遍历结构体类型稍显不便，但一旦你知晓其存在，就很容易绕开。大多数项目不会尝试以这种方式遍历字段树。

## 通过名称定位字段

上一节描述的问题并不影响 `FieldByName()` 方法。该方法会搜索具有特定名称的字段，并正确设置其返回的 `StructField` 中的 `Index` 字段，如清单 28-17 所示。

```
package main
import (
"reflect"
//"strings"
//"fmt"
)
func describeField(s interface{}, fieldName string) {
structType := reflect.TypeOf(s)
field, found := structType.FieldByName(fieldName)
if (found) {
Printfln("Found: %v, Type: %v, Index: %v",
field.Name, field.Type, field.Index)
index := field.Index
for len(index) > 1 {
index = index[0: len(index) -1]
field = structType.FieldByIndex(index)
Printfln("Parent : %v, Type: %v, Index: %v",
field.Name, field.Type, field.Index)
}
Printfln("Top-Level Type: %v" , structType)
} else {
Printfln("Field %v not found", fieldName)
}
}
func main() {
describeField( Purchase{}, "Price" )
}
清单 28-17
在 reflection 文件夹的 main.go 文件中通过名称定位结构体字段
```

`describeField()` 函数使用了 `FieldByName()` 方法，它会定位第一个具有指定名称的字段，并返回一个 `Index` 字段已正确设置的 `StructField`。随后使用一个 `for` 循环反向遍历类型层次结构，依次检查每个父级。编译并执行该项目，你将看到以下结果：

```
Found: Price, Type: float64, Index: [1 2]
Parent : Product, Type: main.Product, Index: [1]
Top-Level Type: main.Purchase
```

请注意，我必须使用 `FieldByName()` 方法返回的 `StructField` 中的 `Index` 值，因为使用 `FieldByIndex()` 方法向上遍历层次结构会引入上一节描述的问题。

## 检查结构体标签

`StructField.Tag` 字段提供了与字段关联的结构体标签的详细信息。结构体标签只能通过反射来检查，这限制了它们的用途。大多数项目只在定义结构体时使用标签，以便为其他包提供指导，如第 21 章中处理 JSON 数据所示。

`Tag` 字段返回一个 `StructTag` 值，该值是 `string` 类型的别名。结构体标签本质上是一个包含编码键值对的字符串，创建 `StructTag` 别名类型的原因是为了能够定义表 28-15 中描述的方法。

**表 28-15：StructTag 类型定义的方法**

| 名称 | 描述 |
| --- | --- |
| `Get(key)` | 该方法返回一个 `string`，其中包含指定键的值；如果未定义任何值，则返回空字符串。 |
| `Lookup(key)` | 该方法返回一个 `string`，其中包含指定键的值（如果未定义任何值，则返回空字符串），并返回一个 `bool` 值：如果值已定义则为 `true`，否则为 `false`。 |

表 28-15 中的方法类似，区别在于 `Lookup()` 方法能够区分未定义值的键和值被定义为空字符串的键。清单 28-18 定义了一个带有标签的结构体，并演示了这些方法的使用。

```
package main
import (
"reflect"
//"strings"
//"fmt"
)
func inspectTags(s interface{}, tagName string) {
structType := reflect.TypeOf(s)
for i := 0; i < structType.NumField(); i++ {
field := structType.Field(i)
tag := field.Tag
valGet := tag.Get(tagName)
valLookup, ok := tag.Lookup(tagName)
Printfln("Field: %v, Tag %v: %v", field.Name, tagName, valGet)
Printfln("Field: %v, Tag %v: %v, Set: %v",
field.Name, tagName, valLookup, ok)
}
}
type Person struct {
Name string `alias:"id"`
City string `alias:""`
Country string
}
func main() {
inspectTags(Person{}, "alias")
}
清单 28-18
在 reflection 文件夹的 main.go 文件中检查结构体标签
```

`inspectTags()` 函数枚举结构体类型中定义的字段，并使用 `Get()` 和 `Lookup()` 方法来获取指定的标签。该函数应用于 `Person` 类型，该类型在其某些字段上定义了 `alias` 标签。编译并执行该项目，你将看到以下输出：

```
Field: Name, Tag alias: id
Field: Name, Tag alias: id, Set: true
Field: City, Tag alias:
Field: City, Tag alias: , Set: true
Field: Country, Tag alias:
Field: Country, Tag alias: , Set: false
```

`Lookup()` 方法返回的额外结果使得区分 `City` 字段（其 `alias` 标签定义为空字符串）和 `Country` 字段（根本没有 `alias` 标签）成为可能。

## 创建结构体类型

`reflect` 包提供了表 28-16 中描述的函数来创建结构体类型。这不是一个常用的功能，因为其结果是一种只能通过反射使用的类型。

**表 28-16：用于创建结构体类型的 reflect 函数**

| 名称 | 描述 |
| --- | --- |
| `StructOf(fields)` | 该函数使用指定的 `StructField` 切片定义字段，创建一个新的结构体类型。只能指定导出的字段。 |

清单 28-19 创建了一个结构体类型，然后检查其标签。

```
package main
import (
"reflect"
//"strings"
//"fmt"
)
func inspectTags(s interface{}, tagName string) {
structType := reflect.TypeOf(s)
for i := 0; i < structType.NumField(); i++ {
field := structType.Field(i)
tag := field.Tag
valGet := tag.Get(tagName)
valLookup, ok := tag.Lookup(tagName)
Printfln("Field: %v, Tag %v: %v", field.Name, tagName, valGet)
Printfln("Field: %v, Tag %v: %v, Set: %v",
field.Name, tagName, valLookup, ok)
}
}
func main() {
stringType := reflect.TypeOf("this is a string")
structType := reflect.StructOf([] reflect.StructField {
{ Name: "Name", Type: stringType, Tag: `alias:"id"` },
{ Name: "City", Type: stringType,Tag: `alias:""`},
{ Name: "Country", Type: stringType },
})
inspectTags(reflect.New(structType), "alias")
}
清单 28-19
在 reflection 文件夹的 main.go 文件中创建结构体类型
```

此示例创建了一个结构体，其特性与上一节中的 `Person` 结构体相同，包含 `Name`、`City` 和 `Country` 字段。这些字段通过创建 `StructField` 值来描述，这些值本身只是普通的 Go 结构体。使用 `New()` 函数根据该结构体创建一个新值，并将其传递给 `inspectTags()` 函数。编译并执行该项目，你将收到以下输出：

```
Field: typ, Tag alias:
Field: typ, Tag alias: , Set: false
Field: ptr, Tag alias:
Field: ptr, Tag alias: , Set: false
Field: flag, Tag alias:
Field: flag, Tag alias: , Set: false
```



### 使用结构体值

`Value` 接口定义了表 28-17 中描述的方法，用于处理结构体值。

**表 28-17** 用于处理结构体的 Value 方法

| 名称 | 描述 |
| --- | --- |
| `NumField()` | 该方法返回结构体值类型所定义的字段数量。 |
| `Field(index)` | 该方法返回一个 `Value`，反映指定索引处的字段。 |
| `FieldByIndex(indices)` | 该方法返回一个 `Value`，反映指定索引处的嵌套字段。 |
| `FieldByName(name)` | 该方法返回一个 `Value`，反映找到的第一个具有指定名称的字段。 |
| `FieldByNameFunc(func)` | 该方法将每个字段（包括嵌套字段）的名称传递给指定函数，并返回一个 `Value` 反映函数返回 `true` 的第一个字段，以及一个指示是否找到匹配项的 `bool`。 |

表 28-17 中的方法对应于上一节中描述的处理结构体类型的方法。一旦你理解了结构体类型的构成，就可以为你感兴趣的每个字段获取一个 `Value`，并应用基本的反射功能，如清单 28-20 所示。

```
package main
import (
"reflect"
//"strings"
//"fmt"
)
func getFieldValues(s interface{}) {
structValue := reflect.ValueOf(s)
if structValue.Kind() == reflect.Struct {
for i := 0; i < structValue.NumField(); i++ {
fieldType := structValue.Type().Field(i)
fieldVal := structValue.Field(i)
Printfln("Name: %v, Type: %v, Value: %v",
fieldType.Name, fieldType.Type, fieldVal)
}
} else {
Printfln("Not a struct")
}
}
func main() {
product := Product{ Name: "Kayak", Category: "Watersports", Price: 279 }
customer := Customer{ Name: "Acme", City: "Chicago" }
purchase := Purchase { Customer: customer, Product: product, Total: 279,
taxRate: 10 }
getFieldValues(purchase)
}
清单 28-20
在 reflection 文件夹的 main.go 文件中读取结构体字段值
```

`getFieldValues` 函数枚举结构体定义的字段，并输出字段类型和值的详细信息。编译并执行该项目，你将看到以下输出：

```
Name: Customer, Type: main.Customer, Value: {Acme Chicago}
Name: Product, Type: main.Product, Value: {Kayak Watersports 279}
Name: Total, Type: float64, Value: 279
Name: taxRate, Type: float64, Value: 10
```

## 设置结构体字段值

一旦你获取了结构体字段的 `Value`，就可以像更改任何其他反射值一样更改该字段，如清单 28-21 所示。

```
package main
import (
"reflect"
//"strings"
//"fmt"
)
func setFieldValue(s interface{}, newVals map[string]interface{}) {
structValue := reflect.ValueOf(s)
if (structValue.Kind() == reflect.Ptr &&
structValue.Elem().Kind() == reflect.Struct) {
for name, newValue := range newVals {
fieldVal := structValue.Elem().FieldByName(name)
if (fieldVal.CanSet()) {
fieldVal.Set(reflect.ValueOf(newValue))
} else if (fieldVal.CanAddr()) {
ptr := fieldVal.Addr()
if (ptr.CanSet()) {
ptr.Set(reflect.ValueOf(newValue))
} else {
Printfln("Cannot set field via pointer")
}
} else {
Printfln("Cannot set field")
}
}
} else {
Printfln("Not a pointer to a struct")
}
}
func getFieldValues(s interface{}) {
structValue := reflect.ValueOf(s)
if structValue.Kind() == reflect.Struct {
for i := 0; i < structValue.NumField(); i++ {
fieldType := structValue.Type().Field(i)
fieldVal := structValue.Field(i)
Printfln("Name: %v, Type: %v, Value: %v",
fieldType.Name, fieldType.Type, fieldVal)
}
} else {
Printfln("Not a struct")
}
}
func main() {
product := Product{ Name: "Kayak", Category: "Watersports", Price: 279 }
customer := Customer{ Name: "Acme", City: "Chicago" }
purchase := Purchase { Customer: customer, Product: product, Total: 279,
taxRate: 10 }
setFieldValue(&purchase, map[string]interface{} {
"City": "London", "Category": "Boats", "Total": 100.50,
})
getFieldValues(purchase)
}
清单 28-21
在 reflection 文件夹的 main.go 文件中设置结构体字段
```

与其他数据类型一样，反射只能通过指向结构体的指针来更改值。使用 `Elem` 方法跟随指针，以便可以使用表 28-17 中描述的方法之一获取反映字段的 `Value`。使用 `CanSet` 方法来确定是否可以设置字段。

对于非嵌套结构体的字段，需要额外步骤：使用 `Addr` 方法为字段值创建一个指针，如下所示：

```
...
} else if (fieldVal.CanAddr()) {
ptr := fieldVal.Addr()
if (ptr.CanSet()) {
ptr.Set(reflect.ValueOf(newValue))
...
```

如果没有这个额外步骤，非嵌套字段的值将无法更改。清单 28-21 中的更改改变了 `City`、`Category` 和 `Total` 字段的值，当编译并执行项目时，会产生以下输出：

```
Name: Customer, Type: main.Customer, Value: {Acme London}
Name: Product, Type: main.Product, Value: {Kayak Boats 279}
Name: Total, Type: float64, Value: 100.5
Name: taxRate, Type: float64, Value: 10
```

请注意，即使在清单 28-21 中调用了 `Addr` 方法来创建指针值之后，我仍然使用了 `CanSet` 方法。反射不能用于设置未导出的结构体字段，因此我需要执行额外的检查，以避免因尝试设置永远无法设置的字段而导致 panic。（实际上，有一些变通方法可以设置未导出的字段，但它们很糟糕，我不建议使用它们。如果你决心设置未导出的字段，网络搜索会为你提供所需的详细信息。）

## 总结

在本章中，我继续描述了 Go 的反射功能，解释了如何将它们用于指针、数组、切片、映射和结构体。在下一章中，我将完成对这个重要但复杂的功能的描述。

