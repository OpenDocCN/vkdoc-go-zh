# 8. HTTP

万维网是一个拥有数百万用户的大型分布式系统。一个站点可以通过运行 HTTP 服务器成为网络主机。虽然 Web 客户端通常是使用浏览器的用户，但还有许多其他“用户代理”，例如网络爬虫、Web 应用程序客户端等。

Web 构建在 HTTP（超文本传输协议）之上，该协议通常分层在套接字（例如 TCP）之上。HTTP 已经经历了四个公开可用的版本。版本 1.1（第三个版本）是开发者最熟悉的版本。向 HTTP/2 的幕后过渡已经占据了当前 HTTP 流量的 60% 以上。HTTP/3 是最新的更新；由于性能优势，向这个新版本的过渡正在持续增加。

本章是对 HTTP 的概述，随后介绍用于管理 HTTP 连接的 Go API。

## URL 和资源

URL 指定了*资源*的位置。资源通常是一个静态文件，例如 HTML 文档、图像或声音文件。但越来越多地，它可能是一个动态生成的对象，也许是基于数据库中存储的信息。

当用户代理请求一个资源时，返回的不是资源本身，而是该资源的某种*表示*。例如，如果资源是一个静态文件，那么发送给用户代理的是该文件的一个副本。

多个 URL 可能指向同一个资源，HTTP 服务器将为每个 URL 返回该资源的适当表示。例如，一家公司可能使用同一产品的不同 URL 在内部和外部提供产品信息。产品的内部表示可能包含诸如产品内部联系人员等信息，而外部表示可能包括销售该产品的商店位置。

这种对资源的看法意味着 HTTP 协议可以相当简单直接，而 HTTP 服务器则可能任意复杂。HTTP 必须将请求从用户代理传递到服务器并返回一个字节流，而服务器则可能需要执行任意数量的请求处理工作。



### i18n

互联网日益国际化（i18n）引发了一些复杂问题。主机名可以采用称为 IDN（国际化域名）的国际化形式给出。为了与不支持 Unicode 的旧有实现（如旧版邮件服务器）保持兼容，非 ASCII 域名会被映射为称为 *punycode* 的 ASCII 表示形式。例如，域名 `日本語.jp` 的 punycode 值为 `xn—wgv71a119e.jp`。例如，我们可以使用 `telnet` 来查看转换。

```
$ mkdir ch8
$ ch ch8
ch8$ telnet 日本語.jp 80
Trying 2001:218:3001:7::110...
Connected to xn--wgv71a119e.jp.
Escape character is '^]'.
^]
telnet> quit
Connection closed.
```

