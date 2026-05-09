# 构建 RESTful 服务

前几章讨论了构建基于 Web 的系统涉及的各个方面，包括使用 `http` 包的 Go Web 编程基础、用于渲染 UI 的 `html/template` 包、各种身份验证方法、HTTP 中间件以及使用 MongoDB 进行数据持久化。

本章将利用前几章讨论的概念，实现一个功能完整的 RESTful API 应用程序，名为 TaskManager。该应用程序还将包含一些新特性，例如管理外部包依赖的能力以及使用 Docker 部署 HTTP 服务器。



### RESTful API：数字化转型的基石

表述性状态转移（REST）是一种用于构建可扩展 Web 服务的架构风格。RESTful 系统通常通过使用 HTTP 动词，在超文本传输协议（HTTP）上进行通信。REST 架构风格由罗伊·菲尔丁在其博士论文中首次描述，其中他列出了应用于 REST 架构的六项约束。

以下是罗伊·菲尔丁描述的六项约束：

*   统一接口
*   无状态
*   可缓存
*   客户端-服务器
*   分层系统
*   按需代码（可选）

注意

为了更好地理解 REST 架构风格，请阅读罗伊·菲尔丁的博士论文，地址为 [`http://www.ics.uci.edu/∼fielding/pubs/dissertation/rest_arch_style.htm`](http://www.ics.uci.edu/%E2%88%BCfielding/pubs/dissertation/rest_arch_style.htm)。

REST 架构最大的优势在于它使用了 Web 编程的基本组件。如果你具备 HTTP 编程的基础知识，就能轻松地将 REST 架构风格应用于你的应用程序。它使用 HTTP 作为传输系统与远程服务器通信。你可以使用 XML 和 JSON 作为客户端与服务器之间的数据交换格式。资源是 RESTful 系统中的关键概念，正如菲尔丁在其论文中所描述的那样：

> REST 中信息的关键抽象是资源。任何可以被命名的信息都可以成为一个资源：一份文档或图像、一项时效性服务（例如“洛杉矶今天的天气”）、一个其他资源的集合、一个非虚拟对象（例如一个人），等等。换句话说，任何可能成为作者超文本引用目标的概念，都必须符合资源的定义。资源是一组实体的概念映射，而不是在任何一个特定时间点与该映射相对应的实体。

通过使用统一资源标识符（URI）和 HTTP 动词，你可以对资源执行操作。假设你定义了资源 `"/api/employees"`。现在，你可以使用 HTTP 动词 `Get` 来检索员工资源的信息，使用 `Post` 来创建新的员工资源，使用 `Put` 来更新现有员工资源，以及使用 `Delete` 来删除员工资源。

遵循 REST 架构风格的 Web 服务 API 被称为 RESTful API。当构建 RESTful API 时，你会使用 XML 和 JSON 作为客户端应用程序与 API 服务器之间交换数据的格式。一些 API 同时支持 XML 和 JSON 两种格式；其他 API 则只支持一种数据格式：要么 XML，要么 JSON。在为移动应用程序构建 API 时，最常用的数据格式是 JSON，因为它比 XML 更轻量，也更容易使用。在本章中，你将构建一个基于 JSON 的 RESTful API。

#### 使用 RESTful API 进行 API 驱动开发

API（通常是 RESTful API）通过在企业的数字化转型过程中提供支持，正逐渐成为现代应用的基石。RESTful API 能够实现应用之间更好的集成，无论其技术、平台和设备如何。它们在许多应用开发场景中都是非常重要的组件，包括大数据、物联网（IoT）、微服务架构、Web 应用和移动应用。

在移动化时代，基于 API 的开发的重要性大大提升。在构建移动应用时，RESTful API 既可以用作后端，也可以用于构建 Web 前端应用。在企业移动化中，最大的挑战并非构建移动前端应用，而是将企业数据暴露给移动设备。这些数据可能跨越多个系统、技术和平台。RESTful API 使您能够将企业数据暴露给移动设备。在大多数企业移动化场景中，RESTful 服务层是实施的重要组成部分。

过去，Web 应用的开发方式是在服务器端实现所有功能，包括使用服务器端模板的用户界面渲染逻辑。如今，API 正成为 Web 应用和移动应用的核心：开发人员在服务器端构建 RESTful 服务，然后利用这些 RESTful 服务作为后端，为 Web 和移动端构建前端应用。

### Go：构建 RESTful 服务的绝佳技术栈

我相信，对于构建包含所有功能（包括 UI 渲染）都在服务器端实现的传统 Web 应用来说，Go 可能不是一个绝佳的选择。我并非说 Go 不适合构建传统 Web 应用；你可以直接使用 Go 标准库包来构建传统 Web 应用。像 Beego 和 Revel 这样的 Go Web 开发框架也适合构建 Web 应用，但我坚信，对于构建高度可扩展的 RESTful API 系统来为现代应用提供后端支持，Go 是一个绝佳的选择。在可用于构建 RESTful API 的技术栈中，我强烈推荐 Go，它在构建可扩展的后端系统方面提供了高性能、更强的并发支持以及简洁性。Go 的简洁性和丰富的包生态系统非常适合编写松耦合、易于维护的 RESTful 服务。

标准库包 `http` 为构建可扩展的 Web 系统提供了坚实的基础。它内置了并发支持，将每个 HTTP 请求放在一个独立的 `goroutine` 中执行。`http` 包具有大量的可扩展点，能够很好地与第三方包生态系统协作。如果你想扩展或替换 `http` 包的某些功能，可以轻松构建自己的包，而不会破坏该包的设计目标。有多个有用的第三方包扩展了 `http` 包的功能，使你能够使用 Go 构建高度可扩展的 Web 应用和 RESTful API。

在构建基于 Web 的系统时，生态系统非常重要，而 Go 拥有许多成熟的数据库驱动程序包。第 8 章 讨论了如何使用 `mgo` 包与 MongoDB 协同工作。主要的**关系型数据库**和**NoSQL 数据库**都有可用的数据库驱动程序包。如果你的后端系统需要与中间件消息系统协同工作，可以找到相应的客户端库包。NSQ，一个实时分布式消息平台，就是用 Go 构建的。简而言之，Go 生态系统（包括第三方包和工具）提供了使用 Go 构建可扩展的 RESTful API 以及其他基于 Web 的后端系统所需的一切。



#### Go：微服务架构的伟大技术栈

软件开发实践不断演进，以应对新的挑战。作为一种编程语言，Go 是解决当今构建大型应用难题的优秀范例。软件架构同样在演进。微服务架构是软件架构领域最热门的概念，它是构建大型分布式应用的软件开发方式的一次根本性转变。

过去，许多应用采用单体架构模式，即应用的所有组件（包括表示层组件、业务逻辑、数据库访问逻辑和应用集成逻辑）都被打包成一个可部署单元。随着这些应用的范围和功能不断演进，它们通过添加新功能和软件组件，始终作为一个单一应用单元进行维护。由于将所有内容作为一个可部署单元进行维护的复杂性，扩展和维护单体应用始终是一个巨大的挑战。微服务架构是一种替代性的架构模式，它解决了单体架构的局限性和挑战。

微服务架构是软件开发的一种范式转变，其核心是构建可独立部署的单个软件模块。与开发大型单体应用不同，微服务架构建议构建松散耦合、小型的软件服务，这些服务可以独立于其他服务进行部署。与单体架构不同，微服务架构中的每个服务都是独立开发与部署的，因此你可以轻松地扩展开发和部署。你甚至可以使用不同的技术栈来开发单个服务，因为它们独立于其他服务进行部署和更新。

##### 微服务架构中的 RESTful 服务

在微服务架构中，开发的是松散耦合的软件服务，通过组合这些服务来构建应用。在这种架构方法中，每个微服务都需要与其他微服务通信；RESTful API 可用于这些微服务之间的通信。RESTful API 是微服务架构中非常重要的组件。

Go 是使用微服务模式构建应用的绝佳技术栈，其设计理念与微服务架构模式相得益彰。Go 编程的惯用方法是开发小型的软件组件作为包（packages），然后通过组合这些包来构建应用。

#### 构建 RESTful API

让我们利用从前几章获得的知识，构建一个简单的 RESTful API 应用。这个 RESTful API 应用为构建一个简单的“TaskManager”应用提供后端支持，该应用可用于管理任务，并提供每个任务的更新和备注。该应用的数据将持久化到 MongoDB 数据库中。这个 TaskManager RESTful API 将是一个基于 JSON 的 API，使用 JSON 数据格式在客户端应用和 RESTful API 服务器之间发送和接收消息。

##### 第三方包

构建 TaskManager 应用将使用以下第三方包：

- `gopkg.in/mgo.v2`：`mgo` 包为 Go 语言提供了一个 MongoDB 驱动。
- `gopkg.in/mgo.v2/bson`：`mgo` 的子包，为 Go 语言提供了 BSON 规范的实现。
- `github.com/gorilla/mux`：`mux` 包实现了一个请求路由器和分发器。
- `github.com/dgrijalva/jwt-go`：`jwt-go` 包实现了用于处理 JSON Web 令牌（JWT）的辅助函数。
- `github.com/codegangsta/negroni`：`Negroni` 包为 HTTP 中间件提供了一种惯用的方法。

##### 应用结构

图 9-1 展示了 RESTful API 应用的高级文件夹结构。

![A978-1-4842-1052-9_9_Fig1_HTML.jpg](img/A978-1-4842-1052-9_9_Fig1_HTML.jpg)

图 9-1. 应用文件夹结构

图 9-2 展示了 RESTful API 应用完整版本的文件夹结构及相关文件。

![A978-1-4842-1052-9_9_Fig2_HTML.jpg](img/A978-1-4842-1052-9_9_Fig2_HTML.jpg)

图 9-2. 完整应用的文件夹及相关文件

除了 `keys` 和 `Godeps` 文件夹，所有其他文件夹都代表 Go 包。`keys` 文件夹包含用于签署 JWT 并进行验证的密钥，用于通过 JWT 对 API 进行身份验证。`Godeps` 文件夹用于使用 `godep` 第三方工具管理应用的外部依赖。

RESTful API 应用被划分为以下包：

- `common`：实现一些工具函数，并提供应用的初始化逻辑
- `controllers`：实现应用的应用处理器
- `data`：实现与 MongoDB 数据库的持久化逻辑
- `models`：描述应用的数据模型
- `routers`：为 RESTful API 实现 HTTP 请求路由



##### 数据模型

该应用提供用于管理任务的 API。用户可以添加任务，并针对单个任务提供更新和备注。让我们为这个应用定义数据模型，以便与 `MongoDB` 数据库一起使用。

清单 9-1 定义了 RESTful API 应用的数据模型。

**清单 9-1.** `models.go` 中的应用数据模型

```go
package models

import (
    "time"
    "gopkg.in/mgo.v2/bson"
)

type (
    User struct {
        Id           bson.ObjectId `bson:"_id,omitempty" json:"id"`
        FirstName    string        `json:"firstname"`
        LastName     string        `json:"lastname"`
        Email        string        `json:"email"`
        Password     string        `json:"password,omitempty"`
        HashPassword []byte        `json:"hashpassword,omitempty"`
    }

    Task struct {
        Id          bson.ObjectId `bson:"_id,omitempty" json:"id"`
        CreatedBy   string        `json:"createdby"`
        Name        string        `json:"name"`
        Description string        `json:"description"`
        CreatedOn   time.Time     `json:"createdon,omitempty"`
        Due         time.Time     `json:"due,omitempty"`
        Status      string        `json:"status,omitempty"`
        Tags        []string      `json:"tags,omitempty"`
    }

    TaskNote struct {
        Id          bson.ObjectId `bson:"_id,omitempty" json:"id"`
        TaskId      bson.ObjectId `json:"taskid"`
        Description string        `json:"description"`
        CreatedOn   time.Time     `json:"createdon,omitempty"`
    }
)
```

创建了三个结构体：`User`、`Task` 和 `TaskNote`。`User` 结构体代表应用的用户。用户需要在应用上注册才能创建任务。经过身份验证的用户可以添加任务，这些任务将由 `Task` 结构体表示。用户可以针对每个任务添加备注，这些备注将由 `TaskNote` 结构体表示。`TaskNote` 实体保存其父实体 `Task` 的子细节。

第 8 章展示了如何通过将子文档嵌入到父文档中来建立父子关系。这种方法适用于某些场景，但在其他上下文中，文档引用也同样适用。在这里，父子关系是通过文档引用建立的。每当创建一个 `TaskNote` 对象时，通过在该对象中指定 `TaskId`，便会建立对父 `Task` 文档的引用。使用这种方法，你需要对 `MongoDB` 数据库执行单独的查询，才能获取 `Task` 对象和 `TaskNote` 对象的文档。当你采用嵌入文档的方式建立一对多关系时，只需执行单个查询就能获取父实体和子实体的信息，因为子文档是嵌入在父实体中的。

##### RESTful API 的资源建模

在设计与 RESTful API 时，资源建模是一个非常关键的概念；它是设计 RESTful API 的基础层。让我们利用 URI 作为资源标识符和 HTTP 动词来定义 RESTful API 的资源。

表 9-1 展示了为 RESTful API 确定的资源。

**表 9-1.** 为 RESTful API 确定的资源

| URI | HTTP 动词 | 功能 |
| --- | --- | --- |
| `/users/register` | `Post` | 创建新用户。 |
| `/users/login` | `Post` | 用户登录系统，登录成功则返回 JWT。 |
| `/tasks` | `Post` | 创建新任务。 |
| `/tasks/{id}` | `Put` | 更新现有任务。 |
| `/tasks` | `Get` | 获取所有任务。 |
| `/tasks/{id}` | `Get` | 根据给定 ID 获取单个任务。ID 值来自路由参数。 |
| `/tasks/users/{id}` | `Get` | 获取与某个用户关联的所有任务。用户 ID 值来自路由参数。 |
| `/tasks/{id}` | `Delete` | 根据给定 ID 删除现有任务。ID 值来自路由参数。 |
| `/notes` | `Post` | 针对现有任务创建新备注。 |
| `/notes/{id}` | `Put` | 更新现有的任务备注。 |
| `/notes` | `Get` | 获取所有任务备注。 |
| `/notes/{id}` | `Get` | 根据给定 ID 获取单个备注。ID 值来自路由参数。 |
| `/notes/tasks/{id}` | `Get` | 根据给定任务 ID 获取所有任务备注。ID 值来自路由参数。 |
| `/notes/{id}` | `Delete` | 根据给定 ID 删除现有备注。ID 值来自路由参数。 |

###### 将资源映射到应用路由

你需要将资源标识符映射到 HTTP 服务器的应用路由，以便为发往 RESTful API 资源的 HTTP 请求执行相应的应用处理器。所有路由都组织在 `routers` 包中，其中为每个资源路径分别编写了 Go 源文件来指定路由。在这个 RESTful API 应用中，使用了三个实体：`User`、`Task` 和 `TaskNote`。这些实体映射到三个资源：`"/users"`、`"/tasks"` 和 `"/notes"`。

图 9-3 展示了 `routers` 包目录的结构：

![A978-1-4842-1052-9_9_Fig3_HTML.jpg](img/A978-1-4842-1052-9_9_Fig3_HTML.jpg)

**图 9-3.** `routers` 包的结构

###### 用户资源的路由

让我们定义用户资源的路由（见清单 9-2）。

**清单 9-2.** `user.go` 中用户资源的路由

```go
package routers

import (
    "github.com/gorilla/mux"
    "github.com/shijuvar/go-web/taskmanager/controllers"
)

func SetUserRoutes(router *mux.Router) *mux.Router {
    router.HandleFunc("/users/register", controllers.Register).Methods("POST")
    router.HandleFunc("/users/login", controllers.Login).Methods("POST")
    return router
}
```

`SetUserRoutes` 函数接收一个指向 Gorilla `mux` 路由器对象 (`mux.Router`) 的指针作为参数，并返回 `mux.Router` 对象的指针。这里指定了两条路由：一条用于注册新用户，另一条用于用户登录系统。应用处理器函数从 `controllers` 包中调用，该包将在本章后面讨论。


###### 任务资源的路由

用户必须登录系统才能访问`Task`和`TaskNote`的资源。当用户使用用户名和密码登录时，系统会颁发一个 JWT，该 JWT 可作为访问`Task`和`TaskNote`实体资源的授权凭证。服务器通过一个 HTTP 中间件来验证`Task`和`TaskNote`资源 HTTP 请求中的 JWT，该中间件封装了应用程序处理器，并确保 HTTP 请求包含有效的 JWT 承载令牌。

第三方包`Negroni`用于处理 HTTP 中间件（参考第 6 章）。`common`包中有一个名为`Authorize`的中间件处理函数，用于通过 JWT 授权 HTTP 请求。在 RESTful API 应用中，无需在所有路由上使用授权中间件；当访问`User – Register and Login`资源时，不应调用此中间件函数。身份验证中间件函数应用于`Task`和`TaskNote`实体。在此，`Task`实体的资源映射到 URI`"/tasks"`，因此需要添加授权中间件来处理`"/tasks"`URL 路径。`Negroni`包允许您为特定 URL 路径的路由添加中间件。

清单 9-3 展示了为`Tasks`资源指定的路由。

**清单 9-3.** `task.go`中任务资源的路由

```go
package routers

import (
    "github.com/codegangsta/negroni"
    "github.com/gorilla/mux"
    "github.com/shijuvar/go-web/taskmanager/common"
    "github.com/shijuvar/go-web/taskmanager/controllers"
)

func SetTaskRoutes(router *mux.Router) *mux.Router {
    taskRouter := mux.NewRouter()
    taskRouter.HandleFunc("/tasks", controllers.CreateTask).Methods("POST")
    taskRouter.HandleFunc("/tasks/{id}", controllers.UpdateTask).Methods("PUT")
    taskRouter.HandleFunc("/tasks", controllers.GetTasks).Methods("GET")
    taskRouter.HandleFunc("/tasks/{id}", controllers.GetTaskById).Methods("GET")
    taskRouter.HandleFunc("/tasks/users/{id}", controllers.GetTasksByUser).Methods("GET")
    taskRouter.HandleFunc("/tasks/{id}", controllers.DeleteTask).Methods("DELETE")
    router.PathPrefix("/tasks").Handler(negroni.New(
        negroni.HandlerFunc(common.Authorize),
        negroni.Wrap(taskRouter),
    ))
    return router
}
```

##### 添加路由特定中间件

您可以向路由路径`"/tasks"`添加授权中间件，以限制只有经过身份验证的用户才能访问。在`SetTaskRoutes`函数中，创建了一个新的`mux router`路由器实例，指定了`"/tasks"`资源的路由，并将授权中间件包装到路由路径`"/tasks"`的处理函数中：

```go
router.PathPrefix("/tasks").Handler(negroni.New(
    negroni.HandlerFunc(common.Authorize),
    negroni.Wrap(taskRouter),
))
```

###### TaskNote 资源的路由

`TaskNote`实体映射到 URI`"/notes"`。与`"/tasks"`资源类似，需要为`"/notes"`资源添加授权中间件。

清单 9-4 展示了为`TaskNote`资源指定的路由。

**清单 9-4.** `note.go`中 TaskNote 资源的路由

```go
package routers

import (
    "github.com/codegangsta/negroni"
    "github.com/gorilla/mux"
    "github.com/shijuvar/go-web/taskmanager/common"
    "github.com/shijuvar/go-web/taskmanager/controllers"
)

func SetNoteRoutes(router *mux.Router) *mux.Router {
    noteRouter := mux.NewRouter()
    noteRouter.HandleFunc("/notes", controllers.CreateNote).Methods("POST")
    noteRouter.HandleFunc("/notes/{id}", controllers.UpdateNote).Methods("PUT")
    noteRouter.HandleFunc("/notes/{id}", controllers.GetNoteById).Methods("GET")
    noteRouter.HandleFunc("/notes", controllers.GetNotes).Methods("GET")
    noteRouter.HandleFunc("/notes/tasks/{id}", controllers.GetNotesByTask).Methods("GET")
    noteRouter.HandleFunc("/notes/{id}", controllers.DeleteNote).Methods("DELETE")
    router.PathPrefix("/notes").Handler(negroni.New(
        negroni.HandlerFunc(common.Authorize),
        negroni.Wrap(noteRouter),
    ))
    return router
}
```

###### 初始化 RESTful API 的路由

现在已为 RESTful API 应用指定了所有路由。我们来编写代码，初始化前面步骤中指定的所有路由。

清单 9-5 初始化了 RESTful API 的所有路由。

**清单 9-5.** 在`router.go`中初始化路由

```go
package routers

import (
    "github.com/gorilla/mux"
)

func InitRoutes() *mux.Router {
    router := mux.NewRouter().StrictSlash(false)
    // Routes for the User entity
    router = SetUserRoutes(router)
    // Routes for the Task entity
    router = SetTaskRoutes(router)
    // Routes for the TaskNote entity
    router = SetNoteRoutes(router)
    return router
}
```

当 HTTP 服务器启动时，会调用`main.go`中的`InitRoutes`函数，本章稍后将讨论这一点。

##### 设置 RESTful API 应用

RESTful API 应用的资源建模已完成。本节重点介绍如何设置应用，并在执行一些必要的初始化逻辑后启动 HTTP 服务器。

在启动 HTTP 服务器之前，请执行以下步骤：

- 通过从`config.json`读取配置值，初始化`common`包的`AppConfig`标识符。`AppConfig`标识符提供配置值，例如 HTTP 服务器和 MongoDB 的主机 URI、MongoDB 数据库名称以及访问 MongoDB 数据库的身份验证凭据。
- 初始化用于签署 JWT 和验证令牌的非对称加密密钥。
- 使用`mgo`包创建 MongoDB `Session`对象。
- 向 MongoDB 集合添加索引。

这些是在启动 HTTP 服务器之前调用的一次性活动。这些功能在`common`包中实现。我们来看看`common`包中实现的自举逻辑（将在以下各节中讨论）。

图 9-4 展示了`common`包中包含的源文件。

![A978-1-4842-1052-9_9_Fig4_HTML.jpg](img/A978-1-4842-1052-9_9_Fig4_HTML.jpg)

**图 9-4.** common 包的源文件


###### 初始化配置值

配置值存放于名为 `config.json` 的文件中，该文件存储了诸如 HTTP 服务器和 MongoDB 服务器的主机 URI、连接 MongoDB 的身份验证凭据等值。这样有助于避免在应用中使用硬编码字符串。`config.json` 文件被读取后，会将 JSON 值解码到变量 `AppConfig` 中。配置值从此包变量中读取，用于启动 HTTP 服务器和连接 MongoDB 数据库。

清单 9-6 将 `config.json` 中的 JSON 字符串解码，并将值放入 `AppConfig` 中。

**清单 9-6.**  在 `utils.go` 中初始化 `AppConfig`

```go
package common

import (
        "encoding/json"
        "log"
        "os"
)

type configuration struct {
        Server, MongoDBHost, DBUser, DBPwd, Database string
}

// AppConfig 保存来自 config.json 文件的配置值
var AppConfig configuration

// Initialize AppConfig
func initConfig() {
        loadAppConfig()
}

// 读取 config.json 并解码到 AppConfig 中
func loadAppConfig() {
        file, err := os.Open("common/config.json")
        defer file.Close()
        if err != nil {
                log.Fatalf("[loadConfig]: %s\n", err)
        }

        decoder := json.NewDecoder(file)
        AppConfig = configuration{}
        err = decoder.Decode(&AppConfig)
        if err != nil {
                log.Fatalf("[loadAppConfig]: %s\n", err)
        }
}
```

###### 加载私钥/公钥 RSA 密钥

RSA 密钥被加载到两个变量中，用于表示私钥和公钥，这些密钥用于应用的授权基础设施。私钥/公钥用于生成 JWT 和验证 HTTP 请求中的令牌，以授权对应用受保护资源的访问。私钥/公钥在 HTTP 服务器启动前被加载到两个变量中，以便它们可以用于登录处理程序以及授权中间件中对 HTTP 请求的授权。

使用命令行工具 `OpenSSL` 生成密钥。要生成私钥，请在命令行窗口中运行以下命令：

```
openssl genrsa -out app.rsa 1024
```

此命令生成一个名为 `app.rsa` 的 1024 位密钥。要为私钥生成对应的公钥，请在命令行窗口中运行以下命令：

```
openssl rsa -in app.rsa -pubout > app.rsa.pub
```

此代码生成一个名为 `app.rsa.pub` 的对应公钥。RSA 密钥存储在 `keys` 目录中。

清单 9-7 从 `keys` 文件夹加载私钥/公钥 RSA 密钥，并将它们存储到两个变量中。

**清单 9-7.**  在 `auth.go` 中初始化私钥/公钥

```go
package common

import (
        "io/ioutil"
)

// 使用非对称加密 RSA 密钥
const (
    // openssl genrsa -out app.rsa 1024
    privKeyPath = "keys/app.rsa"
    // openssl rsa -in app.rsa -pubout > app.rsa.pub
    pubKeyPath  = "keys/app.rsa.pub"
)

// 用于签名的私钥和用于验证的公钥
var (
    verifyKey, signKey []byte
)

// 在启动 HTTP 处理程序之前读取密钥文件
func initKeys() {
        var err error
        signKey, err = ioutil.ReadFile(privKeyPath)
        if err != nil {
                log.Fatalf("[initKeys]: %s\n", err)
        }

        verifyKey, err = ioutil.ReadFile(pubKeyPath)
        if err != nil {
                log.Fatalf("[initKeys]: %s\n", err)
                panic(err)
        }
}
```

私钥用于签署 JWT；公钥用于验证 HTTP 请求中的 JWT，以访问 RESTful API 的资源。你可以使用 `OpenSSL` 工具生成 RSA 密钥。

###### 创建 MongoDB 会话对象

MongoDB `Session` 对象在 HTTP 服务器启动前创建。`mongoUtils.go` 中的 `createDbSession` 函数通过调用 `mgo` 包的 `DialWithInfo` 函数来创建一个 `mgo.Session` 对象。`DialWithInfo` 函数会建立一个到由 `DialInfo` 类型实例提供的 MongoDB 服务器集群的新 `Session`。MongoDB 的 URI 从 `AppConfig` 变量中读取。`Session` 对象将通过 `GetSession` 函数访问。每当执行 CRUD 操作时，都会调用 `GetSession` 函数，并且 `Session` 对象会使用 `mgo.Session` 的 `Copy` 方法进行复制。一个复制的 `Session` 对象将用于所有 CRUD 操作。

清单 9-8 提供了 `createDbSession` 和 `GetSession` 的实现，以便与 `Session` 对象一起使用。

**清单 9-8.**  `mongoUtils.go` 中的 MongoDB 会话

```go
package common

import (
    "gopkg.in/mgo.v2"
)

var session *mgo.Session

func GetSession() *mgo.Session {
        if session == nil {
                var err error
                session, err = mgo.DialWithInfo(&mgo.DialInfo{
                        Addrs:    []string{AppConfig.MongoDBHost},
                        Username: AppConfig.DBUser,
                        Password: AppConfig.DBPwd,
                        Timeout:  60 * time.Second,
                })
                if err != nil {
                        log.Fatalf("[GetSession]: %s\n", err)
                }
        }
        return session
}

func createDbSession() {
        var err error
        session, err = mgo.DialWithInfo(&mgo.DialInfo{
                Addrs:    []string{AppConfig.MongoDBHost},
                Username: AppConfig.DBUser,
                Password: AppConfig.DBPwd,
                Timeout:  60 * time.Second,
        })
        if err != nil {
                log.Fatalf("[createDbSession]: %s\n", err)
        }
}
```

###### 向 MongoDB 添加索引

索引可以提高在 MongoDB 中执行查询的性能。由于 `User` 集合经常使用 `email` 字段进行查询，`Task` 集合使用 `createdby` 字段，而 `TaskNote` 集合使用 `taskid` 字段，因此为这些字段添加了索引。`addIndexes` 函数将索引添加到 MongoDB 集合中。

清单 9-9 在 MongoDB 中添加索引。

**清单 9-9.**  在 `mongoUtils.go` 中向 MongoDB 添加索引

```go
// 向 MongoDB 添加索引
func addIndexes() {
    var err error
    userIndex := mgo.Index{
        Key:        []string{"email"},
        Unique:     true,
        Background: true,
        Sparse:     true,
    }
    taskIndex := mgo.Index{
        Key:        []string{"createdby"},
        Unique:     false,
        Background: true,
        Sparse:     true,
    }
    noteIndex := mgo.Index{
        Key:        []string{"taskid"},
        Unique:     false,
        Background: true,
        Sparse:     true,
    }

    // 向 MongoDB 添加索引
    session := GetSession().Copy()
    defer session.Close()

    userCol := session.DB(AppConfig.Database).C("users")
    taskCol := session.DB(AppConfig.Database).C("tasks")
    noteCol := session.DB(AppConfig.Database).C("notes")

    err = userCol.EnsureIndex(userIndex)
    if err != nil {
        log.Fatalf("[addIndexes]: %s\n", err)
    }

    err = taskCol.EnsureIndex(taskIndex)
    if err != nil {
        log.Fatalf("[addIndexes]: %s\n", err)
    }

    err = noteCol.EnsureIndex(noteIndex)
    if err != nil {
        log.Fatalf("[addIndexes]: %s\n", err)
    }
}
```



##### `common` 包中的初始化逻辑

`common` 包中的 `bootstrapper.go` 源文件提供了一个 `StartUp()` 函数，用于在 HTTP 服务器启动前调用必要的初始化逻辑。`main.go` 会调用 `common` 包中的 `StartUp()` 函数。

清单 9-10 给出了 `StartUp()` 函数的实现，该函数会在运行 HTTP 服务器之前调用所需的初始化逻辑。

**清单 9-10.** `bootstrapper.go` 中的 `StartUp()` 函数

```go
package common

func StartUp() {
    // 初始化 AppConfig 变量
    initConfig()
    // 初始化用于 JWT 认证的私钥/公钥
    initKeys()
    // 启动一个 MongoDB 会话
    createDbSession()
    // 向 MongoDB 添加索引
    addIndexes()
}
```

##### 启动 HTTP 服务器

HTTP 服务器在 `main.go` 中创建。

清单 9-11 给出了 `main.go` 的实现。

**清单 9-11.** `main.go` 中的程序入口点

```go
package main

import (
    "log"
    "net/http"
    "github.com/codegangsta/negroni"
    "github.com/shijuvar/go-web/taskmanager/common"
    "github.com/shijuvar/go-web/taskmanager/routers"
)

// 程序入口点
func main() {
    // 调用启动逻辑
    common.StartUp()
    // 获取 mux 路由器对象
    router := routers.InitRoutes()
    // 创建一个 negroni 实例
    n := negroni.Classic()
    n.UseHandler(router)
    server := &http.Server{
        Addr:    common.AppConfig.Server,
        Handler: n,
    }
    log.Println("正在监听...")
    server.ListenAndServe()
}
```

HTTP 服务器在 `main.go` 中创建，其中调用了 `common` 包的 `StartUp()` 函数来执行 RESTful API 应用的初始化逻辑。接着调用 `routers` 包的 `InitRoutes()` 函数来获取 `*mux.Router`，用于创建 `Negroni` 处理器。通过提供 `Negroni` 处理器来创建 `http.Server` 对象，最后启动 HTTP 服务器。HTTP 服务器的主机 URI 从 `common.AppConfig` 中读取。

### 认证

以下是应用中定义的认证工作流程：

- 用户通过向资源 `"/users/register"` 发送 HTTP 请求来注册到系统。
- 已注册用户可以通过向资源 `"/users/login"` 发送 HTTP 请求来登录系统。
- 服务器验证登录凭证，并生成一个 JWT 作为访问令牌，用于访问 RESTful API 服务器的受保护资源。
- 用户可以使用该 JWT 来访问 RESTful API 的受保护资源。用户必须将此令牌作为 Bearer 令牌，通过 HTTP 头 `"Authorization"` 发送。

认证工作流程将在后续章节中更详细地描述。

#### 生成和验证 JWT

JWT 用于授权 HTTP 请求以访问 RESTful API 资源。第三方包 `jwt-go` 用于处理 JWT（参见第 7 章）。`common` 包中的 `auth.go` 源文件提供了生成 JWT 以及使用中间件处理函数验证令牌的功能。第三方包 `jwt-go` 用于生成和验证 JWT，并使用私钥对 JWT 进行签名。如果登录用户在系统中成功通过身份验证，将在处理请求 `"/users/login"` 的应用处理函数中调用它。

清单 9-12 展示了在 HTTP 中间件处理函数中生成和验证 JWT 的辅助函数。

**清单 9-12.** `auth.go` 中用于 JWT 认证的辅助函数

```go
package common

import (
    "io/ioutil"
    "log"
    "net/http"
    "time"
    jwt "github.com/dgrijalva/jwt-go"
)

// 使用非对称加密/RSA 密钥
// 私钥/公钥文件位置
const (
    // openssl genrsa -out app.rsa 1024
    privKeyPath = "keys/app.rsa"
    // openssl rsa -in app.rsa -pubout > app.rsa.pub
    pubKeyPath  = "keys/app.rsa.pub"
)

// 用于签名的私钥和用于验证的公钥
var (
    verifyKey, signKey []byte
)

// 在启动 http 处理函数之前读取密钥文件
func initKeys() {
    var err error
    signKey, err = ioutil.ReadFile(privKeyPath)
    if err != nil {
        log.Fatalf("[initKeys]: %s\n", err)
    }
    verifyKey, err = ioutil.ReadFile(pubKeyPath)
    if err != nil {
        log.Fatalf("[initKeys]: %s\n", err)
        panic(err)
    }
}

// 生成 JWT 令牌
func GenerateJWT(name, role string) (string, error) {
    // 为 rsa 256 创建一个签名器
    t := jwt.New(jwt.GetSigningMethod("RS256"))
    // 设置 JWT 令牌的声明（claims）
    t.Claims["iss"] = "admin"
    t.Claims["UserInfo"] = struct {
        Name string
        Role string
    }{name, role}
    // 设置 JWT 令牌的过期时间
    t.Claims["exp"] = time.Now().Add(time.Minute * 20).Unix()
    tokenString, err := t.SignedString(signKey)
    if err != nil {
        return "", err
    }
    return tokenString, nil
}

// 用于验证 JWT 令牌的中间件
func Authorize(w http.ResponseWriter, r *http.Request, next http.HandlerFunc) {
    // 验证令牌
    token, err := jwt.ParseFromRequest(r, func(token *jwt.Token) (interface{}, error) {
        // 使用公钥验证令牌，公钥是私钥的对应部分
        return verifyKey, nil
    })
    if err != nil {
        switch err.(type) {
        case *jwt.ValidationError: // JWT 验证错误
            vErr := err.(*jwt.ValidationError)
            switch vErr.Errors {
            case jwt.ValidationErrorExpired: // JWT 已过期
                DisplayAppError(
                    w,
                    err,
                    "访问令牌已过期，请获取新令牌",
                    401,
                )
                return
            default:
                DisplayAppError(w,
                    err,
                    "解析访问令牌时出错！",
                    500,
                )
                return
            }
        default:
            DisplayAppError(w,
                err,
                "解析访问令牌时出错！",
                500)
            return
        }
    }
    if token.Valid {
        next(w, r)
    } else {
        DisplayAppError(w,
            err,
            "无效的访问令牌",
            401,
        )
    }
}
```

##### 生成 JWT

`GenerateJWT()` 函数使用私钥生成 JWT。在 JWT 上设置了各种声明（claims），包括令牌的过期信息。`go-jwt` 包用于对编码后的安全令牌进行签名。当登录过程成功时，在处理请求 `"/users/login"` 的应用处理函数中会调用 `GenerateJWT()` 函数。

当用户登录系统时，服务器会返回一个如下的 JSON 响应：

```json
{"data":
    {"user":{"id":"55b9f7e13f06221910000001","firstname":"Shiju","lastname":"Varghese","email":"shijuvar@gmail.com"},
    "token":"eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJVc2VySW5mbyI6eyJOYW1lIjoic2hpanVAZ21haWwuY29tIiwiUm9sZSI6Im1lbWJlciJ9LCJleHAiOjE0MzgyNTI5MzgsImlzcyI6ImFkbWluIn0.WtdM55KE0cNlj5c2VYwtIUQS8L6UI_ViLiwe0wH_0cpDj0dKkMTMtZ6LSHoIxtZyt92z19WX5gQCi3z-7Mly4kPe5Yvp3IXuDNdgJvBkQvEd_xg0-Vx9bhm_ztf0Hb2CInsVgux49EIxgjFoinwdzrxmM9ZbY7msBYSKutcRKLU"}
}
```

用户可以从这个 JSON 响应中获取 JSON 字段 `"token"` 中的 JWT，该令牌可用于授权 HTTP 请求以访问 RESTful API 的受保护资源。

JWT 由三部分组成，用 `.`（句点）分隔：
- `头部（Header）`
- `载荷（Payload）`
- `签名（Signature）`

JWT 是一种基于 JSON 的安全编码，可以解码以获取这些部分的 JSON 表示。

图 9-5 显示了`头部`和`载荷`部分的解码表示。

![A978-1-4842-1052-9_9_Fig5_HTML.jpg](img/A978-1-4842-1052-9_9_Fig5_HTML.jpg)

**图 9-5.** 解码后的 JWT JSON 表示

`头部`部分包含生成令牌所使用的算法（这里是 RS256）以及类型（JWT）。`载荷`部分携带 JWT 的声明（claims），您可以在其中提供用户信息、过期时间以及其他关于 JWT 的信息。



##### 向服务器发送 JWT

当用户成功登录系统后，他们会提供一个 JWT 作为访问令牌，用于授权后续对 API 服务器的 HTTP 请求。因此，每当发送 HTTP 请求访问受保护资源时，必须在 HTTP 请求中携带 JWT 才能获得 API 服务器的授权。登录流程成功完成后，返回的令牌字符串可以存入任意类型的客户端存储中，以便在发送 HTTP 请求访问 RESTful API 资源时轻松获取。

在前端 Web 应用中，你可以使用 HTML5 Web 存储（`localStorage/sessionStorage`）或 Web Cookie 来持久化存储 JWT。但将 JWT 存入各种客户端应用存储时，需要考虑安全性问题。

当发送 HTTP 请求访问 RESTful API 资源时，必须在 HTTP 请求头`"Authorization"`中提供 Bearer 令牌。

以下是通过`"Authorization"`请求头发送 JWT 字符串的格式：

`"Authorization": "Bearer token_string""

下面是一个示例，演示如何通过`"Authorization"`请求头将 JWT 作为 Bearer 令牌提供：

`"Authorization":"Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ"`

##### 授权 JWT

`Authorize`中间件处理函数负责授权 HTTP 请求，它会验证 HTTP 请求的`"Authorization"`请求头中是否包含有效的 JWT 作为 Bearer 令牌。`go-jwt`包的`ParseFromRequest`辅助函数用于使用公钥验证令牌。在本应用中，使用一个私钥对令牌进行签名，因此其对应的公钥也用于验证令牌：

```
token, err := jwt.ParseFromRequest(r, func(token *jwt.Token) (interface{}, error) {
    return verifyKey, nil
})
```

如果请求包含有效的令牌，中间件函数会调用中间件堆栈中的下一个处理函数。如果令牌无效，则会调用`DisplayAppError`辅助函数，以 JSON 格式显示 HTTP 错误。调用`DisplayAppError`函数时，会为错误提供自定义错误消息和 HTTP 状态码。HTTP 状态码 401 表示 HTTP 状态`"Unauthorized"`：

```
if token.Valid {
    next(w, r)
} else {
    w.WriteHeader(http.StatusUnauthorized)
    DisplayAppError(
        w,
        err,
        "Invalid Access Token",
        401,
    )
}
```

当调用`next(w, r)`时，它会调用下一个处理函数。使用`Negroni`包来组织中间件堆栈，其签名格式为`func (http.ResponseWriter, *http.Request, http.HandlerFunc)`，用于编写与`Negroni`配合使用的中间件处理函数。所有对 URL 路径`"/tasks"`和`"/notes"`的 HTTP 请求都必须通过有效的访问令牌进行授权。因此，`Authorize`中间件函数被添加到`Negroni`中间件堆栈中。

以下是`routers`包中`task.go`文件里的代码块，用于向`"/tasks"`路径添加身份验证中间件函数：

```
func SetTaskRoutes(router *mux.Router) *mux.Router {
    taskRouter := mux.NewRouter()
    taskRouter.HandleFunc("/tasks", controllers.CreateTask).Methods("POST")
    taskRouter.HandleFunc("/tasks/{id}", controllers.UpdateTask).Methods("PUT")
    taskRouter.HandleFunc("/tasks", controllers.GetTasks).Methods("GET")
    taskRouter.HandleFunc("/tasks/{id}", controllers.GetTaskById).Methods("GET")
    taskRouter.HandleFunc("/tasks/users/{id}", controllers.GetTasksByUser).Methods("GET")
    taskRouter.HandleFunc("/tasks/{id}", controllers.DeleteTask).Methods("DELETE")
    router.PathPrefix("/tasks").Handler(negroni.New(
        negroni.HandlerFunc(common.Authorize),
        negroni.Wrap(taskRouter),
    ))
    return router
}
```

你可以通过`router`实例的`PathPrefix`函数，向特定路由添加中间件函数。

以下是`routers`包中`note.go`文件里的代码块，用于向`"/notes"`路径添加身份验证中间件函数：

```
func SetNoteRoutes(router *mux.Router) *mux.Router {
    noteRouter := mux.NewRouter()
    noteRouter.HandleFunc("/notes", controllers.CreateNote).Methods("POST")
    noteRouter.HandleFunc("/notes/{id}", controllers.UpdateNote).Methods("PUT")
    noteRouter.HandleFunc("/notes/{id}", controllers.GetNoteById).Methods("GET")
    noteRouter.HandleFunc("/notes", controllers.GetNotes).Methods("GET")
    noteRouter.HandleFunc("/notes/tasks/{id}", controllers.GetNotesByTask).Methods("GET")
    noteRouter.HandleFunc("/notes/{id}", controllers.DeleteNote).Methods("DELETE")
    router.PathPrefix("/notes").Handler(negroni.New(
        negroni.HandlerFunc(common.Authorize),
        negroni.Wrap(noteRouter),
    ))
    return router
}
```

中间件函数非常适合在应用程序处理程序之间实现共享功能。中间件函数也可以应用于特定路由，就像对 URL 路径`"/tasks"`和`"/notes"`做的那样。

#### 应用程序处理程序

前面章节介绍了应用程序数据模型、RESTful API 资源建模及其与应用程序 HTTP 路由的映射、使用基本初始化逻辑设置 HTTP 服务器，以及 RESTful API 的身份验证。现在，让我们来看看为每个路由处理 HTTP 请求的应用程序处理程序。

应用程序处理程序组织在`controllers`包中。图 9-6 显示了`controllers`包中包含的源文件。

![A978-1-4842-1052-9_9_Fig6_HTML.jpg](img/A978-1-4842-1052-9_9_Fig6_HTML.jpg)

**图 9-6.** `controllers`包的源文件



##### 用于显示 HTTP 错误的辅助函数

Go 语言中的错误处理与大多数主流编程语言不同，因为它提供了一种简单且极简的方法来处理异常，即使用内置的`error`类型。`error`类型的值用于指示应用程序中的异常状态。当你查看标准库包时，会发现大多数函数都返回多个值，其中包括一个`error`类型的值，用于指示异常状态。

你可以检查`error`类型的值，以查看函数执行期间是否发生了任何异常：

```
file, err := os.Open("common/config.json")
if err != nil {
    log.Fatalf("[loadConfig]: %s\n", err)
}
```

如果错误变量包含任何值，则会调用`log`包的`Fatalf`函数。`Fatalf`函数在写入日志消息后终止程序。在 RESTful API 应用程序的 HTTP 处理程序中，程序不会因任何类型的异常状态而终止。相反，会以 JSON 响应作为 HTTP 响应返回，并附带适当的 HTTP 状态码。在`common`包中编写了一个辅助函数，用于以 JSON 格式显示 HTTP 错误，以便客户端应用程序能够理解其 HTTP 请求中的任何问题。

清单 9-13 提供了一个辅助函数，用于以 JSON 格式显示 HTTP 错误，该函数在错误响应中包含适当的 HTTP 状态码。

**清单 9-13.** `utils.go` 中用于显示错误的辅助函数

```
package common

import (
    "encoding/json"
    "log"
    "net/http"
)

type (
    appError struct {
        Error      string `json:"error"`
        Message    string `json:"message"`
        HttpStatus int    `json:"status"`
    }
    errorResource struct {
        Data appError `json:"data"`
    }
)

func DisplayAppError(w http.ResponseWriter, handlerError error, message string, code int) {
    errObj := appError{
        Error:      handlerError.Error(),
        Message:    message,
        HttpStatus: code,
    }
    log.Printf("AppError]: %s\n", handlerError)
    w.Header().Set("Content-Type", "application/json; charset=utf-8")
    w.WriteHeader(code)
    if j, err := json.Marshal(errorResource{Data: errObj}); err == nil {
        w.Write(j)
    }
}
```

编写了一个名为 `DisplayAppError` 的辅助函数，用于以 JSON 格式提供错误消息作为 HTTP 响应。客户端应用程序可以检查 HTTP 状态码，以验证 HTTP 请求是否成功。使用名为 `appError` 的结构体类型创建用于提供错误消息的模型对象。在 `appError` 类型中，`Error` 属性用于保存错误对象的字符串值，`Message` 属性用于保存关于错误的自定义消息，`HttpStatus` 属性用于保存 HTTP 状态码。通过提供 `appError` 的值来创建 `errorResource` 类型的实例，以将响应编码为 JSON。

图 9-7 展示了一个包含过期访问令牌的无效 HTTP 请求的错误响应。

![A978-1-4842-1052-9_9_Fig7_HTML.jpg](img/A978-1-4842-1052-9_9_Fig7_HTML.jpg)

**图 9-7.** 用于 JSON 错误消息的 HTTP 响应

`DisplayAppError` 函数是一个简单的辅助函数，用于提供 HTTP 错误。你也可以在 HTTP 中间件中编写错误处理逻辑，该逻辑可用于包装应用程序处理程序，这是在 Go Web 应用程序中实现错误处理的更优雅方法。如果在应用程序处理程序中发生任何错误，你可以返回一个用于保存错误数据的模型对象，并且在中间件函数中，你可以检查错误模型是否包含任何值。如果发生错误，你还可以提供 HTTP 错误响应。本章不讨论这种方法，但你可以在构建自己的 Web 应用程序时尝试使用。

##### 处理 HTTP 请求生命周期中的数据

让我们定义一个用于在 HTTP 请求生命周期内处理数据的类型。这里定义了一个`Context`结构体类型，将 MongoDB 的`Session`对象作为其属性，并暴露了`DbCollection`方法用于获取 MongoDB 的`Collection`对象，以及`Close`方法用于关闭 MongoDB 的`Session`对象。

清单 9-14 提供了`Context`结构体类型的代码块，该类型将与 HTTP 处理函数一起使用，以在 HTTP 请求生命周期内保存数据。

**清单 9-14.** `context.go` 中的 `Context` 结构体

```
package controllers

import (
    "gopkg.in/mgo.v2"
    "github.com/shijuvar/go-web/taskmanager/common"
)

// Struct used for maintaining HTTP Request Context
type Context struct {
    MongoSession *mgo.Session
}

// Close mgo.Session
func (c *Context) Close() {
    c.MongoSession.Close()
}

// Returns mgo.collection for the given name
func (c *Context) DbCollection(name string) *mgo.Collection {
    return c.MongoSession.DB(common.AppConfig.Database).C(name)
}

// Create a new Context object for each HTTP request
func NewContext() *Context {
    session := common.GetSession().Copy()
    context := &Context{
        MongoSession: session,
    }
    return context
}
```

这个源文件提供了一个`NewContext`函数，该函数通过提供 MongoDB `Session`对象的副本，返回一个`Context`类型的实例。调用`common`包的`GetSession`函数来获取`Session`对象，并从中创建一个副本。在应用程序处理程序中，调用`NewContext`函数来获取`Context`类型的实例，该实例中的`Context`类型的 MongoDB `Session`对象用于对 MongoDB 数据库执行 CRUD 操作。

在`Context`类型中，你只是存储了 MongoDB 的`Session`对象，但你可以在`Context`类型中使用任何类型的数据，以便与 HTTP 请求的生命周期一起使用。在许多用例中，你可能需要在各种中间件处理函数和应用程序处理函数之间共享这些数据。简而言之，你需要在不同的处理函数之间共享数据。

在这种情况下，你可以使用一种机制来存储对象，以便在 HTTP 请求上下文中工作。来自 Gorilla Web 工具包（www.gorillatoolkit.org/pkg/context）的`context`包提供了将数据放入 HTTP `Context`对象的功能，以便在 HTTP 请求的生命周期内保存数据。你可以将一个处理程序中的数据放入 HTTP 上下文中，该数据可以被其他处理程序访问。在 RESTful API 示例中，你在应用程序处理程序中使用这些数据，并且不需要在处理程序之间共享数据。因此，你没有将`Context`结构体放入 HTTP `Context`对象中。

#### 用户资源的处理程序

以下是为用户资源指定的路由：

```
router.HandleFunc("/users/register", controllers.Register).Methods("POST")
router.HandleFunc("/users/login", controllers.Login).Methods("POST")
```

为用户资源指定了两条路由。资源`/users/register`用于将用户注册到系统中，`/users/login`用于登录系统以获取访问令牌，该令牌用于授权访问 RESTful API 资源的 HTTP 请求。



##### JSON 资源模型

RESTful API 是一种基于 JSON 的 API，客户端应用需要以 JSON 格式发送数据，服务器也同样以 JSON 格式返回响应。为了遵循 JSON API 标准（`http://jsonapi.org`），我们定义了资源模型，用于以格式化 JSON 发送和接收数据。`"data"` 被定义为所有 JSON 表示形式的根元素。

列表 9-15 展示了将与 `Users` 资源配合使用的资源模型。

**列表 9-15.** 在 `resources.go` 中处理 `/users` 的 JSON 资源

```
package controllers

import (
	"github.com/shijuvar/go-web/taskmanager/models"
)

type (
	//For Post - /user/register
	UserResource struct {
		Data models.User `json:"data"`
	}

	//For Post - /user/login
	LoginResource struct {
		Data LoginModel `json:"data"`
	}

	// Response for authorized user Post - /user/login
	AuthUserResource struct {
		Data AuthUserModel `json:"data"`
	}

	//Model for authentication
	LoginModel struct {
		Email    string `json:"email"`
		Password string `json:"password"`
	}

	//Model for authorized user with access token
	AuthUserModel struct {
		User  models.User `json:"user"`
		Token string      `json:"token"`
	}
)
```

##### 用户资源的处理函数

`Users` 资源的应用处理函数编写在 `userController.go` 源文件中，该文件归属于 `controllers` 包。

列表 9-16 提供了 `Users` 资源处理函数的实现。

**列表 9-16.** `userController.go` 中的应用处理函数

```
package controllers

import (
	"encoding/json"
	"net/http"
	"github.com/shijuvar/go-web/taskmanager/common"
	"github.com/shijuvar/go-web/taskmanager/data"
	"github.com/shijuvar/go-web/taskmanager/models"
)

// Handler for HTTP Post - "/users/register"
// Add a new User document
func Register(w http.ResponseWriter, r *http.Request) {
	var dataResource UserResource
	// Decode the incoming User json
	err := json.NewDecoder(r.Body).Decode(&dataResource)
	if err != nil {
		common.DisplayAppError(
			w,
			err,
			"Invalid User data",
			500,
		)
		return
	}
	user := &dataResource.Data
	context := NewContext()
	defer context.Close()
	c := context.DbCollection("users")
	repo := &data.UserRepository{c}
	// Insert User document
	repo.CreateUser(user)
	// Clean-up the hashpassword to eliminate it from response
	user.HashPassword = nil
	if j, err := json.Marshal(UserResource{Data: *user}); err != nil {
		common.DisplayAppError(
			w,
			err,
			"An unexpected error has occurred",
			500,
		)
		return
	} else {
		w.Header().Set("Content-Type", "application/json")
		w.WriteHeader(http.StatusCreated)
		w.Write(j)
	}
}

// Handler for HTTP Post - "/users/login"
// Authenticate with username and apssword
func Login(w http.ResponseWriter, r *http.Request) {
	var dataResource LoginResource
	var token string
	// Decode the incoming Login json
	err := json.NewDecoder(r.Body).Decode(&dataResource)
	if err != nil {
		common.DisplayAppError(
			w,
			err,
			"Invalid Login data",
			500,
		)
		return
	}
	loginModel := dataResource.Data
	loginUser := models.User{
		Email:    loginModel.Email,
		Password: loginModel.Password,
	}
	context := NewContext()
	defer context.Close()
	c := context.DbCollection("users")
	repo := &data.UserRepository{c}
	// Authenticate the login user
	if user, err := repo.Login(loginUser); err != nil {
		common.DisplayAppError(
			w,
			err,
			"Invalid login credentials",
			401,
		)
		return
	} else { //if login is successful
		// Generate JWT token
		token, err = common.GenerateJWT(user.Email, "member")
		if err != nil {
			common.DisplayAppError(
				w,
				err,
				"Eror while generating the access token",
				500,
			)
			return
		}
		w.Header().Set("Content-Type", "application/json")
		user.HashPassword = nil
		authUser := AuthUserModel{
			User:  user,
			Token: token,
		}
		j, err := json.Marshal(AuthUserResource{Data: authUser})
		if err != nil {
```



##### 注册新用户

要注册新用户，客户端应用程序应向 URI `"/users/register"` 发送 HTTP `Post` 请求。在 `Register` 处理函数中，传入的 JSON 字符串被解码为 `UserResource` 类型，并通过访问 `UserResource` 对象的 `Data` 属性来创建 `models.User` 结构体的实例。

`Context` 类型用于在应用程序处理函数中访问 MongoDB 的 `Session` 对象（`mgo.Session`）。因此，通过调用 `DbCollection` 方法来创建 `Context` 类型的实例和 MongoDB 的 `Collection` 对象（`mgo.Collection`）：

```
context := NewContext()
defer context.Close()
c := context.DbCollection("users")
```

`Context` 类型的 `Close` 方法被添加到 `defer` 函数中，以关闭 MongoDB 的 `Session` 对象，这是该 `Session` 对象的一个副本。在所有 HTTP 处理函数中，都会创建 MongoDB `Session` 对象的一个副本，并在单个 HTTP 请求生命周期内使用同一个实例，最后通过 `defer` 函数释放资源。

在 `Register` 处理函数中，通过提供 MongoDB 的 `Collection` 对象来创建 `UserRepository` 的实例。`UserRepository` 结构体的 `CreateUser` 方法用于将 `User` 对象持久化到 MongoDB 数据库中。所有数据持久化逻辑都编写在 `data` 包中：

```
repo := &data.UserRepository{c}
// Insert User document
repo.CreateUser(user)
```

`UserRepository` 提供了针对 `User` 实体的所有 CRUD 操作。（本章下一节将讨论此主题。）`Register` 处理函数将新创建的 `User` 实体的 JSON 表示形式作为响应发送回去。

现在，让我们使用 RESTful API 客户端工具 `"Postman"`（[`www.getpostman.com/`](http://www.getpostman.com/)）来测试 `Users` 资源的功能。该工具允许你测试你的 API（对于测试 RESTful API 也非常有用）。

图 9-8 展示了使用 RESTful API 客户端工具 `"Postman"` 发送到 URI 端点 `"users/register"` 的 HTTP `Post` 请求。

![A978-1-4842-1052-9_9_Fig8_HTML.jpg](img/A978-1-4842-1052-9_9_Fig8_HTML.jpg)

图 9-8. HTTP Post 到 “/users/register”

图 9-9 展示了来自 RESTful API 服务器的响应，表明已创建新资源。

![A978-1-4842-1052-9_9_Fig9_HTML.jpg](img/A978-1-4842-1052-9_9_Fig9_HTML.jpg)

图 9-9. 来自 “/users/register” 的 HTTP 响应

##### 登录系统

客户端应用程序的用户必须获取 JWT 才能访问 RESTful API 的受保护资源。要获取令牌，用户必须使用用户名和密码登录系统。如果用户通过身份验证，服务器将发回一个可用于访问 RESTful API 资源的 JWT。

应用程序处理函数 `Login` 用于处理对登录资源的 HTTP 请求。如果登录成功，则调用 `common` 包的 `GenerateJWT` 函数来生成 JWT。生成的 JWT 包含在 JSON 响应中：

```go
if user, err := repo.Login(loginUser); err != nil {
	common.DisplayAppError(
		w,
		err,
		"Invalid login credentials",
		401,
	)
	return
} else { //if login is successful
	// Generate JWT token
	token, err = common.GenerateJWT(user.Email, "member")
	if err != nil {
		common.DisplayAppError(
			w,
			err,
			"Eror while generating the access token",
			500,
		)
		return
	}

	w.Header().Set("Content-Type", "application/json")
	user.HashPassword = nil
	authUser := AuthUserModel{
		User:  user,
		Token: token,
	}

	j, err := json.Marshal(AuthUserResource{Data: authUser})
	if err != nil {
		common.DisplayAppError(
			w,
			err,
			"An unexpected error has occurred",
			500,
		)
		return
	}

	w.WriteHeader(http.StatusOK)
	w.Write(j)
}
```



图 9-10 展示了通过向 URI 端点 `"users/register"` 发送 HTTP `Post` 请求，将登录凭据发送至系统以获取 JWT 的过程。

![A978-1-4842-1052-9_9_Fig10_HTML.jpg](img/A978-1-4842-1052-9_9_Fig10_HTML.jpg)

图 9-10. 向 `/users/login` 发送 HTTP Post 请求

图 9-11 展示了成功登录系统后，RESTful API 服务器返回的 HTTP 响应。

![A978-1-4842-1052-9_9_Fig11_HTML.jpg](img/A978-1-4842-1052-9_9_Fig11_HTML.jpg)

图 9-11. 来自 `/users/login` 的 HTTP 响应

服务器会将包含用户信息和 JWT 的响应发回给客户端应用。

##### 使用 MongoDB 实现数据持久化

第 8 章 讨论了如何使用 MongoDB 执行 CRUD 操作。在 RESTful API 应用程序中，数据持久化的逻辑被组织在 `data` 包中。其中包含独立的结构体类型，用于针对应用程序的每个数据模型执行 CRUD 操作。

图 9-12 展示了 `data` 包中包含的源文件。

![A978-1-4842-1052-9_9_Fig12_HTML.jpg](img/A978-1-4842-1052-9_9_Fig12_HTML.jpg)

图 9-12. data 包中的源文件

代码清单 9-17 提供了 `userRepository.go` 的源代码，该文件实现了插入新用户及使用用户凭据登录的功能。

代码清单 9-17. `userRepository.go` 中的用户实体数据持久化逻辑

```
package data

import (
	"github.com/shijuvar/go-web/taskmanager/models"
	"golang.org/x/crypto/bcrypt"
	"gopkg.in/mgo.v2"
	"gopkg.in/mgo.v2/bson"
)

type UserRepository struct {
	C *mgo.Collection
}

func (r *UserRepository) CreateUser(user *models.User) error {
	obj_id := bson.NewObjectId()
	user.Id = obj_id
	hpass, err := bcrypt.GenerateFromPassword([]byte(user.Password), bcrypt.DefaultCost)
	if err != nil {
		panic(err)
	}
	user.HashPassword = hpass
	//清除传入的文本密码
	user.Password = ""
	err = r.C.Insert(&user)
	return err
}

func (r *UserRepository) Login(user models.User) (u models.User, err error) {
	err = r.C.Find(bson.M{"email": user.Email}).One(&u)
	if err != nil {
		return
	}
	// 验证密码
	err = bcrypt.CompareHashAndPassword(u.HashPassword, []byte(user.Password))
	if err != nil {
		u = models.User{}
	}
	return
}
```

`UserRepository` 结构体有一个类型为 `*mgo.Collection` 的属性 `C`。`UserRepository` 结构体的实例是通过 `userController.go` 源文件创建的，它需要提供一个 `mgo.Collection` 对象，而该对象将由 `Context` 类型创建。`bcrypt` 包用于通过哈希算法加密密码，并使用 `CompareHashAndPassword` 方法验证密码。

#### Tasks 资源的处理器

以下是针对 `Tasks` 资源配置的路由：

```
taskRouter := mux.NewRouter()
taskRouter.HandleFunc("/tasks", controllers.CreateTask).Methods("POST")
taskRouter.HandleFunc("/tasks/{id}", controllers.UpdateTask).Methods("PUT")
taskRouter.HandleFunc("/tasks", controllers.GetTasks).Methods("GET")
taskRouter.HandleFunc("/tasks/{id}", controllers.GetTaskById).Methods("GET")
taskRouter.HandleFunc("/tasks/users/{id}", controllers.GetTasksByUser).Methods("GET")
taskRouter.HandleFunc("/tasks/{id}", controllers.DeleteTask).Methods("DELETE")

router.PathPrefix("/tasks").Handler(negroni.New(
	negroni.HandlerFunc(common.Authorize),
	negroni.Wrap(taskRouter),
))
```

对 URL 路径 `"/tasks"` 的请求会被一个名为 `Authorize` 的授权中间件处理器修饰，该处理器由 `Negroni` 栈提供。已认证的用户可以通过在 HTTP 请求中提供 JWT 作为访问令牌发送至服务器，从而创建 `Tasks`。

##### JSON 资源模型

对 `"/tasks"` 资源的操作使用名为 `Task` 的数据模型来持久化数据对象的值。



列表 9-18 提供了用于表示 JSON 数据的资源模型，这些模型用于通过 `Tasks` 资源发送和接收消息。

**列表 9-18.** `resources.go` 中用于处理 “/tasks” 的 `JSON` 资源

```
// JSON 资源的模型
type (
	// 用于 Post/Put - /tasks
	// 用于 Get - /tasks/id
	TaskResource struct {
		Data models.Task `json:"data"`
	}

	// 用于 Get - /tasks
	TasksResource struct {
		Data []models.Task `json:"data"`
	}
)
```

##### Tasks 资源的处理函数

`Tasks` 资源的应用程序处理函数被写在 `taskController.go` 源文件中，该文件位于 `controllers` 包中。

列表 9-19 提供了 `Tasks` 资源的处理函数。

**列表 9-19.** `taskController.go` 中的应用程序处理函数

```
package controllers

import (
	"encoding/json"
	"log"
	"net/http"

	"github.com/gorilla/mux"
	"github.com/shijuvar/go-web/taskmanager/data"
	"gopkg.in/mgo.v2"
	"gopkg.in/mgo.v2/bson"
)

// HTTP Post 的处理函数 - "/tasks"
// 插入一条新的 Task 文档
func CreateTask(w http.ResponseWriter, r *http.Request) {
	var dataResource TaskResource

	// 解码传入的 Task json
	err := json.NewDecoder(r.Body).Decode(&dataResource)
	if err != nil {
		common.DisplayAppError(
			w,
			err,
			"无效的任务数据",
			500,
		)
		return
	}
	task := &dataResource.Data
	context := NewContext()
	defer context.Close()
	c := context.DbCollection("tasks")
	repo := &data.TaskRepository{c}
	// 插入一个任务文档
	repo.Create(task)
	if j, err := json.Marshal(TaskResource{Data: *task}); err != nil {
		common.DisplayAppError(
			w,
			err,
			"发生了意外的错误",
			500,
		)
		return
	} else {
		w.Header().Set("Content-Type", "application/json")
		w.WriteHeader(http.StatusCreated)
		w.Write(j)
	}
}

// HTTP Get 的处理函数 - "/tasks"
// 返回所有 Task 文档
func GetTasks(w http.ResponseWriter, r *http.Request) {
	context := NewContext()
	defer context.Close()
	c := context.DbCollection("tasks")
	repo := &data.TaskRepository{c}
	tasks := repo.GetAll()
	j, err := json.Marshal(TasksResource{Data: tasks})
	if err != nil {
		common.DisplayAppError(
			w,
			err,
			"发生了意外的错误",
			500,
		)
		return
	}
	w.WriteHeader(http.StatusOK)
	w.Header().Set("Content-Type", "application/json")
	w.Write(j)
}

// HTTP Get 的处理函数 - "/tasks/{id}"
// 通过 id 返回单个 Task 文档
func GetTaskById(w http.ResponseWriter, r *http.Request) {
	// 从传入的 url 中获取 id
	vars := mux.Vars(r)
	id := vars["id"]
	context := NewContext()
	defer context.Close()
	c := context.DbCollection("tasks")
	repo := &data.TaskRepository{c}
	task, err := repo.GetById(id)
	if err != nil {
		if err == mgo.ErrNotFound {
			w.WriteHeader(http.StatusNoContent)
			return
		} else {
			common.DisplayAppError(
				w,
				err,
				"发生了意外的错误",
				500,
			)
			return
		}
	}
	if j, err := json.Marshal(task); err != nil {
		common.DisplayAppError(
			w,
			err,
			"发生了意外的错误",
			500,
		)
		return
	} else {
		w.Header().Set("Content-Type", "application/json")
		w.WriteHeader(http.StatusOK)
		w.Write(j)
	}
}

// HTTP Get 的处理函数 - "/tasks/users/{id}"
// 返回某个用户创建的所有任务
func GetTasksByUser(w http.ResponseWriter, r *http.Request) {
	// 从传入的 url 中获取 id
	vars := mux.Vars(r)
	user := vars["id"]
	context := NewContext()
	defer context.Close()
	c := context.DbCollection("tasks")
	repo := &data.TaskRepository{c}
	tasks := repo.GetByUser(user)
	j, err := json.Marshal(TasksResource{Data: tasks})
	if err != nil {
		common.DisplayAppError(
			w,
			err,
			"发生了意外的错误",
			500,
		)
		return
	}
	w.WriteHeader(http.StatusOK)
	w.Header().Set("Content-Type", "application/json")
	w.Write(j)
}

// HTTP Put 的处理函数 - "/tasks/{id}"
// 更新一个现有的 Task 文档
func UpdateTask(w http.ResponseWriter, r *http.Request) {
	// 从传入的 url 中获取 id
	vars := mux.Vars(r)
```



`id := bson.ObjectIdHex(vars["id"])`

`var dataResource TaskResource`

`// 解码传入的 Task JSON`

`err := json.NewDecoder(r.Body).Decode(&dataResource)`

`if err != nil {`

`common.DisplayAppError(`

`w,`

`err,`

`"无效的任务数据",`

`500,`

`)`

`return`

`}`

`task := &dataResource.Data`

`task.Id = id`

`context := NewContext()`

`defer context.Close()`

`c := context.DbCollection("tasks")`

`repo := &data.TaskRepository{c}`

`// 更新一个现有的 Task 文档`

`if err := repo.Update(task); err != nil {`

`common.DisplayAppError(`

`w,`

`err,`

`"发生意外错误",`

`500,`

`)`

`return`

`} else {`

`w.WriteHeader(http.StatusNoContent)`

`}`

`}`

`// HTTP Delete 的处理函数 - "/tasks/{id}"`

`// 删除一个现有的 Task 文档`

`func DeleteTask(w http.ResponseWriter, r *http.Request) {`

`vars := mux.Vars(r)`

`id := vars["id"]`

`context := NewContext()`

`defer context.Close()`

`c := context.DbCollection("tasks")`

`repo := &data.TaskRepository{c}`

`// 删除一个现有的 Task 文档`

`err := repo.Delete(id)`

`if err != nil {`

`common.DisplayAppError(`

`w,`

`err,`

`"发生意外错误",`

`500,`

`)`

`return`

`}`

`w.WriteHeader(http.StatusNoContent)`

`}`

`taskController.go` 中的处理函数使用 `"data"` 包对数据模型 `Task` 执行 CRUD 操作，该操作在 `taskRepository` 结构体中实现。

清单 9-20 展示了 `taskRepository.go` 中编写的数据持久化逻辑。

清单 9-20. `taskRepository.go` 中 Task 实体的数据持久化逻辑

```
package data

import (
	"time"
	"github.com/shijuvar/go-web/taskmanager/models"
	"gopkg.in/mgo.v2"
	"gopkg.in/mgo.v2/bson"
)

type TaskRepository struct {
	C *mgo.Collection
}

func (r *TaskRepository) Create(task *models.Task) error {
	obj_id := bson.NewObjectId()
	task.Id = obj_id
	task.CreatedOn = time.Now()
	task.Status = "Created"
	err := r.C.Insert(&task)
	return err
}

func (r *TaskRepository) Update(task *models.Task) error {
	// 对 MogoDB 进行部分更新
	err := r.C.Update(bson.M{"_id": task.Id},
		bson.M{"$set": bson.M{
			"name":        task.Name,
			"description": task.Description,
			"due":         task.Due,
			"status":      task.Status,
			"tags":        task.Tags,
		}})
	return err
}

func (r *TaskRepository) Delete(id string) error {
	err := r.C.Remove(bson.M{"_id": bson.ObjectIdHex(id)})
	return err
}

func (r *TaskRepository) GetAll() []models.Task {
	var tasks []models.Task
	iter := r.C.Find(nil).Iter()
	result := models.Task{}
	for iter.Next(&result) {
		tasks = append(tasks, result)
	}
	return tasks
}

func (r *TaskRepository) GetById(id string) (task models.Task, err error) {
	err = r.C.FindId(bson.ObjectIdHex(id)).One(&task)
	return
}

func (r *TaskRepository) GetByUser(user string) []models.Task {
	var tasks []models.Task
	iter := r.C.Find(bson.M{"createdby": user}).Iter()
	result := models.Task{}
	for iter.Next(&result) {
		tasks = append(tasks, result)
	}
	return tasks
}
```

##### 测试 Tasks 资源的 API 操作

让我们使用一个 RESTful API 客户端工具来测试 `Tasks` 资源的 API 操作。要访问 `Tasks` 资源的 API 操作，客户端应用程序必须在 `"Authorization"` 头中提供一个 JWT 作为 Bearer Token。你可以从 URI 端点 `"/users/login"` 获取 JWT。

图 9-13 展示了通过向 `"/Tasks"` 发送 HTTP `Post` 请求，在请求体中提供 JSON 数据以创建一个新的 `Task` 资源。

![A978-1-4842-1052-9_9_Fig13_HTML.jpg](img/A978-1-4842-1052-9_9_Fig13_HTML.jpg)

图 9-13. 向 `"/tasks"` 发送 HTTP Post 的请求体

图 9-14 展示了向 `"/Tasks"` 发送的 HTTP `Post` 请求在 `"Authorization"` 头中提供了一个 JWT。

![A978-1-4842-1052-9_9_Fig14_HTML.jpg](img/A978-1-4842-1052-9_9_Fig14_HTML.jpg)

图 9-14. 向 `"/tasks"` 发送 HTTP Post 的授权头



图 9-15 展示了 RESTful API 服务器对 HTTP `POST` 请求 `"/tasks"` 的响应结果。

![A978-1-4842-1052-9_9_Fig15_HTML.jpg](img/A978-1-4842-1052-9_9_Fig15_HTML.jpg)

图 9-15. HTTP `POST` 请求 `"/tasks"` 的响应结果

图 9-16 展示了 HTTP `GET` 请求 `"/tasks/{id}"` 的请求内容。

![A978-1-4842-1052-9_9_Fig16_HTML.jpg](img/A978-1-4842-1052-9_9_Fig16_HTML.jpg)

图 9-16. HTTP `GET` 请求 `"/tasks/{id}"`

图 9-17 展示了 RESTful API 服务器对 HTTP `GET` 请求 `"/tasks{id}"` 的响应结果。

![A978-1-4842-1052-9_9_Fig17_HTML.jpg](img/A978-1-4842-1052-9_9_Fig17_HTML.jpg)

图 9-17. HTTP `GET` 请求 `"/tasks{id}"` 的响应结果

---

#### Notes 资源的处理程序

对 `"/notes"` 资源的操作使用了数据模型 `TaskNote` 来持久化数据对象的值。

以下是为 `Notes` 资源指定的路由：

`noteRouter := mux.NewRouter()`
`noteRouter.HandleFunc("/notes", controllers.CreateNote).Methods("POST")`
`noteRouter.HandleFunc("/notes/{id}", controllers.UpdateNote).Methods("PUT")`
`noteRouter.HandleFunc("/notes/{id}", controllers.GetNoteById).Methods("GET")`
`noteRouter.HandleFunc("/notes", controllers.GetNotes).Methods("GET")`
`noteRouter.HandleFunc("/notes/tasks/{id}", controllers.GetNotesByTask).Methods("GET")`
`noteRouter.HandleFunc("/notes/{id}", controllers.DeleteNote).Methods("DELETE")`
`router.PathPrefix("/notes").Handler(negroni.New(`
    `negroni.HandlerFunc(common.Authorize),`
    `negroni.Wrap(noteRouter),`
`))`

---

##### JSON 资源模型

代码清单 9-21 提供了用于表示与 `Notes` 资源发送和接收消息的 JSON 数据的资源模型：

**代码清单 9-21.**  `resources.go` 中用于处理 `"/notes"` 的 JSON 资源

```go
//JSON 资源模型
type (
    // 用于 Post/Put - /notes
    NoteResource struct {
        Data NoteModel `json:"data"`
    }
    // 用于 Get - /notes
    // 用于 /notes/tasks/id
    NotesResource struct {
        Data []models.TaskNote `json:"data"`
    }
    //TaskNote 的模型
    NoteModel struct {
        TaskId      string `json:"taskid"`
        Description string `json:"description"`
    }
)
```

`Users` 和 `Tasks` 资源的实现已完成。你可以像实现 `"/tasks"` 资源一样，为 `"/notes"` 资源实现 API 操作。

> **注**  
> TaskManager 应用的完整源代码可在 `github.com/shijuvar/go-web/tree/master/taskmanager` 获取。

---

### 使用 `godep` 管理 Go 依赖

在前面的章节中，你完成了一个名为 TaskManager 的 RESTful API 应用，该应用使用了一些第三方包。现在，让我们专注于管理应用的依赖关系，以减少构建应用时的外部构建依赖，从而提高工作效率。

当你使用大量第三方包开发 Go 应用时，依赖管理往往是一件令人头疼的事情。大多数软件开发团队使用版本控制系统来管理和分发源代码。一个合适的依赖管理工具是加速开发和构建过程的重要组成部分。

许多技术栈，如 Ruby 和 Node.js，都提供了具有更好依赖管理功能的包管理系统。这些环境提供了集中式仓库系统来获取外部包，因此为这些技术栈提供依赖管理系统非常容易。Go 没有为其包生态系统提供集中式系统，这是为了像 Go 的大多数特性一样保持简单而设计的。这在简化外部包的使用和分发的同时，也带来了管理外部依赖的一些局限性。默认情况下，Go 不提供任何管理外部依赖的机制。

`godep` 工具通过固定依赖关系，帮助你以可重现的方式构建包。该工具能确保你获得更好的可重复构建体验。

#### 安装 `godep` 工具

要安装 `godep` 工具，请运行以下命令：

`go get github.com/tools/godep`

#### 在 TaskManager 中使用 `godep`

让我们将 `godep` 用于 RESTful API 应用，以管理外部依赖。运行以下命令：

`godep save -r`

`save` 命令会将应用的依赖关系列表保存到 `Godeps` 目录下的 `Godeps.json` 文件中。`Godeps.json` 文件包含了应用依赖关系和 Go 版本的 JSON 表示。它还会将依赖关系的源代码复制到 `Godeps/_workspace` 目录中，该目录反映了 `GOPATH` 的结构。

图 9-18 展示了 `godep save` 命令为管理应用依赖关系所创建的结构。

![A978-1-4842-1052-9_9_Fig18_HTML.jpg](img/A978-1-4842-1052-9_9_Fig18_HTML.jpg)

图 9-18. `Godeps` 目录结构

代码清单 9-22 展示了 `Godeps.json` 文件。

**代码清单 9-22.**  TaskManager 应用的 `Godeps.json` 文件

```json
{
    "ImportPath": "github.com/shijuvar/go-web/taskmanager",
    "GoVersion": "go1.5.1",
    "Deps": [
        {
            "ImportPath": "github.com/codegangsta/negroni",
            "Comment": "v0.1-70-gc7477ad",
            "Rev": "c7477ad8e330bef55bf1ebe300cf8aa67c492d1b"
        },
        {
            "ImportPath": "github.com/dgrijalva/jwt-go",
            "Comment": "v2.2.0-23-g5ca8014",
            "Rev": "5ca80149b9d3f8b863af0e2bb6742e608603bd99"
        },
        {
            "ImportPath": "github.com/gorilla/context",
            "Rev": "215affda49addc4c8ef7e2534915df2c8c35c6cd"
        },
        {
            "ImportPath": "github.com/gorilla/mux",
            "Rev": "8a875a034c69b940914d83ea03d3f1299b4d094b"
        },
        {
            "ImportPath": "golang.org/x/crypto/bcrypt",
            "Rev": "02a186af8b62cb007f392270669b91be5527d39c"
        },
        {
            "ImportPath": "golang.org/x/crypto/blowfish",
            "Rev": "02a186af8b62cb007f392270669b91be5527d39c"
        },
        {
            "ImportPath": "gopkg.in/mgo.v2",
            "Comment": "r2015.10.05-1-g4d04138",
            "Rev": "4d04138ffef2791c479c0c8bbffc30b34081b8d9"
        }
    ]
}
```

你可以在任何时候运行 `godep save` 命令来更新新导入的包。

#### 恢复应用的依赖关系

要在目标机器上恢复应用的依赖关系，请运行以下命令：

`godep restore`

`godep restore` 命令会将 `Godeps/Godeps.json` 中指定的包版本安装到 `$GOPATH` 中，从而修改 `GOPATH` 位置中包的状态。

---

### 使用 Docker 部署 HTTP 服务器

你已经完成了 TaskManager 应用，并使用 `godep` 工具提供了依赖管理基础设施。现在是时候提供部署基础设施，以便将 RESTful API 应用部署到生产服务器上了。你可以在本地服务器和云计算环境中部署 HTTP 服务器。（在第 11 章中，你将学习如何使用 Google Cloud Platform 部署 Go 服务器。）

Linux 容器（[`https://linuxcontainers.org/`](https://linuxcontainers.org/)）正逐渐成为部署和运行应用的首选方式，无论这些应用是在本地服务器还是云计算平台上运行。Docker 是一项革命性的技术，它利用 Linux 容器来构建、交付和运行应用。你可以使用 Docker 在本地服务器和云计算基础设施中运行应用。

在本节中，我们将为 TaskManager 应用编写一个 `Dockerfile`，用于与 Docker 配合，将应用部署到本地服务器和云环境中。`Dockerfile` 是一个文本文档，用于使用 Docker 构建和运行应用。在编写 `Dockerfile` 之前，让我们先简要讨论一下 Docker。

### Docker 简介



