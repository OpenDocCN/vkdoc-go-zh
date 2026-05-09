# 6. 使用 JSON 工作

JSON 是用于在 Web 应用程序上传输数据的最流行数据格式之一。与许多现代语言一样，Go 也支持数据与 JSON 格式之间的相互转换。在本章中，我们将带你了解处理编码为 JSON 的数据的方案，特别是如何编码和解码 JSON 格式的数据以在 Go 应用程序中使用。

## JSON 包

JavaScript 对象表示法（JSON）是最流行的数据序列化和交换格式之一。Go 中支持 JSON 编码和解码的内置包称为 `encoding/json`。使用 `encoding/json` 包，你可以通过两种方式使用 JSON。你可以使用 `Unmarshal()` 和 `Marshal()` 方法对数据进行编码和解码。这两种方法都作用于 `byte` 切片。解码器需要一个 `io.Reader` 对象才能工作，而编码器则需要 `io.Writer`。在 Go 中，编码称为*编组（marshalling）*，解码称为*解组（unmarshalling）*。此外，使用 `Unmarshal()`，你可以将 `[]byte` 转换为 JSON 格式，而 `Marshal()` 则将 JSON 格式的数据转换为 `[]byte`。`encoding/json` 包定义了不同 Go 类型与 JSON 之间的转换，如表 6-1 所示。

**表 6-1.** `encoding/json` 包定义的 Go 类型与 JSON 之间的转换

| Go 类型 | JSON 类型 |
| --- | --- |
| `bool` | 布尔值 |
| `int`, `float32`, `float64`, … | 数字 |
| `string` | 字符串 |
| `nil` | 空值 |
| `[]interface{}` | 数组 |
| `map[string]interface{}` | 对象 |
| `time.Time` | 字符串 |
| `[]byte` | 字符串（base64 编码） |



## 使用 Go 语言解析 JSON

为了学习如何在 Go 语言中解析 JSON 数据，我们来看一个例子。假设你需要处理从气象测量站接收到的读数。这些读数以 JSON 消息的形式呈现，包含以下格式的时间、站点名称、温度和降雨量信息：

```
{
"time": "2022-02-15T00:00:00Z",
"station": "DS9",
"temperature": 21.6,
"rain": 0
}
```

你可以使用代码清单 6-1 来实现这个目标。第一步是定义一个名为 `Record` 的结构体，它包含 `Time`、`Station`、`Temperature` 和 `Rain` 这些成员字段。这个结构体将用于存储从气象站接收到的读数。请注意，结构体成员字段的名称都是大写字母开头的。这是因为我们希望导出它们，即让程序的其他部分可以访问它们。在 Go 语言中，`json` 包包含了将 JSON 格式数据从大写转换为小写的逻辑，所以你不用担心这个问题。`readRecord()` 函数负责从接收到的读数中读取并解码 JSON 数据。它接收一个 `io.Reader` 对象作为输入，并返回一个 `Record` 类型的结构体实例，如果发生错误则一并返回。在 `readRecord()` 函数内部，会创建一个空的记录。此外，与 `io.reader` 对象一样，我们使用 `decoder` 来解码接收到的 JSON 数据。解码器会解码数据，并将结果存储在一个名为 `rec` 的结构体实例中，通过指针传递并处理可能出现的错误。

```
package main
import (
"encoding/json"
"fmt"
"io"
"log"
"os"
"time"
)
// Record 是一条天气记录
type Record struct {
Time    time.Time
Station string
Temperature    float64 //`json: "temperature"` //摄氏度
Rain    float64 //毫米
}
func readRecord(r io.Reader) (Record, error) {
var rec Record
dec := json.NewDecoder(r)
if err := dec.Decode(&rec); err != nil {
return Record{}, err
}
return rec, nil
}
func main() {
file, err := os.Open("record.json")
fmt.Printf("file: %v\n", file)
if err != nil {
log.Fatal(err)
}
defer file.Close()
rec, err := readRecord(file)
if err != nil {
log.Fatal(err)
}
fmt.Printf("%+v\n", rec)
}
代码清单 6-1
使用 JSON 包的 Go 语言方案
```

**输出：**

