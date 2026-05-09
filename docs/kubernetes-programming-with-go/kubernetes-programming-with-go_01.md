# 引言

早在 2017 年，我就职于一家构建视频流媒体软件的公司。那年年底，包括我在内的一支小团队接到了一个新任务：将公司开发的视频 CDN 部署在 Kubernetes 上。我们决定探索使用自定义资源和 Operator 的概念来部署这个 CDN。

当时的 Kubernetes 版本是 1.9，自定义资源定义（CRD）概念刚刚在 1.7 版本中发布，而`sample-controller`仓库是我们已知的唯一能帮助构建 Operator 的文档。由于 Kubernetes 生态系统异常活跃，在接下来的几个月里涌现出各种工具，特别是 Kubebuilder SDK。就这样，我们的项目启动了。

从那一刻起，我花了大量时间探索如何构建 Operator 以及其它与 Kubernetes API 交互的程序。但问题已经形成：我是从具体到一般开始学习 Kubernetes 编程的，因而花了很长时间才完全理解 Kubernetes API 的内部机制。

我写这本书的初衷是希望它能教会新的 Kubernetes 开发者如何用 Go 语言从一般到具体地进行 Kubernetes API 编程。

