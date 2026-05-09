# 使用自定义资源

在上一章中，你已经了解了如何使用 `CustomResourceDefinition` 资源声明一个新的自定义资源，使 Kubernetes API 能够提供服务，以及如何使用 `kubectl` 创建该自定义资源的新实例。但到目前为止，你还没有任何 Go 库可以让你处理自定义资源的实例。

本章将探究在 Go 中处理自定义资源的各种可能性：

- 为自定义资源生成专用的 Clientset 代码。
- 使用 API Machinery 库的 `unstructured` 包和 Client-go 库的 `dynamic` 包。

## 生成 Clientset

代码仓库 [`https://github.com/kubernetes/code-generator`](https://github.com/kubernetes/code-generator) 包含了 Go 代码生成器。第三章中“包的内容”一节对这些生成器进行了非常简要的概述，其中探讨了 API 库的内容。

要使用这些生成器，你首先需要为自定义资源定义的类别编写 Go 结构体。在本例中，你将编写 `MyResource` 和 `MyResourceList` 类别的结构体。

为了与 API 库中的组织方式保持一致，你将把这些类型写在一个 `types.go` 文件中，该文件位于 `pkg/apis/<group>/<version>/` 目录下。

同样，为了与生成器正确配合，你项目的根目录必须位于一个以你的项目的 Go 包命名的目录中。例如，如果项目的包是 `github.com/myid/myproject`（在项目的 `go.mod` 文件的第一行定义），那么项目的根目录必须位于 `github.com/myid/myproject/` 目录下。

举个例子，让我们开始一个新项目。你可以在你选择的目录（通常是包含你所有 Go 项目的目录）中执行以下命令。

```
$ mkdir -p github.com/myid/myresource-crd
$ cd github.com/myid/myresource-crd
$ go mod init github.com/myid/myresource-crd
$ mkdir -p pkg/apis/mygroup.example.com/v1alpha1/
$ cd pkg/apis/mygroup.example.com/v1alpha1/
```

然后，在这个目录中，你可以创建包含这些类别结构体定义的 `types.go` 文件。以下是匹配上一章中 CRD 架构的结构体定义。

```
package v1alpha1
import (
metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
"k8s.io/apimachinery/pkg/util/intstr"
)
type MyResource struct {
metav1.TypeMeta   `json:",inline"`
metav1.ObjectMeta `json:"metadata,omitempty"`
Spec MyResourceSpec `json:"spec"`
}
type MyResourceSpec struct {
Image  string             `json:"image"`
Memory resource.Quantity `json:"memory"`
}
type MyResourceList struct {
metav1.TypeMeta `json:",inline"`
metav1.ListMeta `json:"metadata,omitempty"`
Items []MyResource `json:"items"`
}
```

现在，你需要运行两个生成器：

- `deepcopy-gen` – 这将为每个类别结构体生成一个 `DeepCopyObject()` 方法，这些类型需要实现 `runtime.Object` 接口。
- `client-gen` – 这将为组/版本生成 clientset。

### 使用 `deepcopy-gen`

#### 安装 `deepcopy-gen`

要安装 `deepcopy-gen` 可执行文件，你可以使用 `go install` 命令：

```
go install k8s.io/code-generator/cmd/deepcopy-gen@v0.24.4
```

你可以使用 `@latest` 标签来使用 Kubernetes 代码的最新版本，也可以选择一个特定版本。

#### 添加注解

`deepcopy-gen` 生成器需要注解才能工作。它首先需要在包级别定义 `// +k8s:deepcopy-gen=package` 注解。该注解要求 `deepcopy-gen` 为包中*所有*结构体生成 `deepcopy` 方法。

为此，你可以在 `types.go` 所在的目录中创建一个 `doc.go` 文件，以添加此注解：

```
// pkg/apis/mygroup.example.com/v1alpha1/doc.go
// +k8s:deepcopy-gen=package
package v1alpha1
```

默认情况下，`deepcopy-gen` 将生成 `DeepCopy()` 和 `DeepCopyInto()` 方法，但不会生成 `DeepCopyObject()`。为此，你需要在每个类别结构体之前添加另一个注解（`// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object`）。

```
// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object
type MyResource struct {
[...]
// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object
type MyResourceList struct {
[...]
```

#### 运行 `deepcopy-gen`

生成器需要一个包含文本（通常是许可证）的文件，该文本将被添加到生成文件的开头。为此，你可以创建一个名为 `hack/boilerplate.go.txt` 的空文件（或者包含你偏好的内容，别忘了文本应该是 Go 注释）。

你需要运行 `go mod tidy` 以使生成器正常工作（或者如果你更喜欢将 Go 依赖项进行 vendor，则运行 `go mod vendor`）。最后，你可以运行 `deepcopy-gen` 命令，它将生成一个文件 `pkg/apis/mygroup.example.com/v1alpha1/zz_generated.deepcopy.go`：

```
$ go mod tidy
$ deepcopy-gen --input-dirs github.com/myid/myresource-crd/pkg/apis/mygroup.example.com/v1alpha1
-O zz_generated.deepcopy
--output-base ../../..
--go-header-file ./hack/boilerplate.go.txt
```

注意 `"../../.."` 作为输出基础目录。它会将输出基础目录放在你创建项目目录的目录中：

`$ mkdir -p github.com/myid/myresource-crd`

如果与你创建的目录数量不同（三个之外），你需要根据你创建的子目录数量进行调整。

此时，你的项目应具有以下文件结构：

```
├── go.mod
├── hack
│   └── boilerplate.go.txt
├── pkg
│   └── apis
│       └── mygroup.example.com
│           └── v1alpha1
│               ├── doc.go
│               ├── types.go
│               └── zz_generated.deepcopy.go
```

### 使用 `client-gen`

#### 安装 `client-go`

要安装 `client-gen` 可执行文件，你可以使用 `go install` 命令：

```
go install k8s.io/code-generator/cmd/client-gen@v0.24.4
```

你可以使用 `@latest` 标签来使用 Kubernetes 代码的最新版本，也可以选择一个特定版本，如果你想以可重复的方式运行该命令。

#### 添加注解

你需要向 `types.go` 文件中定义的结构体添加注解，以指示要为哪些类型定义 Clientset。要使用的注解是 `// +genclient`。

- `// +genclient`（不带选项）将要求 `client-gen` 为命名空间的资源生成 Clientset。
- `// +genclient:nonNamespaced` 将为非命名空间的资源生成 Clientset。
- `+genclient:onlyVerbs=create,get` 将仅生成这些动词，而不是默认生成所有动词。
- `+genclient:skipVerbs=watch` 将生成除这些动词之外的所有动词，而不是默认生成所有动词。
- `+genclient:noStatus` – 如果带注解的结构体中存在 `Status` 字段，则会生成一个 `updateStatus` 函数。使用此选项，你可以禁用到 `updateStatus` 函数的生成（请注意，如果 `Status` 字段不存在，则不需要此选项）。

你正在创建的自定义资源是命名空间的，因此你可以使用不带选项的注解：

```
// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object
// +genclient
type MyResource struct {
[...]
```


好的，作为一名高级文档工程师和翻译员，我将严格按照您提供的注意事项和示例，将给定的英文文本翻译成中文。


### 添加 `AddToScheme` 函数

生成的代码依赖于资源包中定义的 `AddToScheme` 函数。为了遵循 API 库中的约定，您需要将此函数写入一个 `register.go` 文件中，该文件位于目录 `pkg/apis/<group>/<version>/` 下。

通过将 Kubernetes API 库中本地 Kubernetes 资源的 `register.go` 文件作为样板，您将获得如下所示文件。唯一的改动是组名（❶）、版本名（❷）以及要注册到 scheme 中的资源列表（❸）。

```
package v1alpha1
import (
metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
"k8s.io/apimachinery/pkg/runtime"
"k8s.io/apimachinery/pkg/runtime/schema"
)
const GroupName = "mygroup.example.com"          ❶
var SchemeGroupVersion = schema.GroupVersion{
Group: GroupName,
Version: "v1alpha1",                         ❷
}
var (
SchemeBuilder      = runtime.NewSchemeBuilder(addKnownTypes)
localSchemeBuilder = &SchemeBuilder
AddToScheme        = localSchemeBuilder.AddToScheme
)
func addKnownTypes(scheme *runtime.Scheme) error {
scheme.AddKnownTypes(SchemeGroupVersion,
&MyResource{},                         ❸
&MyResourceList{},
)
metav1.AddToGroupVersion(scheme, SchemeGroupVersion)
return nil
}
```

### 运行 `client-gen`

`client-gen` 需要一个文件，其中包含要添加到生成文件开头的文本（通常是许可证）。您将使用与 `deepcopy-gen` 相同的文件：`hack/boilerplate.go.txt`。

您可以运行 `client-gen` 命令，该命令会在 `pkg/clientset/clientset` 目录下生成文件：

```
client-gen \
--clientset-name clientset
--input-base ""
--input github.com/myid/myresource-crd/pkg/apis/mygroup.example.com/v1alpha1
--output-package github.com/myid/myresource-crd/pkg/clientset
--output-base ../../..
--go-header-file hack/boilerplate.go.txt
```

请注意 `"../../.."` 作为输出基目录。它会将输出基目录放在您为项目创建目录的目录下：

`$ mkdir -p github.com/myid/myresource-crd`

如果您创建的目录层级与此处的三个子目录不同，则需要对此进行调整。

请注意，当您更新自定义资源的定义时，需要再次运行此命令。建议将此命令放在 `Makefile` 中，以便每次修改定义自定义资源的文件时自动运行它。

### 使用生成的客户端集

现在，客户端集已经生成，并且类型实现了 `runtime.Object` 接口，您可以像使用本地 Kubernetes 资源一样使用自定义资源。例如，以下代码将使用专用的客户端集列出 `default` 命名空间上的自定义资源：

```
import (
"context"
"fmt"
"github.com/myid/myresource-crd/pkg/clientset/clientset"
metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
"k8s.io/client-go/tools/clientcmd"
)
config, err :=
clientcmd.NewNonInteractiveDeferredLoadingClientConfig(
clientcmd.NewDefaultClientConfigLoadingRules(),
nil,
).ClientConfig()
if err != nil {
return err
}
clientset, err := clientset.NewForConfig(config)
if err != nil {
return err
}
list, err := clientset.MygroupV1alpha1().
MyResources("default").
List(context.Background(), metav1.ListOptions{})
if err != nil {
return err
}
for _, res := range list.Items {
fmt.Printf("%s\n", res.GetName())
}
```

### 使用生成的 *fake* 客户端集

`client-gen` 工具也会生成一个 *fake* 客户端集，您可以像使用 Client-go 库中的 *fake* 客户端集一样使用它。更多信息，请参见第 7 章的“Fake 客户端集”一节。

## 使用 `Unstructured` 包和动态客户端

`Unstructured` 和 `UnstructuredList` 类型定义在 API Machinery 库的 `unstructured` 包中。要使用的 `import` 如下：

```
import (
"k8s.io/apimachinery/pkg/apis/meta/v1/unstructured"
)
```

这些类型可用于表示任何 Kubernetes *种类*，无论是列表还是非列表。

### `Unstructured` 类型

`Unstructured` 类型被定义为一个包含唯一 `Object` 字段的结构体：

```
type Unstructured struct {
// Object 是一个与 JSON 兼容的映射，
// 其子元素可以是 string、float、int、bool、[]interface{}
// 或 map[string]interface{}。
Object map[string]interface{}
}
```

使用此类型，可以定义任何 Kubernetes 资源，而无需使用类型化的结构体（例如，在 API 库中定义的 `Pod` 结构体）。

为此类型定义了**获取方法**和**设置方法**，用于访问所有表示 Kubernetes *种类* 的结构体所共有的 `TypeMeta` 和 `ObjectMeta` 字段中的通用字段。

#### 访问 `TypeMeta` 字段的获取方法和设置方法

`APIVersion` 和 `Kind` 的获取方法/设置方法可用于直接获取和设置 `TypeMeta` 的 `apiVersion` 和 `Kind` 字段。

`GroupVersionKind` 的获取方法/设置方法可用于将对象中指定的 `apiVersion` 和 kind 与 `GroupVersionKind` 值进行相互转换。

```
GetAPIVersion() string
GetKind() string
GroupVersionKind() schema.GroupVersionKind
SetAPIVersion(version string)
SetKind(kind string)
SetGroupVersionKind(gvk schema.GroupVersionKind)
```

#### 访问 `ObjectMeta` 字段的获取方法和设置方法

为 `ObjectMeta` 结构体的所有字段定义了获取方法和设置方法。该结构体的详细信息可在第 3 章的“`ObjectMeta` 字段”一节中找到。

例如，访问 `Name` 字段的获取方法和设置方法是 `GetName() string` 和 `SetName(name string)`。

#### 用于创建和转换的方法

- `NewEmptyInstance() runtime.Unstructured` – 返回一个新的实例，该实例仅从接收者复制了 `apiVersion` 和 `kind` 字段。
- `MarshalJSON() ([]byte, error)` – 返回接收者的 JSON 表示形式。
- `UnmarshalJSON(b []byte) error` – 使用传入的 JSON 表示形式填充接收者。
- `UnstructuredContent() map[string]interface{}` – 返回接收者的 `Object` 字段的值。
- `SetUnstructuredContent(content map[string]interface{})` – 设置接收者的 `Object` 字段的值。
- `IsList() bool` – 通过检查是否存在 `items` 字段且其为数组，返回 true 如果接收者描述的是一个列表。
- `ToList() (*UnstructuredList, error)` – 将接收者转换为 `UnstructuredList`。



### 访问非元字段的辅助函数

以下辅助函数可用于获取和设置 `Unstructured` 实例中 `Object` 字段特定字段的值。

请注意，这些辅助函数是函数，而不是 `Unstructured` 类型的方法。

它们都接受：

- 第一个参数 `obj`，类型为 `map[string]interface{}`，用于传递 `Unstructured` 实例的 `Object` 字段，
- 最后一个参数 `fields`，类型为 `...string`，用于传递导航到对象内各层级所需的键。请注意，不支持数组/切片语法。

`Setter` 函数接受第二个参数，用于设置给定对象中特定字段的值。`Getter` 函数返回三个值：

- 所请求字段的值（如果可能）
- 一个布尔值，指示是否找到了所请求的字段
- 一个错误，如果找到了该字段但其类型与请求的类型不符

辅助函数的名称如下：

- `RemoveNestedField` – 用于移除所请求的字段
- `NestedFieldCopy`, `NestedFieldNoCopy` – 返回所请求字段的副本或原始值
- `NestedBool`, `NestedFloat64`, `NestedInt64`, `NestedString`, `SetNestedField` – 用于获取和设置 bool / float64 / int64 / string 类型的字段
- `NestedMap`, `SetNestedMap` – 用于获取和设置 `map[string]interface{}` 类型的字段
- `NestedSlice`, `SetNestedSlice` – 用于获取和设置 `[]interface{}` 类型的字段
- `NestedStringMap`, `SetNestedStringMap` – 用于获取和设置 `map[string]string` 类型的字段
- `NestedStringSlice`, `SetNestedStringSlice` – 用于获取和设置 `[]string` 类型的字段

#### 示例

作为示例，以下是一些定义 `MyResource` 实例的代码：

```go
import (
    myresourcev1alpha1 "github.com/myid/myresource-crd/pkg/apis/mygroup.example.com/v1alpha1"
    "k8s.io/apimachinery/pkg/apis/meta/v1/unstructured"
)

func getResource() (*unstructured.Unstructured, error) {
    myres := unstructured.Unstructured{}
    myres.SetGroupVersionKind(
        myresourcev1alpha1.SchemeGroupVersion.
            WithKind("MyResource"))
    myres.SetName("myres1")
    myres.SetNamespace("default")
    err := unstructured.SetNestedField(
        myres.Object,
        "nginx",
        "spec", "image",
    )
    if err != nil {
        return err
    }
    // 使用 int64
    err = unstructured.SetNestedField(
        myres.Object,
        int64(1024*1024*1024),
        "spec", "memory",
    )
    if err != nil {
        return err
    }
    // 或使用 string
    err = unstructured.SetNestedField(
        myres.Object,
        "1024Mo",
        "spec", "memory",
    )
    if err != nil {
        return err
    }
    return &myres, nil
}
```

### `UnstructuredList` 类型

`UnstructuredList` 类型被定义为一个结构体，包含一个 `Object` 字段和一个由 `Unstructured` 实例组成的切片作为列表项：

```go
type UnstructuredList struct {
    Object map[string]interface{}
    // Items 是一个非结构化对象列表。
    Items []Unstructured
}
```

#### 访问 `TypeMeta` 字段的 Getter 和 Setter

可以使用 `APIVersion` 和 `Kind` 的 Getter/Setter 来直接获取和设置 `TypeMeta` 的 `apiVersion` 和 `kind` 字段。

可以使用 `GroupVersionKind` 的 Getter/Setter 来将对象中指定的 `apiVersion` 和 `kind` 与 `GroupVersionKind` 值相互转换。

```
GetAPIVersion() string
GetKind() string
GroupVersionKind() schema.GroupVersionKind
SetAPIVersion(version string)
SetKind(kind string)
SetGroupVersionKind(gvk schema.GroupVersionKind)
```

#### 访问 `ListMeta` 字段的 Getter 和 Setter

这些 getter 和 setter 用于获取和设置与 `List` 操作结果相关的值，这些值位于列表的 `ListMeta` 字段中。

```
GetResourceVersion() string
GetContinue() string
GetRemainingItemCount() *int64
SetResourceVersion(version string)
SetContinue(c string)
SetRemainingItemCount(c *int64)
```

#### 用于创建和转换的方法

- `NewEmptyInstance() runtime.Unstructured` – 创建一个新的非结构化对象实例，使用从列表接收器复制的 `apiVersion` 和 `kind`。
- `MarshalJSON() ([]byte, error)` – 返回接收器的 JSON 表示形式。
- `UnmarshalJSON(b []byte) error` – 用传入的 JSON 表示形式填充接收器。
- `UnstructuredContent() map[string]interface{}`, `SetUnstructuredContent(content map[string]interface{})` – 获取接收器的 `Object` 字段的值。
- `EachListItem(fn func(runtime.Object) error) error` – 对列表中的每个列表项执行 `fn` 函数。

### 在类型化对象与非结构化对象之间转换

API Machinery 库的 `runtime` 包提供了在类型化对象和非结构化对象之间进行转换的工具，这些对象可以是资源或列表。

- ❶ 获取转换器
- ❷ 将一个非结构化对象（定义一个 Pod）转换为类型化的 Pod
- ❸ 将一个类型化的 Pod 转换为非结构化对象

```go
import (
    "k8s.io/apimachinery/pkg/runtime"
)

converter := runtime.DefaultUnstructuredConverter          ❶
var pod corev1.Pod
converter.FromUnstructured(                                   ❷
    u.UnstructuredContent(), &pod,
)
var u unstructured.Unstructured
u.Object = converter.ToUnstructured(&pod)                    ❸
```

### 动态客户端

正如你在 第 6 章 中所见，Client-go 提供了能够与 Kubernetes API 交互的客户端：用于访问类型化资源的 Clientset，用于向 API 进行底层 REST 调用的 REST 客户端，以及用于获取 API 提供的资源信息的 Discovery 客户端。

它还提供了另一个客户端，即 `dynamic` 客户端，用于处理通过 `Unstructured` 类型描述的非类型化资源。

#### 获取动态客户端

`dynamic` 包提供了用于创建类型为 `dynamic.Interface` 的动态客户端的函数。

- `func NewForConfig(c *rest.Config) (Interface, error)` – 此函数返回一个动态客户端，使用通过第 6 章“连接到集群”一节中所述方法之一构建的 `rest.Config`。
- `func NewForConfigOrDie(c *rest.Config) Interface` – 此函数与前一个类似，但在发生错误时会触发 panic，而不是返回错误。此函数可用于硬编码配置，你希望断言其有效性。
- `NewForConfigAndClient(c *rest.Config, httpClient *http.Client) (Interface, error)` – 此函数返回一个 `dynamic` 客户端，使用提供的 `rest.Config` 和提供的 `httpClient`。前面的函数 `NewForConfig` 使用通过函数 `rest.HTTPClientFor` 构建的默认 HTTP 客户端。如果你想在构建 `dynamic` 客户端之前自定义 HTTP 客户端，可以使用此函数。



#### 使用动态客户端

`dynamic` 客户端实现了 `dynamic.Interface`，其定义如下：

```
type Interface interface {
Resource(resource schema.GroupVersionResource)
NamespaceableResourceInterface
}
```

`dynamic` 客户端的唯一直接方法是 `Resource(gvr)`，它返回一个实现了 `NamespaceableResourceInterface` 的对象。

```
type NamespaceableResourceInterface interface {
Namespace(string) ResourceInterface
ResourceInterface
}
```

可以将对 `Resource(gvr)` 的调用与对 `Namespace(ns)` 方法的调用进行链式调用，返回一个实现了 `ResourceInterface` 的对象。或者，如果 `Resource(gvr)` 没有与 `Namespace(ns)` 方法进行链式调用，它同样实现了 `ResourceInterface`。

因此，在调用 `Resource(gvr)` 后，你可以根据情况选择：如果 `gvr` 描述的资源是一个命名空间资源，并且你想定义在哪个命名空间上操作，则可以链式调用 `Namespace(ns)`；否则，如果你希望进行集群范围的操作，可以省略这个调用（针对非命名空间资源）。

`ResourceInterface` 的定义如下（为简洁起见，省略了函数的完整签名）：

```
type ResourceInterface interface {
Create(...)
Update(...)
UpdateStatus(...)
Delete(...)
DeleteCollection(...)
Get(...)
List(...)
Watch(...)
Patch(...)
Apply(...)          // 从 v1.25 版本开始
ApplyStatus(...)     // 从 v1.25 版本开始
}
```

这些方法适用于 `Unstructured` 和 `UnstructuredList` 类型。例如，`Create` 方法接受一个 `Unstructured` 对象作为输入，并返回创建后的 `Unstructured` 对象；而 `List` 方法则返回一个 `UnstructuredList` 形式的对象列表：

```
Create(
ctx context.Context,
obj *unstructured.Unstructured,
options metav1.CreateOptions,
subresources ...string,
) (*unstructured.Unstructured, error)
List(
ctx context.Context,
opts metav1.ListOptions,
) (*unstructured.UnstructuredList, error)
```

如果将这些方法的签名与 Client-go 客户端集提供的方法进行比较，你会发现它们非常相似。以下是一些不同之处：

*   客户端集中使用的类型化对象（例如 `corev1.Pod`）在 `dynamic` 客户端中被 `Unstructured` 所取代。
*   动态方法中包含了一个 `subresource…` 参数，而客户端集则为子资源提供了特定的方法。

关于各种操作行为的更多信息，你可以参考第 6 章。

#### 示例

以下是一个在集群中创建 `MyResource` 实例的示例代码：

```
import (
"context"
myresourcev1alpha1 "github.com/feloy/myresource-crd/pkg/apis/mygroup.example.com/v1alpha1"
metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
"k8s.io/client-go/dynamic"
)
func CreateMyResource(
dynamicClient dynamic.Interface,
u *unstructured.Unstructured,
) (*unstructured.Unstructured, error) {
gvr := myresourcev1alpha1.
SchemeGroupVersion.
WithResource("myresources")
return dynamicClient.
Resource(gvr).
Namespace("default").
Create(
context.Background(),
u,
metav1.CreateOptions{},
)
}
```

#### *模拟* 动态客户端

正如你在第 7 章所见，Client-go 库为客户端集、发现客户端和 REST 客户端提供了 *模拟* 实现。该库还为 `dynamic` 客户端提供了一个 *模拟* 实现。

与其他 *模拟* 实现类似，你可以使用 `PrependReactor` 及 `dynamicClient.Fake` 对象上的类似方法来注册 reactor，并可以在测试执行后检查 `dynamicClient.Fake` 对象的 `Actions`。

函数 `fake.NewSimpleDynamicClient` 用于创建一个新的 *模拟* `dynamic` 客户端。

```
func NewSimpleDynamicClient(
scheme *runtime.Scheme,
objects ...runtime.Object,
) *FakeDynamicClient)
```

`objects` 参数表示在创建模拟集群时需要初始化的资源。以下是一个针对使用 `dynamic` 客户端的函数的测试示例。

`NewSimpleDynamicClient` 函数被调用时，需要传入用于创建初始资源的参数。请注意，初始资源的预期类型是 `runtime.Object`，这是由 `Unstructured` 实现的一个接口。

```
func TestCreateMyResourceWhenResourceExists(t *testing.T) {
myres, err := getResource()
if err != nil {
t.Error(err)
}
dynamicClient := fake.NewSimpleDynamicClient(
runtime.NewScheme(),
myres,
)
// 此处并未实际使用，仅作展示用途
dynamicClient.Fake.PrependReactor(
"create",
"myresources",
func(
action ktesting.Action,
) (handled bool, ret runtime.Object, err error) {
return false, nil, nil
})
_, err = CreateMyResource(dynamicClient, myres)
if err == nil {
t.Error("应该出现错误")
}
actions := dynamicClient.Fake.Actions()
if len(actions) != 1 {
t.Errorf("操作数量应为 %d，实际为 %d", 1, len(actions))
}
}
```

## 总结

本章探讨了在 Go 中处理自定义资源的不同解决方案。

第一种方案是基于自定义资源的定义生成 Go 代码，以生成专用于该特定自定义资源定义的客户端集，从而让你能够像处理原生 Kubernetes 资源一样处理自定义资源实例。

另一种方案是使用 Client-go 库中的 `dynamic` 客户端，并依赖 `Unstructured` 类型来定义自定义资源。



