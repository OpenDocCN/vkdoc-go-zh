# 13. 使用 Kubebuilder 创建 Operator

在前面的章节中，你已经学习了如何使用**自定义资源定义**（CRD）定义由 API 服务器提供的新资源，以及如何使用`controller-runtime`库构建 Operator。

`kubebuilder` SDK 专门用于帮助你创建新资源及其相关的 Operator。它提供了用于引导项目（定义 Manager）以及向项目添加资源及其相关控制器的命令。

生成了新自定义资源和控制器的源代码后，你需要根据资源的业务领域实现缺失的部分。`kubebuilder` SDK 随后会提供工具，用于构建自定义资源定义和 Manager，并将其部署到集群。

### 安装 Kubebuilder

`kubebuilder` 以单个二进制文件的形式提供。你可以从项目的发布页面^(⁴)下载该二进制文件，并将其安装到你的`PATH`中。该二进制文件适用于 Linux 和 MacOS 系统。

### 创建项目

第一步是创建项目。项目最初将包含：

*   定义 Manager 的 Go 源代码（目前没有任何控制器）
*   一个`Dockerfile`，用于构建包含 Manager 二进制文件的镜像，以便部署到集群
*   用于帮助将 Manager 部署到集群的 Kubernetes 清单文件
*   一个`Makefile`，其中定义了用于测试、构建和部署 Manager 的命令

要创建项目，首先创建一个空目录并`cd`进入该目录，然后执行以下`kubebuilder init`命令：

*   ❶ **域名**，用作 GVK 组名称的后缀。此项目中定义的自定义资源可以属于多个组，但所有组都将属于同一个域。例如，`mygroup1.myid.dev`和`mygroup2.myid.dev`
*   ❷ **Go 模块名称**，用于所生成的 Manager Go 代码

```bash
$ mkdir myresource-kb
$ cd myresource-kb
$ kubebuilder init \
    --domain myid.dev \
    --repo github.com/myid/myresource
```

你可以通过运行以下命令来查看`Makefile`中可用的命令：

```bash
$ make help
```

你可以使用以下命令构建 Manager 的二进制文件：

```bash
$ make build
```

然后，你可以本地运行 Manager：

```bash
$ make run
```

或者

```bash
$ ./bin/manager
```

此时，最好初始化一个源码控制项目（例如，一个 git 项目），并用生成的这些文件创建第一个修订版本。这样，你将能够检查后续执行的`kubebuilder`命令所做的更改。例如，如果你使用 git：

```bash
$ git init
$ git commit -am 'kubebuilder init --domain myid.dev --repo github.com/myid/myresource'
```


## 向项目添加自定义资源

目前，Manager 并未管理任何 Controller。即使您可以构建并运行它，也无法用它执行任何操作。

接下来要执行的 `kubebuilder` 命令是 `kubebuilder create api`，用于向项目添加自定义资源及其相关的 Controller。该命令将询问您是否要创建该资源及其 Controller。请对每个问题回复 **y**。

*   ❶ 自定义资源的组。该组名后会自动拼接域名，以构成完整的 GVK 组。
*   ❷ 资源的版本。
*   ❸ 资源的类型（Kind）。

```
$ kubebuilder create api
--group mygroup                   ❶
--version v1alpha1                ❷
--kind MyResource                 ❸
Create Resource [y/n]
y
Create Controller [y/n]
y
```

您可以通过运行以下 `git` 命令来查看该命令所做的更改：

```
$ git status
On branch main
Changes not staged for commit:
(use "git add ..." to update what will be committed)
(use "git restore ..." to discard changes in working directory)
modified:   PROJECT
modified:   go.mod
modified:   go.sum
modified:   main.go
Untracked files:
(use "git add ..." to include in what will be committed)
api/
config/crd/
config/rbac/myresource_editor_role.yaml
config/rbac/myresource_viewer_role.yaml
config/samples/
controllers/
no changes added to commit (use "git add" and/or "git commit -a")
```

`PROJECT` 文件包含了项目的定义。最初它只包含作为 `init` 命令标志提供的域名和 `repo`。现在，它也包含了第一个资源的定义。`main.go` 文件和 `controllers` 目录定义了该自定义资源的 Controller。

`api/v1alpha1` 目录已被创建，其中包含使用 Go 结构体定义的自定义资源，以及由 `deepcopy-gen`（参见第 8 章的“运行 `deepcopy-gen`”部分）在 `controller-gen` 工具的帮助下生成的代码。它还包含了 `AddToScheme` 函数的定义，用于将此新自定义资源添加到 Scheme 中。

