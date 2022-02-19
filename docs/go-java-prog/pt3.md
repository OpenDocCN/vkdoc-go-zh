# 第三部分 Go 图书馆调查

Go Library Survey

这一部分将简要介绍标准的 Go 库。它旨在让您了解和熟悉这些库以及它们能做什么，而不是详细介绍它们的用途。并非每个库中的所有类型和函数都会被提及，只有一些更普遍有用的(有些固执己见的选择)会被提及。许多函数都是不言自明的(它们的名字本质上暗示了它们的功能)，因此可能不再进一步描述。对于其他功能，只做简要说明。一些类型和/或功能将包括示例。

我们将从 Go 标准包库的一个简短的调查开始。它旨在让读者了解并熟悉 Go 运行时函数。它并不打算成为每个库函数的完整教程或参考，但对于许多库来说，它提供了足够的细节来开始成功地使用它们。有关每个库的更多信息，请参见在线 Go 文档(位于 [`www.golang.org/pkg`](http://www.golang.org/pkg) )。我们将只讨论标准的 Go 库。

像 Java 一样，Go 既有标准(包含在 Go install 中)库，也有第三方(也就是社区提供的)库。在作者看来，Java 标准库通常在本质上更全面，但是 Go 库足够丰富，可以编写许多有用的程序。此外，Go 版本通常更容易学习和使用，至少在最初是这样。

讨论所有社区提供的 Go 库，比如 Java 社区库，将是一项超人的任务，会产生成千上万的页面。在一本书里也不可能与这些库保持同步。因此，本文不打算这样做。

在某些情况下，Go 标准库具有 Java 标准库所没有的功能。例如，Go 内置了创建生产服务器的支持，尤其是 HTTP 服务器。在 Java 中，这通常需要额外的库，比如来自*Apache*T2 的库。org 或者 Spring *。org* 。所以标准围棋类似于一个简化功能 *Java 扩展版*(JEE——现在被称为 *Jakarta 扩展版* <sup>[1](#Fn1)</sup> )的环境。

与 Java 相比，Go 的一个非常薄弱的地方是本机 GUI。Java 有 *AWT* 、 *Swing* 和 *JavaFX* GUI 库，等等。Go 没有类似的东西，但是 Go 有一些库可以帮助 Java 在 JEE 中包含的 web GUIs 生成 HTML(或者 CSS 或者 JSON)。一些 Go 社区成员提供了本地 GUI 支持，但似乎没有一个是完全跨操作系统平台的。此外，许多不是纯粹的 Go，需要 CGo 访问本机操作系统库。

在逐个函数的基础上比较这些库可能需要数百页(如果不是数千页的话)。类似地，仅仅为程序员提供 Go 库中所有函数的参考就需要数百页。本书不会试图这样做。在线 Go 文档可用于此目的。

注意，Go 库被安排在包中，所以在这个上下文中，包和库通常是同义词(但是有些库可能有多个包)。

图书馆调查将由 Go 中的可用内容驱动。因此，有些 Java 库可能根本不包含在内。一些 Go 库不会被调查，因为它们不经常使用，或者不打算用于高级或内部(比如 Go 构建系统或运行时)使用。

这本书将比较一些关键的库，并提供一些他们提供的大多数程序员会使用的函数的例子。这基本上是将`java.lang`和`java.util`(及其子包)包中的一些类和函数与它们的 Go 等价物进行比较。还会提到其他 Java 包中的一些类和函数。

一些 Go 库是非常低级的(例如，不安全的内存访问或直接操作系统访问)，通常由更高级别的库包装；将只讨论更高级别的库。一些 Go 库处理 Go 运行时和工具集的实现，这里就不讨论了。一些 Go 库处理低级调试和跟踪，它们将不被讨论。

一些 Go 库处理跨进程和网络的*远程过程调用* (RPC ),这里就不讨论了。RPC 现在不如 HTTP 访问使用得频繁，HTTP 访问将被包括在内。同样，Go 的邮件支持就不讨论了。

这里没有列出所描述的包中的所有类型和功能。请参阅完整的联机文档。这个站点描述了标准包和一些补充包(其中一些是实验性的),并提供了一些到第三方包的链接。场地( [`https://golang.org/pkg/`](https://golang.org/pkg/) )如图 [P3-1](#Fig1) (标准库隐藏)。

![../images/516433_1_En_3_PartFrontmatter/516433_1_En_3_Fig1_HTML.jpg](../images/516433_1_En_3_PartFrontmatter/516433_1_En_3_Fig1_HTML.jpg)

图 P3-1

Go 扩展包摘要

*   Pkg.go.dev 多为搜索引擎；你需要提供关键词。

*   [`https://github.com/golang/go/wiki/Projects`](https://github.com/golang/go/wiki/Projects) 是 Go 搜索引擎的目录，也是跨多个领域的项目列表。这是一个寻找社区图书馆贡献的好来源。

标准库包含许多包；一些总结如下。标准库的许多类型和功能在社区产品中都有增强版(通常是替换版)。一些社区产品完全偏离了标准的库 API 模式，并且有不同的方法来提供特性。在承诺使用标准库 API 之前，建议对社区备选库进行一次调查(比如 web 搜索)。通常，它们更丰富，功能更强。

这种现象在 Java 中不太普遍，大多数开发人员使用标准库来实现他们提供的任何功能。一个明显的例外是标准的 HTTP 客户端，它可能很难使用并且功能有限。因此，它存在许多社区增强。注 Java 11 提供了一个替代的 HTTP 客户端，它有了很大的改进，可能会淘汰一些社区产品。

下面没有列出所有的类型和类型的函数/方法，只列出了最常用的。许多包都有常量和变量的定义。仅列出这些常用值。如果没有为下面的类型定义构造函数(`NewXxx`)方法，请使用零值声明。

包变量通常包含标准化的`error`类型，这些类型可以用来与包中方法返回的错误进行比较。这些值的使用类似于 Java 的标准异常类型(例如，`IllegalArgumentException`或`ArrayIndexOutOfBoundsException`)。例如，`zip`包定义了这些错误:

```
var (
      ErrFormat    = errors.New("zip: not a valid zip file")
      ErrAlgorithm = errors.New("zip: unsupported compression algorithm")
      ErrChecksum  = errors.New("zip: checksum error")
)

```

请注意，这为代码中任何类似的错误设置了一种命名模式，并为错误消息文本设置了一种可能的样式。

下面列出的许多类型，尤其是结构类型，都实现了`Stringer`接口。这在以下方法集中没有记录。

Go 标准包被安排在一个浅层次结构中。一些通用功能被分组在父包下，而更多的特定功能被分组在子包下。

虽然标准层次结构中的软件包会随着时间的推移而增加，但下面列出了一些具有代表性的软件包，并简要描述了它们的用途:

*   归档–空，参见嵌套

*   归档/TAR–TAR 读取权限

*   归档/ZIP–ZIP 读/写访问

*   bufio–通过低级无缓冲 I/O 功能提供缓冲 I/O

*   内置——描述 Go 内置类型和标识符

*   字节–处理字节片的函数

*   压缩–空，参见嵌套

*   压缩/BZIP–BZIP 2 解压缩

*   压缩/压缩–处理压缩格式数据

*   压缩/gzip–处理 gzip 格式数据

*   压缩/lzw–处理莱姆佩尔-齐夫-韦尔奇格式数据

*   compress/zlob–处理 zlib 格式的数据

*   容器-空，参见嵌套

*   容器/堆–处理堆(树，其中每个节点是任何子树的最小值，最小值在顶部)

*   容器/列表–双向链表

*   容器/环–循环列表

*   上下文–报告超时和取消操作的方法

*   crypto–保存加密常数

*   加密/aes–提供 AES 加密

*   加密/密码–支持分组密码

*   加密/des–提供 DES 和 3DES 加密

*   cypto/dsa–提供 DSA 支持

*   crypto/ECD sa–提供椭圆曲线 dsa

*   crypto/ed25519–提供 ed 25519 签名

*   加密/椭圆–提供素数域上的椭圆曲线

*   crypto/hmac–提供 hmac 认证

*   crypto/md5–提供 MD5 哈希

*   crypto/rand–提供加密字符串随机数

*   crypto/rc4–提供 RC4 加密

*   加密/rsa–提供 RSA 加密

*   crypto/sha1–提供 SHA-1 哈希

*   crypto/SHA 256–提供 SHA-224 和 SHA-256 哈希

*   crypto/sha 512–提供 SHA-384 和几个 SHA-512 哈希

*   加密/微妙-提供加密助手功能

*   crypto/tls–提供 TLS(在 HTTPS 使用)支持

*   crypto/x509–提供 X.509 证书支持

*   crypto/pkix–为 x509 提供数据

*   数据库-空，参见嵌套

*   数据库/SQL–支持对数据库的 JDBC 式访问

*   数据库/驱动程序–支持数据库驱动程序(SPI)

*   调试–空，参见嵌套

*   调试/dwarf-支持 DWARF 信息

*   调试/elf–支持 ELF 目标文件

*   调试/gosym–在运行时访问符号/行号

*   调试/macho-支持 Mach-O 目标文件

*   调试/PE–支持可移植的可执行文件

*   debug/Plan9 obj–支持 plan 9 目标文件

*   编码–支持各种格式转换的接口

*   编码/ascii85–支持 ascii 85 编码

*   编码/ASN 1–支持 ASN.1 编码

*   编码/base32–支持 base 32 编码

*   编码/base64–支持 base64 编码

*   编码/二进制–支持序列化基本类型

*   编码/csv–支持读取/写入 CSV 格式数据

*   encoding/gob–支持序列化复杂(结构)类型

*   编码/十六进制–二进制到十六进制字符串转换

*   encoding/json–二进制到 JSON 字符串转换

*   编码/pem–提供 PEM 编码

*   编码/XML–XML 解析器

*   错误–支持 Go 错误类型

*   exp var——支持导出 Go 运行时状态(如 JMX)

*   flag–支持解析命令行标志(即开关)

*   fmt–支持扫描/格式化值

*   go–空，请参见嵌套

*   go/ast–支持访问基于 Go 的 ast

*   go build–支持处理 Go 包

*   go/constant–支持 Go 常量

*   go/doc–支持在 AST 中处理 Go 文档注释

*   go/format–支持格式化 Go 源代码

*   go/importer–支持处理`import`语句

*   go/parser–解析 Go 源代码

*   go/printer–支持格式化 AST

*   go/scanner–对 Go 源进行词法分析(标记化)

*   go/tokens–各种 Go 源令牌的枚举

*   go/types–支持 Go 源代码中的类型和类型检查

*   哈希–定义各种哈希

*   hash/adler32–支持 Adler 32 校验和

*   hash/crc32–支持 32 位 crc 校验和

*   hash/crc 64–支持 64 位 CRC 校验和

*   哈希/fnv–支持 fnv 哈希

*   hash/map hash–字节序列的散列

*   html–支持转义 HTML 值

*   HTML/模板–HTML 注入安全模板

*   图像-帮助制作 2D 图像的功能

*   图像/颜色–提供颜色库

*   图像/调色板-介质颜色调色板

*   图像/绘图–支持 2D 绘图

*   image/gif-gif 支援格式

*   图像/jpeg–支持 JPEG 格式

*   影像/png 支援 png 格式

*   索引–空，参见嵌套

*   index/suffix array–使用后缀数组提供子字符串搜索

*   io–支持低级 I/O 操作

*   iou til–提供 I/O 助手

*   日志–提供基本的日志记录

*   日志/系统日志–提供对操作系统日志的访问

*   数学–提供基本的数学函数

*   math/big——支持大整数、大浮点数和有理数

*   math/bits–支持整数的位级访问

*   cmplx–为复数提供帮助器/函数

*   math/rand–支持生成随机数

*   mime–提供对 MIME 类型的支持

*   mime/multipart–支持多部分数据

*   mime/quoted printable–支持引用的可打印编码

*   net——支持 TCP/IP 和 UDP 网络、DNS 解析和套接字

*   net/http–支持 HTTP 客户端和服务器

*   net/cgi–支持 CGI 服务器

*   net/cookiejar–支持 HTTP cookies

*   net/fgci——支持“快速”CGI 服务器

*   支持 HTTP 交互的模拟测试

*   net/HTTP trace–支持 HTTP 请求跟踪

*   net/HTTP util–提供 HTTP 助手函数

*   net/pprof–支持 HTTP 服务器分析

*   net/mail–支持电子邮件处理

*   net/rpc–支持基本的 RPC 消息传递和序列化

*   net/rpc/jasonrpc——支持使用 JSON 主体的 RPC

*   net/SMTP–支持电子邮件发送/接收

*   net/text proto–支持基于文本头的(例如 HTTP)网络协议

*   支持解析和处理 url

*   os–提供对操作系统(OS)提供的功能的访问

*   OS/exec–支持运行外部进程

*   操作系统/信号–支持处理操作系统信号

*   操作系统/用户–支持操作系统用户/组和凭据

*   os/path–支持处理 OS 文件系统路径

*   操作系统/文件路径–路径帮助函数

*   插件–为动态加载的插件提供(有限的)支持

*   reflect——支持在运行时自省和创建类型和实例

*   正则表达式——支持计算正则表达式

*   regex/syntax–支持解析正则表达式

*   运行时–支持管理正在运行的 Go 应用程序

*   运行时/CGO——支持访问用 C 编写的函数

*   运行时/调试–为运行时诊断提供支持

*   运行时/pprof–生成分析数据

*   运行时/跟踪–支持运行时跟踪；比日志记录更实用

*   排序–支持排序或切片和集合

*   srtconv–转换成(格式化程序)/转换成(解析程序)字符串的各种转换器

*   字符串——`string`类型的各种助手

*   sync–提供同步原语

*   同步/原子-提供进行原子更新的功能

*   syscall 提供各种低级操作系统(OS)功能；可能因操作系统而异，并非在所有操作系统类型上都可用

*   测试–提供类似 JUnit 的测试和代码计时

*   测试/io test–测试 I/O 操作的助手

*   测试/快速——提供测试用例中使用的助手

*   文本-空，参见嵌套

*   文本/扫描仪–提供字符串扫描/标记

*   文本/tab writer–提供列对齐的文本输出

*   文本/模板–支持在模板中可编程地插入文本

*   文本/解析–支持解析模板

*   时间-提供对日期、时间、时间戳、持续时间、瞬间的支持

*   time/tz data–无需操作系统帮助即可支持时区

*   unicode–空，参见嵌套

*   unicode/ut F16–支持 16 位 Unicode 字符

*   unicode/utf 32–提供对 32 位 Unicode 字符(又名符文)的支持

*   Unicode/utf8–支持 UTF 8 Unicode 字符

*   不安全–为体系结构敏感的数据和指针提供支持

虽然这个列表很长，但是比 Java 标准版中所有包的类似列表要短得多。尽管如此，所提供的功能通常足以创建丰富的 web 客户端和服务器，它们是现代微服务的基础，是一个关键的 Go 用例。

举个例子(基于一个围棋网站的例子)，`utf8`包允许人们从(UTF-8)字符串中提取符文:

```
var text = "The 世界 is a crazy place!"  // world defined in UTF-8
var runes = make([]rune, 0, len(text))
for len(text) > 0 {
      rune, runeLen := utf8.DecodeRuneInString(text)
      fmt.Printf("%c(%d, %d)\n", rune, rune, runeLen)
      runes = append(runes, rune)
      text = text[runeLen:]
}

```

它输出一个包含输入的所有符文的切片；它的长度可能(在本例中将)比输入长度短。

作为另一个例子，考虑这个简单的方法来测量一些代码的运行时间:

```
func TimeIt(timeThis func() error) (dur time.Duration, err error) {
      start := time.Now()
      err = timeThis()
      dur = time.Now().Sub(start)
      return
}

```

随着

```
elapsed, _ := TimeIt(func() (err error) {
      time.Sleep(1 * time.Second)
      return
})

```

`elapsed`的结果大约是`1e9`。

该示例可以用另一种方式重做:

```
func TimeIt(timeThis func() error) (dur time.Duration, err error) {
      start := time.Now()  // must declare before use
      defer func(){
            dur = time.Now().Sub(start)
      }()
      err = timeThis()
      return
}

```

接下来的章节讨论了几个 Go 软件包，每个都提供了一个更完整的软件包概要。在这些包的许多描述中，有一些用名称模式描述的功能:

`Xxxx`–使用默认参数函数提供一些行为

`XxxxFunc`–使用提供的参数函数提供相同的行为，以实现自定义行为变体

提供的函数通常是谓词。

<aside aria-label="Footnotes" class="FootnoteSection BookFrontmatterFootnoteSection" epub:type="footnotes">Footnotes [1](#Fn1_source)

[T2`https://en.wikipedia.org/wiki/Jakarta_EE`](https://en.wikipedia.org/wiki/Jakarta_EE)

 </aside>