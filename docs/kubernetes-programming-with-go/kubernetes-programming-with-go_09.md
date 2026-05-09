# 4. 使用通用类型

上一章描述了如何使用 Go 结构体定义 Kubernetes 资源。特别是，它解释了 Kubernetes API 库中包的内容，以及与每个 Kubernetes `Kind` 关联的每个资源的通用字段。

本章将介绍在定义 Kubernetes 资源时可以在各种位置使用的通用类型。

## 指针

Go 结构体中的可选值通常声明为指向值的指针。因此，如果你不想指定可选值，只需将其保留为 `nil` 指针；如果需要指定一个值，则必须创建该值并传递其引用。

`Kubernetes Utils` 库中的 `pointer` 包定义了用于声明此类可选值的工具函数。

```
import (
    "k8s.io/utils/pointer"
)
```



### 获取值的引用

`Int`、`Int32`、`Int64`、`Bool`、`String`、`Float32`、`Float64` 和 `Duration` 函数接受一个同类型的参数，并返回一个指向该传入值的指针。例如，`Int32` 函数的定义如下：

```
func Int32(i int32) *int32 {
return &i
}
```

然后，你可以这样使用它：

```
spec := appsv1.DeploymentSpec{
Replicas: pointer.Int32(3),
[...]
}
```

### 解引用指针

另一种方式是获取指针所引用的值，如果指针为 `nil`，则返回一个默认值。

`IntDeref`、`Int32Deref`、`Int64Deref`、`BoolDeref`、`StringDeref`、`Float32Deref`、`Float64Deref` 和 `DurationDeref` 函数接受一个指针和一个同类型的默认值作为参数，如果指针不为 `nil`，则返回它所引用的值，否则返回默认值。例如，`Int32Deref` 函数的定义如下：

```
func Int32Deref(ptr *int32, def int32) int32 {
if ptr != nil {
return *ptr
}
return def
}
```

然后，你可以这样使用它：

```
replicas := pointer.Int32Deref(spec.Replicas, 1)
```

### 比较两个引用的值

比较两个指针值可能会很有用，当它们都为 `nil`，或者它们引用了两个相等的值时，认为它们是相等的。

`Int32Equal`、`Int64Equal`、`BoolEqual`、`StringEqual`、`Float32Equal`、`Float64Equal` 和 `DurationEqual` 函数接受两个同类型的指针，如果两个指针都为 `nil`，或者它们引用了相等的值，则返回 `true`。例如，`Int32Equal` 函数的定义如下：

```
func Int32Equal(a, b *int32) bool {
if (a == nil) != (b == nil) {
return false
}
if a == nil {
return true
}
return *a == *b
}
```

然后，你可以这样使用它：

```
eq := pointer.Int32Equal(
spec1.Replicas,
spec2.Replicas,
)
```

请注意，要测试一个可选值（同时考虑其默认值）是否相等，你应该改用以下方式：

```
isOne := pointer.Int32Deref(spec.Replicas, 1) == 1
```

## 数量

`Quantity` 是一种数字的定点表示法，用于定义要分配的资源数量（例如，内存、CPU 等）。`Quantity` 能够表示的最小值是 1 纳（10^(−9)）。