`config/samples` 目录包含一个新文件，以 YAML 格式定义了一个自定义资源实例。`config/rbac` 目录包含两个新文件，定义了新的 `ClusterRole` 资源——一个用于查看，一个用于编辑 `MyResource` 实例。`config/crd` 目录包含用于构建 CRD 的 kustomize 文件。

## 构建和部署 Manifest 文件

以下命令构建要部署到集群的 Manifest 文件：

```
$ make manifests
```

此命令会构建两个 Manifest 文件：

*   `config/rbac/role.yaml` – 将被分配给 Manager 使用的服务账户的 `ClusterRole`，授予对自定义资源的访问权限。
*   `config/crd/bases/mygroup.myid.dev_myresources.yaml` – 自定义资源的定义。

以下命令将部署定义 CRD 的 Manifest 文件：

```
$ make install
[...]
bin/kustomize build config/crd | kubectl apply -f -
customresourcedefinition.apiextensions.k8s.io/myresources.mygroup.myid.dev created
```

## 本地运行 Manager

这次，Manager 在管理一个 Controller，负责协调 `MyResource` 实例。由 `kubebuilder` 生成的 `Reconcile` 函数目前不执行任何操作，直接返回且不报错：

```
func (r *MyResourceReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
_ = log.FromContext(ctx)
// TODO(user): your logic here
return ctrl.Result{}, nil
}
```

当 `Reconcile` 函数被调用时，让我们添加一条日志：

```
-     _ = log.FromContext(ctx)
+     log := log.FromContext(ctx)
+     log.Info("reconcile")
```

以下命令将在本地执行 Manager。它将使用您的 `kubeconfig` 文件以及关联的权限连接到集群：

```
$ make run
[...]
INFO     setup     starting manager
INFO     Starting server     [...]
INFO     Starting server     [...]
INFO     Starting EventSource     [...]
INFO     Starting Controller     [...]
INFO     Starting workers     [...]
```

在另一个终端中，您可以使用提供的示例来创建一个 `MyResource` 实例：

```
$ kubectl apply -f config/samples/mygroup_v1alpha1_myresource.yaml
```

在第一个终端中，应该会出现一条新日志，表明 `Reconcile` 函数已被调用：

```
$ make run
[...]
INFO     reconcile          [...]
```

## 个性化自定义资源

`kubebuilder create api` 命令创建了一个包含 `Spec` 和 `Status` 字段的资源模板，其中 Spec 包含一个 `Foo` 字段，Status 为空。

在本节中，您将了解如何更新此模板以添加必要的字段。

### 编辑 Go 结构体

您可以编辑 `api/v1alpha1/myresource_types.go` 文件来个性化自定义资源。在此示例中，您将使用 `Image` 和 `Memory` 字段作为 `Spec`，并使用 `State` 字段作为 `Status`：

```
// MyResourceSpec defines the desired state of MyResource
type MyResourceSpec struct {
Image  string            `json:"image"`
Memory resource.Quantity `json:"memory"`
}
// MyResourceStatus defines the observed state of MyResource
type MyResourceStatus struct {
State string `json:"state"`
}
```

### 启用 Status 子资源

您可以看到 `MyResource` 结构体前的注释中的注解。

第一个注解 `//+kubebuilder:object:root=true` 表明 `MyResource` 是一个 *Kind*。有了这个注解，`kubebuilder` 将为此结构体生成 `DeepCopyObject()` 方法。更多详情请参见第 9 章的“添加注解”部分。

第二个注解 `//+kubebuilder:subresource:status` 表明必须为 `MyResource` 启用 `status` 子资源。有了这个注解，`kubebuilder` 会将 status 作为子资源添加到 `config/crd/bases/` 中 CRD 的 YAML 定义中。更多详情请参见第 8 章的“资源版本定义”部分。

`Status` 默认已启用，因此您无需对这些注解进行任何更改。

### 定义打印机列

第 8 章的“附加打印机列”部分展示了如何通过编辑 CRD 的 YAML 定义来为自定义资源声明打印机列。

使用 `kubebuilder` 时，CRD 的 YAML 定义是自动生成的，因此您不能直接编辑它。取而代之的是，您可以使用 `+kubebuilder:printcolumn` 注解。每个注解都接受 `name`、`type` 和 `JSONPath` 值，这与 YAML 定义中的 `additionalPrinterColumns` 条目类似。

例如，添加以下注解将定义 `Image`、`State` 和 `Age` 打印机列：

