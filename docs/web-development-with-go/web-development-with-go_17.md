# 在 URL 中使用 rpc 后缀：
$ URL='https://app-id.appspot.com/_ah/api/discovery/v1/apis/tasks/v1/rpc'
$ curl -s $URL > tasks.rpc.discovery
```

此处，`"app-id"` 是 App Engine 应用的应用程序 ID。

要为 Android 生成 tasks API 的客户端库，请在终端窗口中运行以下命令：

```
##### 在 URL 中使用 rest 后缀
$ URL='https://app-id.appspot.com/_ah/api/discovery/v1/apis/tasks/v1/rest'
$ curl -s $URL > tasks.rest.discovery
$ endpointscfg.py gen_client_lib java tasks.rest.discovery
```

`endpointscfg.py` 工具从 `go_appengine` 目录（Google App Engine SDK for Go 的目录）运行，以生成用于 Android 应用程序的 Java 库。它应生成一个名为 `tasks.rest.zip` 的 zip 文件。

### 总结

Go 是一种为适配现代硬件和 IT 基础设施而设计的编程语言，用于构建大规模可扩展的应用。在云计算时代，Go 是构建可扩展应用的理想编程语言选择。借助 Google Cloud 平台，Go 开发人员可以通过利用 Google Cloud 平台提供的工具和 API，轻松地在云上构建大规模可扩展的应用。

Google Cloud 提供了 App Engine，这是一种平台即服务（PaaS）产品，允许 Go 开发人员使用 App Engine SDK for Go 和用于各种云服务的 Go API 构建高度可扩展且可靠的 Web 应用。当你构建 App Engine 应用时，无需花时间管理 IT 基础设施。相反，你可以完全专注于构建应用，该应用可在需要增加计算资源时自动扩展。借助 App Engine，开发人员无需再处理系统管理、负载均衡、扩展和服务器维护等问题。

App Engine SDK for Go 提供了 `goapp` 工具，允许你在开发 Web 服务器中测试 App Engine 应用。`goapp` 工具还可用于将 App Engine Web 应用部署到在 App Engine 沙箱环境中运行的 Google Cloud 生产环境。

App Engine 环境提供了一个 `main` 包，因此你不能使用名为 `main` 的包。相反，你可以在你选择的包中编写 HTTP 处理程序逻辑。App Engine 应用可以轻松访问 Google Cloud 平台提供的许多云服务。当你在 App Engine 应用中利用这些云服务时，无需配置这些服务以与 App Engine 一起使用，因为你可以通过使用相应的 Go API 直接从应用中访问它们。



Google Cloud 平台提供了多种托管服务用于持久化存储结构化数据，包括 `Google Cloud SQL`、`Google Cloud Datastore` 和 `Google Bigtable`。当构建需要处理大量数据的 `App Engine` 应用时，`Google Cloud Datastore` 是最佳的数据库选择。你可以使用 `App Engine` 和 `Cloud Datastore` 构建具有大规模扩展能力的 Web 应用，而无需花费时间管理 IT 基础设施。

`Google Cloud Endpoints` 是一项 `App Engine` 服务，用于为移动客户端应用和 Web 客户端应用构建后端 API。它提供了快速构建后端 API 所需的工具、库和功能，并能从 `App Engine` 应用中生成原生客户端库。虽然你可以使用常规的 `App Engine` 应用创建后端 API，但 `Cloud Endpoints` 通过提供额外功能简化了开发过程。`Cloud Endpoints` 允许你为 iOS、Android、JavaScript 和 Dart 生成原生客户端库，从而使客户端应用开发者无需编写用于访问后端 API 的包装器。

### 参考

[`https://github.com/GoogleCloudPlatform/gcloud-golang`](https://github.com/GoogleCloudPlatform/gcloud-golang)

