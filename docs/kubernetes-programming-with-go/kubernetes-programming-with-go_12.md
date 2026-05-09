# 7. 使用 Client-go 测试应用程序

`Client-go` 库提供了多个可与 Kubernetes API 交互的客户端。

- `kubernetes.Clientset` 提供了一组客户端，每个客户端对应 API 的一个组/版本，用于对资源执行 Kubernetes 操作（如创建、更新、删除等）。
- `rest.RESTClient` 提供一个用于对资源执行 REST 操作的客户端（如获取、发布、删除等）。
- `discovery.DiscoveryClient` 提供一个用于发现 API 所提供资源的客户端。

所有这些客户端都实现了由 Client-go 库定义的接口：`kubernetes.Interface`、`rest.Interface` 和 `discovery.DiscoveryInterface`。

此外，Client-go 库还提供了这些接口的*模拟*实现，以帮助你为函数编写单元测试。这些*模拟*实现定义在 `fake` 包中，每个包都位于*真实*实现的目录内：`kubernetes/fake`、`rest/fake` 和 `discovery/fake`。

`testing` 目录包含*模拟*客户端常用的通用工具——例如，对象追踪器或调用追踪系统。在本章中，你将了解到这些工具。

为了能够使用这些客户端测试你的函数，函数需要定义一个参数来传递客户端实现，且该参数的类型必须是接口，而非具体类型。例如：

```go
func CreatePod(
ctx context.Context,
clientset kubernetes.Interface,
name string,
namespace string,
image string,
) (pod *corev1.Pod, error)
```

通过这种方式，你将在函数外部创建客户端，并在函数内部简单地使用任何实现。

对于实际代码，你将按照第 6 章所述创建客户端。对于测试，你将使用 *fake* 包中的辅助函数来替换客户端。

## Fake Clientset

以下函数用于创建一个*模拟*的 `Clientset`：

```go
import "k8s.io/client-go/kubernetes/fake"
func NewSimpleClientset(
objects ...runtime.Object,
) *Clientset
```

*模拟* `Clientset` 由一个对象追踪器支持，该追踪器处理*创建*、*更新*和*删除*操作时不进行任何验证或变更，并在响应*获取*和*列出*操作时返回对象。

你可以将一组 Kubernetes 对象作为参数传递，这些对象将被添加到 Clientset 的对象追踪器中。例如，使用以下代码创建一个*模拟* `Clientset` 并调用前面定义的 `CreatePod` 函数：

```go
import "k8s.io/client-go/kubernetes/fake"
clientset := fake.NewSimpleClientset()
pod, err := CreatePod(
context.Background(),
clientset,
aName,
aNs,
anImage,
)
```

在测试期间，调用被测函数（此处为 `CreatePod`）后，你有多种方法可以验证该函数是否按预期执行。让我们考虑一下 `CreatePod` 函数的这个实现：

```go
func CreatePod(
ctx context.Context,
clientset kubernetes.Interface,
name string,
namespace string,
image string,
) (pod *corev1.Pod, err error) {
podToCreate := corev1.Pod{
Spec: corev1.PodSpec{
Containers: []corev1.Container{
{
Name:  "runtime",
Image: image,
},
},
},
}
podToCreate.SetName(name)
return clientset.CoreV1().
Pods(namespace).
Create(
ctx,
&podToCreate,
metav1.CreateOptions{},
)
}
```

### 检查函数结果

当使用*模拟* `Clientset` 调用 `CreatePod` 函数时，不会调用实际的 Kubernetes API，资源也不会在 `etcd` 数据库中生成，并且不会对资源执行任何验证或变更。相反，资源将以其原始状态存储在内存存储中，仅进行极少的转换。

在此示例中，传递给 `Create` 函数的 `podToCreate` 与 `Create` 函数返回的 Pod 之间唯一的转换是命名空间，该命名空间通过调用 `Pods(namespace)` 传递，并被添加到返回的 `Pod` 中。

要测试 `CreatePod` 函数返回的值是否符合预期，你可以编写以下测试代码：

```go
func TestCreatePod(t *testing.T) {
var (
name      = "a-name"
namespace = "a-namespace"
image     = "an-image"
wantPod = &corev1.Pod{
ObjectMeta: v1.ObjectMeta{
Name:      "a-name",
Namespace: "a-namespace",
},
Spec: corev1.PodSpec{
Containers: []corev1.Container{
{
Name:  "runtime",
Image: "an-image",
},
},
},
}
)
clientset := fake.NewSimpleClientset()
gotPod, err := CreatePod(
context.Background(),
clientset,
name,
namespace,
image,
)
if err != nil {
t.Errorf("err = %v, want nil", err)
}
if !reflect.DeepEqual(gotPod, wantPod) {
t.Errorf("CreatePod() = %v, want %v",
gotPod,
wantPod,
)
}
}
```

