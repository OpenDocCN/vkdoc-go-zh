# 3. 在 Go 中使用 API 资源

本书前两章描述了 Kubernetes API 是如何设计的，以及如何使用 HTTP 请求访问它。具体来说，你已经了解到 API 管理的资源被组织成 `Group-Version-Resource`，并且客户端和 API 服务器之间交换的对象由 Kubernetes API 定义为 `Kind`。该章还展示了这些数据在交换过程中可以编码为 JSON、YAML 或 Protobuf 格式，具体取决于客户端设置的 HTTP 头。

在接下来的章节中，你将了解如何使用 Go 语言访问此 API。使用 Kubernetes API 所需的两个重要 Go 库是 [apimachinery](https://github.com/kubernetes/apimachinery) 和 [api](https://github.com/kubernetes/api)。

API Machinery 是一个通用库，负责在 Go 结构体与以 JSON（或 YAML 或 Protobuf）格式编写的对象之间进行数据序列化。这使得客户端以及 API 服务器的开发者能够使用 Go 结构体编写数据，并在 HTTP 交换过程中透明地以 JSON（或 YAML 或 Protobuf）格式使用这些资源。

API Machinery 库是通用的，因为它不包含任何 Kubernetes API 资源定义。它使得 Kubernetes API 可扩展，并使 API Machinery 可用于任何其他使用相同机制（即 `Kind` 和 `Group-Version-Resource`）的 API。

而 API 库则是一个 Go 结构体的集合，用于在 Go 中处理 Kubernetes API 定义的资源。


## API 库源码与导入

API 库的源码可通过 [`https://github.com/kubernetes/api`](https://github.com/kubernetes/api) 获取。如果你想为该库做贡献，请注意源码并非由此仓库管理，而是由中央仓库 [`https://github.com/kubernetes/kubernetes`](https://github.com/kubernetes/kubernetes) 中的 `staging/src/k8s.io/api` 目录管理，并且源码会从 `kubernetes` 仓库同步到 `api` 仓库。

若要将 API 库的包导入到 Go 源码中，你需要使用 `k8s.io/api` 前缀——例如：

```
import "k8s.io/api/core/v1"
```

API 库中的包遵循 API 的 **Group-Version-Resource** 结构。当你需要使用某个资源的结构体时，需要按照以下模式导入与该资源 *组* 和 *版本* 相关的包：

```
import "k8s.io/api/<group>/<version>"
```

## 包的内容

我们以 `k8s.io/api/apps/v1` 包为例，来检视其包含的文件。

### types.go

该文件可以被视为包的主文件，因为它定义了所有 *Kind* 结构体及其他相关的子结构体。它还定义了在这些结构体中发现的所有枚举字段的类型和常量。例如，考虑 `Deployment` *Kind*；`Deployment` 结构体首先定义如下：

```
type Deployment struct {
    metav1.TypeMeta
    metav1.ObjectMeta
    Spec      DeploymentSpec
    Status    DeploymentStatus
}
```

然后，相关的子结构体 `DeploymentSpec` 和 `DeploymentStatus` 使用以下方式定义：

```
type DeploymentSpec struct {
    Replicas      *int32
    Selector      *metav1.LabelSelector
    Template      v1.PodTemplateSpec
    Strategy      DeploymentStrategy
    [...]
}
type DeploymentStatus struct {
    ObservedGeneration      int64
    Replicas                int32
    [...]
    Conditions              []DeploymentCondition
}
```

然后，对前一个结构体中用作类型的每个结构体都按照相同的方式继续定义。

在 `DeploymentCondition` 结构体（此处未展示）中使用的 `DeploymentConditionType` 类型，以及该枚举的可能值，定义如下：

```
type DeploymentConditionType string
const (
    DeploymentAvailable   DeploymentConditionType = "Available"
    DeploymentProgressing DeploymentConditionType = "Progressing"
    DeploymentReplicaFailure DeploymentConditionType = "ReplicaFailure"
)
```

你可以看到每个 *Kind* 都嵌入了两个结构体：`metav1.TypeMeta` 和 `metav1.ObjectMeta`。这是为了让它们能被 API Machinery 识别所必需的。`TypeMeta` 结构体包含该 *Kind* 的 GVK 信息，而 `ObjectMeta` 则包含该 *Kind* 的元数据，例如其名称。

### register.go

该文件定义了与此包相关的组和版本，以及该组和版本中的 *Kind* 列表。当你需要指定来自该组-版本的资源的组和版本时，可以使用公共变量 `SchemeGroupVersion`。

它还声明了一个函数 `AddToScheme`，可用于将组、版本和 *Kind* 添加到一个 *Scheme* 中。*Scheme* 是 API Machinery 中使用的一种抽象，用于在 Go 结构体和 **Group-Version-Kinds** 之间创建映射关系。这将在第 5 章 "API Machinery" 中进一步讨论。

### doc.go

该文件及后续文件包含一些高级信息，你在着手用 Go 编写第一个 Kubernetes 资源时并不需要理解它们，但它们将帮助你理解如何在接下来的章节中使用自定义资源定义声明新的资源。

`doc.go` 文件包含以下用于生成文件的指令：

```
// +k8s:deepcopy-gen=package
// +k8s:protobuf-gen=package
// +k8s:openapi-gen=true
```

第一条指令由 `deepcopy-gen` 生成器用于生成 `zz_generated.deepcopy.go` 文件。第二条指令由 `go-to-protobuf` 生成器用于生成以下文件：`generated.pb.go` 和 `generated.proto`。第三条指令由 `genswaggertypedocs` 生成器用于生成 `types_swagger_doc_generated.go` 文件。

### generated.pb.go 和 generated.proto

这些文件由 `go-to-protobuf` 生成器生成。API Machinery 在将数据序列化为 Protobuf 格式或从 Protobuf 格式反序列化时使用它们。

### types_swagger_doc_generated.go

该文件由 `genswaggertypedocs` 生成器生成。它在生成 Kubernetes API 的完整 `swagger` 定义时使用。

### zz_generated.deepcopy.go

该文件由 `deepcopy-gen` 生成器生成。它包含为包中定义的每个类型生成的 `DeepCopyObject` 方法的定义。此方法对于结构体实现 `runtime.Object` 接口是必要的，该接口定义在 API Machinery 库中，并且 API Machinery 期望所有 *Kind* 结构体都实现这个 `runtime.Object` 接口。

该接口在[此文件](https://github.com/kubernetes/apimachinery/blob/master/pkg/runtime/interfaces.go)中定义如下：

```
type Object interface {
    GetObjectKind() schema.ObjectKind
    DeepCopyObject() Object
}
```

另一个必要的方法 `GetObjectKind` 会自动添加到嵌入了 `TypeMeta` 结构体的结构体中——所有 *Kind* 结构体都是这种情况。`TypeMeta` 结构体具有定义如下的方法：

```
func (obj *TypeMeta) GetObjectKind() schema.ObjectKind {
    return obj
}
```

## core/v1 中的特定内容

`core/v1` 包除了定义 Core 资源的结构体外，还为特定类型定义了实用方法，这些方法在将这些类型整合到你的代码中时会很有用。

### ObjectReference

`ObjectReference` 可用于以唯一方式引用任何对象。该结构体定义如下：

```
type ObjectReference struct {
    APIVersion          string
    Kind                string
    Namespace           string
    Name                string
    UID                 types.UID
    ResourceVersion     string
    FieldPath           string
}
```

为此类型定义了三个方法：

*   `SetGroupVersionKind(gvk schema.GroupVersionKind)` – 该方法将根据作为参数传递的 `GroupVersionKind` 值来设置 `APIVersion` 和 `Kind` 字段。

*   `GroupVersionKind() schema.GroupVersionKind` – 该方法将基于 `ObjectReference` 的 `APIVersion` 和 `Kind` 字段返回一个 `GroupVersionKind` 值。

*   `GetObjectKind() schema.ObjectKind` – 该方法将 `ObjectReference` 对象转换为 `ObjectKind`。前面两个方法实现了这个 `ObjectKind` 接口。由于 `ObjectReference` 上的 `DeepCopyObject` 方法也已定义，因此 `ObjectReference` 将满足 `runtime.Object` 接口。


### ResourceList

`ResourceList`类型被定义为一个映射（map），其键为`ResourceName`，值为`Quantity`。该类型在各类 Kubernetes 资源中用于定义资源的`limits`和`requests`。

在 YAML 中，其用法示例如下（为容器定义资源`requests`和`limits`）：

```
apiVersion: v1
kind: Pod
metadata:
name: mypod
spec:
containers:
- name: runtime
resources:
requests:
memory: "64Mi"
cpu: "250m"
limits:
memory: "128Mi"
cpu: "500m"
```

在 Go 语言中，可以如下编写`requests`部分：

```
requests := corev1.ResourceList{
corev1.ResourceMemory:
*resource.NewQuantity(64*1024*1024, resource.BinarySI),
corev1.ResourceCPU:
*resource.NewMilliQuantity(250, resource.DecimalSI),
}
```

下一章将更详细地描述如何使用`resource.Quantity`类型定义资源量。`ResourceList`类型提供以下方法：

*   `Cpu() *resource.Quantity` – 返回映射中 CPU 键对应的资源量，格式为十进制（如 1, 250 m 等）
*   `Memory() *resource.Quantity` – 返回映射中 Memory 键对应的资源量，格式为二进制（如 512 Ki, 64 Mi 等）
*   `Storage() *resource.Quantity` – 返回映射中 Storage 键对应的资源量，格式为二进制（如 512 Mi, 1 Gi 等）
*   `Pods() *resource.Quantity` – 返回映射中 Pods 键对应的资源量，格式为十进制（如 1, 10 等）
*   `StorageEphemeral() *resource.Quantity` – 返回映射中 StorageEphemeral 键对应的资源量，格式为二进制（如 512 Mi, 1 Gi 等）

对于以上每个方法，如果映射中未定义该键，则返回一个值为`Zero`的`Quantity`。

还有一个内部使用的辅助方法，可用于获取非标准格式的资源量：

*   `Name(name ResourceName, defaultFormat resource.Format) *resource.Quantity` – 返回指定键`Name`在`defaultFormat`格式下的资源量。

`ResourceName`类型的枚举值包括：`ResourceCPU`、`ResourceMemory`、`ResourceStorage`、`ResourceEphemeralStorage`和`ResourcePods`。

### Taint

`Taint`资源用于应用于节点（Node），确保不容忍这些污点的 Pod 不会被调度到这些节点上。`Taint`结构体定义如下：

```
type Taint struct {
Key            string
Value          string
Effect         TaintEffect
TimeAdded      *metav1.Time
}
```

`TaintEffect`枚举可取以下值：

```
TaintEffectNoSchedule          = "NoSchedule"
TaintEffectPreferNoSchedule    = "PreferNoSchedule"
TaintEffectNoExecute           = "NoExecute"
```

控制平面在特殊条件下使用的已知`Taint`键，在此包中定义如下：

```
TaintNodeNotReady
= "node.kubernetes.io/not-ready"
TaintNodeUnreachable
= "node.kubernetes.io/unreachable"
TaintNodeUnschedulable
= "node.kubernetes.io/unschedulable"
TaintNodeMemoryPressure
= "node.kubernetes.io/memory-pressure"
TaintNodeDiskPressure
= "node.kubernetes.io/disk-pressure"
TaintNodeNetworkUnavailable
= "node.kubernetes.io/network-unavailable"
TaintNodePIDPressure
= "node.kubernetes.io/pid-pressure"
TaintNodeOutOfService
= "node.kubernetes.io/out-of-service"
```

`Taint`结构体上定义了以下两个方法：

*   `MatchTaint(taintToMatch *Taint) bool` – 如果两个污点的`key`和`effect`值相同，则此方法返回`true`。
*   `ToString() string` – 此方法返回`Taint`的字符串表示，格式为：`<key>=<value>:<effect>`、`<key>=<value>:`、`<key>:<effect>`或`<key>`。

### Toleration

`Toleration`资源用于应用于 Pod，使其能够容忍特定节点上的污点。`Toleration`结构体定义如下：

```
type Toleration struct {
Key                  string
Operator             TolerationOperator
Value                string
Effect               TaintEffect
TolerationSeconds    *int64
}
```

`TolerationOperator`枚举可取以下值：

```
TolerationOpExists    = "Exists"
TolerationOpEqual     = "Equal"
```

`TaintEffect`枚举可取以下值：

```
TaintEffectNoSchedule              = "NoSchedule"
TaintEffectPreferNoSchedule        = "PreferNoSchedule"
TaintEffectNoExecute               = "NoExecute"
```

`Toleration`结构体上定义了以下两个方法：

*   `MatchToleration(tolerationToMatch *Toleration) bool` – 如果两个容忍度的`Key`、`Operator`、`Value`和`Effect`值分别相同，则此方法返回`true`。
*   `ToleratesTaint(taint *Taint) bool` – 如果该容忍度能够容忍指定的`Taint`，则此方法返回`true`。容忍度容忍污点的规则如下：
    *   **Effect**：如果`Toleration`的 effect 为空，则匹配所有 Taint 的 effect；否则，Toleration 和 Taint 的 effect 必须匹配。
    *   **Operator**：如果`TolerationOperator`为`Exists`，则匹配所有 Taint 的值；否则，`TolerationOperator`为`Equal`，且 Toleration 和 Taint 的值必须匹配。
    *   **Key**：如果 Toleration 的 Key 为空，则`TolerationOperator`必须为`Exists`，且匹配所有 Taint 的 key（无论值是什么）；否则，Toleration 和 Taint 的 key 必须匹配。

### Well-Known Labels

控制平面会为节点添加标签；有关使用的已知键及其用法，可在`core/v1`包的`well_known_labels.go`文件中找到。以下是最广为人知的标签。

节点上运行的 kubelet 会填充以下标签：

```
LabelOSStable      = "kubernetes.io/os"
LabelArchStable    = "kubernetes.io/arch"
LabelHostname      = "kubernetes.io/hostname"
```

当节点运行在云提供商上时，可设置以下标签，用于表示（虚拟）机器的实例类型、可用区（zone）和区域（region）：

```
LabelInstanceTypeStable
= "node.kubernetes.io/instance-type"
LabelTopologyZone
= "topology.kubernetes.io/zone"
LabelTopologyRegion
= "topology.kubernetes.io/region"
```

## Writing Kubernetes Resources in Go

你将需要用 Go 语言编写 Kubernetes 资源，以便通过 HTTP 请求或更常用的 client-go 库在集群中创建或更新资源。client-go 库将在后续章节中讨论；但现在，让我们专注于用 Go 语言编写资源。

要创建或更新资源，需要为与该资源关联的`Kind`创建相应的结构体。例如，要创建一个`deployment`，你需要创建`Deployment`类型；为此，需要初始化一个在 API 库的`apps/v1`包中定义的`Deployment`结构体。



### 导入包

在使用结构体之前，需要先导入定义该结构体的包。如本章开头所述，包名的模式为 `k8s.io/api/<group>/<version>`。路径的最后一部分是版本号，但不要将其与 Go 模块的版本号混淆。

区别在于：当你导入 Go 模块的特定版本（例如 `k8s.io/klog/v2`）时，会使用版本号前的部分作为前缀来访问包的符号，无需定义任何别名。原因在于，在该库中，`v2` 目录并不存在，而是代表一个分支名，并且该包内的文件都以 `package klog` 开头，而不是 `package v2`。

相反，当使用 Kubernetes API 库时，版本号是库中一个包的真实名称，且该包内的文件确实以 `package v1` 开头。

如果不为导入定义别名，则必须使用版本名称来使用该包的符号。但仅凭版本号在阅读代码时缺乏意义；如果从同一文件导入多个包，最终会出现多个 `v1` 包名，这是不可能的。

惯例是使用组名定义别名；或者，如果你想使用同一组的不同版本，或想更明确地表明别名指向某个 API 组/版本，可以创建组/版本别名：

```
import (
corev1 "k8s.io/api/core/v1"
appsv1 "k8s.io/api/apps/v1"
)
```

现在，你可以实例化一个 `Deployment` 结构体：

```
myDep := appsv1.Deployment{}
```

要编译程序，你需要获取该库。为此，请使用：

```
$ go get k8s.io/api@latest
```

或者，如果你想使用特定版本的 Kubernetes API（例如 Kubernetes 1.23），请使用：

```
$ go get k8s.io/api@v0.23
```

所有与 *Kind* 相关的结构体首先嵌入两个通用结构体：`TypeMeta` 和 `ObjectMeta`。两者都在 API Machinery 库的 `/pkg/apis/meta/v1` 包中声明。

### `TypeMeta` 字段

`TypeMeta` 结构体定义如下：

```
type TypeMeta struct {
Kind           string
APIVersion     string
}
```

通常你无需自己设置这些字段的值，因为 API Machinery 通过维护一个 *Scheme*（即 Group-Version-Kind 与 Go 结构体之间的映射）从结构体的类型推断出这些值。请注意，`APIVersion` 值是另一种将 Group-Version 表示为单个字段（包含 `<group>/<version>`，对于传统核心组则仅为 `v1`）的方式。

### `ObjectMeta` 字段

`ObjectMeta` 结构体定义如下（已移除已弃用字段及内部字段）：

```
Type ObjectMeta {
Name                string
GenerateName        string
Namespace           string
UID                 types.UID
ResourceVersion     string
Generation          int64
Labels              map[string]string
Annotations         map[string]string
OwnerReferences     []OwnerReference
[...]
}
```

API Machinery 库的 `/pkg/apis/meta/v1` 包为该结构体的字段定义了 Getter 和 Setter 方法。由于 `ObjectMeta` 嵌入在资源结构体中，因此可以在资源对象本身中使用这些方法。

#### Name

该结构体最重要的信息是资源的名称。你可以使用 `Name` 字段指定资源的确切名称，或者使用 `GenerateName` 字段让 Kubernetes API 为你选择一个唯一名称；它通过在 `GenerateName` 值后添加后缀来确保唯一性。

你可以在资源对象上使用 `GetName() string` 和 `SetName(name string)` 方法来访问其内嵌 `ObjectMeta` 的 `Name` 字段，例如：

```
configmap := corev1.ConfigMap{}
configmap.SetName("config")
```

#### Namespace

有命名空间的资源需要放置在特定的命名空间中。你可能认为需要在 `Namespace` 字段中指定此命名空间，但在创建或更新资源时，你将在请求路径中定义放置资源的命名空间。第 2 章已展示在 `project1` 命名空间中创建 Pod 的请求为：

```
$ curl $HOST/api/v1/namespaces/project1/pods
```

如果你在 `Pod` 结构体中指定的命名空间与 `project1` 不同，将会收到错误：“所提供对象的命名空间与请求中发送的命名空间不匹配。”因此，在创建资源时，无需设置 `Namespace` 字段。

#### UID、ResourceVersion 和 Generation

`UID` 是集群中跨过去、现在和未来资源的唯一标识符。它由控制平面设置，并且在资源的整个生命周期中从未更新。它必须用于引用资源，而不是其种类、名称和命名空间，因为这些可能随时间描述不同的资源。

`ResourceVersion` 是一个不透明值，代表资源的版本。`ResourceVersion` 在每次资源更新时都会改变。

该 `ResourceVersion` 用于乐观并发控制：如果你从 Kubernetes API 获取资源的特定版本，修改它，然后将其发送回 API 进行更新；API 将检查你发送回的资源的 `ResourceVersion` 是否为最新版本。如果在此过程中另一个进程修改了该资源，`ResourceVersion` 将已更改，则你的请求将被拒绝；在这种情况下，你需要读取新版本并再次更新。这不同于悲观并发控制，后者需要在读取资源前获取锁，并在更新后释放锁。

`Generation` 是一个序号，可由资源的控制器用来表示期望状态（`Spec`）的版本。它仅在资源的 `Spec` 部分更新时更新，而其他部分（标签、注解、状态）更新时不会改变。控制器通常使用 `Status` 部分中的 `ObservedGeneration` 字段来指示最后处理的是哪个 generation，并在状态中反映出来。



#### 标签与注解

标签和注解在 Go 中定义为映射（map），其键和值均为字符串。

尽管标签和注解的用途大相径庭，但它们的赋值方式却相同。本节将讨论如何填充 `labels` 字段，但该方法同样适用于注解。

如果你已知要添加为标签的键和值，编写标签字段最简单的方式是直接编写映射，例如：

```
mylabels := map[string]string{
"app.kubernetes.io/component": "my-component",
"app.kubernetes.io/name":      "a-name",
}
```

你也可以向现有映射添加标签，例如：

```
mylabels["app.kubernetes.io/part-of"] = "my-app"
```

如果需要基于动态值构建标签字段，API Machinery 库中提供的 `labels` 包包含一个 `Set` 类型，可能会有所帮助。

```
import "k8s.io/apimachinery/pkg/labels"
mylabels := labels.Set{}
```

该类型提供了以下函数和方法进行操作：

* `ConvertSelectorToLabelsMap` 函数：将选择器字符串转换为 `Set`。
* `Conflicts` 函数：检查两个 `Set` 是否存在冲突标签。冲突标签是指具有相同键但不同值的标签。
* `Merge` 函数：将两个 `Set` 合并为一个 `Set`。如果两个集合之间存在冲突标签，则结果 `Set` 中将使用第二个集合中的标签。
* `Equals` 函数：检查两个 `Set` 是否具有相同的键和值。
* `Has` 方法：指示 `Set` 是否包含某个键。
* `Get` 方法：返回 `Set` 中给定键的值，如果该键未在 `Set` 中定义，则返回空字符串。

你可以用值实例化一个 `Set`，并像操作映射（map）一样为其填充单个值：

```
mySet := labels.Set{
"app.kubernetes.io/component": "my-component",
"app.kubernetes.io/name":      "a-name",
}
mySet["app.kubernetes.io/part-of"] = "my-app"
```

你可以使用以下方法来访问资源的标签和注解：

* `GetLabels() map[string]string`
* `SetLabels(labels map[string]string)`
* `GetAnnotations() map[string]string`
* `SetAnnotations(annotations map[string]string)`

### 设置 APIVersion 和 Kind

使用 `Client-go` 库时（第 4 章将展示如何使用），`APIVersion` 和 `Kind` 不会被自动设置；你需要在复制对象之前，在被引用的对象上设置它们，或者直接将其设置到 `ownerReference` 中：

```
import (
corev1 "k8s.io/api/core/v1"
metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)
// 获取要引用的对象
pod, err := clientset.CoreV1().Pods(myns).
Get(context.TODO(), mypodname, metav1.GetOptions{})
If err != nil {
return err
}
// 方案 1：设置 Pod 的 APIVersion 和 Kind
// 然后复制 Pod 中的所有信息
pod.SetGroupVersionKind(
corev1.SchemeGroupVersion.WithKind("Pod"),
)
ownerRef := metav1.OwnerReference{
APIVersion: pod.APIVersion,
Kind:       pod.Kind,
Name:       pod.GetName(),
UID:        pod.GetUID(),
}
// 方案 2：从 Pod 复制 Name 和 UID
// 然后在 OwnerReference 上设置 APIVersion 和 Kind
ownerRef := metav1.OwnerReference{
Name: pod.GetName(),
UID:  pod.GetUID(),
}
ownerRef.APIVersion, ownerRef.Kind =
corev1.SchemeGroupVersion.WithKind("Pod").
ToAPIVersionAndKind()
```

`APIVersion` 包含与 `Group` 和 `Version` 相同的信息。你可以从 `SchemeGroupVersion` 变量（类型为 `schema.GroupVersion`）中获取这些信息，该变量定义在与资源相关的 API 库的包中（此处 `Pod` 资源对应的包是 `k8s.io/api/core/v1`）。然后，你可以添加 `Kind` 来创建一个 `schema.GroupVersionKind`。

对于第一种方案，你可以在被引用的对象上使用 `SetGroupVersionKind` 方法，根据 `GroupVersionKind` 设置 `APIVersion` 和 `Kind`。对于第二种方案，使用 `GroupVersionKind` 上的 `ToAPIVersionAndKind` 方法获取相应的 `APIVersion` 和 `Kind` 值，然后将它们赋值给 `OwnerReference`。

第 5 章将描述 `API Machinery` 库以及与 `Group`、`Version` 和 `Kind` 相关的所有类型和方法。`OwnerReference` 结构体还包含两个可选的布尔字段：`Controller` 和 `BlockOwnerDeletion`。

#### 设置 Controller

`Controller` 字段指示被引用的对象是否是管理型控制器（或 Operator）。控制器或 Operator 必须在其拥有（owned）的资源上将此值设置为 true，以表明它正在管理该资源。

Kubernetes API 将拒绝在同一资源上添加两个 `Controller` 字段值为 true 的 `OwnerReference`。这样一来，一个资源就不可能被两个不同的控制器管理。

请注意，这与被多个资源拥有是不同的。一个资源可以有不同的所有者；在这种情况下，当所有所有者都被删除时，该资源将被删除，无论哪个所有者是 `Controller`（如果有的话）。

控制器的这种唯一性对于那些可以“采用（adopt）”资源的控制器很有用。例如，`ReplicaSet` 可以采用一个与 `ReplicaSet` 选择器匹配的现有 Pod，但仅当该 `Pod` 尚未被另一个 `ReplicaSet` 或其他控制器控制时才可以。

此字段的值是一个指向布尔值的指针。你可以在设置 `Controller` 字段时声明一个布尔值并使用其地址，或者使用 `Kubernetes Utils` 库中的 `BoolPtr` 函数：

```
// 方案 1：声明一个值并使用其地址
controller := true
ownerRef.Controller = &controller
// 方案 2：使用 BoolPtr 函数
import (
"k8s.io/utils/pointer"
)
ownerRef.Controller = pointer.BoolPtr(true)
```

#### 设置 BlockOwnerDeletion

`OwnerReference` 有助于控制器或其他进程无需处理拥有资源的删除事宜：当所有者被删除时，由 Kubernetes 垃圾收集器负责删除被拥有的资源。

此行为是可配置的。在对资源执行 `Delete` 操作时，你可以使用 `Propagation Policy`（传播策略选项）：

* **Orphan（孤儿）**：指示 Kubernetes API 孤立被拥有的资源，使其不被垃圾收集器删除。
* **Background（后台）**：指示 Kubernetes API 在所有者资源被删除后立即从 DELETE 操作返回，而无需等待垃圾收集器删除拥有资源。
* **Foreground（前台）**：指示 Kubernetes API 在所有者和**`BlockOwnerDeletion` 设置为 true 的拥有资源**被删除后才从 DELETE 操作返回。Kubernetes API 不会等待其他拥有资源被删除。

因此，如果你正在编写一个需要等待所有拥有资源被删除的控制器或其他进程，该进程将需要将所有拥有资源上的 `BlockOwnerDeletion` 字段设置为 true，并在删除所有者资源时使用 Foreground（前台）传播策略。



### 规范与状态

在通用的 `Type` 和 `Metadata` 之后，资源定义通常由两部分组成：`Spec` 和 `Status`。

请注意，这并非适用于所有资源。例如，核心的 `ConfigMap` 和 `Secret` 资源（仅举两例）就不包含 `Spec` 和 `Status` 部分。更普遍的是，包含配置数据且不受任何控制器管理的资源不包含这些字段。

`Spec` 是用户定义的部分，表示用户期望的状态。`Spec` 将由管理该资源的`控制器`读取，控制器会根据 `Spec` 在集群上创建、更新或删除资源，并将其操作结果检索到资源的 `Status` 部分。控制器读取 `Spec`、应用到集群并检索 `Status` 的此过程称为`调谐循环`。

### 与编写 YAML 清单的比较

当你编写一个与 `kubectl` 配合使用的 Kubernetes 清单时：

-   清单以 `apiVersion` 和 `kind` 开头。
-   `metadata` 字段包含该资源的所有元数据。
-   随后是 `Spec` 和 `Status` 字段（或其他字段）。

当你用 Go 语言编写 Kubernetes 结构体时，会发生以下情况：

-   结构体的类型决定了 `apiVersion` 和 `Kind`；无需指定它们。
-   元数据可以通过嵌入 `metav1.ObjectMeta` 结构体来定义，或者通过使用资源上的*元数据设置器*来定义。
-   随后是 `Spec` 和 `Status` 字段（或其他字段），使用它们自己的 Go 结构体或其他类型。

举个例子，以下是使用 YAML 清单与 Go 语言定义 `Pod` 的方法对比。YAML 格式：

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    - component: my-component,
spec:
  containers:
    - image: nginx
      name: nginx
```

在 Go 语言中，当你为元数据使用设置器时：

```
pod := corev1.Pod{
    Spec: corev1.PodSpec{
        Containers: []corev1.Container{
            {
                Name:  "runtime",
                Image: "nginx",
            },
        },
    },
}
pod.SetName("my-pod")
pod.SetLabels(map[string]string{
    "component": "my-component",
})
```

或者，在 Go 语言中，当你将 `metav1.ObjectMeta` 结构体嵌入到 `Pod` 结构体中时：

```
pod2 := corev1.Pod{
    ObjectMeta: metav1.ObjectMeta{
        Name: "nginx",
        Labels: map[string]string{
            "component": "mycomponent",
        },
    },
    Spec: corev1.PodSpec{
        Containers: []corev1.Container{
            {
                Name:  "runtime",
                Image: "nginx",
            },
        },
    },
}
```

## 完整示例

这个完整示例使用截至目前学习的概念，通过 `POST` 请求在集群上创建一个 `Pod`。

-   ➊ 使用 Go 结构体构建一个 `Pod` 对象，如本章前面所示
-   ➋ 使用序列化器（更多相关信息请参见第 5 章）将 `Pod` 对象序列化为 JSON
-   ➌ 构建一个 HTTP `POST` 请求，其主体包含要创建的、已序列化为 JSON 的 `Pod`
-   ➍ 使用构建好的请求调用 Kubernetes API
-   ➎ 从响应中获取主体
-   ➏ 根据响应状态码：

如果请求返回 2xx 状态码：

-   ➐ 将响应主体反序列化为 `Pod` Go 结构体
-   ➑ 将创建的 `Pod` 对象以 JSON 格式显示以供参考；

否则：

-   ➒ 将响应主体反序列化为 `Status` Go 结构体
-   ➓ 将 `Status` 对象以 JSON 格式显示以供参考：

```
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "net/http"

    corev1 "k8s.io/api/core/v1"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/apimachinery/pkg/runtime"
    "k8s.io/apimachinery/pkg/runtime/schema"
    "k8s.io/apimachinery/pkg/runtime/serializer/json"
)

func createPod() error {
    pod := createPodObject() ➊
    serializer := getJSONSerializer()
    postBody, err := serializePodObject(serializer, pod) ➋
    if err != nil {
        return err
    }
    reqCreate, err := buildPostRequest(postBody) ➌
    if err != nil {
        return err
    }
    client := &http.Client{}
    resp, err := client.Do(reqCreate) ➍
    if err != nil {
        return err
    }
    defer resp.Body.Close()
    body, err := io.ReadAll(resp.Body) ➎
    if err != nil {
        return err
    }
    if resp.StatusCode < 300 { ➏
        createdPod, err := deserializePodBody(serializer, body) ➐
        if err != nil {
            return err
        }
        json, err := json.MarshalIndent(createdPod, "", "  ")
        if err != nil {
            return err
        }
        fmt.Printf("%s\n", json) ➑
    } else {
        status, err := deserializeStatusBody(serializer, body) ➒
        if err != nil {
            return err
        }
        json, err := json.MarshalIndent(status, "", "  ")
        if err != nil {
            return err
        }
        fmt.Printf("%s\n", json) ➓
    }
    return nil
}

func createPodObject() *corev1.Pod { ➊
    pod := corev1.Pod{
        Spec: corev1.PodSpec{
            Containers: []corev1.Container{
                {
                    Name:  "runtime",
                    Image: "nginx",
                },
            },
        },
    }
    pod.SetName("my-pod")
    pod.SetLabels(map[string]string{
        "app.kubernetes.io/component": "my-component",
        "app.kubernetes.io/name":      "a-name",
    })
    return &pod
}

func serializePodObject( ➋
    serializer runtime.Serializer,
    pod *corev1.Pod,
) (
    io.Reader,
    error,
) {
    var buf bytes.Buffer
    err := serializer.Encode(pod, &buf)
    if err != nil {
        return nil, err
    }
    return &buf, nil
}

func buildPostRequest( ➌
    body io.Reader,
) (
    *http.Request,
    error,
) {
    reqCreate, err := http.NewRequest(
        "POST",
        "http://127.0.0.1:8001/api/v1/namespaces/default/pods",
        body,
    )
    if err != nil {
        return nil, err
    }
    reqCreate.Header.Add(
        "Accept",
        "application/json",
    )
    reqCreate.Header.Add(
        "Content-Type",
        "application/json",
    )
    return reqCreate, nil
}

func deserializePodBody( ➐
    serializer runtime.Serializer,
    body []byte,
) (
    *corev1.Pod,
    error,
) {
    var result corev1.Pod
    _, _, err := serializer.Decode(body, nil, &result)
    if err != nil {
        return nil, err
    }
    return &result, nil
}

func deserializeStatusBody( ➒
    serializer runtime.Serializer,
    body []byte,
) (
    *metav1.Status,
    error,
) {
    var status metav1.Status
    _, _, err := serializer.Decode(body, nil, &status)
    if err != nil {
        return nil, err
    }
    return &status, nil
}

func getJSONSerializer() runtime.Serializer {
    scheme := runtime.NewScheme()
    scheme.AddKnownTypes(
        schema.GroupVersion{
            Group:   "",
            Version: "v1",
        },
        &corev1.Pod{},
        &metav1.Status{},
    )
    return json.NewSerializerWithOptions(
        json.SimpleMetaFactory{},
        nil,
        scheme,
        json.SerializerOptions{},
    )
}
```

## 总结

在本章中，你了解了第一个用于在 Go 中操作 Kubernetes 的库——API 库。它本质上是一组用于声明 Kubernetes 资源的 Go 结构体。本章还探讨了 API Machinery 库中定义的所有资源通用的元数据字段的定义。

本章末尾有一个程序示例，演示了如何使用 API 库构建 `Pod` 定义，然后通过 HTTP 请求调用 API 服务器在集群中创建此 `Pod`。

接下来的章节将探讨其他基础库——API Machinery 和 Client-go，使用它们你将不再需要手动构建 HTTP 请求。

