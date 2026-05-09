# 8. 使用 MongoDB 进行持久化

在构建 Web 应用时，应用数据的持久化非常重要。你可以使用结构体（structs）定义 Go 应用的数据模型，并针对这些结构体进行编程以处理应用数据，但你需要为应用数据提供持久化存储。

本章将向你展示如何将应用数据持久化到 MongoDB 中，MongoDB 是一种流行的 NoSQL 数据库。

本章涵盖以下内容：

- MongoDB 简介
- `mgo` 包
- 使用 `mgo` 处理 MongoDB
- 使用 MongoDB 进行持久化

### MongoDB 简介

MongoDB 是一种流行的 NoSQL 数据库，已被广泛应用于现代 Web 应用。MongoDB 是一个开源文档数据库，提供高性能、高可用性和自动扩展。MongoDB 是一种非关系型数据库，它将数据以文档形式存储在一种称为 BSON（Binary JSON）的二进制表示中。简而言之，MongoDB 是 BSON 文档的数据存储。Go 的结构体可以轻松地序列化为 BSON 文档。

有关 MongoDB 的更多详细信息以及下载和安装说明，请查看 MongoDB 网站：[www.mongodb.org/](http://www.mongodb.org/)。

注意：NoSQL（通常被解释为“不仅仅是 SQL”）数据库提供了一种数据存储和检索机制。它为关系数据库中用于设计数据模型的表格关系提供了一种替代方法。NoSQL 数据库旨在应对现代应用开发中的挑战，例如处理海量数据、易于扩展和高性能。与关系数据库相比，NoSQL 数据库可以提供更高的性能、更好的可扩展性和更低的存储成本。NoSQL 数据库有多种类型：文档数据库、图数据库、键值存储和宽列存储。MongoDB 是一种流行的文档数据库。

一个 MongoDB 数据库包含一组集合，集合由文档组成。一个文档由一个或多个字段组成，你可以在其中将数据存储为一组键值对。在 MongoDB 中，你将文档持久化到集合中，这类似于关系数据库中的表。文档类似于行，文档中的字段类似于列。

与关系数据库不同，MongoDB 为文档的 Schema（模式）提供了更高程度的灵活性，使你能够在数据模型演变时更改文档 Schema。当你使用 Go 处理 MongoDB 时，你的 Go 结构体可以成为文档的 Schema；只要你希望改变应用数据的结构，就可以随时更改 Schema。一个集合中的文档不必具有相同的字段集或 Schema，并且集合文档中的公共字段可以保存不同类型的数据。当你在多个开发迭代中以演进的方式构建应用时，这种动态 Schema 特性非常有用。

### MongoDB 入门

安装 MongoDB 后，你可以使用 Go 的 MongoDB 驱动程序从 Go 应用开始使用它。（请记住，在运行应用之前，必须启动 MongoDB 进程。）你可以通过命令行输入 `mongod` 命令并附带相应的命令行选项来启动 MongoDB。

在本章中，你将学习如何使用第三方包 `mgo`（一个流行的 Go MongoDB 驱动程序）将数据持久化到 MongoDB 并进行处理。

#### MongoDB 的 `mgo` 驱动程序简介

`mgo`（发音为 mango）是一个用于 Go 的 MongoDB 驱动程序，支持 MongoDB 的主要功能。`mgo` 包允许你使用其遵循 Go 习惯用法的简单 API，从 Go 应用轻松地处理 MongoDB。`mgo` 是一个成熟的包，创建于 2010 年，自 2011 年起由 MongoDB Inc. 赞助。它是一个经过实战检验的库，用于大规模生产场景，例如 Facebook 的 Parse.com，这是一个使用 Go 编写的流行的移动后端即服务平台。

##### 安装 `mgo`

要安装 `mgo` 包，请在终端中运行以下命令：

```
go get gopkg.in/mgo.v2
```

要使用 `mgo`，你必须将 `gopkg.in/mgo.v2` 及其子包 `gopkg.in/mgo.v2/bson` 添加到导入列表中：

```
import (
   "gopkg.in/mgo.v2"
   "gopkg.in/mgo.v2/bson"
)
```



#### 连接到 MongoDB

要开始使用 MongoDB，你必须使用 `Dial` 函数获取一个 MongoDB `Session`（参见代码清单 8-1）。`Dial` 函数会建立与 `url` 参数所指定的 MongoDB 服务器的连接。

代码清单 8-1. 连接到 MongoDB 服务器并获取会话

```
session, err := mgo.Dial("localhost")
```

`Dial` 函数也可以用于连接服务器集群（参见代码清单 8-2）。当将 MongoDB 数据库扩展为服务器集群时，这个功能非常有用。

代码清单 8-2. 连接 MongoDB 服务器集群并获取会话

```
session, err := mgo.Dial("server1.mongolab.com,server2.mongolab.com")
```

你也可以使用 `DialWithInfo` 函数来建立与单台服务器或服务器集群的连接（参见代码清单 8-3）。与 `Dial` 函数的区别在于，你可以通过使用 `DialInfo` 类型的值向集群提供额外信息。`DialWithInfo` 会建立一个指向由 `DialInfo` 类型定义的 MongoDB 服务器集群的新 `Session`。`DialWithInfo` 函数允许你在建立服务器连接时自定义一些值。当你使用 `Dial` 函数建立连接时，默认超时值为 10 秒，因此如果 `Dial` 无法连接到服务器，它将在 10 秒后超时。当你使用 `DialWithInfo` 函数建立连接时，你可以指定 `Timeout` 属性的值。

代码清单 8-3. 使用 `DialWithInfo` 连接 MongoDB 服务器集群

```
mongoDialInfo := &mgo.DialInfo{
    Addrs:    []string{"localhost"},
    Timeout:  60 * time.Second,
    Database: "taskdb",
    Username: "shijuvar",
    Password: "password@123",
}

session, err := mgo.DialWithInfo(mongoDialInfo)
```

`mgo.Session` 对象管理着一个到 MongoDB 的连接池。一旦你获取了一个 `Session` 对象，就可以对 MongoDB 执行读写操作。MongoDB 服务器会依据多种一致性规则进行查询。`Session` 对象的 `SetMode` 会更改该会话的一致性模式。有三种可用的**一致性模式**：Eventual、Monotonic 和 Strong。

代码清单 8-4 建立了一个 `Session` 对象并设置了一致性模式。

代码清单 8-4. 建立一个会话对象并设置一致性模式

```
session, err := mgo.Dial("localhost")
if err != nil {
    panic(err)
}
defer session.Close()

// 将会话切换到单调（monotonic）行为。
session.SetMode(mgo.Monotonic, true)
```

在其生命周期结束时，通过调用 `Close` 方法来关闭 `Session` 对象非常重要。在上一个代码清单中，是通过使用 `defer` 函数来调用 `Close` 方法的。

#### 访问集合

为了对 MongoDB 执行 CRUD 操作，需要创建一个 `*mgo.Collection` 对象，该对象代表 MongoDB 的集合。你可以通过调用 `*mgo.Database` 的 `C` 方法来创建 `*mgo.Collection` 对象。`mgo.Database` 类型代表命名数据库，可以通过调用 `*mgo.Session` 的 `DB` 方法来创建它。

代码清单 8-5 访问了名为 `"categories"` 的 MongoDB 集合。

代码清单 8-5. 访问一个 MongoDB 集合

```
c := session.DB("taskdb").C("categories")
```

`DB` 方法返回一个代表名为 `taskdb` 数据库的值，该值获取了一个类型为 `Database` 的实例。类型 `Database` 的方法 `C` 返回一个代表名为 `"categories"` 的集合的值。`Collection` 对象可用于执行 CRUD 操作。

### MongoDB 的 CRUD 操作

你现在已经知道如何使用 `Dial` 和 `DialWithInfo` 函数来建立一个 MongoDB `Session` 对象。接下来让我们看看如何使用 `mgo` 包对 MongoDB 数据库执行 CRUD 操作。在 Go 中，你可以将结构体、映射和切片作为 BSON 文档持久化到 MongoDB 数据库中。当你以 Go 类型（如结构体、映射和切片）持久化数据时，`mgo` 驱动会自动将其序列化为 BSON 文档。

#### 插入文档

你可以使用 `mgo.Collection` 的 `Insert` 方法向 MongoDB 插入文档。`Insert` 方法将一个或多个文档插入到集合中。使用该方法，你可以插入结构体、映射和文档切片的值。



##### 插入结构体值

在定义 Go 应用程序的数据模型时，结构体类型应是你的首选。因此，在使用 MongoDB 时，你主要通过提供结构体类型的值来将 BSON 文档插入到 MongoDB 集合中。当使用 `Insert` 方法时，MongoDB 的 `mgo` 驱动会自动将结构体值序列化为 BSON 文档。

清单 8-6 向 MongoDB 集合中插入了三个文档。

**清单 8-6.** 将结构体值插入 MongoDB

```
package main

import (
	"fmt"
	"log"
	"gopkg.in/mgo.v2"
	"gopkg.in/mgo.v2/bson"
)

type Category struct {
	Id          bson.ObjectId `bson:"_id,omitempty"`
	Name        string
	Description string
}

func main() {
	session, err := mgo.Dial("localhost")
	if err != nil {
		panic(err)
	}
	defer session.Close()

	// 可选。将会话切换为单调行为。
	// 读取可能不是完全最新的，但它们始终能看到向前推进的变更历史，
	// 在同一会话的连续查询中读取的数据将保持一致，
	// 并且会话内所做的修改将在后续查询中被观察到（读你所写）。
	// http://godoc.org/labix.org/v2/mgo#Session.SetMode
	session.SetMode(mgo.Monotonic, true)

	//获取集合
	c := session.DB("taskdb").C("categories")

	doc := Category{
		bson.NewObjectId(),
		"Open Source",
		"Tasks for open-source projects",
	}
	//插入一个类别对象
	err = c.Insert(&doc)
	if err != nil {
		log.Fatal(err)
	}

	//插入两个类别对象
	err = c.Insert(&Category{bson.NewObjectId(), "R & D", "R & D Tasks"},
		&Category{bson.NewObjectId(), "Project", "Project Tasks"})

	var count int
	count, err = c.Count()
	if err != nil {
		log.Fatal(err)
	} else {
		fmt.Printf("%d records inserted", count)
	}
}
```

创建了一个名为 `Category` 的结构体来定义数据模型，并将结构体值持久化到 MongoDB 数据库中。你可以将 `_id` 字段的类型指定为 `bson.ObjectId`。`ObjectId` 是一个 12 字节的 BSON 类型，用于保存唯一标识的值。存储在 MongoDB 集合中的 BSON 文档需要一个唯一的 `_id` 字段作为主键。

当你插入新文档时，请提供一个包含唯一 `ObjectId` 的 `_id` 字段。如果未提供 `_id` 字段，MongoDB 将添加一个包含 `ObjectId` 的 `_id` 字段。当向 MongoDB 集合插入记录时，你可以调用 `bson.NewObjectId()` 为 `bson.ObjectId` 生成一个唯一值。请标记 `_id` 字段，以便当 `mgo` 驱动将值序列化为 BSON 文档时将其序列化为 `_id`，并且指定 `omitempty` 标签，以便在序列化为 BSON 时如果值为空则忽略该值。

`mgo.Collection` 的 `Insert` 方法用于将值持久化到 MongoDB。通过指定名称 `"categories"` 创建 `Collection` 对象，并通过调用 `Insert` 方法将值插入到 `"categories"` 集合中。`Insert` 方法将一个或多个文档插入集合。首先插入一个包含 `Category` 结构体值的文档，然后插入两个文档到集合中。`mgo` 驱动会自动将结构体值序列化为 BSON 表示形式并插入集合。

在这个示例程序中，插入了三个文档。调用 `Collection` 对象的 `Count` 方法来获取集合中的记录数，并最终打印计数。

当你第一次运行该程序时，应得到以下输出：

```
3 records inserted
```

插入 `Map` 和 `文档切片` 使用 Go 结构体是定义数据模型并将数据持久化到 MongoDB 的惯用方式。但在某些上下文中，你可能需要从映射和切片中持久化值。

清单 8-7 展示了如何持久化映射对象和文档切片（`bson.D`）的值。

**清单 8-7.** 从映射和文档切片（bson.D）插入值

```
package main

import (
	"log"
	"gopkg.in/mgo.v2"
	"gopkg.in/mgo.v2/bson"
)

func main() {
	session, err := mgo.Dial("localhost")
	if err != nil {
		panic(err)
	}
	defer session.Close()

	session.SetMode(mgo.Monotonic, true)

	//获取集合
	c := session.DB("taskdb").C("categories")

	docM := map[string]string{
		"name":        "Open Source",
		"description": "Tasks for open-source projects",
	}

	//插入一个映射对象
	err = c.Insert(docM)
	if err != nil {
		log.Fatal(err)
	}

	docD := bson.D{
		{"name", "Project"},
		{"description", "Project Tasks"},
	}

	//插入一个文档切片
	err = c.Insert(docD)
	if err != nil {
		log.Fatal(err)
	}
}
```

通过使用映射对象和文档切片（`bson.D`）插入了一个文档。`bson.D` 类型表示包含有序元素的 BSON 文档。因为 `Insert` 方法期望参数值为 `interface{}` 类型，所以可以提供结构体、映射和文档切片。


好的，作为高级文档工程师和翻译员，我将严格遵循您的注意事项和示例格式，将给定的英文文本翻译成中文。


##### 插入嵌入式文档

关系型数据库与文档型数据库的主要区别在于，后者不支持用于建模领域模型的关系型方法。如果要在关系型数据库中建立一对多关系，可以创建一个父表和一个子表，父表中的每条记录都可以与子表中的多条记录相关联。为了在这种关系模型中保证数据完整性，可以在子表中定义一个外键约束，指向父表的主键。

MongoDB 集合具有灵活的 schema，因此可以通过不同的方式定义数据模型来实现相同的目标。您可以根据应用程序的上下文，选择合适的策略来定义数据模型。为了在相关的数据之间建立关系，您可以将相关文档嵌入到主文档中，或者在两个文档之间建立引用。在建立相关数据之间的关系时，第一种策略（嵌入文档）在某些场景中表现良好；后一种策略则适用于不同的场景。

> **注意**  
> 在第 9 章中，你将学习如何使用文档间的引用来建立关系。有关数据模型概念的更多信息，请查阅 MongoDB 文档： [`docs.mongodb.org/manual/data-modeling/`](https://docs.mongodb.org/manual/data-modeling/)

清单 8-8 是一个示例，展示了一个使用嵌入式文档来描述相关数据之间关系的数据模型。在 `Category` 和 `Tasks` 数据之间的一对多关系中，`Category` 拥有多个 `Task` 实体。

**清单 8-8.** 插入包含嵌入式文档的文档

```go
package main

import (
    "log"
    "time"
    "gopkg.in/mgo.v2"
    "gopkg.in/mgo.v2/bson"
)

type Task struct {
    Description string
    Due         time.Time
}

type Category struct {
    Id          bson.ObjectId `bson:"_id,omitempty"`
    Name        string
    Description string
    Tasks       []Task
}

func main() {
    session, err := mgo.Dial("localhost")
    if err != nil {
        panic(err)
    }
    defer session.Close()
    session.SetMode(mgo.Monotonic, true)

    //获取集合
    c := session.DB("taskdb").C("categories")

    //嵌入子集合
    doc := Category{
        bson.NewObjectId(),
        "Open-Source",
        "Tasks for open-source projects",
        []Task{
            Task{"Create project in mgo", time.Date(2015, time.August, 10, 0, 0, 0, 0, time.UTC)},
            Task{"Create REST API", time.Date(2015, time.August, 20, 0, 0, 0, 0, time.UTC)},
        },
    }

    //插入一个嵌入了 Tasks 的 Category 对象
    err = c.Insert(&doc)
    if err != nil {
        log.Fatal(err)
    }
}
```

这里创建了一个 `Category` 结构体，其中为 `Tasks` 字段指定了一个 `Task` 结构体切片，用于嵌入子集合。嵌入文档使你能够通过一次查询就获取到父文档及其关联的子文档（你无需再执行另一个查询来获取子文档的详细信息）。

#### 读取文档

`Collection` 的 `Find` 方法允许你查询 MongoDB 集合。当调用 `Find` 方法时，你可以提供一个文档来过滤集合数据。`Find` 方法使用提供的文档来准备一个查询。为了提供查询集合的文档，你可以提供一个 map 或一个能够被编组为 BSON 的结构体值。你可以使用通用 map，例如 `bson.M`，来提供用于查询数据的文档。要查询集合中的所有文档，你可以使用 `nil` 作为参数，这等同于一个空文档，例如 `bson.M{}`。`Find` 方法返回 `mgo.Query` 类型的值，你可以通过 `One`、`For`、`Iter` 或 `Tail` 等方法来检索结果。

##### 检索所有记录

清单 8-9 通过为 `Find` 方法提供 `nil` 作为参数值，来检索集合中的所有文档。该查询针对清单 8-8 中定义的数据模型执行。

**清单 8-9.** 查询集合中的所有文档

```go
iter := c.Find(nil).Iter()
    result := Category{}
    for iter.Next(&result) {
        fmt.Printf("Category:%s, Description:%s\n", result.Name, result.Description)
        tasks := result.Tasks
        for _, v := range tasks {
            fmt.Printf("Task:%s Due:%v\n", v.Description, v.Due)
        }
    }

if err = iter.Close(); err != nil {
        log.Fatal(err)
    }
```

`Query` 对象的 `Iter` 方法用于遍历文档。`Iter` 执行查询并返回一个可以遍历所有结果的迭代器。当你使用嵌入式文档定义父子关系时，可以在一次查询中获取到相关的文档。

##### 排序记录

文档可以使用 `Query` 类型提供的 `Sort` 方法进行排序。`Sort` 方法准备查询，以便根据提供的字段名对返回的文档进行排序。

清单 8-10 对集合中的文档进行排序。

**清单 8-10.** 使用 `Sort` 方法对文档排序

```go
iter := c.Find(nil).Sort("name").Iter()
    result := Category{}
    for iter.Next(&result) {
        fmt.Printf("Category:%s, Description:%s\n", result.Name, result.Description)
        tasks := result.Tasks
        for _, v := range tasks {
            fmt.Printf("Task:%s Due:%v\n", v.Description, v.Due)
        }
    }

    if err = iter.Close(); err != nil {
        log.Fatal(err)
    }
```

你可以通过提供一个以 `–`（减号）为前缀的字段名，以相反顺序对值进行排序，如清单 8-11 所示。

**清单 8-11.** 反向排序文档

```go
iter := c.Find(nil).Sort("-name").Iter()
```

##### 检索单条记录

清单 8-12 通过提供一个 map 对象来执行查询，从而从集合中检索单条记录。

**清单 8-12.** 从集合中查询单条记录

```go
result := Category{}
err := c.Find(bson.M{"name": "Open-Source"}).One(&result)
if err != nil {
    log.Fatal(err)
}

fmt.Printf("Category:%s, Description:%s\n", result.Name, result.Description)
tasks := result.Tasks
for _, v := range tasks {
    fmt.Printf("Task:%s Due:%v\n", v.Description, v.Due)
}
```

这里提供了 `bson.M` map 类型来查询数据。它使用为 `Name` 字段提供的值来查询集合。`Query` 类型的 `One` 方法执行查询并将获取到的第一个文档解组到结果参数中。

`Collection` 类型提供了 `FindId` 方法，这是一个便捷辅助方法，等同于以下代码：

```go
query := c.Find(bson.M{"_id": id})
```

`FindId` 方法允许你使用文档的 `_id` 字段来查询集合。

清单 8-13 使用 `FindId` 方法查询集合。

**清单 8-13.** 使用 `FindId` 查询单条记录

```go
result = Category{}
err = c.FindId(obj_id).One(&result)
```



#### 更新文档

`Collection`类型的`Update`方法允许你更新现有文档。

以下是`Update`方法的方法签名：

```
func (c *Collection) Update(selector interface{}, update interface{}) error
```

`Update`方法从集合中查找单个文档，将其与所提供的选择器文档进行匹配，并根据所提供的更新文档对其进行修改。通过在更新文档中使用关键字`"$set"`，可以执行部分更新。

清单 8-14 更新了一个现有文档。

**清单 8-14.** 更新文档

```
err := c.Update(bson.M{"_id": id},
        bson.M{"$set": bson.M{
            "description": "创建开源项目",
            "tasks": []Task{
                Task{"评估 Negroni 项目", time.Date(2015, time.August, 15, 0, 0, 0, 0, time.UTC)},
                Task{"探索 mgo 项目", time.Date(2015, time.August, 10, 0, 0, 0, 0, time.UTC)},
                Task{"探索 Gorilla Toolkit", time.Date(2015, time.August, 10, 0, 0, 0, 0, time.UTC)},
            },
        }})
```

对`descriptions`和`tasks`字段执行了部分更新。`Update`方法查找具有给定`_id`值的文档，并根据所提供的文档修改这些字段。

#### 删除文档

`Collection`类型的`Remove`方法允许你删除单个文档。

以下是`Remove`方法的方法签名：

```
func (c *Collection) Remove(selector interface{}) error
```

`Remove`从集合中查找单个文档，将其与所提供的选择器文档进行匹配，并将其从数据库中删除。

清单 8-15 从集合中删除了单个文档。

**清单 8-15.** 删除单个文档

```
err := c.Remove(bson.M{"_id": id})
```

匹配`_id`字段的单个文档被删除。

`Collection`类型的`RemoveAll`方法允许你删除多个文档。

以下是`RemoveAll`方法的方法签名：

```
func (c *Collection) RemoveAll(selector interface{}) (info *ChangeInfo, err error)
```

`RemoveAll`从集合中查找所有文档，匹配所提供的选择器文档，并从数据库中删除所有文档。如果要删除集合中的所有文档，请将选择器文档作为`nil`值传递。

清单 8-16 删除了集合中的所有文档。

**清单 8-16.** 删除集合中的所有文档

```
c.RemoveAll(nil)
```

### MongoDB 中的索引

与关系型数据库相比，MongoDB 数据库可以在读操作上提供高性能。除了 MongoDB 的默认性能行为之外，你还可以通过向 MongoDB 集合添加索引来进一步提高性能。集合中的索引可为频繁使用的查询提供高性能的读取操作。MongoDB 在[集合](http://docs.mongodb.org/manual/reference/glossary/#term-collection)级别定义索引，并支持对 MongoDB 集合中文档的任何字段或子字段建立索引。

所有 MongoDB 集合默认都有一个针对`_id`字段的索引。如果你不提供`_id`字段，MongoDB 进程（`mongod`）会创建一个具有唯一`ObjectId`值的`_id`字段。`_id`索引是唯一的（你可以将其视为主键）。除了`_id`字段上的默认索引外，你还可以向任何字段添加索引。

如果你经常通过特定字段进行过滤来查询集合，则应应用索引以确保更好的读操作性能。`mgo`驱动通过使用`Collection`的`EnsureIndex`方法支持创建索引，在该方法中，你可以添加`mgo.Index`类型作为参数。

清单 8-17 对`name`字段应用了一个唯一索引，之后使用`Name`字段进行查询。

**清单 8-17.** 在 MongoDB 集合中创建索引

```
package main

import (
    "fmt"
    "log"
    "gopkg.in/mgo.v2"
    "gopkg.in/mgo.v2/bson"
)

type Category struct {
    Id          bson.ObjectId `bson:"_id,omitempty"`
    Name        string
    Description string
}

func main() {
    session, err := mgo.Dial("localhost")
    if err != nil {
        panic(err)
    }
    defer session.Close()
    session.SetMode(mgo.Monotonic, true)
    c := session.DB("taskdb").C("categories")
    c.RemoveAll(nil)

    // 索引
    index := mgo.Index{
        Key:        []string{"name"},
        Unique:     true,
        DropDups:   true,
        Background: true,
        Sparse:     true,
    }

    // 创建索引
    err = c.EnsureIndex(index)
    if err != nil {
        panic(err)
    }

    // 插入三个分类对象
    err = c.Insert(
        &Category{bson.NewObjectId(), "开源", "开源项目任务"},
        &Category{bson.NewObjectId(), "研发", "研发任务"},
        &Category{bson.NewObjectId(), "项目", "项目任务"},
    )
    if err != nil {
        panic(err)
    }

    result := Category{}
    err = c.Find(bson.M{"name": "开源"}).One(&result)
    if err != nil {
        log.Fatal(err)
    } else {
        fmt.Println("描述:", result.Description)
    }
}
```

创建了一个`mgo.Index`类型的实例，并通过提供一个`Index`类型实例作为参数来调用`EnsureIndex`函数：

```
index := mgo.Index{
    Key:        []string{"name"},
    Unique:     true,
    DropDups:   true,
    Background: true,
    Sparse:     true,
}
err = c.EnsureIndex(index)
if err != nil {
    panic(err)
}
```

`Index`类型的`Key`属性允许你指定要应用为集合索引的字段名切片。这里，字段`name`被指定为索引。由于字段名是以切片形式提供的，你可以在单个`Index`类型实例中提供多个字段。`Index`类型的`Unique`属性可防止两个文档具有相同的索引键。

默认情况下，索引是升序排列的。要创建降序索引，字段名应以短划线前缀（`-`）指定，如下所示：

```
Key:   []string{"-name"}
```



### 管理会话

`mgo` 包的 `Dial` 方法用于建立与 MongoDB 服务器集群的连接，并返回一个 `mgo.Session` 对象。你可以使用 `Session` 对象管理所有 CRUD 操作，该对象也管理着与 MongoDB 服务器的连接池。连接池是数据库连接的缓存，当需要与数据库建立新连接时，可以复用已有连接。在开发 Web 应用时，为所有 CRUD 操作使用单一的全局 `Session` 对象是一种非常糟糕的做法。

以下是 Web 应用中管理 `Session` 对象的推荐流程：

- 使用 `Dial` 方法获取一个 `Session` 对象。
- 在单个 HTTP 请求的生命周期内，通过对 `Dial` 方法返回的 `Session` 对象调用 `New`、`Copy` 或 `Clone` 方法来创建一个新的 `Session` 对象。这种方法能让 `Session` 对象恰当地使用连接池。
- 使用新获取的 `Session` 对象在 HTTP 请求的生命周期内执行所有 CRUD 操作。

`New` 方法会创建一个与原始 `Session` 具有相同参数的新 `Session`。`Copy` 方法的工作方式与 `New` 类似，但复制的 `Session` 会保留原始 `Session` 的精确认证信息。`Clone` 方法的工作方式与 `Copy` 类似，但它还会复用与原始 `Session` 相同的套接字（即与服务器的连接）。

清单 8-18 是一个示例 HTTP 服务器，它在 HTTP 请求的生命周期内使用复制的 `Session` 对象。在此示例中，创建了一个结构体类型来持有 `Session` 对象，以便于从应用程序处理程序管理数据库操作。

**清单 8-18.** 为每个 HTTP 请求使用新 Session 的 HTTP 服务器

```
package main

import (
	"encoding/json"
	"log"
	"net/http"
	"github.com/gorilla/mux"
	"gopkg.in/mgo.v2"
	"gopkg.in/mgo.v2/bson"
)

var session *mgo.Session

type (
	Category struct {
		Id          bson.ObjectId `bson:"_id,omitempty"`
		Name        string
		Description string
	}
	DataStore struct {
		session *mgo.Session
	}
)

//Close mgo.Session
func (d *DataStore) Close() {
	d.session.Close()
}

//Returns a collection from the database.
func (d *DataStore) C(name string) *mgo.Collection {
	return d.session.DB("taskdb").C(name)
}

//Create a new DataStore object for each HTTP request
func NewDataStore() *DataStore {
	ds := &DataStore{
		session: session.Copy(),
	}
	return ds
}

//Insert a record
func PostCategory(w http.ResponseWriter, r *http.Request) {
	var category Category
	// Decode the incoming Category json
	err := json.NewDecoder(r.Body).Decode(&category)
	if err != nil {
		panic(err)
	}
	ds := NewDataStore()
	defer ds.Close()
	//Getting the mgo.Collection
	c := ds.C("categories")
	//Insert record
	err = c.Insert(&category)
	if err != nil {
		panic(err)
	}
	w.WriteHeader(http.StatusCreated)
}

//Read all records
func GetCategories(w http.ResponseWriter, r *http.Request) {
	var categories []Category
	ds := NewDataStore()
	defer ds.Close()
	//Getting the mgo.Collection
	c := ds.C("categories")
	iter := c.Find(nil).Iter()
	result := Category{}
	for iter.Next(&result) {
		categories = append(categories, result)
	}
	w.Header().Set("Content-Type", "application/json")
	j, err := json.Marshal(categories)
	if err != nil {
		panic(err)
	}
	w.WriteHeader(http.StatusOK)
	w.Write(j)
}

func main() {
	var err error
	session, err = mgo.Dial("localhost")
	if err != nil {
		panic(err)
	}
	r := mux.NewRouter()
	r.HandleFunc("/api/categories", GetCategories).Methods("GET")
	r.HandleFunc("/api/categories", PostCategory).Methods("POST")
	server := &http.Server{
		Addr:    ":8080",
		Handler: r,
	}
	log.Println("Listening...")
	server.ListenAndServe()
}
```

定义了一个 `DataStore` 结构体类型来简化 `mgo.Session` 的管理。为 `DataStore` 类型添加了两个方法：`Close` 和 `C`。`Close` 方法通过调用 `Session` 对象的 `Close` 方法来释放新创建的 `Session` 对象的资源。该方法在 HTTP 请求生命周期结束后，通过 `defer` 函数调用。`C` 方法返回指定名称的 `Collection` 对象：

```
type DataStore struct {
	session *mgo.Session
}

//Close mgo.Session
func (d *DataStore) Close() {
	d.session.Close()
}

//Returns a collection from the database.
func (d *DataStore) C(name string) *mgo.Collection {
	return d.session.DB("taskdb").C(name)
}
```

`NewDataStore` 函数通过使用 `Dial` 方法获取的 `Session` 的 `Copy` 方法来提供一个新 `Session` 对象，进而创建一个新的 `DataStore` 对象：

```
func NewDataStore() *DataStore {
	ds := &DataStore{
		session: session.Copy(),
	}
	return ds
}
```

调用 `NewDataStore` 函数来创建一个 `DataStore` 对象，以便为 HTTP 请求的生命周期提供复制的 `Session` 对象。对于每个路由的处理程序，都通过 `DataStore` 类型使用一个新的 `Session` 对象。简而言之，使用全局 `Session` 对象并非良好实践；推荐做法是为每个 HTTP 请求的生命周期使用一个复制的 `Session` 对象。这种方法允许你在需要时拥有多个 `Session` 对象。

### 总结

在开发 Web 应用时，将应用数据持久化到持久化存储中非常重要。在本章中，你学习了如何使用 `mgo` 包（一个 Go 语言的 MongoDB 驱动程序）将数据持久化到 MongoDB 中。

MongoDB 是一个开源文档数据库，提供高性能、高可用性和自动扩展能力。MongoDB 是最流行的 NoSQL 数据库，已被现代 Web 应用广泛使用。`mgo` 驱动程序提供了一个遵循 Go 语言习惯的简单 API。

你可以为 MongoDB 集合的字段添加索引，从而为读操作提供高性能。在 Go 语言中开发 Web 应用时，使用全局 `mgo.Session` 对象并非良好实践。管理 `Session` 的正确方法是，对通过 `mgo` 包的 `Dial` 方法获取的 `Session` 对象调用 `Copy`、`Clone` 或 `New` 来创建一个新的 `Session` 对象。

下一章将进一步展示如何使用 `mgo` 包操作 MongoDB 数据库。

