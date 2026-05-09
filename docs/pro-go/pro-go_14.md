# 26. 使用数据库

在本章中，我将介绍用于操作 SQL 数据库的 Go 标准库。这些功能提供了对数据库所提供能力的抽象表示，并依赖驱动程序包来处理特定数据库的实现。

市面上有众多数据库的驱动程序，其列表可在 [`https://github.com/golang/go/wiki/sqldrivers`](https://github.com/golang/go/wiki/sqldrivers) 找到。数据库驱动程序以 Go 包的形式分发，并且大多数数据库都有多个驱动程序包。一些驱动程序包依赖于 `cgo`（这允许 Go 代码使用 C 库），而另一些则纯粹用 Go 编写。

本章我将使用 SQLite 数据库（在第三部分中，我将使用更复杂的数据库）。SQLite 是一款卓越的嵌入式数据库，支持广泛的平台，免费提供，且无需安装和配置服务器组件。表 26-1 列出了标准库数据库功能的背景信息。

**表 26-1.** 使用数据库的背景信息

| 问题 | 答案 |
| --- | --- |
| 它是什么？ | `database/sql` 包提供了用于操作 SQL 数据库的功能。 |
| 它为什么有用？ | 关系型数据库仍然是存储大量结构化数据最有效的方式，并广泛应用于大多数大型项目中。 |
| 如何使用它？ | 驱动程序包为特定的数据库提供支持，而 `database/sql` 包则提供一组类型，使得数据库能被一致地使用。 |
| 有陷阱或局限性吗？ | 这些功能不会自动从结果行中填充结构体字段。 |
| 有替代方案吗？ | 有一些第三方包构建在这些功能之上，用于简化或增强其使用。 |

**关于数据库的抱怨**

您可能会想联系我抱怨本书中数据库的选择。您肯定不是唯一一个，因为数据库选择是我收到最多邮件的主题之一。抱怨通常暗示我选择了“错误”的数据库，这通常意味着“不是发件人使用的数据库”。

在联系我之前，请考虑两点。首先，这不是一本关于数据库的书，SQLite 采用的零配置方法意味着大多数读者无需诊断设置和配置问题就能跟随示例操作。其次，SQLite 是一款出色的数据库，许多项目因其没有传统的服务器组件而忽视它，尽管许多项目并不需要或受益于拥有一个独立的数据库服务器。

如果您是 Oracle/DB2/MySQL/MariaDB 的忠实用户，并希望将连接代码直接复制粘贴到您的项目中，我在此致歉。但这种方法让我能够专注于 Go 语言，并且您会在所选驱动程序的文档中找到所需的代码示例。

表 26-2 总结了本章内容。

**表 26-2.** 本章小结

| 问题 | 解决方案 | 清单 |
| --- | --- | --- |
| 为项目添加对特定类型数据库的支持 | 使用 `go get` 命令添加数据库驱动程序包 | 8 |
| 打开和关闭数据库 | 使用 `Open` 函数和 `Close` 方法 | 9, 10 |
| 查询数据库 | 使用 `Query` 方法并使用 `Scan` 方法处理 `Rows` 结果 | 11–16, 22, 23 |
| 查询单行数据库记录 | 使用 `QueryRow` 方法并处理 `Row` 结果 | 17 |
| 执行不产生行结果的查询或语句 | 使用 `Exec` 方法并处理其产生的 `Result` | 18 |
| 预处理一条语句以便重复使用 | 创建一条预处理语句 | 19, 20 |
| 将多个查询作为单个工作单元执行 | 使用事务 | 21 |

## 本章准备

为准备本章，请打开一个新的命令提示符，导航到一个方便的位置，并创建一个名为 `data` 的目录。在 `data` 文件夹中运行清单 26-1 所示的命令来创建一个模块文件。

**提示** 您可以从 [`https://github.com/apress/pro-go`](https://github.com/apress/pro-go) 下载本章及本书所有其他章节的示例项目。如果在运行示例时遇到问题，请参阅第 2 章以获取帮助。

```
go mod init data
```

**清单 26-1.** 初始化模块

将一个名为 `printer.go` 的文件添加到 `data` 文件夹中，其内容如清单 26-2 所示。

```
package main
import "fmt"
func Printfln(template string, values ...interface{}) {
fmt.Printf(template + "\n", values...)
}
```

**清单 26-2.** `data` 文件夹中 `printer.go` 文件的内容

将一个名为 `main.go` 的文件添加到 `data` 文件夹中，其内容如清单 26-3 所示。

```
package main
func main() {
Printfln("Hello, Data")
}
```

**清单 26-3.** `data` 文件夹中 `main.go` 文件的内容

在 `data` 文件夹中运行清单 26-4 所示的命令来编译并运行项目。

```
go run .
```

**清单 26-4.** 编译并执行项目

该命令将产生以下输出：

```
Hello, Data
```

### 准备数据库

本章使用的 SQLite 数据库管理器将在稍后安装，但首先需要一个工具包来根据 SQL 文件创建数据库。（在第三部分中，我将演示在应用程序内部创建数据库的过程。）

为了定义将创建数据库的 SQL 文件，请将一个名为 `products.sql` 的文件添加到 `data` 文件夹中，其内容如清单 26-5 所示。

```
DROP TABLE IF EXISTS Categories;
DROP TABLE IF EXISTS Products;
CREATE TABLE IF NOT EXISTS Categories (
Id INTEGER NOT NULL PRIMARY KEY,
Name TEXT
);
CREATE TABLE IF NOT EXISTS Products (
Id INTEGER NOT NULL PRIMARY KEY,
Name TEXT,
Category INTEGER,
Price decimal(8, 2),
CONSTRAINT CatRef FOREIGN KEY(Category) REFERENCES Categories (Id)
);
INSERT INTO Categories (Id, Name) VALUES
(1, "Watersports"),
(2, "Soccer");
INSERT INTO Products (Id, Name, Category, Price) VALUES
(1, "Kayak", 1, 279),
(2, "Lifejacket", 1, 48.95),
(3, "Soccer Ball", 2, 19.50),
(4, "Corner Flags", 2, 34.95);
```

**清单 26-5.** `data` 文件夹中 `products.sql` 文件的内容

请前往 [`https://www.sqlite.org/download.html`](https://www.sqlite.org/download.html)，查找适用于您操作系统的预编译二进制文件部分，并下载工具包。我无法在本章中包含链接，因为 URL 包含软件包版本号，当您阅读本章时可能已发生变化。

解压 zip 归档文件，并将 `sqlite3` 或 `sqlite3.exe` 文件复制到 `data` 文件夹中。在 `data` 文件夹中运行清单 26-6 所示的命令来创建数据库。

**注意** 适用于 Linux 的预编译二进制文件是 32 位的，对于仅支持 64 位的操作系统，这可能需要安装一些额外的软件包。

```
./sqlite3 products.db ".read products.sql"
```

**清单 26-6.** 创建数据库

为了确认数据库已创建并填充了数据，请在 `data` 文件夹中运行清单 26-7 所示的命令。

```
./sqlite3 products.db "select * from PRODUCTS"
```

**清单 26-7.** 测试数据库

如果数据库创建正确，您将看到以下输出：

```
1|Kayak|1|279
2|Lifejacket|1|48.95
3|Soccer Ball|2|19.5
4|Corner Flags|2|34.95
```

如果您在遵循本章示例时需要重置数据库，可以删除 `products.db` 文件，然后再次运行清单 26-6 中的命令。



### 安装数据库驱动

Go 标准库包含用于简单且一致地处理数据库的功能，但依赖数据库驱动包来实现这些功能以支持每种特定的数据库引擎或服务器。如前所述，本章使用 SQLite，有一个优秀的纯 Go 驱动可用。在 `data` 文件夹中运行清单 26-8 所示的命令来安装该驱动包。

```
go get modernc.org/sqlite
清单 26-8 安装 SQL 驱动包
```

大多数数据库服务器是独立设置的，因此数据库驱动会打开一个到独立进程的连接。SQLite 是一个嵌入式数据库，并包含在驱动包中，这意味着无需额外配置。

### 打开数据库

标准库提供了 `database/sql` 包用于处理数据库。表 26-3 中描述的函数用于打开数据库，以便在应用程序中使用。

表 26-3 用于打开数据库的 `database/sql` 函数

| 名称 | 描述 |
| --- | --- |
| `Drivers()` | 此函数返回一个字符串切片，每个字符串包含一个数据库驱动的名称。 |
| `Open(driver, connectionStr)` | 此函数使用指定的驱动和连接字符串打开数据库。结果是一个指向 `DB` 结构体的指针（用于与数据库交互）和一个指示打开数据库时出现问题的 `error`。 |

在 `data` 文件夹中添加一个名为 `database.go` 的文件，代码如清单 26-9 所示。

```
package main
import (
"database/sql"
_ "modernc.org/sqlite"
)
func listDrivers() {
for _, driver := range sql.Drivers() {
Printfln("Driver: %v", driver)
}
}
func openDatabase() (db *sql.DB, err error) {
db, err = sql.Open("sqlite", "products.db")
if (err == nil) {
Printfln("Opened database")
}
return
}
清单 26-9 data 文件夹中 database.go 文件的内容
```

空白标识符用于导入数据库驱动包，该包会加载驱动并允许其注册为 SQL API 的提供程序：

```
...
_ "modernc.org/sqlite"
...
```

导入该包仅用于初始化，不会直接使用，尽管你可能会遇到需要一些初始配置的驱动。数据库通过 `database/sql` 包来使用，如清单 26-9 中定义的函数所示。`listDrivers` 函数会输出可用的驱动，尽管在此示例项目中只有一个。`openDatabase` 函数使用表 26-3 中描述的 `Open` 函数来打开数据库：

```
...
db, err = sql.Open("sqlite", "products.db")
...
```

`Open` 函数的参数是使用的驱动名称和数据库的连接字符串，该字符串特定于所使用的数据库引擎。SQLite 使用数据库文件名来打开数据库。

`Open` 函数返回一个指向 `sql.DB` 结构体的指针和一个报告任何打开数据库问题的 `error`。`DB` 结构体提供对数据库的访问，而不暴露数据库引擎或其连接的细节。

我将在后续部分描述 `DB` 结构体提供的功能。但首先，我将只使用一个方法，如清单 26-10 所示。

```
package main
func main() {
listDrivers()
db, err := openDatabase()
if (err == nil) {
db.Close()
} else {
panic(err)
}
}
清单 26-10 在 data 文件夹的 main.go 文件中使用 DB 结构体
```

`main` 方法调用 `listDrivers` 函数以打印已加载驱动的名称，然后调用 `openDatabase` 函数来打开数据库。尚未对数据库执行任何操作，但调用了 `Close` 方法。该方法（如表 26-4 所述）会关闭数据库并阻止后续操作。

表 26-4 用于关闭数据库的 DB 方法

| 名称 | 描述 |
| --- | --- |
| `Close()` | 此函数关闭数据库并阻止后续操作。 |

虽然调用 `Close` 方法是个好习惯，但只有在完全完成对数据库的操作时才需要这样做。单个 `DB` 可用于对同一数据库重复执行查询，并且与数据库的连接会在后台自动管理。无需为每个查询调用 `Open` 方法来获取新的 `DB`，然后在查询完成后使用 `Close` 将其关闭。

编译并执行该项目，你将看到以下输出，其中显示了数据库驱动的名称并确认数据库已打开：

```
Driver: sqlite
Opened database
```

### 执行语句和查询

`DB` 结构体用于执行 SQL 语句，使用表 26-5 中描述的方法，这些方法将在后续部分演示。

表 26-5 用于执行 SQL 语句的 DB 方法

| 名称 | 描述 |
| --- | --- |
| `Query(query, ...args)` | 此方法使用可选的占位符参数执行指定查询。结果是一个包含查询结果的 `Rows` 结构体，以及一个指示执行查询时出现的问题的 `error`。 |
| `QueryRow(query, ...args)` | 此方法使用可选的占位符参数执行指定查询。结果是一个 `Row` 结构体，代表查询结果中的第一行。参见“为单行执行查询”部分。 |
| `Exec(query, ...args)` | 此方法执行不返回数据行的语句或查询。该方法返回一个描述数据库响应的 `Result`，以及一个指示执行问题的 `error`。参见“执行其他查询”部分。 |

在数据库中使用上下文

在第 30 章中，我描述了 `context` 包及其定义的 `Context` 接口，该接口用于在服务器处理请求时管理请求。`database/sql` 包中定义的所有重要方法也都有接受 `Context` 参数的版本，这对于利用诸如请求处理超时等功能非常有用。本章未列出这些方法，但我在第 3 部分中广泛使用了 `Context` 接口——包括使用接受它们作为参数的 `database/sql` 方法——在那一部分中，我使用 Go 及其标准库创建了一个 Web 应用平台和一个在线商店。



#### 查询多行数据

`Query` 方法会执行一个从数据库中检索一行或多行数据的查询。`Query` 方法返回一个 `Rows` 结构体，其中包含查询结果以及一个用于指示问题的 `error`。行数据通过表 26-6 中描述的方法进行访问。

**表 26-6** `Rows` 结构体的方法

| 名称 | 描述 |
| --- | --- |
| `Next()` | 此方法会前进到下一个结果行。结果是一个 `bool` 值，当存在可读取的数据时为 `true`，当到达数据末尾时为 `false`，此时会自动调用 `Close` 方法。 |
| `NextResultSet()` | 当同一个数据库响应中包含多个结果集时，此方法会前进到下一个结果集。如果还有另一组行需要处理，则该方法返回 `true`。 |
| `Scan(...targets)` | 此方法将当前行中的 SQL 值赋值给指定的变量。通过指针进行赋值，如果无法扫描这些值，该方法会返回一个 `error`。详细信息请参见“理解 Scan 方法”一节。 |
| `Close()` | 此方法阻止对结果的进一步枚举，当不需要全部数据时使用。如果使用 `Next` 方法前进直到其返回 `false`，则无需调用此方法。 |

清单 26-11 演示了一个简单的查询，展示了如何使用 `Rows` 结构体。

```
package main
import "database/sql"
func queryDatabase(db *sql.DB) {
rows, err := db.Query("SELECT * from Products")
if (err == nil) {
for (rows.Next()) {
var id, category int
var name string
var price float64
rows.Scan(&id, &name, &category, &price)
Printfln("Row: %v %v %v %v", id, name, category, price)
}
} else {
Printfln("Error: %v", err)
}
}
func main() {
//listDrivers()
db, err := openDatabase()
if (err == nil) {
queryDatabase(db)
db.Close()
} else {
panic(err)
}
}
清单 26-11
在 data 文件夹的 main.go 文件中查询数据库
```

`queryDatabase` 函数使用 `Query` 方法对 `Products` 表执行一个简单的 `SELECT` 查询，该查询会生成一个 `Rows` 结果和一个 `error`。如果 `error` 为 `nil`，则使用 `for` 循环通过调用 `Next` 方法遍历结果行，当存在待处理的行时返回 `true`，到达数据末尾时返回 `false`。

`Scan` 方法用于从结果行中提取值，并将其赋值给 Go 变量，如下所示：

```
...
rows.Scan(&id, &name, &category, &price)
...
```

变量的指针按照从数据库中读取列的顺序传递给 `Scan` 方法。必须注意确保 Go 变量能够表示将要赋值给它们的 SQL 结果。编译并执行该项目，你将看到以下输出：

```
Opened database
Row: 1 Kayak 1 279
Row: 2 Lifejacket 1 48.95
Row: 3 Soccer Ball 2 19.5
Row: 4 Corner Flags 2 34.95
```

#### 理解 Scan 方法

`Scan` 方法对其接收的参数数量、顺序和类型很敏感。如果参数数量与结果中的列数不匹配，或者参数无法存储结果值，则会返回一个错误，如清单 26-12 所示。

```
...
func queryDatabase(db *sql.DB) {
rows, err := db.Query("SELECT * from Products")
if (err == nil) {
for (rows.Next()) {
var id, category int
var name int
var price float64
scanErr := rows.Scan(&id, &name, &category, &price)
if (scanErr == nil) {
Printfln("Row: %v %v %v %v", id, name, category, price)
} else {
Printfln("Scan error: %v", scanErr)
break
}
}
} else {
Printfln("Error: %v", err)
}
}
...
清单 26-12
在 data 文件夹的 main.go 文件中不匹配的 Scan
```

清单 26-12 中对 `Scan` 方法的调用为一个在数据库中存储为 SQL `TEXT` 类型的值提供了一个 `int` 类型变量。编译并执行该项目，你会看到 `Scan` 方法返回了一个错误：

```
Scan error: sql: Scan error on column index 1, name "Name": converting driver.Value type string ("Kayak") to a int: invalid syntax
```

`Scan` 方法不会仅仅跳过导致问题的列，如果有问题，将不会扫描任何值。

#### 理解 SQL 值如何被扫描

`Scan` 方法最常见的问题是 SQL 数据类型与要扫描到的 Go 变量之间不匹配。`Scan` 方法在将 SQL 值映射到 Go 值时确实提供了一些灵活性。以下是最重要规则的简要总结：

- SQL 字符串、数值和布尔值可以映射到它们对应的 Go 类型，但需注意数值类型以防止溢出。
- SQL 数值和布尔类型可以扫描到 Go 字符串中。
- SQL 字符串可以扫描到 Go 数值类型，但前提是该字符串可以使用常规 Go 特性（在第 5 章中描述）进行解析，并且没有溢出。
- SQL 时间值可以扫描到 Go 字符串或 `*time.Time` 值。
- 任何 SQL 值都可以扫描到空接口的指针（`*interface{}`），这允许你将值转换为其他类型。

这些是最有用的映射，但有关完整详细信息，请参阅 Go 文档中关于 `Scan` 方法的部分。通常，我倾向于保守地选择类型，并且我经常将值扫描到 Go 字符串中，然后自己解析该值以管理转换过程。清单 26-13 将所有结果值都扫描到了字符串中。

```
package main
import "database/sql"
func queryDatabase(db *sql.DB) {
rows, err := db.Query("SELECT * from Products")
if (err == nil) {
for (rows.Next()) {
var id, category string
var name string
var price string
scanErr := rows.Scan(&id, &name, &category, &price)
if (scanErr == nil) {
Printfln("Row: %v %v %v %v", id, name, category, price)
} else {
Printfln("Scan error: %v", scanErr)
break
}
}
} else {
Printfln("Error: %v", err)
}
}
func main() {
//listDrivers()
db, err := openDatabase()
if (err == nil) {
queryDatabase(db)
db.Close()
} else {
panic(err)
}
}
清单 26-13
在 data 文件夹的 main.go 文件中扫描到字符串
```

这种方法确保你能将 SQL 结果获取到 Go 应用程序中，尽管它确实需要额外的工作来解析这些值以便使用它们。编译并执行该项目，你将看到以下输出：

```
Row: 1 Kayak 1 279
Row: 2 Lifejacket 1 48.95
Row: 3 Soccer Ball 2 19.5
Row: 4 Corner Flags 2 34.95
```



##### 将值扫描至结构体

`Scan`方法仅作用于单个字段，这意味着它不支持自动填充结构体的字段。相反，你必须提供指向各个字段的指针，这些字段的结果中包含值，如清单 26-14 所示。

> **注意**  
> 在本章末尾，我演示了如何使用 Go 的`reflect`包动态地将行扫描到结构体中。详情请参阅“使用反射将数据扫描到结构体”部分。

```
package main
import "database/sql"
type Product struct {
Id int
Name string
Category int
Price float64
}
func queryDatabase(db *sql.DB) []Product {
products := []Product {}
rows, err := db.Query("SELECT * from Products")
if (err == nil) {
for (rows.Next()) {
p := Product{}
scanErr := rows.Scan(&p.Id, &p.Name, &p.Category, &p.Price)
if (scanErr == nil) {
products = append(products, p)
} else {
Printfln("Scan error: %v", scanErr)
break
}
}
} else {
Printfln("Error: %v", err)
}
return products
}
func main() {
db, err := openDatabase()
if (err == nil) {
products := queryDatabase(db)
for i, p := range products {
Printfln("#%v: %v", i, p)
}
db.Close()
} else {
panic(err)
}
}
清单 26-14
在 data 文件夹下的 main.go 文件中将结果扫描至结构体
```

此示例扫描了相同的结果数据，但创建了一个`Product`切片。编译并执行该项目，你将看到以下输出：

```
Opened database
#0: {1 Kayak 1 279}
#1: {2 Lifejacket 1 48.95}
#2: {3 Soccer Ball 2 19.5}
#3: {4 Corner Flags 2 34.95}
```

这种方法可能很冗长——如果你有大量结果类型需要解析，还会产生重复——但它的优点是简单且可预测，并且可以轻松适应结果的复杂性。例如，清单 26-15 更改了发送到数据库的查询，使其包含`Categories`表中的数据。

```
package main
import "database/sql"
type Category struct {
Id int
Name string
}
type Product struct {
Id int
Name string
Category
Price float64
}
func queryDatabase(db *sql.DB) []Product {
products := []Product {}
rows, err := db.Query(`
SELECT Products.Id, Products.Name, Products.Price,
Categories.Id as Cat_Id, Categories.Name as CatName
FROM Products, Categories
WHERE Products.Category = Categories.Id`)
if (err == nil) {
for (rows.Next()) {
p := Product{}
scanErr := rows.Scan(&p.Id, &p.Name, &p.Price,
&p.Category.Id, &p.Category.Name)
if (scanErr == nil) {
products = append(products, p)
} else {
Printfln("Scan error: %v", scanErr)
break
}
}
} else {
Printfln("Error: %v", err)
}
return products
}
func main() {
db, err := openDatabase()
if (err == nil) {
products := queryDatabase(db)
for i, p := range products {
Printfln("#%v: %v", i, p)
}
db.Close()
} else {
panic(err)
}
}
清单 26-15
在 data 文件夹下的 main.go 文件中扫描更复杂的结果
```

此示例中 SQL 查询的结果包含来自`Categories`表的数据，这些数据被扫描到一个嵌套的结构体字段中。编译并执行该项目，你将看到以下输出，其中包含来自两个表的数据：

```
Opened database
#0: {1 Kayak {1 Watersports} 279}
#1: {2 Lifejacket {1 Watersports} 48.95}
#2: {3 Soccer Ball {2 Soccer} 19.5}
#3: {4 Corner Flags {2 Soccer} 34.95}
```

### 使用占位符执行语句

`Query`方法中的可选参数是查询字符串中占位符的值，这使得单个字符串可以用于不同的查询，如清单 26-16 所示。

```
package main
import "database/sql"
type Category struct {
Id int
Name string
}
type Product struct {
Id int
Name string
Category
Price float64
}
func queryDatabase(db *sql.DB, categoryName string) []Product {
products := []Product {}
rows, err := db.Query(`
SELECT Products.Id, Products.Name, Products.Price,
Categories.Id as Cat_Id, Categories.Name as CatName
FROM Products, Categories
WHERE Products.Category = Categories.Id
AND Categories.Name = ?`, categoryName)
if (err == nil) {
for (rows.Next()) {
p := Product{}
scanErr := rows.Scan(&p.Id, &p.Name, &p.Price,
&p.Category.Id, &p.Category.Name)
if (scanErr == nil) {
products = append(products, p)
} else {
Printfln("Scan error: %v", scanErr)
break
}
}
} else {
Printfln("Error: %v", err)
}
return products
}
func main() {
db, err := openDatabase()
if (err == nil) {
for _, cat := range []string { "Soccer", "Watersports"} {
Printfln("--- %v Results ---", cat)
products := queryDatabase(db, cat)
for i, p := range products {
Printfln("#%v: %v %v %v", i, p.Name, p.Category.Name, p.Price)
}
}
db.Close()
} else {
panic(err)
}
}
清单 26-16
在 data 文件夹下的 main.go 文件中使用查询占位符
```

此示例中的 SQL 查询字符串包含一个问号（`?`字符），表示一个占位符。这避免了为每个查询构建字符串的需要，并确保值被正确转义。编译并执行该项目，你将看到以下输出，展示了`queryDatabase`函数如何使用不同的占位符值调用`Query`方法：

```
Opened database
--- Soccer Results ---
#0: Soccer Ball Soccer 19.5
#1: Corner Flags Soccer 34.95
--- Watersports Results ---
#0: Kayak Watersports 279
#1: Lifejacket Watersports 48.95
```

### 执行单行查询

`QueryRow`方法执行一个预期返回单行的查询，这避免了枚举结果的需要，如清单 26-17 所示。

```
package main
import "database/sql"
type Category struct {
Id int
Name string
}
type Product struct {
Id int
Name string
Category
Price float64
}
func queryDatabase(db *sql.DB, id int) (p Product) {
row := db.QueryRow(`
SELECT Products.Id, Products.Name, Products.Price,
Categories.Id as Cat_Id, Categories.Name as CatName
FROM Products, Categories
WHERE Products.Category = Categories.Id
AND Products.Id = ?`, id)
if (row.Err() == nil) {
scanErr := row.Scan(&p.Id, &p.Name, &p.Price,
&p.Category.Id, &p.Category.Name)
if (scanErr != nil) {
Printfln("Scan error: %v", scanErr)
}
} else {
Printfln("Row error: %v", row.Err().Error())
}
return
}
func main() {
db, err := openDatabase()
if (err == nil) {
for _, id := range []int { 1, 3, 10 } {
p := queryDatabase(db, id)
Printfln("Product: %v", p)
}
db.Close()
} else {
panic(err)
}
}
清单 26-17
在 data 文件夹下的 main.go 文件中查询单行
```

`QueryRow`方法返回一个`Row`结构体，它代表单行结果，并定义了表 26-7 中描述的方法。

**表 26-7** `Row`结构体定义的方法

| 名称 | 描述 |
| --- | --- |
| `Scan(...targets)` | 此方法将当前行中的 SQL 值分配给指定的变量。值通过指针分配，并返回一个指示无法扫描值或结果中没有行的`error`。如果响应中有多行，除第一行外，所有其他行将被丢弃。 |
| `Err()` | 此方法返回一个错误，指示执行查询时出现的问题。 |

`Row`是`QueryRow`方法的唯一结果，其`Err`方法返回执行查询时的错误。`Scan`方法只会扫描结果的第一行，如果结果中没有行，则返回一个`error`。编译并执行该项目，你将看到以下结果，其中包括结果中没有行时由`Scan`方法产生的错误：

```
Opened database
Product: {1 Kayak {1 Watersports} 279}
Product: {3 Soccer Ball {2 Soccer} 19.5}
Scan error: sql: no rows in result set
Product: {0  {0 } 0}
```



### 执行其他查询

`Exec` 方法用于执行不生成行的语句。`Exec` 方法的结果是一个 `Result` 值，其中定义了表 26-8 中描述的方法，以及一个指示执行语句时出现问题的 `error`。

**表 26-8** Result 方法

| 名称 | 描述 |
| --- | --- |
| `RowsAffected()` | 此方法返回受语句影响的行数，以 `int64` 表示。此方法还会返回一个 `error`，当解析响应时出现问题或数据库不支持此功能时使用。 |
| `LastInsertId()` | 此方法返回一个 `int64`，表示执行语句时数据库生成的值，通常是自动生成的键。此方法还会返回一个 `error`，当数据库返回的值无法解析为 Go `int` 时使用。 |

清单 26-18 演示了如何使用 `Exec` 方法向 `Products` 表中插入新行。

```
package main
import "database/sql"
type Category struct {
Id int
Name string
}
type Product struct {
Id int
Name string
Category
Price float64
}
func queryDatabase(db *sql.DB, id int) (p Product) {
row := db.QueryRow(`
SELECT Products.Id, Products.Name, Products.Price,
Categories.Id as Cat_Id, Categories.Name as CatName
FROM Products, Categories
WHERE Products.Category = Categories.Id
AND Products.Id = ?`, id)
if (row.Err() == nil) {
scanErr := row.Scan(&p.Id, &p.Name, &p.Price,
&p.Category.Id, &p.Category.Name)
if (scanErr != nil) {
Printfln("Scan error: %v", scanErr)
}
} else {
Printfln("Row error: %v", row.Err().Error())
}
return
}
func insertRow(db *sql.DB, p *Product) (id int64) {
res, err := db.Exec(`
INSERT INTO Products (Name, Category, Price)
VALUES (?, ?, ?)`, p.Name, p.Category.Id, p.Price)
if (err == nil) {
id, err = res.LastInsertId()
if (err != nil) {
Printfln("Result error: %v", err.Error())
}
} else {
Printfln("Exec error: %v", err.Error())
}
return
}
func main() {
db, err := openDatabase()
if (err == nil) {
newProduct := Product { Name: "Stadium", Category:
Category{ Id: 2}, Price: 79500 }
newID := insertRow(db, &newProduct)
p := queryDatabase(db, int(newID))
Printfln("New Product: %v", p)
db.Close()
} else {
panic(err)
}
}
清单 26-18
在 data 文件夹的 main.go 文件中插入一行
```

`Exec` 方法支持占位符，清单 26-18 中的语句使用 `Product` 结构体的字段向 `Products` 表插入新行。调用 `Result.LastInsertId` 方法获取数据库分配给新行的键值，然后使用该键值查询新添加的行。编译并执行项目，你将看到以下输出：

```
Opened database
New Product: {5 Stadium {2 Soccer} 79500}
```

如果重复执行项目，你将看到不同的结果，因为每个新行都会被分配一个新的主键值。

## 使用预编译语句

`DB` 结构体提供了创建预编译语句的支持，随后可以使用这些预编译语句来执行预编译的 SQL。表 26-9 描述了用于创建预编译语句的 `DB` 方法。预编译语句由 `Stmt` 结构体表示，该结构体定义了表 26-10 中描述的方法。

**表 26-9** 用于创建预编译语句的 DB 方法

| 名称 | 描述 |
| --- | --- |
| `Prepare(query)` | 此方法为指定的查询创建一个预编译语句。结果是 `Stmt` 结构体和一个指示准备语句时出现问题的 `error`。 |

> **注意**：`database/sql` 包的一个奇特之处在于，表 26-5 中描述的许多方法也会创建预编译语句，但这些语句在单次查询后就被丢弃。

**表 26-10** Stmt 结构体定义的方法

| 名称 | 描述 |
| --- | --- |
| `Query(...vals)` | 此方法执行预编译语句，附带可选的占位符值。结果是 `Rows` 结构体和一个 `error`。此方法等同于 `DB.Query` 方法。 |
| `QueryRow(...vals)` | 此方法执行预编译语句，附带可选的占位符值。结果是 `Row` 结构体和一个 `error`。此方法等同于 `DB.QueryRow` 方法。 |
| `Exec(...vals)` | 此方法执行预编译语句，附带可选的占位符值。结果是 `Result` 和一个 `error`。此方法等同于 `DB.Exec` 方法。 |
| `Close()` | 此方法关闭语句。关闭后无法再执行语句。 |

清单 26-19 演示了如何创建预编译语句。

```
package main
import (
"database/sql"
_ "modernc.org/sqlite"
)
func listDrivers() {
for _, driver := range sql.Drivers() {
Printfln("Driver: %v", driver)
}
}
var insertNewCategory *sql.Stmt
var changeProductCategory *sql.Stmt
func openDatabase() (db *sql.DB, err error) {
db, err = sql.Open("sqlite", "products.db")
if (err == nil) {
Printfln("Opened database")
insertNewCategory, _ = db.Prepare("INSERT INTO Categories (Name) VALUES (?)")
changeProductCategory, _ =
db.Prepare("UPDATE Products SET Category = ? WHERE Id = ?")
}
return
}
清单 26-19
在 data 文件夹的 database.go 文件中使用预编译语句
```

预编译语句在数据库打开后创建，并且仅在调用 `DB.Close` 方法之前有效。清单 26-20 使用预编译语句向数据库添加一个新类别，并将一个产品分配给它。

```
package main
import "database/sql"
type Category struct {
Id int
Name string
}
type Product struct {
Id int
Name string
Category
Price float64
}
func queryDatabase(db *sql.DB, id int) (p Product) {
row := db.QueryRow(`
SELECT Products.Id, Products.Name, Products.Price,
Categories.Id as Cat_Id, Categories.Name as CatName
FROM Products, Categories
WHERE Products.Category = Categories.Id
AND Products.Id = ?`, id)
if (row.Err() == nil) {
scanErr := row.Scan(&p.Id, &p.Name, &p.Price,
&p.Category.Id, &p.Category.Name)
if (scanErr != nil) {
Printfln("Scan error: %v", scanErr)
}
} else {
Printfln("Row error: %v", row.Err().Error())
}
return
}
func insertAndUseCategory(name string, productIDs ...int) {
result, err := insertNewCategory.Exec(name)
if (err == nil) {
newID, _ := result.LastInsertId()
for _, id := range productIDs {
changeProductCategory.Exec(int(newID), id)
}
} else {
Printfln("Prepared statement error: %v", err)
}
}
func main() {
db, err := openDatabase()
if (err == nil) {
insertAndUseCategory("Misc Products", 2)
p := queryDatabase(db, 2)
Printfln("Product: %v", p)
db.Close()
} else {
panic(err)
}
}
清单 26-20
在 data 文件夹的 main.go 文件中使用预编译语句
```

`insertAndUseCategory` 函数使用了预编译语句。编译并执行项目，你将看到以下输出，反映了 `Misc Products` 类别的添加：

```
Opened database
Product: {2 Lifejacket {3 Misc Products} 48.95}
```



### 使用事务

事务允许执行多条语句，使得它们要么全部应用到数据库，要么全部不应用。`DB` 结构体定义了表 26-11 中描述的用于创建新事务的方法。

**表 26-11** 用于创建事务的 `DB` 方法

| 名称 | 描述 |
| --- | --- |
| `Begin()` | 此方法开始一个新事务。返回结果是一个指向 `Tx` 值的指针，以及一个指示创建事务时出现问题的 `error`。 |

事务由 `Tx` 结构体表示，该结构体定义了表 26-12 中描述的方法。

**表 26-12** `Tx` 结构体定义的方法

| 名称 | 描述 |
| --- | --- |
| `Query(query, ...args)` | 此方法等同于表 26-5 中描述的 `DB.Query` 方法，但查询是在事务范围内执行的。 |
| `QueryRow(query, ...args)` | 此方法等同于表 26-5 中描述的 `DB.QueryRow` 方法，但查询是在事务范围内执行的。 |
| `Exec(query, ...args)` | 此方法等同于表 26-5 中描述的 `DB.Exec` 方法，但查询/语句是在事务范围内执行的。 |
| `Prepare(query)` | 此方法等同于表 26-9 中描述的 `DB.Query` 方法，但它创建的预编译语句是在事务范围内执行的。 |
| `Stmt(statement)` | 此方法接受一个在事务范围外创建的预编译语句，并返回一个在事务范围内执行的语句。 |
| `Commit()` | 此方法将待定的更改提交到数据库，返回一个指示应用更改时出现问题的 `error`。 |
| `Rollback()` | 此方法中止事务，以便丢弃待定的更改。此方法返回一个指示中止事务时出现问题的 `error`。 |

上一节定义的 `insertAndUseCategory` 函数是一个适合使用事务的好例子（尽管很简单），因为它涉及两个关联操作。清单 26-21 引入了一个事务，如果没有产品匹配指定的 ID，该事务将被回滚。

```
package main
import "database/sql"
// ...为简洁起见省略了语句...
func insertAndUseCategory(db *sql.DB, name string, productIDs ...int) (err error) {
tx, err := db.Begin()
updatedFailed := false
if (err == nil) {
catResult, err := tx.Stmt(insertNewCategory).Exec(name)
if (err == nil) {
newID, _ := catResult.LastInsertId()
preparedStatement := tx.Stmt(changeProductCategory)
for _, id := range productIDs {
changeResult, err := preparedStatement.Exec(newID, id)
if (err == nil) {
changes, _ := changeResult.RowsAffected()
if (changes == 0) {
updatedFailed = true
break
}
}
}
}
}
if (err != nil || updatedFailed) {
Printfln("Aborting transaction %v", err)
tx.Rollback()
} else {
tx.Commit()
}
return
}
func main() {
db, err := openDatabase()
if (err == nil) {
insertAndUseCategory(db, "Category_1", 2)
p := queryDatabase(db, 2)
Printfln("Product: %v", p)
insertAndUseCategory(db, "Category_2", 100)
db.Close()
} else {
panic(err)
}
}
```

*清单 26-21：在 `data` 文件夹的 `main.go` 文件中使用事务*

第一次调用 `insertAndUseCategory` 会成功，更改将应用到数据库。第二次调用 `insertAndUseCategory` 会失败，这意味着事务被终止，并且第一个语句创建的类别不会应用到数据库。编译并执行项目，您将看到以下输出：

```
Opened database
Product: {2 Lifejacket {4 Category_1} 48.95}
Aborting transaction 
```

您可能会看到略有不同的结果，特别是如果您再次运行此示例，因为新创建的类别数据库行将被分配一个包含在输出中的唯一 ID。


好的，作为高级文档工程师和翻译员，我将严格按照您的注意事项和示例，将给定的英文文本翻译成中文。


### 使用反射将数据扫描到结构体中

反射是一种允许在运行时检查和操作类型和值的特性。反射是一个高级且复杂的功能，将在第 27 章到第 29 章中详细描述。我不会在本章中描述反射，但 `Rows` 结构体定义了一些方法，这些方法在使用反射处理数据库响应时非常有用，如下表 26-13 所述。你可能希望在阅读完反射章节后再回到这个示例。

**表 26-13** 与反射一起使用的 `Rows` 方法

| 名称 | 描述 |
| --- | --- |
| `Columns()` | 这个方法返回一个包含结果列名称的字符串切片和一个 `error`，当结果集已关闭时使用。 |
| `ColumnTypes()` | 这个方法返回一个 `*ColumnType` 切片，描述了结果列的数据类型。此方法还会返回一个 `error`，当结果集已关闭时使用。 |

## 理解反射的缺点

反射是一个高级特性，从我用了三章来描述如何使用它这一点就可以看出。这个示例只是为了展示使用 `database/sql` 包提供的信息可以实现什么。为简单起见，此示例对结果行的结构有固定的预期。

如代码清单 26-14 所示，指定单个字段是扫描到结构体中最简单、最健壮的方法。如果你决定动态地扫描到结构体，那么可以考虑一些经过充分测试的第三方包，例如 SQLX ([`https://github.com/jmoiron/sqlx`](https://github.com/jmoiron/sqlx))。

这些方法描述了从数据库返回的行的结构。`Columns` 方法返回一个包含结果列名称的字符串切片。`ColumnTypes` 方法返回一个指向 `ColumnType` 结构体的指针切片，该结构体定义了表 26-14 中描述的方法。

**表 26-14** `ColumnType` 方法

| 名称 | 描述 |
| --- | --- |
| `Name()` | 此方法返回结果中指定的列名，表示为 `string` 类型。 |
| `DatabaseTypeName()` | 此方法返回数据库中列类型的名称，表示为 `string` 类型。 |
| `Nullable()` | 此方法返回两个 `bool` 结果。如果数据库类型可以为 `null`，则第一个结果为 `true`。如果驱动程序支持可空值，则第二个结果为 `true`。 |
| `DecimalSize()` | 此方法返回关于十进制值大小的详细信息。结果是一个指定精度的 `int64`，一个指定刻度的 `int64`，以及一个 `bool`，对于十进制类型为 `true`，对于其他类型为 `false`。 |
| `Length()` | 此方法返回可变长度数据库类型的长度。结果是一个指定长度的 `int64` 和一个 `bool`，对于定义了长度的类型为 `true`，对于其他类型为 `false`。 |
| `ScanType()` | 此方法返回一个 `reflect.Type`，指示使用 `Rows.Scan` 方法扫描此列时将使用的 Go 类型。有关使用 `reflect` 包的详细信息，请参阅第 27 章到第 29 章。 |

代码清单 26-22 使用 `Columns` 方法将结果数据中的列名与结构体字段匹配，并使用 `ColumnType.ScanType` 方法确保结果类型可以安全地赋值给匹配的结构体字段。

**警告**

如前所述，此示例依赖于后续章节中描述的特性。你应该阅读第 27 章到第 29 章，并在理解了 Go 反射的工作原理后再回到这个示例。

```go
package main
import (
"database/sql"
_ "modernc.org/sqlite"
"reflect"
"strings"
)
func listDrivers() {
for _, driver := range sql.Drivers() {
Printfln("Driver: %v", driver)
}
}
var insertNewCategory *sql.Stmt
var changeProductCategory *sql.Stmt
func openDatabase() (db *sql.DB, err error) {
db, err = sql.Open("sqlite", "products.db")
if (err == nil) {
Printfln("Opened database")
insertNewCategory, _ = db.Prepare("INSERT INTO Categories (Name) VALUES (?)")
changeProductCategory, _ =
db.Prepare("UPDATE Products SET Category = ? WHERE Id = ?")
}
return
}
func scanIntoStruct(rows *sql.Rows, target interface{}) (results interface{},
err error) {
targetVal := reflect.ValueOf(target)
if (targetVal.Kind() == reflect.Ptr) {
targetVal = targetVal.Elem()
}
if (targetVal.Kind() != reflect.Struct) {
return
}
colNames, _ := rows.Columns()
colTypes, _ := rows.ColumnTypes()
references := []interface{} {}
fieldVal := reflect.Value{}
var placeholder interface{}
for i, colName := range colNames {
colNameParts := strings.Split(colName, ".")
fieldVal = targetVal.FieldByName(colNameParts[0])
if (fieldVal.IsValid() && fieldVal.Kind() == reflect.Struct &&
len(colNameParts) > 1 ) {
var namePart string
for _, namePart = range colNameParts[1:] {
compFunction := matchColName(namePart)
fieldVal = fieldVal.FieldByNameFunc(compFunction)
}
}
if (!fieldVal.IsValid() ||
!colTypes[i].ScanType().ConvertibleTo(fieldVal.Type())) {
references = append(references, &placeholder)
} else if (fieldVal.Kind() != reflect.Ptr && fieldVal.CanAddr()) {
fieldVal = fieldVal.Addr()
references = append(references, fieldVal.Interface())
}
}
resultSlice := reflect.MakeSlice(reflect.SliceOf(targetVal.Type()), 0, 10)
for rows.Next() {
err = rows.Scan(references...)
if (err == nil) {
resultSlice = reflect.Append(resultSlice, targetVal)
} else {
break
}
}
results = resultSlice.Interface()
return
}
func matchColName(colName string) func(string) bool {
return func(fieldName string) bool {
return strings.EqualFold(colName, fieldName)
}
}
```

**代码清单 26-22** 在 `data` 目录下的 `database.go` 文件中使用反射扫描结构体

`scanIntoStruct` 函数接受一个 `Rows` 值和一个目标值，数据将被扫描到该目标值中。使用 Go 反射特性来查找结构体中具有相同名称的字段，匹配时不区分大小写。对于嵌套的结构体字段，列名必须与用点号分隔的字段名对应，以便例如 `Category.Name` 字段将从名为 `category.name` 的结果列中扫描出来。

为 `Scan` 方法创建了一个指针切片，并将扫描的结构体值添加到一个切片中，该切片用于生成方法结果。如果没有结构体字段与结果列匹配，则使用一个虚拟值，因为 `Scan` 方法期望一组完整的指针用于扫描数据。代码清单 26-23 使用这个新函数来扫描查询结果。

```go
package main
import "database/sql"
type Category struct {
Id int
Name string
}
type Product struct {
Id int
Name string
Category
Price float64
}
func queryDatabase(db *sql.DB) (products []Product, err error) {
rows, err := db.Query(`SELECT Products.Id, Products.Name, Products.Price,
Categories.Id as "Category.Id", Categories.Name as "Category.Name"
FROM Products, Categories
WHERE Products.Category = Categories.Id`)
if (err != nil) {
return
} else {
results, err := scanIntoStruct(rows, &Product{})
if err == nil {
products = (results).([]Product)
} else {
Printfln("Scanning error: %v", err)
}
}
return
}
func main() {
db, err := openDatabase()
if (err == nil) {
products, _ := queryDatabase(db)
for _, p := range products {
Printfln("Product: %v", p)
}
db.Close()
} else {
panic(err)
}
}
```

**代码清单 26-23** 在 `data` 目录下的 `main.go` 文件中扫描查询结果



查询数据库时，需要指定列名，这些列名将与`Product`和`Category`结构体定义的字段匹配。正如我在第 27 章中解释的，通过反射生成的结果需要进行类型断言来缩小其类型。

此示例的效果是，扫描是基于将结果列与结构体字段名和类型进行动态匹配来完成的。编译并执行该项目，您将看到以下输出：

```
Opened database
Product: {1 Kayak {1 Watersports} 279}
Product: {2 Lifejacket {4 Category_1} 48.95}
Product: {3 Soccer Ball {2 Soccer} 19.5}
Product: {4 Corner Flags {2 Soccer} 34.95}
Product: {5 Stadium {2 Soccer} 79500}
```

## 总结

在本章中，我描述了 Go 标准库对使用 SQL 数据库的支持，这些支持简单、设计良好且易于使用。在下一章中，我将开始描述 Go 的反射特性，这些特性允许在运行时确定和使用类型。

