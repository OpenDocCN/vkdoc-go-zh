# 6. HTTP 中间件

最后两章探讨了构建 Web 应用程序和 Web API 的各个方面。本章将介绍 HTTP 中间件，它在构建真实世界的 Web 应用程序时可以简化开发工作。Go 开发者社区在构建 Web 应用程序时，对于采用完整的 Web 应用程序框架的兴趣并不浓厚。相反，他们更倾向于使用诸如基础构建块之类的标准库包，以及一些必要的第三方库，例如 Gorilla `mux`。编写和使用 HTTP 中间件是坚持这种策略的一种基本方法。你可以将许多横切关注点，例如安全性、HTTP 请求和响应日志记录、压缩 HTTP 响应以及缓存，实现为中间件组件；并且这些中间件组件可以应用于许多应用程序处理器或跨应用程序的处理器。



### HTTP 中间件简介

在构建 Web 应用程序时，您可能需要为部分或所有 HTTP 请求处理程序执行一些共享功能。例如，假设您想将所有 HTTP 请求记录到 Web 服务器；为每个 HTTP 请求处理程序实现日志记录逻辑将是一项繁琐的工作。最好使用特殊类型的处理程序来实现共享行为，并将其应用于部分或所有应用程序处理程序。通过使用 HTTP 中间件，您可以在 Web 应用程序中实现此类功能。使用 HTTP 中间件处理程序是在应用程序中实现共享行为的一种绝佳方式。

中间件是一段可插拔且自包含的代码，它包装一个 Web 应用程序。它可以用来在 HTTP 请求处理程序中实现共享行为。中间件组件可以插入到应用程序中，作为请求处理周期中的另一个层，它可以在 HTTP 请求处理程序之前或之后执行某些逻辑。例如，要记录所有 HTTP 请求，您可以编写日志记录中间件，将其装饰到请求处理程序中，从而为每个发往 Web 服务器的 HTTP 请求应用日志记录功能。由于中间件是独立的代码段，它不会影响您的应用程序逻辑。您可以在需要时将中间件组件装饰到 HTTP 请求处理程序中，也可以在不需要时将其移除。

以下是您可以使用中间件的一些示例场景：

- 记录 HTTP 请求和响应
- 压缩 HTTP 响应
- 编写通用的响应头
- 创建数据库会话对象
- 实现安全性和验证身份凭证

### 编写 HTTP 中间件

Go 标准库包 `net/http` 提供了诸如 `StripPrefix` 和 `TimeoutHandler` 之类的函数，它们类似于中间件：它们包装请求处理程序，并在请求处理周期中提供额外的实现。`StripPrefix` 和 `TimeoutHandler` 都将 `http.Handler` 以及其他参数作为参数，并返回一个 `http.Handler`，以便您可以轻松地将其包装到正常的处理程序中，以执行一些额外的逻辑。`StripPrefix` 函数接受一个字符串前缀和一个 `http.Handler` 作为参数，并返回一个处理程序，该处理程序通过从请求 URL 的路径中移除给定前缀来服务 HTTP 请求（参见列表 6-1）。

列表 6-1. 使用 `StripPrefix` 包装 `http.FileServer` 处理程序

```
package main

import (
    "net/http"
)

func main() {
    // 要以备选 URL 路径 (/public/) 提供磁盘目录 (/public) 中的服务，
    // 请使用 StripPrefix 在 FileServer 看到请求 URL
    // 的路径之前修改它：
    fs := http.FileServer(http.Dir("public"))
    http.Handle("/public/", http.StripPrefix("/public/", fs))
}
```

`StripPrefix` 函数包装了 `http.FileServer` 处理程序，并提供了额外的功能，因为它通过从请求 URL 的路径中移除给定前缀并调用给定的处理程序对象来修改请求 URL 的路径。

列表 6-2 是 `StripPrefix` 函数的 Go 源代码。

列表 6-2. 来自 `net/http` 库的 `StripPrefix` 源码

```
func StripPrefix(prefix string, h Handler) Handler {
    if prefix == "" {
        return h
    }
    return HandlerFunc(func(w ResponseWriter, r *Request) {
        if p := strings.TrimPrefix(r.URL.Path, prefix); len(p) < len(r.URL.Path) {
            r.URL.Path = p
            h.ServeHTTP(w, r)
        } else {
            NotFound(w, r)
        }
    })
}
```

`StripPrefix` 函数通过将一个签名如 `func(w ResponseWriter, r *Request)` 的匿名函数转换为 `HandlerFunc` 类型，来返回一个 `http.Handler` 对象。当您编写包装处理程序函数时，您可以在正常处理程序函数之前或之后执行额外的逻辑。在 `StripPrefix` 函数中，`http` 库在执行正常处理程序之前执行逻辑。该函数通过调用 `ServeHTTP` 方法来调用给定的处理程序（应用程序处理程序或另一个包装处理程序）：

```
if p := strings.TrimPrefix(r.URL.Path, prefix); len(p) < len(r.URL.Path) {
    r.URL.Path = p
    h.ServeHTTP(w, r)
} else {
    NotFound(w, r)
}
```

您可以像 `http` 包实现包装处理程序函数一样轻松地编写 HTTP 中间件。您可以通过实现签名如 `func(http.Handler) http.Handler` 的函数来编写中间件函数。如果您想向中间件函数传递任何值，您可以像 `http` 包中的 `StripPrefix` 函数那样，将它们作为函数参数与 `http.Handler` 一起提供。