此测试将断言：给定名称、命名空间和镜像，当 `Pod` 中未发生验证或变更时，函数的结果将是 `wantPod`。

无法使用此测试来了解使用*真实*客户端时会发生什么，因为结果会不同——也就是说，*真实*客户端及其底层 API 会变更对象以添加默认值等。



### 对操作作出反应

*伪造的* `Clientset` 会原样存储资源，不进行任何验证或变更。在测试过程中，你可能希望模拟各种控制器对资源所做的更改。为此，*伪造的* `clientset` 提供了添加*反应器*的方法。*反应器*是在对特定资源执行特定操作时执行的函数。

除 `Watch` 和 `Proxy` 外，所有操作的反应器函数类型定义如下：

```
type ReactionFunc func(
action Action,
) (handled bool, ret runtime.Object, err error)
```

`Watch` 操作的反应器函数类型定义如下：

```
type WatchReactionFunc func(
action Action,
) (handled bool, ret watch.Interface, err error)
```

`Proxy` 操作的反应器函数类型定义如下：

```
type ProxyReactionFunc func(
action Action,
) (
handled bool,
ret restclient.ResponseWrapper,
err error
)
```

*伪造的* `Clientset` 的 `Fake` 字段维护了几个 `Reaction` 函数的列表：`ReactionChain`、`WatchReactionChain` 和 `ProxyReactionChain`。

每次调用对资源的操作时，反应器都会以链式方式执行。第一个返回 *true* 值 `handled` 的反应器将立即终止该链，后续的反应器将不会被调用。

你可以通过调用 *伪造的* `Clientset` 的 `Fake` 字段上的以下方法来注册新的 `Reaction` 函数：

- `AddReactor(verb, resource string, reaction ReactionFunc)`
- `PrependReactor(verb, resource string, reaction ReactionFunc)`
- `AddWatchReactor(resource string, reaction WatchReactionFunc)`
- `PrependWatchReactor(resource string, reaction WatchReactionFunc)`
- `AddProxyReactor(resource string, reaction ProxyReactionFunc)`
- `PrependProxyReactor(resource string, reaction ProxyReactionFunc)`

`verb` 和 `resource` 值用于指示将调用反应器的操作。

你可以为这两个参数指定 `"*"` 值，以指示该反应器必须针对任何 `verb` 和/或任何 `resource` 的操作执行。请注意，`Reactor` 函数会将 `action` 作为参数；`Reactor` 可以使用此 `action` 参数对调用的操作进行额外的过滤。

当创建*伪造的* `Clientset` 时，会为两个链添加一个 `Reactor`：`ReactionChain` 和 `WatchReactionChain`。这些反应器包含*伪造的* `Clientset` 使用前文讨论过的对象跟踪器的代码。由于这些反应器的存在，例如，当你使用*伪造的* `Clientset` 调用 `Create` 操作时，该资源将被附加到一个内存数据库中，随后对该资源调用 `Get` 操作将返回先前保存的资源。

如果你不想使用此默认行为，可以将这些链 `ReactionChain` 和/或 `WatchReactionChain` 重新定义为空链。请注意，这些默认反应器始终返回一个 *true* 值 `handled`；如果在默认反应器仍位于反应器列表开头时使用 `AddReactor` 或 `AddWatchReactor` 添加反应器，则这些新添加的反应器将永远不会被请求。

如果你希望对传入的资源进行一些验证或变更，你可以使用 `PrependReactor` 或 `PrependWatchReactor` 在反应器前添加，这样它们可以：

- 提前返回一个 *true* 值 `handled` 和一个要返回的特定对象值（并且默认的 `Reactor` 将不会被调用）。
- 更改要返回的对象并返回一个 *false* 值 `handled`。被更改的对象将成为下一个反应器的源对象。

例如，使用以下代码来更改已创建的 Pod，为其 `NodeName` 字段添加一个值：

