# 8. 使用自定义资源定义扩展 Kubernetes API

在本书的第一部分，你已经了解到 Kubernetes API 是按组来组织的。每个组包含一个或多个资源，每个资源都有不同的版本。

为了在 Go 中操作 Kubernetes API，存在两个基础库。API 机制库^(¹) 提供了与 API 通信的工具，这些工具独立于 API 所服务的资源。API 库^(²) 则提供了 Kubernetes API 中原生资源的定义，以便与 API 机制库配合使用。

`Client-go` 库利用 API 机制库和 API 库，提供了访问 Kubernetes API 的途径。

Kubernetes API 可以通过其自定义资源定义（CRD）机制进行扩展。

`CustomResourceDefinition` 是一种特定的 Kubernetes 资源，用于以动态方式定义由 API 提供的新 Kubernetes 资源。

为 Kubernetes 定义新资源，用于表示*领域特定资源*，例如数据库、CI/CD 任务或证书。

结合自定义控制器，这些自定义资源可以由控制器在集群中实现。

与其他资源一样，你可以对这种资源执行获取、列表、创建、删除和更新操作。CRD 资源是一种非命名空间资源。

该 CRD 资源定义在 `apiextensions.k8s.io/v1` 组和版本中。访问该资源的 HTTP 路径遵循访问非核心和非命名空间资源的标准格式：`/apis/<组>/<版本>/<资源复数名称>`，即 `/apis/apiextensions.k8s.io/v1/customresourcedefinitions/`。

与其他原生资源不同，此资源的 Go 定义并未声明在 API 库中，而是声明在 `apiextensions-apiserver` 库中。要从 Go 源码中访问这些定义，你需要使用以下导入语句：

```
import (
"k8s.io/apiextensions-apiserver/pkg/apis/apiextensions/v1"
)
```

## 在 Go 中执行操作

要使用 Go 对 CRD 资源执行操作，你可以使用 clientset，它类似于 `client-go` 库提供的 clientset，但包含在 `apiextensions-apiserver` 库中。

要使用此 clientset，你需要使用以下导入语句：

```
import (
"k8s.io/apiextensions-apiserver/pkg/client/clientset/clientset"
)
```

你可以像使用 `Client-go` 的 Clientset 一样，以完全相同的方式使用此 CRD clientset。

例如，下面展示了如何列出集群中声明的 CRD：

```
import (
"context"
"fmt"
"k8s.io/apiextensions-apiserver/pkg/client/clientset/clientset"
metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)
// config 是 client-go 中定义的标准 rest.Config
clientset, err := clientset.NewForConfig(config)
if err != nil {
return err
}
ctx := context.Background()
list, err := clientset.ApiextensionsV1().
CustomResourceDefinitions().
List(ctx, metav1.ListOptions{})
```

## 深入了解 CustomResourceDefinition

`CustomResourceDefinition` Go 结构的定义如下：

```
type CustomResourceDefinition struct {
metav1.TypeMeta
metav1.ObjectMeta
Spec CustomResourceDefinitionSpec
Status CustomResourceDefinitionStatus
}
```

与其他任何 Kubernetes 资源一样，CRD 资源嵌入了 `TypeMeta` 和 `ObjectMeta` 结构。`ObjectMeta` 字段中的 `Name` 值必须等于 `<Spec.Names.Plural>` + `"."` + `<Spec.Group>`。

与由控制器或 Operator 管理的资源类似，该资源包含一个用于定义期望状态的 `Spec` 结构，以及一个用于包含由控制器或 Operator 描述的该资源状态的结构 `Status`。

```
type CustomResourceDefinitionSpec struct {
Group           string
Scope           ResourceScope
Names           CustomResourceDefinitionNames
Versions        []CustomResourceDefinitionVersion
Conversion      *CustomResourceConversion
PreserveUnknownFields bool
}
```

Spec 的 `Group` 字段指明了资源所在的组名。

`Scope` 字段指明资源是否具有命名空间。`ResourceScope` 类型包含两个值：`ClusterScoped` 和 `NamespaceScoped`。

### 命名资源

`Names` 字段指明了资源的各种名称及其相关信息。Kubernetes API 的发现机制会使用这些信息，以便将 CRD 包含在发现调用的结果中。`CustomResourceDefinitionNames` 类型定义如下：

```
type CustomResourceDefinitionNames struct {
Plural         string
Singular       string
ShortNames     []string
Kind           string
ListKind       string
Categories     []string
}
```