#### 如何编写 HTTP 中间件

编写 HTTP 中间件的基本步骤如下：

- 编写一个函数，以 `http.Handler` 作为函数参数，这样您就可以将其他中间件处理程序以及正常的应用程序处理程序作为函数参数传入。您可以通过在中间件函数内调用 `ServeHTTP` 方法来调用这些处理程序函数。
- 从中间件函数返回 `http.Handler`，以便与其他中间件处理程序链接，并与正常的应用程序处理程序一起包装。由于中间件函数返回 `http.Handler`，您可以将其注册到 `ServeMux` 对象中。

列表 6-3 展示了编写 HTTP 中间件的模式。

列表 6-3. 编写中间件的模式

```
func middlewareHandler(next http.Handler) http.Handler {
  return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    // 在执行应用程序处理程序之前，在此处放置中间件逻辑
    next.ServeHTTP(w, r)
    // 在执行应用程序处理程序之后，在此处放置中间件逻辑
  })
}
```



#### 编写日志记录中间件

让我们编写一个 HTTP 中间件来记录所有 HTTP 请求，这样该行为就可以应用于多个应用程序处理器。

清单 6-4 是一个日志记录中间件的示例。

**清单 6-4.** 日志记录中间件

```
package main

import (
	"fmt"
	"log"
	"net/http"
	"time"
)

func loggingHandler(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		start := time.Now()
		log.Printf("Started %s %s", r.Method, r.URL.Path)
		next.ServeHTTP(w, r)
		log.Printf("Completed %s in %v", r.URL.Path, time.Since(start))
	})
}

func index(w http.ResponseWriter, r *http.Request) {
	log.Println("Executing index handler")
	fmt.Fprintf(w, "Welcome!")
}

func about(w http.ResponseWriter, r *http.Request) {
	log.Println("Executing about handler")
	fmt.Fprintf(w, "Go Middleware")
}

func iconHandler(w http.ResponseWriter, r *http.Request) {
}

func main() {
	http.HandleFunc("/favicon.ico", iconHandler)
	indexHandler := http.HandlerFunc(index)
	aboutHandler := http.HandlerFunc(about)
	http.Handle("/", loggingHandler(indexHandler))
	http.Handle("/about", loggingHandler(aboutHandler))
	server := &http.Server{
		Addr: ":8080",
	}
	log.Println("Listening...")
	server.ListenAndServe()
}
```

运行这个程序，并向`"/"`和`"/about"`发起请求。你应该会得到类似如下的日志输出：

```
2015/04/25 19:40:10 Started GET /
2015/04/25 19:40:10 Executing index handler
2015/04/25 19:40:10 Completed / in 0
2015/04/25 19:40:18 Started GET /about
2015/04/25 19:40:18 Executing about handler
2015/04/25 19:40:18 Completed /about in 1.0005ms
```

在前面的程序中，`loggingHandler`是 HTTP 中间件，它装饰了`"/"`和`"/about"`路由对应的应用程序处理器。中间件允许你将共享的行为功能复用到多个处理器上。在日志记录中间件处理器中，日志消息会在执行应用程序处理器之前和之后写入。该中间件函数在调用应用程序处理器之前记录请求的 HTTP 方法和 URL 路径，并在执行应用程序处理器之后记录执行所花费的时间。通过调用`next.ServeHTTP(w, r)`，中间件函数可以执行应用程序处理器函数：

```
func loggingHandler(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		start := time.Now()
		log.Printf("Started %s %s", r.Method, r.URL.Path)
		next.ServeHTTP(w, r)
		log.Printf("Completed %s in %v", r.URL.Path, time.Since(start))
	})
}
```

应用程序处理器函数被转换为`HandlerFunc`函数类型，并传递给日志记录中间件处理器函数：

```
indexHandler := http.HandlerFunc(index)
aboutHandler := http.HandlerFunc(about)

http.Handle("/", loggingHandler(indexHandler))
http.Handle("/about", loggingHandler(aboutHandler))
```

#### 控制 HTTP 中间件的执行流

由于 HTTP 中间件处理器函数将`http.Handler`作为函数参数并返回一个`http.Handler`，你可以轻松地与其他中间件处理器链式调用，并最终调用应用程序处理器。当你与其他中间件处理器链式调用时，理解中间件处理器的控制流至关重要。让我们编写几个中间件处理器，并将它们封装到应用程序处理器中。

清单 6-5 展示了中间件处理器的控制流。

**清单 6-5.** 中间件处理器的控制流

