# 章节概览

本书的目标读者应具备使用 REST API 的经验，无论是通过 HTTP 还是使用特定语言的客户端进行访问；并且对 Kubernetes 平台有一定的了解（基本上是作为用户层面），例如，有借助 YAML 清单部署此类 API 或前端应用的经验。

*   本书第一章探讨了 Kubernetes API 及其如何实现 REST 原则。它特别关注 API 提出的组-版本-资源（Group-Version-Resource）组织方式和种类（Kind）概念。
*   第二章继续介绍 API 提供的操作以及每个操作在 HTTP 协议下的细节。
*   第三章至第五章描述了用于操作 Kubernetes API 的通用“低级”Go 库：API 库和 API Machinery 库。
*   第六章和第七章涵盖了`Client-go`库——在 Go 中操作 Kubernetes API 的高级库——以及如何使用该库进行单元测试。

至此，读者应已能够熟练构建与 Kubernetes API 原生资源交互的 Go 应用程序。

*   第八章和第九章介绍了自定义资源的概念以及如何在 Go 中操作它们。
*   第十章至第十二章涵盖了如何使用`controller-runtime`库实现 Kubernetes Operator。
*   第十三章探索了 Kubebuilder SDK，这是一个帮助开发和部署 Kubernetes Operator 的工具。

读完本书，读者应能开始用 Go 语言构建 Kubernetes Operator，并对幕后发生的事情有非常深刻的理解。