`Plural` 是资源的小写复数名称，用于 URL 中访问该资源，例如 `pods`。此字段为必填项。

`Singular` 是资源的小写单数名称，例如 `pod`。此字段为可选项，如果未指定，则使用 `Kind` 字段的小写值。

`ShortNames` 是资源的小写短名称列表，可用于在诸如 `kubectl get <短名称>` 的命令中调用该资源。例如，`services` 资源声明了短名称 `svc`，因此你可以执行 `kubectl get svc` 来代替 `kubectl get services`。

`Kind` 是资源的*驼峰式*单数名称，用于资源序列化，例如 `Pod` 或 `ServiceAccount`。此字段为必填项。

`ListKind` 是用于该资源列表序列化的名称，例如 `PodList`。此字段为可选项，如果未指定，则使用 `Kind` 值加上 `List` 后缀。

`Categories` 是该资源所属的已分组资源的列表，可供诸如 `kubectl get <类别>` 的命令使用。`all` 类别是众所周知的，但也存在其他类别（`api-extensions`），并且你可以定义自己的类别名称。



### 资源版本的定义

到目前为止提供的所有信息对所有版本的资源均有效。

`Versions` 字段包含版本特定信息，以定义列表的形式存在，每个版本的资源对应一个定义。`CustomResourceDefinitionVersion` 类型的定义如下：

```
type CustomResourceDefinitionVersion struct {
Name                      string
Served                    bool
Storage                   bool
Deprecated                bool
DeprecationWarning        *string
Schema                    *CustomResourceValidation
Subresources              *CustomResourceSubresources
AdditionalPrinterColumns  []CustomResourceColumnDefinition
}
```

`Name` 表示版本名称。Kubernetes 资源使用标准格式表示版本：`v<number>[(alpha|beta)<number>]`，但你可以使用任何你想要的格式。

`Served` 布尔值指示此特定版本是否必须由 API Server 提供服务。如果不是，该版本仍然被定义并可用作存储（见下文），但用户不能以该特定版本创建或获取资源实例。

`Storage` 布尔值指示此特定版本是否用于持久化资源。必须恰好有一个版本将此字段设置为 true。你已经在第 5 章的“转换”部分看到，同一资源的不同版本之间存在转换函数。用户可以在任何可用的版本中创建资源，API Server 会将其转换为 `Storage=true` 的版本，然后再将数据持久化到 `etcd` 中。

`Deprecated` 布尔值指示此特定版本的资源是否已被弃用。如果为 true，服务器将在对此版本的响应中添加一个 Warning 头。

`DeprecationWarning` 是当 `Deprecated` 为 true 时返回给调用者的警告消息。如果 `Deprecated` 为 true 且此字段为 nil，则会发送一条默认警告消息。

`Schema` 描述此资源版本的架构。该架构用于验证在创建或更新资源时发送到 API 的数据。此字段是可选的。架构将在“资源的架构”一节中更详细地讨论。

`Subresources` 定义为该资源版本提供的子资源。此字段是可选的。此字段的类型定义如下：

- 如果 `Status` 不为 nil，则将提供 “/status” 子资源。
- 如果 `Scale` 不为 nil，则将提供 “/scale” 子资源。

```
type CustomResourceSubresources struct {
Status *CustomResourceSubresourceStatus
Scale *CustomResourceSubresourceScale
}
```

`AdditionalPrinterColumns` 是当用户请求 Table 输出格式时要返回的额外列的列表。此字段是可选的。有关 Table 输出格式的更多信息，请参见第 2 章的“获取结果为表格”一节。你将看到如何在本章的“额外打印列”一节中定义额外的打印列。

### 版本间转换

`Conversion` 指示如何处理此资源版本间的转换。此字段的类型定义如下：

```
type CustomResourceConversion struct {
Strategy ConversionStrategyType
Webhook *WebhookConversion
}
```

`Strategy` 可以取 `NoneConverter` 或 `WebhookConverter` 值。`NoneConverter` 值指示 API Server 在持久化之前仅更改 `APIVersion` 字段。

`WebhookConverter` 值指示 API Server 调用外部 webhook 来进行转换。当使用此值时，`Webhook` 字段需要包含 API Server 调用 webhook 所需的全部信息。

## 资源的架构

特定版本的资源架构是使用 OpenAPI v3 架构格式定义的。^(³) 对该格式的完整描述超出了本书的范围。以下是一个简短的介绍，可以帮助你为你的资源定义一个架构。