[`https://cloud.google.com`](https://cloud.google.com/)

Google Cloud 服务信息图：[`https://cloud.google.com/`](https://cloud.google.com/)

Endpoints 架构图：[`https://cloud.google.com/appengine/docs/python/endpoints/`](https://cloud.google.com/appengine/docs/python/endpoints/)



\[content\]

# 索引 A

#### Alice 包

#### App Engine 应用配置文件
`goapp deploy` 命令  
Google Developers Console 项目创建  
项目详情  
HTTP 服务器  
任务表单  
测试

#### 数组

#### 身份验证
基于 API 的方式与授权  
基于 Cookie 的方式  
JWT（参见 JSON Web 令牌 (JWT)）  
社交身份  
基于令牌的方式  
用户凭据

### B

#### 行为驱动开发 (BDD)
定义  
Ginkgo（参见 Ginkgo，BDD 风格测试）  
TDD

#### 空白标识符

## BSON

## 二进制 JSON (BSON)

#### 缓冲通道

### C

#### CaaS
参见 容器即服务 (CaaS)

#### 云计算
优势  
自动扩缩容能力  
CaaS  
Google Cloud 平台服务  
托管和运行应用程序  
IaaS  
PaaS  
基于服务的消费模型

#### 并发
描述  
Go 语言  
goroutine

#### 容器即服务 (CaaS)

#### 基于 Cookie 的身份验证

#### CRUD 操作，MongoDB
`Find` 方法  
处理函数  
`Insert` 方法  
嵌入式文档  
映射对象和文档切片  
结构体值  
查询对象，文档  
`Remove` 方法  
单条记录  
`Sort` 方法  
`Update` 方法

#### 自定义处理器
实现  
`messageHandler` 类型

### D

#### 数据集合
数组  
映射  
切片（参见 切片）

#### DefaultServeMux

#### Docker
`Dockerfile`，TaskManager 应用程序  
引擎标志和命令  
Hub  
Linux 容器

### E

#### 嵌入类型
方法，覆盖

#### 错误处理

### F

#### 全功能 Web 框架

#### 函数
`defer`  
`panic`  
`recover`

### G

#### GAE
参见 Google App Engine (GAE)

#### Ginkgo，BDD 风格测试
引导程序，套件文件  
Gomega  
安装  
HTTP API 服务器目录结构，重构后的应用程序  
`lib` 包  
`main` 包  
规格容器  
`FakeUserRepository`  
HTTP 处理函数  
运行测试套件  
`UserRepository` 接口  
`users_tests.go`，`lib_test` 包

#### Godeps
依赖管理系统  
安装  
`restore` 命令  
TaskManager 应用程序

#### `godoc` 工具

#### Go 文档

#### Go 生态系统

#### Go 工具
语言  
库

#### `GOMAXPROCS`

#### Go Mobile 项目

#### Google App Engine (GAE)
Cloud Bigtable  
Cloud Datastore  
Google Cloud SQL  
Go SDK  
日志服务  
Memcache  
PaaS  
“沙箱”环境  
搜索服务  
安全扫描  
任务队列服务  
流量拆分  
用户认证服务

#### Google Cloud Bigtable

#### Google Cloud Datastore
App Engine 应用程序  
数据，Google Developers Console  
任务表单  
任务列表页面  
实体  
特性  
Go 包  
键名  
`PendingKey`  
查询  
保存应用程序处理器

#### Google Cloud Endpoints
架构  
后端 API  
`addTask` 操作  
APIs Explorer  
应用程序数据模型  
HTTP 请求与响应  
`listTasks` 操作  
结构体类型  
任务 API 操作  
`TaskService` 暴露方法  
`TaskService` 注册，HTTP 服务器  
客户端库  
`endpoints` 包安装

#### Google Cloud 平台服务

#### Google Cloud SQL

#### Go 编程语言
优势  
`calc` 包  
代码复用性  
编译  
描述  
特性  
通道  
并发  
极简语言  
OOP  
静态类型编程语言

#### Go Playground
goroutine  
Hello World 程序

#### Go Mobile 项目
共享库  
程序测试

## Go 代码

#### Gorilla 处理器

#### Gorilla Web 工具包

#### Goroutine

#### Go 工具命令
`fmt` 命令  
`godoc` 工具  
安装二进制发行版  
安装验证命令  
Mac OS  
Go 工作区  
代码组织  
路径  
`GOPATH` 环境变量  
包  
子目录

### H

#### 处理器
CRUD 操作  
定义  
`ServeHTTP` 方法  
写入响应头和响应体

#### `html/template` 包
添加页面数据结构与数据存储  
编辑页面  
文件夹结构，Web 应用程序  
辅助函数  
首页  
`main` 函数  
脚本注入  
视图与模板定义文件

#### HTTP 应用程序
`ResponseRecorder`  
HTTP API 服务器  
`NewRecorder` 函数  
`ServeHTTP` 方法  
TDD  
`TestGetUsers` 服务器  
HTTP API 服务器  
`httptest.NewServer` 函数  
`TestCreateUserClient`  
`TestGetUsersClient`  
`http.HandlerFunc` 类型

#### HTTP 中间件组件
控制流  
Gorilla 上下文  
日志记录  
Negroni（参见 Negroni）  
场景  
第三方库  
Alice 包  
Gorilla 处理器  
编写日志记录模式  
步骤  
`StripPrefix` 函数

#### HTTP 请求
处理器  
请求-响应范式  
`ServeMux` 多路复用器  
`http.Server` 结构体

#### 混合独立/App Engine 应用程序
App Engine SDK  
目录结构  
Go 工具 `hybridapp`  
`lib` 任务包

### I

#### 基础设施即服务 (IaaS) 模型

#### 接口
组合与方法覆盖  
具体实现  
包含 `PrintName` 和 `PrintDetails` 方法的示例程序  
已定义的类型

### J, K

#### JSON API
操作  
数据持久化  
错误处理  
处理函数  
HTTP 请求生命周期  
登录资源  
笔记资源  
`Register` 处理函数  
资源模型  
RESTful API  
`taskController.go` 源文件  
`taskRepository.go`  
任务资源

#### JSON Web 令牌 (JWT)
API 服务器  
`DisplayAppError` 函数  
编码和解码的安全令牌  
生成与验证  
头部与载荷部分  
HTML5 Web 存储 / Web Cookie  
HTTP 中间件  
JSON 对象  
`jwt-go` 包  
中间件函数  
`ParseFromRequest` 函数  
运行与测试，API 服务器  
验证

### L

#### `ListenAndServe` 签名

### M

#### 映射，数据集合

#### 微服务架构

#### MongoDB
BSON  
集合  
`createDbSession` 函数  
CRUD 操作（参见 CRUD 操作，MongoDB）  
`GetSession`  
索引  
`mgo` 驱动  
久经考验的库  
连接  
安装  
NoSQL 数据库  
`Session` 对象  
`DataStore` 结构体类型  
HTTP 服务器  
Web 应用程序  
`TaskNote` 集合

#### 单体架构方法

#### 多路复用器配置

### N

#### Negroni
定义  
安装  
特定路由的中间件函数  
`negroni.Handler` 接口  
路由栈中间件  
`net/http` 包  
可组合性与可扩展性  
全功能 Web 应用程序  
标准库

### O

#### OAuth 2
移动和 Web 应用程序  
社交身份提供商  
Twitter 和 Facebook

#### 面向对象编程 (OOP) 语言

### P, Q

#### PaaS
参见 平台即服务 (PaaS) 模型

#### 包
别名  
空白标识符  
可执行程序  
`GOPATH` 目录  
`GOROOT` 目录  
`import`  
`init` 函数  
库包  
`main` 函数  
共享库  
`strconv`  
第三方，安装

#### 并行性

#### 平台即服务 (PaaS) 模型

#### 指针方法接收器
`&`（取地址）运算符  
调用方法  
`ChangeLocation` 函数  
描述  
包含 `PrintDetails` 方法的 `person` 结构体  
`PrintName` 方法

### R

#### RESTful API
配置值  
数据模型  
数字化转型  
`Dockerfile`  
前端/后端应用程序  
Godep  
JSON  
JWT 身份验证（参见 JSON Web 令牌 (JWT)）  
微服务架构  
MongoDB Session 对象（参见 MongoDB）  
私钥/公钥 RSA 密钥  
资源建模  
`routers` 包目录  
TaskNote 资源  
任务资源  
用户资源  
设置栈  
`StartUp` 函数  
TaskManager 应用程序结构  
第三方包  
URI  
Web 和移动应用程序  
XML

### S

#### `ServeMux.HandleFunc` 函数

#### 单页应用 (SPA) 架构

#### 切片
函数  
遍历切片  
长度与容量  
`make` 函数  
nil 切片

#### 静态类型语言

#### 静态 Web 服务器
访问  
`FileServer` 函数  
文件夹结构  
`ListenAndServe` 函数  
`ServeMux.Handle` 函数

#### 结构体
另请参见 指针方法接收器  
调用方法  
类  
声明，包含一组字段  
字段规范  
实例  
结构体字面量  
具有行为的类型

### T

#### 测试驱动开发 (TDD)

## 文本/模板包
另请参见 `html/template` 包  
集合对象  
数据结构定义  
管道  
结构体字段  
变量声明

#### 第三方包安装

#### 基于令牌的身份验证

#### 类型组合
优势，继承  
嵌入  
示例程序

### U

#### 无缓冲通道

#### 统一资源标识符 (URI)

#### 单元测试
BDD（参见 行为驱动开发 (BDD)）  
基准测试  
覆盖率标志  
定义  
`Parallel` 方法  
`Reverse` 和 `SwapCase` 函数  
独立包  
`Skip` 方法  
软件开发  
字符串工具函数  
TDD  
第三方包  
Web 应用程序（参见 HTTP 应用程序）

## URI
参见 统一资源标识符 (URI)

### W, X, Y, Z

#### Web 与微服务
HTTP 包  
单体应用  
RESTful API  
SPA 架构

\[/content\]