```
import (
"context"
corev1 "k8s.io/api/core/v1"
"k8s.io/client-go/kubernetes/fake"
"k8s.io/apimachinery/pkg/runtime"
ktesting "k8s.io/client-go/testing"
)
clientset := fake.NewSimpleClientset()
clientset.Fake.PrependReactor("create", "pods", func(
action ktesting.Action,
) (handled bool, ret runtime.Object, err error) {
act := action.(ktesting.CreateAction)
ret = act.GetObject()
pod := ret.(*corev1.Pod)
pod.Spec.NodeName = "node1"
return false, pod, nil
})
pod, _ := CreatePod(
context.Background(),
clientset,
name,
namespace,
image,
)
```



### 检查 Actions

如果你正在使用 Go 团队提供的 `mock` 系统（[github.com/golang/mock](https://github.com/golang/mock)），你可能想以类似的方式 `mock` 对 `Client-go` 库的调用。`mock` 系统帮助你验证测试代码执行期间，是否以特定参数调用了特定函数，并返回了特定值。

遵循这一原则，要评估 `CreatePod` 函数，你可能需要检查 `Clientset` 的 `Create` 函数是否以特定参数被调用。为此，`fake` Clientset 会记录在其上执行的所有 `Actions`，你可以访问这些 `Actions` 来检查它们是否符合你的预期。

在执行待评估的代码后，你可以通过以下调用访问 `Actions`：

```
actions := clientset.Actions()
```

该调用返回一个对象列表，这些对象实现了以下 `Action` 接口：

```
type Action interface {
GetNamespace() string
GetVerb() string
GetResource() schema.GroupVersionResource
Matches(verb, resource string) bool
[...]
}
```

在第 6 章的“使用 Clientset”部分，你了解了在资源上执行操作的模式（对于非命名空间资源，省略命名空间）如下：

- `GetNamespace()` – `Action` 上的此方法返回调用中指定的 `namespace`。
- `GetVerb()` – `Action` 上的此方法返回调用中使用的 `Operation`。
- `GetResource()` – `Action` 上的此方法返回一个 `GVR`，由调用中使用的 `GroupVersion` 和 `Resource` 构建。
- `Matches(verb, resource string)` – 检查 `verb` 和 `resource` 是否与调用中使用的 `Operation` 和 `Resource` 匹配。

```
clientset.
GroupVersion().
Resource(namespace).
Operation(ctx, options)
```

使用这四个方法，你可以检查在 `Clientset` 上执行的操作是否符合你的预期。

在检查 `Action` 的 Verb 之后，你可以将其转换为与 `Verb` 相关的接口之一：`GetAction`、`ListAction`、`CreateAction`、`UpdateAction`、`DeleteAction`、`DeleteCollectionAction`、`PatchAction`、`WatchAction`、`ProxyGetAction` 和 `GenericAction`。

这些接口提供了更多方法来获取特定于 `Operation` 的信息。

- **GetAction 接口** – 定义如下：

```
type GetAction interface {
Action
GetName() string
}
```

如第 6 章的“获取资源信息”部分所示，执行 `Get` 操作的方法签名是：

- `Get(ctx context.Context, name string, opts metav1.GetOptions)` – `GetAction` 接口中的 `GetName()` 方法将返回作为参数传递给 `Get` 方法的 `name`。

- **ListAction 接口** – 定义如下：

```
type ListRestrictions struct {
Labels labels.Selector
Fields fields.Selector
}

type ListAction interface {
Action
GetListRestrictions() ListRestrictions
}
```

如第 6 章的“获取资源列表”部分所示，执行 `List` 操作的方法签名是：

- `List(ctx context.Context, opts metav1.ListOptions)` – 其中 `ListOptions` 定义如下：

```
type ListOptions struct {
LabelSelector string
FieldSelector string
[...]
}
```

`ListAction` 接口中的 `GetListRestrictions()` 方法将返回通过 `ListOptions` 结构体作为参数传递给 `List` 方法的 `LabelSelector` 和 `FieldSelector`。

- **CreateAction 接口** – 定义如下：

```
type CreateAction interface {
Action
GetObject() runtime.Object
}
```

如第 6 章的“创建资源”部分所示，执行 `Create` 操作的方法签名如下（此处以 `Pods` 为例）：

- `Create(ctx context.Context, pod *v1.Pod, opts metav1.CreateOptions)`



- `GetObject()`方法来自`CreateAction`接口，会返回作为参数传递给`Create`方法的对象（这里是`pod`）

- **`UpdateAction`接口** – 定义如下：

```go
type UpdateAction interface {
    Action
    GetObject() runtime.Object
}
```

如第 6 章“更新资源”部分所示，执行`Update`操作的方法签名（这里是针对`Pods`）：

- `Update(ctx context.Context, pod *v1.Pod, opts metav1.UpdateOptions)`

- `GetObject()`方法来自`UpdateAction`接口，会返回作为参数传递给`Update`方法的对象（这里是`pod`）

- **`DeleteAction`接口** – 代码如下：

```go
type DeleteAction interface {
    Action
    GetName() string
    GetDeleteOptions() metav1.DeleteOptions
}
```

如第 6 章“删除资源”部分所示，执行`Delete`操作的方法签名如下：

- `Delete(ctx context.Context, name string, opts metav1.DeleteOptions)`

- `GetName()`方法来自`DeleteAction`接口，会返回作为参数传递给`Delete`方法的名称（`name`）

- `GetDeleteOptions()`

- 方法来自`DeleteAction`接口，会返回作为参数传递给`Delete`方法的`opts`

- **`DeleteCollectionAction`接口** – 定义如下：

- `type ListRestrictions struct {`

```go
type DeleteCollectionAction interface {
    Action
    GetListRestrictions() ListRestrictions
}
```

```go
Labels  labels.Selector
Fields  fields.Selector
}
```

如第 6 章“删除资源集合”部分所示，执行`DeleteCollection`操作的方法签名是：

- `DeleteCollection(ctx context.Context, opts metav1.DeleteOptions, listOpts metav1.ListOptions)` – 其中`ListOptions`定义如下：

- `GetListRestrictions()`方法来自`DeleteCollectionAction`接口，会返回通过`ListOptions`结构传递给`DeleteCollection`方法的`LabelSelector`和`FieldSelector`

- **`PatchAction`接口** – 定义如下：

```go
type ListOptions struct {
    LabelSelector string
    FieldSelector string
    [...]
}
```

```go
type PatchAction interface {
    Action
    GetName() string
    GetPatchType() types.PatchType
    GetPatch() []byte
}
```

如第 6 章“使用策略合并补丁更新资源”和“在服务端应用资源”部分所示，执行`Patch`操作的方法签名是：

- `GetName()`方法来自`PatchAction`接口，会返回作为参数传递给`Patch`方法的名称（`name`）

- `GetPatchType()`方法来自`PatchAction`接口，会返回作为参数传递给`Patch`方法的`pt`

- `GetPatch()`方法来自`PatchAction`接口，会返回作为参数传递给`Patch`方法的`data`

- **`WatchAction`接口** – 定义如下：

```go
Patch(
    ctx context.Context,
    name string,
    pt types.PatchType,
    data []byte,
    opts metav1.PatchOptions,
    subresources ...string,
)
```

- `type WatchRestrictions struct {`

```go
type WatchAction interface {
    Action
    GetWatchRestrictions() WatchRestrictions
}
```

- 如第 6 章“监控资源”部分所示，执行`Watch`操作的方法签名是：

- `Watch(ctx context.Context, opts metav1.ListOptions)`

- `GetWatchRestrictions()`方法来自`WatchAction`接口，会返回通过`ListOptions`结构传递给`Watch`方法的`LabelSelector`、`FieldSelector`和`ResourceVersion`

```go
Labels          labels.Selector
Fields          fields.Selector
ResourceVersion string
}
```

以下示例展示了如何使用`Action`和`CreateAction`接口及其方法来测试`CreatePod`函数：

- ➊ 创建一个`fake` `Clientset`
- ➋ 调用`CreatePod`函数进行测试
- ➌ 获取函数执行期间执行的操作
- ➍ 断言操作数量
- ➎ 获取执行期间执行的第一个且唯一的操作
- ➏ 断言操作中传递的命名空间（`namespace`）值
- ➐ 断言操作中使用的动词（`Verb`）和资源（`Resource`）
- ➑ 将`Action`转换为`CreateAction`接口
- ➒ 断言`CreateAction`中传递的对象值

```go
import (
    "context"
    "reflect"
    "testing"
    corev1 "k8s.io/api/core/v1"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/client-go/kubernetes/fake"
    ktesting "k8s.io/client-go/testing"
)
func TestCreatePodActions(t *testing.T) {
    var (
        name      = "a-name"
        namespace = "a-namespace"
        image     = "an-image"
        wantPod = &corev1.Pod{
            ObjectMeta: metav1.ObjectMeta{
                Name: "a-name",
            },
            Spec: corev1.PodSpec{
                Containers: []corev1.Container{
                    {
                        Name:  "runtime",
                        Image: "an-image",
                    },
                },
            },
        }
        wantActions = 1
    )
    clientset := fake.NewSimpleClientset()           // ➊
    _, _ = CreatePod(                                    // ➋
        context.Background(),
        clientset,
        name,
        namespace,
        image,
    )
    actions := clientset.Actions()                     // ➌
    if len(actions) != wantActions {                     // ➍
        t.Errorf("# actions = %d, want %d",
            len(actions),
            wantActions,
        )
    }
    action := actions[0]                               // ➎
    actionNamespace := action.GetNamespace()          // ➏
    if actionNamespace != namespace {
        t.Errorf("action namespace = %s, want %s",
            actionNamespace,
            namespace,
        )
    }
    if !action.Matches("create", "pods") {          // ➐
        t.Errorf("action verb = %s, want create",
            action.GetVerb(),
        )
        t.Errorf("action resource = %s, want pods",
            action.GetResource().Resource,
        )
    }
    createAction := action.(ktesting.CreateAction)     // ➑
    obj := createAction.GetObject()                     // ➒
    if !reflect.DeepEqual(obj, wantPod) {
        t.Errorf("create action object = %v, want %v",
            obj,
            wantPod,
        )
    }
}
```



## 伪造 REST 客户端

在 Client-go 库的 `rest/fake` 包中提供了一个*伪造*的 `RESTClient` 结构体。该结构体定义如下：

```go
import "k8s.io/client-go/rest/fake"
type RESTClient struct {
NegotiatedSerializer runtime.NegotiatedSerializer
GroupVersion         schema.GroupVersion
VersionedAPIPath     string
Err error
Req *http.Request
Client *http.Client
Resp *http.Response
}
```

`NegotiatedSerializer` 字段接受一个 Codec。你需要使用 Client-go 库提供的 Codec，以便能够对来自 Kubernetes API 的资源进行编码和解码。

Client-go 库中的 `RESTClient` 是特定于一个组（Group）和版本（Version）的。你可以通过 `GroupVersion` 字段来指定组和版本。

`VersionedAPIPath` 可用于指定 API 路径的前缀。其他字段则用于模拟请求的结果：

-   如果 `Err` 不为 nil，那么对于使用该 `RESTClient` 的任何调用，都会立即返回此错误。
-   否则，如果 `Client` 不为 nil，则会使用此 `http.Client` 为使用该 `RESTClient` 的任何调用发起 HTTP 请求。
-   再否则，`Resp` 将作为对使用该 `RESTClient` 的任何调用的响应返回。

通过 `Err` 和 `Resp` 字段，你可以硬编码对 `RESTClient` 的调用结果，要么返回特定的错误，要么返回特定的响应。

使用 `Client` 字段，你可以编写一些高级代码，这些代码会根据请求返回特定的结果，或基于请求进行一些计算，等等。例如，下面是一个使用 `RESTClient` 从特定命名空间 `getPods` 的函数：

```go
func getPods(
ctx context.Context,
restClient rest.Interface,
ns string,
) ([]corev1.Pod, error) {
result := corev1.PodList{}
err := restClient.Get().
Namespace(ns).
Resource("pods").
Do(ctx).
Into(&result)
if err != nil {
return nil, err
}
return result.Items, nil
}
```

要从你的代码中调用此函数，你需要从 Clientset 提供一个 `RESTClient`——例如，使用以下代码：

```go
restClient := clientset.CoreV1().RESTClient()
pods, err := getPods(
context.Background(),
restClient,
"default",
)
```

为了评估函数在返回特定值时的行为，你应该改用*伪造*的 `RESTClient`。例如，要断言当 `RESTClient` 调用返回错误时函数的结果，你可以创建这个*伪造的* `RESTClient`：

```go
import (
"context"
"errors"
corev1 "k8s.io/api/core/v1"
"k8s.io/client-go/kubernetes/scheme"
"k8s.io/client-go/rest/fake"
)
restClient := &fake.RESTClient{
GroupVersion:               corev1.SchemeGroupVersion,
NegotiatedSerializer:      scheme.Codecs,
Err: errors.New("an error from the rest client"),
}
pods, err := getPods(
context.Background(),
restClient,
"default",
)
```

另一个例子，要断言当调用返回 NotFound（404）状态时函数的结果，你可以定义以下*伪造的* `RESTClient`：

```go
import (
"net/http"
corev1 "k8s.io/api/core/v1"
"k8s.io/client-go/kubernetes/scheme"
"k8s.io/client-go/rest/fake"
)
restClient := &fake.RESTClient{
GroupVersion:         corev1.SchemeGroupVersion,
NegotiatedSerializer: scheme.Codecs,
Err: nil,
Resp: &http.Response{
StatusCode: http.StatusNotFound,
},
}
```

## FakeDiscovery 客户端

在 Client-go 库的 `discovery/fake` 包中定义了一个 `FakeDiscovery` 结构体。该结构体定义如下：

```go
type FakeDiscovery struct {
*testing.Fake
FakedServerVersion *version.Info
}
```

可以从*伪造的* Clientset 中访问此类 `FakeDiscovery` 客户端的实现：

```go
import (
"k8s.io/client-go/kubernetes/fake"
fakediscovery "k8s.io/client-go/discovery/fake"
)
clientset := fake.NewSimpleClientset()
discoveryClient, ok :=
clientset.Discovery().(*fakediscovery.FakeDiscovery)
if !ok {
t.Fatalf("couldn't convert Discovery() to *FakeDiscovery")
}
```

### 存根 ServerVersion

`FakeDiscovery` 结构体提供了一个 `FakedServerVersion` 字段，你可以用它来模拟由 `discovery.DiscoveryInterface` 的 `ServerVersion()` 方法返回的服务器版本。例如，下面是一个你应该测试的函数，它检查服务器版本是否至少是所提供的版本：

```go
func checkMinimalServerVersion(
clientset kubernetes.Interface,
minMinor int,
) (bool, error) {
discoveryClient := clientset.Discovery()
info, err := discoveryClient.ServerVersion()
if err != nil {
return false, err
}
major, err := strconv.Atoi(info.Major)
if err != nil {
return false, err
}
minor, err := strconv.Atoi(info.Minor)
if err != nil {
return false, err
}
return major == 1 && minor >= minMinor, nil
}
```

现在，你可以在 Discovery 客户端返回特定版本（此处为 1.10）时断言函数的结果。

作为练习，你可以使用表驱动测试重写此测试函数，以便能够测试函数针对各种服务器版本值的结果。

```go
func Test_getServerVersion(t *testing.T) {
client := fake.NewSimpleClientset()
fakeDiscovery, ok := client.Discovery().(*fakediscovery.FakeDiscovery)
if !ok {
t.Fatalf("couldn't convert Discovery() to *FakeDiscovery")
}
fakeDiscovery.FakedServerVersion = &version.Info{
Major: "1",
Minor: "10",
}
res, err := checkMinimalServerVersion(client, 20)
if res != true && err != nil {
t.Error(err)
}
}
```

### 动作

*伪造的* Discovery 客户端的方法 `ServerVersion`、`ServerGroups`、`ServerResourcesForGroupVersion` 和 `ServerGroupsAndResources` 都使用了*伪造的* Clientset 所使用的*动作（Actions）*概念，如本章的“对动作做出反应”和“检查动作”部分所述。

为了评估你的函数对这些方法的使用，你可以通过添加反应器或检查执行的动作，来断言 Discovery 客户端执行的动作。请注意，这些方法使用的动词（Verb）和资源（Resource）是：

-   `ServerVersion` – `"get"`, `"version"`
-   `ServerGroups` – `"get"`, `"group"`
-   `ServerResourcesForGroupVersion` – `"get"`, `"resource"`
-   `ServerGroupsAndResources` – `"get"`, `"group"` 和 `"get"`, `"resource"`

### 模拟资源

对于真实实现，`ServerGroups`、`ServerResourcesForGroupVersion` 和 `ServerGroupsAndResources` 方法依赖于 API 服务器定义的组和资源。

对于*伪造的*实现，这些方法依赖于*伪造的* Clientset 的 `Fake` 字段中的 `Resources` 字段。要使用各种组和资源值来模拟这些方法的结果，你可以在调用这些方法之前填充此 `Resources` 字段。

## 结论

本章展示了如何使用 Clientset、REST 客户端和 Discovery 客户端的*伪造的*实现；所有这些都有助于为使用 Client-go 库的函数编写单元测试。

这结束了本书的第一部分，该部分涵盖了用于处理原生 Kubernetes 资源的库。在接下来的章节中，你将学习如何通过创建自定义资源来扩展 Kubernetes API。