非 ASCII 域名到 punycode 值的转换并非由 Go 的 `net` 库自动执行，但有一个名为 `golang.org/x/net/idna` 的扩展包，可用于在 Unicode 及其 punycode 值之间进行转换。关于此主题，在“Figure Out IDNA Punycode Story”([`https://github.com/golang/go/issues/13835`](https://github.com/golang/go/issues/13835)) 中正在进行讨论。

我们可以针对前述域名 `日本語.jp` 尝试使用 IDNs 包；创建文件 `punycode.go`。

```
ch8$ vi punycode.go
package main
import (
"fmt"
"golang.org/x/net/idna"
"net/url"
)
func main() {
s := "https://日本語.jp:8443"
r1, _ := idna.ToASCII(s)
r2, _ := idna.ToUnicode(r1)
fmt.Println(r1)
fmt.Println(r2)
fmt.Println(url.QueryEscape(s))
}
ch8$ go mod init example.com/user/punycode
ch8$ go mod tidy
ch8$ go run punycode.go
xn--https://-5y4qg6h355l.jp:8443
https://日本語.jp:8443
https%3A%2F%2F%E6%97%A5%E6%9C%AC%E8%AA%9E.jp%3A8443
```

国际化域名带来了所谓的*同形字*攻击的可能性。许多 Unicode 字符外观相似，例如俄语中的 o (U+043E)、希腊语中的 o (U+03BF) 和英语中的 o (U+006F)。使用诸如 `google.com`（包含两个俄语 o）这类同形字的域名可能会造成混乱。已知有多种防御措施，例如始终显示 punycode（此处为 `xn—ggle-55da.com`，使用 Punycode 转换器）。

URI/URL 中的路径处理起来更为复杂，因为它指向相对于某个 HTTP 服务器的路径，而该服务器可能运行在特定的本地化环境中。编码可能不是 UTF-8，甚至不是 Unicode。IRI（国际化资源标识符）通过首先将任何本地化字符串转换为 UTF-8，然后对任何非 ASCII 字节进行百分号转义来管理这一点。标题为“An Introduction to Multilingual Web Addresses”([`https://www.w3.org/International/articles/idn-and-iri/`](https://www.w3.org/International/articles/idn-and-iri/)) 的 W3C 页面提供了更多信息。从其他编码转换为 UTF-8 在第 6 章中已介绍，而 Go 在 `net/url` 中提供了 `QueryEscape/QueryUnescape` 函数，并在 Go 1.8 中提供了 `PathEscape/PathUnescape` 函数来进行百分号转换。

### HTTP 特性

HTTP 是一种无状态、无连接、可靠的协议。在最简单的形式中，来自用户代理的每个请求都被可靠地处理，然后连接被断开。

在最早的 HTTP 版本中，每个请求都涉及一个独立的 TCP 连接，因此如果需要许多资源（例如 HTML 页面中嵌入的图片），则需要在短时间内建立和拆除许多 TCP 连接。

HTTP 1.1 在 HTTP 中添加了许多优化，这增加了简单结构的复杂性，但创建了一个更高效、更可靠的协议。HTTP/2 采用了二进制形式以进一步提高效率。HTTP/3 更进一步，用 UDP 取代了典型的 TCP 套接字，并进行了其他相关改进（例如，通过 TLS 提供内置安全性）。

### 版本

HTTP 有四个版本：

*   版本 0.9 (1991)：完全过时
*   版本 1.0 (1996)：几乎过时
*   版本 1.1 (1999)：目前最流行的版本
*   版本 2 (2015)：最新版本
*   版本 3 (~2022)：处于正式批准的最终阶段，已投入生产使用

每个版本都必须理解早期版本的请求和响应。

#### HTTP/0.9

HTTP/0.9 是 1991 年由蒂姆·伯纳斯-李定义的原始 HTTP。您可以在此处找到规范：[`https://www.w3.org/Protocols/HTTP/AsImplemented.html`](https://www.w3.org/Protocols/HTTP/AsImplemented.html)。

请求格式：

```
Request = Simple-Request
Simple-Request = "GET" SP Request-URI CRLF
```

##### 响应格式

响应采用以下形式：

```
Response = Simple-Response
Simple-Response = [Entity-Body]
```

上述内容只是对 HTTP/0.9 的一瞥（例如，SP 代表“空格”）。由于它已不再使用，我们不再花更多时间在上面。

#### HTTP/1.0

该版本向请求和响应中添加了更多信息。0.9 格式并未被“扩展”，而是与新版本并存。此时，W3C 和 IETF 组织的参与度更高了。HTTP/1.0 规范可在此处找到：[`https://datatracker.ietf.org/doc/html/rfc1945`](https://datatracker.ietf.org/doc/html/rfc1945)。

值得注意的是，在此期间，Roy Fielding 教授因开发 REST 而闻名。他关于 REST 的论文大部分内容可在此处找到：[`https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm`](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)。

##### 请求格式

从客户端到服务器的请求格式是：

```
Request = Simple-Request | Full-Request
Simple-Request = "GET" SP Request-URI CRLF
Full-Request = Request-Line
*(General-Header
| Request-Header
| Entity-Header)
CRLF
[Entity-Body]
```

`Simple-Request` 是 HTTP/0.9 请求，并且必须由 `Simple-Response` 回复。

`Request-Line` 的格式如下：

```
Request-Line = Method SP Request-URI SP HTTP-Version CRLF
```

其中：

```
Method = "GET" | "HEAD" | POST |
extension-method
```

示例如下：

```
GET http://jan.newmarch.name/index.html HTTP/1.0
```

##### 响应格式

响应采用以下形式：

```
Response = Simple-Response | Full-Response
Simple-Response = [Entity-Body]
Full-Response = Status-Line
*(General-Header
| Response-Header
| Entity-Header)
CRLF
[Entity-Body]
```

`Status-Line` 提供了关于请求处理结果的信息：

```
Status-Line = HTTP-Version SP Status-Code SP Reason-Phrase CRLF
```

示例如下：

```
HTTP/1.0 200 OK
```

状态行中的状态码如下：

```
Status-Code =     "200" ; OK
| "201" ; Created
| "202" ; Accepted
| "204" ; No Content
| "301" ; Moved permanently
| "302" ; Moved temporarily
| "304" ; Not modified
| "400" ; Bad request
| "401" ; Unauthorized
| "403" ; Forbidden
| "404" ; Not found
| "500" ; Internal server error
| "501" ; Not implemented
| "502" ; Bad gateway
| "503" | Service unavailable
| extension-code
```

有些状态码在 HTTP/1.0 标准发布时尚未定义。例如，203 “非权威信息” 和 303 “查看其他位置” 是在 HTTP/1.1 中定义的。

`General-Header` 通常是日期，而 `Response-Header` 是位置、服务器或认证字段。

`Entity-Header` 包含关于后续 `Entity-Body` 的有用信息：

```
Entity-Header = Allow
| Content-Encoding
| Content-Length
| Content-Type
| Expires
| Last-Modified
| extension-header
```

例如（其中字段类型在 `//` 后给出）：

```
HTTP/1.1 200 OK                             // status line
Date: Fri, 29 Aug 2003 00:59:56 GMT         // general header
Server: Apache/2.0.40 (Unix)                // response header
Content-Length: 1595                        // entity header
Content-Type: text/html; charset=ISO-8859-1 // entity header
```



#### HTTP 1.1

`HTTP 1.1` 修复了 `HTTP 1.0` 的诸多问题，但也因此变得更加复杂。该版本通过扩展或完善 `HTTP 1.0` 的可用选项来实现。例如：

- 增加了更多命令，如 `TRACE` 和 `CONNECT`。
- `HTTP 1.1` 收紧了请求 URL 的规则以支持代理处理。如果请求通过代理转发，URL 应为绝对 URL，如下所示：

```
GET http://www.w3.org/index.html HTTP/1.1
```

否则，应使用绝对路径，并且必须包含 `Host` 头部字段，如下所示：

- 增加了更多属性，如 `If-Modified-Since`，同样供代理使用。

```
GET /index.html HTTP/1.1
Host: www.w3.org
```

具体变更包括：

- 主机名识别（支持虚拟主机）
- 内容协商（支持多语言）
- 持久连接（减少 TCP 开销，但非常复杂）
- 分块传输
- 字节范围（请求文档的部分内容）
- 代理支持

得益于万维网的普及，HTTP 持续得到改进。`HTTP/1.1` 的初始开发始于 1997 年，一直持续到 2014 年。你可以在此找到关于 `HTTP/1.1` 的更多信息，包括原始规范以及之后对协议进行重新记录的更详细文档。

- 单文档（原始格式）：
  - 原始版 (1997) – [`https://datatracker.ietf.org/doc/html/rfc2068`](https://datatracker.ietf.org/doc/html/rfc2068)
  - 更新版 (1999) – [`https://datatracker.ietf.org/doc/html/rfc2616`](https://datatracker.ietf.org/doc/html/rfc2616)
- 详细系列（均为 2014 年版）：
  - 消息语法与路由 – [`https://datatracker.ietf.org/doc/html/rfc7230`](https://datatracker.ietf.org/doc/html/rfc7230)
  - 语义与上下文 – [`https://datatracker.ietf.org/doc/html/rfc7231`](https://datatracker.ietf.org/doc/html/rfc7231)
  - 条件请求 – [`https://datatracker.ietf.org/doc/html/rfc7232`](https://datatracker.ietf.org/doc/html/rfc7232)
  - 范围请求 – [`https://datatracker.ietf.org/doc/html/rfc7233`](https://datatracker.ietf.org/doc/html/rfc7233)
  - 缓存 – [`https://datatracker.ietf.org/doc/html/rfc7234`](https://datatracker.ietf.org/doc/html/rfc7234)
  - 认证 – [`https://datatracker.ietf.org/doc/html/rfc7235`](https://datatracker.ietf.org/doc/html/rfc7235)

#### HTTP 主要版本升级

`HTTP/0.9` 协议仅有一页篇幅。`HTTP/1.0` 协议的描述约有 20 页，并包含了 `HTTP/0.9` 协议。`HTTP/1.1` 协议长达 120 页，是对 `HTTP/1.0` 的实质性扩展，而 `HTTP/2` 大约为 96 页。`HTTP/2` 规范仅对 `HTTP/1.1` 规范进行了补充。即将完成的 `HTTP/3` 规范大约为 75 页，在传输方面增加了更多功能和改进。

#### HTTP/2

所有早期版本的 HTTP 都是基于文本的。`HTTP/2` 最重大的不同之处在于它是一种二进制格式。为确保向后兼容性，不能通过向旧服务器发送二进制消息来判断其反应。取而代之的是，发送一条带有额外属性的 `HTTP 1.1` 消息，询问服务器是否愿意切换至 `HTTP/2`。如果服务器不理解这些额外字段，它会以正常的 `HTTP 1.1` 回复响应，会话便继续使用 `HTTP 1.1`。否则，服务器可以回复表明其愿意切换，之后会话便可继续使用 `HTTP/2`。

#### HTTP/3

`HTTP/3` 进一步改进了 `HTTP/2` 中开创的理念，包括：

- 流多路复用
- 逐流流量控制
- 低延迟连接建立

为了使得 HTTP 传输性能更佳，一种新的传输机制被创建出来。

简要对各主要协议进行比较：

- `HTTP/1.1` 可在多种传输层和会话层上使用。
- `HTTP/2` 主要用于基于 TCP 的 TLS（传输层安全性协议）之上。
- `HTTP/3` 基于一种名为 QUIC 的新传输协议，使用相同的语义。

`HTTP/2` 改进了 HTTP over TCP 的缺陷，但并未与 TCP 完全集成（例如，跨连接的拥塞控制未能统一管理）。`HTTP/3` 将 `HTTP/2` 的大部分控制功能与 TLS 合并为一个名为 QUIC 的新协议。从某种意义上说，QUIC 合并了第 4 层和第 5 层，但运行在 UDP（第 4 层）之上。使用 UDP 使得 `HTTP/3` 能够基于现有网络运行（不过仍然存在一些担忧，例如 TCP 流量通常经过优化，而中间路由器对 UDP 的优化较少）。

即使有前述改进，网络工程师通常仍需在 `HTTP/1.1` 的领域内工作。浏览器和服务器（例如代理）的创建者将需要学习更多内容，包括 `HTTP/2` 和 `HTTP/3`。

您可以在此了解有关 QUIC 传输的信息： [`https://datatracker.ietf.org/doc/html/draft-ietf-quic-transport`](https://datatracker.ietf.org/doc/html/draft-ietf-quic-transport)。

您可以在此了解有关 HTTP 和 QUIC 的信息： [`https://datatracker.ietf.org/doc/html/draft-ietf-quic-http`](https://datatracker.ietf.org/doc/html/draft-ietf-quic-http)。

## 简单用户代理

用户代理（如浏览器）发出请求并获取响应。Go 在 `net/http` 包中提供了一组用于请求和响应的类型。首先，我们来了解 `net/http.Response`。

### 响应类型

响应类型如下所示：

```
type Response struct {
    Status     string // 例如 "200 OK"
    StatusCode int    // 例如 200
    Proto      string // 例如 "HTTP/1.0"
    ProtoMajor int    // 例如 1
    ProtoMinor int    // 例如 0
    Header Header
    Body io.ReadCloser
    ContentLength int64
    TransferEncoding []string
    Close bool
    Uncompressed bool
    Trailer Header
    Request *Request
    TLS *tls.ConnectionState
}
```

以及以下辅助方法：

```
func (r *Response) Cookies() []*Cookie
func (r *Response) Location() (*url.URL, error)
func (r *Response) ProtoAtLeast(major, minor int) bool
func (r *Response) Write(w io.Writer) error
```

其中一些方法是为了方便使用，例如 `Cookies` 和 `Location`；另一些则用于辅助连接管理，例如 `ProtoAtLeast` 和 `Write`。

有关确切细节，请参阅文档（例如，`go doc -short net/http.Response.Body`）。

我们从 `Response` 类型开始介绍，是因为 Go 提供了辅助函数来发起请求；稍后，我们将研究 `Request` 类型。



### HEAD 方法

我们通过示例来研究这种数据结构。在 `net/http` 包中，每种 HTTP 请求类型都有其对应的 Go 函数。最简单的请求来自名为 `HEAD` 的用户代理，它用于请求关于某个资源及其 HTTP 服务器的信息。可以使用此函数发起查询：

```
func Head(url string) (r *Response, err error)
```

响应的状态位于响应字段 `Status` 中，而 `Header` 字段是 HTTP 响应中头部字段的映射。一个名为 `head.go` 的程序，用于发起此请求并显示结果，内容如下：

```
ch8$ vi head.go
/* Head
*/
package main
import (
"fmt"
"log"
"net/http"
"os"
)
func main() {
if len(os.Args) != 2 {
log.Fatalln("Usage: ", os.Args[0], "host:port")
}
url := os.Args[1]
response, err := http.Head(url)
if err != nil {
log.Fatalln(err)
}
fmt.Println(response.Status)
for k, v := range response.Header {
fmt.Println(k+":", v)
}
}
```

当对 `go.dev` 运行时，我们看到：

```
ch8$ go run head.go https://go.dev
200 OK
Content-Security-Policy: [connect-src 'self' www.google-analytics.com stats.g.doubleclick.net ; default-src 'self' ; font-src 'self' fonts.googleapis.com fonts.gstatic.com data: ; frame-ancestors 'self' ; frame-src 'self' www.google.com feedback.googleusercontent.com www.googletagmanager.com scone-pa.clients6.google.com www.youtube.com player.vimeo.com ; img-src 'self' www.google.com www.google-analytics.com ssl.gstatic.com www.gstatic.com gstatic.com data: * ; object-src 'none' ; script-src 'self' 'sha256-n6OdwTrm52KqKm6aHYgD0TFUdMgww4a0GQlIAVrMzck=' 'sha256-4ryYrf7Y5daLOBv0CpYtyBIcJPZkRD2eBPdfqsN3r1M=' 'sha256-sVKX08+SqOmnWhiySYk3xC7RDUgKyAkmbXV2GWts4fo=' www.google.com apis.google.com www.gstatic.com gstatic.com support.google.com www.googletagmanager.com www.google-analytics.com ssl.google-analytics.com tagmanager.google.com ; style-src 'self' 'unsafe-inline' fonts.googleapis.com feedback.googleusercontent.com www.gstatic.com gstatic.com tagmanager.google.com ;]
Strict-Transport-Security: [max-age=31536000; includeSubDomains; preload]
X-Cloud-Trace-Context: [7ba1cc2dfaeebe50e11befbb48523327]
Date: [Thu, 31 Mar 2022 23:19:03 GMT]
Server: [Google Frontend]
Content-Type: [text/html; charset=utf-8]
Vary: [Accept-Encoding]
```

此响应来自我们无法控制的服务器，并且可能在途中经过其他服务器。显示的字段可能不同，当然，字段的值也会不同。以下是一些响应头部的简要说明：

- **Vary**：告知源服务器在选取资源表示时，除了 `"method"`、`Host` 和请求目标之外，还要使用哪些字段
  - Vary 是 HTTP/1.1 的一部分 – [`https://datatracker.ietf.org/doc/html/rfc7231#section-7.1.4`](https://datatracker.ietf.org/doc/html/rfc7231#section-7.1.4)
- **Strict-Transport-Security**：服务器用来告知浏览器（客户端）使用 HTTPS 而非 HTTP
  - 也称为 HSTS，是 HTTP 的一个附加策略 – [`https://www.rfc-editor.org/rfc/rfc6797`](https://www.rfc-editor.org/rfc/rfc6797)
- **X-Cloud-Trace-Context**：是 Google Cloud Platform 使用的追踪头部
- **Date**：源服务器生成响应的时间
- **Server**：用于标识生成响应的软件
  - Server 是 HTTP/1.1 的一部分 – [`https://datatracker.ietf.org/doc/html/rfc2616#section-14.38`](https://datatracker.ietf.org/doc/html/rfc2616#section-14.38)
- **Alt-Svc**：代表替代服务，允许源服务器的资源在另一个位置甚至不同的协议上权威可用
  - 该提案可在此处找到：[`https://datatracker.ietf.org/doc/html/rfc7838`](https://datatracker.ietf.org/doc/html/rfc7838)
  - 该提案解释了添加 `Alt-Svc` 是为了阐明资源的位置与资源标识之间的区别。在这里，我们看到它被用来解释我们可以将请求修改为 `h3` (HTTP/3) 甚至 QUIC！
    - `"ma"` 表示此位置/协议上资源可用的最大期限（max-age）。
    - `"v"` 用于指示使用的协议版本，是某些仍在使用的早期 QUIC-HTTP 草案的一部分。
- **Content-Type**：用于解析响应体

虽然本书不侧重于网络的任何特定方面，主要关注 Go 的网络能力；但我们认为解释相关项目（例如上述头部）是有启发性的。

像 `Head` 这样的便捷函数背后有着未言明的复杂性。让我们看一下 `Head` 的文档。

```
ch8$ go doc net/http.Head
package http // import "net/http"
func Head(url string) (resp *Response, err error)
Head 向指定的 URL 发起一个 HEAD 请求。如果响应是以下重定向状态码之一，Head 会跟随重定向，最多重定向 10 次：
301 (Moved Permanently)
302 (Found)
303 (See Other)
307 (Temporary Redirect)
308 (Permanent Redirect)
Head 是 DefaultClient.Head 的一个包装器
要使用指定的 context.Context 发起请求，请使用
NewRequestWithContext 和 DefaultClient.Do。
```

以下是需要考虑的几个关键点：

- `30z` 状态码会导致重定向（即另一个请求）。
- 存在一个默认客户端：

```
ch8$ go doc net/http.DefaultClient
package http // import "net/http"
var DefaultClient = &Client{}
DefaultClient 是默认的 Client，被 Get、Head 和 Post 使用。
```

我们将在后续章节中学习更多关于此客户端及其相关的 `Do` 方法的内容。



### GET 方法

通常，我们希望获取资源的表示形式，而不仅仅是获取关于它的信息。`GET` 请求可以做到这一点，可以通过以下方式完成：

```
func Get(url string) (r *Response, finalURL string, err error)
```

响应的内容位于响应字段 `Body` 中，其类型为 `io.ReadCloser`。我们可以使用程序 `get.go` 将内容打印到屏幕上：

```
ch8$ vi get.go
/* Get
*/
package main
import (
"fmt"
"io"
"log"
"net/http"
"net/http/httputil"
"os"
"strings"
)
func main() {
if len(os.Args) != 2 {
log.Fatalln("Usage: ", os.Args[0], "host:port")
}
url := os.Args[1]
response, err := http.Get(url)
checkError(err)
if response.StatusCode != http.StatusOK {
log.Fatalln(response.StatusCode)
}
fmt.Println("The response header is")
b, _ := httputil.DumpResponse(response, false)
fmt.Print(string(b))
contentTypes := response.Header["Content-Type"]
if !acceptableCharset(contentTypes) {
log.Fatalln("Cannot handle", contentTypes)
}
fmt.Println("The response body is")
var buf [512]byte
reader := response.Body
for {
n, err := reader.Read(buf[0:])
if err != nil {
if err == io.EOF {
fmt.Print(string(buf[0:n]))
reader.Close()
break
}
checkError(err)
}
fmt.Print(string(buf[0:n]))
}
}
func acceptableCharset(contentTypes []string) bool {
// each type is like [text/html; charset=utf-8]
// we want the UTF-8 only
for _, cType := range contentTypes {
if strings.Index(cType, "utf-8") != -1 {
return true
}
}
return false
}
func checkError(err error) {
if err != nil {
log.Fatalln(err)
}
}
```

当 `get.go` 客户端对 Go 官网运行时，我们看到以下内容。

```
ch8$ go run get.go https://go.dev
The response header is
HTTP/2.0 200 OK
Cache-Control: private
Content-Security-Policy: connect-src 'self' www.google-analytics.com stats.g.doubleclick.net ; default-src 'self' ; font-src 'self' fonts.googleapis.com fonts.gstatic.com data: ; frame-ancestors 'self' ; frame-src 'self' www.google.com feedback.googleusercontent.com www.googletagmanager.com scone-pa.clients6.google.com www.youtube.com player.vimeo.com ; img-src 'self' www.google.com www.google-analytics.com ssl.gstatic.com www.gstatic.com gstatic.com data: * ; object-src 'none' ; script-src 'self' 'sha256-n6OdwTrm52KqKm6aHYgD0TFUdMgww4a0GQlIAVrMzck=' 'sha256-4ryYrf7Y5daLOBv0CpYtyBIcJPZkRD2eBPdfqsN3r1M=' 'sha256-sVKX08+SqOmnWhiySYk3xC7RDUgKyAkmbXV2GWts4fo=' www.google.com apis.google.com www.gstatic.com gstatic.com support.google.com www.googletagmanager.com www.google-analytics.com ssl.google-analytics.com tagmanager.google.com ; style-src 'self' 'unsafe-inline' fonts.googleapis.com feedback.googleusercontent.com www.gstatic.com gstatic.com tagmanager.google.com ;
Content-Type: text/html; charset=utf-8
Date: Thu, 31 Mar 2022 23:33:22 GMT
Server: Google Frontend
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Vary: Accept-Encoding
X-Cloud-Trace-Context: d6e6efc338f0723fe7550203794db95c
The response body is

(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
})(window,document,'script','dataLayer','GTM-W8MVQXG');

(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
})(window,document,'script','dataLayer','GTM-W8MVQXG');

The Go Programming Language

...
```

这个请求是通过 HTTP/2 发送的。Go 库已经为你执行了版本协商。基于 TLS 握手和其他信息，Go 会升级到 HTTP/2。同样，这些便捷函数和相关的客户端使得在 Go 中开始使用 HTTP 变得很容易。如果你想继续使用 HTTP/1.1，可以尝试以下覆盖方法：

```
GODEBUG=http2client=0 go run get.go https://go.dev
```

相关代码的文档如下所示：

```
ch8$ go doc -u net/http onceSetNextProtoDefaults
package http // import "net/http"
func (srv *Server) onceSetNextProtoDefaults()
onceSetNextProtoDefaults configures HTTP/2, if the user hasn't configured
otherwise. (by setting srv.TLSNextProto non-nil) It must only be called via
srv.nextProtoOnce (use srv.setupHTTP2_*).
func (t *Transport) onceSetNextProtoDefaults()
onceSetNextProtoDefaults initializes TLSNextProto. It must be called via
t.nextProtoOnce.Do.
```

稍后，我们将研究自定义客户端行为。上一章讨论的类型存在重要的字符集问题。服务器将使用某种字符集编码，并且可能使用某种传输编码来传送内容。通常，这取决于用户代理和服务器之间的协商，但我们在 `GET` 命令中使用的简单方法不包含协商的用户代理部分。因此，服务器可以发送它想要的任何字符编码。

在最初编写时，我（Jan）在中国（并且可以访问 Google）。当我在 `www.google.com` 上尝试这个程序时，Google 的服务器试图通过猜测我的位置，并用中文字符集 Big5 向我发送文本，这“很有帮助”！如何告诉服务器我可以接受哪种字符编码将在后面讨论。


```markdown
## 配置 HTTP 请求

Go 也提供了一个较低级别的接口，供用户代理与 HTTP 服务器通信。正如你所料，这不仅能让你对客户端请求拥有更多控制权，也要求你在构建请求时投入更多精力。不过，复杂性仅略有增加。

用于构建请求的数据类型是 `Request` 类型；请查阅相关文档。

```
type Request struct {
    Method string
    URL *url.URL
    Proto      string // "HTTP/1.0"
    ProtoMajor int    // 1
    ProtoMinor int    // 0
    Header Header
    Body io.ReadCloser
    GetBody func() (io.ReadCloser, error)
    ContentLength int64
    TransferEncoding []string
    Close bool
    Host string
    Form url.Values
    PostForm url.Values
    MultipartForm *multipart.Form
    Trailer Header
    RemoteAddr string
    RequestURI string
    TLS *tls.ConnectionState
    Cancel <-chan struct{}
    Response *Response
}
```

以下是相关方法：

```
func (r *Request) AddCookie(c *Cookie)
func (r *Request) BasicAuth() (username, password string, ok bool)
func (r *Request) Clone(ctx context.Context) *Request
func (r *Request) Context() context.Context
func (r *Request) Cookie(name string) (*Cookie, error)
func (r *Request) Cookies() []*Cookie
func (r *Request) FormFile(key string) (multipart.File, *multipart.FileHeader, error)
func (r *Request) FormValue(key string) string
func (r *Request) MultipartReader() (*multipart.Reader, error)
func (r *Request) ParseForm() error
func (r *Request) ParseMultipartForm(maxMemory int64) error
func (r *Request) PostFormValue(key string) string
func (r *Request) ProtoAtLeast(major, minor int) bool
func (r *Request) Referer() string
func (r *Request) SetBasicAuth(username, password string)
func (r *Request) UserAgent() string
func (r *Request) WithContext(ctx context.Context) *Request
func (r *Request) Write(w io.Writer) error
func (r *Request) WriteProxy(w io.Writer) error
```

`Request` 的一些方法用于客户端设置；另一些则被服务器用来检索信息。

请求中可以存储大量信息。你无需填充所有字段，只需关注那些你感兴趣的字段。使用如下方式创建具有默认值的请求是最简单的方法，例如：

```
request, err := http.NewRequest("GET", url.String(), nil)
```

一旦创建了请求，你就可以修改这些字段。例如，要指定只接收 UTF-8 编码，可以像下面这样向请求添加一个 `Accept-Charset` 字段：

```
request.Header.Add("Accept-Charset", "UTF-8;q=1, ISO-8859-1;q=0")
```

（注意，除非像我们这样在列表中明确指定，否则默认的 ISO-8859-1 始终获得值为 1。别忘了 HTTP 1.1 规范可是 1999 年的产物！）

设置客户端请求的 `charset` 很简单。但对于服务器返回的字符集值如何处理，则存在一些困惑。返回的资源*应该*包含一个 `Content-Type`，用以指定内容的媒体类型，例如 `text/html`。如果合适，媒体类型应该声明字符集，例如 `text/html; charset=UTF-8`。如果没有指定字符集，那么根据 HTTP 规范，应将其视为默认的 ISO-8859-1 字符集。但 HTML4 规范指出，由于许多服务器并不遵守此规定，你不能做出任何假设。

如果服务器的 `Content-Type` 中指定了字符集，就假定它是正确的。如果未指定，考虑到超过 50% 的页面使用 UTF-8 编码，还有部分使用 ASCII，因此假定为 UTF-8 是安全的。只有不到 10% 的页面可能会出问题 :-(。

### 客户端对象

要向服务器发送请求并获取回复，使用便捷对象 `Client` 是最简单的方式。该对象可以管理多个请求，并处理诸如服务器是否保持 TCP 连接存活等问题。

下面的程序 `clientget.go` 对此进行了说明。

该程序展示了如何添加 HTTP 头部，正如我们添加 `Accept-Charset` 头部以仅接受 UTF-8 编码。这里有一个小问题，由 Go 的一个 bug 引起，该 bug 仅在 Go 1.8 中修复。`Client.Do` 函数在收到 301、302、303 或 307 响应时会自动执行重定向。在 Go 1.8 之前，它在重定向时不会复制 HTTP 头部。

如果你尝试访问像 [`http://www.google.com`](http://www.google.com) 这样的站点，它会重定向到像 [`http://www.google.com.au`](http://www.google.com.au) 这样的站点，但会丢失 `Accept-Charset` 头部，并返回 ISO-8859-1（按照 1999 年 HTTP 1.1 规范，这是正确的行为）。考虑到这个前提 —— 即该程序在 Go 1.8 之前的版本上可能无法给出正确结果 —— 程序如下：

```
ch8$ vi clientget.go
/* ClientGet
*/
package main

import (
    "io"
    "fmt"
    "log"
    "net/http"
    "net/http/httputil"
    "net/url"
    "os"
    "strings"
)

func main() {
    if len(os.Args) != 2 {
        log.Fatalln("Usage: ", os.Args[0], "http://host:port/page")
    }
    url, err := url.Parse(os.Args[1])
    checkError(err)

    client := &http.Client{}

    request, err := http.NewRequest("HEAD", url.String(), nil)
    // 仅接受 UTF-8
    request.Header.Add("Accept-Charset", "utf-8;q=1,ISO-8859-1;q=0")
    checkError(err)

    response, err := client.Do(request)
    checkError(err)

    if response.StatusCode != http.StatusOK {
        log.Fatalln(response.Status)
    }

    fmt.Println("响应头部信息为")
    b, _ := httputil.DumpResponse(response, false)
    fmt.Print(string(b))

    chSet := getCharset(response)
    if chSet != "utf-8" {
        log.Fatalln("无法处理", chSet)
    }

    var buf [512]byte
    reader := response.Body
    fmt.Println("获取到响应体")
    for {
        n, err := reader.Read(buf[0:])
        if err != nil {
            if err == io.EOF {
                fmt.Print(string(buf[0:n]))
                break
            }
            checkError(err)
        }
        fmt.Print(string(buf[0:n]))
    }
}

func getCharset(response *http.Response) string {
    contentType := response.Header.Get("Content-Type")
    if contentType == "" {
        // 猜测
        return "utf-8"
    }
    idx := strings.Index(contentType, "charset=")
    if idx == -1 {
        // 猜测
        return "utf-8"
    }
    // 找到 charset 后将其移除
    chSet := strings.Trim(contentType[idx+8:], " ")
    return strings.ToLower(chSet)
}

func checkError(err error) {
    if err != nil {
        log.Fatalln("致命错误 ", err.Error())
    }
}
```

例如，按如下方式运行该程序：

```
ch8$ go run clientget.go https://go.dev
响应头部信息为
HTTP/2.0 200 OK
Connection: close
Content-Security-Policy: connect-src 'self' www.google-analytics.com stats.g.doubleclick.net ; default-src 'self' ; font-src 'self' fonts.googleapis.com fonts.gstatic.com data: ; frame-ancestors 'self' ; frame-src 'self' www.google.com feedback.googleusercontent.com www.googletagmanager.com scone-pa.clients6.google.com www.youtube.com player.vimeo.com ; img-src 'self' www.google.com www.google-analytics.com ssl.gstatic.com www.gstatic.com gstatic.com data: * ; object-src 'none' ; script-src 'self' 'sha256-n6OdwTrm52KqKm6aHYgD0TFUdMgww4a0GQlIAVrMzck=' 'sha256-4ryYrf7Y5daLOBv0CpYtyBIcJPZkRD2eBPdfqsN3r1M=' 'sha256-sVKX08+SqOmnWhiySYk3xC7RDUgKyAkmbXV2GWts4fo=' www.google.com apis.google.com www.gstatic.com gstatic.com support.google.com www.googletagmanager.com www.google-analytics.com ssl.google-analytics.com tagmanager.google.com ; style-src 'self' 'unsafe-inline' fonts.googleapis.com feedback.googleusercontent.com www.gstatic.com gstatic.com tagmanager.google.com ;
Content-Type: text/html; charset=utf-8
Date: Thu, 31 Mar 2022 23:45:09 GMT
Server: Google Frontend
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Vary: Accept-Encoding
X-Cloud-Trace-Context: 9caa17c0647c300f6e52d795661bf512
获取到响应体
```


## 代理处理

现今，HTTP 请求通过特定的 HTTP 代理进行转发已非常普遍。这是在构成 TCP 连接并在应用层运行的服务器之外的另一层。企业使用代理来限制员工可访问的内容，而许多组织则使用 Cloudflare 等代理服务作为缓存，减轻自身服务器的负载。通过代理访问网站需要客户端进行额外的处理。

### 简单代理

HTTP 1.1 规定了 HTTP 应如何通过代理工作。应向代理发送 `GET` 请求，但所请求的 URL 应是目标服务器的完整 URL。此外，HTTP 头部应包含 `Host` 字段，并将其设置为代理地址。只要代理配置为允许通过此类请求，即完成所有必要步骤。

Go 将这一功能视为 HTTP 传输层的一部分。为此，它提供了一个 `Transport` 结构体。该结构体包含一个字段，可设置为一个*函数*，该函数返回代理的 URL。如果我们有一个代理的 URL 字符串，可以创建相应的传输对象，然后将其赋值给客户端对象，如下所示：

```
// 准备传输
proxyURL, err := url.Parse(proxyString)
// 支持 HTTP 代理的 RoundTripper 实现
// 参见 go doc net/http.RoundTripper
transport := &http.Transport{Proxy: http.ProxyURL(proxyURL)}
// 用于发送 HTTP 请求
client := &http.Client{Transport: transport}
```

然后客户端便可以像之前一样继续操作。

以下程序 `proxyget.go` 演示了这一点。

```
ch8$ vi proxyget.go
/* ProxyGet
*/
package main
import (
"fmt"
"io"
"log"
"net/http"
"net/url"
"os"
)
func main() {
if len(os.Args) != 2 {
log.Fatalln("Usage: ", os.Args[0], "http://host:port/page")
}
rawURL := os.Args[1]
url, err := url.Parse(rawURL)
checkError(err)
response, err := http.Get(url.String())
checkError(err)
fmt.Println("Read ok")
if response.StatusCode != http.StatusOK {
log.Fatalln(response.StatusCode)
}
fmt.Println("Response ok")
var buf [512]byte
reader := response.Body
for {
n, err := reader.Read(buf[0:])
if err != nil {
if err == io.EOF {
fmt.Print(string(buf[0:n]))
reader.Close()
break
}
checkError(err)
}
fmt.Print(string(buf[0:n]))
}
}
func checkError(err error) {
if err != nil {
log.Fatalln("Fatal error ", err.Error())
}
}
```

如果你有一个代理，比如位于 `XYZ.com` 的 8080 端口，可以按如下方式测试：

```
ch8$ go run proxyget.go http://XYZ.com:8080 https://www.google.com
```

如果没有合适的代理进行测试，可以下载并安装 Squid 代理（[`http://www.squid-cache.org/`](http://www.squid-cache.org/)）到你的电脑上。例如，在安装了 Homebrew 的 Mac 上：

```
ch8$ brew install squid       // 通过 Homebrew 安装
ch8$ brew service start squid // 在 3128 端口运行 Squid
```

现在，你可以针对本地运行的代理来运行客户端。

```
ch8$ HTTP_PROXY=localhost:3128 go run proxyget.go https://www.google.com
Read ok
Response ok
<html itemscope=""...
```

这个程序通过环境变量 `HTTP_PROXY` 将代理信息传递给程序。应用程序获知代理信息的方式有很多种。大多数浏览器都有配置菜单，可以输入代理信息；但 Go 应用程序无法获取这些信息。一些应用程序可能通过网络代理自动发现协议（WPAD – [`https://en.wikipedia.org/wiki/Web_Proxy_Autodiscovery_Protocol`](https://en.wikipedia.org/wiki/Web_Proxy_Autodiscovery_Protocol)），通过网络中某个位置的 `proxy.pac` 文件来获取代理信息。Go 目前（尚）无法解析这些 JavaScript 文件，因此无法使用它们。特定的操作系统可能有系统特定的方式来指定代理，Go 也无法访问这些信息。但是，如果代理信息设置在操作系统环境变量中，例如 `HTTP_PROXY` 或 `http_proxy`，Go 可以通过以下函数找到它：

```
func ProxyFromEnvironment(req *Request) (*url.URL, error)
```

如果你的程序运行在这种环境中，可以使用此函数，而无需显式知道代理参数。更多信息请参阅 `go doc net/http ProxyFromEnvironment`。

### 认证代理

某些代理会要求提供用户名和密码进行身份验证，才能通过请求。一种常见的方案是“基本认证”，即将用户名和密码拼接成字符串 `"user:password"`，然后进行 Base64 编码。随后，通过 HTTP 请求头 `"Proxy-Authorization"` 将其发送给代理，并指明是基本认证。

以下程序 `proxyauthget.go` 演示了这一点，在之前的代理程序中添加了 `Proxy-Authentication` 头部：

```
ch8$ vi proxyauthget.go
/* ProxyAuthGet
*/
package main
import (
"encoding/base64"
"fmt"
"io"
"net/http"
"net/http/httputil"
"net/url"
"os"
)
const auth = "jannewmarch:mypassword"
func main() {
if len(os.Args) != 3 {
fmt.Println("Usage: ", os.Args[0], "http://proxy-host:port http://host:port/page")
os.Exit(1)
}
proxy := os.Args[1]
proxyURL, err := url.Parse(proxy)
checkError(err)
rawURL := os.Args[2]
url, err := url.Parse(rawURL)
checkError(err)
// 编码认证信息
basic := "Basic " +
base64.StdEncoding.EncodeToString([]byte(auth))
transport := &http.Transport{Proxy: http.ProxyURL(proxyURL)}
client := &http.Client{Transport: transport}
request, err := http.NewRequest("GET", url.String(), nil)
request.Header.Add("Proxy-Authorization", basic)
dump, _ := httputil.DumpRequest(request, false)
fmt.Println(string(dump))
// 发送请求
response, err := client.Do(request)
checkError(err)
fmt.Println("Read ok")
if response.Status != "200 OK" {
fmt.Println(response.Status)
os.Exit(2)
}
fmt.Println("Response ok")
var buf [512]byte
reader := response.Body
for {
n, err := reader.Read(buf[0:])
if err != nil {
os.Exit(0)
}
fmt.Print(string(buf[0:n]))
}
os.Exit(0)
}
func checkError(err error) {
if err != nil {
if err == io.EOF {
return
}
fmt.Println("Fatal error ", err.Error())
os.Exit(1)
}
}
```

这类程序似乎没有公开的测试站点。我在一个使用认证代理的工作环境中进行了测试。设置此类代理超出了本书的范围。关于如何设置，有一篇名为“如何使用用户名和密码基本认证设置 Squid 代理”的讨论（参见 [`http://stackoverflow.com/questions/3297196/how-to-set-up-a-squid-proxy-with-basic-username-and-password-authentication`](http://stackoverflow.com/questions/3297196/how-to-set-up-a-squid-proxy-with-basic-username-and-password-authentication)）。

当前代码硬编码了用户名和密码。如果使用错误的登录信息，可能会收到类似以下的错误：

```
ch8$ go run proxyauthget.go http://localhost:3128/ http://www.google.com
GET / HTTP/1.1
Host: www.google.com
Proxy-Authorization: Basic amphbm5ld21hcmNoOm15cGFzc3dvcmQ=
Read ok
407 Proxy Authentication Required
exit status 2
```

如果正常工作，你将收到与无需认证的代理类似的结果。

```
ch8$ go run proxyauthget.go http://localhost:3128/ http://www.google.com
GET / HTTP/1.1
Host: www.google.com
Proxy-Authorization: Basic amFubmV3bWFyY2g6bXlwYXNzd29yZA==
Read ok
Response ok
<html...
```



## 客户端的 HTTPS 连接

为了安全加密的连接，HTTP 使用了第 7 章中描述的 TLS 协议。HTTP+TLS 协议被称为 HTTPS，它使用 `https://` 的 URL 而非 `http://` 的 URL。

服务器必须在客户端接受其数据之前返回有效的 X.509 证书。如果证书有效，那么 Go 会在底层处理一切，前面给出的客户端在 HTTPS 的 URL 下也能正常运行。也就是说，像之前的 `clientget.go` 这样的程序无需修改即可运行——你只需给它一个 HTTPS 的 URL 即可。

许多站点拥有无效的证书。它们可能已过期，可能是自签名而非由公认的证书颁发机构签名，或者可能存在错误（例如服务器名称不正确）。像 Firefox 这样的浏览器会显示一个带有“赶紧离开这里！”按钮的巨大警告，但你可以自行承担风险继续访问，很多人也正是这样做的。

当遇到证书错误时，Go 目前会直接中止。不过，你可以配置客户端忽略证书错误。当然，不建议这样做——证书配置错误的站点可能存在其他问题。

在第 7 章中，我们生成了自签名的 X.509 证书。在本章稍后部分，我们将提供一个使用 X.509 证书的 HTTPS 服务器，如果使用了自签名证书，那么 `clientget.go` 将会产生以下错误：

```
x509: certificate signed by unknown authority
```

通过启用传输（Transport）配置标志 `InsecureSkipVerify`，可以创建一个忽略这些错误并继续执行的客户端。这个不安全的程序是 `tlsunsafeclientget.go`：

```
ch8$ vi tlsunsafeclientget.go
/* TLSUnsafeClientGet
*/
package main
import (
"crypto/tls"
"fmt"
"log"
"net/http"
"net/url"
"os"
"strings"
)
func main() {
if len(os.Args) != 2 {
log.Fatalln("Usage: ", os.Args[0], "https://host:port/page")
}
url, err := url.Parse(os.Args[1])
checkError(err)
if url.Scheme != "https" {
log.Fatalln("Not https scheme ", url.Scheme)
}
transport := &http.Transport{}
transport.TLSClientConfig = &tls.Config{InsecureSkipVerify: false}
client := &http.Client{Transport: transport}
request, err := http.NewRequest("GET", url.String(), nil)
// only accept UTF-8
checkError(err)
response, err := client.Do(request)
checkError(err)
if response.StatusCode != http.StatusOK {
log.Fatalln(response.Status)
}
fmt.Println("get a response")
chSet := getCharset(response)
fmt.Printf("got charset %s\n", chSet)
if chSet != "UTF-8" {
log.Fatalln("Cannot handle", chSet)
}
var buf [512]byte
reader := response.Body
fmt.Println("got body")
for {
n, err := reader.Read(buf[0:])
checkError(err)
fmt.Print(string(buf[0:n]))
}
}
func getCharset(response *http.Response) string {
contentType := response.Header.Get("Content-Type")
if contentType == "" {
// guess
return "UTF-8"
}
idx := strings.Index(contentType, "charset:")
if idx == -1 {
// guess
return "UTF-8"
}
return strings.Trim(contentType[idx:], " ")
}
func checkError(err error) {
if err != nil {
log.Fatalln("Fatal error ", err.Error())
}
}
```

在创建服务器端之前，让我们先尝试连接 `https://gooogle.com`（注意三个“o”），它可能不是有效的。

```
ch8$ go run tlsunsafeclientget.go https://gooogle.com
Fatal error  Get "https://gooogle.com": x509: certificate is valid for www.google.com, not gooogle.com
exit status 1
```

### 服务器端

构建客户端的另一面是处理 HTTP 请求的 Web 服务器。最简单——也是最早期——的服务器只是返回文件的副本。然而，在当前的服务器中，任何 URL 现在都可以触发任意的计算。

#### 文件服务器

我们从基本的文件服务器开始。Go 提供了一个*多路复用器*，这是一个读取并解释请求的对象。它将请求分发给 `handlers`（处理器），这些处理器在自己的线程中运行。因此，读取 HTTP 请求、解码请求以及在其各自的线程中分支到合适函数的大部分工作都由 Go 替我们完成了。

对于文件服务器，Go 还提供了一个 `FileServer` 对象，它知道如何从本地文件系统提供文件。它需要一个“根”目录（即本地系统中文件树的顶层）以及一个用于匹配 URL 的模式。最简单的模式是 `/`，它匹配任何 URL 的顶层。这将匹配所有 URL。

有了这些对象，一个从本地文件系统提供文件的 HTTP 服务器就变得异常简单了。这就是 `fileserver.go`：

```
ch8$ vi fileserver.go
/* File Server
*/
package main
import (
"log"
"net/http"
)
func main() {
// deliver files from the directory /tmp/www
fileServer := http.FileServer(http.Dir("/tmp/www"))
// register the handler and deliver requests to it
err := http.ListenAndServe(":8000", fileServer)
if err != nil {
log.Fatalln(err)
}
// That's it!
}
```

这个服务器甚至能为请求不存在的文件资源时提供 `"404 not found"` 消息！如果请求的文件是一个目录，它会返回一个由 `<pre> ... </pre>` 标签包裹的列表，并且不附带其他 HTML 头部或标记。如果使用 Wireshark 或一个简单的 telnet 客户端，目录会作为 `text/html` 发送，HTML 文件作为 `text/html`，Perl 文件作为 `text/x-perl`，Java 文件作为 `text/x-java`，以此类推。`FileServer` 使用了一些类型识别功能，并将其包含在 HTTP 请求中，但它无法提供像 Apache 这类服务器对标记的那种控制能力。

服务器和 curl 客户端的运行情况如下：

```
ch8$ go run FileServer.go
ch8$ curl localhost:8000
404 page not found
ch8$ mkdir -p /tmp/www/
ch8$ echo hi > /tmp/www/hi.txt
ch8$ curl localhost:8000/

hi.txt

ch8$ curl localhost:8000/hi.txt
hi
```



#### 处理函数

在上述最后一个程序中，处理函数作为第二个参数传递给了 `ListenAndServe`。可以通过先调用 `Handle` 或 `HandleFunc` 来注册任意数量的处理函数，其签名如下：

```
func Handle(pattern string, handler Handler)
func HandleFunc(pattern string, handler func(ResponseWriter, *Request))
```

`ListenAndServe` 的第二个参数可以为 `nil`，此时请求会被分派给所有已注册的处理函数。每个处理函数应拥有不同的 URL 模式。例如，文件处理函数的 URL 模式可能为 `/`，而函数处理函数的 URL 模式可能为 `/cgi-bin`（我们之前使用了 `/tmp/www`）。更具体的模式优先级高于更通用的模式。

常见的 CGI 程序有 `test-cgi`（用 shell 编写）和 `printenv`（用 Perl 编写），它们会打印环境变量的值。可以编写一个类似于 `printenv.go` 工作方式的处理函数。

```
ch8$ vi printenv.go
/* Print Env
*/
package main
import (
"fmt"
"net/http"
"os"
)
func main() {
// 大多数文件的文件处理函数
fileServer := http.FileServer(http.Dir("/tmp/www"))
http.Handle("/", fileServer)
// /cgi-bin/printenv 的函数处理函数
http.HandleFunc("/cgi-bin/printenv", printEnv)
// 将请求分派给处理函数
err := http.ListenAndServe(":8000", nil)
checkError(err)
// 就是这样!
}
func printEnv(writer http.ResponseWriter, req *http.Request) {
env := os.Environ()
writer.Write([]byte("Environment"))
for _, v := range env {
writer.Write([]byte(v + "\n"))
}
writer.Write([]byte(""))
}
func checkError(err error) {
if err != nil {
fmt.Println("Fatal error ", err.Error())
os.Exit(1)
}
}
```

按如下方式运行服务器：

```
ch8$ go run printenv.go
```

现在我们按如下方式运行一个 curl 客户端：

```
ch8$ curl localhost:8000/cgi-bin/printenv
EnvironmentTERM_PROGRAM=Apple_Terminal
SHELL=/bin/zsh
TERM=xterm-256color
TMPDIR=/var/folders/c5/l52zshy12q1bsfhp_sbdtg5r0000gn/T/
TERM_PROGRAM_VERSION=444
TERM_SESSION_ID=803B4A5C-F403-40E8-97CC-0EC807C35D77
USER=ronaldpetty
SSH_AUTH_SOCK=/private/tmp/com.apple.launchd.vj8cVxhj0c/Listeners
```

在这个程序中使用 `cgi-bin` 目录有点取巧：它不像 CGI 脚本那样调用外部程序。它只是调用了 Go 函数 `printEnv`。Go 确实具备使用 `os.ForkExec` 调用外部程序的能力，但尚不支持像 Apache 的 `mod_perl` 那样的动态可链接模块。在这种情况下，你很可能希望将结果包装成合适的 HTML。

#### 绕过默认多路复用器

Go 服务器接收到的 HTTP 请求通常由一个多路复用器处理，它会检查 HTTP 请求中的路径并调用相应的文件处理函数等。你可以定义自己的处理函数。这些处理函数可以通过调用 `http.HandleFunc` 注册到默认多路复用器，该函数接受一个模式和一个函数。像 `ListenAndServe` 这样的函数随后会接受一个 `nil` 处理函数。上一个示例就是这样做的。

然而，如果你想接管多路复用器的角色，那么可以将一个非 `nil` 的函数作为处理函数传递给 `ListenAndServe`。这个函数将负责管理请求和响应。

以下示例虽然简单，但说明了其用法。该多路复用器函数会针对 *所有* 对 `serverhandler.go` 的请求返回一个 `'204 No content'`：

```
ch8$ vi serverhandler.go
/* ServerHandler
*/
package main
import (
"net/http"
)
func main() {
myHandler := http.HandlerFunc(func(rw http.ResponseWriter,
request *http.Request) {
// 只返回无内容 - 可以设置任意请求头，任意响应体
rw.WriteHeader(http.StatusNoContent)
})
http.ListenAndServe(":8080", myHandler)
}
```

可以通过运行 `telnet` 来测试服务器，输出如下所示：

```
ch8$ curl -v localhost:8080
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET / HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.79.1
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 204 No Content
< Date: Fri, 01 Apr 2022 03:21:41 GMT
<
* Connection #0 to host localhost left intact
```

## HTTPS

对于安全的加密连接，HTTP 使用 TLS，这在第 7 章中有所描述。HTTP+TLS 协议称为 HTTPS，并使用 `https://` URL 而非 `http://` URL。

要使服务器使用 HTTPS，它需要一个 X.509 证书以及该证书的私钥文件。Go 目前要求这些文件采用第 7 章中使用的 PEM 编码。然后将 HTTP 函数 `ListenAndServe` 替换为 HTTPS（HTTP+TLS）函数 `ListenAndServeTLS`。

前面给出的文件服务器程序可以写成 HTTPS 服务器，即 `httpsfileserver.go`：

```
ch8$ vi httpsfileserver.go
/* HTTPSFileServer
*/
package main
import (
"net/http"
"log"
)
func main() {
// 从目录 /tmp/www 提供文件服务
fileServer := http.FileServer(http.Dir("/tmp/www"))
// 注册处理函数并将请求分派给它
err := http.ListenAndServeTLS(":8000", "jan.newmarch.name.pem",
"private.pem", fileServer)
if err != nil {
log.Fatalln(err)
}
}
```

例如，可通过 `https://localhost:8000/index.html` 访问此服务器。如果证书是自签名证书，则需要一个不安全的客户端来访问服务器内容。例如：

```
ch8$ go run httpsfileserver.go
ch8$ curl -i https://localhost:8000
curl: (60) SSL certificate problem: self signed certificate
More details here: https://curl.se/docs/sslcerts.html
curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```

我们可以通过 `-k` 标志指示 curl 客户端进行不安全连接。

```
ch8$ curl -ik https://localhost:8000
HTTP/2 200
content-type: text/html; charset=utf-8
last-modified: Mon, 27 Dec 2021 04:30:01 GMT
content-length: 41
date: Mon, 27 Dec 2021 04:46:56 GMT

hi.txt

```

我们也可以通过 Go 客户端代码来实现这一点（`tlsunsafeclientget.go`）。

当 `InsecureSkipVerify` 设置为 `true` 时：

```
ch8$ go run tlsunsafeclientget.go https://localhost:8000
get a response
got charset UTF-8
got body
```

当 `InsecureSkipVerify` 设置为 `false` 时：

```
ch8$ go run tlsunsafeclientget.go https://localhost:8000
Fatal error  Get "https://localhost:8000": x509: certificate signed by unknown authority
exit status 1
```

如果你想要一个同时支持 HTTP 和 HTTPS 的服务器，可以在各自的 `go` 例程中运行每个监听器。

## 结论

Go 对 HTTP 提供了广泛的支持。这并不奇怪，因为 Go 的部分初衷就是为了满足 Google 自身服务器的需求。本章讨论了 Go 为 HTTP 和 HTTPS 提供的不同级别的支持。在关于 Gorilla 的章节中，我们将更详细地讨论 Go 的多路复用请求（路径 -> 代码）。

# 9. 模板

大多数服务器端语言都有一种机制，可以获取主要是静态的页面并插入动态生成的组件，例如项目列表。典型的例子包括 Java Server Pages 中的脚本、PHP 脚本等。Go 在 `template` 包中采用了一种相对简单的脚本语言。

该包旨在将文本作为输入，并根据使用对象的值转换原始文本，输出不同的文本。与 JSP 或类似技术不同，它不局限于 HTML 文件，但很可能在该领域得到最大应用。我们首先描述 `text/template` 包，然后再介绍 `html/template` 包。

原始源代码称为*模板*，它由直接传递的文本和可以作用于文本并改变文本的内嵌命令组成。命令由 `{{ ... }}` 分隔，类似于 JSP 命令 `<%= ... =%>` 和 PHP 的 `<?php ... ?>`。在 Go 的模板模块中，命令也称为动作。



## 插入对象值

模板应用于 Go 对象。可以插入来自该 Go 对象的字段，并且可以“深入”对象以查找子字段等。当前对象用*光标*表示，因此要插入当前对象的值，请使用 `{{.}}`。默认情况下，该包使用 `fmt` 包来确定用作插入值的字符串。

要插入当前光标对象字段的值，请使用以 `.` 为前缀的字段名。例如，如果当前光标对象的类型是

```go
type Person struct {
    Name   string
    Age    int
    Emails []string
    Jobs   []*Job
}
```

您可以像下面这样插入 `Name` 和 `Age` 的值：

```
The name is {{.Name}}.
The age is {{.Age}}.
```

您可以使用 `range` 命令遍历数组或其他列表中的元素。因此，要访问 `Emails` 数组的内容，可以像这样：

```
{{range .Emails}}
The email is {{.}}
{{end}}
```

在遍历电子邮件的循环过程中，光标 `.` 依次被设置为每一封电子邮件。循环结束后，光标恢复为 person 对象。如果 `Job` 定义如下：

```go
type Job struct {
    Employer string
    Role     string
}
```

并且我们想要访问某个 `person` 的 `jobs` 的字段，我们可以像之前一样使用 `{{range .Jobs}}`。另一种方法是切换当前对象到 `Jobs` 字段。这可以通过 `{{with ...}} ... {{end}}` 结构来实现，此时 `{{.}}` 是 `Jobs` 字段，它是一个数组：

```
{{with .Jobs}}
{{range .}}
An employer is {{.Employer}}
and the role is {{.Role}}
{{end}}
{{end}}
```

您可以对任何字段使用此方法，而不仅仅限于数组。

### 使用模板

一旦您有了模板，就可以将其应用于对象，使用该对象填充模板值来生成新的字符串。这是一个两步过程：首先解析模板，然后将其应用于对象。结果输出到一个 `Writer`，如下所示：

```go
t := template.New("Person template")
t, err := t.Parse(templ)
if err == nil {
    buff := bytes.NewBufferString("")
    t.Execute(buff, person)
}
```

一个将模板应用于对象并打印到标准输出的示例程序是 `printperson.go`：

```go
$ mkdir ch9
ch9$ cd ch9
ch9$ vi printperson.go
/* PrintPerson
*/
package main
import (
    "log"
    "os"
    "text/template"
)
type Person struct {
    Name   string
    Age    int
    Emails []string
    Jobs   []Job
}
type Job struct {
    Employer string
    Role     string
}
const templ = `The name is {{.Name}}.
The age is {{.Age}}.
{{range .Emails}}
An email is {{.}}
{{end}}
{{with .Jobs}}
{{range .}}
An employer is {{.Employer}}
and the role is {{.Role}}
{{end}}
{{end}}
`
func main() {
    job1 := Job{Employer: "Box Hill Institute", Role: "Director, Commerce and ICT"}
    job2 := Job{Employer: "Canberra University", Role: "Adjunct Professor"}
    person := Person{
        Name:   "jan",
        Age:    66,
        Emails: []string{"jan@newmarch.name",
            "jan.newmarch@gmail.com"},
        Jobs:   []Job{job1, job2},
    }
    t := template.New("Person template")
    t, err := t.Parse(templ)
    checkError(err)
    err = t.Execute(os.Stdout, person)
    checkError(err)
}
func checkError(err error) {
    if err != nil {
        log.Fatalln("Fatal error ", err.Error())
    }
}
```

输出结果如下：

```
ch9$ go run printperson.go
The name is jan.
The age is 66.
An email is jan@newmarch.name
An email is jan.newmarch@gmail.com
An employer is Box Hill Institute
and the role is Director, Commerce and ICT
An employer is Canberra University
and the role is Adjunct Professor
```

请注意，此打印输出中存在大量空白（换行符）。这是由于我们在模板中使用的空白所致。如果您想减少这些空白，可以像下面这样消除模板中的换行符：

```
{{range .Emails}} An email is {{.}} {{end}}
```

另一种方法是使用命令分隔符 `{{-` 和 `-}}`，分别消除紧接在前的文本的所有尾部空白和紧接在后的文本的所有前导空白。

在示例中，我们在程序中使用了一个字符串作为模板。您也可以使用 `template.ParseFiles()` 函数从文件加载模板。出于我无法理解的原因（并且在早期版本中不需要），分配给模板的名称必须与文件列表中第一个文件的基本名称相同。这是一个 bug 吗？

### 管道

上述转换将文本片段插入到模板中。这些文本片段本质上是任意的，取决于字段的字符串值。如果我们希望它们作为 HTML 文档（或其他专门形式）的一部分出现，则必须转义特定的字符序列。例如，要在 HTML 文档中显示任意文本，我们必须将 `<` 改为 `&lt;`。Go 模板有许多内置函数，其中之一是 `html()`。这些函数的作用类似于 UNIX 管道，从标准输入读取并写入标准输出。

要获取当前对象 `.` 的值并对其应用 HTML 转义，您可以在模板中编写一个“管道”：

```
{{. | html}}
```

这是另一个例子，我们添加了一条格式化消息，说明一个人有多少份工作。

```go
const templ = `The name is {{.Name}}.
The age is {{.Age}}.
{{range .Emails}}
An email is {{.}}
{{end}}
{{with .Jobs}}
{{range .}}
An employer is {{.Employer}}
and the role is {{.Role}}
{{end}}
{{ . | len | printf "%d jobs total" }}
{{end}}
`
```

运行修改后的先前示例，我们现在会看到一行新的输出：

```
go run printperson.go
...
2 jobs total
```

其他函数也类似。管道传输时还有其他注意事项，包括参数如何传递。您可以通过以下链接了解有关在模板中使用管道的更多信息：[`https://pkg.go.dev/text/template#hdr-Pipelines`](https://pkg.go.dev/text/template#hdr-Pipelines)。



## 定义函数

模板使用对象的字符串表示形式，通过 `fmt` 包将对象转换为字符串来插入值。但有时这并非所需。例如，为避免垃圾邮件发送者获取电子邮件地址，通常会将符号 `@` 替换为单词“at”，如“jan at newmarch.name”。如果我们想使用模板以这种形式显示电子邮件地址，就必须构建一个自定义函数来完成此转换。

每个模板函数都有一个在模板中使用的名称，以及一个关联的 Go 函数。它们通过以下类型进行关联：

```
type FuncMap map[string]interface{}
```

例如，如果我们希望模板函数名为 `emailExpand`，并与 Go 函数 `EmailExpander` 关联，我们可以按照以下方式将其添加到模板的函数中：

```
t = t.Funcs(template.FuncMap{"emailExpand": EmailExpander})
```

`EmailExpander` 的签名通常是这样的：

```
func EmailExpander(args ...interface{}) string
```

在我们感兴趣的用例中，该函数应只有一个参数，且该参数为字符串。Go 模板库中的现有函数有一些初始代码来处理不符合规范的情况，因此我们只需照搬即可。然后只需进行简单的字符串操作来更改电子邮件地址的格式。程序文件是 `printemails.go`：

```go
ch9$ vi printemails.go
/* PrintEmails
*/
package main
import (
"log"
"os"
"strings"
"text/template"
)
type Person struct {
Name   string
Emails []string
}
const templ = `The name is {{.Name}}.
{{range .Emails}}
An email is "{{. | emailExpand}}"
{{end}}`
func main() {
person := Person{
Name: "jan",
Emails: []string{"jan@newmarch.name",
"jan.newmarch@gmail.com"},
}
t, err := template.New("Person template").Funcs(
template.FuncMap{
"emailExpand": func(emailAddress string) string {
return strings.Replace(emailAddress, "@", " at ", -1)
},
},
).Parse(templ)
err = t.Execute(os.Stdout, person)
checkError(err)
}
func checkError(err error) {
if err != nil {
log.Fatalln("Fatal error ", err.Error())
}
}
```

输出结果如下：

```
ch9$ go run printemails.go
The name is jan.
An email is "jan at newmarch.name"
An email is "jan.newmarch at gmail.com"
```

## 变量

模板包允许你定义和使用变量。为了说明其动机，请考虑如何打印每个人的电子邮件地址，并*在其前面加上*姓名。我们使用的类型仍然是这样：

```
type Person struct {
Name      string
Emails    []string
}
```

为了访问电子邮件字符串，我们使用如下的 `range` 语句：

```
{{range .Emails}}
{{.}}
{{end}}
```

但在此时，我们无法访问 `Name` 字段，因为 `.` 当前正在遍历数组元素，而 `Name` 不在这个作用域内。解决方案是将 `Name` 字段的值保存在一个变量中，该变量可以在其作用域内的任何地方被访问。我们将同样的思路应用于循环变量。模板中的变量以 `$` 为前缀。因此我们这样写：

```
{{$name := .Name}}
{{range $idx, $email := .Emails}}
Name is {{$name}}, email is {{$email}}
{{end}}
```

程序文件是 `printnameemails.go`：

```go
ch9$ vi printnameemails.go
/**
* PrintNameEmails
*/
package main
import (
"log"
"os"
"text/template"
)
type Person struct {
Name   string
Emails []string
}
const templ = `{{$name := .Name}}
{{ $numEmails := .Emails | len -}}
{{range $idx, $email := .Emails -}}
Name is {{$name}}, email {{$email}} is {{ $idx | increment }} of {{ $numEmails }}
{{end}}
`
func main() {
person := Person{
Name: "jan",
Emails: []string{"jan@newmarch.name",
"jan.newmarch@gmail.com"},
}
t, err := template.New("Person template").Funcs(
template.FuncMap{
"increment": func(val int) int {
return val + 1
},
},
).Parse(templ)
checkError(err)
err = t.Execute(os.Stdout, person)
checkError(err)
}
func checkError(err error) {
if err != nil {
log.Fatalln("Fatal error ", err.Error())
}
}
```

输出结果如下：

```
ch9$ go run printnameemails.go
Name is jan, email jan@newmarch.name is 1 of 2
Name is jan, email jan.newmarch@gmail.com is 2 of 2
```



### 条件语句

继续使用 `Person` 示例，假设你只想打印出邮件列表，而不深入查看。你可以通过模板来实现：

```
Name is {{.Name}}
Emails are {{.Emails}}
```

这将输出以下内容：

```
Name is jan
Emails are [jan@newmarch.name jan.newmarch@gmail.com]
```

因为这是 `fmt` 包显示列表的方式。

在许多情况下，如果这正是你想要的，那可能没问题。让我们考虑一种*几乎*正确但又不完全正确的情况。有一个用于序列化对象的 JSON 包，我们在第 4 章中介绍过。它会生成以下内容：

```
{"Name": "jan",
"Emails": ["jan@newmarch.name", "jan.newmarch@gmail.com"]
}
```

JSON 包是你在实践中使用的，但让我们看看是否可以使用模板生成 JSON 输出。我们仅通过已有的模板就能做一些类似的事情。作为 JSON 序列化器，这*几乎*是正确的：

```
{"Name": "{{.Name}}",
"Emails": {{.Emails}}
}
```

它将生成：

```
{"Name": "jan",
"Emails": [jan@newmarch.name jan.newmarch@gmail.com]
}
```

这有两个问题：地址没有加引号，并且列表元素应该用 `,` 分隔。

这样如何——查看数组元素，给它们加上引号，并添加逗号？

```
{"Name": {{.Name}},
"Emails": [
{{range .Emails}}
"{{.}}",
{{end}}
]
}
```

这将生成：

```
{"Name": "jan",
"Emails": ["jan@newmarch.name", "jan.newmarch@gmail.com",]
}
```

（外加一些空白符）。

同样，它几乎正确，但如果你仔细看，会发现最后一个列表元素后面多了一个尾随的 `,`。根据 JSON 语法（参见 [`http://www.json.org/`](http://www.json.org/)），这个尾随的 `,` 是不允许的。不同的实现处理方式可能有所不同。

我们想要的是打印每个元素，除了最后一个元素外，每个元素后面都跟一个 `,`。这实际上有点难办，所以更好的方法是打印每个元素，除了*第一个*元素外，每个元素*前面*都加一个 `,`（这个技巧来自 Stack Overflow 上的 "brianb" – [`http://stackoverflow.com/questions/201782/can-you-use-a-trailing-comma-in-a-json-object`](http://stackoverflow.com/questions/201782/can-you-use-a-trailing-comma-in-a-json-object)）。这更容易，因为第一个元素的索引是零，而许多编程语言（包括 Go 模板语言）将零视为布尔值 `false`。

条件语句的一种形式是 `{{if pipeline}} T1 {{else}} T0 {{end}}`。我们需要 `pipeline` 是邮件数组的索引。幸运的是，`range` 语句的变体提供了这个功能。有两种引入变量的形式：

```
{{range $elmt := array}}
{{range $index, $elmt := array}}
```

因此，我们遍历数组，如果索引是 `false`（0），则只打印元素。否则，我们在元素前面加上 `,` 再打印。模板如下：

```
{"Name": "{{.Name}}",
"Emails": [
{{range $index, $elmt := .Emails}}
{{if $index}}
, "{{$elmt}}"
{{else}}
"{{$elmt}}"
{{end}}
{{end}}
]
}
```

完整的程序是 `printjsonemails.go`：

```
ch9$ vi printjsonemails.go
/**
* PrintJSONEmails
*/
package main
import (
"bytes"
"encoding/json"
"fmt"
"log"
"os"
"text/template"
)
type Person struct {
Name   string
Emails []string
}
const templ = `{"Name": "{{- .Name -}}", "Emails": [
{{- range $index, $elmt := .Emails -}}
{{- if $index -}}
, "{{- $elmt -}}"
{{- else -}}
"{{- $elmt -}}"
{{- end -}}
{{- end -}}
] }`
func main() {
person := Person{
Name: "jan",
Emails: []string{"jan@newmarch.name",
"jan.newmarch@gmail.com"},
}
t := template.New("Person template")
t, err := t.Parse(templ)
checkError(err)
err = t.Execute(os.Stdout, person)
checkError(err)
// check via validity json package
var b bytes.Buffer
err = t.Execute(&b, person)
checkError(err)
if json.Valid(b.Bytes()) {
fmt.Println("\nvalid json")
}
}
func checkError(err error) {
if err != nil {
log.Fatalln("Fatal error ", err.Error())
}
}
```

这给出了正确的 JSON 输出，它还运行了 `json.Valid()` 检查函数来验证是否为有效的 JSON。

```
ch9$
go run printjsonemails.go
{"Name": "jan", "Emails": ["jan@newmarch.name", "jan.newmarch@gmail.com"] }
valid json
```

在结束本节之前，请注意，格式化带逗号分隔符的列表的问题，可以通过在 Go 中定义合适的函数来解决，这些函数作为模板函数可用。借用另一种编程语言的一句名言：“条条大路通罗马！”以下程序由 Roger Peppe 发送给我，名为 `sequence.go`：

```
ch9$ vi sequence.go
/* Sequence.go
* Copyright Roger Peppe
*/
package main
import (
"errors"
"fmt"
"os"
"text/template"
)
var tmpl = `{{$comma := sequence "" ", "}}
{{range $}}{{$comma.Next}}{{.}}{{end}}
{{$comma := sequence "" ", "}}
{{$colour := cycle "black" "white" "red"}}
{{range $}}{{$comma.Next}}{{.}} in {{$colour.Next}}{{end}}
`
var fmap = template.FuncMap{
"sequence": sequenceFunc,
"cycle":    cycleFunc,
}
func main() {
t, err := template.New("").Funcs(fmap).Parse(tmpl)
if err != nil {
fmt.Printf("parse error: %vn", err)
return
}
err = t.Execute(os.Stdout, []string{"a", "b", "c",
"d", "e", "f"})
if err != nil {
fmt.Printf("exec error: %vn", err)
}
}
type generator struct {
ss []string
i  int
f  func(s []string, i int) string
}
func (seq *generator) Next() string {
s := seq.f(seq.ss, seq.i)
seq.i++
return s
}
func sequenceGen(ss []string, i int) string {
if i >= len(ss) {
return ss[len(ss)-1]
}
return ss[i]
}
func cycleGen(ss []string, i int) string {
return ss[i%len(ss)]
}
func sequenceFunc(ss ...string) (*generator, error) {
if len(ss) == 0 {
return nil, errors.New("sequence must have at least one element")
}
return &generator{ss, 0, sequenceGen}, nil
}
func cycleFunc(ss ...string) (*generator, error) {
if len(ss) == 0 {
return nil, errors.New("cycle must have at least one element")
}
return &generator{ss, 0, cycleGen}, nil
}
```

输出如下：

```
ch9$ go run sequence.go
a, b, c, d, e, f
a in black, b in white, c in red, d in black, e in white, f in red
```

## html/template 包

前面的程序都处理的是 `text/template` 包。它执行转换，而不考虑文本可能使用的任何上下文。例如，如果 `PrintPerson.go` 中的文本变为

```
job1 := Job{Employer: "alert('Could be nasty!')", Role: "Director, Commerce and ICT"}
```

程序将生成以下文本：

```
An employer is alert('Could be nasty!')
```

如果下载到浏览器，这将导致意外效果。

在管道中使用 `html` 命令可以减少这种情况，例如 `{{. | html}}`，并将生成以下内容：

```
An employer is &lt;script&gt;alert('Could be nasty!')&lt;/script&gt
```

将这种过滤器应用于所有表达式会变得繁琐。此外，它可能无法捕获潜在的、危险的 JavaScript、CSS 或 URI 表达式。

`html/template` 包旨在解决这些问题。只需将 `text/template` 替换为 `html/template` 这一简单步骤，就会对生成的文本应用适当的转换，对其进行净化，使其适用于 Web 上下文。

当使用 `go doc` 查看任一模板包时，请务必注意您正在查看的是哪一个。`go doc template` 实际上是 `go doc html/template`，而 `go doc text/template` 是用于非 HTML 包的。

## 总结

Go 模板包对于某些涉及插入对象值的文本转换非常有用。例如，它没有正则表达式的强大功能，但速度更快，并且在许多情况下，比正则表达式更容易使用。



# 10. 完整的 Web 服务器

本章主要作为 HTTP 章节的实践应用，将使用标准 Go 语言构建一个完整的网站。同时介绍如何使用模板在文本文件中嵌入表达式以插入变量值，并生成重复内容区域。内容涉及序列化数据和 Unicode 字符集。本章中的程序篇幅较长且结构复杂，因此不会完整列出所有代码，但可从本书的 GitHub 网站[`https://github.com/Apress/network-prog-with-go-2e`](https://github.com/Apress/network-prog-with-go-2e)下载。

Jan 正在学习中文。更准确地说，经过多年尝试，他仍然在*尝试*学习中文。当然，他没有静下心来好好学，而是尝试了各种技术辅助工具。他试过课本、视频和许多其他教学材料。最终他意识到进步缓慢的原因是*没有一款好用的中文闪卡程序*，因此出于学习需要，他决定自己开发一个。

他找到一个 Python 程序完成了部分功能。但遗憾的是，该程序编写质量不佳，经过几次彻底改造尝试后，他得出结论：从头开始编写更好。当然，网络解决方案远比独立程序优越，因为这样中文班的其他同学以及所有学习者都能共享使用。而服务器将用 Go 语言编写。

他采用了张朋朋所著《汉语口语速成》（Sinolingua 出版社，2007 年，ISBN 978-7-80052577-3）中的词汇，但该程序适用于任何词汇集。

## 浏览器界面示意图

在浏览器中查看的最终程序包含三种页面，如图 10-1 所示。

`![](img/436770_2_En_10_Fig1_HTML.png)`

图 10-1

浏览器页面

主页显示闪卡集列表（见图 10-2）。包含当前可用的闪卡集列表、展示方式（随机卡牌顺序、优先显示中文或英文、或随机显示）、以及选择展示整套卡片还是仅显示单词列表。

`![](img/436770_2_En_10_Fig2_HTML.jpg)`

图 10-2

网站主页

闪卡集每次显示一张卡片，样式如图 10-3 所示。

`![](img/436770_2_En_10_Fig3_HTML.jpg)`

图 10-3

显示所有组件的典型闪卡

闪卡集中的单词列表如图 10-4 所示。

`![](img/436770_2_En_10_Fig4_HTML.jpg)`

图 10-4

闪卡集中的单词列表

## 浏览器端文件

浏览器端包含 HTML、CSS 和 JavaScript 文件，以及托管它们的 Go 代码。逻辑路径及相关文件如下：

* 主页路径（`/` 和 `/flashcards.html`）：
  * `css/listflashcardsstylesheet.css`
* 闪卡集路径（`showflashcards.html`）：
  * `css/cardstylesheet.css`
  * `jscript/jquery.js`
  * `jscript/slideviewer.js`
* 闪卡集单词路径（`listwords.html`）：
  * `css/listflashcardsstylesheet.css`
  * `jscript/sortable.js`

整体项目结构如下：

```
$ mkdir ch10
$ cd ch10
ch10$ tree .
.
├── cedict_ts.u8
├── css
│   ├── cardstylesheet.css
│   └── listflashcardsstylesheet.css
├── dictionary.go
├── flashcards.go
├── flashcardsets
│   ├── common_words
│   ├── lesson_04_surname_first_name
│   ├── lesson_05_country_nationality
│   └── lesson_06_city_native_place
├── html
│   ├── listflashcards.html
│   ├── listwords.html
│   └── showflashcards.html
├── jscript
│   ├── jquery.js
│   ├── slideviewer.js
│   └── sorttable.js
├── pinyinformatter.go
└── server.go
4 directories, 17 files
```

## 基础服务器

服务器是上一章讨论的 HTTP 服务器。它包含多个处理不同 URL 的函数。函数概述如下：

| 路径 | 函数 | 传递的 HTML |
| --- | --- | --- |
| `/` | `listFlashCards` | `html/listflashcards.html` |
| `/flashcards.html` | `listFlashCards` | `html/listflashcards.html` |
| `/flashcardSets` | `manageFlashCards` | `html/showflashcards.html` |
| `/flashcardSets` | `manageFlashCards` | `html/listwords.html` |
| `/jscript/*` | `fileServer` | 来自 `/jscript` 目录的文件 |
| `/html/*/css/*` | `fileServer` | 来自 `/html` 目录 / `/css` 目录的文件 |

服务器文件为 `server.go`，位于 [`https://github.com/Apress/network-prog-with-go-2e`](https://github.com/Apress/network-prog-with-go-2e) 的 `ch10` 目录下。

```
ch10$ cat server.go
/* Server
*/
package main
import (
"fmt"
"html/template"
"log"
"net/http"
"os"
)
const (
DefaultSet           = "common_words"
DefaultAmount        = "Random"
ActionShow           = "Show cards in set"
ActionList           = "List words in set"
ActionUnknown        = "Unknown action"
URLFlashCardSetsPath = "flashcardSets"
FlashCardPage        = "flashcards.html"
ListFlashCardPage    = "list" + FlashCardPage
ShowFlashCardPage    = "show" + FlashCardPage
ListWordsPage        = "listwords.html"
CardOrderSequential  = "Sequential"
CardOrderRandom      = "Random"
)
var showHalf = map[string]string{
"Random":  "RANDOM_HALF",
"English": "ENGLISH_HALF",
"Chinese": "CHINESE_HALF",
}
func main() {
if len(os.Args) != 2 {
log.Fatalln("Usage: ", os.Args[0], ":port")
}
port := os.Args[1]
http.HandleFunc("/", listFlashCards)
fileServer := http.StripPrefix("/jscript/", http.FileServer(http.Dir("jscript")))
http.Handle("/jscript/", fileServer)
fileServer = http.StripPrefix("/html/", http.FileServer(http.Dir("html")))
http.Handle("/html/", fileServer)
fileServer = http.StripPrefix("/css/", http.FileServer(http.Dir("css")))
http.Handle("/css/", fileServer)
http.HandleFunc("/"+FlashCardPage, listFlashCards)
http.HandleFunc("/"+URLFlashCardSetsPath, manageFlashCards)
// deliver requests to the handlers
err := http.ListenAndServe(port, nil)
checkError(err)
}
func listFlashCards(rw http.ResponseWriter, req *http.Request) {
...
}
/*
* Called from listflashcards.html on form submission
*/
func manageFlashCards(rw http.ResponseWriter, req *http.Request) {
...
}
func showFlashCards(rw http.ResponseWriter, cardname, order, half string) {
...
}
func listWords(rw http.ResponseWriter, cardname string) {
...
}
func httpErrorHandler(rw http.ResponseWriter, err error) {
http.Error(rw, err.Error(), http.StatusInternalServerError)
}
func checkError(err error) {
if err != nil {
log.Fatalln("Fatal error ", err.Error())
}
}
```

接下来我们将讨论各个函数的具体实现。



### listFlashCards 函数

`listFlashCards` 函数用于生成顶层页面的 HTML。闪卡名称列表是可扩展的，由 `flashcardSets` 目录下的文件条目集合构成。该列表用于创建顶层页面中的表格，最好使用模板包来实现：

```
Flashcard Sets

{{range $i, $e := .}}

{{$e}}

{{end}}
```

其中 range 循环遍历名称列表。文件 `html/listflashcards.html` 包含了该模板，以及卡片顺序侧边列表、半卡显示和底部表单按钮的 HTML。省略侧边列表和提交按钮后，HTML 如下所示：

```
ch10$ cat html/listflashcards.html

Flashcards

Flashcards

Flashcard Sets

{{range $i, $e := .}}

{{$e}}

{{end}}

Card order

Random 

Sequential 

Half card display

Random 

English 

Chinese 

Show cards in set

List words in set
```

在 `server.go` 中，`listFlashCards` 函数将模板应用于此，代码如下：

```
func listFlashCards(rw http.ResponseWriter, req *http.Request) {
flashCardsNames := ListFlashCardsNames()
t, err := template.ParseFiles("html/" + ListFlashCardPage)
if err != nil {
httpErrorHandler(rw, err)
return
}
t.Execute(rw, flashCardsNames)
}
```

在 `flashcards.go` 中，`ListFlashCardsNames()` 函数遍历闪卡目录，返回一个字符串数组（每个闪卡集的文件名）：

```
func ListFlashCardsNames() []string {
flashcardsDir, err := os.Open("flashcardsets")
if err != nil {
return nil
}
files, err := flashcardsDir.Readdir(-1)
fileNames := make([]string, len(files))
for n, f := range files {
fileNames[n] = f.Name()
}
sort.Strings(fileNames)
return fileNames
}
```

### manageFlashCards 函数

在 `server.go` 中，按下“Show Cards in Set”按钮或“List Words in Set”按钮时，会调用 `manageFlashCards` 函数来管理表单提交。它从表单请求中提取值，然后在 `showFlashCards` 和 `listWords` 之间做出选择：

```
/*
* 在表单提交时从 listflashcards.html 调用
*/
func manageFlashCards(rw http.ResponseWriter, req *http.Request) {
set := req.FormValue("flashcardSets")
order := req.FormValue("order")
action := req.FormValue("submit")
half := req.FormValue("half")
//如果未设置
//http://localhost:8000/flashcardSets?flashcardSets=common_words&order=Random&half=Random&submit=Show+cards+in+set
if len(set) == 0 {
set = DefaultSet
order = DefaultAmount
action = ActionShow
half = DefaultAmount
}
cardname := URLFlashCardSetsPath + "/" + set
fmt.Printf("Set %s, order %s, action %s, half %s, cardname %s\n", set, order, action, half, cardname)
switch action {
case ActionShow:
showFlashCards(rw, cardname, order, half)
case ActionList:
listWords(rw, cardname)
default:
fmt.Println(ActionUnknown)
}
}
```

## 中文词典

之前的代码相当通用：使用 `FileServer` 提供静态文件，基于目录中的文件列表通过模板生成 HTML 表格，并处理 HTML 表单中的信息。要进一步探究每张卡片显示的内容，我们需要深入研究应用特定的细节，这意味着要查看单词的来源（词典）、如何表示单词和卡片，以及如何将闪卡数据发送到浏览器。首先，来看词典。

中文是一门复杂的语言——难道不都是吗 :-( 。书面形式是表意的，即“象形文字”，而不是使用字母表。但这种书面形式随着时间的推移而演变，甚至在最近分成了两种形式：台湾和香港使用的“繁体”中文，以及中国大陆使用的“简体”中文。虽然大多数汉字是相同的，但大约有 1000 个汉字不同。因此，一本中文词典通常会有同一个汉字的两种书写形式。

大多数像我这样的西方人看不懂这些汉字。因此有一种称为拼音的“拉丁化”形式，它使用基于拉丁字母的注音字母来书写汉字。它不完全等同于拉丁字母，因为中文是一种声调语言，拼音形式必须标出声调（类似于法语和其他欧洲语言中的重音符号）。因此，一本典型的词典需要显示四项内容：繁体字、简体字、拼音和英文。此外（就像英语一样），一个词可能有多种含义。例如，[`http://www.mandarintools.com/worddict.html`](http://www.mandarintools.com/worddict.html) 上有一个免费的中/英词典，更好的是，你可以将其下载为 UTF-8 文件。其中，词“好”有这样的条目：

| **繁体** | **简体** | **拼音** | **英文** | **含义** |
| --- | --- | --- | --- | --- |
| 好 | 好 | hǎo | good | /好/良好/适当/易于/非常/如此/(表示完成或准备好的后缀)/ |

这本词典有一个小问题。大多数键盘不擅长表示变音符号，比如 ǎ 中的抑扬符。因此，虽然汉字是用 Unicode 编写的，但拼音字符却不是。尽管存在诸如 ǎ 等字母的 Unicode 字符，但许多词典（包括本词典）都使用拉丁字母 `a`，并将声调放在词尾。这里，它是第三声，所以 `hǎo` 被写为 `hao3`。这使得只有美式键盘且没有 Unicode 编辑器的人也能更容易地用拼音交流。Web 服务器使用的词典副本是 `cedict_ts.u8`。

这种数据格式的不匹配并不是大问题。只是在原始文本词典和浏览器显示之间的某个环节，需要进行数据转换。Go 模板允许通过定义自定义模板来完成这一点，所以我选择了这条路。其他方法包括在读取词典时进行处理，或在 JavaScript 中显示最终字符。

### 词典类型

在 `dictionary.go` 中，我们使用 `DictionaryEntry` 来存储一个词的基本信息：

```
type DictionaryEntry struct {
Traditional  string
Simplified   string
Pinyin       string
Translations []string
}
```

前面的词“好”将由以下内容表示：

```
DictionaryEntry {Traditional: 好,
Simplified: 好,
Pinyin: `hao3`
Translations: []string{`good`, `well`,`proper`,
`good to`, `easy to`, `very`, `so`,
`(suffix indicating completion or readiness)`}
}
```

词典本身只是这些条目的一个数组：

```
type Dictionary struct {
Entries []*DictionaryEntry
}
```



### 闪卡组

一张闪卡用于表示一个中文词语及其英文翻译。我们已经看到，一个中文词语可能有多种英文含义。但这部词典有时也会多次出现同一个中文词语。例如，“好”至少出现两次，一次是我们已经见过的含义，另一次则是“喜爱”的意思。虽然这显得有些多余，但为了应对这种情况，每张闪卡都附带了一整本词典。通常情况下，词典中只有一个词条！闪卡的其余部分只是简体字和英文单词，作为可能的检索键，代码来自 `flashcards.go`：

```
type FlashCard struct {
    Simplified string
    English    string
    Dictionary *Dictionary
}
```

闪卡组（*set*）是这些闪卡的数组，外加组名以及将发送到浏览器用于展示的信息：随机或固定顺序，先显示每张卡的正面或背面，或者随机顺序，代码来自 `flashcards.go`：

```
type FlashCards struct {
    Name      string
    CardOrder string
    ShowHalf  string
    Cards     []*FlashCard
}
```

我们已经展示过该类型的一个函数——`ListFlashCardsNames()`。该类型还有一个加载闪卡组 JSON 文件的函数。它使用了第 4*章中提到的技术，代码在 `flashcards.go` 中：

```
func LoadJSON(r io.Reader, key any) {
    decoder := json.NewDecoder(r)
    err := decoder.Decode(key)
    checkError(err)
}
```

一个典型的闪卡组包含常用词语。当 JSON 文件被美化打印（例如使用 jq）时，部分内容如下所示：

```
{
    "ShowHalf":"",
    "Cards":[
        {
            "Simplified":"\u4f60\u597d",
            "Dictionary":{
                "Entries":[
                    {
                        "Traditional":"\u4f60\u597d",
                        "Pinyin":"ni3 hao3",
                        "Translations":[
                            "hello",
                            "hi",
                            "how are you?"
                        ],
                        "Simplified":"\u4f60\u597d"
                    }
                ]
            },
            "English":"hello"
        },
        {
            "Simplified":"\u5582",
            "Dictionary":{
                "Entries":[
                    {
                        "Traditional":"\u5582",
                        "Pinyin":"wei4",
                        "Translations":[
                            "hello (interj., esp. on telephone)",
                            "hey",
                            "to feed (sb or some animal)"
                        ],
                        "Simplified":"\u5582"
                    }
                ]
            },
            "English":"hello (interj., esp. on telephone)"
        },
    ],
    "CardOrder":"",
    "Name":"Common Words"
}
```

### 修复音调

在完成服务器代码之前，还有最后一项主要任务。词典中给出的音调标记位于拼音单词的末尾，例如 `hao3` 对应 hǎo。可以使用自定义模板将其转换为 Unicode，如第 9 章所述。

以下是拼音格式化器的代码。除非你*真的*对了解拼音格式化规则感兴趣，否则不必费心阅读。程序是 `pinyinformatter.go`：

```
ch10$ cat pinyinformatter.go
package main
import (
    "fmt"
    "strings"
)
func PinyinFormatter(args ...interface{}) string {
    ok := false
    var s string
    if len(args) == 1 {
        s, ok = args[0].(string)
    }
    if !ok {
        s = fmt.Sprint(args...)
    }
    fmt.Println("Formatting func " + s)
    // the string may consist of several pinyin words
    // each one needs to be changed separately and then
    // added back together
    words := strings.Fields(s)
    for n, word := range words {
        // convert "u:" to "ü" if present
        uColon := strings.Index(word, "u:")
        if uColon != -1 {
            parts := strings.SplitN(word, "u:", 2)
            word = parts[0] + "ü" + parts[1]
        }
        println(word)
        // get last character, will be the tone if present
        chars := []rune(word)
        tone := chars[len(chars)-1]
        if tone == '5' {
            // there is no accent for tone 5
            words[n] = string(chars[0 : len(chars)-1])
            println("lost accent on", words[n])
            continue
        }
        if tone  '4' {
            // not a tone value
            continue
        }
        words[n] = addAccent(word, int(tone))
    }
    s = strings.Join(words, ` `)
    return s
}
var (
    // maps 'a1' to '\u0101' etc
    aAccent = map[int]rune{
        '1': '\u0101',
        '2': '\u00e1',
        '3': '\u01ce',
        '4': '\u00e0'}
    eAccent = map[int]rune{
        '1': '\u0113',
        '2': '\u00e9',
        '3': '\u011b',
        '4': '\u00e8'}
    iAccent = map[int]rune{
        '1': '\u012b',
        '2': '\u00ed',
        '3': '\u01d0',
        '4': '\u00ec'}
    oAccent = map[int]rune{
        '1': '\u014d',
        '2': '\u00f3',
        '3': '\u01d2',
        '4': '\u00f2'}
    uAccent = map[int]rune{
        '1': '\u016b',
        '2': '\u00fa',
        '3': '\u01d4',
        '4': '\u00f9'}
    üAccent = map[int]rune{
        '1': 'ǖ',
        '2': 'ǘ',
        '3': 'ǚ',
        '4': 'ǜ'}
)
func addAccent(word string, tone int) string {
    /*
    * Based on "Where do the tone marks go?"
    * at http://www.pinyin.info/rules/where.html
    */
    n := strings.Index(word, "a")
    if n != -1 {
        aAcc := aAccent[tone]
        // replace 'a' with its tone version
        word = word[0:n] + string(aAcc) + word[(n+1):len(word)-1]
    } else {
        n := strings.Index(word, "e")
        if n != -1 {
            eAcc := eAccent[tone]
            word = word[0:n] + string(eAcc) +
                word[(n+1):len(word)-1]
        } else {
            n = strings.Index(word, "ou")
            if n != -1 {
                oAcc := oAccent[tone]
                word = word[0:n] + string(oAcc) + "u" +
                    word[(n+2):len(word)-1]
            } else {
                chars := []rune(word)
                length := len(chars)
                // put tone onthe last vowel
            L:
                for n, _ := range chars {
                    m := length - n - 1
                    switch chars[m] {
                    case 'i':
                        chars[m] = iAccent[tone]
                        break L
                    case 'o':
                        chars[m] = oAccent[tone]
                        break L
                    case 'u':
                        chars[m] = uAccent[tone]
                        break L
                    case 'ü':
                        chars[m] = üAccent[tone]
                        break L
                    default:
                    }
                }
                word = string(chars[0 : len(chars)-1])
            }
        }
    }
    return word
}
```

### ListWords 函数

现在我们可以返回到服务器中剩余的函数了。一个是列出闪卡组中的单词。它使用闪卡组的模板填充一个 HTML 表格。该 HTML 使用模板包遍历 `FlashCards` 结构体并插入其中的字段：

```
ch10$ cat html/listwords.html

Words for {{.Name}}

Words for {{.Name}}

 English 
 Pinyin 
 Traditional 
 Simplified 

{{range .Cards}}

{{.English}}

{{with .Dictionary}}
{{range .Entries}}

{{.Pinyin|pinyin}}

{{.Traditional}}

{{.Simplified}}

{{end}}
{{end}}

{{end}}

 Return to Flash Cards list

```

`server.go` 中实现此功能的 Go 函数使用了上一节讨论的 `PinyinFormatter`：

```
func listWords(rw http.ResponseWriter, cardname string) {
    fmt.Println("Loading card name", cardname)
    cards := new(FlashCards)
    LoadJSON(cardname, cards)
    fmt.Println("loaded cards", len(cards.Cards))
    fmt.Println("Card name", cards.Name)
    t := template.New("listwords.html")
    t = t.Funcs(template.FuncMap{"pinyin": PinyinFormatter})
    t, err := t.ParseFiles("html/listwords.html")
    if err != nil {
        fmt.Println("Parse error " + err.Error())
        http.Error(rw, err.Error(), http.StatusInternalServerError)
        return
    }
    err = t.Execute(rw, cards)
    if err != nil {
        fmt.Println("Execute error " + err.Error())
        http.Error(rw, err.Error(), http.StatusInternalServerError)
        return
    }
}
```

这将填充好的表格发送到浏览器，如图 10-4 所示。


好的，作为高级文档工程师和翻译员，我将严格按照您提供的格式和要求，将给定的英文文本翻译成中文。


## `showFlashCards` 函数

完成服务器的最后一个函数是 `showFlashCards`。此函数根据浏览器提交的表单，更改卡片组中 `CardOrder` 和 `ShowHalf` 的默认值。然后，它应用 `PinyinFormatter` 并将生成的文档发送到浏览器。我捕获了使用 UNIX 命令 `script` 进行的命令行会话的输出，然后运行了以下命令：

```
GET /flashcardSets?flashcardSets=Common+Words&order=Random&half=Chinese&submit=Show+cards+in+set HTTP/1.0
```

结果的一部分如下所示：

```
ch10$ cat ./html/showflashcards.html

Flashcards for {{.Name}}

cardOrder = {{- .CardOrder }};
showHalfCard = {{- .ShowHalf }};

 -->

Flashcards for {{.Name}}

{{range .Cards}}

English: {{.English}}

{{with .Dictionary}}
{{range .Entries}}

Pinyin: {{.Pinyin|pinyin}}

Traditional: {{.Traditional}}

Simplified: {{.Simplified}}

Translations:
{{range .Translations}}
{{.}} 
{{end}}

{{end}}
{{end}}

{{end}}

Press &lt;Space&gt; or Tap to continue

 Return to Flash Cards list

```

实际的函数如下所示。我们可以看到卡片组、展示顺序和渲染，这与之前的函数非常相似。

```
func showFlashCards(rw http.ResponseWriter, cardname, order, half string) {
cards := new(FlashCards)
content, err := os.Open(cardname)
checkError(err)
LoadJSON(content, &cards)
switch order {
case CardOrderSequential:
cards.CardOrder = "SEQUENTIAL"
default:
cards.CardOrder = "RANDOM"
}
if v, ok := showHalf[half]; ok {
cards.ShowHalf = v
} else {
cards.ShowHalf = showHalf["Chinese"]
}
fmt.Printf("Loading card %s, half %s, loaded # of %d, card name %s\n", cardname, half, len(cards.Cards), cards.Name)
t, err := template.New(ShowFlashCardPage).Funcs(template.FuncMap{"pinyin": PinyinFormatter}).ParseFiles("html/" + ShowFlashCardPage)
if err != nil {
httpErrorHandler(rw, err)
return
}
err = t.Execute(rw, cards)
if err != nil {
httpErrorHandler(rw, err)
return
}
}
```

除了渲染之外，我们还有基本的错误处理：`checkError` 用于处理严重错误（足以导致关闭），而 `httpErrorHandler` 则在我们希望指示 HTTP 500 状态时使用。

### 浏览器中的呈现

本系统的最后一部分是此 HTML 在浏览器中的显示方式。图 10-3 展示了一个由四部分组成的屏幕，分别显示英语、简体中文、备用翻译以及繁简对照。这是通过下载到服务器的 JavaScript 程序（使用 `FileServer` Go 对象完成）实现的。`slideviewer.js` JavaScript 文件实际上相当长，因此在文中省略。它包含在位于 [`https://github.com/Apress/network-prog-with-go-2e`](https://github.com/Apress/network-prog-with-go-2e) 的程序文件中。

### 运行服务器

然后，可以使用如下命令在端口 `8000`（或其他端口）上运行服务器：

```
ch10$ go run *.go :8000
```

或

```
ch10$ go run server.go pinyinformatter.go flashCards.go dictionary.go :8000
```

## 总结

本章介绍了一个相对简单但完整的 Web 服务器，它使用了静态和动态网页，并包含表单处理，同时利用模板来简化编码。

# 11. HTML

万维网最初是为了提供 HTML 文档而创建的。现在，它被用来提供各种文档以及不同类型的数据。尽管如此，HTML 仍然是网络上传输的主要文档类型。

HTML 经历了大量版本迭代，当前版本是 HTML5。还有许多“供应商”版本的 HTML，它们引入了一些从未成为标准的标签。

HTML 足够简单，可以手动编辑。因此，许多 HTML 文档是“格式错误的”，这意味着它们不符合该语言的语法。HTML 解析器通常不是很严格，会接受许多“非法的”文档。

`html` 包本身只有两个函数：`EscapeString` 和 `UnescapeString`。它们能正确处理诸如 `<` 之类的字符，将其转换为 `&lt;` 再转换回来。

此函数的一个主要用途可能是对 HTML 文档中的标记进行转义，这样当它在浏览器中显示时，会展示所有标记（很像 Linux 版 Chrome 中的 `Ctrl+U` 或 Mac 版 Chrome 中的 `Option+Cmd+U`）。

我更倾向于使用它来将程序的文本显示为网页。大多数编程语言都有 `<` 符号，许多语言还有 `&`。如果不进行适当的转义，这些字符会扰乱 HTML 查看器。我喜欢直接从文件系统中显示程序文本，而不是复制粘贴到文档中，以避免不同步。

下面的 `escapestring.go` 程序是一个 Web 服务器，它以预格式化代码的形式显示其 URL，并对棘手的字符进行了转义：

```
$ mkdir ch11
$ cd ch11
ch11$ vi escapestring.go
/*
* This program serves a file in preformatted, code layout
* Useful for showing program text, properly escaping special
* characters like '' and '&'
*/
package main
import (
"fmt"
"html"
"log"
"net/http"
"os"
)
func main() {
http.HandleFunc("/", escapeString)
err := http.ListenAndServe(":8080", nil)
checkError(err)
}
func escapeString(rw http.ResponseWriter, req *http.Request) {
fmt.Println(req.URL.Path)
bytes, err := os.ReadFile("." + req.URL.Path)
if err != nil {
rw.WriteHeader(http.StatusNotFound)
return
}
escapedStr := html.EscapeString(string(bytes))
htmlText := "" +
escapedStr +
" "
rw.Write([]byte(htmlText))
}
func checkError(err error) {
if err != nil {
log.Fatalln("Error ", err.Error())
}
}
```

当它运行时，从包含 `escapestring.go` 程序的目录提供文件服务，浏览器将使用 URL `localhost:8080/escapestring.go` 正确显示该文件。

使用以下命令运行服务器：

```
ch11$ go run escapestring.go
```

例如，使用以下命令（或在浏览器中）运行客户端：

```
ch11$ curl localhost:8080/escapestring.go
/*
* This program serves a file in preformatted, code layout
...
}

```

在这两种情况下，结果都是我们原始的代码（包裹在一个 pre/code 块中）！如果您输错了名称，将会收到 HTTP 404 错误。



## `html/template` 包

网络服务器可能遭受多种攻击，最著名的是 SQL 注入攻击，即用户代理在网页表单中输入特意设计的数据，这些数据会传入数据库并造成严重破坏。Go 语言并没有提供特别的机制来避免这种情况，因为不同数据库在 SQL 注入技术方面存在诸多差异，导致成功的攻击方式各不相同。SQL 注入防御速查表（参见 [`https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html`](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)）总结了针对此类攻击的防御措施。其中最主要的方法是使用 SQL *预编译语句*来避免此类攻击，这可以通过 `database/sql` 包中的 `Prepare` 函数实现。

更隐蔽的攻击基于 XSS——跨站脚本攻击。攻击者不是试图攻击网站本身，而是在服务器上存储恶意代码来攻击该网站的任何客户端。

这些攻击基于向数据库字符串中插入数据，当这些数据（例如）被传送给浏览器时，将攻击浏览器，并通过浏览器攻击网站客户端。（关于此攻击的几种变体，请参见“OWASP: XSS 的类型”—— [`https://owasp.org/www-community/Types_of_Cross-Site_Scripting`](https://owasp.org/www-community/Types_of_Cross-Site_Scripting)。）

例如，在请求发表博客评论的位置可能被插入 JavaScript 代码，将浏览器重定向到攻击者的网站：

```
window.location='http://attacker/'
```

Go 的 `html/template` 包是在 `text/template` 包的基础上构建的。该包假定模板本身是可信的，但模板处理的数据可能不可信。`html/template` 包增加的功能是对数据进行适当的转义，以消除 XSS 的可能性。它基于 Mike Samuel 和 Prateek Saxena 撰写的论文“Using Type Inference to Make Web Templates Robust Against XSS”。请访问 [`https://rawgit.com/mikesamuel/sanitized-jquery-templates/trunk/safetemplate.html#problem_definition`](https://rawgit.com/mikesamuel/sanitized-jquery-templates/trunk/safetemplate.html#problem_definition) 阅读该论文，以了解该包背后的理论以及包文档本身。

简而言之，按照 `text/template` 包的方式准备模板，如果生成的文本要交付给 HTML 代理，则使用 `html/template` 包。

## HTML 词法分析

Go 子仓库中的 `golang.org/x/net/html` 包包含一个 HTML 词法分析器。它允许你构建一个 HTML 词法单元的解析树。该包符合 HTML5 标准。

下面是一个在 `readhtml.go` 中使用该包的示例程序。

```
ch11$ vi readhtml.go
/* Read HTML
*/
package main
import (
"fmt"
"golang.org/x/net/html"
"io"
"log"
"os"
"strings"
)
func main() {
if len(os.Args) != 2 {
log.Fatalln("Usage: ", os.Args[0], "file")
}
file := os.Args[1]
bytes, err := os.ReadFile(file)
checkError(err)
r := strings.NewReader(string(bytes))
z := html.NewTokenizer(r)
depth := 0
for {
tt := z.Next()
for n := 0; n < depth; n++ {
fmt.Print(" ")
}
switch tt {
case html.ErrorToken:
if z.Err() == io.EOF {
fmt.Println("EOF")
} else {
fmt.Println("Error ", z.Err().Error())
}
os.Exit(0)
case html.TextToken:
fmt.Println("Text: \"" + z.Token().String() + "\"")
case html.StartTagToken, html.EndTagToken:
fmt.Println("Tag: \"" + z.Token().String() + "\"")
if tt == html.StartTagToken {
depth++
} else {
depth--
}
}
}
}
func checkError(err error) {
if err != nil {
log.Fatalln("Fatal error ", err.Error())
}
}
```

当我们对一个像 `sample.html` 这样简单的 HTML 文档运行 `readhtml.go` 时：

```
ch11$ vi sample.html

 Test HTML 

 Header one 

Test para

```

在设置好依赖项后，它会产生以下输出：

```
$ go mod init example.com/user/readhtml
$ go mod tidy
$ go run readhtml.go sample.html
Tag: ""
Text: "
"
Tag: ""
Text: "
"
Tag: ""
Text: " Test HTML "
Tag: ""
Text: "
"
Tag: ""
Text: "
"
Tag: ""
Text: "
"
Tag: ""
Text: " Header one "
Tag: ""
Text: "
"
Tag: ""
Text: "
Test para
"
Tag: ""
Text: "
"
Tag: ""
Text: "
"
Tag: ""
Text: "
"
EOF
```

（它产生的所有空白字符都是正确的。）

## XHTML/HTML

在 XML 包中也对 XHTML/HTML 提供了有限的支持，这将在下一章讨论。

## JSON

正如第 4 章所述，Go 对 JSON 提供了良好的支持。

## 结论

这个包本身内容不多。其子包 `html/template` 已在第 9 章关于模板的部分讨论过。

# 12. XML

XML 是一种重要的标记语言，主要旨在作为一种使用文本格式表示结构化数据的手段。用我们在第 4 章使用的术语来说，它可以被视为一种将数据结构序列化为文本文档的方式。它用于描述诸如 DocBook 和 XHTML 之类的文档。它用于专门的标记语言，如 MathML 和 CML（化学标记语言）。它用于将数据编码为 Web 服务的 SOAP 消息，并且 Web 服务可以使用 WSDL（Web 服务描述语言）来指定。

在最简单的层面上，XML 允许你在文本文档中定义自己的标签。标签可以嵌套，也可以与文本交错。每个标签还可以包含带有值的属性。例如，文件 `person.xml` 可能包含：

```
$ mkdir ch12
$ cd ch12
ch12$ vi person.xml

 Newmarch 
 Jan 

jan@newmarch.name

j.newmarch@boxhill.edu.au

```

任何 XML 文档的结构可以通过多种方式描述：

*   文档类型定义 (DTD) 擅长描述结构。
*   XML 模式擅长描述 XML 文档所使用的数据类型。
*   RELAX NG 被提出作为这两种方式的替代方案。

对于每种定义 XML 文档结构方式的相对价值存在争论。我们不打算深入参与这场争论，因为 Go 语言并不支持其中任何一种。Go 无法根据某种模式检查文档的有效性，而只能检查其格式是否良好。即使是格式良好性也是 XML 文档的一个重要特征，并且在实际中 HTML 文档经常在这方面出现问题。这使得 XML 即使对于非常复杂的数据也适合进行表示，而 HTML 则不行。

本章讨论四个主题：将 Go 数据编组和解编组为 XML、解析 XML 流，以及 XHTML。



## 解组 XML

Go 提供了一个名为 `Unmarshal` 的函数，用于将 XML 解组到 Go 数据结构中。解组并非完美无缺：Go 和 XML 是不同的语言。

在深入细节之前，我们先看一个简单的例子。首先，考虑前面给出的 XML 文档（`person.xml`）：

```xml
<person>
<name>
<family>Newmarch</family>
<personal>Jan</personal>
</name>
<email type="personal">
jan@newmarch.name
</email>
<email type="work">
j.newmarch@boxhill.edu.au
</email>
</person>
```

我们希望将其映射到 Go 结构体：

```go
type Person struct {
    Name  Name
    Email []Email
}
type Name struct {
    Family   string
    Personal string
}
type Email struct {
    Type    string
    Address string
}
```

这需要几点说明：

- 解组使用 Go 的反射包。这要求所有字段都必须是导出的，即以大写字母开头。早期版本的 Go 使用不区分大小写的匹配来匹配字段，例如将 XML 字符串 `"name"` 匹配到字段 `Name`。但现在使用的是*区分大小写*的匹配。为了执行匹配，结构体字段必须添加标签，以指明要匹配的 XML 字符串。这会将 `Person` 更改为以下形式：

```go
type Person struct {
    Name  Name    `xml:"name"`
    Email []Email `xml:"email"`
}
```

- 虽然字段标签可以将 XML 字符串附加到字段上，但它无法对结构体名称进行此操作。需要一个额外的字段，字段名为 `XMLName`。这只影响顶层结构体 `Person`：

```go
type Person struct {
    XMLName Name    `xml:"person"`
    Name    Name    `xml:"name"`
    Email   []Email `xml:"email"`
}
```

- 重复的标签映射到 Go 中的切片。

- 标签内的属性仅当 Go 字段具有标签 `,attr` 时才会匹配到结构体中的字段。`Email` 的 `Type` 字段就是这种情况，匹配 `email` 标签的 `type` 属性需要 `xml:"type,attr"`。

- 如果 XML 标签没有属性且仅包含字符数据，那么它将以相同的名称（但区分大小写）匹配一个 `string` 字段。因此，标签 `xml:"family"` 及其字符数据 `Newmarch` 映射到字符串字段 `Family`。

- 但如果标签有属性，则必须映射到一个结构体。Go 将字符数据分配给具有标签 `,chardata` 的字段。`email` 数据和具有标签 `,chardata` 的字段 `Address` 就是这种情况。

解组上述文档的程序是 `unmarshal.go`：

```go
/*
ch12$ vi unmarshal.go
/* Unmarshal
*/
package main
import (
"encoding/xml"
"fmt"
"log"
)
type Person struct {
XMLName Name    `xml:"person"`
Name    Name    `xml:"name"`
Email   []Email `xml:"email"`
}
type Name struct {
Family   string `xml:"family"`
Personal string `xml:"personal"`
}
type Email struct {
Type    string `xml:"type,attr"`
Address string `xml:",chardata"`
}
func main() {
str := `
<person>
<name>
<family>Newmarch</family>
<personal>Jan</personal>
</name>
<email type="personal">
jan@newmarch.name
</email>
<email type="work">
j.newmarch@boxhill.edu.au
</email>
</person>
`
var person Person
err := xml.Unmarshal([]byte(str), &person)
checkError(err)
// now use the person structure e.g.
fmt.Println("Family name: \"" + person.Name.Family + "\"")
for _, email := range person.Email {
fmt.Println(email)
}
}
func checkError(err error) {
if err != nil {
log.Fatalln("Fatal error ", err.Error())
}
}
/*
ch12$ go run unmarshal.go
Family name: " Newmarch "
{personal
jan@newmarch.name
}
{work
j.newmarch@boxhill.edu.au
}
*/
```

（注意空格是正确的。）严格的规则在包规范中给出；请参阅 `go doc -all encoding/xml`。

## 编组 XML

Go 也支持将数据结构编组到 XML 文档中。该函数是：

```go
func Marshal(v interface}{) ([]byte, error)
```

一个编组简单结构的程序是 `marshal.go`：

```go
/*
ch12$ vi marshal.go
/* Marshal
*/
package main
import (
"encoding/xml"
"fmt"
)
type Person struct {
XMLName xml.Name `xml:"person"`
Name    Name     `xml:"name"`
Email   []Email  `xml:"email"`
}
type Name struct {
Family   string `xml:"family"`
Personal string `xml:"personal"`
}
type Email struct {
Kind    string "attr"
Address string "chardata"
}
func main() {
person := Person{
Name: Name{Family: "Newmarch", Personal: "Jan"},
Email: []Email{Email{Kind: "home", Address: "jan"},
Email{Kind: "work", Address: "jan"}}}
buff, _ := xml.Marshal(person)
fmt.Println(string(buff))
}
/*
ch12$ go run marshal.go
NewmarchJanhomejanworkjan
*/
```

它会产生没有空白的文本。

## 解析 XML

Go 有一个 XML 解析器，可以使用 `encoding/xml` 包中的 `NewDecoder` 创建。该函数接受一个 `io.Reader` 作为参数，并返回一个指向 `Decoder` 的指针。该类型的主要方法是 `Token`，它返回输入流中的下一个令牌。令牌是以下类型之一：`StartElement`、`EndElement`、`CharData`、`Comment`、`ProcInst` 或 `Directive`。

XML 类型是 `StartElement`、`EndElement`、`CharData`、`Comment`、`ProcInst` 和 `Directive`。下面将对其进行描述。

### StartElement 类型

`StartElement` 类型是一个具有两个字段类型的结构体：

```go
type StartElement struct {
    Name Name
    Attr []Attr
}
```

其中：

```go
type Attr struct {
    Name  Name
    Value string
}
```

### EndElement 类型

这也是一个结构体，如下所示：

```go
type EndElement struct {
    Name Name
}
```

### CharData 类型

此类型表示标签括起来的文本内容，是一个简单类型：

```go
type CharData []byte
```

### Comment 类型

类似地，对于注释，此类型与 `CharData` 的类型相似：

```go
type Comment []byte
```

### ProcInst 类型

`ProcInst` 表示形式为 `<?target inst?>` 的 XML 处理指令：

```go
type ProcInst struct {
    Target string
    Inst   []byte
}
```

### Directive 类型

`Directive` 表示形式为 `<!text>` 的 XML 指令。字节不包括 `<!` 和 `>` 标记。

```go
type Directive []byte
```

一个打印 XML 文档树结构的程序是 `parsexml.go`：

```go
/*
ch12$ vi parsexml.go
/* Parse XML
*/
package main
import (
"encoding/xml"
"fmt"
"io/ioutil"
"log"
"os"
"strings"
)
func main() {
if len(os.Args) != 2 {
log.Fatalln("Usage: ", os.Args[0], "file")
}
file := os.Args[1]
bytes, err := ioutil.ReadFile(file)
checkError(err)
r := strings.NewReader(string(bytes))
parser := xml.NewDecoder(r)
depth := 0
for {
token, err := parser.Token()
if err != nil {
break
}
switch elmt := token.(type) {
case xml.StartElement:
name := elmt.Name.Local
printElmt(name+":start", depth)
depth++
case xml.EndElement:
depth--
name := elmt.Name.Local
printElmt(name+":end", depth)
case xml.CharData:
printElmt(string([]byte(elmt)), depth)
case xml.Comment:
printElmt("Comment", depth)
case xml.ProcInst:
printElmt("ProcInst", depth)
case xml.Directive:
printElmt("Directive", depth)
default:
fmt.Println("Unknown")
}
}
}
func printElmt(s string, depth int) {
slimS := strings.TrimSpace(s)
if len(slimS) == 0 {
return
}
for n := 0; n < depth; n++ {
fmt.Print("  ")
}
fmt.Println(slimS)
}
func checkError(err error) {
if err != nil {
log.Fatalln("Fatal error ", err.Error())
}
}
*/
```

请注意，解析器会包含所有 `CharData`，包括标签之间的空白。

如果我们对前面给出的 `person` 数据结构运行 `parsexml.go` 程序，如下所示：

```
ch12$ go run parsexml.go person.xml
person:start
name:start
family:start
Newmarch
family:end
personal:start
Jan
personal:end
name:end
email:start
jan@newmarch.name
email:end
email:start
j.newmarch@boxhill.edu.au
email:end
person:end
```

请注意，由于没有使用 DTD 或其他 XML 规范，分词器正确地打印出了所有空白。（DTD 可能指定可以忽略空白，但如果没有 DTD，则不能做此假设。）为了使输出更美观，我们通过 `Trim` 移除了额外的空格。

使用此解析器存在一个潜在的陷阱。它会重用字符串的空间，因此一旦你看到一个令牌，如果之后还想引用它，就需要复制其值。Go 提供了诸如 `func (c CharData) Copy() CharData` 之类的方法来复制数据；请参阅 `go doc encoding.xml.Copy`。



## XHTML

HTML 并不符合 XML 语法规范。它存在未闭合的标签，例如 `<br>`。XHTML 是对 HTML 的清理，使其符合 XML 规范。因此，XHTML 文档可以使用之前提到的 XML 管理技术。XHTML 的普及程度似乎并未达到最初的预期。我个人猜测，这是因为 HTML 解析器通常容错性较好，在浏览器中通常能合理地渲染文档；而即使是浏览器中的 XHTML 解析器也往往更为严格，一旦遇到任何 XML 错误，通常就无法渲染任何内容。这对于面向用户的软件来说，并非合适的行为。

## HTML

`xml` 包提供了一定程度支持，用于处理可能不符合 XML 规范的 HTML 文档。通过关闭严格的解析检查，之前讨论的 XML 解析器可以处理许多 HTML 文档。

```
parser := xml.NewDecoder(r)
parser.Strict = false
parser.AutoClose = xml.HTMLAutoClose
parser.Entity = xml.HTMLEntity
```

## 结论

Go 语言对处理 XML 字符串提供了基本支持。目前它还没有处理 XML 模式或 RELAX NG 等 XML 规范语言的机制。

## 远程过程调用

Socket 和 HTTP 编程都使用消息传递范式。客户端向服务器发送消息，服务器通常再发送回一条消息。双方都负责以双方都能理解的格式创建消息，并从这些消息中读取数据。

然而，大多数独立的应用程序并不大量使用消息传递技术。通常，首选机制是*函数*（或方法、过程）调用。在这种风格中，程序将使用一组参数调用一个函数，并在函数调用完成后获得一组返回值。这些值可能是函数返回值，或者，如果地址作为参数传递，那么这些地址所指向的内容可能会被改变。

远程过程调用（RPC）是尝试将这种编程风格引入网络世界。因此，客户端将进行一个看起来像是普通过程调用的操作。客户端会将此调用打包成网络消息并发送给服务器。服务器将解包此消息，并在服务器端将其恢复为一个过程调用。此调用的结果将被打包以返回给客户端。

图解如下，如图 13-1 所示。

![](img/436770_2_En_13_Fig1_HTML.png)

**图 13-1** 远程过程调用步骤

步骤如下：

1.  客户端调用客户端过程存根。存根将参数打包成网络消息。此过程称为*编组*。

2.  存根调用操作系统内核中的网络例程来发送消息。

3.  内核将消息发送到远程系统。这可以是面向连接或无连接的。

4.  服务器端过程存根从网络消息中解组参数。

5.  服务器端过程存根执行服务器过程实现。

6.  过程执行完毕，将执行权返回给服务器端过程存根。

7.  服务器端存根将返回值编组到网络消息中。

8.  返回消息被发送回去。

9.  客户端过程存根使用网络例程读取消息。

10. 消息被解组，返回值被设置在栈上以提供给客户端程序。

实现 RPC 有两种常见风格。第一种以 Sun 的 ONC/RPC 和 CORBA 为代表。在这种风格中，服务的规范是用某种抽象语言（例如 CORBA IDL，即接口定义语言）给出的。然后，这被编译成客户端和服务器的代码。客户端随后编写一个普通的程序，其中包含对过程/函数/方法的调用，并与生成的客户端代码链接。服务器端代码本身就是一个服务器，它与您编写的过程实现相链接。

在这种第一种方式中，客户端代码在外观上与普通过程调用几乎相同。通常，会有少量额外的代码用于定位服务器。在 Sun 的 ONC 中，必须知道服务器的地址；在 CORBA 中，调用命名服务来查找服务器的地址；在 Java RMI 中，IDL 就是 Java 本身，并使用命名服务来查找服务的地址。

在第二种风格中，您必须使用一个特殊的客户端 API。您在客户端将此函数名称及其参数交给这个库。在服务器端，您必须显式地自己编写服务器以及远程过程实现。

许多 RPC 系统（如 Web 服务）都采用第二种方法。Go 语言的 RPC 也采用这种方法。

## Go 语言的 RPC

迄今为止，Go 语言的 RPC 是 Go 独有的。它与其他 RPC 系统不同，因此一个 Go 客户端只能与一个 Go 服务器通信。它使用第 4 章讨论的 Gob 序列化系统，该系统定义了可以使用的数据类型。

RPC 系统通常会对可以跨网络调用的函数做出一些限制。这样 RPC 系统才能正确确定发送哪些值参数，接收哪些引用参数的答案，以及如何发出错误信号。

在 Go 语言中，限制如下：

*   方法的类型是导出的（以大写字母开头）。

*   方法是导出的。

*   该方法有两个参数，这两个参数都是导出（或内置）类型。第一个用于传入方法的数据；第二个用于返回结果。

*   方法的第二个参数是一个指针。

*   它的返回值类型是 `error`。

例如，以下是一个有效的函数：

```
F(T1, &T2) error
```

对参数的限制意味着您通常需要定义一个结构体类型。Go 语言的 RPC 使用 `gob` 包进行数据的编组和解组，因此参数类型必须遵循前面章节讨论的 Gob 规则。

我们将遵循 Go 文档中给出的示例，因为它说明了重点。服务器执行两个简单的操作——它们不需要 RPC 的“重活”，但易于理解。这两个操作是将两个整数相乘，以及求第一个整数除以第二个整数后的商和余数。

要操作的两个值被定义在一个结构体中：

```
type Values struct {
A, B int
}
```

乘积只是一个 `int` 类型，而商/余数是另一个结构体：

```
type Quotient struct {
Quo, Rem int
}
```

我们将有两个函数，`multiply` 和 `divide`，可以在 RPC 服务器上被调用。这些函数需要向 RPC 系统注册。`Register` 函数接受一个参数，该参数是一个接口。因此我们需要一个包含这两个函数的类型：

```
type Arith int
func (t *Arith) Multiply(args *Args, reply *int) error {
*reply = args.A * args.B
return nil
}
func (t *Arith) Divide(args *Args, quo *Quotient) error {
if args.B == 0 {
return errors.New("divide by zero")
}
quo.Quo = args.A / args.B
quo.Rem = args.A % args.B
return nil
}
```

`Arith` 的底层类型被指定为 `int`。这没关系——任何类型都可以。

现在可以使用 `Register` 注册此类型的对象，然后 RPC 系统就可以调用其方法了。



### HTTP RPC 服务器

任何 RPC 都需要一种传输机制来通过网络传递消息。Go 可以使用 HTTP 或 TCP。HTTP 机制的优点在于可以利用 HTTP 支持库。你需要向 HTTP 层添加一个 RPC 处理器，通过 `HandleHTTP` 实现，然后启动一个 HTTP 服务器。完整代码在 `arithserver.go` 中：

```
$ mkdir ch13
$ cd ch13
ch13$ vi arithserver.go
/* ArithServer
*/
package main
import (
"errors"
"fmt"
"net/http"
"net/rpc"
)
type Values struct {
A, B int
}
type Quotient struct {
Quo, Rem int
}
type Arith int
func (t *Arith) Multiply(args *Values, reply *int) error {
*reply = args.A * args.B
return nil
}
func (t *Arith) Divide(args *Values, quo *Quotient) error {
if args.B == 0 {
return errors.New("divide by zero")
}
quo.Quo = args.A / args.B
quo.Rem = args.A % args.B
return nil
}
func main() {
arith := new(Arith)
rpc.Register(arith)
rpc.HandleHTTP()
err := http.ListenAndServe(":1234", nil)
if err != nil {
fmt.Println(err.Error())
}
}
```

并通过以下命令运行：

```
ch13$ go run arithserver.go
```

让服务器保持运行。接下来，我们来看 RPC 服务的用户，即 RPC 客户端。

### HTTP RPC 客户端

客户端需要与 RPC 服务器建立 HTTP 连接。它需要准备一个结构体，包含要发送的值以及用于存储结果的变量地址。然后，它可以使用这些参数发起一个 `Call`：

*   要执行的远程函数名称
*   要发送的值
*   用于存储结果的变量地址

一个调用算术服务器两个函数的客户端是 `arithclient.go`：

```
ch13$ vi arithclient.go
/* ArithClient
*/
package main
import (
"fmt"
"log"
"net/rpc"
"os"
)
type Args struct {
A, B int
}
type Quotient struct {
Quo, Rem int
}
func main() {
if len(os.Args) != 2 {
fmt.Println("Usage: ", os.Args[0], "server")
os.Exit(1)
}
serverAddress := os.Args[1]
client, err := rpc.DialHTTP("tcp", serverAddress+":1234")
if err != nil {
log.Fatal("dialing:", err)
}
// Synchronous call
args := Args{17, 8}
var reply int
err = client.Call("Arith.Multiply", args, &reply)
if err != nil {
log.Fatal("arith error:", err)
}
fmt.Printf("Arith: %d*%d=%d\n", args.A, args.B, reply)
var quot Quotient
err = client.Call("Arith.Divide", args, &quot)
if err != nil {
log.Fatal("arith error:", err)
}
fmt.Printf("Arith: %d/%d=%d remainder %d\n", args.A, args.B,
quot.Quo, quot.Rem)
}
```

运行时，我们看到以下输出：

```
ch13$ go run arithclient.go localhost
Arith: 17*8=136
Arith: 17/8=2 remainder 1
```

### TCP RPC 服务器

使用 TCP 套接字的服务器版本是 `tcparithserver.go`：

```
ch13$ vi tcparithserver.go
/* TCPArithServer
*/
package main
import (
"errors"
"log"
"net"
"net/rpc"
)
type Args struct {
A, B int
}
type Quotient struct {
Quo, Rem int
}
type Arith int
func (t *Arith) Multiply(args *Args, reply *int) error {
*reply = args.A * args.B
return nil
}
func (t *Arith) Divide(args *Args, quo *Quotient) error {
if args.B == 0 {
return errors.New("divide by zero")
}
quo.Quo = args.A / args.B
quo.Rem = args.A % args.B
return nil
}
func main() {
arith := new(Arith)
rpc.Register(arith)
tcpAddr, err := net.ResolveTCPAddr("tcp", ":1234")
checkError(err)
listener, err := net.ListenTCP("tcp", tcpAddr)
checkError(err)
/* This works:
rpc.Accept(listener)
*/
/* and so does this:
*/
for {
conn, err := listener.Accept()
if err != nil {
continue
}
rpc.ServeConn(conn)
}
}
func checkError(err error) {
if err != nil {
log.Fatalln("Fatal error ", err.Error())
}
}
```

请注意，对 `Accept` 的调用是阻塞的，仅用于处理客户端连接。如果服务器还想执行其他工作，则应在 `go` 协程中调用此函数。

按如下方式启动服务器：

```
ch13$ go run tcparithserver.go
```

和之前一样，现在我们来看相关的客户端。

### TCP RPC 客户端

一个使用 TCP 服务器并调用算术服务器两个函数的客户端是 `tcparithclient.go`：

```
ch13$ vi tcparithclient.go
/* TCPArithClient
*/
package main
import (
"fmt"
"log"
"net/rpc"
"os"
)
type Args struct {
A, B int
}
type Quotient struct {
Quo, Rem int
}
func main() {
if len(os.Args) != 2 {
log.Fatalln("Usage: ", os.Args[0], "server:port")
}
service := os.Args[1]
client, err := rpc.Dial("tcp", service)
if err != nil {
log.Fatalln("dialing:", err)
}
// Synchronous call
args := Args{17, 8}
var reply int
err = client.Call("Arith.Multiply", args, &reply)
if err != nil {
log.Fatalln("arith error:", err)
}
fmt.Printf("Arith: %d*%d=%d\n", args.A, args.B, reply)
var quot Quotient
err = client.Call("Arith.Divide", args, &quot)
if err != nil {
log.Fatalln("arith error:", err)
}
fmt.Printf("Arith: %d/%d=%d remainder %d\n", args.A, args.B,
quot.Quo, quot.Rem)
}
```

运行时，我们看到以下内容：

```
ch13$ go run tcparithclient.go localhost:1234
Arith: 17*8=136
Arith: 17/8=2 remainder 1
```

在选择使用 `rpc.HandleHTTP` 还是 `rpc.ServeConn` 时，主要取决于对控制权和速度的考量。TCP 服务器位于 HTTP 之下，因此为我们提供了更多的控制权。如果这一点不重要，那么之前的 `arithserver.go` 可能更合适。

### 值的匹配

你可能已经注意到，HTTP 客户端和 HTTP 服务器中值参数的类型并不相同。在服务器中，我们使用了 `Values`，而在客户端中，我们使用了 `Args`。这无关紧要，因为我们遵循了 Gob 序列化的规则，并且两个结构体字段的名称和类型是匹配的。当然，更好的编程实践是让名称保持一致！

然而，这确实指出了使用 Go RPC 时的一个潜在陷阱。如果我们将服务器中的结构体改为这样：

```
type Values struct {
C, B int
}
```

那么 Gob 不会有问题。在客户端，解组时会忽略服务器提供的 `C` 值，并使用 `A` 的默认零值。如果，比如说，发生了除以 `A`（零值）的操作，这可能会导致问题。

使用 Go RPC 需要程序员严格执行字段名和类型的稳定性。我们注意到，没有版本控制机制来执行这一点，也没有机制让 Gob 指示任何可能的不匹配。也没有必需的外部表示形式作为参考。如果只是添加字段，那么可能没有问题，但仍然需要控制。或许为数据结构添加一个 `version` 字段会有所帮助。

## JSON

本节在之前概念的基础上没有增加新内容。只是对数据使用了不同的“线缆”格式，即 JSON 而非 Gob。因此，客户端或服务器可以用其他理解套接字和 JSON 的语言编写。

### JSON RPC 服务器

使用 JSON 编码的服务器版本是 `jsonarithserver.go`：

```
ch13$ vi jsonarithserver.go
/* JSONArithServer
*/
package main
import (
"errors"
"log"
"net"
"net/rpc"
"net/rpc/jsonrpc"
)
type Args struct {
A, B int
}
type Quotient struct {
Quo, Rem int
}
type Arith int
func (t *Arith) Multiply(args *Args, reply *int) error {
*reply = args.A * args.B
return nil
}
func (t *Arith) Divide(args *Args, quo *Quotient) error {
if args.B == 0 {
return errors.New("divide by zero")
}
quo.Quo = args.A / args.B
quo.Rem = args.A % args.B
return nil
}
func main() {
arith := new(Arith)
rpc.Register(arith)
tcpAddr, err := net.ResolveTCPAddr("tcp", ":1234")
checkError(err)
listener, err := net.ListenTCP("tcp", tcpAddr)
checkError(err)
/* This works:
rpc.Accept(listener)
*/
/* and so does this:
*/
for {
conn, err := listener.Accept()
if err != nil {
continue
}
jsonrpc.ServeConn(conn)
}
}
func checkError(err error) {
if err != nil {
log.Fatalln("Fatal error ", err.Error())
}
}
```

按如下方式运行：

```
ch13$ go run jsonarithserver.go
```

接下来，我们来看相关的客户端。



### JSON RPC 客户端

调用算术服务器两个函数的客户端是 `jsonarithclient.go`：

```
$ vi jsonarithclient.go
/* JSONArithCLient
*/
package main
import (
"fmt"
"log"
"net/rpc/jsonrpc"
"os"
)
type Args struct {
A, B int
}
type Quotient struct {
Quo, Rem int
}
func main() {
if len(os.Args) != 2 {
log.Fatalln("Usage: ", os.Args[0], "server:port")
}
service := os.Args[1]
client, err := jsonrpc.Dial("tcp", service)
if err != nil {
log.Fatalln("dialing:", err)
}
// Synchronous call
args := Args{17, 8}
var reply int
err = client.Call("Arith.Multiply", args, &reply)
if err != nil {
log.Fatalln("arith error:", err)
}
fmt.Printf("Arith: %d*%d=%d\n", args.A, args.B, reply)
var quot Quotient
err = client.Call("Arith.Divide", args, &quot)
if err != nil {
log.Fatalln("arith error:", err)
}
fmt.Printf("Arith: %d/%d=%d remainder %d\n", args.A, args.B,
quot.Quo, quot.Rem)
}
```

它的运行方式如下：

```
$ go run jsonarithclient.go localhost:1234
Arith: 17*8=136
Arith: 17/8=2 remainder 1
```

虽然不太明显（除了 `jsonrpc.Dial` 之外），请求和响应是通过 `encoding/json` 包进行编码和解码的。可以查阅相关文档，例如 `go doc -u jsonrpc.WriteRequest` 或源代码本身，位置在：`/usr/local/go/src/net/rpc/jsonrpc/client.go`。

## 结论

RPC 是一种流行的分布式应用程序方式。本文基于 `Gob` 或 `JSON` 序列化技术，并使用 `HTTP` 和 `TCP` 作为传输层，展示了实现 RPC 的几种方法。

# 14. REST

在前面的章节中，我们了解了 `HTTP`，并给出了一个 Web 系统的示例。然而，我们并未赋予该系统任何特定的结构，只是秉持着对问题足够简单的原则。有一种由 `HTTP 1.1` 的主要作者之一（Roy Fielding）开发的架构风格，称为 `REST`（表述性状态转移）。在本章中，我们将探讨 `REST` 风格，以及它对于构建 Web 应用程序意味着什么。为此，我们必须回到基础概念。

`REST` 有许多必须遵循的组件，才能恰当地应用 `REST` 这个术语。不幸的是，它已经成为一个流行词，许多应用程序只包含了 `REST` 的“片段”，而非完整实现。我们将讨论 Richardson 成熟度模型，该模型描述了 API 在通往 RESTful 之路上走得多远。

在上一章中，我们讨论了 RPC（远程过程调用）。这是一种与 `REST` 完全不同的风格。我们还将比较这两种风格，探讨何时适合使用每种风格。

## URI 与资源

资源是我们在网络或互联网上想要与之交互的“事物”。我喜欢把它们想象成对象，但并不要求其实现必须基于对象——它们只是应该“看起来像”一个事物，可能包含组件。

每个资源都有一个或多个地址，称为 URI（统一资源标识符）。

注意

其国际化形式是 IRI——国际化资源标识符。

这些地址具有以下通用形式：

```
scheme:[//[user:password@]host[:port]][/]path[?query][#fragment]
```

典型的例子是 URL（统一资源定位符），其中 scheme 是 `http` 或 `https`，host 通过其 IP 地址或 DNS 名称指代一台计算机，如下所示：

```
https://jan.newmarch.name/IoT/index.html
```

还有一些非 HTTP 的 URL 方案，例如 `telnet`、`news` 和 `IPP`（互联网打印协议）。这些方案也包含位置组件。此外，还有其他类型，例如 URN（统一资源名称），它们通常是其他标识系统的包装器，并且不包含位置信息。例如，IETF 为通过 ISBN 标识的图书制定了一个标准的 URN 方案，比如本书的 ISBN：

```
urn:ISBN:978-1-4842-2692-6
```

这些 URN 往往并未广泛使用，但仍然存在。IANA 统一资源名称（URN）命名空间列表可在 `https://www.iana.org/assignments/urn-namespaces/urn-namespaces.xhtml` 找到。原始的方案（如 ISBN）仍然在使用中。

资源的正式定义可能难以精确界定。例如，`http://www.google.com` 在某种意义上代表 Google（它是 URL 的方案和主机部分），但该主机肯定不是某台固定的计算机。同样，本书的 ISBN 代表了本书的某些信息，但肯定不是指任何现存的副本（在撰写本章时，虽然 ISBN 已存在，但副本尚不存在）。

尽管如此，我们仍将资源视为基本概念，而 URI 是这些资源的标识符。IETF 在统一资源标识符（URI）：通用语法（`https://www.ietf.org/rfc/rfc3986.txt`）中同样定义得比较模糊：“术语‘资源’在一般意义上用于指代任何可能被 URI 标识的东西。”

一个资源可能拥有多个 URI。作为一个人，我有许多不同的标识符：我的税号指向我的一个方面，即我的财务事务；我的医保卡号指向我作为医疗服务的接受者；我的名字（相当独特）常用于指代我的不同方面。我的 URL `https://jan.newmarch.name` 指向我选择在网站上公开的那些方面。而 Google、LinkedIn、Facebook、Twitter 等可能也有某种 URI 来标记它们选择保存的我的那些方面。

各方都认同的是，资源是*名词*，而不是*动词*或*形容词*。对于银行账户，类似 `http://mybank/myaccount/withdraw` 的 URL 不被视为资源，因为它包含了动词 `withdraw`。同样，`http://amazon.com/buy/book-id` 不会标记一个资源，因为它包含了动词 `buy`（亚马逊没有这样的 URL）。

这是 `REST` 在 `HTTP` 上的第一个关键点：识别信息系统中的资源，并为其分配 URL。这方面有一些惯例，最常见的是，如果存在层级结构，则应在 URL 路径中反映出来。然而，这并不是必须的，因为信息也应该通过其他方式给出。

`REST` 设计 URI 的方法仍然有点像一门艺术。合法（且完全合规）的 URI 未必就是“好的”`REST` URI，许多所谓 RESTful API 的示例中，其 URI 也并非完全符合 RESTful 风格。2PartsMagic 在 RESTful URI 设计（`http://blog.2partsmagic.com/restful-uri-design/`）中提供了关于设计恰当 URI 的良好建议。

`REST` 设计 URI 的方法仍然有点像一门艺术。合法（且完全合规）的 URI 未必就是“好的”`REST` URI，许多所谓 RESTful API 的示例中，其 URI 也并非完全符合 RESTful 风格。Golang 开发者博客简要概述了 `REST` 的一些概念：`https://go.dev/doc/tutorial/web-service-gin`。除了示例之外，OpenAPI 是一个标准，可以帮助我们围绕“http apis”进行设计和工具化工作：`https://spec.openapis.org/oas/latest.html`。



## 表述（Representations）

资源的表述是指以某种形式捕捉资源相关信息的东西。例如，从税务局 URI 获取的我的表述可能是我的澳大利亚纳税申报单；从本地披萨店获取的我的表述则可能是一份披萨购买记录；而从我自己的网站获取的我的表述则是一个 HTML 文档。

这就是 REST 的关键之一：URI 标识资源，对资源的请求返回该资源的表述。资源本身保留在服务器上，根本不会被发送到客户端。事实上，资源甚至可能根本不存在任何具体的形态。例如，一个表述可能是通过访问某个 URI 触发的 SQL 查询结果生成的。

REST 并没有专门讨论协商资源表述的可能性。HTTP 1.1 有一个专门章节详细说明了如何实现这一点，涉及*服务器*、*客户端*和*透明*协商。客户端可以使用`Accept`头来指定偏好，例如：

```
Accept: application/xml; q=1.0, application/json; q=0.5
Accept-Language: fr
Accept-Charset: utf8
```

这表明客户端优先选择`application/xml`格式，但也接受`application/json`。服务器可以接受其中一种格式，或者回复它愿意接受的格式。

## REST 动词（REST Verbs）

你可以对 URI 发出特定的请求。如果你在向某个 URL 发送 HTTP 请求，HTTP 定义了可以发出的请求类型：`GET`、`PUT`、`POST`、`DELETE`、`HEAD`、`OPTIONS`、`TRACE`和`CONNECT`，以及像`PATCH`这样的扩展。这些动词的数量非常有限！这与我们对面向对象（O/O）编程的预期截然不同。例如，Java 的 JLabel 有大约 250 个方法，比如`getText`和`setHorizontalAlignment`。在某种程度上，我们可以将这些 OO 方法与 REST/HTTP 进行映射；一个 JLabel 实例就是一个资源，我们可以通过`getText`来“获取”它。虽然这在某些简单情况下很容易做到，但如何映射全部 250 个方法，或者是否应该映射，就不那么明确了。

现在，REST 通常被理解为仅使用 HTTP 中的四个动词：`GET`、`PUT`、`POST`和`DELETE`。`GET`大致对应于 OO 语言中的 getter 方法，而`PUT`则大致对应于 OOP 语言中的 setter 方法。如果一个 JLabel 是一个 REST 资源（事实并非如此），那么单一的`GET`动词如何能替代 JLabel 中上百个 getter 方法呢？

答案在于 URI 的`PATH`组件。一个标签包含文本、对齐方式等属性。这些实际上是标签的子资源，可以写成标签的子 URI。因此，如果标签的 URI 是[`http://jan.newmarch.name/my_label`](http://jan.newmarch.name/my_label)，那么子资源可以有（举例）以下 URI：

```
http://jan.newmarch.name/my_label/text
http://jan.newmarch.name/my_label/horizontalAlignment
```

如果你只想操作标签的文本，你可以使用文本资源的 URI，而不是在标签本身上使用 getter/setter 方法。

### GET 动词

要获取资源的表述，你可以对资源执行`GET`操作。这将返回资源的某种表述。对于这种选择，可能性是无穷无尽的。例如，请求本书的索引可能会返回一个使用 UTF-8 字符集、以 XML 文档形式呈现的法语索引表述，或者许多其他可能性。客户端和服务器可以协商这些可能性。

`GET`动词必须是*幂等*的。也就是说，重复请求应返回相同的结果（在表述类型范围内）。例如，多次请求传感器的温度应返回相同的结果（除非温度当然已经改变）。

默认情况下，幂等性允许缓存。这有助于减少网络流量，并可能为传感器节省电池电量。缓存并非总能得到保证：一个返回其被访问次数的资源，每次被访问时都会给出不同的结果。这是一种不常见的行为，会通过 HTTP 的`Cache-Control`头部来标识。

### PUT 动词

如果你想改变资源的状态，你可以使用`PUT`来提交新的值。`PUT`有两个主要限制：

*   你只能更改你知道其 URI 的资源的状态。
*   你发送的表述必须涵盖资源的所有组成部分。

例如，如果你只想更改标签中的文本，你应将`PUT`消息发送到 URL [`http://jan.newmarch.name/my_label/text`](http://jan.newmarch.name/my_label/text)，*而不是*发送到[`http://jan.newmarch.name/my_label`](http://jan.newmarch.name/my_label)。发送给标签则需要发送所有上百个字段。

`PUT`是幂等的，但不是*安全的*。也就是说，它会改变资源的状态，但重复调用会将其改为相同的状态。

`PUT`和`DELETE`不是 HTML 的一部分，大多数浏览器不直接支持它们。它们可以在支持 Ajax 的浏览器中被调用。关于为何不包含它们，有几个讨论。例如，请参阅 [`http://softwareengineering.stackexchange.com/questions/114156/why-are-there-are-no-put-and-delete-methods-on-html-forms`](http://softwareengineering.stackexchange.com/questions/114156/why-are-there-are-no-put-and-delete-methods-on-html-forms) 上的 “为什么 HTML 表单中没有 PUT 和 DELETE 方法？”。

### DELETE 动词

这会删除资源。它是幂等的，但不安全。

安全方法（例如`GET`）是指没有意外副作用的方法。不安全方法（例如`PUT`）可能具有潜在风险。无法撤销更改就是一件不安全的事情。理想情况下，`GET`返回资源的某个版本，仅此而已。`PUT`会覆盖资源，如果你没有检索先前的版本，它可能会永远消失。

幂等方法是指无论执行多少次，对同一请求产生相同副作用的方法。`DELETE`会移除一个资源，如果你第二次调用它，“结果”是相同的；该资源要么被删除（如果它被再次创建），要么保持不存在状态。

有关“幂等”和“安全”概念的更多信息，请参阅 [`https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html`](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html)。


好的，作为一名高级文档工程师和翻译员，我将遵循您的指示，将给定的英文文本翻译成中文，并严格按照注意事项保留所有格式和符号。


### POST 动词

`POST` 是一个包揽其他动词未涵盖情况的万能用语。关于 `POST` 的两种用途存在共识：

-   如果你想创建一个新资源但不知道它的 URI，那么可以将该资源的一种表示形式 `POST` 到一个知道如何创建该资源的 URI。返回的表示形式应包含新资源的 URI。*这一点很重要*。要与新资源交互，你必须知道它的 URI，而 `POST` 的返回结果会告诉你这一点。
-   如果一个资源有很多属性，而你只想更改其中一两个，那么可以 `POST` 一个仅包含更改后值的表示形式。

关于 `PUT` 和 `POST` 在边缘情况下的各自角色，存在激烈的争论。如果你想创建一个新资源，并且*确实*知道它将拥有的 URI，那么你可以使用 `PUT` 或 `POST`。选择哪一个似乎取决于其他因素。

SOAP 被设计为基于 HTTP 的 RPC 系统。它对*一切*都使用 `POST`。HTML 在表单中继续使用 `POST`，而本应有使用 `PUT` 的选项。出于这些原因，除非万不得已，否则我不会使用 `POST`。我想其他人有他们自己原则性的理由来使用 `POST` 而不是 `PUT`，但我完全不知道这些理由是什么 :-)。

由于其开放式的范围，`POST` 几乎可以用于任何事情。许多用途可能违反了 REST 模型，正如 SOAP 所充分证明的那样。但其中一些用途可能是合法的。`POST` 通常不是幂等的且不安全，尽管特定情况可能并非如此。以下 Stack Overflow 帖子中包含了关于 `PUT` vs. `PATCH` vs. `POST` 的深入讨论：[`https://stackoverflow.com/questions/28459418/use-of-put-vs-patch-methods-in-rest-api-real-life-scenarios`](https://stackoverflow.com/questions/28459418/use-of-put-vs-patch-methods-in-rest-api-real-life-scenarios)。

### 不维护状态（即无状态）

让我们先明确一点：Cookie 被排除在外。Cookie 通常用于跟踪用户与服务器交互过程中的状态，典型的例子是购物车。在服务器端创建一个结构，并返回一个 cookie，用于标识这是要使用的购物车。

REST 决定不在服务器上维护任何客户端状态。这简化了交互，也回避了在客户端或服务器崩溃后如何恢复一致性的棘手问题。如果服务器不需要维护任何状态，那么将产生更健壮的服务器模型。通常，与安全相关的项目会设置在 cookie 中。REST 端点应返回这些资源的表示形式。与身份验证相关的 cookie 通常超越这些资源，因此不符合 REST 原则。有关此主题的更多信息，请参见：[`https://www.ics.uci.edu/~fielding/pubs/dissertation/evaluation.htm#sec_6_3_4_2`](https://www.ics.uci.edu/~fielding/pubs/dissertation/evaluation.htm#sec_6_3_4_2)。

如果你不能使用 cookie，那该怎么办？其实很简单：在服务器上创建一个购物车。在 REST 下，这只能通过响应 `POST` 请求发生，该请求会为新资源返回一个新的 URI。所以你就使用这个新的 URI。你可以对这个 URI 执行 `GET`、`PUT`、`POST` 和 `DELETE` 操作，直接在资源上完成所有你想做的事情，而无需使用 cookie 进行变通。

### HATEOAS

HATEOAS 代表“超媒体作为应用状态的引擎”。它被公认为是一个糟糕的首字母缩写词，但已被沿用下来。其基本原则是，从一个 URI 导航到另一个以某种方式关联的 URI，不应通过任何带外机制来完成，而必须将新链接以某种超链接形式嵌入到第一个 URI 的表示形式中。这是 REST 的一个关键特性，但常常被忽略。

REST 并未规定链接的格式。它们可以使用 HTML 链接标签、嵌入在 PDF 文档中的 URL 或 XML 文档中给出的链接来提供。没有 URL 简单表示形式的格式不被视为超媒体语言，也不包含在 REST 中。

此外，REST 也没有明确说明链接的含义以及如何提取适当的链接。Fielding 在他的博客“REST APIs must be hypertext-driven”中写道，该博客位于 [`http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven`](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven)：

-   进入一个 REST API 时，除了初始 URI（书签）和一套适用于目标受众（即期望任何可能使用该 API 的客户端都能理解）的标准化媒体类型外，不应有任何先验知识。从那时起，所有的应用状态转换都必须由客户端选择服务器提供的选项来驱动，这些选项存在于接收到的表示形式中，或隐含在用户对这些表示形式的操作中。

IANA 维护了一个可用的关系类型注册表（IANA：链接关系，位于 [`http://www.iana.org/assignments/link-relations/link-relations.xhtml`](http://www.iana.org/assignments/link-relations/link-relations.xhtml)）。Web 链接 RFC 5988 描述了 web 链接注册表（[`https://datatracker.ietf.org/doc/html/rfc5988`](https://datatracker.ietf.org/doc/html/rfc5988)）。HTML5 规范定义了一小部分关系，并指向 Microformats 的 `rel` 值（[`http://microformats.org/wiki/existing-rel-values#HTML5_link_type_extensions`](http://microformats.org/wiki/existing-rel-values#HTML5_link_type_extensions)）以获取更大的列表。这些文档虽然有用，但不能直接付诸实施。我们稍后将看一个示例。

像 cookie 这样的机制，或者像用于 SOAP 的 WSDL 这样的外部 API 规范，都被 REST 有效地排除了。它们不是资源表示形式中包含的超链接。HATEOAS 通过允许在已知 URI 之外进行动态发现，将客户端与服务器解耦；使我们能够独立地演进 API。这与 RPC 不同，在 RPC 中，IDL 驱动着接口。



## 表示链接

链接在 HTML 文档中是标准化的。`Link`标签定义了一个 HTML 元素，该元素只能出现在 HTML 的头部部分。例如，一本包含章节等内容的书，如果其链接以 HTML `link`元素的形式给出，可能看起来像这样：

```
...
```

HTML 中的链接关系分为两种：一种是当前文档所需的链接（例如 CSS 文件），另一种是指向相关资源的链接（如前所述）。第一种类型的链接通常对用户不可见地下载。第二种类型通常不会由浏览器显示，但遵循 HATEOAS 原则的用户代理会使用它们。

XML 有各种链接规范，包括 XLink 和 Atom。Atom 似乎更受欢迎。

基于 XLink 的链接如下所示：

```
...
...
```

基于 Atom 的链接如下所示：

```
...
...
```

对于 JSON，其格式并未标准化。REST 烹饪书（`http://restcookbook.com/Mediatypes/json/`）指出了标准化的缺失，并引用了 W3C 规范 JSON-LD 1.0：“一种基于 JSON 的链接数据序列化格式”，以及 HAL（超文本应用语言）。诸如开放连接基金会等组织似乎使用他们自己内部开发的格式，但这适用于 CoAP，另一个基于 REST 的系统。

JSON-LD 使用术语`@id`来表示一个 URL，例如：

```
{
"name": Jan Newmarch:,
"homepage": {"@id": "https://jan.newmarch.name/"}
}
```

值得一提的是，W3C 在`https://www.w3.org/wiki/LinkHeader`也有一套关于 HTTP Link 头的规范，服务器可能会将其返回给客户端。例如，JSON-LD 利用此规范指向 HTTP 响应体中包含的 JSON 文档的规范。同样值得注意的是，一些实现以相同的方式处理 HTML Link 或 HTTP Header。

这会影响将链接信息从服务器传递到用户代理的序列化方法。用户代理和服务器必须就所使用的格式达成一致。对于 HTML（或 XHTML），这是标准化的。对于 XML，可以在文档中对链接系统进行引用。对于 JSON-LD，这可以在`Accept HTTP`头中通过`application/ld+json`来表示。

## 使用 REST 进行事务处理

REST 如何处理事务以及实际上任何其他类型的过程？它们并未在 Fielding 的原始论文中讨论。

维基百科上关于 HATEOAS 的词条给出了一个管理事务的糟糕示例。它从一个类似这样的 HTTP 请求开始：

```
GET /accounts/12345 HTTP/1.1
Host: bank.example.org
Accept: application/json
...
```

该请求返回一个 JSON 文档作为账户的表示，如下所示：

```
HTTP/1.1 200 OK
{
"account": {
"account_number": 12345,
"balance": {
"currency": "usd",
"value": 100.00
},
"links": {
"deposits": "/accounts/12345/deposits",
"withdrawals": "/accounts/12345/withdrawals",
"transfers": "/accounts/12345/transfers",
"close-requests": "/accounts/12345/close-requests"
}
}
}
```

如果我们请求 xml 格式，将会看到以下内容：

```
Content-Type: application/xml
Content-Length: ...

100.00

```

这给出了相关资源（存款、取款、转账和关闭）的 URI。然而，这些资源是*动词*，而不是*名词*，这非常不好。它们如何与 HTTP 动词交互？你是用`GET`来取款吗？用`POST`？用`PUT`？如果你`DELETE`一个取款操作会发生什么——这是否意味着回滚一个事务？在 REST 中，我们尽量避免将动词用作资源；然而，在 RPC 中，并不禁止将动词用作端点/过程调用。

更好的方式，如 Stack Overflow 帖子“REST 中的事务？”（参见`http://stackoverflow.com/questions/147207/transactions-in-rest`）中讨论的那样，是向账户发送`POST`请求，要求创建一个新事务：

```
POST /account/12345/transaction HTTP/1.1
```

这将返回一个新事务的 URL：

```
http://bank.example.org/account/12345/txn123
```

现在，交互通过这个事务 URL 进行，例如通过`PUT`一个新值来执行并提交事务。这里我们使用 XML。

```
PUT /account/12345/txn123

/account/56789

```

Mihindukulasooriya 等人在“RESTful 事务模型的七个挑战”（参见`http://ws-rest.org/2014/sites/default/files/wsrest2014_submission_4.pdf`）中给出了关于事务和 REST 更详细的讨论。类似的管理过程模型也被提出用于那些非单步的操作。通过前述 PDF，我们可以了解在设计提供事务能力的 RESTful API 时需要考虑的一些事项。

### 表 14-2 现有 RESTful 事务模型分析

| 特性 | 事务模型 |
| --- | --- |
| | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
| 事务属性 |
| 原子性 | T | T | T¹ | T | T | T | T |
| 隔离性 | T | T² | F | T | F | F | T |
| REST 约束 |
| 统一接口 | T | F | T | T | T | T | T |
| 无状态性 | T | T³ | F | T³ | F | T | T |
| HATEOAS | F | F | F | T | F | T | F |
| HTTP 相关属性 |
| 语义未违反 | T | T | T | T | F | T | T |
| 支持常用动词 | T | T | F | F | T | T | T |
| 低开销 | T | T | T | F | F | T | T |
| 其他属性 |
| 可选性 | T | ? | ? | T | ? | ? | F |
| 可发现性 | ? | ? | ? | T | ? | ? | T |
| 分布式事务 | X | X | T | ? | T | T | ? |
| 理论证明 | ? | ? | ? | T | T | T | ? |
| 可用实现 | T | ? | ? | T | ? | T | T |
| 性能评估 | ? | ? | ? | ? | ? | T | ? |
| 图例 - T 真 / F 假 / ? 未知或模型中未定义 |
| 1 – 假定操作可被补偿 2 – 可能存在丢失更新问题 3 – 参见第 3.3 节 |

### 表 14-1 RESTful 事务模型

| 编号 | 年份 | 事务模型 |
| --- | --- | --- |
| 1 | ~2000 | 使用过载 POST 的批处理事务 |
| 2 | 2007 | 将事务作为资源 |
| 3 | 2009 | 使用 REST 的乐观事务技术 |
| 4 | 2009 | 一致且可恢复的 RESTful 事务模型 |
| 5 | 2010 | 基于时间戳的 RESTful 服务两阶段提交协议 |
| 6 | 2011 | 尝试-取消/确认模式 |
| 7 | 2012 | 原子 REST 批处理事务 |



## 理查森成熟度模型

许多系统声称是 RESTful 的，但大多数并非如此。我们甚至遇到过声称 SOAP 是 RESTful 的系统，这显然是心智扭曲的典型案例。马丁·福勒讨论了理查森成熟度模型，该模型根据系统对 REST 的符合程度对其进行分类。（参见 [`https://martinfowler.com/articles/richardsonMaturityModel.html`](https://martinfowler.com/articles/richardsonMaturityModel.html)。）

### 第 0 级

该模型的起点是使用 HTTP 作为远程交互的传输系统，但完全不使用 Web 的任何机制。本质上，你是在利用 HTTP 作为自己远程交互机制（通常基于远程过程调用）的隧道机制。

### 第 1 级：资源

在 RMM 中迈向 REST 荣耀的第一步是引入资源。因此，现在我们不再将所有请求发送到单一的服务端点，而是开始与各个独立的资源进行通信。

### 第 2 级：HTTP 动词

在 0 级和 1 级中，我所有的交互都使用了 HTTP POST 动词，但有些人会改用或补充使用 GET。在这些级别上，这没有太大区别，两者都被用作隧道机制，让你可以通过 HTTP 隧道传输交互。第 2 级则脱离了这种模式，尽可能按照 HTTP 自身的使用方式来使用 HTTP 动词。

### 第 3 级：超媒体控制

最后一级引入了你经常听到的那个丑陋缩写 HATEOAS（超媒体作为应用状态引擎）。它解决了如何从可预约时间段列表，进而知道如何操作来预约的问题。

## 再谈闪卡系统

在第 10 章中，我们考虑了一个 Web 系统，它由服务器和在浏览器中渲染的 HTML 页面组成，并使用 JavaScript 和 CSS 来控制浏览器端的交互。当时并未尝试进行任何特别结构化的设计，只是将其视为一个传统的 Web 系统。

### 回顾

第 10 章的 Web 系统用于演示使用所谓的闪卡进行语言学习。系统每次向用户展示一张卡片，卡片上显示一种语言的单词，用户希望记住其翻译，而翻译内容则通过“翻转”卡片来显示。该系统会呈现一组不同的卡片集，然后在选中的卡片集中逐一显示卡片。

现在我们采用 REST 方法，将同样的场景构建为一个 HTTP 客户端-服务器系统。我们将进行以下若干改动：

-   根据场景设置合适的 URL。这些 URL 包括“根”URL `/`，以及每个闪卡集的 URL，此外还有每张闪卡的 URL。
-   所有用户交互代码（HTML、JavaScript 和 CSS）都将被省略。服务器将与任意的用户代理通信，而许多用户代理无法理解这些 UI 代码。
-   服务器不会维护或管理任何客户端状态。在 Web 示例中，表单数据从浏览器发送到服务器，服务器会以稍有不同的形式立即将其返回。需要维护状态的客户端应自行处理。
-   服务器将被设置成能够管理多种不同的序列化格式，并在客户端-服务器协商后提供合适的格式。
-   将大量使用 HTTP 机制，特别是在错误处理和内容协商方面。

### URL

本系统的 URL 以及可以执行的操作如下：

| URL | 动作 | 效果 |
| --- | --- | --- |
| `/` | `GET` | 获取闪卡集列表 |
| `/` | `POST` | 创建新的闪卡集 |
| `/flashcardSets/<set>` | `GET` | 获取该闪卡集中的卡片列表 |
| `/flashcardSets/<set>` | `POST` | 为该闪卡集创建新卡片 |
| `/flashcardSets/<set>` | `DELETE` | 如果闪卡集为空，则删除该闪卡集 |
| `/flashcardSets/<set>/<card>` | `GET` | 获取卡片内容 |
| `/flashcardSets/<set>/<card>` | `DELETE` | 从该闪卡集中删除卡片 |

这与第 10 章中描述的系统略有不同。主要的结构性差异在于，每张卡片都作为闪卡集的成员拥有自己的 URL。

服务器将处理的示例 URL 包括以下内容：

| 根 URL | 闪卡集 URL | 单张闪卡 URL |
| --- | --- | --- |
| `/` | `/flashcardSet/CommonWords` | `/flashcardSet/CommonWords/`你好 |

### ServeMux（多路复用器）

REST 基于对 URL 执行的少量动作。尝试使用 REST 原则的系统必须基于 URL。

服务器多路复用器会检查客户端请求的 URL，并根据 URL 模式调用处理程序。标准的 Go 多路复用器 `net/http.ServeMux` 使用特定的模式匹配机制：如果 URL 以 `/` 结尾，则表示以该 URL 为根的 URL 子树。如果 URL 不以 `/` 结尾，则仅表示该 URL 本身。URL 会与具有最长模式匹配的处理程序进行匹配。URL 甚至可以包含域名作为限定符。

我们需要为根 URL `/` 设置一个处理程序。该处理程序也会匹配任何类似 `/passwords` 的 URL，除非有其他处理程序捕获它。在本系统中，没有其他处理程序会这样做，因此，在 `/` 的处理程序中，我们需要对这种请求返回错误。

一个棘手的部分出现了，因为我们为 URL 使用了层次结构。某个特定的闪卡集将是 `/flashcardSets/CommonWords`。这实际上将是该特定闪卡集中卡片的目录。我们必须注册*两个*处理程序：一个用于 URL `/flashcardSets/CommonWords`（这是闪卡集资源），另一个用于 `/flashcardSets/CommonWords/`（注意末尾的 `/`），这是包含各个卡片及其 URL 的子树。

在 main 函数中用于注册这些处理程序的代码如下：

```go
http.HandleFunc(`/`, handleFlashCardSets)
files, err := ioutil.ReadDir(`flashcardsets`)
checkError(err)
for _, file := range files {
    fmt.Println(file.Name())
    cardset_url := `/flashcardSets/` + url.QueryEscape(file.Name())
    http.HandleFunc(cardset_url, handleOneFlashCardSet)
    http.HandleFunc(cardset_url + `/`, handleOneFlashCard)
}
```

请注意我们使用了 `QueryEscape` 函数。这是为了对 URL 中可能出现的任何特殊字符进行转义。例如，文件名中的 `$` 应编码为 `%44`。我们*确实*需要使用这样一个函数：我们的 URL 将包含中文字符，这些字符需要经过转义编码才能在 URL 中表示。这由 `QueryEscape` 完成，但有一个例外：路径中的空格应编码为 `%20`，而在表单数据中则应编码为 `+`。`PathEscape` 函数可以正确完成此操作。为了避免此问题，我们将从 URL 中移除空格。



## 内容协商

任何网络用户代理都可以尝试与任何网络服务器通信。浏览器与 HTML 服务器通信是我们在网络上常见的典型情况，但许多人也会熟悉使用其他用户代理（例如`curl`、`wget`甚至`telnet`）！浏览器和其他工具会利用 HTTP 响应中的`Content-Type`来确定如何处理提供的内容。

对于 Web 应用程序而言，用户代理必须能够理解服务器正在交付什么，因为它试图参与一个可能没有用户协助的交互过程。RPC 系统通常使用客户端和服务器共同遵守的外部规范。但这里的情况并非如此。

解决方案是双方必须就内容格式达成一致。这是在 HTTP 层面完成的。客户端会声明其可接受的格式范围。如果服务器同意，则继续通信。如果不同意，服务器会告知客户端它所能接受的格式，然后客户端在可能的情况下重新开始协商。

协商使用 MIME 类型。有数百种标准类型：`text/html`、`application/pdf`、`application/xml`等。浏览器可以渲染其接收到的任何 HTML 文档。像 VLC 这样支持 HTTP 的音乐播放器可以播放其接收到的任何 MP3 文件。但对于闪卡应用程序来说，它无法处理任何通用格式，只能处理符合预期结构的消息。没有任何标准的 MIME 类型适合为这个闪卡应用程序协商专门的协议。因此，我们自行创建。客户端和服务器必须知道它们正在处理共享的 MIME 类型，否则无法正常通信。

IANA 为创建自己的 MIME 类型制定了规则。我使用的类型是 `application/x.flashcards`。服务器将能够提供 JSON 和 XML 格式，因此两种可接受的 MIME 类型是 `application/x.flashcards+xml` 和 `application/x.flashcards+json`。

HTTP 内容协商规定，用户代理可以建议一个可接受格式的列表，并赋予 0 到 1 之间的权重，如下所示：

```
Accept: application/x.flashcards+xml; q=0.8,
application/x.flashcards+json; q=0.4
```

服务器可以检查请求，并决定自己是否能处理该格式。我们在服务器中使用以下代码来确定用户代理是否请求了某种类型，以及其权重是多少（0 表示未请求）：

```go
const flashcard_xml string = "application/x.flashcards+xml"
const flashcard_json string = "application/x.flashcards+json"
type ValueQuality struct {
	Value   string
	Quality float64
}
/* 基于 https://siongui.github.io/2015/02/22/go-parse-accept-language/ */
func parseValueQuality(s string) []ValueQuality {
	var vqs []ValueQuality
	strs := strings.Split(s, `,`)
	for _, str := range strs {
		trimmedStr := strings.Trim(str, ` `)
		valQ := strings.Split(trimmedStr, `;`)
		if len(valQ) == 1 {
			vq := ValueQuality{valQ[0], 1}
			vqs = append(vqs, vq)
		} else {
			qp := strings.Split(valQ[1], `=`)
			q, err := strconv.ParseFloat(qp[1], 64)
			if err != nil {
				q = 0
			}
			vq := ValueQuality{valQ[0], q}
			vqs = append(vqs, vq)
		}
	}
	return vqs
}
func qualityOfValue(value string, vqs []ValueQuality) float64 {
	for _, vq := range vqs {
		if value == vq.Value {
			return vq.Quality
		}
	}
	// 未找到
	return 0
}
```

如果服务器不接受用户代理请求的任何类型，则返回 HTTP 状态码 406“不可接受”（Not acceptable），并提供可接受格式的列表。服务器中执行此操作的代码段如下：

```go
func handleFlashCardSets(rw http.ResponseWriter, req *http.Request) {
...
if req.Method == "GET" {
    acceptTypes := parseValueQuality(req.Header.Get("Accept"))
    q_xml := qualityOfValue(flashcard_xml, acceptTypes)
    q_json := qualityOfValue(flashcard_json, acceptTypes)
    if q_xml == 0 && q_json == 0 {
        // 在 Accept 头中找不到 XML 或 JSON
        rw.Header().Set("Content-Type", "flashcards+xml, flashcards+json")
        rw.WriteHeader(http.StatusNotAcceptable)
        return
    }
...
```

这展示了 HTTP 服务器一种常见的 REST 模式：接收到 HTTP 请求后，检查服务器能否处理该请求。如果不能，则返回 HTTP 错误；如果可以，尝试处理请求。如果处理失败，则返回 HTTP 错误；处理成功，则返回相应的 HTTP 成功码和结果。

### GET /

所有闪卡集都存储在目录 `flashcardsets` 中。`GET /` 请求需要列出所有这些文件，并以适合客户端的格式准备好。该格式是一个闪卡集名称及其*URL 地址*的列表。HATEOAS 需要这些 URL：名称列表告诉我们有哪些闪卡集，但客户端需要它们的 URL 才能进入与其中一个闪卡集交互的阶段。

服务器中每个 `FlashcardSet` 的结构体类型包含集合的名称及其 URL（作为字符串）：

```go
type FlashcardSet struct {
	Name string
	Link string
}
```

服务器上的闪卡集合可以从闪卡集目录构建。`ioutil.ReadDir()` 将创建一个 `os.FileInfo` 数组。需要将其转换为文件名列表，如下所示：

```go
files, err := ioutil.ReadDir(`flashcardsets`)
checkError(err)
numfiles := len(files)
cardSets := make([]FlashcardSet, numfiles, numfiles)
for n, file := range files {
	fmt.Println(file.Name())
	cardSets[n].Name = file.Name()
	// 应使用 PathEscape，但在 Go 1.6 中不可用
	cardSets[n].Link = `/flashcardSets/` + url.QueryEscape(file.Name())
}
```

这会创建一个文件名数组以及服务器上资源的相对链接，格式为 `/<name>`。对于 `CommonWords` 集合，相对链接 URL 将为 `/flashcardSets/CommonWords`。协议（`http` 或 `https`）和主机（例如"localhost"）留给客户端自行处理。

不幸的是，文件名可能包含 URL 路径名中不合法的字符。`url.PathEscape` 函数可以正确转义所有这些字符。`url.QueryEscape` 函数除了文件名中的空格会被替换为 `+` 而不是 `%20` 之外，其他都能正确处理。

最后，服务器判断是优先使用 JSON 还是 XML，并通过模板生成正确输出给客户端。对于 XML，模板代码如下：

```go
...
if q_xml >= q_json {
    // 优先使用 XML
    t, err := template.ParseFiles("xml/ListFlashcardSets.xml")
    if err != nil {
        fmt.Println("模板错误")
        http.Error(rw, err.Error(), http.StatusInternalServerError)
        return
    }
    rw.Header().Set("Content-Type", flashcard_xml)
    t.Execute(rw, cardSets)
} else {
    // 优先使用 JSON
...
```

XML 模板如下：

```xml
ch14 % cat xml/ListFlashcardSets.xml
<flashcards>
{{range .}}
<flashcardSet>
<name>{{.Name}}</name>
<link>{{.Link}}</link>
</flashcardSet>
{{end}}
</flashcards>
```

对于仅包含 `CommonWords` 和 `Lesson04` 两个集合的列表，发送给客户端的内容如下：

```xml
<flashcards>
<flashcardSet>
<name>Common Words</name>
<link>/flashcardSets/CommonWords</link>
</flashcardSet>
<flashcardSet>
<name>Lesson04</name>
<link>/flashcardSets/Lesson04</link>
</flashcardSet>
</flashcards>
```

### POST /

此处，客户端请求创建一个新的闪卡集。预期客户端将提供闪卡集的名称。我们使其看起来像表单提交数据：

```
name=
```

这种情况比 `GET` 简单得多。以表单数据的形式从请求中获取值。然后检查请求的名称中是否包含危险字符，例如将闪卡集命名为 `/etc/passwd`。如果包含，则返回 403“禁止”（Forbidden）。如果名称看起来没问题，则使用该名称创建一个目录。如果创建失败（目录可能已存在），再次返回 403。否则，返回 201“已创建”（Created）和新资源的相对 URL：

```go
...
} else if req.Method == "POST" {
    name := req.FormValue(`name`)
    if hasIllegalChars(name) {
        rw.WriteHeader(http.StatusForbidden)
        return
    }
    // 去掉所有空格，因为它们很烦人
    name = strings.Replace(name, ` `, ``, -1)
    err := os.Mkdir(`flashcardsets/`+name,
        (os.ModeDir | os.ModePerm))
    if err != nil {
        rw.WriteHeader(http.StatusForbidden)
        return
    }
    rw.WriteHeader(http.StatusCreated)
    base_url := req.URL.String()
    new_url := base_url + `flashcardSets/` + name + `/`
    rw.Write([]byte(new_url))
} else {
    rw.WriteHeader(http.StatusMethodNotAllowed)
}
...
```



### 处理其他 URL

我们讨论过服务器处理带有 `GET` 和 `POST` 请求的 `/` URL 的代码。此应用还有另外两类 URL：处理集合中的卡片和处理单张卡片。不过，就编码而言，这并没有带来新的概念。

*   获取集合中的卡片列表是另一种目录列表操作。
*   向集合发布一张新卡片，意味着在对应目录下创建一个包含客户端内容的文件。
*   删除一个集合意味着移除一个目录。如果目录为空，则操作正常；否则会引发错误。
*   获取一张卡片意味着读取卡片文件并发送其内容。
*   删除一张卡片意味着移除一个文件。

所有这些操作都没有特别新颖之处。我们尚未完成某些操作（如 `DELETE`）的代码：这些操作会返回 HTTP 状态码 501 `'未实现'`。此外，我们将单张卡片的内容作为 `text/plain` 返回：它们具有第 10 章中使用的复杂 JSON/Go 结构，但这对于讨论本系统的 REST 方面并非必需。

## 完整服务器

以下是处理对 `/` 的请求以及从该路径到其他 URL 的完整服务器代码。运行它需要闪卡集合和单张卡片，这些文件位于 `ch14` 文件夹中，可从 [`https://github.com/Apress/network-prog-with-go-2e`](https://github.com/Apress/network-prog-with-go-2e) 获取。

```
ch14$ vi server.go
/* Server
*/
package main
import (
"fmt"
"html/template"
"log"
"net/http"
"net/url"
"os"
"regexp"
"strconv"
"strings"
)
type FlashcardSet struct {
Name string
Link string
}
type Flashcard struct {
Name string
Link string
}
const flashcard_xml string = "application/x.flashcards+xml"
const flashcard_json string = "application/x.flashcards+json"
type ValueQuality struct {
Value   string
Quality float64
}
/* 基于 https://siongui.github.io/2015/02/22/go-parse-accept-language/ */
func parseValueQuality(s string) []ValueQuality {
var vqs []ValueQuality
strs := strings.Split(s, `,`)
for _, str := range strs {
trimmedStr := strings.Trim(str, ` `)
valQ := strings.Split(trimmedStr, `;`)
if len(valQ) == 1 {
vq := ValueQuality{valQ[0], 1}
vqs = append(vqs, vq)
} else {
qp := strings.Split(valQ[1], `=`)
q, err := strconv.ParseFloat(qp[1], 64)
if err != nil {
q = 0
}
vq := ValueQuality{valQ[0], q}
vqs = append(vqs, vq)
}
}
return vqs
}
func qualityOfValue(value string, vqs []ValueQuality) float64 {
for _, vq := range vqs {
if value == vq.Value {
return vq.Quality
}
}
return 0
}
func main() {
if len(os.Args) != 2 {
log.Fatalln("用法: ", os.Args[0], ":端口号\n")
}
port := os.Args[1]
http.HandleFunc(`/`, handleFlashCardSets)
files, err := os.ReadDir(`flashcardsets`)
checkError(err)
for _, file := range files {
fmt.Println(file.Name())
cardset_url := `/flashcardSets/` + url.QueryEscape(file.Name())
fmt.Println("正在为 ", cardset_url, " 添加处理器")
http.HandleFunc(cardset_url, handleOneFlashCardSet)
http.HandleFunc(cardset_url+`/`, handleOneFlashCard)
}
// 将请求交付给处理器
err = http.ListenAndServe(port, nil)
checkError(err)
}
func hasIllegalChars(s string) bool {
// 检查是否存在可能跳出当前目录的字符
b, err := regexp.Match("[/$~]", []byte(s))
if err != nil {
fmt.Println(err)
return true
}
if b {
return true
}
return false
}
func handleOneFlashCard(rw http.ResponseWriter, req *http.Request) {
// 应为 PathUnescape
path, _ := url.QueryUnescape(req.URL.String())
// 去掉开头的 '/'
path = path[1:]
if req.Method == http.MethodGet {
fmt.Println("正在处理卡片: ", path)
json_contents, err := os.ReadFile(path)
if err != nil {
rw.WriteHeader(http.StatusNotFound)
rw.Write([]byte(`资源未找到`))
return
}
// 这里偷懒，直接返回 text/plain 格式的内容
rw.Write(json_contents)
return
} else if req.Method == http.MethodDelete {
rw.WriteHeader(http.StatusNotImplemented)
} else {
rw.WriteHeader(http.StatusMethodNotAllowed)
}
return
}
func handleFlashCardSets(rw http.ResponseWriter, req *http.Request) {
if req.URL.String() != `/` {
// 此函数仅处理 '/'
rw.WriteHeader(http.StatusNotFound)
rw.Write([]byte("资源未找到\n"))
return
}
if req.Method == "GET" {
acceptTypes := parseValueQuality(req.Header.Get("Accept"))
fmt.Println(acceptTypes)
q_xml := qualityOfValue(flashcard_xml, acceptTypes)
q_json := qualityOfValue(flashcard_json, acceptTypes)
if q_xml == 0 && q_json == 0 {
// 在 Accept 头部中未找到 XML 或 JSON
rw.Header().Set("Content-Type", "flashcards+xml, flashcards+json")
rw.WriteHeader(http.StatusNotAcceptable)
return
}
files, err := os.ReadDir(`flashcardsets`)
checkError(err)
numfiles := len(files)
cardSets := make([]FlashcardSet, numfiles, numfiles)
for n, file := range files {
fmt.Println(file.Name())
cardSets[n].Name = file.Name()
// 应为 PathEscape，但在 go 1.6 中不可用
cardSets[n].Link = `/flashcardSets/` + url.QueryEscape(file.Name())
}
parseFile := "xml/ListFlashcardSets.xml"
flashcardType := flashcard_xml
if q_xml = q_json {
// 优先选择 XML
t, err := template.ParseFiles("xml/ListOneFlashcardSet.xml")
if err != nil {
fmt.Println("模板错误")
http.Error(rw, err.Error(), http.StatusInternalServerError)
return
}
rw.Header().Set("Content-Type", flashcard_xml)
t.Execute(os.Stdout, cards)
t.Execute(rw, cards)
} else {
// 优先选择 JSON
t, err := template.ParseFiles("json/ListOneFlashcardSet.json")
if err != nil {
fmt.Println("模板错误", err)
http.Error(rw, err.Error(), http.StatusInternalServerError)
return
}
rw.Header().Set("Content-Type", flashcard_json)
t.Execute(rw, cards)
}
} else if req.Method == http.MethodPost {
name := req.FormValue(`name`)
if hasIllegalChars(name) {
rw.WriteHeader(http.StatusForbidden)
return
}
err := os.Mkdir(`flashcardsets/`+name,
(os.ModeDir | os.ModePerm))
if err != nil {
rw.WriteHeader(http.StatusForbidden)
return
}
rw.WriteHeader(http.StatusCreated)
base_url := req.URL.String()
new_url := base_url + `flashcardSets/` + name
_, _ = rw.Write([]byte(new_url))
} else if req.Method == http.MethodDelete {
rw.WriteHeader(http.StatusNotImplemented)
} else {
rw.WriteHeader(http.StatusMethodNotAllowed)
}
return
}
func checkError(err error) {
if err != nil {
log.Fatalln("致命错误 ", err.Error())
}
}
```

按如下方式运行：

```
ch14$ go run server.go :8000
```

## 客户端

客户端相对简单，没有提供真正新颖的内容。此客户端仅以 XML 格式请求内容。一个新的部分是，闪卡集合的内容在 `cardset` 标签内包含作为超文本属性的链接。在 `Card` 结构中，这可以通过标签标签 `xml:"href,attr"` 转换为结构体的一个字段。

此客户端在 `getFlashcardSets()` 函数中获取闪卡集合列表及其 URL（步骤 1）。该函数返回一个 `FlashcardSets` 结构体。例如，它可以用于向用户展示一个列表，供用户选择特定的集合。一旦选定，就可以使用该集合的 URL 与资源进行交互。

然后，此客户端在 `createFlashcardSet()` 函数中创建一个名为 `NewSet` 的闪卡集合（步骤 2）。客户端首次运行时，会创建该集合，并返回该集合的 URL。再次运行时，由于集合已存在，将从服务器收到一个操作被禁止的错误。

接着，此客户端从服务器提供的 URL 中选取第一个闪卡集合，并请求该集合所包含的卡片（步骤 3）。然后，它从该集合中选取第一张卡片并获取其内容（步骤 4）。

客户端代码是 `client.go`：



```go
ch14$ vi client.go
/* 客户端
*/
package main

import (
"encoding/xml"
"fmt"
"io"
"log"
"net/http"
"net/http/httputil"
"net/url"
"os"
"strings"
)

const flashcard_xml string = "application/x.flashcards+xml"
const flashcard_json string = "application/x.flashcards+json"

type FlashcardSets struct {
XMLName string    `xml:"cardsets"`
CardSet []CardSet `xml:"cardset"`
}

type CardSet struct {
XMLName string `xml:"cardset"`
Name    string `xml:"name"`
Link    string `xml:"href,attr"`
Cards   []Card `xml:"card"`
}

type Card struct {
Name string `xml:"name"`
Link string `xml:"href,attr"`
}

func getter(url *url.URL, client *http.Client, acceptType string) *http.Response {
request, err := http.NewRequest("GET", url.String(), nil)
checkError(err)
if acceptType != "" {
request.Header.Add("Accept", flashcard_xml)
}
response, err := client.Do(request)
checkError(err)
if response.StatusCode != http.StatusOK {
log.Fatalln(err, response)
}
fmt.Println("响应头为：")
b, _ := httputil.DumpResponse(response, false)
fmt.Print(string(b))
return response
}

func getOneFlashcard(url *url.URL, client *http.Client) string {
// 以字符串形式获取卡片，不做任何处理
response := getter(url, client, "")
body, err := io.ReadAll(response.Body)
checkError(err)
content := string(body[:])
//fmt.Printf("主体为 %s", content)
return content
}

func getOneFlashcardSet(url *url.URL, client *http.Client) CardSet {
// 获取一组卡片
response := getter(url, client, flashcard_xml)
body, err := io.ReadAll(response.Body)
content := string(body[:])
fmt.Printf("主体为 %s", content)
var sets CardSet
contentType := getContentType(response)
if contentType == "XML" {
err = xml.Unmarshal(body, &sets)
checkError(err)
fmt.Println("XML：", sets)
return sets
}
/* else if contentType == "JSON" {
var sets FlashcardSetsJson
err = json.Unmarshal(body, &sets)
checkError(err)
fmt.Println("JSON: ", sets)
}
*/
return sets
}

func getFlashcardSets(url *url.URL, client *http.Client) FlashcardSets {
// 获取顶层 /
response := getter(url, client, flashcard_xml)
body, err := io.ReadAll(response.Body)
content := string(body[:])
fmt.Printf("主体为 %s", content)
var sets FlashcardSets
contentType := getContentType(response)
if contentType == "XML" {
err = xml.Unmarshal(body, &sets)
checkError(err)
fmt.Println("XML：", sets)
return sets
}
return sets
}

func createFlashcardSet(url1 *url.URL, client *http.Client, name string) string {
data := make(url.Values)
data[`name`] = []string{name}
response, err := client.PostForm(url1.String(), data)
checkError(err)
if response.StatusCode != http.StatusCreated {
fmt.Println(`错误：`, response.Status)
return ``
}
body, err := io.ReadAll(response.Body)
content := string(body[:])
return content
}

func main() {
if len(os.Args) != 2 {
log.Fatalln("用法：", os.Args[0], "http://host:port/page")
}
url, err := url.Parse(os.Args[1])
checkError(err)
client := &http.Client{}
// 步骤 1：获取闪卡集合列表
flashcardSets := getFlashcardSets(url, client)
fmt.Println("步骤 1：", flashcardSets)
// 步骤 2：尝试创建一个新的闪卡集合
new_url := createFlashcardSet(url, client, `NewSet`)
fmt.Println("步骤 2：新闪卡集合的 URL 为：", new_url)
// 步骤 3：使用第一个闪卡集合，
//         获取其中的卡片列表
set_url, _ := url.Parse(os.Args[1] + flashcardSets.CardSet[0].Link)
fmt.Println("正在请求闪卡集合 URL：", set_url.String())
oneFlashcardSet := getOneFlashcardSet(set_url, client)
fmt.Println("步骤 3：", oneFlashcardSet)
// 步骤 4：获取一张闪卡的内容
//         偷懒，仅以 text/plain 形式获取，不做任何处理
card_url, _ := url.Parse(os.Args[1] + oneFlashcardSet.Cards[0].Link)
fmt.Println("正在请求 URL：", card_url.String())
oneFlashcard := getOneFlashcard(card_url, client)
fmt.Println("步骤 4", oneFlashcard)
}

func getContentType(response *http.Response) string {
contentType := response.Header.Get("Content-Type")
if strings.Contains(contentType, flashcard_xml) {
return "XML"
}
if strings.Contains(contentType, flashcard_json) {
return "JSON"
}
return ""
}

func checkError(err error) {
if err != nil {
log.Fatalln("致命错误 ", err.Error())
}
}
```

运行方式如下：

```
ch14$ go run client.go http://localhost:8000/
...（所有闪卡相关内容）...
```

## 使用 REST 还是 RPC

REST 与 RPC 的主要区别在于交互方式。在 RPC 中，你是在调用函数，传递对象或基本类型作为参数，并返回对象或基本类型。函数是*动词*：对*那个*做*这个*。而 REST 则是与对象交互，要求它们展示自身状态或以某种方式改变状态。

这种差异体现在上一章讨论的 Go RPC 机制和本章的 REST 机制上。在基于 HTTP 的 Go RPC 中，服务器注册的是*函数*；而在 REST 中，服务器注册的是*URL*的处理器。

哪个更好？都不好。哪个更快？都不快。哪种更适合受控环境？可能是 RPC。哪种更适合开放环境？可能是 REST。

你会看到基于速度和资源分配的争论。基于二进制系统的 RPC 可能比基于文本的 HTTP 系统更快。但 SOAP 是一种基于 HTTP 的文本 RPC 系统，可能比 REST 更慢。HTTP2 使用二进制格式，在传输 BSON 等二进制数据时，速度可能与其他二进制系统相当。更令人困惑的是，Apache Thrift RPC 允许选择数据格式（二进制、紧凑二进制、JSON 和文本）和传输方式（套接字、文件和共享内存）。一个系统展示了所有选项！

更重要的因素可能是运营环境的受控程度。RPC 系统是紧密耦合的，一个组件故障可能导致整个系统崩溃。当存在单一管理权威、有限的硬件和软件配置以及清晰的故障修复渠道时，RPC 系统可以工作得很好。

另一方面，Web 是未受控的。没有单一权威——即使是 DNS 这样的“通用”服务也是高度分布式的。存在大量不同的硬件、操作系统和软件；几乎没有执行任何策略的可能性；如果出了问题，往往找不到可以指明修复的人。在这种情况下，松散耦合的系统可能更好。

基于 HTTP 的 REST 非常适合这种情况。HATEOAS 允许服务器动态重新配置，按需更改 URL（甚至指向不同的服务器！）。HTTP 设计为在可能的情况下缓存结果。防火墙通常被配置为允许 HTTP 流量并阻止大多数其他协议。REST 在这里是一个不错的选择。

需要注意的是，REST 并非唯一可能的基于 HTTP 的系统。前面已经提到了 SOAP。还有许多商业上非常成功的系统是“几乎” REST 的——Richardson 成熟度模型的第 1 级和第 2 级。它们无法享受 REST/HTTP 组合的全部好处，但仍然可以工作。

毫无疑问，未来会出现其他模型。在物联网领域，CoAP 在低功耗无线系统中很流行。它也是基于 REST 的，但与 HTTP-REST 的方式略有不同。

## 结论

REST 是 Web 的架构模型。它可以以多种不同方式应用，特别是作为 HTTP 和 CoAP（例如低功耗、有损网络协议）。本章阐述了 REST 在 HTTP 中的应用方式。



# 15. WebSocket

浏览器等网络用户代理与 Apache 等网络服务器之间的标准交互模型是：用户代理发出 HTTP 请求，服务器对每个请求做出单一回复。对于浏览器而言，请求通过点击链接、在地址栏输入 URL、点击前进或后退按钮等方式发起。响应被视为一个新页面，并加载到浏览器窗口中。

这种传统模型存在诸多缺陷。首先，每个请求都会打开并关闭一个新的 TCP 连接。HTTP 1.1 通过允许持久连接解决了这个问题，使得连接可以在短时间内保持打开状态，从而允许对同一服务器发起多个请求（例如，请求图片）。

虽然 HTTP 1.1 持久连接缓解了加载包含大量图片的页面速度慢的问题，但并未改进交互模型。即使采用了表单，模型仍然是通过提交表单并将响应显示为新页面。JavaScript 有助于在提交前对表单数据进行错误检查，但并未改变这一模型。

AJAX（异步 JavaScript 和 XML）在用户交互模型上取得了重大进展。它允许浏览器发起请求，并仅使用响应通过 HTML 文档对象模型（DOM）就地更新显示内容。但同样，交互模型本身并未改变，AJAX 仅仅影响了浏览器管理返回页面的方式。Go 语言没有为 AJAX 提供显式的额外支持，因为无需如此：HTTP 服务器看到的只是一个可能携带 XML 或 JSON 数据的普通 HTTP POST 请求，这可以通过已讨论的技术来处理。

所有这些仍然是浏览器（或用户代理）到服务器的通信。所缺少的是服务器到浏览器的通信，即浏览器已经建立了到服务器的 TCP 连接，并从服务器读取消息。这可以由 WebSocket 来填补：浏览器（或任何用户代理）保持一个到 WebSocket 服务器的长期 TCP 连接。该 TCP 连接允许任何一侧发送任意数据包，因此任何应用层协议都可以在 WebSocket 上使用。

启动 WebSocket 的方式是用户代理发送一个特殊的 HTTP 请求，该请求表明“切换至 WebSocket”。HTTP 请求底层的 TCP 连接保持打开状态，但用户代理和服务器都转而使用 WebSocket 协议，而不是获取 HTTP 响应并关闭套接字。

请注意，仍然是浏览器或用户代理发起 WebSocket 连接。浏览器自身并不运行 TCP 服务器。尽管 IETF RFC 6455 规范（参见 [`https://tools.ietf.org/html/rfc6455`](https://tools.ietf.org/html/rfc6455)）较为复杂，但该协议的设计使其相对易于使用。客户端打开一个 HTTP 连接，然后用其自身的 WS 协议替换 HTTP 协议，并重用同一个 TCP 连接或新建一个连接。

Go 语言在其子仓库中提供了一些对 WebSocket 的支持，但实际上推荐使用第三方包。本章将探讨这两个包。

## WebSocket 服务器

一个 WebSocket 服务器最初是一个 HTTP 服务器，接受 TCP 连接并处理这些 TCP 连接上的 HTTP 请求。当一个请求进来，要求将该连接切换为 WebSocket 连接时，协议处理器会从 HTTP 处理器变为 WebSocket 处理器。因此，只有那条 TCP 连接的角色发生了改变；该服务器对于其他请求仍然是一个 HTTP 服务器，而那条连接底层的 TCP 套接字则被用作 WebSocket。

在第 8 章（HTTP）中讨论的一个简单服务器，注册了诸如文件处理器或函数处理器等各种处理器。为了处理 WebSocket 请求，我们只需注册一个不同类型的处理器——即 WebSocket 处理器。服务器使用哪个处理器取决于 URL 模式。例如，可能为 `/` 注册一个文件处理器，为 `/cgi-bin/...` 注册一个函数处理器，而为 `/ws` 注册一个 WebSocket 处理器。

一个仅期望用于 WebSocket 的 HTTP 服务器可以按如下方式运行：

```
func main() {
    http.Handle("/", websocket.Handler(WSHandler))
    err := http.ListenAndServe(":12345", nil)
    checkError(err)
}
```

一个更复杂的服务器可以通过简单地添加更多处理器来同时处理 HTTP 和 WebSocket 请求。

在实现方面，我们有多种选择，包括手动管理原始 HTTP 连接（TCP 劫持），以及使用 `x` 包的 WebSocket 实现，甚至使用知名的第三方包。

## `golang.org/x/net/websocket` 包

Go 语言有一个名为 `golang.org/x/net/websocket` 的子仓库包。

该包的文档说明如下：

*此包目前缺少另一个更活跃维护的 WebSocket 包所具有的一些功能：* [`pkg.go.dev/github.com/gorilla/websocket`](https://pkg.go.dev/github.com/gorilla/websocket)。*遗憾的是，即使是 Gorilla WebSocket 包也正在寻找一位活跃的首席维护者。尽管如此，它仍然是领先的实现。*

这表明，使用替代包可能是更好的选择。然而，为了与本书其余部分保持一致（即使用 Go 团队的包），我们在此处探讨此包。后续章节将探讨替代包。



### 消息对象

HTTP 是一种流式协议，而 WebSocket 则是基于帧的。你需要准备一个数据块（大小不限），并将其作为一组帧来发送。帧可以包含采用 UTF-8 编码的字符串或字节序列。

使用 WebSocket 最简单的方式就是准备一个数据块，让 Go WebSocket 库将其打包为一组帧数据，通过网络发送，并在接收端以相同的数据块形式接收。`websocket` 包中包含一个名为 `Message` 的便捷对象，专门用于此目的。`Message` 对象有两个方法——`Send` 和 `Receive`，它们都将 WebSocket 作为第一个参数。第二个参数可以是用于存储数据的变量的地址，也可以是要发送的数据。发送字符串数据的代码如下所示：

```
msgToSend := "Hello"
err := websocket.Message.Send(ws, msgToSend)
var msgToReceive string
err := websocket.Message.Receive(conn, &msgToReceive)
```

发送字节数据的代码如下所示：

```
dataToSend := []byte{0, 1, 2}
err := websocket.Message.Send(ws, dataToSend)
var dataToReceive []byte
err := websocket.Message.Receive(conn, &dataToReceive)
```

接下来给出一个用于发送和接收字符串数据的回显服务器。请注意，在 WebSocket 中，任何一方都可以发起消息发送，而在这个服务器中，当客户端连接时，我们是从服务器向客户端发送消息（发送/接收），而不是更常见的接收/发送服务器。该服务器文件为 `echoserver.go`：

```
$ mkdir ch15
$ cd ch15
$ vi echoserver.go
/* EchoServer
*/
package main
import (
"fmt"
"golang.org/x/net/websocket"
"log"
"net/http"
)
func Echo(ws *websocket.Conn) {
fmt.Println("Echoing")
for n := 0; n < 10; n++ {
msg := "Hello  " + string(n+48)
fmt.Println("Sending to client: " + msg)
err := websocket.Message.Send(ws, msg)
if err != nil {
fmt.Println("Can't send")
break
}
var reply string
err = websocket.Message.Receive(ws, &reply)
if err != nil {
fmt.Println("Can't receive")
break
}
fmt.Println("Received back from client: " + reply)
}
}
func main() {
http.Handle("/", websocket.Handler(Echo))
err := http.ListenAndServe(":12345", nil)
checkError(err)
}
func checkError(err error) {
if err != nil {
log.Fatalln("Fatal error ", err.Error())
}
}
```

按如下方式运行：

```
ch15$ go mod init example.com/user/echoserver
ch15$ go mod tidy
ch15$ go run echoserver.go
```

与该服务器通信的客户端文件是 `echoclient.go`：

```
ch15$ vi echoclient.go
/* EchoClient
*/
package main
import (
"fmt"
"golang.org/x/net/websocket"
"io"
"os"
"log"
)
func main() {
if len(os.Args) != 2 {
log.Fatalln("Usage: ", os.Args[0], "ws://host:port")
}
service := os.Args[1]
conn, err := websocket.Dial(service, "",
"http://localhost:12345")
checkError(err)
var msg string
for {
err := websocket.Message.Receive(conn, &msg)
if err != nil {
if err == io.EOF {
// graceful shutdown by server
break
}
fmt.Println("Couldn't receive msg " +
err.Error())
break
}
fmt.Println("Received from server: " + msg)
// return the msg
err = websocket.Message.Send(conn, msg)
if err != nil {
fmt.Println("Couldn't return msg")
break
}
}
}
func checkError(err error) {
if err != nil {
log.Fatalln("Fatal error ", err.Error())
}
}
```

按如下方式运行：

```
ch15$ go run echoclient.go ws://localhost:12345
```

*客户端*侧的输出内容即为*服务器*所发送的内容：

```
Received from server: Hello  0
...
Received from server: Hello  9
```

回到服务器端，我们能看到以下内容：

```
Echoing
Sending to client: Hello  0
Received back from client: Hello  0
...
Sending to client: Hello  9
Received back from client: Hello  9
```

上述会话证明了我们能够通过 WebSocket 发送和接收数据。请注意上述服务器和客户端代码中的 `Message` 对象及其用法。根据文档描述：

```
ch15$ go doc golang.org/x/net/websocket.Message
package websocket // import "golang.org/x/net/websocket"
var Message = Codec{marshal, unmarshal}
Message is a codec to send/receive text/binary data in a frame on WebSocket
connection. To send/receive text frame, use string type. To send/receive
binary frame, use []byte type.
Trivial usage:
import "websocket"
// receive text frame
var message string
websocket.Message.Receive(ws, &message)
// send text frame
message = "hello"
websocket.Message.Send(ws, message)
// receive binary frame
var data []byte
websocket.Message.Receive(ws, &data)
// send binary frame
data = []byte{0, 1, 2}
websocket.Message.Send(ws, data)
```

### JSON 对象

可以预见，许多 WebSocket 客户端和服务器将使用 JSON 格式交换数据。对于 Go 程序而言，这意味着 Go 对象将被序列化为 JSON 格式（如第 4 章所述），然后作为 UTF-8 字符串发送，而接收方将读取此字符串并将其反序列化回 Go 对象。

`websocket` 包中名为 `JSON` 的便捷对象可以为你完成此操作。它拥有 `Send` 和 `Receive` 方法，用于发送和接收数据，与 `Message` 对象类似。

我们考虑这样一个场景：客户端使用 WebSocket（它可以双向发送消息）向服务器发送一个 `Person` 对象。从客户端读取消息并将其打印到服务器标准输出的服务器文件是 `personserverjson.go`：

```
ch15$ vi personserverjson.go
/* PersonServerJSON
*/
package main
import (
"fmt"
"golang.org/x/net/websocket"
"log"
"net/http"
)
type Person struct {
Name   string
Emails []string
}
func ReceivePerson(ws *websocket.Conn) {
var person Person
err := websocket.JSON.Receive(ws, &person)
if err != nil {
fmt.Println("Can't receive")
} else {
fmt.Println("Name: " + person.Name)
for _, e := range person.Emails {
fmt.Println("An email: " + e)
}
}
}
func main() {
http.Handle("/", websocket.Handler(ReceivePerson))
err := http.ListenAndServe(":12345", nil)
checkError(err)
}
func checkError(err error) {
if err != nil {
log.Fatalln("Fatal error ", err.Error())
}
}
```

以 JSON 格式发送 `Person` 对象的客户端文件是 `personclientjson.go`：

```
ch15$ vi personclientjson.go
/* PersonClientJSON
*/
package main
import (
"fmt"
"golang.org/x/net/websocket"
"log"
"os"
)
type Person struct {
Name   string
Emails []string
}
func main() {
if len(os.Args) != 2 {
log.Fatalln("Usage: ", os.Args[0], "ws://host:port")
}
service := os.Args[1]
conn, err := websocket.Dial(service, "",
"http://localhost")
checkError(err)
person := Person{Name: "Jan",
Emails: []string{"ja@newmarch.name",
"jan.newmarch@gmail.com"},
}
err = websocket.JSON.Send(conn, person)
if err != nil {
fmt.Println("Couldn't send msg " + err.Error())
}
}
func checkError(err error) {
if err != nil {
log.Fatalln("Fatal error ", err.Error())
}
}
```

按如下方式运行服务器：

```
ch15$ go run personserverjson.go
```

按如下方式运行客户端：

```
ch15$ go run personclientjson.go ws://localhost:12345
```

*服务器*侧的输出内容即为*客户端*所发送的数据：

```
Name: Jan
An email: ja@newmarch.name
An email: jan.newmarch@gmail.com
```

与之前一样，查阅相关的 JSON 对象文档。

```
ch15$ go doc golang.org/x/net/websocket.JSON
package websocket // import "golang.org/x/net/websocket"
var JSON = Codec{jsonMarshal, jsonUnmarshal}
JSON is a codec to send/receive JSON data in a frame from a WebSocket
connection.
Trivial usage:
import "websocket"
type T struct {
Msg string
Count int
}
// receive JSON type T
var data T
websocket.JSON.Receive(ws, &data)
// send JSON type T
websocket.JSON.Send(ws, data)
```



### 编解码器类型

`Message` 和 `JSON` 对象都是 `Codec` 类型的实例。该类型定义如下：

```
type Codec struct {
	Marshal   func(v interface{}) (data []byte, payloadType byte, err error)
	Unmarshal func(data []byte, payloadType byte, v interface{}) (err error)
}
```

`Codec` 类型实现了前面用到的 `Send` 和 `Receive` 方法。要了解更多关于该类型的信息，请运行：

`ch15$ go doc golang.org/x/net/websocket.Codec`

WebSocket 很可能也会用于交换 XML 数据。我们可以通过封装第 12 章讨论的 XML `marshal` 和 `unmarshal` 方法来构建一个 XML `Codec` 对象，从而得到一个合适的 `Codec` 对象。

我们可以创建一个名为 `xmlcodec.go` 的 `XMLCodec` 包，具体如下：

`ch15$ vi xmlcodec.go`

```
/* XMLCodec
*/
package main
import (
	"encoding/xml"
	"golang.org/x/net/websocket"
)
func xmlMarshal(v interface{}) (msg []byte, payloadType byte, err error) {
	msg, err = xml.Marshal(v)
	return msg, websocket.TextFrame, nil
}
func xmlUnmarshal(msg []byte, payloadType byte, v interface{}) (err error) {
	err = xml.Unmarshal(msg, v)
	return err
}
var XMLCodec = websocket.Codec{xmlMarshal, xmlUnmarshal}
```

然后，我们可以将诸如 `Person` 这样的 Go 对象序列化为 XML 文档，并从客户端发送到服务器。接收该文档并将其打印到标准输出的服务器代码如下 (`personserverxml.go`)：

`ch15$ vi personserverxml.go`

```
/* PersonServerXML
*/
package main
import (
	"fmt"
	"golang.org/x/net/websocket"
	"log"
	"net/http"
)
type Person struct {
	Name   string
	Emails []string
}
func ReceivePerson(ws *websocket.Conn) {
	var person Person
	err := XMLCodec.Receive(ws, &person)
	if err != nil {
		fmt.Println("Can't receive")
	} else {
		fmt.Println("Name: " + person.Name)
		for _, e := range person.Emails {
			fmt.Println("An email: " + e)
		}
	}
}
func main() {
	http.Handle("/", websocket.Handler(ReceivePerson))
	err := http.ListenAndServe(":12345", nil)
	checkError(err)
}
func checkError(err error) {
	if err != nil {
		log.Fatalln("Fatal error ", err.Error())
	}
}
```

以 XML 形式发送 `Person` 对象的客户端代码为 `personclientxml.go`：

`ch15$ vi personclientxml.go`

```
/* PersonClientXML
*/
package main
import (
	"fmt"
	"golang.org/x/net/websocket"
	"log"
	"os"
)
type Person struct {
	Name   string
	Emails []string
}
func main() {
	if len(os.Args) != 2 {
		log.Fatalln("Usage: ", os.Args[0], "ws://host:port")
	}
	service := os.Args[1]
	conn, err := websocket.Dial(service, "", "http://localhost")
	checkError(err)
	person := Person{Name: "Jan",
		Emails: []string{"ja@newmarch.name",
			"jan.newmarch@gmail.com"},
	}
	err = XMLCodec.Send(conn, person)
	if err != nil {
		fmt.Println("Couldn't send msg " + err.Error())
	}
	os.Exit(0)
}
func checkError(err error) {
	if err != nil {
		log.Fatalln("Fatal error ", err.Error())
	}
}
```

服务器运行方式如下：

`ch15$ go run personserverxml.go xmlcodec.go`

客户端运行方式如下：

`ch15$ go run personclientxml.go xmlcodec.go ws://localhost:12345`

*服务器*端的输出即*客户端*发送的内容：

```
Name: Jan
An email: ja@newmarch.name
An email: jan.newmarch@gmail.com
```

和之前一样，请查阅相关的编解码器对象文档（`go doc encoding/xml`）。

下一步合理的改进是增强安全性；接下来我们将 WebSocket 升级为使用 TLS。

### 基于 TLS 的 WebSocket

WebSocket 可以构建在安全的 TLS 套接字之上。我们在第 8 章讨论了如何使用第 7 章中的证书来使用 TLS 套接字。这种方法在 WebSocket 中保持不变。也就是说，我们使用 `http.ListenAndServeTLS` 代替 `http.ListenAndServe`。

以下是一个使用 TLS 的回声服务器 `echoservertls.go`：

`ch15$ vi echoservertls.go`

```
/* EchoServerTLS
*/
package main
import (
	"log"
	"fmt"
	"golang.org/x/net/websocket"
	"net/http"
)
func Echo(ws *websocket.Conn) {
	fmt.Println("Echoing")
	for n := 0; n < 10; n++ {
		msg := "Hello  " + string(n+48)
		fmt.Println("Sending to client: " + msg)
		err := websocket.Message.Send(ws, msg)
		if err != nil {
			fmt.Println("Can't send")
			break
		}
		var reply string
		err = websocket.Message.Receive(ws, &reply)
		if err != nil {
			fmt.Println("Can't receive")
			break
		}
		fmt.Println("Received back from client: " + reply)
	}
}
func main() {
	http.Handle("/", websocket.Handler(Echo))
	err := http.ListenAndServeTLS(":12345",
		"jan.newmarch.name.pem",
		"private.pem", nil)
	checkError(err)
}
func checkError(err error) {
	if err != nil {
		log.Fatalln("Fatal error ", err.Error())
	}
}
```

在运行服务器之前，我们需要从第 7/8 章创建并使用的证书和密钥。

```
ch15$ cp ../ch7/jan.newmarch.name.pem . ; cp ../ch7/private.pem .
ch15$ go run echoservertls.go
```

客户端与之前的回声客户端相同。唯一变化的是 URL，它使用 `wss` 方案代替了 `ws` 方案：

```
ch15$ go run echoclient wss://localhost:12345/
Fatal error  websocket.Dial wss://localhost:12345: x509: certificate signed by unknown authority
exit status 1
```

如果服务器提供的 TLS 证书是有效的，这就能正常工作。但我使用的证书无效：它是自签名的，这通常是你即将进入危险区域的信号。如果仍然要继续，你需要采用与第 8 章相同的“移除安全检查”方法，即启用 TLS `InsecureSkipVerify` 标志。程序 `echoclienttls.go` 就是通过设置一个使用该标志的配置，然后调用 `DialConfig` 代替 `Dial` 来实现的。

`ch15$ vi echoclienttls.go`

```
/* EchoClientTLS
*/
package main
import (
	"crypto/tls"
	"fmt"
	"golang.org/x/net/websocket"
	"io"
	"log"
	"os"
)
func main() {
	if len(os.Args) != 2 {
		log.Fatalln("Usage: ", os.Args[0], "wss://host:port")
	}
	config, err := websocket.NewConfig(os.Args[1],
		"http://localhost")
	checkError(err)
	tlsConfig := &tls.Config{InsecureSkipVerify: true}
	config.TlsConfig = tlsConfig
	conn, err := websocket.DialConfig(config)
	checkError(err)
	var msg string
	for {
		err := websocket.Message.Receive(conn, &msg)
		if err != nil {
			if err == io.EOF {
				// 服务器优雅关闭
				break
			}
			fmt.Println("Couldn't receive msg " +
				err.Error())
			break
		}
		fmt.Println("Received from server: " + msg)
		// 返回消息
		err = websocket.Message.Send(conn, msg)
		if err != nil {
			fmt.Println("Couldn't return msg")
			break
		}
	}
}
func checkError(err error) {
	if err != nil {
		log.Fatalln("Fatal error ", err.Error())
	}
}
```

运行客户端如下（假设 `echoservertls.go` 正在运行）：

```
ch15$ go run echoclienttls.go wss://localhost:12345
Received from server: Hello  0
...
Received from server: Hello  9
```

回到服务器端，我们看到以下输出：

```
Echoing
Sending to client: Hello  0
Received back from client: Hello  0
...
Sending to client: Hello  9
Received back from client: Hello  9
```



### HTML 页面中的 WebSocket

WebSocket 最初的驱动力是允许浏览器等 HTTP 用户代理与服务器之间进行全双工交互。典型用例通常是浏览器中的 JavaScript 程序与服务器进行交互。在本节中，我们将构建一个 Web/WebSocket 服务器，该服务器将提供一个 HTML 页面，该页面会建立一个 WebSocket，并使用 WebSocket 显示来自该服务器的信息。我们正在研究如图 15-1 所示的场景。

![](img/436770_2_En_15_Fig1_HTML.png)

图 15-1

全双工交互场景

物联网（IoT）时代已经来临。因此，我们可以预期来自传感器和传感器网络的数据将用于驱动执行器，并在浏览器中显示关于物联网网络的信息。有无数关于使用树莓派和 Arduino 构建传感器网络的书籍，但我们将通过在一个“传感器”上显示 CPU 温度，并在网页中每隔几秒更新一次，来大幅简化此场景。

来自 Debian 软件包 `lm-sensors` 的 Linux `sensors` 命令会将其所知的传感器值写入标准输出。在我的台式机上，`sensors` 命令的输出如下所示：

```
acpitz-virtual-0
Adapter: Virtual device
temp1:        +27.8°C  (crit = +105.0°C)
temp2:        +29.8°C  (crit = +105.0°C)
coretemp-isa-0000
Adapter: ISA adapter
Physical id 0:  +58.0°C  (high = +105.0°C, crit = +105.0°C)
Core 0:         +57.0°C  (high = +105.0°C, crit = +105.0°C)
Core 1:         +58.0°C  (high = +105.0°C, crit = +105.0°C)
```

刷新时，`Core 0` 和 `Core 1` 的温度通常可能会发生变化。

在 Windows 上，执行相同操作的命令如下：

```
wmic /namespace:\\root\wmi PATH MSAcpi_ThermalZoneTemperature get CurrentTemperature
```

运行时，其输出类似：

```
42.4° C
```

在 Mac 上，使用来自 [`https://github.com/lavoiesl/osx-cpu-temp`](https://github.com/lavoiesl/osx-cpu-temp) 的命令 `osx-cpu-temp`。如果你不想经历这些步骤，只需替换为一个更简单的程序，例如 `date`。

我们提供了一个 Go 程序，用于从 `ROOT_DIR` 目录提供 HTML 文档，然后从 URL `GetTemp` 建立一个 WebSocket。服务器端的 WebSocket 每两秒获取 `sensors` 的输出，并将其发送到套接字的客户端。出于某种原因，这个 Web/WebSocket 服务器运行在端口 `12345` 上。将 `exec.Command` 调用替换为任何其他有趣的系统调用。这里，我们使用简单的 `sensors.sh` 作为我们的温度程序（1 到 100 之间的随机数）。

```
ch15$ vi sensors.sh
#!/bin/sh
echo $(( ( RANDOM % 100 )  + 1 ))
```

确保使其可执行：`chmod 700 sensors.sh`。

服务器是 `temperatureserver.go`：

```
ch15$ vi temperatureserver.go
/* TemperatureServer
*/
package main
import (
"fmt"
"golang.org/x/net/websocket"
"log"
"net/http"
"os/exec"
"time"
)
var ROOT_DIR = "."
func GetTemp(ws *websocket.Conn) {
for {
msg, err := exec.Command(ROOT_DIR + "/sensors.sh").CombinedOutput()
checkError(err)
fmt.Println("Sending to client: " + string(msg[:]))
err = websocket.Message.Send(ws, string(msg[:]))
if err != nil {
fmt.Println("Can't send")
break
}
time.Sleep(time.Duration(2) * time.Second)
var reply string
err = websocket.Message.Receive(ws, &reply)
if err != nil {
fmt.Println("Can't receive")
break
}
fmt.Println("Received back from client: " + reply)
}
}
func main() {
fileServer := http.FileServer(http.Dir(ROOT_DIR))
http.Handle("/GetTemp", websocket.Handler(GetTemp))
http.Handle("/", fileServer)
err := http.ListenAndServe(":12345", nil)
checkError(err)
}
func checkError(err error) {
if err != nil {
log.Fatalln("Fatal error ", err.Error())
}
}
```

按如下方式运行：

```
ch15$ go run temperatureserver.go
```

用于启动此功能的顶级 HTML 文件是 `websocket.html`；请确保将此文件复制到 `ROOT_DIR`。

```

function WebSocketTest()
{
if ("WebSocket" in window)
{
alert("WebSocket is supported by your Browser!");
// Let us open a web socket
var ws = new WebSocket("ws://localhost:12345/GetTemp");
ws.onopen = function()
{
alert("WS is opened...");
};
ws.onmessage = function (evt)
{
var received_msg = evt.data;
// uncomment next line if you want to get alerts on each message
//alert("Message is received..." + received_msg);
document.getElementById("temp").innerHTML = "" + received_msg + ""
ws.send("Message received")
};
ws.onclose = function()
{
// websocket is closed.
alert("Connection is closed...");
};
}
else
{
// The browser doesn't support WebSocket
alert("WebSocket NOT supported by your Browser!");
}
}

Run temperature sensor

```

在浏览器中访问 `http://localhost:12345/websocket.html`；点击“Run temperature sensor”以查看温度中继。

该程序使用 JavaScript 打开一个 WebSocket 连接，并处理 `onopen`、`onmessage` 和 `onclose` 事件。它使用 `evt.data` 和 `send` 函数进行读写。它在一个预格式化元素中呈现数据，数据格式与之前完全相同。每两秒刷新一次。HTML 文档的结构基于 HTML5 - WebSocket：[`https://www.tutorialspoint.com/html5/html5_websocket.htm`](https://www.tutorialspoint.com/html5/html5_websocket.htm)。

## `github.com/gorilla/websocket` 包

用于 WebSocket 功能的替代包是 `github.com/gorilla/websocket` 包。

### 回声服务器

使用此包的回声服务器是 `echoservergorilla.go`。它通过引入对 `websocket.Upgrader` 对象的调用，使 HTTP 到 WebSocket 的转换更加明确。它还更清楚地区分发送文本消息和二进制消息。

```
ch15$ vi echoservergorilla.go
/* EchoServerGorilla
*/
package main
import (
"fmt"
"github.com/gorilla/websocket"
"log"
"net/http"
)
var upgrader = websocket.Upgrader{
ReadBufferSize:  1024,
WriteBufferSize: 1024,
}
func Handler(w http.ResponseWriter, r *http.Request) {
fmt.Println("Handling /")
conn, err := upgrader.Upgrade(w, r, nil)
if err != nil {
fmt.Println(err)
return
}
for n := 0; n < 10; n++ {
msg := "Hello  " + string(n+48)
fmt.Println("Sending to client: " + msg)
err = conn.WriteMessage(websocket.TextMessage,
[]byte(msg))
_, reply, err := conn.ReadMessage()
if err != nil {
fmt.Println("Can't receive")
break
}
fmt.Println("Received back from client: " +
string(reply[:]))
}
conn.Close()
}
func main() {
http.HandleFunc("/", Handler)
err := http.ListenAndServe("localhost:12345", nil)
checkError(err)
}
func checkError(err error) {
if err != nil {
log.Fatalln("Fatal error ", err.Error())
}
}
```

按如下方式运行服务器：

```
ch15$ go mod init example.com/user/echoservergorilla # or skip and use existing module name
ch15$ go mod tidy
ch15$ go run echoservergorilla.go
```



### Echo 客户端

使用此包的 Echo 客户端是 `echoclientgorilla.go`：

```go
/* EchoClientGorilla
*/
package main

import (
    "fmt"
    "github.com/gorilla/websocket"
    "io"
    "log"
    "net/http"
    "os"
)

func main() {
    if len(os.Args) != 2 {
        log.Fatalln("Usage: ", os.Args[0], "ws://host:port")
    }
    service := os.Args[1]
    header := make(http.Header)
    header.Add("Origin", "http://localhost:12345")
    conn, _, err := websocket.DefaultDialer.Dial(service, header)
    checkError(err)
    for {
        _, reply, err := conn.ReadMessage()
        if err != nil {
            if err == io.EOF {
                // graceful shutdown by server
                fmt.Println(`EOF from server`)
                break
            }
            if websocket.IsCloseError(err, websocket.CloseAbnormalClosure) {
                fmt.Println(`Close from server`)
                break
            }
            fmt.Println("Couldn't receive msg " + err.Error())
            break
        }
        fmt.Println("Received from server: " + string(reply[:]))
        // return the msg
        err = conn.WriteMessage(websocket.TextMessage, reply)
        if err != nil {
            fmt.Println("Couldn't return msg")
            break
        }
    }
}

func checkError(err error) {
    if err != nil {
        log.Fatalln("Fatal error ", err.Error())
    }
}
```

客户端运行方式如下：

```bash
ch15$ go run echoclientgorilla.go ws://localhost:12345
Received from server: Hello  0
...
Received from server: Hello  9
Close from server
```

回到服务器端，我们可以看到：

```
Handling /
Sending to client: Hello  0
Received back from client: Hello  0
Sending to client: Hello  1
...
Sending to client: Hello  9
Received back from client: Hello  9
```

## 结论

WebSocket 标准是 IETF 的 RFC，因此预计不会有重大变化。这将允许 HTTP 用户代理和服务器建立双向套接字连接，并应使某些交互风格变得更加简单。Go 支持这些以及额外的 WebSocket 包，例如 [`https://pkg.go.dev/nhooyr.io/websocket`](https://pkg.go.dev/nhooyr.io/websocket)。

# 16. Gorilla

Go 标准库以及相关的“x”包提供了大量功能，我们可以利用这些功能来构建网站。Go 倾向于尽可能采用更简单的编码风格；但简单并不总意味着容易。除非你在领域（Web 开发）和实现栈（Go/Web）方面拥有足够的经验，否则使用一个提供额外易用性功能的工具包可能更容易和/或更好。Gorilla 就是这样一个工具包。我们已经通过 WebSocket 接触过 Gorilla；现在我们将更集中地审视其主要功能，并最终了解它相对于纯 Go 实现为我们带来了什么。

根据 Gorilla 网站 ([`https://www.gorillatoolkit.org/`](https://www.gorillatoolkit.org/)) 的描述，以下是当前包含的包。我们将学习如何使用其中一些包，并探索它们如何集成和扩展标准库。

*   `mux`：一个功能强大的 URL 路由器和调度器，与默认的 `http.ServeMux` 完全兼容
*   `reverse`：为基于正则表达式的多路复用器生成可逆的正则表达式
*   `rpc`：通过 HTTP 实现 RPC，并使用编解码器支持 JSON-RPC
*   `schema`：将表单值转换为结构体
*   `securecookie`：对经过身份验证且可选择加密的 cookie 值进行编码和解码
*   `sessions`：保存 cookie 和文件系统会话，并允许自定义会话后端
*   `websocket`：实现 RFC 6455 中定义的 WebSocket 协议
*   `csrf`：提供跨站请求伪造 (CSRF) 预防中间件
*   `handlers`：为 Go 的 `net/http` 包收集的有用处理器集合

你可能会注意到，由于其精心设计的架构，这是一个顶级的工具选择。接下来，我们将讨论中间件模式，这将指导我们如何集成 Gorilla。

## 中间件模式

根据 [`https://en.wikipedia.org/wiki/Middleware`](https://en.wikipedia.org/wiki/Middleware) 的定义，中间件被描述为“软件胶水”。Web 服务的中间件允许我们将功能包装在我们的应用特定代码周围。这些包装器又可以形成链式结构。我们这样做是为了添加诸如日志记录和/或身份验证等功能，而不是污染我们的业务逻辑。例如，在我们的客户端处理逻辑中不需要“记录”HTTP 请求；它可以（并且应该）在它外部（在另一个包装器中）完成。

基本的中间件模式如下所示：

```
路由器 -> 中间件 -> 应用程序
```

我们从一个简单的例子开始，并回顾 Go 的一些内置 Web 功能。这将帮助我们理解 Gorilla 如何与我们的代码（以及 Go 标准库）集成，以及它提供了哪些超越标准库的功能。

Go 通过 `net/http` 包提供了 HTTP 客户端和服务器的实现。该包提供了一个 `type Server`，它与底层连接对象协同工作，并反序列化 HTTP 请求或序列化 HTTP 响应。然后，该请求被交给一个 `type ServeMux`。`ServeMux` 反过来为我们执行路由（即多路复用）。提供了一个名为 `DefaultServeMux` 的默认 `ServeMux`。还提供了一个名为 `net.Handler` 的接口，该处理程序提供了处理 HTTP 请求时使用的方法。如前所述，我们可以拥有多个处理程序，并将这些处理程序链接在一起。事实证明，`ServeMux` 也是一个 `Handler`，这意味着我们也可以将它们链接起来。

现在我们可以更深入地想象 Go 本身是如何使用中间件模式的。

```
连接  服务器  ServeMux  处理程序  应用程序函数
```

让我们回顾一下从 `Handler` 向后的主要文档。

```bash
$ go doc http.Handler
package http // import "net/http"

type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}

Handler 响应 HTTP 请求。

ServeHTTP 应将回复标头和数据写入 ResponseWriter，然后返回。
返回表示请求已完成；在 ServeHTTP 调用完成后或与其同时，
使用 ResponseWriter 或从 Request.Body 读取是无效的。

根据 HTTP 客户端软件、HTTP 协议版本以及客户端与 Go 服务器之间的任何中介，
在写入 ResponseWriter 之后可能无法从 Request.Body 读取。
谨慎的处理程序应先读取 Request.Body，然后再进行回复。

除了读取 body 之外，处理程序不应修改提供的 Request。

如果 ServeHTTP 发生 panic，服务器（ServeHTTP 的调用者）会假定 panic 的影响
仅限于活动请求。它会恢复 panic，将堆栈跟踪记录到服务器错误日志，
然后根据 HTTP 协议，要么关闭网络连接，要么发送 HTTP/2 RST_STREAM。
要中止处理程序以使客户端看到中断的响应但服务器不记录错误，
请使用值 ErrAbortHandler 进行 panic。

func AllowQuerySemicolons(h Handler) Handler
func FileServer(root FileSystem) Handler
func MaxBytesHandler(h Handler, n int64) Handler
func NotFoundHandler() Handler
func RedirectHandler(url string, code int) Handler
func StripPrefix(prefix string, h Handler) Handler
func TimeoutHandler(h Handler, dt time.Duration, msg string) Handler
```

目前，关键部分围绕 `Handler.ServeHTTP`。这是请求传入并最终得到响应的地方。接下来，查看 `ServeMux` 的详细信息以及与 `Handler` 的关系。



```go
$ go doc net/http ServeMux
package http // import "net/http"
type ServeMux struct {
// 包含未导出的字段
}

ServeMux 是一个 HTTP 请求多路复用器。它会将每个传入请求的 URL 与已注册的模式列表进行匹配，并调用与 URL 最匹配的模式对应的处理程序。

模式指定固定的、以根路径开头的路径，例如 "/favicon.ico"，或以根路径开头的子树，例如 "/images/"（注意尾部斜杠）。较长的模式优先于较短的模式，因此，如果同时为 "/images/" 和 "/images/thumbnails/" 注册了处理程序，那么对于以 "/images/thumbnails/" 开头的路径将调用后者，而前者将接收 "/images/" 子树中其他任何路径的请求。

请注意，由于以斜杠结尾的模式表示一个根子树，因此模式 "/" 会匹配所有未被其他已注册模式匹配的路径，而不仅仅是 Path == "/" 的 URL。

如果已注册了一个子树，并且接收到的请求指定了该子树的根路径但没有尾部斜杠，ServeMux 会将该请求重定向到子树根路径（添加尾部斜杠）。可以通过为没有尾部斜杠的路径单独注册来覆盖此行为。例如，注册 "/images/" 会导致 ServeMux 将 "/images" 的请求重定向到 "/images/"，除非 "/images" 已被单独注册。

模式可以可选地以主机名开头，从而将匹配限制为仅匹配该主机上的 URL。特定于主机的模式优先于通用模式，因此处理程序可以为两个模式 "/codesearch" 和 "codesearch.google.com/" 注册，而不会同时接管对 "http://www.google.com/" 的请求。

ServeMux 还负责清理 URL 请求路径和 Host 头，去除端口号，并将任何包含 "." 或 ".." 元素或重复斜杠的请求重定向到一个等效且更简洁的 URL。

func NewServeMux() *ServeMux
func (mux *ServeMux) Handle(pattern string, handler Handler)
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request))
func (mux *ServeMux) Handler(r *Request) (h Handler, pattern string)
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request)
```

如你所见，`ServeMux` 匹配给定的模式并将其传递给配置的 `Handler`，然后该处理程序再调用我们的业务逻辑。继续向左看，我们来查阅简略的 `Server` 文档。

```
$ go doc net/http.Server
package http // import "net/http"
type Server struct {
// Addr 可选择指定服务器监听的 TCP 地址，格式为 "host:port"。
// 如果为空，则使用 ":http"（端口 80）。
// 服务名称在 RFC 6335 中定义，并由 IANA 分配。
// 有关地址格式的详细信息，请参阅 net.Dial。
Addr string
Handler Handler // 要调用的处理程序，如果为 nil，则使用 http.DefaultServeMux
...
```

虽然有所截断，但我们可以在一开始就看到，一个 `Server` 持有一个 `Handler`，并且根据注释，似乎我们有一个名为 `DefaultServeMux` 的 `ServeMux` 实例。

了解了连接关系后，我们现在来看一些例子。

## 标准库 ServeMux 示例

在这里，我们演示了 Go 可能提供的最简单的 Web 服务（至少在 `net/http` 包中），以及一个更清晰地展示中间件模式的示例。

```
$ mkdir ch16
$ ch16
ch16$ vi simple.go
package main
import (
"net/http"
"fmt"
)
func main() {
err := http.ListenAndServe(":8080",nil)
fmt.Println(err)
}
```

运行服务器 `go run simple.go`，通过任何方法/路径访问 `localhost:8080` 都会返回 404。

```
ch16$ curl -v --head localhost:8080
*   Trying ::1:8080...
* Connected to localhost (::1) port 8080 (#0)
> HEAD / HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.77.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
HTTP/1.1 404 Not Found
< Content-Type: text/plain; charset=utf-8
Content-Type: text/plain; charset=utf-8
< X-Content-Type-Options: nosniff
X-Content-Type-Options: nosniff
< Date: Mon, 31 Jan 2022 00:43:57 GMT
Date: Mon, 31 Jan 2022 00:43:57 GMT
< Content-Length: 19
Content-Length: 19
<
* Connection #0 to host localhost left intact
```

这表明存在一个默认处理程序，返回 404。接下来，我们尝试一个 GET 请求。

```
ch16$ curl -X GET localhost:8080/didthiswork
404 page not found
```

事实证明，在底层，我们已经有一个处理程序链。`Server` 正在调用 `ServeMux.ServeHTTP`，而后者又会调用 `NotFoundHandler`。`ServeMux` 通过 `ServeHTTP` 方法支持 `Handler` 接口。作为敏锐的观察者，你可能已经注意到，在前面的 `Handler` 文档中，列出了 `NotFoundHandler`。

```
ch16$ go doc net/http NotFoundHandler
package http // import "net/http"
func NotFoundHandler() Handler
NotFoundHandler 返回一个简单的请求处理程序，该程序对每个请求都会回复“404 page not found”的响应。
```

如果你喜欢冒险，可以查看代码，开始了解它们是如何连接起来的。

```
ch16$ vi /usr/local/go/src/net/http/server.go
...
// NotFound 回复一个 HTTP 404 not found 错误给请求。
func NotFound(w ResponseWriter, r *Request) { Error(w, "404 page not found", StatusNotFound) }
// NotFoundHandler 返回一个简单的请求处理程序
// 该程序对每个请求都会回复“404 page not found”的响应。
func NotFoundHandler() Handler { return HandlerFunc(NotFound) }
...
```

如你所见，`NotFound` 函数被转换为 `HandlerFunc`（在我们的示例中，它将 `ServeHTTP` 调用代理给 `NotFound`）。`HandlerFunc` 在 Go 中被称为函数适配器。

## 自定义多路复用器

我们最后一个专注于标准库的例子将展示如何管理多个 `ServeMux` 实例。此示例的目标是展示“注册”一个新的 `ServeMux` 如何处理路由的一个子集。在这里，“默认的”（`DefaultServeMux`）处理路由 `/outermux`，而我们新的“内部”`ServeMux` 将处理更深层嵌套的路由（例如，`/outermux/innermux`）。

```
ch16$ vi complex.go
package main
import (
"net/http"
)
func main() {
http.Handle("/outermux", http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
w.Write([]byte("/outermux"))
}))
inner := http.NewServeMux()
inner.Handle("/innermux/", http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
w.Write([]byte("../innermux"))
}))
http.Handle("/outermux/innermux/", http.StripPrefix("/outermux", inner))
http.ListenAndServe(":8080", nil)
}
```

在我们接下来执行测试时，请注意请求的路径和生成的日志。根据前面 `ServeMux` 的文档，要特别注意如何结束路径，是否带有尾部斜杠 `/`。

```
ch16$ curl localhost:8080/outermux
/outermux
ch16$ curl localhost:8080/outermux/
404 page not found
ch16$ curl localhost:8080/outermux/innermux
Moved Permanently.
ch16$ curl -iL localhost:8080/outermux/innermux
HTTP/1.1 301 Moved Permanently
Content-Type: text/html; charset=utf-8
Location: /outermux/innermux/
Date: Mon, 31 Jan 2022 01:16:24 GMT
Content-Length: 54
HTTP/1.1 200 OK
Date: Mon, 31 Jan 2022 01:16:24 GMT
Content-Length: 11
Content-Type: text/plain; charset=utf-8
../innermux
ch16$ curl -iL localhost:8080/outermux/innermux/
HTTP/1.1 200 OK
Date: Mon, 31 Jan 2022 01:16:28 GMT
Content-Length: 11
Content-Type: text/plain; charset=utf-8
../innermux
```

你可能想知道最后一行是做什么的 (`http.ListenAndServe(":8080", nil)`)。

具体来说，`nil` 是做什么的？这就是我们可以覆盖默认 `ServeMux` 的地方。当你传入 `nil` 时，“第一个”多路复用器仍然是 `Serve` 类型使用的预创建实例 `DefaultServeMux`。你可以通过查看 `GOROOT/src/net/http/server.go` 中的代码来确认这些关系。



## `gorilla/mux`

我们通过学习 `mux` 包来开始了解 Gorilla。`mux` 包旨在与标准库中现有的 `Serve` 类型协同工作。

```
ch16$ vi gmux.go
package main
import (
"net/http"
"github.com/gorilla/mux"
)
func buildHandler(message string) func(http.ResponseWriter, *http.Request) {
return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
w.Write([]byte(message))
})
}
func main() {
r := mux.NewRouter()
r.HandleFunc("/", buildHandler("HomeHandler"))
r.HandleFunc("/products", buildHandler("ProductsHandler"))
r.HandleFunc("/articles", buildHandler("ArticlesHandler"))
http.ListenAndServe(":8080", r)
}
```

根据我们已知的知识，通过覆盖 `ListenAndServe` 的最后一个参数，我们实际上是用自己的路由器（在 mux 的语境中称为 `Router`）覆盖了默认的 `ServeMux`。

```
ch16$ go mod init ch16.example.com
ch16$ go mod tidy
ch16$ go run gmux.go
```

接下来的测试有望证明一切并未发生太大变化。

```
ch16$ curl localhost:8080/articles
ArticlesHandler
ch16$ curl localhost:8080/
HomeHandler
```

如果我们审视 `Router` 的代码，特别是其 `ServerHTTP` 方法，我们就能确认路由映射现在由 Gorilla 的 `mux` 负责处理。

```
~/ch16$ go doc --src mux Router.ServeHTTP
package mux // import "github.com/gorilla/mux"
// ServeHTTP 分发在匹配路由中注册的处理器。
//
// 当匹配成功时，可以通过调用 mux.Vars(request) 来获取路由变量。
func (r *Router) ServeHTTP(w http.ResponseWriter, req *http.Request) {
if !r.skipClean {
path := req.URL.Path
if r.useEncodedPath {
path = req.URL.EscapedPath()
}
// 清理路径为标准形式并进行重定向。
if p := cleanPath(path); p != path {
// 增加了 3 行代码 (Philip Schlump) - 修复了丢失查询字符串和 #whatever 的问题。
// 这与 Go 1.2 r.c.4 中针对相同问题的修复一致。Go Issue:
// http://code.google.com/p/go/issues/detail?id=5252
url := *req.URL
url.Path = p
p = url.String()
w.Header().Set("Location", p)
w.WriteHeader(http.StatusMovedPermanently)
return
}
}
var match RouteMatch
var handler http.Handler
if r.Match(req, &match) {
handler = match.Handler
req = requestWithVars(req, match.Vars)
req = requestWithRoute(req, match.Route)
}
if handler == nil && match.MatchErr == ErrMethodMismatch {
handler = methodNotAllowedHandler()
}
if handler == nil {
handler = http.NotFoundHandler()
}
handler.ServeHTTP(w, req)
}
```

## 为什么要关注

到目前为止，我们似乎并没有为已经提供的 `DefaultServeMux` 增添任何价值。然而，表象是会骗人的。根据 `ServeMux` 的文档，我们似乎只能轻松解析“路径”来做出路由决策。我们能否基于其他条件（除了路径）来匹配请求，比如查询参数或 HTTP 头部信息？没有 Gorilla 我们也能够访问这些内容；但我们缺少一种结构化的使用方式。

请求可以根据以下条件进行匹配：

- URL 主机名 (`example.com`)
- 路径 (`/about`)
- 路径前缀 (`/animals/cats`)
- 模式（已提交表单值）
- HTTP 头部信息 (`Content-Type: text/html; charset=UTF-8`)
- 查询参数 (`?key=value&dog=cat`)
- HTTP 方法 (`GET /search?q=test HTTP/2`)
- 自定义匹配器（自定义函数）

Gorilla 的一个优秀特性是能够使用链式调用来关联多个匹配元素。下面是一个示例，通过使用各种匹配器来修改我们之前的例子。

```
~/ch16$ vi gmux.go
package main
import (
"github.com/gorilla/mux"
"net/http"
)
func buildHandler(message string) func(http.ResponseWriter, *http.Request) {
return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
w.Write([]byte(message))
})
}
func main() {
r := mux.NewRouter()
r.HandleFunc("/", buildHandler("HomeHandler"))
r.HandleFunc("/products", buildHandler("ProductsHandler"))
r.HandleFunc("/articles", buildHandler("ArticlesHandler")).Host("example.com").Methods("GET").Schemes("http")
http.ListenAndServe(":8080", r)
}
```

运行服务器。

```
~/ch16$ go run gmux.go
```

现在我们来测试我们新增和已有的路由；注意 `/articles` 现在受到特定域名、方法和模式的限制。

```
~/ch16$ curl --resolve example.com:8080:127.0.0.1 -IX GET http://example.com:8080/articles
HTTP/1.1 200 OK
Date: Mon, 07 Mar 2022 18:40:46 GMT
Content-Length: 15
Content-Type: text/plain; charset=utf-8
~/ch16$ curl --resolve example.com:8080:127.0.0.1 -IX POST http://example.com:8080/articles
HTTP/1.1 405 Method Not Allowed
Date: Mon, 07 Mar 2022 18:40:54 GMT
Content-Length: 0
~/ch16$ curl 127.0.0.1:8080/articles
404 page not found
~/ch16$
```

以下是官方文档中的一些示例：

- `r.PathPrefix("/products/")`
- `r.Methods("GET", "POST")`
- `r.Schemes("https")`
- `r.Headers("X-Requested-With", "XMLHttpRequest")`
- `r.Queries("key", "value")`
- `r.MatcherFunc(func(r *http.Request, rm *RouteMatch) bool { return r.ProtoMajor == 0 })`

再次强调，使用标准库也能实现所有这些功能（很多开发者认为你应该这么做），但手头有一个现成的工具包会让选择变得更容易。Gorilla `mux` 的功能远不止于此。我们的目标是突出它与标准库的关系以及它是如何进行扩展（提供额外功能）的。你可以在这里了解更多：[`https://github.com/gorilla/mux`](https://github.com/gorilla/mux)。



## Gorilla 处理器

Gorilla 在 `"handlers"` 包中提供了一组处理器。秉承简洁易用的主题，其中包含以下处理器供我们在中间件链中使用。

以下是官方说明：

* `LoggingHandler`：用于以 Apache 通用日志格式记录 HTTP 请求
* `CombinedLoggingHandler`：用于以 Apache 和 Nginx 常用的 Apache 组合日志格式记录 HTTP 请求
* `CompressHandler`：用于对响应进行 gzip 压缩
* `ContentTypeHandler`：用于根据一系列可接受的内容类型验证请求
* `MethodHandler`：用于将 HTTP 方法与 `map[string]http.Handler` 中的处理器进行匹配
* `ProxyHeaders`：用于在 Go 服务器运行于 HTTP 反向代理之后时，基于 `X-Forwarded-For`、`X-Real-IP`、`X-Forwarded-Proto` 和 RFC7239 `Forwarded` 头部填充 `r.RemoteAddr` 和 `r.URL.Scheme`
* `CanonicalHost`：用于在处理多个域名（即多个 CNAME 别名）时重定向到首选主机
* `RecoveryHandler`：用于从意外的 panic 中恢复

让我们从 `LoggingHandlers` 开始，看其中几个例子。

```
~/ch16$ go get github.com/gorilla/handlers
~/ch16$ go doc handlers.LoggingHandler
package handlers // import "github.com/gorilla/handlers"
func LoggingHandler(out io.Writer, h http.Handler) http.Handler
LoggingHandler return a http.Handler that wraps h and logs requests to out
in Apache Common Log Format (CLF).
See http://httpd.apache.org/docs/2.2/logs.html#common for a description of
this format.
LoggingHandler always sets the ident field of the log to -
Example:
r := mux.NewRouter()
r.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
w.Write([]byte("This is a catch-all route"))
})
loggedRouter := handlers.LoggingHandler(os.Stdout, r)
http.ListenAndServe(":1123", loggedRouter)
```

我们将复制原始示例并添加此中间件。

```
~/ch16$ cat logging.go
package main
import (
"github.com/gorilla/mux"
"github.com/gorilla/handlers"
"os"
"net/http"
)
func buildHandler(message string) func(http.ResponseWriter, *http.Request) {
return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
w.Write([]byte(message))
})
}
func main() {
r := mux.NewRouter()
r.HandleFunc("/", buildHandler("HomeHandler"))
r.HandleFunc("/products", buildHandler("ProductsHandler"))
r.HandleFunc("/articles", buildHandler("ArticlesHandler")).Host("example.com").Methods("GET").Schemes("http")
loggedRouter := handlers.LoggingHandler(os.Stdout, r)
http.ListenAndServe(":8080", loggedRouter)
}
```

注意上述代码中的链式调用。我们没有将路由器对象直接传递给 `ListenAndServe`，而是将其传递给 `LoggingHandler`，后者将其链接起来。然后我们将其结果（一个满足 `ServeMux` 接口的对象）传递给 `ListenAndServe`。

如果你运行服务器并从另一个终端访问，你将看到记录的请求和响应信息。

```
~/ch16$ go run logging.go
~/ch16$ curl --resolve example.com:8080:127.0.0.1 -IX GET http://example.com:8080/articles
HTTP/1.1 200 OK
Date: Sun, 03 Apr 2022 04:38:07 GMT
Content-Length: 15
Content-Type: text/plain; charset=utf-8
```

在服务器端，我们看到：

```
127.0.0.1 - - [07/Mar/2022:19:04:20 +0000] "GET /articles HTTP/1.1" 200 15
```

此格式定义在：[`http://httpd.apache.org/docs/2.2/logs.html#common`](http://httpd.apache.org/docs/2.2/logs.html#common)。你会注意到我们还有另一个名为 `CombinedLoggingHandler` 的处理器；其格式定义在：[`http://httpd.apache.org/docs/2.2/logs.html#combined`](http://httpd.apache.org/docs/2.2/logs.html#combined)。以下是两种格式的简要对比。

* 通用格式 `– "%h %l %u %t \"%r\" %>s %b"`
* 组合格式 `– "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-agent}i\""`

现在我们尝试一个 `ContentTypeHandler` 处理器的例子。

```
~/ch16$ go doc handlers.ContentTypeHandler
package handlers // import "github.com/gorilla/handlers"
func ContentTypeHandler(h http.Handler, contentTypes ...string) http.Handler
ContentTypeHandler wraps and returns a http.Handler, validating the request
content type is compatible with the contentTypes list. It writes a HTTP 415
error if that fails.
Only PUT, POST, and PATCH requests are considered.
```

请注意，它仅适用于部分 HTTP 方法（那些用于创建/更新的方法）；其他方法会被静默传递。

```
~/ch16$ cat contenttype.go
package main
import (
"github.com/gorilla/handlers"
"github.com/gorilla/mux"
"net/http"
)
func buildHandler(message string) func(http.ResponseWriter, *http.Request) {
return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
w.Write([]byte(message))
})
}
func main() {
r := mux.NewRouter()
r.HandleFunc("/", buildHandler("HomeHandler"))
r.HandleFunc("/products", buildHandler("ProductsHandler"))
l := handlers.ContentTypeHandler(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
w.Write([]byte("json articles only"))
}), "application/json")
r.Handle("/articles", l)
http.ListenAndServe(":8080", r)
}
```

可以看到，对于 `"/articles"` 路由，我们希望确保任何上传的数据都具有 json 内容类型。

```
~/ch16$ go run contenttype.go
```

接下来，我们演示预期的拦截（在 POST 请求中验证 `"application/json"`）以及忽略该验证的情况。

```
~/ch16$ curl -IX POST -H 'Content-Type: application/json' 127.0.0.1:8080/articles
HTTP/1.1 200 OK
Date: Mon, 07 Mar 2022 19:28:04 GMT
Content-Length: 18
Content-Type: text/plain; charset=utf-8
~/ch16$ curl -IX POST -H 'Content-Type: application/xml' 127.0.0.1:8080/articles
HTTP/1.1 415 Unsupported Media Type
Content-Type: text/plain; charset=utf-8
X-Content-Type-Options: nosniff
Date: Mon, 07 Mar 2022 19:28:11 GMT
Content-Length: 81
~/ch16$ curl -IX GET -H 'Content-Type: application/xml' 127.0.0.1:8080/articles
HTTP/1.1 200 OK
Date: Mon, 07 Mar 2022 19:28:20 GMT
Content-Length: 18
Content-Type: text/plain; charset=utf-8
```

这些处理器可以按路由使用，也可以用于所有路由。还有其他可用的处理器；请查阅文档了解详情：`go doc gorilla/handlers | grep ^func`。

## 更多 Gorilla 示例

剩余部分是一些 Gorilla 包的示例，演示了它们的主要机制。为了达到我们的目的，我们着重关注它们与标准库功能在比较上的异同。



### `gorilla/rpc`

Gorilla 的 `json` 包提供了与第 13 章中利用 JSON 的 `rpc` 类似的功能。我们注册服务，该服务进而接收 HTTP 请求。

```
ch16$ vi rpc.go
package main
import (
"github.com/gorilla/rpc"
"github.com/gorilla/rpc/json"
"net/http"
)
type HelloArgs struct {
Who string
}
type HelloReply struct {
Message string
}
type HelloService struct{}
func (h *HelloService) Say(r *http.Request, args *HelloArgs, reply *HelloReply) error {
reply.Message = "Hello, " + args.Who + "!"
return nil
}
func main() {
s := rpc.NewServer()
s.RegisterCodec(json.NewCodec(), "application/json")
s.RegisterService(new(HelloService), "")
http.Handle("/rpc", s)
http.ListenAndServe(":8080", nil)
}
```

启动服务器。

```
~/ch16$ go mod tidy
~/ch16$ go run rpc.go
```

从客户端，我们将基于 JSON 的负载发送到正确的端点。

```
~/ch16$ curl -X POST -H "Content-Type: application/json" \
-d '{"method":"HelloService.Say","params":[{"Who":"Test"}], "id":"1"}' \
http://localhost:8080/rpc
{"result":{"Message":"Hello, Test!"},"error":null,"id":"1"}
```

`gorilla/rpc` 包源自 `net/rpc` 包，但有如下显着差异。

```
~/ch16$ go doc gorilla/rpc
package rpc // import "github.com/gorilla/rpc"
Package gorilla/rpc is a foundation for RPC over HTTP services, providing
access to the exported methods of an object through HTTP requests.
This package derives from the standard net/rpc package but uses a single
HTTP request per call instead of persistent connections. Other differences
compared to net/rpc:
- Multiple codecs can be registered in the same server.
- A codec is chosen based on the Content-Type header from the request.
- Service methods also receive http.Request as parameter.
- This package can be used on Google App Engine.
...
```

虽然文档示例中没有提到，但第三方已经创建了用于 XML 等的“其他”编解码器，它们利用了 `gorilla/rpc` 包接口。

### `gorilla/schema`

Gorilla 的 schema 包使得将表单中的字段解组到我们的结构体中变得容易。在这里，我们创建了一个包含两个字段的 `Person` 结构体。最终，我们“提交”一个包含这些字段的表单并填充它们。

```
~/ch16$ cat schema.go
package main
import (
"fmt"
"github.com/gorilla/schema"
"net/http"
)
var decoder = schema.NewDecoder()
type Person struct {
Name  string
Phone string
}
func main() {
http.HandleFunc("/schema", func(res http.ResponseWriter, req *http.Request) {
req.ParseMultipartForm(0)
var person Person
decoder.Decode(&person, req.PostForm)
message := fmt.Sprintf("Hello %v from area %v", person.Name, person.Phone)
res.Write([]byte(message))
})
http.ListenAndServe(":8080", nil)
}
```

启动服务器。

```
~/ch16$ go mod tidy
~/ch16$ go run schema.go
```

注意 curl 如何为我们填入 Content-Type，即 `multipart/form-data`（浏览器中的 HTML 表单也会填入 Content-Type）。

```
~/ch16$ curl -sF "Name=Ron" -F "Phone=312" -v localhost:8080/schema && echo
*   Trying 127.0.0.1:8080...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 8080 (#0)
> POST /schema HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.68.0
> Accept: */*
> Content-Length: 239
> Content-Type: multipart/form-data; boundary=------------------------0458a5d725c3c300
>
* We are completely uploaded and fine
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Date: Mon, 07 Mar 2022 21:57:45 GMT
< Content-Length: 23
< Content-Type: text/plain; charset=utf-8
<
* Connection #0 to host localhost left intact
Hello Ron from area 312
```

### `gorilla/securecookie`

securecookie 的文档开头如下：

```
~/ch16$ go doc securecookie
package securecookie // import "github.com/gorilla/securecookie"
Package securecookie encodes and decodes authenticated and optionally
encrypted cookie values.
Secure cookies can't be forged, because their values are validated using
HMAC. When encrypted, the content is also inaccessible to malicious eyes.
...
```

虽然关于 cookie 使用的争论仍在继续，我们只是展示了创建和往返到服务器的过程。我们不关心本地存储（客户端）。

底层使用的是 AES 加密；支持标准长度的块密钥（16、24、32 字节）。在我们的示例中，我们将其设置为 `nil`；因此，我们没有使用加密。

```
~/ch16$ cat securecookie.go
package main
import (
"fmt"
"github.com/gorilla/securecookie"
"net/http"
)
var hashKey = []byte("very-secret")
var s = securecookie.New(hashKey, nil)
func SetCookieHandler(w http.ResponseWriter, r *http.Request) {
value := map[string]string{
"foo": "bar",
}
if encoded, err := s.Encode("cookie-name", value); err == nil {
cookie := &http.Cookie{
Name:  "cookie-name",
Value: encoded,
Path:  "/",
}
http.SetCookie(w, cookie)
fmt.Println(cookie)
}
}
func ReadCookieHandler(w http.ResponseWriter, r *http.Request) {
if cookie, err := r.Cookie("cookie-name"); err == nil {
value := make(map[string]string)
fmt.Println(cookie.Value)
if err = s.Decode("cookie-name", cookie.Value, &value); err == nil {
fmt.Fprintf(w, "The value of foo is %q", value["foo"])
}
fmt.Println(cookie, err)
}
}
func main() {
http.HandleFunc("/set", SetCookieHandler)
http.HandleFunc("/read", ReadCookieHandler)
http.ListenAndServe(":8080", nil)
}
```

启动服务器。

```
~/ch16$ go mod tidy
~/ch16$ go run securecookie.go
```

`/set` 端点将简单地创建我们的 cookie。

```
~/ch16$ curl -I localhost:8080/set
HTTP/1.1 200 OK
Set-Cookie: cookie-name=MTY0NjY5MDU3MXxEdi1CQkFFQ180SUFBUXdCREFBQURQLUNBQUVEWm05dkEySmhjZz09fMds29irowo8h_9MByVigjT1qh0t7hnvAHprwtX9nc1h; Path=/
Date: Mon, 07 Mar 2022 22:02:51 GMT
```

要确认已收到，我们发送回编码后的 cookie（请务必使用您之前获得的那个）。

```
~/ch16$ curl -b "cookie-name=MTY0NjY5MDU3MXxEdi1CQkFFQ180SUFBUXdCREFBQURQLUNBQUVEWm05dkEySmhjZz09fMds29irowo8h_9MByVigjT1qh0t7hnvAHprwtX9nc1h" localhost:8080/read && echo
The value of foo is "bar"
```

## 结论

“Gorilla”这个名字以 `Go` 开头，并引起人们对濒危灵长类动物的关注。随着时间的推移，Gorilla 不再是丛林里唯一的工具包，但它仍然在被讨论之中。我们在这里没有涵盖所有的包，但希望这能让您对标准库的扩展方式有所了解。

# 17. 测试

在本章中，我们将探讨测试网络相关代码的方法和示例。使用单元测试和集成测试技术，我们努力捕获已知和预期的行为。理想情况下，单元测试应避免所有外部因素。例如，利用网络的测试最准确地描述为集成测试。对于我们代码而言，网络（外部因素）可能会施加不当影响，例如延迟或限制负载大小。知道如何设计测试并不总是那么容易，更不用说如何分离和减少外部影响了。拥有许多未知或受控的内部机制可能会导致测试不稳定，即那些似乎会随机失败的测试。

本章不是要学习 Go 测试工具的基础知识。存在关于基础知识的优秀资源，例如：

- [`https://pkg.go.dev/testing`](https://pkg.go.dev/testing)
- [`https://quii.gitbook.io/learn-go-with-tests/`](https://quii.gitbook.io/learn-go-with-tests/)
- `go doc testing`

我们的目标是学习一些有助于我们设计和管理与网络相关测试的技巧。



## 简洁与缺陷

我们从一组简单的测试开始：两个测试都发送请求并确认收到响应。唯一的区别是一个测试的客户端超时而另一个没有。两个测试中，服务器都通过休眠五秒来模拟工作。我们将单独运行每个测试，然后作为测试套件运行，并审查结果。

```
$ mkdir ch17
$ cd ch17
ch17$ vi basic_http_test.go
package ch17

import (
	"net/http"
	"testing"
	"time"
)

func TestHTTPRoundTrip(t *testing.T) {
	path := "/"
	c := make(chan struct{})
	//服务器
	go func() {
		http.HandleFunc(path, func(w http.ResponseWriter, req *http.Request) {
			time.Sleep(5 * time.Second) // 保持连接
		})
		http.ListenAndServe(":8080", nil)
	}()
	//客户端
	go func() {
		resp, err := http.Get("http://localhost:8080" + path)
		if err != nil {
			t.Error(err)
		} else {
			if resp == nil || resp.StatusCode != http.StatusOK {
				t.Error(resp)
			}
		}
		defer func() {
			c <- struct{}{}
		}()
	}()
	<-c
}

func TestHTTPRoundTripTimeout(t *testing.T) {
	path := "/"
	c := make(chan struct{})
	//服务器
	go func() {
		http.HandleFunc(path, func(w http.ResponseWriter, req *http.Request) {
			time.Sleep(5 * time.Second) // 保持连接
		})
		http.ListenAndServe(":8080", nil)
	}()
	//客户端
	go func() {
		var client = &http.Client{
			Timeout: time.Second * 2,
		}
		resp, err := client.Get("http://localhost:8080" + path)
		if err != nil {
			t.Error(err)
		} else {
			if resp == nil || resp.StatusCode != http.StatusOK {
				t.Error(resp)
			}
		}
		defer func() {
			c <- struct{}{}
		}()
	}()
	<-c
}
```

我们首先运行第一个测试：

```
ch17$ go test -test.run "TestHTTPRoundTrip$" basic_http_test.go
=== RUN   TestHTTPRoundTrip
--- PASS: TestHTTPRoundTrip (5.00s)
PASS
ok      command-line-arguments  5.132s
```

然后运行第二个测试：

```
ch17$ go test -test.run "TestHTTPRoundTripTimeout$" basic_http_test.go
--- FAIL: TestHTTPRoundTripTimeout (2.00s)
basic_http_test.go:56: Get "http://localhost:8080/": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
FAIL
FAIL    command-line-arguments  2.131s
FAIL
```

阅读这些测试后，可能显而易见第二个测试会失败。客户端设置为两秒后超时，而服务器需要五秒处理。缺乏经验的人可能会忽略这些相关测试失败的原因。默认的 HTTP 客户端具有无限超时时间。在我们的例子中，第一个测试中的服务器即使耗时加倍，测试仍然会通过。这并非 Go 语言特有，而是此 Go HTTP 客户端实现的特定行为。如果我们直接同时运行两个测试会发生什么？

```
ch17$ go test basic_http_test.go
panic: http: multiple registrations for /
goroutine 31 [running]:
net/http.(*ServeMux).Handle(0x1529fa0, {0x137fe80, 0x1}, {0x1381c00?, 0x132fee0})
	/usr/local/go/src/net/http/server.go:2478 +0x226
net/http.(*ServeMux).HandleFunc(...)
	/usr/local/go/src/net/http/server.go:2515
net/http.HandleFunc(...)
	/usr/local/go/src/net/http/server.go:2527
command-line-arguments.TestHTTPRoundTripTimeout.func1()
	/Users/ronaldpetty/github.com/apress/network-prog-with-go-2e/ch17/basic_http_test.go:43 +0x37
created by command-line-arguments.TestHTTPRoundTripTimeout
	/Users/ronaldpetty/github.com/apress/network-prog-with-go-2e/ch17/basic_http_test.go:42 +0x5e
FAIL	command-line-arguments	5.225s
FAIL
```

这个错误暴露了一个测试反模式（test antipattern），即测试之间相互依赖。与之前的运行类似，这也暴露了一些实现细节。默认的 `ServeMux` 有一个默认处理器；根据 `server.go:2478` 的代码，我们不允许重新映射路径。

我们可以看到有几个需要考虑的事项，包括：

-   我们实际想要测试什么？
-   我们是否遵循了良好的测试实践（例如，测试独立性）？
-   我们的代码是否正确且性能良好？

如何解决上述问题？正如我们在前面章节中看到的，我们可以为每个测试创建一个新的 "mux"。另一种方法是在运行测试之前设置所有测试路由。但更根本的是，我们应该考虑我们到底在测试什么。例如，我们是在测试服务器是否监听？还是测试我们的 `Handler` 是否基于给定请求提供正确的响应？毕竟，如果我们考虑 OSI 模型的思维方式，HTTP 是我们的请求/响应协议所在层（L7），而 TCP/IP 维持我们的连接（L4 及以下）。由于这些层可以解耦，我们也应该能够解耦我们的测试。即使考虑到这种分离，也不清楚各个部分将在哪里被测试（即单元测试还是集成测试）。思考一下处理器映射：你是对它进行单元测试，还是值得做集成测试？毕竟，除非使用了给定路径，否则处理器甚至不会被触发。

如你所见，我们如何思考网络组件会导致可能不同的测试安排。我们还应该记住，测试的执行顺序是不确定的。虽然看起来测试是从上到下运行的，但这只是实现的一个副作用，并非 `testing` 包中指定的行为。

在我们进行重构时，首先会了解 Go 在测试辅助工具方面提供了什么。



## `httptest` 包

`httptest` 包提供了用于辅助 HTTP 测试的函数和类型。

```
ch17$ go doc httptest
package httptest // import "net/http/httptest"
Package httptest provides utilities for HTTP testing.
const DefaultRemoteAddr = "1.2.3.4"
func NewRequest(method, target string, body io.Reader) *http.Request
type ResponseRecorder struct{ ... }
func NewRecorder() *ResponseRecorder
type Server struct{ ... }
func NewServer(handler http.Handler) *Server
func NewTLSServer(handler http.Handler) *Server
func NewUnstartedServer(handler http.Handler) *Server
```

让我们创建一个新的测试，它类似于之前我们编写的测试，但实际上并不连接到服务器。为了节省篇幅，我们将不再列出已有的测试。

```
ch17$ vi basic_http_test.go
package ch17
import (
"net/http"
"net/http/httptest"
"testing"
"time"
)
func TestHTTPRoundTripNoConnection(t *testing.T) {
path := "/"
req := httptest.NewRequest("GET", path, nil)
res := httptest.NewRecorder()
f := func(w http.ResponseWriter, req *http.Request) {
time.Sleep(5 * time.Second) // holding connection
}
f(res, req)
if res == nil || res.Result().StatusCode != http.StatusOK {
t.Error(res)
}
}
func TestHTTPRoundTrip(t *testing.T) {
...
func TestHTTPRoundTripTimeout(t *testing.T) {
...
```

我们做了一些修改，但捕获了我们的请求、响应以及相关的断言。需要注意几点：我们移除了服务器（即 `http.ListenAndServe`）；处理函数到路径的映射仍然存在，但它明确地体现在 `httptest.NewRequest` 中，而不是 `http.HandleFunc` 里。由于我们使用了一个新的包（即 `httptest`），我们必须修改结果检查，以便更紧密地匹配实际的响应。

我们调用 `httptest.NewRecorder` 会返回一个新的 `httptest.ResponseRecorder` 类型，该类型进而实现了 `http.ResponseWriter` 接口。这使得 `http.Response` 可以被间接填充。但结果并不适合进行深度比较（即 `reflect.DeepEqual`）。

除了处理函数，我们如何测试客户端和模拟的后端呢？`httptest` 包还提供了创建测试服务器端点的能力。

```
ch17$ vi basic_http_test.go
... other tests and imports
func TestHTTPTestRoundTripTimeout(t *testing.T) {
ts := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, req *http.Request) {
time.Sleep(5 * time.Second) // holding connection
}))
defer ts.Close()
var c = &http.Client{
Timeout: time.Second * 2,
}
req, _ := http.NewRequest("GET", ts.URL, nil)
res, err := c.Do(req)
if err != nil {
t.Fatal(err)
}
err = res.Body.Close()
if err != nil {
t.Fatal(err)
}
}
```

我们可以像下面这样运行这个测试：

```
ch17$ go test -test.run "TestHTTPTestRoundTripTimeout$" -v basic_http_test.go
=== RUN   TestHTTPTestRoundTripTimeout
basic_http_test.go:68: Get "http://127.0.0.1:50866": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
--- FAIL: TestHTTPTestRoundTripTimeout (5.00s)
FAIL
FAIL    command-line-arguments  5.155s
FAIL
```

在这些特定的例子中，你可能不希望实际的测试失败。你应该修复路径（使其唯一），并调整超时设置，使其仅在出现意外情况时才触发错误。

## HTTP 层以下

前面的工具集中在 HTTP 上。有没有更通用的技术呢？在 `net` 包中包含了 `Pipe` 类型。

```
ch17$ go doc net.Pipe
package net // import "net"
func Pipe() (Conn, Conn)
Pipe creates a synchronous, in-memory, full duplex network connection; both
ends implement the Conn interface. Reads on one end are matched with writes
on the other, copying data directly between the two; there is no internal
buffering.
```

如文档所述，我们获得了一些连接对象，它们的行为类似于更通用的客户端和服务器。

```
... prior tests and imports
func TestPipe(t *testing.T) {
c := make(chan struct{})
server, client := net.Pipe()
go func() {
time.Sleep(2 * time.Second)
req := make([]byte, 15)
server.SetDeadline(time.Now().Add(1 * time.Second))
_, err := server.Read(req)
t.Log(string(req))
if err != nil {
t.Error(err)
}
defer func() {
server.Close()
c <- struct{}{}
}()
}()
client.SetDeadline(time.Now().Add(1 * time.Second))
_, err := client.Write([]byte("my http request"))
if err != nil {
t.Error(err)
}
defer func() {
client.Close()
}()
<-c
}
```

在前面的例子中，我们试图模拟一些我们在与 HTTP 相关的超时中看到的失败行为。

```
ch17$ go test -test.run "TestPipe$" -v basic_http_test.go
=== RUN   TestPipe
basic_http_test.go:131: write pipe: i/o timeout
basic_http_test.go:119:
basic_http_test.go:121: read pipe: i/o timeout
--- FAIL: TestPipe (3.00s)
FAIL
FAIL    command-line-arguments  3.137s
FAIL
```

请注意，我们是在比 HTTP（即第 7 层）更低的层上进行操作。在大多数情况下，这意味着我们需要自己处理请求和响应，而不是利用像 `http.Request` 或 `http.Response` 这样的现有高级类型。



## 善用标准库

网络上充斥着各种示例，Go 标准库也不例外。在本节，我们将研究几个现有的测试案例，重点关注其风格和（必要的）复杂性。

```
ch17$ go test -test.count=1 -v -test.list ".*" $(go env GOROOT)/src/net/...  | head
TestSortByRFC6724
TestRFC6724PolicyTableClassify
TestRFC6724ClassifyScope
TestRFC6724CommonPrefixLength
TestCgoLookupIP
TestCgoLookupIPWithCancel
TestCgoLookupPort
TestCgoLookupPortWithCancel
TestCgoLookupPTR
TestCgoLookupPTRWithCancel
```

这里有数千个测试，大多数都没有文档说明。我们首先来看一下：

```
$(go env GOROOT)/src/net/http/requestwrite_test.go
```

让我们尝试运行相关的测试（以 `TestRequestWrite` 开头）。

```
ch17$ cd $(go env GOROOT)/src/net/http
http$ go test -test.count=1 -v -test.run "TestRequestWrite*"
=== RUN   TestRequestWrite
--- PASS: TestRequestWrite (0.00s)
=== RUN   TestRequestWriteTransport
=== PAUSE TestRequestWriteTransport
=== RUN   TestRequestWriteClosesBody
--- PASS: TestRequestWriteClosesBody (0.00s)
=== RUN   TestRequestWriteError
--- PASS: TestRequestWriteError (0.00s)
=== RUN   TestRequestWriteBufferedWriter
--- PASS: TestRequestWriteBufferedWriter (0.00s)
=== CONT  TestRequestWriteTransport
--- PASS: TestRequestWriteTransport (0.20s)
PASS
ok      net/http    0.363s
```

我们可以运行这些已有的测试，甚至修改它们来进行测试尝试，这很棒（但请小心，记得备份）。

如果你打开 `requestwrite_test.go`（例如，`sudo vi $(go env GOROOT)/src/net/http/requestwrite_test.go`），请注意测试表的结构；这里我们列出第一个条目。

```
// 来自 requestwrite_test.go
...
type reqWriteTest struct {
Req  Request
Body any // 可选的 []byte 或 func() io.ReadCloser，用于填充 Req.Body
// 这三个字段中的任何一个都可以为空，以跳过该测试。
WantWrite string // Request.Write
WantProxy string // Request.WriteProxy
WantError error // 期望从 Request.Write 返回的错误
}
var reqWriteTests = []reqWriteTest{
// HTTP/1.1 => 分块编码；无 body；无 trailer
0: {
Req: Request{
Method: "GET",
URL: &url.URL{
Scheme: "http",
Host:   "www.techcrunch.com",
Path:   "/",
},
Proto:      "HTTP/1.1",
ProtoMajor: 1,
ProtoMinor: 1,
Header: Header{
"Accept":           {"text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8"},
"Accept-Charset":   {"ISO-8859-1,utf-8;q=0.7,*;q=0.7"},
"Accept-Encoding":  {"gzip,deflate"},
"Accept-Language":  {"en-us,en;q=0.5"},
"Keep-Alive":       {"300"},
"Proxy-Connection": {"keep-alive"},
"User-Agent":       {"Fake"},
},
Body:  nil,
Close: false,
Host:  "www.techcrunch.com",
Form:  map[string][]string{},
},
WantWrite: "GET / HTTP/1.1\r\n" +
"Host: www.techcrunch.com\r\n" +
"User-Agent: Fake\r\n" +
"Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n" +
"Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7\r\n" +
"Accept-Encoding: gzip,deflate\r\n" +
"Accept-Language: en-us,en;q=0.5\r\n" +
"Keep-Alive: 300\r\n" +
"Proxy-Connection: keep-alive\r\n\r\n",
WantProxy: "GET http://www.techcrunch.com/ HTTP/1.1\r\n" +
"Host: www.techcrunch.com\r\n" +
"User-Agent: Fake\r\n" +
"Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n" +
"Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7\r\n" +
"Accept-Encoding: gzip,deflate\r\n" +
"Accept-Language: en-us,en;q=0.5\r\n" +
"Keep-Alive: 300\r\n" +
"Proxy-Connection: keep-alive\r\n\r\n",
},
// HTTP/1.1 => 分块编码；有 body；空 trailer
1: {
...
```

和 Go 中的许多测试一样，我们看到一个非常长的测试表。我们有一个 `Request` 实例（`Req`）以及一个非常相似的字符串化结果（`WantWrite` 和 `WantProxy`）。在深入探讨之前，我们先回顾其中一个测试。

```
// 来自 requestwrite_test.go
...
func TestRequestWrite(t *testing.T) {
for i := range reqWriteTests {
tt := &reqWriteTests[i]
setBody := func() {
if tt.Body == nil {
return
}
switch b := tt.Body.(type) {
case []byte:
tt.Req.Body = io.NopCloser(bytes.NewReader(b))
case func() io.ReadCloser:
tt.Req.Body = b()
}
}
setBody()
if tt.Req.Header == nil {
tt.Req.Header = make(Header)
}
var braw bytes.Buffer
err := tt.Req.Write(&braw)
if g, e := fmt.Sprintf("%v", err), fmt.Sprintf("%v", tt.WantError); g != e {
t.Errorf("writing #%d, err = %q, want %q", i, g, e)
continue
}
if err != nil {
continue
}
if tt.WantWrite != "" {
sraw := braw.String()
if sraw != tt.WantWrite {
t.Errorf("Test %d, expecting:\n%s\nGot:\n%s\n", i, tt.WantWrite, sraw)
continue
}
}
if tt.WantProxy != "" {
setBody()
var praw bytes.Buffer
err = tt.Req.WriteProxy(&praw)
if err != nil {
t.Errorf("WriteProxy #%d: %s", i, err)
continue
}
sraw := praw.String()
if sraw != tt.WantProxy {
t.Errorf("Test Proxy %d, expecting:\n%s\nGot:\n%s\n", i, tt.WantProxy, sraw)
continue
}
}
}
}
...
```

这段代码不算少。它在做什么？以下是其活动列表：

*   枚举测试表（`reqWriteTests`）。
*   设置一个包含 header 和 body 的真实 `Request` 对象。
*   序列化 `Request` 对象（`sraw`）并与 `WantWrite` 或 `WantProxy` 进行比较。

还有一些其他需要注意的点：

*   `io.NopCloser`

`NopCloser` 是一个简化功能的函数示例，具体来说，它阻止了资源的关闭。

```
http$ go doc -u -all io.nopCloser
package io // import "io"
func NopCloser(r Reader) ReadCloser
NopCloser 返回一个 ReadCloser，其 Close 方法是一个空操作，包装了提供的 Reader r。
type nopCloser struct {
Reader
}
func (nopCloser) Close() error
```

*   `tt.Req.Write` 和 `tt.Req.WriteProxy`

如果我们查看这些方法的相关文档：

```
http$ go doc net/http.Request.Write
package http // import "net/http"
func (r *Request) Write(w io.Writer) error
Write 以线路格式写入一个 HTTP/1.1 请求，即头部和主体。
此方法会参考请求的以下字段：
Host
URL
Method（默认为 "GET"）
Header
ContentLength
TransferEncoding
Body
如果 Body 存在，Content-Length <= 0 且 TransferEncoding 未被设置为 "identity"，则 Write 会向 header 中添加 "Transfer-Encoding: chunked"。
Body 在发送后被关闭。
http$ go doc net/http.Request.WriteProxy
package http // import "net/http"
func (r *Request) WriteProxy(w io.Writer) error
WriteProxy 与 Write 类似，但以 HTTP 代理期望的格式写入请求。
具体来说，根据 RFC 7230 的第 5.3 节，WriteProxy 会使用绝对 URI（包含 scheme 和 host）写入请求的初始 Request-URI 行。
在任何情况下，WriteProxy 都会写入一个 Host header，使用 r.Host 或 r.URL.Host。
```

我们看到这两个方法都接受一个 `Writer`，并且都根据各自的用途以略有不同的方式序列化 `Request` 对象。我们在这里不检查它们的差异，但这为我们如何进行自己的测试提供了思路。

接下来，我们回顾 `net` 包中的测试：`TestTCPConnSpecificMethods`。



```
// 来自 protoconn_test.go
...
func TestTCPConnSpecificMethods(t *testing.T) {
la, err := ResolveTCPAddr("tcp4", "127.0.0.1:0")
if err != nil {
t.Fatal(err)
}
ln, err := ListenTCP("tcp4", la)
if err != nil {
t.Fatal(err)
}
ch := make(chan error, 1)
handler := func(ls *localServer, ln Listener) { ls.transponder(ls.Listener, ch) }
ls := (&streamListener{Listener: ln}).newLocalServer()
defer ls.teardown()
if err := ls.buildup(handler); err != nil {
t.Fatal(err)
}
ra, err := ResolveTCPAddr("tcp4", ls.Listener.Addr().String())
if err != nil {
t.Fatal(err)
}
c, err := DialTCP("tcp4", nil, ra)
if err != nil {
t.Fatal(err)
}
defer c.Close()
c.SetKeepAlive(false)
c.SetKeepAlivePeriod(3 * time.Second)
c.SetLinger(0)
c.SetNoDelay(false)
c.LocalAddr()
c.RemoteAddr()
c.SetDeadline(time.Now().Add(someTimeout))
c.SetReadDeadline(time.Now().Add(someTimeout))
c.SetWriteDeadline(time.Now().Add(someTimeout))
if _, err := c.Write([]byte("TCPCONN TEST")); err != nil {
t.Fatal(err)
}
rb := make([]byte, 128)
if _, err := c.Read(rb); err != nil {
t.Fatal(err)
}
for err := range ch {
t.Error(err)
}
}
...
```

该测试本身专注于设置各种客户端连接相关选项，然后使用客户端，并期望一切正常。从我们的角度来看，我们感兴趣的是他们编写测试本身的方式。我们处在协议栈的 L3/L4 层。从高层次来看，其流程如下：

*   准备并验证一个已知的 IP 地址（例如 `127.0.0.1`）和端口（`:0`）
*   创建一个基于 TCP 的监听器。
*   启动服务器（后续详述）。
*   获取服务器的 IP 和分配的端口。
*   使用远程地址创建一个客户端。
*   配置客户端。
*   将数据从客户端写入服务器，然后读取结果。

运行 `TestTCPConnSpecificMethods` 测试，我们看到以下输出：

```
net$ go test -test.count=1 -v -test.run "TestTCPConnSpecificMethods$"
=== RUN   TestTCPConnSpecificMethods
--- PASS: TestTCPConnSpecificMethods (0.00s)
PASS
Socket statistical information:
(inet4, stream, default): opened=2 connected=1 listened=1 accepted=1 closed=3 openfailed=0 connectfailed=1 listenfailed=0 acceptfailed=1 closefailed=0
ok      net 0.210s
```

测试中间部分看起来比我们之前看到的要复杂。

```
ch := make(chan error, 1)
handler := func(ls *localServer, ln Listener) { ls.transponder(ls.Listener, ch) }
ls := (&streamListener{Listener: ln}).newLocalServer()
defer ls.teardown()
if err := ls.buildup(handler); err != nil {
t.Fatal(err)
}
```

上述代码定义在另一个文件中：`mockserver_test.go`。调用 `buildup` 会在一个 Go 协程中运行 handler 代码。从高层次来看，`Accept` 是在 `transponder` 中运行的，那么为什么不用一个“非模拟”的设置呢？进一步审查后，你会发现 `someTimeout` 被应用于大多数与超时相关的设置（例如 `SetDeadline`）。有趣的是，由于该代码存在于以 `_test.go` 结尾的文件中，我们无法通过 `go doc` 查看它。

更多详情请查阅 `$(go env GOROOT)/src/net/mockserver_test.go`。

## 结论

本章旨在引导你思考在测试网络相关代码时所涉及的技术和层次。要成为专业的网络测试开发者，还需要掌握更多概念和工具。例如，模糊测试（fuzzing）或泛型（generics）如何帮助确保代码安全并精简代码？当集成测试变得更加复杂时，例如我们将多个网络服务串联起来，甚至中途协议发生变化时，我们应该考虑什么？



好的，这是根据您的要求排版后的 Markdown 文档。

# 索引

## A
- 抽象语法记法一 (ASN.1)
- ASCII/UTF8 字符 `badtype` 目录字符串
- 日间客户端/服务端编码规则
- 错误文档
- 错误信息
- 接口源代码
- `json` 包
- 编组/解编组函数
- 结果/关联错误
- 源代码
- 结构化类型
- 类型
- Unicode 序列
- UTC 时间类型

美国信息交换标准代码 (ASCII)

应用层协议
- 字节格式服务器
- 字符格式服务器
- 客户端-服务器场景
- 命令行客户端
- 数据格式
- 目录浏览器
- 非正式协议
- 消息格式
- 可能性与问题
- 表示层方面
- 进程
- 协议设计
- 真实世界协议
- 服务端
- 源代码
- 独立应用程序
- 状态信息
    - 客户端状态图
    - DCE 文件系统
    - 分布式系统
    - 文件处理代码
    - NFS 文件系统
    - 伪代码
    - 服务器图
    - 转换图
- 语句
- 字符串处理
- 文本格式
- `textproto` 包
- 版本控制

架构层
- 接受因素
- 管理员
- 应用逻辑
- 异步通信
- 客户端-服务器系统
- 客户端-服务器 *与* 点对点系统
- 通信模型
- 无连接系统
- 连接模型
- 连续体
- 处理
- 数据访问组件
- 分布式系统
- 谬误
- 胖 *与* 瘦
- 加特纳分类
- 网关
- 同构
- 主机级网络
- 延迟
- 中间件模型
- 网络互联
- 数据包封装
- 故障点
- 表示层组件
- 协议层
- 发布/订阅系统
- 可靠网络
- 远程过程调用
- 安全服务器
- 系统
- 流式通信
- 同步通信
- 三层可能性
- 拓扑
- 透明性 请参见 透明性
- 传输成本

异步 JavaScript 和 XML (AJAX)

## B
- 体域网 (BANs)

## C
- 字符集与编码
    - ASCII 码
    - ASCII 字符集
    - 字符代码集
    - 定义
    - 编码
    - 注意事项，Unicode
    - 表意文字语言
    - ISO 8859/Go 系列
    - 非编码字符集
    - 字汇/字符集
    - 传输编码
    - Unicode 字符
    - UTF-8/Go/符文
    - ASCII 字符
    - 客户端和服务器
    - 描述
    - 字符串
    - UTF-16/Go 字节流/提取/检查
    - 客户端/服务器片段代码
    - 小端序/大端序
- 汉语词典
    - 词典类型
    - 拼音
    - 繁体字形式
- 客户端-服务器系统
    - 应用程序
    - 架构层
    - 点对点系统
    - 用户视角
- 面向连接的传输
- 内容协商
    - `GET /` 请求
    - MIME 类型
    - `POST /` 请求
    - 服务器代码段
    - 源代码
    - 网络服务器

## D
- 数据序列化
    - ASN.1 请参见 抽象语法记法一 (ASN.1)
    - 编码包
    - Gob 包
    - JSON 序列化
    - 共同约定
    - 非线性结构
    - 协议缓冲区
        - 代码生成
        - 数据类型
        - `personv3.pb.go` 文件
        - `protoc`
        - 序列化方法
        - 自描述数据
        - 结构化数据
- 分布式系统

## E, F
- 事件驱动系统
    - 行为
- 可扩展标记语言 (XML)
    - `command`/`CharData` 类型定义
    - `Directive`
    - `EndElement` 类型
    - HTML 文档
    - 编组数据结构
    - 解析数据
    - `ProcInst` 类型
    - `StartElement` 类型
    - 结构
    - 解编组结构
    - XHTML
- 外部数据表示 (XDR)

## G
- 加特纳分类
    - 分布式数据库模型
    - 网络文件服务
    - 安全 Shell
    - 表示层
    - 终端仿真
    - 网络事务
- Gob 序列化
    - 二进制数据
    - 客户端/服务器
    - 命令行工具
    - 编组/解编组结构
    - `person.gob` 文件
- Go 编程语言
    - 描述
    - 错误值
    - 函数
    - `GOPATH`
    - 高阶函数
    - 映射
    - 方法
    - 模块
    - 多线程
    - 包
    - 指针
    - 远程过程调用
    - 运行程序
    - 切片/数组
    - 标准库
    - 语句
    - 结构体
    - 类型转换
    - 类型
    - 网站
- Gorilla 工具包
    - 标准
    - `gorilla/mux` 处理器
    - 匹配元素
    - 中间件
    - 模式
    - 多路复用器定制
    - 包信息
    - `rpc` 包
    - `schema` 包
    - `securecookie` 文档
    - `ServeMux`

## H
- 超媒体作为应用状态引擎 (HATEOAS)
- 超文本标记语言 (HTML)
    - 描述
    - `html/template` 包
    - JSON 源代码
    - 标记化
    - XHTML/HTML
    - WebSocket 服务器
- 超文本传输协议 (HTTP)
    - 优势
    - 客户端对象
    - 配置
    - 请求
    - 加密连接
    - HTTPS 服务器代理
    - 处理认证
    - 传输对象
    - 服务器 `curl`
    - 客户端文件服务器
    - 处理器函数
    - 多路复用器角色
    - 测试包
    - URL 请参见 统一资源定位符 (URLs)
    - 用户代理
    - `GET` 请求
    - `HEAD` 方法
    - 请求/获取
    - 响应
    - 响应类型

## I
- 接口定义语言 (IDL)
- 国际化 (i18n)
- IPv4/IPv6 地址

## J, K
- JavaScript 对象表示法 (JSON)
    - 客户端/服务器
    - 雇员对象
    - 超文本标记语言 (HTML)
    - `loadjson.go`
    - 对象/数组/基本值
    - `person.json` 文件
    - 远程过程调用客户端
    - 服务器序列化结果
    - 源代码
    - 类型相关编码
    - WebSocket 服务器

## L
- 局域网 (LAN)

## M, N
- 消息传递
- 城域网 (MANs)
- 中间件模型
    - 组件
    - 函数
    - 库

## O
- 开放网络计算 (ONC)
- 开放系统互连 (OSI)

## P, Q
- 点对点 *与* 客户端-服务器系统
- 个域网 (PANs)
- 协议层
    - 定义
    - ISO OSI 协议
    - 网络通信
    - OSI 层
    - TCP/IP 协议

- 公钥基础设施 (PKI)

## R
- 远程过程调用 (RPC)
    - 客户端代码
    - Go 语言客户端/TCP
    - HTTP 客户端
    - 匹配值
    - 商/余数
    - 限制
    - 服务器 HTTP
    - TCP 套接字
    - JSON 编码
    - 消息传递范式
    - 步骤
- 表现层状态转移 (REST)
    - 客户端
    - 完整服务器组件
    - 内容协商
    - `createFlashcardSet()` 函数
    - 抽认卡
    - 处理请求
    - HATEOAS 链接规范
    - 远程过程调用
    - 表示
    - RESTful 事务模型
    - Richardson 成熟度模型
    - `ServeMux`
    - 无状态事务
    - URI/资源
    - 动词
        - `DELETE`
        - `GET 请求`
        - `HTTP 请求`
        - `PATH 组件`
        - `POST`
        - `PUT 请求`
- Richardson 成熟度模型

## S
- 安全系统
    - 加密工具
    - 数据完整性
    - 困难/微妙的计算问题
    - 功能/级别
    - 哈希算法
    - 十六进制数字
    - 机制
    - 面向网络的应用
    - OSI 七层模型
    - 公钥加密
    - 对称密钥加密
    - 传输层安全
    - X.509 证书
- 服务器分布
- 套接字级编程
    - 寻址
    - 互联网连接
    - `Conn`/`PacketConn`/`listener` 类型
    - 文档
    - Go 测试命令
    - 主机名/地址查找
    - `IPAddr` 类型
    - 掩码操作 (`IPMask` 类型)
    - 方法
    - `net` 包
    - 路由特性
    - IP 地址类型
    - 原始套接字/`IPConn` 类型
    - 服务器端口
    - 服务端口
    - 请求/响应
    - `TCPAddr` 类型
    - 套接字
    - TCP/IP 协议
    - UDP 数据报

## T
- 模板模块
    - 条件语句
    - 函数定义
    - `html/template` 包
    - 插入对象值
    - 管道
    - `printperson.go` 两步过程
    - 变量
- 测试
    - Go 标准库活动
    - 客户端连接方法
    - `requestwrite_test.go`
    - 测试包
    - `TestTCPConnSpecificMethods`
    - `httptest` 包
    - `Pipe` 类型
    - 简单设置/破坏性
    - 工具包
- 传输控制协议/互联网协议 (TCP/IP)
    - 客户端
    - 面向连接的协议
    - 日间服务
    - 截止时间和保活
    - 错误检查
    - IP 数据报
    - 多线程服务器
    - OSI 协议栈
    - 套接字
    - 超时
    - 用户数据报协议
- 透明性
    - 访问
    - 并发/可扩展性
    - 位置
    - 迁移/复制
    - 性能/故障
- 传输层安全 (TLS)
    - 证书颁发机构 (CA)
    - 客户端
    - 加密/解密方案
    - 自签名证书
- 简单文件传输协议 (TFTP)

## U
- 统一资源定位符 (URLs)
    - 适当表示
    - 特征
    - HTTP/0.9
    - HTTP/1.0
        - 请求格式
        - 响应格式
    - HTTP 1.1
    - HTTP/2
    - HTTP/3
    - 国际化 (i18n)
    - punycode 值
    - 资源
    - 响应格式
    - REST 传输机制
    - 版本
- 用户数据报协议 (UDP)
    - 无连接协议
    - 数据报
    - 服务
    - TCP/IP

## V
- 版本控制
    - 兼容性
    - `GET` 请求
    - HTML5 协议设置
    - 网络

## W, X, Y, Z
- 网络服务器
    - 浏览器站点图
    - 文件
    - 抽认卡组件
    - 主页
    - 页面
    - 单词列表
        - 汉语 请参见 汉语词典
        - 修正重音符号
        - 抽认卡集合
        - `listFlashCards` 函数
        - `ListWords` 函数
        - `manageFlashCards` 函数
        - 表示层
        - 服务器
        - `showFlashCards` 函数
- WebSocket 服务器
    - 浏览器/网络服务器
    - 客户端包
    - 回声服务器
    - `github.com/gorilla/websocket` 包
    - `golang.org/x/net/websocket`
        - `Codec` 类型
        - HTML 页面
        - JSON 对象
        - 消息对象
        - 包文档
        - TLS 套接字
        - `XMLCodec` 包
    - HTTP 服务器
    - 发送文本和二进制消息
- 广域网 (WAN)