```
package main

import (
"fmt"
"log"
"net/http"
)

func middlewareFirst(next http.Handler) http.Handler {
return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
log.Println("MiddlewareFirst - Before Handler")
next.ServeHTTP(w, r)
log.Println("MiddlewareFirst - After Handler")
})
}

func middlewareSecond(next http.Handler) http.Handler {
return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
log.Println("MiddlewareSecond - Before Handler")
if r.URL.Path == "/message" {
if r.URL.Query().Get("password") == "pass123" {
log.Println("Authorized to the system")
next.ServeHTTP(w, r)
} else {
log.Println("Failed to authorize to the system")
return
}
} else {
next.ServeHTTP(w, r)
}
log.Println("MiddlewareSecond - After Handler")
})
}

func index(w http.ResponseWriter, r *http.Request) {
log.Println("Executing index Handler")
fmt.Fprintf(w, "Welcome")
}

func message(w http.ResponseWriter, r *http.Request) {
log.Println("Executing message Handler")
fmt.Fprintf(w, "HTTP Middleware is awesome")
}

func iconHandler(w http.ResponseWriter, r *http.Request) {
}

func main() {
http.HandleFunc("/favicon.ico", iconHandler)
http.Handle("/", middlewareFirst(middlewareSecond(http.HandlerFunc(index))))
http.Handle("/message", middlewareFirst(middlewareSecond(http.HandlerFunc(message))))
server := &http.Server{
Addr: ":8080",
}
log.Println("Listening...")
server.ListenAndServe()
}
```

运行这个程序，并向`"http://localhost:8080/"`发起请求。你应该会得到以下输出：

- `2015/04/30 11:10:53 MiddlewareFirst - Before Handler`
- `2015/04/30 11:10:53 MiddlewareSecond - Before Handler`
- `2015/04/30 11:10:53 Executing index Handler`
- `2015/04/30 11:10:53 MiddlewareSecond - After Handler`
- `2015/04/30 11:10:53 MiddlewareFirst - After Handler`

当你向`"http://localhost:8080/message"`发起请求，但提供了错误的查询字符串变量`password`值（比如提供`"http://localhost:8080/message?password=wrongpass"`），你应该会得到以下输出：

- `2015/04/30 11:11:06 MiddlewareFirst - Before Handler`
- `2015/04/30 11:11:06 MiddlewareSecond - Before Handler`
- `2015/04/30 11:11:06 Failed to authorize to the system`
- `2015/04/30 11:11:06 MiddlewareFirst - After Handler`

当你向`"http://localhost:8080/message"`发起请求，并提供了正确的查询字符串变量`password`值（`"http://localhost:8080/message?password=pass123"`），你应该会得到以下输出：

- `2015/04/30 11:11:35 MiddlewareFirst - Before Handler`
- `2015/04/30 11:11:35 MiddlewareSecond - Before Handler`
- `2015/04/30 11:11:35 Authorized to the system`
- `2015/04/30 11:11:35 Executing message Handler`
- `2015/04/30 11:11:35 MiddlewareSecond - After Handler`
- `2015/04/30 11:11:35 MiddlewareFirst - After Handler`

通过查看程序生成的日志消息，你可以轻松理解中间件处理器的控制流。这里，`middlewareFirst`和`middlewareSecond`被调用作为包装处理器；你可以将其应用于应用程序处理器。在中间件函数`middlewareSecond`中，如果请求的 URL 路径是`"/message"`，则会验证查询字符串变量`password`的值。



以下是程序运行并处理对 `"/"` 和 `"/message"` 路由的请求时的控制流程：

控制流向首先进入 `middlewareFirst` 中间件函数。当 `middlewareFirst` 函数中输出一条日志（在调用下一个处理器之前）后，控制流便进入 `middlewareSecond` 中间件函数，并调用 `next.ServeHTTP(w, r)`。当 `middlewareSecond` 函数中又输出一条日志（在调用下一个处理器之前）后，控制流再次调用 `next.ServeHTTP(w, r)` 进入应用处理器：如果请求的 URL 路径是 `"/"`，则直接调用索引应用处理器，无需任何授权。如果请求的 URL 路径是 `"/message"`，则中间件函数会使用查询字符串变量 `password` 来验证请求。如果你没有为查询字符串变量 `password` 提供值 `"pass123"`，控制流会从中间件函数返回，回到 `middlewareFirst` 函数，执行 `next.ServeHTTP(w, r)` 代码块之后的逻辑，然后结束请求处理周期，而不会执行应用处理器。如果请求验证通过，控制流会调用 `next.ServeHTTP(w, r)` 进入应用处理器 `message`。从 `middlewareSecond` 函数调用应用处理器（如果请求验证通过）后，控制流会回到 `middlewareSecond` 函数，并执行 `next.ServeHTTP(w, r)` 代码块之后的逻辑。从 `middlewareSecond` 处理器返回后，控制流会回到 `middlewareFirst` 函数，并执行 `next.ServeHTTP(w, r)` 代码块之后的逻辑。

你可以随时从中间件处理器链中退出，就像 `middlewareSecond` 处理器在请求无效时所做的那样。在这种情况下，如果请求处理周期中存在任何处理器，控制流会回到上一个处理器。当请求 `"/message"` 而未提供任何有效的查询字符串值时，你从中间件处理器返回，应用处理器将不会被调用。最重要的是，你可以在调用中间件处理器函数之前或之后执行某些逻辑。

### 使用第三方中间件

使用中间件是在多个应用间实现可复用代码片段的一个绝佳方式。许多第三方库提供了各种可复用的中间件组件，可用于实现身份验证、日志记录、压缩响应等常见功能。在用 Go 开发 Web 应用时，你可以利用这些第三方库将许多通用功能集成到你的应用程序中。

##### 使用 Gorilla Handlers