```
// +kubebuilder:printcolumn:name="Image",type=string,
JSONPath=`.spec.image`
// +kubebuilder:printcolumn:name="State",type=string,
JSONPath=`.status.state`
// +kubebuilder:printcolumn:name="Age",type="date",
JSONPath=".metadata.creationTimestamp"
```



### 重新生成文件

要重新生成 `DeepCopy` 方法以反映对结构体所做的更改，你需要运行以下命令：

```
$ make
```

要使用新字段重新生成 CRD 的 YAML 定义，必须执行此命令：

```
$ make manifests
```

你还可以编辑示例文件来调整示例资源。该文件是 `config/samples/mygroup_v1alpha1_myresource.yaml`，你可以通过以下方式调整其内容：

```
$ cat > config/samples/mygroup_v1alpha1_myresource.yaml <<EOF
apiVersion: mygroup.myid.dev/v1alpha1
kind: MyResource
metadata:
labels:
app.kubernetes.io/name: myresource
app.kubernetes.io/instance: myresource-sample
app.kubernetes.io/part-of: myresource-kb
app.kuberentes.io/managed-by: kustomize
app.kubernetes.io/created-by: myresource-kb
name: myresource-sample
spec:
image: nginx
memory: 512Mi
EOF
```

现在，CRD 和示例已经兼容，因此你可以将 CRD 应用到集群并更新示例资源：

```
$ make install
[...]
bin/kustomize build config/crd | kubectl apply -f -
customresourcedefinition.apiextensions.k8s.io/myresources.mygroup.myid.dev configured
$ kubectl apply -f
config/samples/mygroup_v1alpha1_myresource.yaml
myresource.mygroup.myid.dev/myresource-sample configured
```

### 实现 Reconcile 函数

由于 `kubebuilder` 基于 `controller-runtime` 库生成代码，你可以几乎不做修改地重用为第 11 章中 `Reconcile` 函数编写的代码。此外，你还可以重用第 12 章中为测试 `Reconcile` 函数而编写的源代码。

### 添加 RBAC 注解

当本地运行 Operator 时，Operator 会使用你的 `kubeconfig` 文件，该文件具有该 `kubeconfig` 特定的授权。如果你使用 `cluster-admin` 账户连接，Operator 将拥有集群上的所有授权，并能执行任何操作。

然而，当 Operator 部署在集群上时，它会使用特定的 Kubernetes Service Account 运行，并被授予受限的授权。这些授权在由 `kubebuilder` 构建和部署的 `ClusterRole` 中定义。

为了帮助 Kubebuilder 构建此 `ClusterRole`，在 `Reconcile` 函数生成的注释中存在注解（为清晰起见，添加了换行符）：

```
//+kubebuilder:rbac:
groups=mygroup.myid.dev,
resources=myresources,
verbs=get;list;watch;create;update;patch;delete
//+kubebuilder:rbac:
groups=mygroup.myid.dev,
resources=myresources/status,
verbs=get;update;patch
//+kubebuilder:rbac:
groups=mygroup.myid.dev,
resources=myresources/finalizers,
verbs=update
```

这些规则将授予对 MyResource 资源的完全访问权限，但不授予对其他资源的访问权限。

一旦开始监听 Deployment 资源，`Reconcile` 函数就需要对其具有读写权限；因此，你需要添加这个新的注解（已添加换行符）：

```
//+kubebuilder:rbac:
groups=apps,
resources=deployments,
verbs=get;list;watch;create;update;patch;delete
```

### 在集群上部署 Operator

要将 Operator 部署到集群，你需要构建容器镜像并将其部署到容器镜像仓库（例如，`DockerHub`、`Quay.io` 或许多其他仓库）。

第一步是在你偏好的容器镜像仓库中创建一个新的仓库，用于存放 Operator 容器的镜像。假设你在 `quay.io/myid` 下创建了一个名为 `myresource` 的仓库，那么镜像的全名将是 `quay.io/myid/myresource`。

对于每次构建，你需要为镜像使用不同的标签，以便正确更新容器。要在本地构建镜像，你需要运行以下命令（注意标签 `v1alpha1-1`）：

```
$ make docker-build
IMG=quay.io/myid/myresource:v1alpha1-1
```

要将其部署到镜像仓库，必须执行以下命令：

```
$ make docker-push
IMG=quay.io/myid/myresource:v1alpha1-1
```

最后，将 Operator 部署到集群：

```
$ make deploy
IMG=quay.io/myid/myresource:v1alpha1-1
```

这将创建一个新的命名空间 `myresource-kb-system`，其中包含一个将执行 Operator 的新部署。你可以使用以下命令检查 Operator 的日志：

