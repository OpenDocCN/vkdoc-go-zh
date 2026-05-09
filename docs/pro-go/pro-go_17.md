# 29. 使用反射，第 3 部分

在本章中，我将完成对 Go 反射支持功能的描述，该内容始于第 27 章，并在第 28 章中继续。在本章中，我将解释如何将反射用于函数、方法、接口和通道。表 29-1 总结了本章内容。

**表 29-1** 章节总结

| 问题 | 解决方案 | 清单 |
| --- | --- | --- |
| 检查并调用反射函数 | 使用针对函数的 `Type` 和 `Value` 方法 | 5–7 |
| 创建新函数 | 使用 `FuncOf` 和 `MakeFunc` 函数 | 8, 9 |
| 检查并调用反射方法 | 使用针对方法的 `Type` 和 `Value` 方法 | 10–12 |
| 检查反射接口 | 使用针对接口的 `Type` 和 `Value` 方法 | 13–15 |
| 检查并使用反射通道 | 使用针对通道的 `Type` 和 `Value` 方法 | 16–19 |



## 准备工作

本章将继续使用第 28 章中的 `reflection` 项目。为做好准备，请在 `reflection` 项目中添加一个名为 `interfaces.go` 的文件，其内容如代码清单 29-1 所示。

**提示**  
你可以从 [`https://github.com/apress/pro-go`](https://github.com/apress/pro-go) 下载本章以及本书其他所有章节的示例项目。如果在运行示例时遇到问题，请参阅第 2 章获取帮助。

```go
package main
import "fmt"
type NamedItem interface {
GetName() string
unexportedMethod()
}
type CurrencyItem interface {
GetAmount() string
currencyName() string
}
func (p *Product) GetName() string {
return p.Name
}
func (c *Customer) GetName() string {
return c.Name
}
func (p *Product) GetAmount() string {
return fmt.Sprintf("$%.2f", p.Price)
}
func (p *Product) currencyName() string {
return "USD"
}
func (p *Product) unexportedMethod() {}
```
*代码清单 29-1：`reflection` 文件夹中 `interfaces.go` 文件的内容*

在 `reflection` 文件夹中添加一个名为 `functions.go` 的文件，其内容如代码清单 29-2 所示。

```go
package main
func Find(slice []string, vals... string) (matches bool) {
for _, s1 := range slice {
for _, s2 := range vals {
if s1 == s2 {
matches = true
return
}
}
}
return
}
```
*代码清单 29-2：`reflection` 文件夹中 `functions.go` 文件的内容*

在 `reflection` 文件夹中添加一个名为 `methods.go` 的文件，其内容如代码清单 29-3 所示。

```go
package main
func (p Purchase) calcTax(taxRate float64) float64 {
return p.Price * taxRate
}
func (p Purchase) GetTotal() float64 {
return p.Price + p.calcTax(.20)
}
```
*代码清单 29-3：`reflection` 文件夹中 `methods.go` 文件的内容*

在 `reflection` 文件夹中运行如代码清单 29-4 所示的命令，以编译并执行项目。

```bash
go run .
```
*代码清单 29-4：编译并执行项目*

该命令将产生以下输出：

```
Name: Customer, Type: main.Customer, Value: {Acme London}
Name: Product, Type: main.Product, Value: {Kayak Boats 279}
Name: Total, Type: float64, Value: 100.5
Name: taxRate, Type: float64, Value: 10
```

### 使用函数类型

如第 9 章所述，函数在 Go 中是一种类型，正如你所预料的，函数可以通过反射进行检查和使用。`Type` 结构体定义了一些可用于检查函数类型的方法，如表 29-2 所述。

*表 29-2：用于处理函数的 Type 方法*

| 名称 | 描述 |
| --- | --- |
| `NumIn()` | 此方法返回函数定义的参数数量。 |
| `In(index)` | 此方法返回一个 `Type`，反映指定索引处的参数。 |
| `IsVariadic()` | 如果最后一个参数是可变参数，此方法返回 `true`。 |
| `NumOut()` | 此方法返回函数定义的结果数量。 |
| `Out(index)` | 此方法返回一个 `Type`，反映指定索引处的结果。 |

代码清单 29-5 使用反射来描述一个函数。

```go
package main
import (
"reflect"
//"strings"
//"fmt"
)
func inspectFuncType(f interface{}) {
funcType := reflect.TypeOf(f)
if (funcType.Kind() == reflect.Func) {
Printfln("Function parameters: %v", funcType.NumIn())
for i := 0 ; i < funcType.NumIn(); i++ {
paramType := funcType.In(i)
if (i < funcType.NumIn() -1) {
Printfln("Parameter #%v, Type: %v", i, paramType)
} else {
Printfln("Parameter #%v, Type: %v, Variadic: %v", i, paramType,
funcType.IsVariadic())
}
}
Printfln("Function results: %v", funcType.NumOut())
for i := 0 ; i < funcType.NumOut(); i++ {
resultType := funcType.Out(i)
Printfln("Result #%v, Type: %v", i, resultType)
}
}
}
func main() {
inspectFuncType(Find)
}
```
*代码清单 29-5：在 `reflection` 文件夹的 `main.go` 文件中反射一个函数*

`inspectFuncType` 函数使用表 29-2 中描述的方法来检查函数类型，并报告其参数和结果。编译并执行该项目，你将看到以下输出，它描述了在代码清单 29-2 中定义的 `Find` 函数：

```
Parameter #0, Type: []string
Parameter #1, Type: []string, Variadic: true
Function results: 1
Result #0, Type: bool
```

输出显示 `Find` 函数有两个参数，最后一个参数是可变参数，并有一个返回值。



### 使用函数值

`Value` 接口定义了表 29-3 中描述的方法用于调用函数。

**表 29-3** 用于调用函数的 `Value` 方法

| 名称 | 描述 |
| --- | --- |
| `Call(params)` | 此函数使用 `[]Value` 作为参数来调用所反射的函数。结果是一个包含函数返回值的 `[]Value`。提供的参数值必须与函数定义的参数相匹配。 |

`Call` 方法调用一个函数并返回一个包含结果的切片。函数的参数使用 `Value` 切片指定，并且 `Call` 方法会自动检测可变参数。结果作为另一个 `Value` 切片返回，如清单 29-6 所示。

```go
package main
import (
"reflect"
//"strings"
//"fmt"
)
func invokeFunction(f interface{}, params ...interface{}) {
paramVals := []reflect.Value {}
for _, p := range params {
paramVals = append(paramVals, reflect.ValueOf(p))
}
funcVal := reflect.ValueOf(f)
if (funcVal.Kind() == reflect.Func) {
results := funcVal.Call(paramVals)
for i, r := range results {
Printfln("Result #%v: %v", i, r)
}
}
}
func main() {
names := []string { "Alice", "Bob", "Charlie" }
invokeFunction(Find, names, "London", "Bob")
}
Listing 29-6
Invoking a Function in the main.go File in the reflection Folder
```

编译并执行该项目，您将看到以下输出：

```
Result #0: true
```

以这种方式调用函数并不是常见需求，因为调用代码本可以直接调用函数，但这个示例清晰地展示了 `Call` 方法的用法，并强调了参数和结果都使用 `Value` 切片来表示。清单 29-7 提供了一个更实际的示例。

```go
package main
import (
"reflect"
"strings"
//"fmt"
)
func mapSlice(slice interface{}, mapper interface{}) (mapped []interface{}) {
sliceVal := reflect.ValueOf(slice)
mapperVal := reflect.ValueOf(mapper)
mapped = []interface{} {}
if sliceVal.Kind() == reflect.Slice && mapperVal.Kind() == reflect.Func &&
mapperVal.Type().NumIn() == 1 &&
mapperVal.Type().In(0) == sliceVal.Type().Elem() {
for i := 0; i < sliceVal.Len(); i++ {
result := mapperVal.Call([]reflect.Value {sliceVal.Index(i)})
for _, r := range result {
mapped = append(mapped, r.Interface())
}
}
}
return
}
func main() {
names := []string { "Alice", "Bob", "Charlie" }
results := mapSlice(names, strings.ToUpper)
Printfln("Results: %v", results)
}
Listing 29-7
Calling a Function on Slice Elements in the main.go File in the reflection Folder
```

`mapSlice` 函数接受一个切片和一个函数，将每个切片元素传递给该函数，并返回结果。可能会倾向于像这样描述函数参数以指定参数数量：

```go
...
mapper func(interface{}) interface{}
...
```

这种方法的问题在于，它限制了可使用的函数，仅能使用那些参数和结果类型被定义为空接口的函数。相反，将整个函数指定为一个单独的空接口值，如下所示：

```go
...
func mapSlice(slice interface{}, mapper interface{}) (mapped []interface{}) {
...
```

这允许使用任何函数，但需要检查该函数以确保它能按预期使用：

```go
...
if sliceVal.Kind() == reflect.Slice && mapperVal.Kind() == reflect.Func &&
mapperVal.Type().NumIn() == 1 &&
mapperVal.Type().In(0) == sliceVal.Type().Elem() {
...
```

这些检查确保了函数定义了一个参数，并且该参数类型与切片元素类型匹配。编译并执行该项目，您将看到以下结果：

```
Results: [ALICE BOB CHARLIE]
```

## 创建和调用新的函数类型和值

`reflect` 包定义了表 29-4 中描述的函数，用于创建新的函数类型和值。

**表 29-4** 用于创建新函数类型和函数值的 `reflect` 函数

| 名称 | 描述 |
| --- | --- |
| `FuncOf(params, results, variadic)` | 此函数创建一个新的 `Type`，它反映了具有指定参数和结果的函数类型。最后一个参数指定该函数类型是否具有可变参数。参数和结果通过 `Type` 切片指定。 |
| `MakeFunc(type, fn)` | 此函数返回一个 `Value`，它反映了一个新函数，该函数是对函数 `fn` 的封装。该函数必须接受一个 `Value` 切片作为其唯一参数，并返回一个 `Value` 切片作为其唯一结果。 |

`FuncOf` 函数的一个用途是创建一个类型签名，并用它来检查函数值的签名，取代上一节中执行的检查，如清单 29-8 所示。

```go
package main
import (
"reflect"
"strings"
//"fmt"
)
func mapSlice(slice interface{}, mapper interface{}) (mapped []interface{}) {
sliceVal := reflect.ValueOf(slice)
mapperVal := reflect.ValueOf(mapper)
mapped = []interface{} {}
if sliceVal.Kind() == reflect.Slice && mapperVal.Kind() == reflect.Func {
paramTypes := []reflect.Type { sliceVal.Type().Elem() }
resultTypes := []reflect.Type {}
for i := 0; i < mapperVal.Type().NumOut(); i++ {
resultTypes = append(resultTypes, mapperVal.Type().Out(i))
}
expectedFuncType := reflect.FuncOf(paramTypes,
resultTypes, mapperVal.Type().IsVariadic())
if (mapperVal.Type() == expectedFuncType) {
for i := 0; i < sliceVal.Len(); i++ {
result := mapperVal.Call([]reflect.Value {sliceVal.Index(i)})
for _, r := range result {
mapped = append(mapped, r.Interface())
}
}
} else {
Printfln("Function type not as expected")
}
}
return
}
func main() {
names := []string { "Alice", "Bob", "Charlie" }
results := mapSlice(names, strings.ToUpper)
Printfln("Results: %v", results)
}
Listing 29-8
Creating a Function Type in the main.go File in the reflection Folder
```

这种方法并不简洁，主要原因是我希望接受那些参数类型与切片元素类型相同但结果类型任意的函数。获取切片元素类型很简单，但我需要做一些工作来创建一个反映映射函数结果的 `Type` 切片，以确保我创建的类型能够被正确比较。编译并执行该项目，您将看到以下输出：

```
Results: [ALICE BOB CHARLIE]
```

`FuncOf` 函数由 `MakeFunc` 函数补充，后者使用函数类型作为模板来创建新函数。清单 29-9 演示了如何使用 `MakeFunc` 函数创建一个可复用的类型化映射函数。



## 关于反射创建函数的示例

```go
package main
import (
"reflect"
"strings"
"fmt"
)
func makeMapperFunc(mapper interface{}) interface{} {
mapVal := reflect.ValueOf(mapper)
if mapVal.Kind() == reflect.Func && mapVal.Type().NumIn() == 1 &&
mapVal.Type().NumOut() == 1 {
inType := reflect.SliceOf( mapVal.Type().In(0))
inTypeSlice := []reflect.Type { inType }
outType := reflect.SliceOf( mapVal.Type().Out(0))
outTypeSlice := []reflect.Type { outType }
funcType := reflect.FuncOf(inTypeSlice, outTypeSlice, false)
funcVal := reflect.MakeFunc(funcType,
func (params []reflect.Value) (results []reflect.Value) {
srcSliceVal := params[0]
resultsSliceVal := reflect.MakeSlice(outType, srcSliceVal.Len(), 10)
for i := 0; i < srcSliceVal.Len(); i++ {
r := mapVal.Call([]reflect.Value { srcSliceVal.Index(i)})
resultsSliceVal.Index(i).Set(r[0])
}
results = []reflect.Value { resultsSliceVal }
return
})
return funcVal.Interface()
}
Printfln("Unexpected types")
return nil
}
func main() {
lowerStringMapper := makeMapperFunc(strings.ToLower).(func([]string)[]string)
names := []string { "Alice", "Bob", "Charlie" }
results := lowerStringMapper(names)
Printfln("Lowercase Results: %v", results)
incrementFloatMapper := makeMapperFunc(func (val float64) float64 {
return val + 1
}).(func([]float64)[]float64)
prices := []float64 { 279, 48.95, 19.50}
floatResults := incrementFloatMapper(prices)
Printfln("Increment Results: %v", floatResults)
floatToStringMapper := makeMapperFunc(func (val float64) string {
return fmt.Sprintf("$%.2f", val)
}).(func([]float64)[]string)
Printfln("Price Results: %v", floatToStringMapper(prices))
}
```

### 理解 `makeMapperFunc` 函数

`makeMapperFunc`函数演示了反射的灵活性，但也展示了反射的冗长和复杂性。理解这个函数的最佳方式是关注其输入和输出。`makeMapperFunc`接受一个将一个值转换为另一个值的函数，其签名如下：

```go
func mapper(int) string
```

这个假设的函数接收一个`int`值并产生一个`string`结果。`makeMapperFunc`使用该函数的类型来生成一个在常规 Go 代码中可能如下所示的函数：

```go
func useMapper(slice []int) []string {
results := []string {}
for _, val := range slice {
results = append(results, mapper(val))
}
return results
}
```

`useMapper`函数是`mapper`函数的包装器。在常规 Go 代码中定义`mapper`和`useMapper`函数很容易，但它们仅适用于单一类型集合。`makeMapperFunc`使用反射，以便它可以接收任何映射函数并生成相应的包装器，然后可以与标准的 Go 类型安全特性一起使用。

#### 第一步：识别映射函数的类型

```go
inType := reflect.SliceOf( mapVal.Type().In(0))
inTypeSlice := []reflect.Type { inType }
outType := reflect.SliceOf( mapVal.Type().Out(0))
outTypeSlice := []reflect.Type { outType }
```

#### 第二步：创建包装器的函数类型

```go
funcType := reflect.FuncOf(inTypeSlice, outTypeSlice, false)
```

#### 第三步：使用 `MakeFunc` 创建包装器函数

```go
funcVal := reflect.MakeFunc(funcType,
func (params []reflect.Value) (results []reflect.Value) {
```

`MakeFunc`函数接受描述函数的`Type`以及新函数将要调用的函数。在 Listing 29-9 中，该函数枚举切片中的元素，为每个元素调用映射函数，并构建结果切片。

#### 结果：类型安全的函数

结果是类型安全的函数，尽管它需要进行类型断言：

```go
lowerStringMapper := makeMapperFunc(strings.ToLower).(func([]string)[]string)
```

`makeMapperFunc`被传入`strings.ToLower`函数，并生成一个接受`string`切片并返回`string`切片的函数。对`makeMapperFunc`的其他调用创建了将`float64`值转换为其他`float64`值的函数，以及将`float64`值转换为货币格式字符串的函数。

#### 运行结果

编译并执行项目，您将看到以下输出：

```text
Lowercase Results: [alice bob charlie]
Increment Results: [280 49.95 20.5]
Price Results: [$279.00 $48.95 $19.50]
```



### 使用方法

`Type` 结构体定义了表 29-5 所描述的方法，用于检查某个结构体所定义的方法。

**表 29-5**  
使用方法的 `Type` 方法

| 名称 | 描述 |
| --- | --- |
| `NumMethod()` | 该方法返回反射的结构体类型所定义的导出方法的数量。 |
| `Method(index)` | 该方法返回指定索引处的反射方法，以 `Method` 结构体表示。 |
| `MethodByName(name)` | 该方法返回具有指定名称的反射方法。返回结果是一个 `Method` 结构体，以及一个 `bool` 值，用于指示是否存在具有指定名称的方法。 |

> **注意：** 反射并不支持创建新方法。它只能用于检查并调用现有方法。

方法使用 `Method` 结构体表示，该结构体定义了表 29-6 所描述的字段。

**表 29-6**  
`Method` 结构体定义的字段

| 名称 | 描述 |
| --- | --- |
| `Name` | 该字段以 `string` 形式返回方法的名称。 |
| `PkgPath` | 该字段用于接口，如“使用接口”部分所述，而不用于通过结构体类型访问的方法。该字段返回一个包含包路径的 `string`。对于导出的字段，返回空字符串；对于未导出的字段，将包含结构体的包名称。 |
| `Type` | 该字段返回一个描述方法函数类型的 `Type`。 |
| `Func` | 该字段返回一个反映方法函数值的 `Value`。调用方法时，第一个参数必须是该方法被调用的结构体，如“调用方法”部分所示。 |
| `Index` | 该字段返回一个 `int`，用于指定方法索引，以便与表 29-5 中描述的 `Method` 方法配合使用。 |

> **注意：** 检查结构体时，从嵌入字段提升而来的方法也包含在本节所述方法产生的结果中。

`Value` 接口也定义了用于处理反射方法的方法，如表 29-7 所述。

**表 29-7**  
使用方法的 `Value` 方法

| 名称 | 描述 |
| --- | --- |
| `NumMethod()` | 该方法返回反射的结构体类型所定义的导出方法的数量。它会调用 `Type.NumMethod` 方法。 |
| `Method(index)` | 该方法返回一个 `Value`，用于反映指定索引处的方法函数。调用该函数时，接收器不作为第一个参数提供，如“调用方法”部分所示。 |
| `MethodByName(name)` | 该方法返回一个 `Value`，用于反映具有指定名称的方法函数。调用该函数时，接收器不作为第一个参数提供，如“调用方法”部分所示。 |

表 29-7 中的方法是便捷功能，它们提供了与表 29-5 中方法相同底层功能的访问途径，尽管在方法如何调用上存在差异，如下一节所述。

代码清单 29-10 定义了一个函数，该函数使用 `Type` 结构体提供的方法来描述某个结构体所定义的方法。

```go
package main
import (
"reflect"
//"strings"
//"fmt"
)
func inspectMethods(s interface{}) {
sType := reflect.TypeOf(s)
if sType.Kind() == reflect.Struct || (sType.Kind() == reflect.Ptr &&
sType.Elem().Kind() == reflect.Struct) {
Printfln("Type: %v, Methods: %v", sType, sType.NumMethod())
for i := 0; i < sType.NumMethod(); i++ {
method := sType.Method(i)
Printfln("Method name: %v, Type: %v",
method.Name, method.Type)
}
}
}
func main() {
inspectMethods(Purchase{})
inspectMethods(&Purchase{})
}
```

*代码清单 29-10：在 reflection 文件夹的 main.go 文件中描述方法*

Go 语言让调用方法变得非常简单，允许通过结构体指针调用为结构体定义的方法，反之亦然。然而，当使用反射来检查类型时，结果并不总是一致，这可以从项目编译并执行后的输出中看出：

```
Type: main.Purchase, Methods: 1
Method name: GetTotal, Type: func(main.Purchase) float64
Type: *main.Purchase, Methods: 2
Method name: GetAmount, Type: func(*main.Purchase) string
Method name: GetTotal, Type: func(*main.Purchase) float64
```

当对 `Purchase` 类型使用反射时，只会列出为 `Product` 定义的方法。但当对 `*Purchase` 类型使用反射时，则会列出为 `Product` 和 `*Product` 定义的方法。请注意，只有导出的方法能通过反射访问——未导出的方法无法被检查或调用。

## 调用方法

`Method` 结构体定义了 `Func` 字段，该字段返回一个 `Value`，可用于调用方法，采用本章前面介绍的相同方法，如代码清单 29-11 所示。

```go
package main
import (
"reflect"
//"strings"
//"fmt"
)
func executeFirstVoidMethod(s interface{}) {
sVal := reflect.ValueOf(s)
for i := 0; i < sVal.NumMethod(); i++ {
method := sVal.Type().Method(i)
if method.Type.NumIn() == 1 {
results := method.Func.Call([]reflect.Value{ sVal })
Printfln("Type: %v, Method: %v, Results: %v",
sVal.Type(), method.Name, results)
break
} else {
Printfln("Skipping method %v %v", method.Name, method.Type.NumIn())
}
}
}
func main() {
executeFirstVoidMethod(&Product { Name: "Kayak", Price: 279})
}
```

*代码清单 29-11：在 reflection 文件夹的 main.go 文件中调用方法*

`executeFirstVoidMethod` 函数枚举参数类型所定义的方法，并调用第一个定义一个参数的方法。当通过 `Method.Func` 字段调用方法时，第一个参数必须是接收器，也就是方法将被调用的结构体值：

```go
...
results := method.Func.Call([]reflect.Value{ sVal })
...
```

这意味着，查找具有一个参数的方法，实际上选择的是不接受参数的方法，这可以从项目编译并执行后的输出结果中看出：

```
Type: *main.Product, Method: GetAmount, Results: [$279.00]
```

`executeFirstVoidMethod` 函数选择了 `GetAmount` 方法。当通过 `Value` 接口调用方法时，不需要指定接收器，如代码清单 29-12 所示。

```go
package main
import (
"reflect"
//"strings"
//"fmt"
)
func executeFirstVoidMethod(s interface{}) {
sVal := reflect.ValueOf(s)
for i := 0; i < sVal.NumMethod(); i++ {
method := sVal.Method(i)
if method.Type().NumIn() == 0 {
results := method.Call([]reflect.Value{})
Printfln("Type: %v, Method: %v, Results: %v",
sVal.Type(), sVal.Type().Method(i).Name, results)
break
} else {
Printfln("Skipping method %v %v",
sVal.Type().Method(i).Name, method.Type().NumIn())
}
}
}
func main() {
executeFirstVoidMethod(&Product { Name: "Kayak", Price: 279})
}
```

*代码清单 29-12：在 reflection 文件夹的 main.go 文件中通过 Value 调用方法*

为了找到可以在不提供额外参数的情况下调用的方法，我必须寻找零参数的方法，因为接收器不是显式指定的。相反，接收器是通过调用 `Call` 方法的 `Value` 来确定的：

```go
...
results := method.Call([]reflect.Value{})
...
```

此示例产生的输出与代码清单 29-11 中的代码相同。



### 使用接口

`Type` 结构体定义了可用于检查接口类型的方法，详见表 29-8。这些方法中的大多数也同样适用于结构体（如前几节所示），但其行为会略有不同。

表 29-8
用于接口的 Type 方法

| 名称                 | 描述                                                                                                                           |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `Implements(type)`   | 如果被反射的 `Value` 实现了指定的接口（该接口同样由 `Value` 表示），则此方法返回 `true`。                                           |
| `Elem()`             | 此方法返回一个 `Value`，该 `Value` 反映了接口所包含的值。                                                                         |
| `NumMethod()`        | 此方法返回为被反射的结构体类型定义的已导出方法数量。                                                                               |
| `Method(index)`      | 此方法返回指定索引处的被反射方法，该方法由 `Method` 结构体表示。                                                              |
| `MethodByName(name)` | 此方法返回具有指定名称的被反射方法。结果是一个 `Method` 结构体和一个 `bool` 值，用于指示是否存在具有指定名称的方法。                 |

在使用反射处理接口时必须小心，因为 `reflect` 包总是从一个值开始，并会尝试处理该值的底层类型。解决此问题的最简单方法是转换一个 `nil` 值，如代码清单 29-13 所示。

```go
package main
import (
"reflect"
//"strings"
//"fmt"
)
func checkImplementation(check interface{}, targets ...interface{}) {
checkType := reflect.TypeOf(check)
if (checkType.Kind() == reflect.Ptr &&
checkType.Elem().Kind() == reflect.Interface) {
checkType := checkType.Elem()
for _, target := range targets {
targetType := reflect.TypeOf(target)
Printfln("Type %v implements %v: %v",
targetType, checkType, targetType.Implements(checkType))
}
}
}
func main() {
currencyItemType := (*CurrencyItem)(nil)
checkImplementation(currencyItemType, Product{}, &Product{}, &Purchase{})
}
```

代码清单 29-13
在 reflection 文件夹的 main.go 文件中反射一个接口

为了指定我想要检查的接口，我将 `nil` 转换为一个指向该接口的指针，如下所示：

```go
...
currencyItemType := (*CurrencyItem)(nil)
...
```

这必须使用指针完成，随后在 `checkImplementation` 函数中使用 `Elem` 方法跟踪该指针，以获取一个反映该接口的 `Type`（在此示例中即 `CurrencyItem`）：

```go
...
if (checkType.Kind() == reflect.Ptr &&
checkType.Elem().Kind() == reflect.Interface) {
checkType := checkType.Elem()
...
```

之后，就可以轻松地使用 `Implements` 方法检查某个类型是否实现了该接口。编译并执行该项目，你将看到以下输出：

```
Type main.Product implements main.CurrencyItem: false
Type *main.Product implements main.CurrencyItem: true
Type *main.Purchase implements main.CurrencyItem: true
```

输出显示 `Product` 结构体并未实现该接口，但 `*Product` 实现了，因为 `*Product` 是用于实现 `CurrencyItem` 所需方法的接收者类型。`*Purchase` 类型也实现了该接口，因为它嵌套了定义所需方法的结构体字段。

## 从接口获取底层值

虽然反射通常会生成具体类型，但在某些情况下，必须使用 `Elem` 方法从接口转移到实现该接口的类型，如代码清单 29-14 所示。

```go
package main
import (
"reflect"
//"strings"
//"fmt"
)
type Wrapper struct {
NamedItem
}
func getUnderlying(item Wrapper, fieldName string) {
itemVal := reflect.ValueOf(item)
fieldVal := itemVal.FieldByName(fieldName)
Printfln("Field Type: %v", fieldVal.Type())
if (fieldVal.Kind() == reflect.Interface) {
Printfln("Underlying Type: %v", fieldVal.Elem().Type())
}
}
func main() {
getUnderlying(Wrapper{NamedItem: &Product{}}, "NamedItem")
}
```

代码清单 29-14
在 reflection 文件夹的 main.go 文件中获取底层接口值

`Wrapper` 类型定义了一个嵌套的 `NamedItem` 字段。`getUnderlying` 函数使用反射获取该字段，并输出字段类型以及通过 `Elem` 方法获取的底层类型。编译并执行该项目，你将看到以下结果：

```
Field Type: main.NamedItem
Underlying Type: *main.Product
```

字段类型是 `NamedItem` 接口，但 `Elem` 方法显示分配给 `NamedItem` 字段的底层值是 `*Product`。

## 检查接口方法

`NumMethod`、`Method` 和 `MethodByName` 方法可以用于接口类型，但结果会包含未导出的方法，而在直接检查结构体类型时则不会出现这种情况，如代码清单 29-15 所示。

```go
package main
import (
"reflect"
//"strings"
//"fmt"
)
type Wrapper struct {
NamedItem
}
func getUnderlying(item Wrapper, fieldName string) {
itemVal := reflect.ValueOf(item)
fieldVal := itemVal.FieldByName(fieldName)
Printfln("Field Type: %v", fieldVal.Type())
for i := 0; i < fieldVal.Type().NumMethod(); i++ {
method := fieldVal.Type().Method(i)
Printfln("Interface Method: %v, Exported: %v",
method.Name, method.PkgPath == "")
}
Printfln("--------")
if (fieldVal.Kind() == reflect.Interface) {
Printfln("Underlying Type: %v", fieldVal.Elem().Type())
for i := 0; i < fieldVal.Elem().Type().NumMethod(); i++ {
method := fieldVal.Elem().Type().Method(i)
Printfln("Underlying Method: %v", method.Name)
}
}
}
func main() {
getUnderlying(Wrapper{NamedItem: &Product{}}, "NamedItem")
}
```

代码清单 29-15
在 reflection 文件夹的 main.go 文件中检查接口方法

这些更改输出了从接口和底层类型获取的方法的详细信息。编译并执行该项目，你将看到以下输出：

```
Field Type: main.NamedItem
Interface Method: GetName, Exported: true
Interface Method: unexportedMethod, Exported: false

Underlying Type: *main.Product
Underlying Method: GetAmount
Underlying Method: GetName
```

`NamedItem` 接口的方法列表包含 `unexportedMethod`，而 `*Product` 的方法列表中不包含该方法。`*Product` 定义了超出接口所需的方法，这就是输出中出现 `GetAmount` 方法的原因。

可以通过接口调用方法，但必须在使用 `Call` 方法之前确保这些方法是已导出的。如果尝试调用未导出的方法，`Call` 将会触发 panic。



### 使用通道类型

`Type` 结构体定义了可用于检查通道类型的方法，见表 29-9。

**表 29-9** 通道的 Type 方法

| 名称 | 描述 |
| --- | --- |
| `ChanDir()` | 该方法返回一个 `ChanDir` 值，用于描述通道方向，该值使用表 29-10 中显示的值之一。 |
| `Elem()` | 该方法返回一个 `Type`，它反映通道携带的类型。 |

由 `ChanDir` 方法返回的 `ChanDir` 结果指示通道的方向，可与表 29-10 中描述的 `reflect` 包常量之一进行比较。

**表 29-10** ChanDir 值

| 名称 | 描述 |
| --- | --- |
| `RecvDir` | 该值表示通道可用于接收数据。当表示为字符串时，该值返回 `<-chan`。 |
| `SendDir` | 该值表示通道可用于发送数据。当表示为字符串时，该值返回 `chan<-`。 |
| `BothDir` | 该值表示通道可用于发送和接收数据。当表示为字符串时，该值返回 `chan`。 |

清单 29-16 演示了如何使用表 29-9 中的方法来检查通道类型。

```
package main
import (
"reflect"
//"strings"
//"fmt"
)
func inspectChannel(channel interface{}) {
channelType := reflect.TypeOf(channel)
if (channelType.Kind() == reflect.Chan) {
Printfln("Type %v, Direction: %v",
channelType.Elem(), channelType.ChanDir())
}
}
func main() {
var c chan<- string
inspectChannel(c)
}
清单 29-16
在 reflection 文件夹的 main.go 文件中检查通道类型
```

此示例中检查的通道是只发送的，当编译并执行项目时，会产生以下输出：

```
Type string, Direction: chan<-
```

### 使用通道值

`Value` 接口定义了表 29-11 中描述的方法，用于处理通道。

**表 29-11** 通道的 Value 方法

| 名称 | 描述 |
| --- | --- |
| `Send(val)` | 此方法在通道上发送由 `Value` 参数反映的值。此方法会阻塞，直到值被发送。 |
| `Recv()` | 此方法从通道接收一个值，该值作为用于反射的 `Value` 返回。此方法还返回一个 `bool`，指示是否接收到值，如果通道已关闭，则返回 `false`。此方法会阻塞，直到接收到值或通道已关闭。 |
| `TrySend(val)` | 此方法发送指定的值，但不会阻塞。`bool` 结果指示值是否已发送。 |
| `TryRecv()` | 此方法尝试从通道接收一个值，但不会阻塞。结果是反映接收到的值的 `Value`，以及一个指示是否已接收到值的 `bool`。 |
| `Close()` | 此方法关闭通道。 |

清单 29-17 定义了一个函数，该函数接收一个通道和一个包含将通过通道发送的值的切片。

```
package main
import (
"reflect"
//"strings"
//"fmt"
)
func sendOverChannel(channel interface{}, data interface{}) {
channelVal := reflect.ValueOf(channel)
dataVal := reflect.ValueOf(data)
if (channelVal.Kind() == reflect.Chan &&
dataVal.Kind() == reflect.Slice &&
channelVal.Type().Elem() == dataVal.Type().Elem()) {
for i := 0; i < dataVal.Len(); i++ {
val := dataVal.Index(i)
channelVal.Send(val)
}
channelVal.Close()
} else {
Printfln("Unexpected types: %v, %v", channelVal.Type(), dataVal.Type())
}
}
func main() {
values := []string { "Alice", "Bob", "Charlie", "Dora"}
channel := make(chan string)
go sendOverChannel(channel, values)
for {
if val, open := <- channel; open {
Printfln("Received value: %v", val)
} else {
break
}
}
}
清单 29-17
在 reflection 文件夹的 main.go 文件中使用通道
```

`sendOverChannel` 函数检查其接收到的类型，枚举切片中的值，并通过通道发送每个值。发送所有值后，通道关闭。编译并执行项目，你将看到以下输出：

```
Received value: Alice
Received value: Bob
Received value: Charlie
Received value: Dora
```

## 创建新的通道类型和值

`reflect` 包定义了表 29-12 中描述的函数，用于创建新的通道类型和值。

**表 29-12** 用于创建通道类型和值的 reflect 包函数

| 名称 | 描述 |
| --- | --- |
| `ChanOf(dir, type)` | 此函数返回一个 `Type`，它反映一个具有指定方向和数据类型（使用 `ChanDir` 和 `Value` 表示）的通道。 |
| `MakeChan(type, buffer)` | 此函数返回一个 `Value`，它反映一个使用指定 `Type` 和 `int` 缓冲区大小创建的新通道。 |

清单 29-18 定义了一个函数，该函数接收一个切片，并用它创建一个通道，然后使用该通道发送切片中的元素。

```
package main
import (
"reflect"
//"strings"
//"fmt"
)
func createChannelAndSend(data interface{}) interface{} {
dataVal := reflect.ValueOf(data)
channelType := reflect.ChanOf(reflect.BothDir, dataVal.Type().Elem())
channel := reflect.MakeChan(channelType, 1)
go func() {
for i := 0; i < dataVal.Len(); i++ {
channel.Send(dataVal.Index(i))
}
channel.Close()
}()
return channel.Interface()
}
func main() {
values := []string { "Alice", "Bob", "Charlie", "Dora"}
channel := createChannelAndSend(values).(chan string)
for {
if val, open := <- channel; open {
Printfln("Received value: %v", val)
} else {
break
}
}
}
清单 29-18
在 reflection 文件夹的 main.go 文件中创建通道
```

`createChannelAndSend` 函数使用切片的元素类型来创建通道类型，然后使用该类型创建一个通道。使用一个 goroutine 将切片中的元素发送到通道，并将该通道作为函数结果返回。编译并执行项目，你将看到以下输出：

```
Received value: Alice
Received value: Bob
Received value: Charlie
Received value: Dora
```



## 从多个通道中选择

第 14 章描述的通道选择功能可以在反射代码中使用，具体通过 `reflect` 包提供的 `Select` 函数实现。表 29-13 对此进行了简要说明，方便快速查阅。

**表 29-13** `reflect` 包中用于选择通道的函数

| 名称 | 描述 |
| --- | --- |
| `Select(cases)` | 该函数接受一个 `SelectCase` 切片，其中每个元素描述一组发送或接收操作。返回结果为：被执行 `SelectCase` 的 `int` 类型索引、接收到的 `Value`（若选中的 case 是读取操作），以及一个指示是否读取到值或通道是否被阻塞/关闭的 `bool` 值。 |

`SelectCase` 结构体用于表示单个 `case` 语句，其字段如表 29-14 所示。

**表 29-14** `SelectCase` 结构体字段

| 名称 | 描述 |
| --- | --- |
| `Chan` | 该字段被赋值为表示通道的 `Value`。 |
| `Dir` | 该字段被赋值为一个 `SelectDir` 值，用于指定当前 case 的通道操作类型。 |
| `Send` | 该字段被赋值为表示发送操作中要发送到通道的值的 `Value`。 |

`SelectDir` 类型是 `int` 的别名，`reflect` 包定义了表 29-15 中的常量，用于指定选择 case 的类型。

**表 29-15** `SelectDir` 常量

| 名称 | 描述 |
| --- | --- |
| `SelectSend` | 该常量表示向通道发送值的操作。 |
| `SelectRecv` | 该常量表示从通道接收值的操作。 |
| `SelectDefault` | 该常量表示 select 语句中的默认分支。 |

使用反射定义 select 语句虽然冗长，但结果可以更灵活，并且能接受比常规 Go 代码更广泛的类型。清单 29-19 使用 `Select` 函数从多个通道读取值。

```
package main

import (
	"reflect"
	//"strings"
	//"fmt"
)

func createChannelAndSend(data interface{}) interface{} {
	dataVal := reflect.ValueOf(data)
	channelType := reflect.ChanOf(reflect.BothDir, dataVal.Type().Elem())
	channel := reflect.MakeChan(channelType, 1)

	go func() {
		for i := 0; i < dataVal.Len(); i++ {
			channel.Send(dataVal.Index(i))
		}
		channel.Close()
	}()

	return channel.Interface()
}

func readChannels(channels ...interface{}) {
	channelsVal := reflect.ValueOf(channels)
	cases := []reflect.SelectCase{}

	for i := 0; i < channelsVal.Len(); i++ {
		cases = append(cases, reflect.SelectCase{
			Chan: channelsVal.Index(i).Elem(),
			Dir:  reflect.SelectRecv,
		})
	}

	for {
		caseIndex, val, ok := reflect.Select(cases)
		if ok {
			Printfln("Value read: %v, Type: %v", val, val.Type())
		} else {
			if len(cases) == 1 {
				Printfln("All channels closed.")
				return
			}
			cases = append(cases[:caseIndex], cases[caseIndex+1:]...)
		}
	}
}

func main() {
	values := []string{"Alice", "Bob", "Charlie", "Dora"}
	channel := createChannelAndSend(values).(chan string)

	cities := []string{"London", "Rome", "Paris"}
	cityChannel := createChannelAndSend(cities).(chan string)

	prices := []float64{279, 48.95, 19.50}
	priceChannel := createChannelAndSend(prices).(chan float64)

	readChannels(channel, cityChannel, priceChannel)
}
```

*清单 29-19: 在 `reflection` 文件夹的 `main.go` 文件中使用 `Select` 函数*

本示例使用 `createChannelAndSend` 函数创建了三个通道，并将它们传递给 `readChannels` 函数。`readChannels` 函数使用 `Select` 函数读取值，直到所有通道关闭。为确保仅在打开的通道上执行读取操作，当 `SelectCase` 所代表的通道关闭时，会将其从传递给 `Select` 函数的切片中移除。编译并执行该项目，你将看到以下输出：

```
Value read: London, Type: string
Value read: Alice, Type: string
Value read: Rome, Type: string
Value read: Bob, Type: string
Value read: Paris, Type: string
Value read: Charlie, Type: string
Value read: 279, Type: float64
Value read: Dora, Type: string
Value read: 48.95, Type: float64
Value read: 19.5, Type: float64
All channels closed.
```

由于使用了 goroutine 来发送通道中的值，你可能看到值的显示顺序有所不同。

## 本章小结

在本章中，我描述了用于处理函数、方法、接口和通道的反射特性，至此完成了从第 27 章开始，并在第 28 章继续的 Go 反射特性的全部介绍。在下一章中，我将介绍用于协调 goroutine 的标准库特性。

