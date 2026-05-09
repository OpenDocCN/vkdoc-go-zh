# Docker：容器化应用平台

Docker 是一个供开发人员和系统管理员在容器虚拟化环境中开发、交付和运行应用的平台。应用容器是一种轻量级的 Linux 环境，你可以利用它来部署和运行独立可部署的代码单元。本章简要讨论了微服务架构，在该架构中，独立可部署的服务单元被组合起来构建更大的应用程序。Docker 容器（应用容器）非常适合在微服务架构中运行微服务。Docker 不仅仅是一项技术；它是一个生态系统，让你能够从各种组件（类似于微服务架构中的微服务）中快速组合应用程序。

在传统计算环境中，你是为虚拟机（VM）开发应用程序的，其目标是理想化的硬件环境，包括操作系统、网络基础设施层等。使用应用容器的最大优势在于，它能将应用程序与基础设施及其运行环境分离开来，为应用程序开发者带来了巨大的机遇。借助 Docker，开发者现在可以针对理想化的操作系统（一个静态的 Linux 环境）开发应用程序，并尽可能快速地交付他们的应用。

Docker 消除了在部署和运行应用程序时可能出现的复杂性和障碍。它还在 Linux 容器技术之上提供了出色的抽象层，使得在容器虚拟化环境下工作变得容易，而这正成为 IT 行业的一场重大革命，这要归功于 Docker 生态系统。

Docker 是用 Go 语言开发的，Go 正成为构建许多创新系统的首选语言。在容器生态系统中，大多数系统都是用 Go 开发的。**Kubernetes** 就是一个用 Go 开发的技术示例，用于对应用容器进行集群管理。你也可以使用 Kubernetes 来对 Docker 应用程序进行集群管理。

尽管 Docker 是一种 Linux 技术，但你也可以通过使用 Docker Toolbox（`www.docker.com/docker-toolbox`）在 Mac 和 Windows 系统上运行它。要获取更多关于 Docker 的信息，请查阅其文档，网址为 `https://docs.docker.com/`。

Docker 生态系统由以下部分组成：

*   **Docker Engine**：一种轻量级、功能强大、开源的容器虚拟化技术，并集成了用于构建和容器化应用程序的工作流程。
*   **Docker Hub**：一个 SaaS 门户网站（`https://hub.docker.com/`），用于共享和管理被称为 Docker 镜像的应用堆栈。

在 Docker 中，镜像和容器是重要的概念。容器是加载了镜像的 Linux 操作系统的“精简”版本。容器是从镜像创建的，镜像是不可变的文件，提供了容器的快照。你可以对现有镜像进行更改，但会将其持久化为一个新镜像。镜像使用 `docker build` 命令创建。当使用 `docker run` 命令运行这些镜像时，就会生成一个容器。镜像存储在 Docker 注册表系统中，例如 Docker Hub（由 Docker 提供）以及私有注册表系统。

#### 编写 Dockerfile

在本章中，目标是创建一个 Dockerfile，来自动化 TaskManager 应用程序的构建过程，使其能在 Docker 化容器中运行。

在使用 Docker 时，你可以通过手动修改基础镜像或构建 Dockerfile 来创建应用容器。Dockerfile 是一个文本文档，包含了自动创建所需镜像的所有指令和命令，无需手动运行命令。Dockerfile 允许你自动化构建过程，以执行若干命令行指令。`docker build` 命令根据 Dockerfile 和构建上下文构建镜像。

通过将一个文本文件命名为 `Dockerfile`（不带文件扩展名）来创建 Dockerfile。通常，你会将此文件放在项目仓库的 `root` 目录下。

清单 9-23 提供了 TaskManager 应用程序的 Dockerfile。

**清单 9-23.** TaskManager 应用程序的 Dockerfile

```
##### 使用 golang 镜像，其中工作空间（GOPATH）配置在 /go 下。
FROM golang

##### 将本地包文件复制到容器的工作空间。
ADD . /go/src/github.com/shijuvar/go-web/taskmanager

##### 设置工作目录
WORKDIR /go/src/github.com/shijuvar/go-web/taskmanager

##### 获取 godeps 用于管理和恢复依赖项
RUN go get github.com/tools/godep

##### 恢复 godep 依赖项
RUN godep restore

##### 在容器内构建 taskmanager 命令。
RUN go install github.com/shijuvar/go-web/taskmanager

##### 容器启动时运行 taskmanager 命令。
ENTRYPOINT /go/bin/taskmanager

##### 服务监听 8080 端口。
EXPOSE 8080
```

让我们探索一下 Dockerfile 中使用的命令：

*   `FROM golang` 命令指示 Docker 从官方的 `golang` Docker 镜像开始，这是一个安装了最新版本 Go 的 Debian 镜像，并且工作空间（`GOPATH`）配置在 `/go`。
*   `ADD` 命令将本地代码复制到容器的工作空间。
*   `WORKDIR` 命令在容器内设置工作目录。
*   `RUN` 命令在容器内运行命令。使用 `RUN` 命令，可以安装 `godep` 工具。
*   `godep restore` 命令将依赖项恢复到容器的 `GOPATH` 位置。
*   使用 `go install` 命令，`taskmanager` 命令会在容器内的 `GOPATH/bin` 位置（`/go/bin`）运行。
*   `ENTRYPOINT` 命令指示 Docker 在容器启动时从 `/go/bin` 位置运行 `taskmanager` 命令。
*   `EXPOSE` 命令指示 Docker 容器在运行时监听指定的网络端口。它暴露了 8080 端口。`EXPOSE` 命令不会将容器的端口公开给外界。为此，需要使用发布标志（`--publish`）通过与外部 HTTP 端口映射来打开端口。