```
{Time:2022-02-02 00:00:00 +0000 UTC Station: DS9 Temp:21.6 Rain:0}
```

## 使用 Go 语言解析复杂 JSON

在某些情况下，JSON 响应可能非常复杂。假设你有一个 API 用于获取一系列站点的状态。JSON 响应如下所示：

```
{
"lastCheckTime": "2022-02-15 09:20:15 PM",
"stations":[
{
"id":3,
"name": "station 3",
"status": "Active",
"statusKey": 1,
"lastCheck": {
"time": "2016-01-22 04:30:15 PM",
"success": true
}
},
{
"id":1,
"name": "station 2",
"status": "Active",
"lastCheck": {
"time": "2020-01-22 04:32:41 PM",
"success": true
}
},
]
}
```

从这个 JSON 响应中，假设你想要打印出所有最后一次检查时间超过某个特定时间阈值的站点名称。为了实现这个目标，请使用代码清单 6-2。在 `laggingStations()` 函数中，我们不用为 JSON 中的每条消息都编写特定的类型，而是使用一个名为 `reply` 的匿名结构体。该结构体包含 `lastCheckTime` 以及站点详情。务必注意确保结构体中的字段名称与 JSON 中的名称匹配。然后我们有一个 `stations`，它是一个结构体切片，包含 `name`、`status` 和 `lastCheckTime`，而后者的 `lastCheck` 字段又是一个结构体，其中 `time` 是 `string` 类型的成员字段。这个例子将 `time` 作为 `string` 处理，因为 JSON 消息中的时间格式与 Go 语言在 JSON 中编码时间的方式不符。

```
package main
import (
"encoding/json"
"fmt"
"io"
"log"
"os"
"time"
)
/*{
"lastCheckTime": "2022-02-15 09:20:15 PM",
"stations":[
{
"id":3,
"name": "station 3",
"status": "Active",
"statusKey": 1,
"lastCheck": {
"time": "2016-01-22 04:30:15 PM",
"success": true
}
},
{
"id":1,
"name": "station 2",
"status": "Active",
"lastCheck": {
"time": "2020-01-22 04:32:41 PM",
"success": true
}
},
]
}
*/
// laggingStations 返回检查时间滞后的站点
func laggingStations(r io.Reader, timeout time.Duration) ([]string, error) {
var reply struct {
LastCheckTime string
Stations      []struct {
Name      string
Status    string
LastCheck struct {
Time string
}
}
}
dec := json.NewDecoder(r)
if err := dec.Decode(&reply); err != nil {
return nil, err
}
checkTime, err := parseTime(reply.LastCheckTime)
if err != nil {
return nil, err
}
var lagging []string
for _, station := range reply.Stations {
if station.Status != "Active" {
continue
}
lastCheck, err := parseTime(station.LastCheck.Time)
if err != nil {
return nil, err
}
if checkTime.Sub(lastCheck) > timeout {
lagging = append(lagging, station.Name)
}
}
return lagging, nil
}
func parseTime(ts string) (time.Time, error) {
return time.Parse("2006-01-02 15:04:05 PM", ts)
}
func main() {
file, err := os.Open("stations.json")
if err != nil {
log.Fatal(err)
}
defer file.Close()
lagging, err := laggingStations(file, time.Minute)
if err != nil {
log.Fatal(err)
}
for _, name := range lagging {
fmt.Println(name)
}
}
代码清单 6-2
在 Go 语言中解析复杂 JSON 的方案
```

**输出：**

```
station 3
```

## 使用 Go 语言生成 JSON

并非所有 Go 类型都能转换为 JSON，在某些情况下，你可能需要为你的类型使用不同的 JSON 编码。如果你希望自定义类型的序列化方式，就需要实现 `json.Marshaler` 接口。如果你还想支持反序列化，则需要实现 `json.Unmarshaler` 接口。代码清单 6-3 展示了如何实现这一点。假设你有一个 `Quantity` 结构体，它包含 `value` 和 `unit` 这两个成员字段。你希望将其编码为 `string`，例如 `42.192km`，而不是编码成包含两个字段的 JSON 对象。为此，你可以在 `Quantity` 类型上定义 `MarshalJSON()` 函数。它返回一个 `byte` 切片和一个可能的错误。这个函数包含三个步骤。首先，你验证数值对应的单位不为空。其次，你将类型转换为 JSON 可识别的类型。在这个例子中，我们使用 `%f` 将 `struct` 转换为 `string`。最后一步，我们使用内置的 `json.Marshal()` 将 `string` 转换为 JSON。

