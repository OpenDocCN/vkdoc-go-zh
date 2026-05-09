# 13. 漏洞扫描器

云服务商的普及使组织能够以可负担的规模部署应用程序。然而，大规模部署应用是一回事，保护应用和资源的安全又是另一回事，这已成为各地组织的难题。安全是一个涵盖众多方面的重大议题。在本章中，你将了解一种用于识别基础设施漏洞的工具。

我们将重点探讨一款在 Linux 系统中检测漏洞的工具。本章的主要目标是理解如何使用该工具及其适用场景，同时通过深入分析源代码，更好地理解其工作原理。在本章中，你将学习：

* 漏洞扫描器的工作原理
* 该工具如何运用不同技术实现其目标
* 端口扫描、命令行执行以及 Go 语言中使用 SQLite 的数据库操作

### 源代码

本章的源代码可从 [`https://github.com/Apress/Software-Development-Go`](https://github.com/Apress/Software-Development-Go) 仓库获取。

### 漏洞扫描器

漏洞扫描器是用于搜索并报告 IT 基础设施中已知漏洞的工具。每个组织都拥有自管或云端管理的 IT 基础设施，其中运行着各种应用程序、网络及其他组件，在安全方面需要持续监控。每天我们都会读到关于新漏洞被发现或被利用的新闻，这些漏洞可能对组织甚至更广泛的社区造成损害。

漏洞扫描器这类工具使用了大量值得学习的有趣技术栈，这正是本章的意图所在。我们将研究一个名为 Vuls 的开源项目（[`https://github.com/future-architect/vuls`](https://github.com/future-architect/vuls)），它使用 Go 语言编写，并观察它如何实现部分功能。目标是将这些知识应用到你自己的项目中，或将其作为知识库来理解此类工具的工作原理。请注意，本章并非安装或使用 Vuls 或漏洞扫描器的专项指南。

本章选择 Vuls 的原因在于该项目由社区持续维护和更新，且拥有较高的星级评分。该工具使用来自不同来源的数据库信息，而非依赖自有数据源，从而在漏洞检测方面保持更新。

Vuls 提供的主要功能包括：

* 支持扫描主流 Linux/FreeBSD 操作系统的漏洞
* 可用于扫描 Amazon、Google 等云基础设施
* 使用多个漏洞数据库，其中之一是美国国家标准与技术研究院（NIST）的国家漏洞数据库
* 可根据需求进行快速扫描、深度扫描或其他类型的扫描
* 通过电子邮件或 Slack 频道发送通知

下一节中，你将下载源代码、编译并使用它来理解其工作原理。

### 使用 Vuls

在本节中，你将探索 Vuls 并执行以下操作：

* 检出代码
* 在本地机器上运行扫描

#### 检出代码

Vuls 需要 Go 1.8 环境，请确保在继续之前已安装。检出代码最简单的方法是使用 `go get` 命令，如下所示：

```
GO111MODULE=off go get github.com/future-architect/vuls
```

确保已将 `GOPATH` 目录设置为用于存储 Go 模块的正确文件夹（我的 `GOPATH` 指向 `/home/nanik/Gopath`）。命令成功运行后，代码将下载到 `GOPATH` 目录下的 `src/github.com/future-architect/vuls` 文件夹中，示例如下：

```
...
drwxrwxr-x  2 nanik nanik   4096 Jun 26 15:48 detector
-rw-rw-r--  1 nanik nanik    596 Jun 26 15:48 Dockerfile
-rw-rw-r--  1 nanik nanik     55 Jun 26 15:48 .dockerignore
...
drwxrwxr-x  2 nanik nanik   4096 Jun 26 15:48 saas
drwxrwxr-x  2 nanik nanik   4096 Jun 26 15:48 scanner
-rw-rw-r--  1 nanik nanik    137 Jun 26 15:48 SECURITY.md
drwxrwxr-x  2 nanik nanik   4096 Jun 26 15:48 server
drwxrwxr-x  3 nanik nanik   4096 Jun 26 15:48 setup
drwxrwxr-x  2 nanik nanik   4096 Jun 26 15:48 subcmds
drwxrwxr-x  2 nanik nanik   4096 Jun 26 15:48 tui
drwxrwxr-x  2 nanik nanik   4096 Jun 26 15:48 util
```

本章的讨论聚焦于 `v0.19.7` 版本，因此需要切换到不同分支。切换到 `GOPATH` 下的 `src/github.com/future-architect/vuls` 目录，并按如下方式切换分支：

```
git checkout v0.19.7
```

代码已就绪，可以开始构建。使用 `make` 命令进行构建：

```
make build
```

编译过程开始，所有相关模块将被下载。编译完成后，你将获得一个名为 `Vuls` 的可执行文件。按如下方式运行应用程序：

```
./vuls
```

你将看到类似如下的输出：

```
Usage: vuls   
Subcommands:
commands         list all command names
flags            describe all known top-level flags
help             describe subcommands and their syntax
Subcommands for configtest:
configtest       Test configuration
Subcommands for discover:
discover         Host discovery in the CIDR
Subcommands for history:
history          List history of scanning.
Subcommands for report:
report           Reporting
Subcommands for scan:
scan             Scan vulnerabilities
Subcommands for server:
server           Server
Subcommands for tui:
tui              Run Tui view to analyze vulnerabilities
Use "vuls flags" for a list of top-level flags
```



#### 运行扫描

Vuls 需要一个 `.toml` 格式的配置文件。在本节中，你可以使用 `chapter13` 目录下名为 `config.toml` 的配置文件，内容如下：

```
[servers.localhost]
host = "localhost"
port = "local"
```

该配置指定了要扫描的机器。在你的示例中，它是你本地机器上的 localhost。它执行标准的本地模式扫描操作，该操作不包含端口扫描（就像被排除的服务一样）。

按如下方式运行 Vuls：

```
./vuls scan --config /config.toml --debug --results-dir 
```

Vuls 使用 `–config` 参数指定的配置运行，并将报告存储在 `–results-dir` 参数指定的目录中。你会看到类似下面的详细输出：

```
[Jun 26 18:33:45]  INFO [localhost] vuls-v0.19.7-build-20220626_181254_91ed318
[Jun 26 18:33:45]  INFO [localhost] Start scanning
[Jun 26 18:33:45]  INFO [localhost] config: /home/nanik/Downloads/config.toml
[Jun 26 18:33:45] DEBUG [localhost] map[string]config.ServerInfo{
"localhost": config.ServerInfo{
ServerName:         "localhost",
User:               "",
...
SSHConfigPath:      "",
KeyPath:            "",
...
}
[Jun 26 18:33:45]  INFO [localhost] Validating config...
...
[Jun 26 18:33:45] DEBUG [localhost] execResult: servername:
cmd: ls /etc/debian_version
exitstatus: 0
stdout: /etc/debian_version
...
[Jun 26 18:33:45] DEBUG [localhost] Executing... cat /etc/issue
[Jun 26 18:33:45] DEBUG [localhost] execResult: servername:
cmd: cat /etc/issue
...
[Jun 26 18:33:45] DEBUG [localhost] Executing... lsb_release -ir
...
cmd: lsb_release -ir
exitstatus: 0
...
[Jun 26 18:33:45] DEBUG [localhost] Executing... type curl
[Jun 26 18:33:45] DEBUG [localhost] execResult: servername:
cmd: type curl
exitstatus: 0
stdout: curl is /usr/bin/curl
...
Scan Summary
================
localhost    pop22.04    2378 installed, 0 updatable
```

Vuls 生成的扫描报告包含关于已扫描或发现内容的全面信息。报告内容如下：

```
{
"jsonVersion": 4,
"lang": "",
"serverUUID": "",
"serverName": "192-168-1-3",
"family": "pop",
"release": "22.04",
...
"ipv4Addrs": [
"192.168.1.3"
],
"ipv6Addrs": [
...
],
"scannedAt": "2022-06-26T18:40:16.045650086+10:00",
"scanMode": "fast mode",
"...
"scannedVia": "remote",
"scannedIpv4Addrs": [
...
],
"scannedIpv6Addrs": [
...
],
"reportedAt": "0001-01-01T00:00:00Z",
"reportedVersion": "",
"reportedRevision": "",
"reportedBy": "",
"errors": [],
...
"release": "5.17.5-76051705-generic",
"version": "",
"rebootRequired": false
},
"packages": {
...
}
},
"config": {
"scan": {
"debug": true,
"logDir": "/var/log/vuls",
"logJSON": false,
"resultsDir": "/home/nanik/go/src/github.com/future-architect/vuls/result",
"default": {},
"servers": {
"192-168-1-3": {
...
}
},
"cveDict": {
...
},
"ovalDict": {
...
},
...
},
"report": {
"logJSON": false,
...
}
}
}
```

在下一节中，你将探索 Vuls 提供的一些功能。

### 从 Vuls 中学习

Vuls 中有许多功能值得学习，并可以在开发系统或安全应用程序时应用。你将了解 Vuls 使用的三个主要功能：端口扫描、使用 SQLite 数据库以及从 Go 应用程序执行命令行。这些功能将在以下部分详细讨论。

#### 端口扫描

端口扫描是一种执行操作以确定网络中哪些端口处于打开状态的方法。端口就像是应用程序选择监听的数字。例如，HTTP 服务器监听端口 80，而 FTP 服务器监听端口 21。不同操作系统中遵循的标准端口号列表可以在 [`www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml`](http://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml) 上查看。

查看 Vuls 源代码（`scanner/base.go`），你可以看到执行网络扫描的以下函数：

```
package scanner
import (
...
nmap "github.com/Ullaakut/nmap/v2"
)
func (l *base) execExternalPortScan(scanDestIPPorts map[string][]string) ([]string, error) {
...
baseCmd := formatNmapOptionsToString(portScanConf)
listenIPPorts := []string{}
for ip, ports := range scanDestIPPorts {
...
scanner, err := nmap.NewScanner(nmap.WithBinaryPath(portScanConf.ScannerBinPath))
...
return listenIPPorts, nil
}
```

该代码使用了来自 `github.com/Ullaakut/nmap` 的开源 `nmap` 库来执行扫描操作。在深入了解该库如何执行 `nmap` 之前，我们先了解一下什么是 `nmap`。工具 `nmap` 是一个用于网络探索和安全审计的命令行工具。它用于收集网络的实时信息，检测网络环境中哪些端口是开放的，检查网络中哪些 IP 地址处于激活状态等等。

请确保你的本地机器上安装了 `nmap` 工具。如果你使用的是基于 Debian 的 Linux 发行版，请使用以下命令进行安装：

```
sudo apt install nmap
```

运行 `nmap` 以检查是否能够成功运行。

```
nmap
```

你会看到类似下面的输出：

```
Nmap 7.80 ( https://nmap.org )
Usage: nmap [Scan Type(s)] [Options] {target specification}
TARGET SPECIFICATION:
...
HOST DISCOVERY:
...
SCAN TECHNIQUES:
...
EXAMPLES:
nmap -v -A scanme.nmap.org
nmap -v -sn 192.168.0.0/16 10.0.0.0/8
nmap -v -iR 10000 -Pn -p 80
```

让我们看一下 `chapter13/nmap` 目录中提供的示例代码，并按如下方式运行：

```
go run main.go
```

该应用程序会运行并扫描你本地机器的开放端口。在我的机器上，输出如下：

```
Host "127.0.0.1":
Port 22/tcp open ssh
Port 631/tcp open ipp
Port 5432/tcp open postgresql
Nmap done: 1 hosts up scanned in 0.020000 seconds
```

该代码检测到了三个开放端口，分别与 `ssh`、`ipp` 和 `postgresql` 应用程序相关。根据你本地机器上开放的端口，你可能会得到不同的结果。

使用 `nmap` 库的代码片段如下：

```
package main
import (
...
"github.com/Ullaakut/nmap/v2"
)
func main() {
...
scanner, err := nmap.NewScanner(
nmap.WithTargets("localhost"),
nmap.WithContext(ctx),
)
...
}
```

该示例代码通过调用 `nmap.NewScanner(..)` 来初始化库。在该函数内部，初始化代码会检查 `nmap` 工具是否已安装，如下面的代码片段所示：

```
func NewScanner(options ...Option) (*Scanner, error) {
...
if scanner.binaryPath == "" {
var err error
scanner.binaryPath, err = exec.LookPath("nmap")
if err != nil {
return nil, ErrNmapNotInstalled
}
}
...
return scanner, nil
}
```

该函数使用 Go 的 `os/exec` 包来检查 `nmap` 工具的存在性。一旦库成功初始化，它会调用 `Run()` 函数来执行扫描操作。

```
package main
import (
...
"github.com/Ullaakut/nmap/v2"
)
func main() {
...
result, warnings, err := scanner.Run()
...
}
```

库的 `Run()` 函数执行以下任务：

*   使用提供的参数执行 `nmap` 工具
*   执行 Go 协程等待来自 `nmap` 工具的结果，该结果将被解析并转换为结构体，然后返回给调用者



变量`result`的类型是`Run`结构体，在库中声明如下：

```
type Run struct {
XMLName xml.Name `xml:"nmaprun"`
Args             string         `xml:"args,attr" json:"args"`
ProfileName      string         `xml:"profile_name,attr" json:"profile_name"`
Scanner          string         `xml:"scanner,attr" json:"scanner"`
StartStr         string         `xml:"startstr,attr" json:"start_str"`
Version          string         `xml:"version,attr" json:"version"`
XMLOutputVersion string         `xml:"xmloutputversion,attr" json:"xml_output_version"`
Debugging        Debugging      `xml:"debugging" json:"debugging"`
Stats            Stats          `xml:"runstats" json:"run_stats"`
ScanInfo         ScanInfo       `xml:"scaninfo" json:"scan_info"`
Start            Timestamp      `xml:"start,attr" json:"start"`
Verbose          Verbose        `xml:"verbose" json:"verbose"`
Hosts            []Host         `xml:"host" json:"hosts"`
PostScripts      []Script       `xml:"postscript>script" json:"post_scripts"`
PreScripts       []Script       `xml:"prescript>script" json:"pre_scripts"`
Targets          []Target       `xml:"target" json:"targets"`
TaskBegin        []Task         `xml:"taskbegin" json:"task_begin"`
TaskProgress     []TaskProgress `xml:"taskprogress" json:"task_progress"`
TaskEnd          []Task         `xml:"taskend" json:"task_end"`
NmapErrors []string
rawXML     []byte
}
```

来自`nmap`的原始输出（XML 格式）如下所示：

```

...

```

#### Exec

在 Vuls 中频繁使用的下一个功能是执行外部工具以作为扫描过程的一部分执行某些操作。以下是一些 Vuls 用于获取网络 IP 信息、获取内核信息、更新包管理器索引以及许多其他功能的命令。

| `apk update` | 从仓库下载更新的包索引 |
| `/sbin/ip -o addr` | 列出网络设备、路由和其他网络相关信息 |
| `uname -r` | 打印系统信息 |
| `stat /proc/1/exe` | 获取 PID 1 的信息 |
| `systemctl status` | 列出注册到 system 的信息 |

不同操作系统使用的命令不同，但使用`os/exec`包运行它们的方式是相同的。

查看位于`chapter13/exec`文件夹中的示例应用，并按如下方式在终端中运行它：

```
go run main.go
```

你将获得如下输出：

```
2022/06/26 21:18:28 --------------
2022/06/26 21:18:28 Running ip link
2022/06/26 21:18:28 --------------
2022/06/26 21:18:28 1: lo:  mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
...
2022/06/26 21:18:28
2022/06/26 21:18:28 ---------------
2022/06/26 21:18:28 Running noexist
2022/06/26 21:18:28 ---------------
2022/06/26 21:18:28 %v exit status 127
2022/06/26 21:18:28 Running uname -r
2022/06/26 21:18:28 ----------------
2022/06/26 21:18:28 5.17.5-76051705-generic
```

该示例应用使用`os/exec`包来执行命令并将输出打印到控制台。以下代码片段展示了使用`os/exec`包的函数：

```
package main
import (
..
ex "os/exec"
)
func main() {
...
Run("ip link")
...
Run("noexist")
...
Run("uname -r")
}
func Run(arg string) {
var cmd *ex.Cmd
cmd = ex.Command("/bin/sh", "-c", arg)
...
}
```

`Run(..)`函数接收一个字符串参数，该参数在调用`Command(..)`函数时被添加到参数中。该示例将传递给`Run(..)`函数的参数作为`/bin/sh`命令工具的一部分来运行。例如，当调用`Run("ip link")`时，它会按如下方式运行：

```
/bin/sh -c ip link
```

该应用指定将输出存储到变量中，以便打印到控制台：

```
func Run(arg string) {
...
cmd.Stdout = &stdoutBuf
cmd.Stderr = &stderrBuf
log.Println(stdoutBuf.String())
log.Println(stderrBuf.String())
}
```

#### SQLite

在本节中，你将学习如何使用 SQLite 数据库。特别地，你将学习如何使用`sqlite3`库来读写数据库。

SQLite 是一个轻量级、自包含的 SQL 数据库，允许应用程序读取和存储信息。应用程序使用标准的 SQL 语法执行不同类型的数据操作，例如插入、更新和删除数据。SQLite 的轻量级和可移植性使其成为不需要集中式数据库的项目中有吸引力的选择。像 Android 这样的手机使用 SQLite 数据库，应用程序可以将其用于移动应用。

在内部，Vuls 大量使用 SQLite 来存储从不同来源下载的数据。你将查看使用 SQLite 的示例应用程序。本节示例代码位于`chapter13/sqlite`目录中。让我们按如下方式从终端运行示例应用程序：

```
go run main.go
```

你将获得如下输出：

```
2022/06/27 19:40:04 Initialize database -  local.db
2022/06/27 19:40:04 Creating table in -   local.db
2022/06/27 19:40:04 Inserting data into -  local.db
Reading Table:
2022/06/27 19:40:04 Total rows read -  [{0 CAD} {1 AUD} {2 AUD} {3 GBP} {4 CAD} {5 EUR} {6 USD} {7 USD} {8 CAD} {9 USD} {10 GBP} {11 CAD} {12 AUD} {13 GBP} {14 EUR} {15 GBP} {16 CAD} {17 USD} {18 AUD} {19 CAD} {20 USD} {21 CAD} {22 EUR} {23 EUR} {24 AUD}]
```

示例代码创建了一个名为`local.db`的新数据库，并创建了一个名为`currencies`的新表。它还向其中插入了一些数据，并将新插入的数据打印到控制台。

以下代码片段展示了初始化数据库的代码：

```
package main
import (
...
_ "github.com/mattn/go-sqlite3"
...
)
...
func main() {
...
dbHandle = InitDB(dbname)
...
}
// InitDB initialize database
func InitDB(filepath string) *sql.DB {
db, err := sql.Open("sqlite3", filepath)
if err != nil {
panic(err)
}
return db
}
```

`InitDB`函数使用`sql.Open`创建新数据库，并传入`sqlite3`作为参数。`sqlite3`参数被`database/sql`模块用作查找适当驱动程序的引用。如果成功，它将返回存储在`db`变量中的`sql.DB`结构体。

`sql.DB`结构体在`database/sql`模块中声明如下：

```
type DB struct {
...
connector driver.Connector
...
closed            bool
...
stop func()
}
```

数据库成功创建后，代码会创建一个名为`currencies`的表，由以下`InitTable(..)`函数执行：

```
...
func InitTable(db *sql.DB) {
q := `
CREATE TABLE IF NOT EXISTS currencies(
Id TEXT NOT NULL PRIMARY KEY,
Name TEXT,
InsertedDatetime DATETIME
);`
_, err := db.Exec(q)
if err != nil {
log.Fatal(err)
}
}
```

该函数使用`db.Exec(..)`函数执行`CREATE TABLE..`SQL 命令。函数`db.Exec(..)`用于对数据库执行查询而不返回任何行。返回值类型为`Result`，在`InitTable(..)`函数中未使用。`Result`结构体在`database/sql`模块中声明如下：

```
type Result interface {
LastInsertId() (int64, error)
RowsAffected() (int64, error)
}
```

成功创建表后，代码继续插入数据。此操作分为两部分。第一部分是准备要插入的数据，如下代码片段所示：

```
func main() {
...
records := []Record{}
for i := 0; i < 25; i++ {
r := (rand.Intn(len(curNames)-0) + 0)
d := strconv.Itoa(i)
rec := Record{Id: d, Name: curNames[r]}
records = append(records, rec)
}
...
}
```

代码创建了一个`Record`结构体数组并填充它，然后将填充后的数组作为参数传递给`InsertData(..)`函数：



`func InsertData(db *sql.DB, records []Record) { q := \` INSERT OR REPLACE INTO currencies( Id, Name, InsertedDatetime ) values(?, ?, CURRENT_TIMESTAMP)\` stmt, err := db.Prepare(q) ... defer stmt.Close() for _, r := range records { _, err := stmt.Exec(r.Id, r.Name) ... } }`

该函数使用了`INSERT INTO`语句，该语句被用于`Prepare()`函数内部。`Prepare()`函数用于创建预处理语句，这些语句可以在之后独立执行。SQL 语句中的值使用参数占位符（占位符由`?`符号标记），这些占位符在使用`Exec()`函数执行时作为参数的一部分被传入。值从`Record`结构体的`Id`和`Name`字段获取。

现在数据已插入表中，代码通过从表中读取数据并将其打印到控制台来完成执行，如下所示：

`func ReadData(db *sql.DB) []Record { q := \` SELECT Id, Name FROM currencies ORDER BY datetime(InsertedDatetime) DESC\` rows, err := db.Query(q) ... var records []Record for rows.Next() { item := Record{} err := rows.Scan(&item.Id, &item.Name) ... records = append(records, item) } return records }`

`ReadData()`函数使用`SELECT` SQL 语句从表中读取字段，结果按`InsertedDateTime`字段降序排序。代码使用了`Query()`函数，返回`Rows`结构体并对其循环遍历。在循环内部，代码使用`Scan()`函数从每一行复制字段，并将数据读取到参数中传入的变量中。在本代码示例中，字段被读取到`item.Id`和`item.Name`中。

传递给`Scan()`的参数数量必须与从表中读取的字段数量一致。使用`Query()`函数时返回的`Rows`结构体定义在`database/sql`模块中。

```
type Rows struct {
    dc          *driverConn
    releaseConn func(error)
    ...
    closemu sync.RWMutex
    closed  bool
    lasterr error
    ...
}
```

### 总结

在本章中，你了解了一个名为 Vuls 的开源安全项目，它提供了漏洞扫描能力。你通过查看代码并在本地机器上执行扫描操作来学习 Vuls。

Vuls 提供了许多功能。在学习 Vuls 工作原理的过程中，你了解了端口扫描、从 Go 语言执行外部命令行应用程序，以及编写使用 SQLite 执行数据库操作的代码。