```
$ kubectl logs
deployment/myresource-kb-controller-manager
-n myresource-kb-system
```

### 创建资源的新版本

Kubernetes API 支持版本化的资源，以及一种在不同版本之间转换资源的机制。任何版本之间的转换都不能丢失任何信息。目前，你已使用 `v1alpha1` 版本创建了自定义资源 MyResource。该资源的 `Spec` 包含 `Image` 和 `Memory` 字段。

你想要创建一个 `v1beta1` 版本，修改 `Memory` 字段，并将其重命名为 `MemoryRequest` 以消除歧义。通过将 Memory 值赋值给适当的字段，你将能够在两个版本之间进行转换。

#### 定义新版本

要创建新版本，你需要再次调用 `kubebuilder create api` 命令，使用相同的 group 和 kind，但使用新版本。此外，由于资源已经存在并且 Controller 已经声明，当被问到是否要创建 Resource 和 Controller 时，你可以回答 `n`。

```
$ kubebuilder create api --group mygroup --version v1beta1 --kind MyResource
Create Resource [y/n]
n
Create Controller [y/n]
n
```

此命令创建了一个新目录 `api/v1beta1`，其中包含资源的默认定义。你需要更新这些文件，为新版本声明自定义资源的定义。你可以将文件 `api/v1alpha1/myresource_types.go` 复制到 `api/v1beta1` 并做必要的修改：

- 文件开头的包名，从 `v1alpha1` 改为 `v1beta1`
- Memory 字段的名称，从 `Memory` 改为 `MemoryRequest`，以及 `yaml` 字段名从 `memory` 改为 `memoryRequest`

```
package v1beta1
[...]
// MyResourceSpec 定义了 MyResource 的期望状态
type MyResourceSpec struct {
Image         string            `json:"image"`
MemoryRequest resource.Quantity `json:"memoryRequest"`
}
```

因为现在有多个版本，所以必须声明资源将以哪种格式存储在 `etcd` 中。你可以选择使用 `v1alpha1` 的格式。为此，可以在 `v1alpha1` 包中 `MyResource` 结构体定义前的注释中使用注解 `+kubebuilder:storageversion`。

```
package v1alpha1
[...]
// +kubebuilder:object:root=true
// +kubebuilder:subresource:status
// +kubebuilder:storageversion
// MyResource 是 myresources API 的模式
type MyResource struct {
[...]
}
```



### 实现 Hub 和 Convertible

`controller-runtime` 库提供了一个*转换*系统。该系统依赖两个接口：

- `Hub` 接口：标记用于存储的版本
- `Convertible` 接口：提供与存储版本之间的转换器

```go
type Hub interface {
    runtime.Object
    Hub()
}
type Convertible interface {
    runtime.Object
    ConvertTo(dst Hub) error
    ConvertFrom(src Hub) error
}
```

你已选择 `v1alpha1` 版本作为存储版本，需要让 `v1alpha1.MyResource` 实现 `Hub` 接口：

```go
package v1alpha1
// Hub 将此类型标记为转换中心。
func (*MyResource) Hub() {}
```

所有其他类型，在本例中即 `v1beta1.MyResource`，必须实现 `Convertible` 接口，并提供与 `Hub` 版本进行相互转换的代码：

```go
package v1beta1
import (
    "github.com/myid/myresource/api/v1alpha1"
    "sigs.k8s.io/controller-runtime/pkg/conversion"
)
func (src *MyResource) ConvertTo(dstRaw conversion.Hub) error {
    dst := dstRaw.(*v1alpha1.MyResource)
    dst.Spec.Memory = src.Spec.MemoryRequest
    // 复制其他字段
    dst.ObjectMeta = src.ObjectMeta
    dst.Spec.Image = src.Spec.Image
    dst.Status.State = src.Status.State
    return nil
}
func (dst *MyResource) ConvertFrom(srcRaw conversion.Hub) error {
    src := srcRaw.(*v1alpha1.MyResource)
    dst.Spec.MemoryRequest = src.Spec.Memory
    // 复制其他字段
    dst.ObjectMeta = src.ObjectMeta
    dst.Spec.Image = src.Spec.Image
    dst.Status.State = src.Status.State
    return nil
}
```

### 设置 Webhook

`kubebuilder create webhook` 命令用于设置 webhook。支持以下几种 webhook 类型：

- *转换 Webhook*：用于帮助在资源版本之间进行转换（标志 `--conversion`）
- *变更准入 Webhook*：用于帮助在新对象上设置默认值（标志 `--defaulting`）
- *验证准入 Webhook*：用于帮助验证已创建或更新的对象（标志 `--programmatic-validation`）