Gorilla Web 工具包（[`www.gorillatoolkit.org/`](http://www.gorillatoolkit.org/)）提供了与 Go 的 `net/http` 包一起使用的一系列处理器。让我们编写一个程序来使用 Gorilla 的 `LoggingHandler` 和 `CompressHandler` 来记录 HTTP 请求和压缩 HTTP 响应。

##### 安装 Gorilla Handlers

要安装 Gorilla handlers，请在终端中运行以下命令：

`$ go get github.com/gorilla/handlers`

##### 使用 Gorilla Handlers

要使用 Gorilla handlers，你必须将 `github.com/gorilla/handlers` 添加到导入列表中：

`import "github.com/gorilla/handlers"`

清单 6-6 展示了 Gorilla 的日志记录和压缩处理器。

**清单 6-6. 使用 Gorilla 的日志记录和压缩处理器**

```
package main

import (
    "fmt"
    "log"
    "net/http"
    "os"
    "github.com/gorilla/handlers"
)

func index(w http.ResponseWriter, r *http.Request) {
    log.Println("Executing index handler")
    fmt.Fprintf(w, "Welcome!")
}

func about(w http.ResponseWriter, r *http.Request) {
    log.Println("Executing about handler")
    fmt.Fprintf(w, "Go Middleware")
}

func iconHandler(w http.ResponseWriter, r *http.Request) {
}

func main() {
    http.HandleFunc("/favicon.ico", iconHandler)
    indexHandler := http.HandlerFunc(index)
    aboutHandler := http.HandlerFunc(about)
    logFile, err := os.OpenFile("server.log", os.O_WRONLY|os.O_CREATE|os.O_APPEND, 0666)
    if err != nil {
        panic(err)
    }
    http.Handle("/", handlers.LoggingHandler(logFile, handlers.CompressHandler(indexHandler)))
    http.Handle("/about", handlers.LoggingHandler(logFile, handlers.CompressHandler(aboutHandler)))
    server := &http.Server{
        Addr: ":8080",
    }
    log.Println("Listening...")
    server.ListenAndServe()
}
```

与清单 6-5 类似，处理器链由 `LoggingHandler` 和 `CompressHandler` 构成，然后封装到应用处理器中。该程序负责记录请求日志，并使用 gzip 或 Deflate 压缩响应。日志文件指定为 `"server.log"`，因此你可以在此文件中查看日志。运行程序并向 `"/"` 和 `"/about"` 发送请求。你应该会在 `"server.log"` 文件中看到以下日志：

* `::1 - - [26/Apr/2015:16:42:41 +0530] "GET / HTTP/1.1" 200 32`
* `::1 - - [26/Apr/2015:16:42:45 +0530] "GET /about HTTP/1.1" 200 37`

你可以看到日志信息还包含了 HTTP 状态码。

#### 使用 Alice 包进行中间件链式调用

第三方 `Alice` 库包（[`https://github.com/justinas/alice`](https://github.com/justinas/alice)）提供了一种便捷的方式来链式调用 HTTP 中间件函数和应用处理器。清单 6-5 中的程序如下链式调用 HTTP 中间件函数：

`http.Handle("/", middlewareFirst(middlewareSecond(http.HandlerFunc(index))))`

通过使用 `Alice` 包，你可以将上述处理器链转换为以下形式：

`http.Handle("/",alice.New(middlewareFirst, middlewareSecond).ThenFunc(http.HandlerFunc(index)))`

这是一种优雅的链式调用中间件函数并将其与应用处理器封装在一起的方法。

##### 安装 Alice

要安装 `Alice`，请在终端中运行以下命令：

`$ go get github.com/justinas/alice`



## 使用 Alice 包

要使用 `Alice` 包，需要将 `github.com/justinas/alice` 添加到导入列表中：

```
import "github.com/justinas/alice"
```

下面我们用 `Alice` 包重写清单 6-6 中的程序（见清单 6-7）。

**清单 6-7.** 使用 Alice 包链接中间件函数

```
package main

import (
    "io"
    "log"
    "net/http"
    "os"
    "github.com/gorilla/handlers"
    "github.com/justinas/alice"
)

func loggingHandler(next http.Handler) http.Handler {
    logFile, err := os.OpenFile("server.log", os.O_WRONLY|os.O_CREATE|os.O_APPEND, 0777)
    if err != nil {
        panic(err)
    }
    return handlers.LoggingHandler(logFile, next)
}

func index(w http.ResponseWriter, r *http.Request) {
    w.Header().Set(
        "Content-Type",
        "text/html",
    )
    io.WriteString(
        w,
        `<doctype html>
<html>
<head>
<title>Index</title>
</head>
<body>
Hello Gopher!
</body>
</html>`,
    )
}

func about(w http.ResponseWriter, r *http.Request) {
    w.Header().Set(
        "Content-Type",
        "text/html",
    )
    io.WriteString(
        w,
        `<doctype html>
<html>
<head>
<title>About</title>
</head>
<body>
Go Web development with HTTP Middleware
</body>
</html>`,
    )
}

func iconHandler(w http.ResponseWriter, r *http.Request) {
    http.ServeFile(w, r, "./favicon.ico")
}

func main() {
    http.HandleFunc("/favicon.ico", iconHandler)
    indexHandler := http.HandlerFunc(index)
    aboutHandler := http.HandlerFunc(about)
    commonHandlers := alice.New(loggingHandler, handlers.CompressHandler)
    http.Handle("/", commonHandlers.ThenFunc(indexHandler))
    http.Handle("/about", commonHandlers.ThenFunc(aboutHandler))
    server := &http.Server{
        Addr: ":8080",
    }
    log.Println("Listening...")
    server.ListenAndServe()
}
```

在这个程序中，使用了两个 Gorilla 处理器来记录请求和压缩响应：`LoggingHandler` 和 `CompressHandler`。响应中包含了 HTML 字符串，用于验证 HTTP 响应的压缩效果。运行程序，向 `"/"` 和 `"/about"` 发送请求，并查看日志文件和 HTTP 响应。

图 6-1 展示了 Fiddler（一个 HTTP 调试工具）中的截图，显示 Web 响应正在使用 gzip 编码进行压缩。

![A978-1-4842-1052-9_6_Fig1_HTML.jpg](img/A978-1-4842-1052-9_6_Fig1_HTML.jpg)

**图 6-1.** 使用 Gorilla 处理器压缩 HTTP 响应

使用 `Alice` 包可以优雅地链接中间件函数，使代码块易于理解。Gorilla 的 `LoggingHandler` 并不具有 `func (http.Handler) http.Handler` 签名，它除了 `http.Handler` 之外，还接受一个日志文件作为函数参数。因此，通过正确使用 `Alice` 包并在多个地方使用该处理器，你可以编写一个用于日志记录的中间件函数，在该函数中通过提供日志文件作为参数来调用 Gorilla 的 `LoggingHandler`。当需要一个带有多个函数参数的中间件函数时，这是一种很好的方法：

```
func loggingHandler(next http.Handler) http.Handler {
    logFile, err := os.OpenFile("server.log", os.O_WRONLY|os.O_CREATE|os.O_APPEND, 0777)
    if err != nil {
        panic(err)
    }
    return handlers.LoggingHandler(logFile, next)
}
```

`loggingHandler` 中间件函数可以与 `Alice` 一起使用，它会用必要的参数调用 Gorilla 的 `LoggingHandler` 函数。当使用中间件处理器时，你可能需要将多个中间件处理器装饰到多个应用程序处理器中。在这种情况下，你可以通过组合多个中间件处理器，使用 `Alice` 创建共同的处理器，并将其应用于多个应用程序处理器。

这里将 `commonHandlers` 定义为用于多个应用程序处理器的共同处理器：

```
indexHandler := http.HandlerFunc(index)
aboutHandler := http.HandlerFunc(about)
commonHandlers := alice.New(loggingHandler, handlers.CompressHandler)
http.Handle("/", commonHandlers.ThenFunc(indexHandler))
http.Handle("/about", commonHandlers.ThenFunc(aboutHandler))
```

`Alice` 包提供了一个流畅的 API 来处理中间件函数，使你可以优雅地链接中间件函数。`Alice` 包是一个非常轻量级的库，代码量不足 100 行。

### 使用 Negroni 包处理中间件

当你直接使用 `net/http` 包构建 Web 应用程序时，第三方库 `Negroni` 是一个很好的伴侣。`Negroni` 是一个设计为与 `net/http` 包兼容的库。它提供了一种在 Go 中使用 HTTP 中间件的惯用方法。它鼓励你使用 `net/http` 处理器，同时提供了一种处理 HTTP 中间件函数的方式。

上一节讨论了 `Alice` 包，它非常适合链接中间件处理器。`Negroni` 提供了一种不同的方法，以简单且非侵入的方式处理 HTTP 中间件函数。如果你更喜欢使用 `Alice` 来处理中间件函数，可以继续使用它，因为 `Alice` 和 `Negroni` 都用略有不同的方式解决了相同的问题。`Negroni` 提供了一个功能完备的库来处理中间件函数，其中还附带了一些常见的中间件函数，用于记录请求、压缩响应、提供静态文件服务器以及从恐慌中恢复。

`Negroni` 允许你在全局级别配置中间件函数，以便与所有应用程序处理器一起使用。你也可以配置中间件函数以用于特定的处理器。当你对所有应用程序处理器使用中间件函数时，无需为每个应用程序处理器单独配置。相反，你只需使用 `Negroni` 配置该中间件函数，`Negroni` 会将配置的中间件函数包装到所有应用程序处理器中。你也可以配置一些中间件函数，使其仅与某些特定的应用程序处理器一起执行。

#### Negroni 入门

`Negroni` 为使用中间件处理器提供了一个简单的编程模型。当你使用它时，还可以使用其默认的中间件函数。让我们安装并编写一个简单的程序，开始使用 `Negroni` 包。



#### 安装 Negroni

要安装 `Negroni`，请在终端中执行以下命令：

`$ go get github.com/codegangsta/negroni`

若要使用 `Negroni` 包，需要将 `github.com/codegangsta/negroni` 包添加到导入列表中：

`import "github.com/codegangsta/negroni"`

清单 6-8 是一个使用 `Negroni` 的示例程序：

**清单 6-8.** 使用 Negroni 的简单 HTTP 服务器

```
package main

import (
    "fmt"
    "net/http"
    "github.com/codegangsta/negroni"
)

func index(w http.ResponseWriter, req *http.Request) {
    fmt.Fprintf(w, "Welcome!")
}

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/", index)
    n := negroni.Classic()
    n.UseHandler(mux)
    n.Run(":8080")
}
```

在这个程序中，你没有使用自己的中间件函数；你只是用 `Negroni` 的默认中间件函数运行了一个 HTTP 服务器。通过调用 `negroni.Classic` 函数来创建一个 `Negroni` 实例。

当使用 `Classic` 函数创建 `Negroni` 实例时，它会提供以下内置中间件函数，这些函数对于大多数 Web 应用程序非常有用：

*   `negroni.Recovery`：恐慌恢复中间件
*   `negroni.Logging`：请求/响应日志记录中间件
*   `negroni.Static`：在 `"public"` 目录下提供静态文件服务

`Negroni` 实例也可以通过调用 `negroni.New` 函数来创建，该函数会返回一个没有预配置任何中间件的新的 `Negroni` 实例：

`n := negroni.New()`

`Negroni` 实例的 `Run` 方法是一个便捷函数，它将 `Negroni` 堆栈作为 HTTP 服务器运行。地址字符串的格式与 `http.ListenAndServe` 相同：

`n.Run(":8080")`

#### 使用 Negroni 进行路由

`Negroni` 实例的 `UseHandler` 方法允许你将自定义的 `http.Handler` 放入中间件堆栈中。

清单 6-9 是一个示例程序，它使用 Gorilla `mux` 作为与 `Negroni` 包一起使用的请求多路复用器。

**清单 6-9.** 使用 Negroni 和 Gorilla mux 的简单 HTTP 服务器

```
package main

import (
    "fmt"
    "net/http"
    "github.com/codegangsta/negroni"
    "github.com/gorilla/mux"
)

func index(w http.ResponseWriter, req *http.Request) {
    fmt.Fprintf(w, "Welcome!")
}

func main() {
    router := mux.NewRouter()
    router.HandleFunc("/", index)
    n := negroni.Classic()
    n.UseHandler(router)
    n.Run(":8080")
}
```

Gorilla 的 `mux.Router` 对象作为与 `Negroni` 一起使用的处理器。你可以将任何实现了 `http.Handler` 接口的对象传递给 `Negroni` 实例的 `UseHandler` 方法。

#### 注册中间件

`Negroni` 通过 `negroni.Handler` 接口管理中间件流程。

清单 6-10 展示了 `negroni.Handler` 接口的定义：

**清单 6-10.** `negroni.Handler` 接口

```
type Handler interface {
  ServeHTTP(rw http.ResponseWriter, r *http.Request, next http.HandlerFunc)
}
```

清单 6-11 提供了为 `Negroni` 编写中间件处理函数以配合 `negroni.Handler` 接口使用的模式。

**清单 6-11.** `negroni.Handler` 接口

```
func myMiddleware(w http.ResponseWriter, r *http.Request, next http.HandlerFunc) {
  // 执行下一个处理函数之前的逻辑
  next(w, r)
  // 执行下一个处理函数之后的逻辑
}
```

与 `Negroni` 兼容的中间件函数的函数签名与前面章节中编写的函数不同。Negroni 中间件堆栈使用以下签名来编写中间件函数：

`func myMiddleware(w http.ResponseWriter, r *http.Request, next http.HandlerFunc)`

在这里，你可以通过传递 `http.ResponseWriter` 对象和 `*http.Request` 对象的值来调用 `http.HandlerFunc` 对象，从而调用中间件堆栈中的下一个处理函数：

```
// 执行下一个处理函数之前的逻辑
  next(w, r)
  // 执行下一个处理函数之后的逻辑
```

你可以使用 `Use` 函数将中间件函数映射到 `Negroni` 处理链中，该函数接受一个 `negroni.Handler` 类型的参数。`Use` 函数将一个 `negroni.Handler` 添加到中间件堆栈中（参见清单 6-12）。处理函数按照它们添加到 `Negroni` 实例的顺序被调用。

**清单 6-12.** 使用 Negroni 注册中间件函数

```
n := negroni.New()
n.Use(negroni.HandlerFunc(myMiddleware))
```

中间件函数被转换为 `negroni.HandlerFunc` 类型，并添加到 `Negroni` 中间件堆栈中。`HandlerFunc` 是一个适配器，它允许普通函数被用作 `Negroni` 处理函数。

#### 为特定路由注册中间件

当使用诸如日志记录之类的中间件函数时，它们可能会被装饰成跨应用程序的处理函数。但是你可能需要为某些特定路由使用中间件函数；例如，你可能希望应用一些中间件函数，使其与某些特定于管理员用户账户的应用程序处理函数一起执行。在这种情况下，你可以创建一个新的 `Negroni` 实例，并将其用作你的路由处理函数。

清单 6-13 是将中间件函数应用于某些特定路由的代码块。

**清单 6-13.** 为特定路由注册中间件处理函数

```
router := mux.NewRouter()
adminRoutes := mux.NewRouter()

// 在此处添加管理员路由

// 为管理员中间件创建一个新的 negroni
router.Handle("/admin", negroni.New(
  Middleware1,
  Middleware2,
  negroni.Wrap(adminRoutes),
))
```



#### 使用 Negroni 中间件栈

清单 6-5 展示了中间件函数的控制流，其中两个中间件函数被装饰到应用处理器中。让我们重写该程序，演示如何用 `Negroni` 编写自定义中间件函数（见清单 6-14）。

**清单 6-14.** 用 Negroni 演示中间件控制流

```
package main

import (
    "fmt"
    "log"
    "net/http"
    "github.com/codegangsta/negroni"
)

func middlewareFirst(w http.ResponseWriter, r *http.Request, next http.HandlerFunc) {
    log.Println("MiddlewareFirst - Before Handler")
    next(w, r)
    log.Println("MiddlewareFirst - After Handler")
}

func middlewareSecond(w http.ResponseWriter, r *http.Request, next http.HandlerFunc) {
    log.Println("MiddlewareSecond - Before Handler")
    if r.URL.Path == "/message" {
        if r.URL.Query().Get("password") == "pass123" {
            log.Println("Authorized to the system")
            next(w, r)
        } else {
            log.Println("Failed to authorize to the system")
            return
        }
    } else {
        next(w, r)
    }
    log.Println("MiddlewareSecond - After Handler")
}

func index(w http.ResponseWriter, r *http.Request) {
    log.Println("Executing index Handler")
    fmt.Fprintf(w, "Welcome")
}

func message(w http.ResponseWriter, r *http.Request) {
    log.Println("Executing message Handler")
    fmt.Fprintf(w, "HTTP Middleware is awesome")
}

func iconHandler(w http.ResponseWriter, r *http.Request) {
}

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/favicon.ico", iconHandler)
    mux.HandleFunc("/", index)
    mux.HandleFunc("/message", message)
    n := negroni.Classic()
    n.Use(negroni.HandlerFunc(middlewareFirst))
    n.Use(negroni.HandlerFunc(middlewareSecond))
    n.UseHandler(mux)
    n.Run(":8080")
}
```

运行程序，分别向 `"/"` 发出请求，以及向 `"/message"` 发出请求（分别提供错误的查询字符串变量 `password` 值和正确的值 `"pass123"`）。你应该会看到类似以下的日志信息：

- `[negroni] listening on :8080`
- `[negroni] Started GET /`
- `2015/04/30 14:45:44 MiddlewareFirst - Before Handler`
- `2015/04/30 14:45:44 MiddlewareSecond - Before Handler`
- `2015/04/30 14:45:44 Executing index Handler`
- `2015/04/30 14:45:44 MiddlewareSecond - After Handler`
- `2015/04/30 14:45:44 MiddlewareFirst - After Handler`
- `[negroni] Completed 200 OK in 1.0008ms`
- `[negroni] Started GET /message`
- `2015/04/30 14:45:52 MiddlewareFirst - Before Handler`
- `2015/04/30 14:45:52 MiddlewareSecond - Before Handler`
- `2015/04/30 14:45:52 Failed to authorize to the system`
- `2015/04/30 14:45:52 MiddlewareFirst - After Handler`
- `[negroni] Completed 0 in 1.0008ms`
- `[negroni] Started GET /message`
- `2015/04/30 14:46:00 MiddlewareFirst - Before Handler`
- `2015/04/30 14:46:00 MiddlewareSecond - Before Handler`
- `2015/04/30 14:46:00 Authorized to the system`
- `2015/04/30 14:46:00 Executing message Handler`
- `2015/04/30 14:46:00 MiddlewareSecond - After Handler`
- `2015/04/30 14:46:00 MiddlewareFirst - After Handler`
- `[negroni] Completed 200 OK in 1.0071ms`

通过查看这些日志信息，你可以很容易地理解处理函数的控制流。

要重写现有程序以适配 `Negroni`，中间件处理函数必须修改为与 `negroni.Handler` 接口兼容：

```
func middlewareFirst(w http.ResponseWriter, r *http.Request, next http.HandlerFunc) {
    //下一个处理器之前的逻辑
    next(w, r)
    //下一个处理器之后的逻辑
}

func middlewareSecond(w http.ResponseWriter, r *http.Request, next http.HandlerFunc) {
    //下一个处理器之前的逻辑
    next(w, r)
    //下一个处理器之后的逻辑
}
```

当中间件处理函数兼容 `negroni.Handler` 接口后，需要使用 `Negroni` 实例的 `Use` 函数将它们添加到 `Negroni` 中间件栈中。处理函数按照添加到 `Negroni` 中间件栈的顺序被调用：

```
n := negroni.Classic()
n.Use(negroni.HandlerFunc(middlewareFirst))
n.Use(negroni.HandlerFunc(middlewareSecond))
```

由于 `Negroni` 实例是通过 `Classic` 函数创建的，中间件栈中会包含以下内置中间件函数：

- `negroni.Recovery`
- `negroni.Logging`
- `negroni.Static`

你也可以在通过 `New` 函数创建 `Negroni` 实例时添加中间件函数：

```
n := negroni.New(
    negroni.NewRecovery(),
    negroni.HandlerFunc(middlewareFirst),
    negroni.HandlerFunc(middlewareSecond),
    negroni.NewLogger(),
    negroni.NewStatic(http.Dir("public")),
)
```

`Negroni` 提供了一个非常简单且优雅的库来处理 HTTP 中间件函数。这是一个非常小巧的库，但在用 Go 构建真实世界的 Web 应用和 RESTful 服务时非常有用。`Negroni` 的主要优势之一是它与 `net/http` 库完全兼容。如果你不喜欢使用全功能的 Web 开发框架在 Go 中构建 Web 应用，那么使用 `Negroni` 制作和使用 HTTP 中间件是构建高效 Web 应用的一个好选择，它有助于实现更好的可重用性和可维护性。

### 在中间件之间共享值

在前面的章节中，你学习了如何在 Go 中制作和使用 HTTP 中间件。示例中间件是独立运行的，不依赖来自其他中间件处理器和应用处理器的任何数据。然而，在某些情况下，你可能需要向下一个中间件提供值，或者在中间件处理器和应用处理器之间共享值。例如，当你通过中间件处理函数授权应用时，它可能需要在请求处理周期中向下一个处理器提供一些特定于用户的值。

### 使用 Gorilla context

有许多第三方包可用于存储在请求生命周期内共享的值。Gorilla Web 工具包中的 `context` 包是一个很好的选择，可用于在请求生命周期内共享值。

要安装 `context`，在终端中运行以下命令：

```
$ go get github.com/gorilla/context
```

要使用 `context` 包，必须将 `github.com/gorilla/context` 包添加到导入列表中：

```
import "github.com/gorilla/context"
```



### 使用 Gorilla context 设置和获取值

要向 `context` 对象设置值，请使用 `Set` 函数（参见清单 6-15）。

**清单 6-15. 在 Gorilla context 中设置值**

```
context.Set(r, "user", "shijuvar")
```

这里的 `r` 是处理函数中的 `*http.Request` 对象。要从 `context` 对象获取值，请使用 `Get` 或 `GetOk` 函数（参见清单 6-16）。

**清单 6-16. 从 Gorilla context 获取值**

```
// val 的值为 "shijuvar"
val := context.Get(r, "user")

// 返回 ("shijuvar", true)
val, ok := context.GetOk(r, foo.MyKey)
```

清单 6-17 是一个示例程序，其中来自中间件处理函数的值被传递给了应用程序处理函数。

**清单 6-17. 中间件函数将值传递给应用程序处理函数**

```
package main

import (
    "fmt"
    "log"
    "net/http"
    "github.com/codegangsta/negroni"
    "github.com/gorilla/context"
)

func Authorize(w http.ResponseWriter, r *http.Request, next http.HandlerFunc) {
    token := r.Header.Get("X-AppToken")
    if token == "bXlVc2VybmFtZTpteVBhc3N3b3Jk" {
        log.Printf("已授权进入系统")
        context.Set(r, "user", "Shiju Varghese")
        next(w, r)
    } else {
        http.Error(w, "未授权", 401)
    }
}

func index(w http.ResponseWriter, r *http.Request) {
    user := context.Get(r, "user")
    fmt.Fprintf(w, "欢迎 %s!", user)
}

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/", index)
    n := negroni.Classic()
    n.Use(negroni.HandlerFunc(Authorize))
    n.UseHandler(mux)
    n.Run(":8080")
}
```

我们创建了一个中间件处理函数来授权 HTTP 请求。从 `request` 对象中读取 HTTP 头部值 `"X-AppToken"`，并验证安全令牌。这是验证 RESTful API 的一种方式，即 HTTP 客户端必须通过 HTTP 头部发送安全令牌。某些 API 会验证令牌并提供一个访问令牌，以便在特定会话中访问应用程序。

在这个程序中，你想将用户名传递给下一个处理函数：应用程序处理函数。因此，你在已授权的中间件处理函数中将用户值设置到 `context` 对象中，该值在 HTTP 请求的生命周期内都是可访问的。这里，`context` 对象的值从 index 应用程序处理函数中被访问。

通过利用 Gorilla `context` 包在中间件处理函数之间共享值，或者在中间件处理函数和应用程序处理函数之间共享值，你可以构建有用的中间件处理函数，从而开发实际应用程序。

### 总结

使用 HTTP 中间件是在 Go 中构建实际应用程序的一种重要且实用的方法。中间件是一段可插拔且独立的代码，它包装了应用程序处理函数，可用于将共享行为实现到整个应用程序的处理函数或某些特定应用程序的处理函数中。

HTTP 中间件允许你使用可插拔的逻辑构建应用程序，从而实现更高水平的可重用性和可维护性。使用 HTTP 中间件，你可以在 HTTP 请求处理函数之前或之后执行某些逻辑。由于 HTTP 中间件是可插拔组件，你可以随时添加或移除它们。

第三方包 `Alice` 允许你使用其流式接口（fluent interface）以优雅的语法实现中间件处理函数的链式调用。第三方包 `Negroni` 是一个处理中间件函数的优秀库，它本身也附带了一些默认的中间件函数。`Negroni` 提供了一种在 Go 中使用 HTTP 中间件的惯用方法。

当你构建实际应用程序时，可能需要在各种中间件处理函数和应用程序处理函数之间共享值。来自 Gorilla Web 工具包的第三方 `context` 包可用于在请求生命周期内共享值。在 Go 中，一个出色的 Web 开发栈是：使用 `net/http` 作为 Web 开发的基本编程模块，使用 `Negroni` 作为处理 HTTP 中间件的处理器，使用 Gorilla `mux` 作为路由器，并使用 Gorilla `context` 作为在请求生命周期内共享值的机制。有了这个 Web 开发栈，你就不再需要一个全功能的 Web 开发框架。