```
package main
import (
"encoding/json"
"fmt"
"os"
)
// Quantity 是数值和单位的组合 (例如 2.7cm)
type Quantity struct {
Value float64
Unit  string
}
// MarshalJSON 实现了 json.Marshaler 接口
// 示例编码: "42.192km"
func (q *Quantity) MarshalJSON() ([]byte, error) {
if q.Unit == "" {
return nil, fmt.Errorf("empty unit")
}
text := fmt.Sprintf("%f%s", q.Value, q.Unit)
return json.Marshal(text)
}
func main() {
q := Quantity{1.78, " meter"}
json.NewEncoder(os.Stdout).Encode(&q) //"1.780000meter"
}
代码清单 6-3
在 Go 语言中生成 JSON 的方案
```

**输出：**

```
1.780000meter
```



## 处理 JSON 中的缺失值与零值

`encoding/json` 包只会填充 JSON 对象中存在的结构体字段。默认情况下，Go 会将所有结构体字段初始化为其零值。换句话说，数字被设置为 0，字符串被视作空字符串。切片、映射和通道被设置为 `nil`，布尔值被设置为 `false`。

假设你有一条来自收据的 `lineItem`（行项目），如图 6-1 所示，其中包含价格、折扣和数量。你期望实现以下逻辑：

- 如果 JSON 对象中没有 `quantity`（数量）字段，则 `quantity` 应为 1。
- 否则，如果存在 `quantity`，则其值必须大于 0。

一种方案是将 `quantity` 设为指向整数的指针（`Quantity *int`）。在这种情况下，如果数量为 `nil`，你就知道 JSON 对象中没有该值。否则，就存在某些值，但使用指针既繁琐又容易出错。

更好的方法如代码清单 6-4 所示，你定义了一个 `NewLineItem()` 函数，该函数返回一个已将 `quantity` 初始化为 1 的 `lineItem`。然后，当从字节切片 `[]byte` 中反序列化 `lineItem` 时，你创建 `NewLineItem` 实例，这意味着 `quantity` 字段的值为 1。接着你可以从 JSON 中解析它。也就是说，如果 JSON 中存在 quantity 字段，它将覆盖存储在 `li` 中的值；否则，该值将保持为 1。然后你对数量字段执行值验证。为了检查此代码的运行情况，我们在 `main()` 函数中传递了两个不同的值。

```go
package main
import (
"encoding/json"
"fmt"
)
//LineItem 表示收据中的一行
type LineItem struct {
SKU      string
Price    float64
Discount float64
Quantity int
}
//NewLineItem 返回一个包含默认值的新行项目
func NewLineItem() LineItem {
return LineItem{
Quantity: 1,
}
}
func unmarshalLineItem(data []byte) (LineItem, error) {
li := NewLineItem()
if err := json.Unmarshal(data, &li); err != nil {
return LineItem{}, nil
}
if li.Quantity < 1 {
return LineItem{}, fmt.Errorf("数量不合法")
}
return li, nil
}
func main() {
data := []byte(`{"sku": "x3xs", "price": 1.2}`)
li, err := unmarshalLineItem(data)
if err != nil {
fmt.Println("错误：", err)
} else {
fmt.Printf("%#v\n", li)
}
data = []byte(`{"sku": "x3xs", "price": 1.2, "quantity": 0}`)
li, err = unmarshalLineItem(data)
if err != nil {
fmt.Println("错误：", err)
} else {
fmt.Printf("%#v\n", li)
}
}
```

**输出：**

```
main.LineItem{SKU:"x3xs", Price:1.2, Discount:0, Quantity:1}
错误：数量不合法
```

### 使用 `mapstructure` 处理任意 JSON

假设你有一个端点，用于接收 JSON 格式的作业相关请求。该请求可以是以下格式的启动请求或状态请求：