用于在 Docker 应用容器中构建和运行 TaskManager API 服务器的 Dockerfile 已经完成。现在让我们根据该 Dockerfile 构建镜像。在 TaskManager 应用程序的 `root` 目录下运行以下命令：

```
docker build -t taskmanager
```

通过执行 Dockerfile 中定义的指令，将构建一个本地镜像。生成的镜像标签为 `taskmanager`。

一个名为 `taskmanager` 的镜像（Docker 构建的结果镜像）已创建。使用以下命令从此镜像运行一个容器：

```
docker run --publish 80:8080 --name taskmanager_api --rm taskmanager
```

让我们探索一下 `docker run` 命令中使用的标志：

*   `--publish` 标志指示 Docker 将容器暴露的 8080 端口发布到外部 80 端口。
*   `--name` 标志为从 `taskmanager` 镜像创建的容器指定一个名称。该容器被命名为 `taskmanager_api`。
*   `--rm` 标志指示 Docker 在容器退出时移除容器镜像。否则，即使容器退出，容器镜像也会保留。

`docker run` 命令通过暴露外部端口 80 来运行容器。你可以通过导航到 `http://localhost:80` 来访问该服务器应用程序。

TaskManager 应用程序的 Dockerfile 提供了在应用容器内运行 HTTP 服务器所需的一切。当你将 TaskManager 应用程序通过 Docker 迁移到生产环境时，待处理的操作是将 MongoDB 实现在应用容器中。TaskManager 应用程序使用 MongoDB 作为持久化存储，因此你必须在另一个容器中运行 MongoDB 数据库。你可以使用一个容器来运行 HTTP 服务器，使用另一个容器来运行 MongoDB 数据库。



运行在容器中的容器化应用被称为在隔离环境中运行的受限服务。这种虚拟化方式使得容器之间相互隔离。但当你构建实际应用时，必须组合多个容器来形成一个应用。在这种情况下，你需要将来自 HTTP 服务器和 MongoDB 数据库的容器组合起来。

在此场景下，你可以使用 Docker Compose (`https://docs.docker.com/compose/`) 在 Docker 平台上定义并运行多容器应用。Docker 的设计理念是将可独立部署的微服务构建到容器中，再将这些服务组合起来构建更大的应用。

### Go Web 框架

在本章中，你学习了如何在没有使用 Go Web 框架的情况下从零开始开发一个 RESTful API 应用。在 Go Web 开发体系中，使用功能完备的 Web 框架在 Go 开发者社区中并不流行。Go 开发者更倾向于使用 Go 标准库包作为构建 Web 应用和 Web API 的基础。在标准库包之上，开发者会使用一些第三方库包来扩展标准库的功能，并借助一些工具函数来加速 Web 开发。

本章采用了同样的方法：使用 `net/http` 标准库包和少量第三方包来开发一个功能完整的应用。在我看来，开发 RESTful API 并不需要 Web 框架。你可以完全不用任何 Web 框架，就能在 Go 中构建高度可扩展的 RESTful API。尽管如此，在某些场景下，特别是开发传统 Web 应用时，使用 Web 框架可能会有所帮助。

以下是一些 Go Web 框架，如果你需要功能完备的框架可以考虑使用：

- Beego (`http://beego.me/`)：一个功能完备的 MVC 框架，内置了 ORM
- Martini (`https://github.com/go-martini/martini`)：一个受 Sinatra（一个 Ruby Web 框架）启发的 Web 框架
- Revel (`https://revel.github.io/`)：一个专注于高生产力的全栈 Web 框架
- Goji (`https://goji.io/`)：一个轻量级且极简的 Web 框架
- Gin (`https://github.com/gin-gonic/gin`)：一个具有 Martini 类似 API 并承诺更好性能的 Web 框架

### 总结

在本章中，你学习了如何使用 Go 构建一个生产可用的 RESTful API。你将 MongoDB 用作 RESTful API 应用的数据存储，并将应用逻辑组织到多个包中，以便于维护应用。（由于书籍章节的限制，应用中忽略了一些业务验证和最佳实践，但仍包含了一些最佳实践。）

使用了 `Negroni` 来处理中间件栈，并为特定路由添加了中间件。创建了一个用于在 HTTP 请求生命周期内保存值的结构体类型。在 HTTP 服务器启动前创建了一个 MongoDB `Session` 对象，并在每次 HTTP 请求执行应用处理器后，获取该 `Session` 对象的副本并关闭。Go 是构建高度可扩展 RESTful API 的优秀技术栈，也是使用微服务架构模式开发应用的完美技术栈。

Go 没有提供集中式仓库来管理第三方包，因此管理外部依赖有点困难。使用了第三方工具 `godep` 来管理 RESTful API 应用的依赖。它允许你将依赖恢复到 `GOPATH` 系统路径下。

Docker 是一个革命性的容器化应用生态系统，使开发者和系统管理员能够在容器虚拟化环境中开发、交付和运行应用。应用容器是一个轻量级的 Linux 环境。为 RESTful API 应用创建了 Dockerfile，以使用 Docker 自动化应用的构建过程。

你可以在 Go 中不使用任何 Web 框架来构建基于 Web 的可扩展后端系统。在本章中，通过使用标准库包 `net/http` 和少量第三方库，创建了一个生产可用的 RESTful API 应用。构建 RESTful API 时可能不需要 Web 框架，但在开发传统 Web 应用时，使用框架可能会有帮助。Beego 是一个功能完备的 Web 框架，提供了包括 ORM 在内的所有功能。

### 参考资料

`https://docs.docker.com/`

`https://github.com/tools/godep`