在此示例中，你将为一个版本为 `v1beta1`（需要被转换的版本）的 `MyResource` 创建一个*转换* webhook：

```shell
$ kubebuilder create webhook --group mygroup --version v1beta1 --kind MyResource --conversion
```

这将在 `main` 函数中添加代码来设置转换 webhook：

```go
(&mygroupv1beta1.MyResource{}).SetupWebhookWithManager(mgr)
```

然后，在 `v1beta1` 包中创建新代码，供 `main` 调用：

```go
package v1beta1
func (r *MyResource) SetupWebhookWithManager(mgr ctrl.Manager) error {
    return ctrl.NewWebhookManagedBy(mgr).For(r).Complete()
}
```

### 更新 Kustomization 文件

你需要在部署清单中启用 `webhook`，方法是取消 `config/default/kustomization.yaml` 和 `config/crd/kustomization.yaml` 文件中与 `[WEBHOOK]` 和 `[CERTMANAGER]` 相关部分的注释。

如果你没有使用任何其他 webhook（即转换或变更），你还需要注释掉以下行：

- 文件 `config/webhook/kustomization.yaml` 中的 `- manifests.yaml`
- 文件 `config/default/kustomization.yaml` 中的 `- webhookcainjection_patch.yaml`

你还需要安装 cert-manager^(⁵)，以基于 `kubebuilder` 添加到 CRD 的注解生成证书。

### 使用不同版本

按照“在集群上部署 Operator”一节所述部署新版本后，你可以创建版本为 `v1alpha1` 或 `v1beta1` 的 `MyResource` 实例。例如，你可以在 `config/sample` 目录中创建两个示例资源：

- `config/samples/mygroup_v1alpha1_myresource.yaml`：

```yaml
apiVersion: mygroup.myid.dev/v1alpha1
kind: MyResource
metadata:
  name: myresource-sample-alpha
spec:
  image: nginx
  memory: 512Mi
```

- `config/samples/mygroup_v1beta1_myresource.yaml`：

```yaml
apiVersion: mygroup.myid.dev/v1beta1
kind: MyResource
metadata:
  name: myresource-sample-beta
spec:
  image: nginx
  memoryRequest: 256Mi
```

然后，你可以使用 `kubectl` 在集群中创建资源：

```shell
$ kubectl apply -f config/samples/mygroup_v1alpha1_myresource.yaml
$ kubectl apply -f config/samples/mygroup_v1beta1_myresource.yaml
```

最后，当你列出资源时，将获得 `v1beta1` 格式的定义，其中包含 `memoryRequest` 字段：

```yaml
$ kubectl get myresources.mygroup.myid.dev -o yaml
apiVersion: v1
kind: List
items:
- apiVersion: mygroup.myid.dev/v1beta1
  kind: MyResource
  metadata:
    name: myresource-sample-alpha
  spec:
    image: nginx
    memoryRequest: 512Mi
  status:
    state: Ready
- apiVersion: mygroup.myid.dev/v1beta1
  kind: MyResource
  metadata:
    name: myresource-sample-beta
  spec:
    image: nginx
    memoryRequest: 256Mi
  status:
    state: Ready
```

请注意，你得到的是 `v1beta1` 定义，因为 `kubectl` 会返回请求资源的首选版本（最新版本）。

如果你想以 `v1alpha1` 格式获取结果，可以在调用 `kubectl get` 时指定：

```shell
$ kubectl get myresources.v1alpha1.mygroup.myid.dev -o yaml
```

该命令将返回使用 `Memory` 字段的资源。

## 结论

本章使用 Kubebuilder SDK，帮助你比自行编写代码和 Kubernetes 清单更轻松地编写和部署 Operator。

请注意，Operator SDK^(⁶) 是构建 Operator 的另一个框架。Operator SDK 可以管理 Go、Helm 和 Ansible 项目。对于 Go 项目，Operator SDK 使用 `kubebuilder` 来生成代码和 Kubernetes 清单。

Operator SDK 在 `kubebuilder` 之上提供了更多功能，用于将构建的 Operator 与 Operator Lifecycle Manager^(⁷) 和 Operator Hub^(⁸) 集成。

由于这些额外功能超出了本书的范围，因此此处不涉及 Operator SDK。但你可以重新学习本章，将命令 `kubebuilder` 替换为 `operator-sdk`，以使用此框架构建你的 Operator。



脚注
1   2   3   4   5