在内部，`Quantity` 要么由 `Integer`（64 位）和 `Scale` 表示，要么，如果 `int64` 不够大而无法支持它，则由一个 `inf.Dec` 值表示（由 [`https://github.com/go-inf/inf`](https://github.com/go-inf/inf) 的包定义）。

```
import (
"k8s.io/apimachinery/pkg/api/resource"
)
```

### 将字符串解析为数量

定义 `Quantity` 的第一种方法是使用以下函数之一，从字符串解析其值：

- `func MustParse(str string) Quantity` – 从 `string` 解析一个数量，如果该字符串不代表一个有效数量，则引发 panic。此函数适用于当你应用一个已知有效的硬编码值时。

- `func ParseQuantity(str string) (Quantity, error)` – 从 `string` 解析一个数量，如果该字符串不代表一个有效数量，则返回一个错误。此函数适用于当你不确定该值是否有效时。

`Quantity` 可以使用符号、数字和后缀来书写。符号和后缀是可选的。后缀可以是二进制、十进制或十进制指数。

定义的二进制后缀有 `Ki` (2¹⁰)、`Mi` (2²⁰)、`Gi` (2³⁰)、`Ti` (2⁴⁰)、`Pi` (2⁵⁰) 和 `Ei` (2⁶⁰)。定义的十进制后缀有 `n` (10^(−9))、`u` (10^(−6))、`m` (10^(−3))、`"" ` (10⁰)、`k` (10³)、`M` (10⁶)、`G` (10⁹)、`T` (10¹²)、`P` (10¹⁵) 和 `E` (10¹⁸)。十进制指数后缀以 `e` 或 `E` 符号后跟十进制指数形式书写——例如，`E2` 代表 10²。

后缀格式（二进制、十进制或指数十进制）会保存在 `Quantity` 中，并在序列化该数量时使用。

使用这些函数，`Quantity` 在内部的表现形式（缩放整数或 `inf.Dec`）将根据解析后的值是否可以被表示为缩放整数来决定。

### 使用 inf.Dec 作为数量

使用以下方法将 `inf.Dec` 用作 `Quantity`：

- `func NewDecimalQuantity(b inf.Dec, format Format) *Quantity` – 通过提供一个 `inf.Dec` 值来声明一个 `Quantity`，并指定序列化时你想要使用的后缀格式。

- `func (q *Quantity) ToDec() *Quantity` – 强制一个 `Quantity`（之前通过解析字符串或使用新函数初始化得来）以 `inf.Dec` 形式存储。

- `func (q *Quantity) AsDec() *inf.Dec` – 获取 `Quantity` 的 `inf.Dec` 表示，而不修改其内部表示形式。

### 使用缩放整数作为数量

使用以下方法将缩放整数用作 `Quantity`：

- `func NewScaledQuantity(value int64, scale Scale) *Quantity` – 通过提供一个 `int64` 值和一个缩放比例来声明一个 `Quantity`。后缀格式将为十进制格式。

- `func (q *Quantity) SetScaled(value int64, scale Scale)` – 用缩放整数覆盖 `Quantity` 的值。后缀格式将保持不变。

- `func (q *Quantity) ScaledValue(scale Scale) int64` – 考虑给定的缩放比例，获取 `Quantity` 的整数表示，而不修改其内部表示形式。

- `func NewQuantity(value int64, format Format) *Quantity` – 通过提供一个 `int64` 值来声明一个 `Quantity`，缩放比例固定为 `0`，并指定序列化时要使用的后缀格式。

- `func (q *Quantity) Set(value int64)` – 用整数覆盖 `Quantity` 的值，缩放比例固定为 `0`。后缀格式将保持不变。

- `func (q *Quantity) Value() int64` – 获取 `Quantity` 的整数表示，缩放比例为 `0`，而不修改其内部表示形式。

- `func NewMilliQuantity(value int64, format Format) *Quantity` – 通过提供一个 `int64` 值来声明一个 `Quantity`，缩放比例固定为 `−3`，并指定序列化时要使用的后缀格式。

- `func (q *Quantity) SetMilli(value int64)` – 用整数覆盖 `Quantity` 的值，缩放比例固定为 `−3`。后缀格式将保持不变。

- `func (q *Quantity) MilliValue() int64` – 获取 `Quantity` 的整数表示，缩放比例为 `−3`，而不修改其内部表示形式。

### 数量的运算

以下是对 `Quantity` 进行运算的方法：

- `func (q *Quantity) Add(y Quantity)` – 将 `y` 数量加到 `q` 数量上。

- `func (q *Quantity) Sub(y Quantity)` – 从 `q` 数量中减去 `y` 数量。

- `func (q *Quantity) Cmp(y Quantity) int` – 比较 `q` 和 `y` 两个数量。如果两个数量相等，则返回 `0`；如果 `q` 大于 `y`，则返回 `1`；如果 `q` 小于 `y`，则返回 `−1`。

- `func (q *Quantity) CmpInt64(y int64) int` – 比较 `q` 数量与 `y` 整数。如果两者相等，则返回 `0`；如果 `q` 大于 `y`，则返回 `1`；如果 `q` 小于 `y`，则返回 `−1`。

- `func (q *Quantity) Neg()` – 使 `q` 成为其自身的相反数。

- `func (q Quantity) Equal(v Quantity) bool` – 测试 `q` 和 `v` 两个数量是否相等。



## IntOrString

Kubernetes 资源的某些字段同时接受整数值或字符串值。例如，端口可以通过端口号或 IANA 服务名称（如 [`https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml`](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml) 所定义）来定义。

另一个例子是可以接受整数或百分比的字段。

```go
import (
"k8s.io/apimachinery/pkg/util/intstr"
)
```

`IntOrString` 结构体定义如下：

```go
type IntOrString struct {
Type   Type
IntVal int32
StrVal string
}
```

`Type` 可以是 `Int` 或 `String`。

由于结构体的所有字段都是公开的，你可以通过直接访问其字段来创建值或从中提取信息。为了方便，以下函数可用于创建和操作 `IntOrString` 值：

*   `func FromInt(val int) IntOrString` – 声明一个包含整数值的 `IntOrString`。
*   `func FromString(val string) IntOrString` – 声明一个包含字符串值的 `IntOrString`。
*   `func Parse(val string) IntOrString` – 声明一个 `IntOrString`，首先尝试从字符串中提取整数，否则将其存储为字符串。
*   `func (intstr *IntOrString) String() string` – 将 `IntOrString` 值作为字符串返回，如果值存储为整数，则使用 `Itoa` 进行转换。
*   `func (intstr *IntOrString) IntValue() int` – 将 `IntOrString` 值作为整数返回。如果存储为字符串，则使用 `Atoi` 返回其转换后的值，如果解析失败则返回 `0`。
*   `func ValueOrDefault(intOrPercent *IntOrString, defaultValue IntOrString) *IntOrString` – 如果 `intOrPercent` 不为 nil，则返回该值，否则返回 `defaultValue`。
*   `func GetScaledValueFromIntOrPercent(intOrPercent *IntOrString, total int, roundUp bool) (int, error)` – 此函数可用于预期包含整数或百分比的 `IntOrString` 值。如果值是整数，则原样返回。

如果值是表示百分比的字符串（数字后跟 `%` 符号），则该函数返回 `total` 值的百分比，并根据 `roundUp` 向上或向下取整。如果字符串无法解析为百分比，则该函数返回一个错误。

## Time

API Machinery 库中定义的 `Time` 类型用于 Kubernetes 资源中声明时间的所有字段。它是 Go `time.Time` 类型的包装器，并提供了包装 `time.Time` 工厂方法的工厂函数。

```go
import (
metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)
```

### 工厂方法

可用的工厂方法如下：

*   `func NewTime(time time.Time) Time` – 返回一个包装了所提供的 `time.Time` 值的 `metav1.Time` 值。
*   `func Date(year int, month time.Month, day, hour, min, sec, nsec int, loc *time.Location) Time` – 基于时间的各个元素（例如，年、月、日、时、分、秒、纳秒、时区）返回一个 `metav1.Time` 值。它是 `time.Date` 函数的包装器。
*   `func Now() Time` – 返回一个包含当前本地时间的 `metav1.Time` 值。它是 `time.Now` 函数的包装器。
*   `func Unix(sec int64, nsec int64) Time` – 返回一个对应于以秒和纳秒表示的给定 Unix 时间的 `metav1.Time` 值。它是 `time.Unix` 函数的包装器。

### Time 上的操作

使用以下定义来定义 `Time` 操作：

*   `func (t *Time) DeepCopyInto(out *Time)` – 返回 `metav1.Time` 值的一个副本。
*   `func (t *Time) IsZero() bool` – 如果 `metav1.Time` 值表示 `零` 时间点，即 UTC 时间 0001 年 1 月 1 日 00:00:00，则返回 true。它是 `time.IsZero` 方法的包装器。
*   `func (t *Time) Before(u *Time) bool` – 如果 `metav1.Time` 时刻 `t` 在 `u` 之前，则返回 true。它是 `time.Before` 方法的包装器。
*   `func (t *Time) Equal(u *Time) bool` – 如果 `metav1.Time` 时刻 `t` 等于 `u`，则返回 true。它是 `time.Equal` 方法的包装器。
*   `func (t Time) Rfc3339Copy() Time` – 返回 `metav1.Time` 值 `t` 的一个副本，移除了其纳秒精度。

## 总结

本章介绍了在 Go 中定义 Kubernetes 资源时使用的常见类型。指针值用于指定可选值，量化值用于指定内存和 CPU 数量，`IntOrString` 类型用于既可以写成整数也可以写成字符串的值（例如，可以通过编号或名称定义的端口），以及 `Time`——一种包装了 Go `Time` 类型的可序列化类型。