```json
{
"type": "start",
"user": "joe",
"count": 3
}
{
"type": "status",
"Id": "689fae62"
}
```

内置的 `encoding/json` 包无法处理任意 JSON。为了克服这一缺点，你可以使用 `mapstructure` 外部包。`mapstructure` 包的用法看起来与 JSON 包相似，但它并非作用于字节切片，而是作用于 `map[string]interface{}`，这在 Go 中代表一个任意的 JSON 对象。代码清单 6-5 说明了这一概念。首先，导入 `mapstructure` 包。然后定义两个结构体——`StartJob` 和 `JobStatus`。`handleStart()` 和 `handleStatus()` 函数分别处理 `start` 和 `status` 请求。`handleRequest()` 函数接收一个 `[]byte` 切片作为输入，并返回任何错误。该函数首先创建一个 `map[string]interface{}`，然后使用 `encoding/json` 包中的内置函数 `json.Unmarshal()` 将数据反序列化到该映射中。接着，使用逗号 ok 惯用语从映射中获取 type 的值。逗号 ok 惯用语用于判断值是否存在。如果值不存在，则返回一条包含消息“`JSON 中缺少类型`”的错误。`val` 是一个空接口；因此，需要通过使用逗号 ok 惯用语进行类型断言，将其转换为 `string` 类型。然后你使用 `switch` 语句来确定请求类型。

```go
package main
import (
"encoding/json"
"fmt"
"mapstructure"
)
//StartJob 是启动作业的请求
type StartJob struct {
Type  string
User  string
Count int
}
//JobStatus 是查询作业状态的请求
type JobStatus struct {
Type string
ID   string
}
func handleStart(req StartJob) error {
fmt.Printf("启动：%#v\n", req)
return nil
}
func handleStatus(req JobStatus) error {
fmt.Printf("状态：%#v\n", req)
return nil
}
func handleRequest(data []byte) error {
var m map[string]interface{}
if err := json.Unmarshal(data, &m); err != nil {
return err
}
val, ok := m["type"]
if !ok {
return fmt.Errorf("JSON 中缺少 'type' 字段")
}
typ, ok := val.(string)
if !ok {
return fmt.Errorf("'type' 不是字符串类型")
}
switch typ {
case "start":
var sj StartJob
if err := mapstructure.Decode(m, &sj); err != nil {
return fmt.Errorf("错误的 'start' 请求：%w", err)
}
return handleStart(sj)
case "status":
var js JobStatus
if err := mapstructure.Decode(m, &js); err != nil {
return fmt.Errorf("错误的 'status' 请求：%w", err)
}
return handleStatus(js)
}
return fmt.Errorf("未知的请求类型：%q", typ)
}
func main() {
data := []byte(`{"type": "start", "user": "joe", "count": 7}`)
if err := handleRequest(data); err != nil {
fmt.Println("错误：", err)
}
data = []byte(`{"type": "status", "id": "seven"}`)
if err := handleRequest(data); err != nil {
fmt.Println("错误：", err)
}
}
```

**输出：**

```
start: main.StartJob{Type:"start", User:"joe", Count:7}
status: main.JobStatus{Type:"status", ID:"seven"}
```

请注意，要成功运行代码清单 6-5，你首先需要从 [`www.github.com/mitchellh/mapstructure`](http://www.github.com/mitchellh/mapstructure) 导入 `mapstructure` 包。为此，请从包含代码清单 6-5 所示程序的文件所在目录下运行代码清单 6-6 中所示的命令。

```
go mod init "github.com/mitchellh/mapstructure"
```

## 总结

本章提供了 Go 实践方案，让您获得在 Go 中处理 JSON 结构化数据的实践经验。Go 提供了一个名为 `json` 的内置包，用于处理 JSON 格式的数据。在本章中，您学习了使用 JSON 包、解析复杂 JSON 数据以及处理 JSON 格式化数据中缺失值或零值的方法。本章还提供了一个使用 `mapstructure` 包处理任意 JSON 数据的 Go 实践方案。

现在您已经了解了如何处理 JSON 数据，在下一章中，您将通过构建和验证 HTTP 服务器来学习如何处理 HTTP 调用。