自定义资源通常有一个 `Spec` 部分和一个 `Status` 部分。为此，你需要声明一个类型为 `object` 的顶层架构，并使用 `properties` 属性声明字段。在此示例中，顶层架构将具有 `Spec` 和 `Status` 属性。然后你可以递归地描述每个属性。

`Spec` 和 `Status` 字段也属于 `object` 类型，并且包含属性。可接受的数据类型包括：

- `string`
- `number`
- `integer`
- `boolean`
- `array`
- `object`

可以为 `string` 和 `number` 类型指定特定格式：

- `string`: date, date-time, byte, int-or-string
- `number`: float, double, int32, int64

在一个对象中，必需的字段使用 `required` 属性指示。

你可以使用 `enum` 属性声明一个属性仅接受一组值。要声明一个值映射（map），你需要使用 `object` 类型并指定 `additionalProperties` 属性。此属性可以接受一个 `true` 值（表示条目可以具有任何类型），或者通过给出一个类型作为值来定义映射中条目的类型：

```
type: object
additionalProperties:
type: string
```

或者：

```
type: object
additionalProperties:
type: object
properties:
code:
type: integer
text:
type: string
```

当声明一个数组时，你必须定义数组项的类型：

```
type: array
items:
type: string
```

例如，这是一个自定义资源的架构，它在 `Spec` 中包含三个字段（`image`、`replicas` 和 `port`），并在 `Status` 中包含一个 `state` 字段。

```
schema:
openAPIV3Schema:
type: object
properties:
spec:
type: object
properties:
image:
type: string
replicas:
type: integer
port:
type: string
format: int-or-string
required: [image,replicas]
status:
type: object
properties:
state:
type: string
enum: [waiting,running]
```



### 部署自定义资源定义

要在集群中部署新的资源定义，需要对 CRD 对象执行 `Create` 操作。你可以使用 `apiextensions-apiserver` 库提供的 `Clientset`，并利用前面章节介绍的结构体在 Go 中编写资源定义；或者，也可以创建一个 YAML 文件，并使用 `kubectl` 进行“应用”。

作为示例，你将使用 YAML 格式和 `kubectl` 在 `mygroup.example.com` 组中创建一个名为 `myresources` 的新资源，其版本为 `v1alpha1`。

- ❶ CRD 资源的组和版本
- ❷ CRD 资源的 `kind`
- ❸ 新资源的完整名称，包括其所属组
- ❹ 新资源所属的组
- ❺ 新资源可以在特定命名空间内创建
- ❻ 新资源的复数名称，用于访问此新资源的路径
- ❼ 资源的单数名称，可以使用 `kubectl get myresource`
- ❽ 新资源的短名称，可以使用 `kubectl get my`、`kubectl get myres`
- ❾ 将资源添加到 `all` 类别中；运行 `kubectl get all` 时，将显示此 kind 的资源
- ❿ `v1alpha1` 版本是为新资源定义的唯一版本
- ⓫ 将新资源的模式(schema)定义为对象，且不包含任何字段

```
apiVersion: apiextensions.k8s.io/v1        ❶
kind: CustomResourceDefinition             ❷
metadata:
name: myresources.mygroup.example.com    ❸
spec:
group: mygroup.example.com               ❹
scope: Namespaced                        ❺
names:
plural: myresources                    ❻
singular: myresource                   ❼
shortNames:
- my                                   ❽
- myres
kind: MyResource
categories:
- all                                  ❾
versions:
- name: v1alpha1                         ❿
served: true
storage: true
schema:
openAPIV3Schema:
type: object                       ⓫
```

现在，你可以使用 `kubectl` 通过以下命令“应用”此资源：

```
$ kubectl apply -f myresource.yaml
customresourcedefinition.apiextensions.k8s.io/myresources.mygroup.example.com created
```

从此以后，你就可以操作这个新资源了。

例如，你可以使用 HTTP 请求获取集群范围或特定命名空间内的资源列表（在运行这些命令之前，需要在另一个终端执行 `kubectl proxy`）：

```
$ curl
http://localhost:8001/apis/mygroup.example.com/v1alpha1/myresources
{"apiVersion":"mygroup.example.com/v1alpha1","items":[],"kind":"MyResourceList","metadata":{"continue":"","resourceVersion":"186523407"}}
$ curl
http://localhost:8001/apis/mygroup.example.com/v1alpha1/namespaces/default/myresources
{"apiVersion":"mygroup.example.com/v1alpha1","items":[],"kind":"MyResourceList","metadata":{"continue":"","resourceVersion":"186524840"}}
```

或者，你也可以使用 `kubectl` 获取这些列表：

```
$ kubectl get myresources
No resources found in default namespace.
```

你可以使用 YAML 格式定义一个新资源，并使用 `kubectl` 将其“应用”到集群：

```
$ kubectl apply -f - <<EOF
apiVersion: mygroup.example.com/v1alpha1
kind: MyResource
metadata:
name: myres1
EOF
$ kubectl get myresources
NAME     AGE
myres1   10s
```

## 附加打印列(Additional Printer Columns)

从之前的 `kubectl get myresources` 命令输出中可以看到，显示的列是资源的*名称*和*年龄*。

CRD 规约中的 `AdditionalPrinterColumns` 字段用于指定在 `kubectl get <resource>` 的输出中希望显示资源的哪些列。

当未定义附加打印列时，默认返回*名称*和*年龄*。如果你指定了列，它们将与*名称*一起返回。如果希望在添加列时保留*年龄*，则需要显式地添加它。

对于每个附加列，你需要指定名称(name)、JSON 路径(JSON path)和类型(type)。名称将用作输出中该列的标题，JSON 路径用于 API 服务器从资源数据中获取该列的值，类型则指示 `kubectl` 如何显示此数据。

下面是一个示例，展示了在 `Spec` 和 `Status` 中定义了一些字段的 `myresource` CRD，同时定义了附加打印列：

```
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
name: myresources.mygroup.example.com
spec:
group: mygroup.example.com
scope: Namespaced
names:
plural: myresources
singular: myresource
shortNames:
- my
- myres
kind: MyResource
categories:
- all
versions:
- name: v1alpha1
served: true
storage: true
subresources:
status: {}
schema:
openAPIV3Schema:
type: object
properties:
spec:
type: object
properties:
image:
type: string
memory:
x-kubernetes-int-or-string: true
status:
type: object
properties:
state:
type: string
additionalPrinterColumns:
- name: image
jsonPath: .spec.image
type: string
- name: memory
jsonPath: .spec.memory
type: string
- name: age
jsonPath: .metadata.creationTimestamp
type: date
```

在此 CRD 中，模式(schema)表明资源的数据将是一个包含两个属性 `spec` 和 `status` 的对象。这些属性本身也是对象。`spec` 对象包含两个属性：类型为 `string` 的 `image`，以及特殊类型 `int or string` 的 `memory`。`status` 对象包含一个类型为 `string` 的属性 `state`。

请注意，你通过在 `subresources` 对象中添加 `status` 条目启用了 `status` 子资源。你为此 CRD 定义了三个附加打印列：`image`、`memory` 和 `age`。`image` 和 `memory` 的 JSON 路径指向模式中定义的字段，而 `age` 的 JSON 路径指向标准 `metadata` 字段中的一个字段。

再次将 CRD 应用到集群后，你可以使用这些字段定义一个新资源，将资源定义应用到集群，并使用 `kubectl` 获取资源列表：

- ❶ CRD 更新了新的模式和附加打印列
- ❷ 为 `image` 字段提供了一个值
- ❸ 为 `memory` 字段提供了一个值；这是一个字符串
- ❹ 自定义资源已更新，包含了定义的新字段
- ❺ `kubectl get` 的输出现在显示了附加列

```
$ kubectl apply -f myresource.yaml
customresourcedefinition.apiextensions.k8s.io/myresources.mygroup.example.com configured                              ❶
$ cat > myres1.yaml <<EOF
apiVersion: mygroup.example.com/v1alpha1
kind: MyResource
metadata:
name: myres1
spec:
image: nginx                              ❷
memory: 1024Mi                            ❸
EOF
$ kubectl apply -f myres1.yaml              ❹
myresource.mygroup.example.com/myres1 configured
$ kubectl get myresources
NAME     IMAGE   MEMORY   AGE
myres1   nginx   1024Mi   12m               ❺
```

## 结论

在本章中，你了解了通过在集群中创建 `CustomResourceDefinition` (CRD) 资源可以扩展 Kubernetes API 提供的资源列表。你了解了此 CRD 资源在 Go 中的结构，以及如何使用专用的 `Clientset` 操作 CRD 资源。

随后，你了解了如何用 YAML 编写 CRD 以及如何使用 `kubectl` 将其部署到集群。然后向 CRD 模式中添加了一些字段，以及一些附加打印列，以便在 `kubectl get` 的输出中显示这些字段。

最后，你了解到只要在集群中创建了 CRD，就可以开始操作新关联 kind 的资源。

脚注 1  2  3



