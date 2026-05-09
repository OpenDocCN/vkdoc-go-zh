# 10. 使用 `controller-runtime` 库编写操作器

正如你在第 1 章所见，*控制器管理器* 是 Kubernetes 架构的重要组成部分。它嵌入了多个*控制器*，每个控制器的职责是监视特定高层资源（如 Deployment）的实例，并使用低层资源（如 Pod）来实现这些高层实例。

例如，Kubernetes 用户在部署无状态应用时可以创建 Deployment。该 Deployment 定义了一个用于在集群中创建 Pod 的*Pod 模板*及若干规范。以下是最重要的规范：

- *副本数*：控制器必须为单个 Deployment 实例部署的相同 Pod 数量。
- *部署策略*：当 Pod 模板更新时，替换 Pod 的方式。
- *默认策略*：*滚动更新*可以在不中断服务的情况下将应用更新到新版本，并支持多种参数。还有一种更简单的策略*重建*，会先停止当前 Pod，再启动替代 Pod。

*Deployment 控制器*将为每个 Pod 模板新版本创建 `ReplicaSet` 资源实例，并更新这些 `ReplicaSet` 的副本数，以满足副本数和策略规范。你会看到已退役版本对应的 `ReplicaSet` 副本数为零，而当前应用版本对应的 `ReplicaSet` 有正数副本。在*滚动更新*期间，两个 `ReplicaSet` 都会拥有正数副本（新版本副本数递增，旧版本副本数递减），从而实现在两个版本间无缝过渡，而不中断服务。

另一方面，*ReplicaSet 控制器*负责维护由*Deployment 控制器*创建的每个 `ReplicaSet` 实例所需的 Pod 副本数量。

第 8 章已经展示了如何定义新的 Kubernetes 资源来扩展 Kubernetes API。尽管原生控制器管理器运行着处理原生 Kubernetes 资源的控制器，但你仍需编写控制器来处理自定义资源。

通常，这类控制器被称为*操作器*，用于处理第三方资源，而将**控制器**这一名称保留给处理原生 Kubernetes 资源的控制器。

`Client-go` 库提供了使用 Go 语言编写控制器和操作器的工具，而 `controller-runtime` 库则基于这些工具，围绕控制器模式提供了抽象层，帮助你编写操作器。你可以通过以下命令安装该库：

```
go get sigs.k8s.io/controller-runtime@v0.13.0
```

你可以从源码仓库 `github.com/kubernetes-sigs/controller-runtime/releases` 获取可用版本。

## 管理器

`controller-runtime` 库提供的第一个重要抽象是*管理器*，它为管理器内运行的所有控制器提供共享资源，包括：

- 用于读写 Kubernetes 资源的 Kubernetes 客户端
- 用于从本地缓存读取 Kubernetes 资源的缓存
- 用于注册所有 Kubernetes 原生和自定义资源的 Scheme

要创建*管理器*，你需要使用提供的 `New` 函数，如下所示：

- ❶ 解析命令行标志，例如使用 `GetConfigOrDie` 处理 `--kubeconfig` 标志；参见下文。

```
import (
"flag"
"sigs.k8s.io/controller-runtime/pkg/client/config"
"sigs.k8s.io/controller-runtime/pkg/manager"
)
flag.Parse()                        ❶
mgr, err := manager.New(
config.GetConfigOrDie(),
manager.Options{},
)
```

第一个参数是 `rest.Config` 对象，如第 6 章“连接到集群”部分所述。请注意，此示例选择了 `controller-runtime` 库提供的 `GetConfigOrDie()` 工具函数，而非使用 `Client-go` 库中的函数。

`GetConfigOrDie()` 函数会尝试获取连接到集群的配置：

- 如果定义了 `--kubeconfig` 标志，则获取其值并读取该路径下的 `kubeconfig` 文件。为此，首先需要执行 `flag.Parse()`
- 如果定义了 `KUBECONFIG` 环境变量，则获取其值并读取该路径下的 `kubeconfig` 文件
- 如果定义了集群内配置（参见第 6 章的“集群内配置”部分），则使用该配置
- 读取 `$HOME/.kube/config` 文件

如果上述方法均不可行，该函数将使程序以代码 `1` 退出。第二个参数是一个选项结构体。

一个重要的选项是 `Scheme`。默认情况下，如果你未为此选项指定任何值，将使用 `Client-go` 库提供的 `Scheme`。如果控制器只需访问原生 Kubernetes 资源，这已经足够。然而，如果你希望控制器访问自定义资源，则需要提供一个能解析该自定义资源的 Scheme。

例如，如果你希望控制器访问第 9 章中定义的自定义资源，则需要在初始化时执行以下代码：

- ❶ 创建一个新的空 Scheme
- ❷ 使用 `Client-go` 库添加原生 Kubernetes 资源
- ❸ 将 `mygroup/v1alpha1` 中包含自定义资源的资源添加到 Scheme
- ❹ 在此管理器中使用的 Scheme

```
import (
"k8s.io/apimachinery/pkg/runtime"
clientgoscheme "k8s.io/client-go/kubernetes/scheme"
mygroupv1alpha1 "github.com/myid/myresource-crd/pkg/apis/mygroup.example.com/v1alpha1"
)
scheme := runtime.NewScheme()                  ❶
clientgoscheme.AddToScheme(scheme)             ❷
mygroupv1alpha1.AddToScheme(scheme)            ❸
mgr, err := manager.New(
config.GetConfigOrDie(),
manager.Options{
Scheme: scheme,                    ❹
},
)
```



## 控制器

第二个重要的抽象是 `Controller`。控制器负责实现特定 Kubernetes 资源实例所给定的规范（Spec）。（在 Operator 场景下，自定义资源由 Operator 处理。）

为此，控制器会监视特定资源（至少是关联的自定义资源，本节中称之为“主资源”），并接收这些资源的 `Watch` 事件（即创建、更新、删除）。当资源上发生事件时，控制器会将包含受事件影响的“主资源”实例的名称和命名空间的 `Request` 放入队列。

请注意，入队的对象仅限于 Operator 所监视的主资源实例。如果事件是由其他资源的实例接收的，则通过跟踪 `ownerReference` 来查找主资源。例如，Deployment 控制器会监视 Deployment 资源和 `ReplicaSet` 资源。所有 `ReplicaSet` 实例都包含指向某个 Deployment 实例的 `ownerReference`。

-   当创建 Deployment 时，控制器会收到一个 `Create` 事件，并且刚创建的 Deployment 实例会被入队。
-   当 `ReplicaSet` 被修改时（例如被某个用户），此 `ReplicaSet` 会收到一个 `Update` 事件，控制器会使用该 `ReplicaSet` 中包含的 `ownerReference` 查找被更新的 `ReplicaSet` 所引用的 Deployment。然后，该引用的 Deployment 实例会被入队。

`Controller` 实现了一个 `Reconcile` 方法，每当队列中有可用的 Request 时，就会调用此方法。此 `Reconcile` 方法接收 `Request` 作为参数，其中包含待协调的主资源的名称和命名空间。

请注意，触发该请求的事件并不属于请求的一部分，因此 `Reconcile` 方法不能依赖此信息。此外，如果事件发生在被拥有的资源上，只有主资源会被入队，`Reconcile` 方法不能依赖是哪个被拥有的资源触发了该事件。

同时，由于同一主资源在短时间内可能发生多个事件，因此请求可以被批量处理，以限制入队的请求数量。

### 创建控制器

要创建一个控制器，你需要使用提供的 `New` 函数：

```
import (
    "sigs.k8s.io/controller-runtime/pkg/controller"
)
controller, err = controller.New(
    "my-operator", mgr,
    controller.Options{
        Reconciler: myReconciler,
    })
```

`Reconciler` 选项是必需的，其值是一个必须实现 `Reconciler` 接口的对象，该接口定义如下：

```
type Reconciler interface {
    Reconcile(context.Context, Request) (Result, error)
}
```

为了方便起见，提供了 `reconcile.Func` 类型，它实现了 `Reconciler` 接口，并且与具有与 `Reconcile` 方法相同签名的函数类型相同。

```
type Func func(context.Context, Request) (Result, error)
func (r Func) Reconcile(ctx context.Context, o Request) (Result, error) { return r(ctx, o) }
```

借助这个 `reconcile.Func` 类型，你可以将一个具有 Reconcile 签名的函数进行类型转换，并将其赋值给 `Reconciler` 选项。例如：

```
controller, err = controller.New(
    "my-operator", mgr,
    controller.Options{
        Reconciler: reconcile.Func(reconcileFunction),
    })
func reconcileFunction(
    ctx context.Context,
    r reconcile.Request,
) (reconcile.Result, error) {
    [...]
    return reconcile.Result{}, nil
}
```

### 监视资源

创建控制器后，你需要向该容器指明要监视哪些资源，以及这些资源是主资源还是被拥有的资源。

使用控制器的 `Watch` 方法来添加一个监视。该方法的定义如下：

```
Watch(
    src source.Source,
    eventhandler handler.EventHandler,
    predicates ...predicate.Predicate,
) error
```

第一个参数指明要监视的事件的来源，其类型是 `source.Source` 接口。`controller-runtime` 库为 `Source` 接口提供了两种实现：

-   `Kind` 来源用于监视特定类型 Kubernetes 对象的事件。`Kind` 结构体的 `Type` 字段是必需的，其值是一个所需类型的对象。例如，如果我们想监视 Deployment，则 `src` 参数的值应为：

```
controller.Watch(
    &source.Kind{
        Type: &appsv1.Deployment{},
    },
    ...
```

-   `Channel` 来源用于监视源自集群外部的事件。`Channel` 结构体的 `Source` 字段是必需的，其值是一个发射 `event.GenericEvent` 类型对象的通道。

第二个参数是事件处理器，其类型是 `handler.EventHandler` 接口。`controller-runtime` 库为 `EventHandler` 接口提供了两种实现：

-   `EnqueueRequestForObject` 事件处理器用于控制器处理的主资源。在这种情况下，控制器会将事件所附带的那个对象放入队列。例如，如果控制器是一个处理第 9 章中创建的自定义资源的 operator，你可以这样写：

```
controller.Watch(
    &source.Kind{
        Type: &mygroupv1alpha1.MyResource{},
    },
    &handler.EnqueueRequestForObject{},
)
```

-   `EnqueueRequestForOwner` 事件处理器用于被主资源拥有的资源。`EnqueueRequestForOwner` 的一个字段是必需的：`OwnerType`。此字段的值是主资源类型的对象；控制器会追踪 ownerReferences，直到找到此类型的对象，并将此对象放入队列。

-   例如，如果控制器处理 `MyResource` 主资源并创建 `Pods` 来实现 `MyResource`，它将希望使用此事件处理器监视 `Pod` 资源，并指定一个 `MyResource` 对象作为 `OwnerType`。

-   如果 `IsController` 字段设置为 true，控制器将只考虑 `Controller: true` 的 ownerReferences。

```
controller.Watch(
    &source.Kind{
        Type: &corev1.Pod{},
    },
    &handler.EnqueueRequestForOwner{
        OwnerType: &mygroupv1alpha1.MyResource{},
        IsController: true,
    },
)
```

第三个参数是可选的谓语列表，其类型是 `predicate.Predicate`。`controller-runtime` 库为 `Predicate` 接口提供了几种实现：

-   `Funcs` 是最通用的实现。`Funcs` 结构体定义如下：

-   你可以将 `Funcs` 结构体的实例作为 Predicate 传递给 `Watch` 方法。
-   未定义的字段表示应处理匹配类型的事件。
-   对于非 nil 的字段，将调用与事件匹配的函数（注意，当 source 为 Channel 时，将调用 `GenericFunc`；见前述），如果该函数返回 true，则该事件将被处理。
-   使用 `Predicate` 的这个实现，你可以为每种事件类型定义特定的函数。

-   `func NewPredicateFuncs(`



```go
type Funcs struct {
	// Create 返回 true 表示应处理 Create 事件
	CreateFunc func(event.CreateEvent) bool
	// Delete 返回 true 表示应处理 Delete 事件
	DeleteFunc func(event.DeleteEvent) bool
	// Update 返回 true 表示应处理 Update 事件
	UpdateFunc func(event.UpdateEvent) bool
	// Generic 返回 true 表示应处理 Generic 事件
	GenericFunc func(event.GenericEvent) bool
}
```

`filter func(object client.Object) bool,` `) Funcs`

- 此函数接受一个`filter`函数，并返回一个`Funcs`结构体，该过滤器将应用于所有事件。使用此函数，您可以定义一个适用于所有事件类型的单一过滤器。
- `ResourceVersionChangedPredicate`结构体仅会为`UpdateEvent`定义一个过滤器。
- 使用此谓词，所有`Create`、`Delete`和`Generic`事件将在无过滤的情况下被处理，而`Update`事件将被过滤，仅处理那些带有`metadata.resourceVersion`变更的更新。
- `metadata.resourceVersion`字段在每次保存资源的新版本时都会更新，无论资源发生了何种更改。
- `GenerationChangedPredicate`结构体仅会为`Update`事件定义一个过滤器。
- 使用此谓词，所有`Create`、`Delete`和`Generic`事件将在无过滤的情况下被处理，而`Update`事件将被过滤，仅处理那些带有`metadata.Generation`递增的更新。
- `metadata.Generation`由 API 服务器在每次资源`Spec`部分发生更新时顺序递增。
- 请注意，某些资源并不遵循此假设。例如，`Deployment`的`Generation`在`Annotations`字段更新时也会递增。
- 对于自定义资源，仅当启用了`Status`子资源时，`Generation`才会递增。
- `AnnotationChangedPredicate`结构体仅会为`Update`事件定义一个过滤器。
- 使用此谓词，所有`Create`、`Delete`和`Generic`事件将被处理，而`Update`事件将被过滤，仅处理那些带有`metadata.Annotations`变更的更新。

## 第一个示例

在第一个示例中，您将创建一个管理器和一个控制器。该控制器将管理一个名为`MyResource`的主要自定义资源，并监听此资源以及`Pod`资源。

`Reconcile`函数将仅显示需要调和的`MyResource`实例的命名空间和名称。

- ➊ 创建一个包含原生资源和自定义资源`MyResource`的`Scheme`。
- ➋ 使用刚创建的`Scheme`创建一个管理器。
- ➌ 创建一个附加到管理器的控制器，并传入一个`Reconciler`实现。
- ➍ 开始将`MyResource`实例作为主要资源进行监听。
- ➎ 开始将`Pod`实例作为附属资源进行监听。
- ➏ 启动管理器。此函数是长时间运行的，仅当发生错误时才会返回。
- ➐ 一个实现了`Reconciler`接口的类型。
- ➑ 实现了`Reconcile`方法。该方法将显示需要调和的实例的命名空间和名称。

```go
package main

import (
	"context"
	"fmt"

	corev1 "k8s.io/api/core/v1"
	"k8s.io/apimachinery/pkg/runtime"
	"sigs.k8s.io/controller-runtime/pkg/client/config"
	"sigs.k8s.io/controller-runtime/pkg/controller"
	"sigs.k8s.io/controller-runtime/pkg/handler"
	"sigs.k8s.io/controller-runtime/pkg/manager"
	"sigs.k8s.io/controller-runtime/pkg/reconcile"
	"sigs.k8s.io/controller-runtime/pkg/source"
	clientgoscheme "k8s.io/client-go/kubernetes/scheme"
	mygroupv1alpha1 "github.com/myid/myresource-crd/pkg/apis/mygroup.example.com/v1alpha1"
)

func main() {
	scheme := runtime.NewScheme() // ❶
	clientgoscheme.AddToScheme(scheme)
	mygroupv1alpha1.AddToScheme(scheme)

	mgr, err := manager.New( // ❷
		config.GetConfigOrDie(),
		manager.Options{
			Scheme: scheme,
		},
	)
	panicIf(err)

	controller, err := controller.New( // ❸
		"my-operator", mgr,
		controller.Options{
			Reconciler: &MyReconciler{},
		})
	panicIf(err)

	err = controller.Watch( // ❹
		&source.Kind{
			Type: &mygroupv1alpha1.MyResource{},
		},
		&handler.EnqueueRequestForObject{},
	)
	panicIf(err)

	err = controller.Watch( // ❺
		&source.Kind{
			Type: &corev1.Pod{},
		},
		&handler.EnqueueRequestForOwner{
			OwnerType:    &corev1.Pod{},
			IsController: true,
		},
	)
	panicIf(err)

	err = mgr.Start(context.Background()) // ❻
	panicIf(err)
}

type MyReconciler struct{} // ➐

func (o *MyReconciler) Reconcile( // ➑
	ctx context.Context,
	r reconcile.Request,
) (reconcile.Result, error) {
	fmt.Printf("reconcile %v\n", r)
	return reconcile.Result{}, nil
}

// panicIf panic if err is not nil
// Please call from main only!
func panicIf(err error) {
	if err != nil {
		panic(err)
	}
}
```

## 使用控制器构建器

`controller-runtime`库提供了一个*控制器构建器*，以使控制器的创建更加简洁。

```go
import (
	"sigs.k8s.io/controller-runtime/pkg/builder"
)

func ControllerManagedBy(m manager.Manager) *Builder
```

`ControllerManagedBy`函数用于初始化一个新的`ControllerBuilder`。构建的控制器将被添加到`m`管理器中。一个流式接口有助于配置构建过程：

- `For(object client.Object, opts ...ForOption) *Builder` – 此方法用于指示控制器处理的主要资源。它只能被调用一次，因为一个控制器只能有一个主要资源。这将在内部调用`Watch`函数，并传入事件处理器`EnqueueRequestForObject`。
  - 可以通过`WithPredicates`函数为此监听添加谓词，该函数返回的结果实现了`ForOption`接口。
- `Owns(object client.Object, opts ...OwnsOption) *Builder` – 此方法用于指示控制器拥有的资源。这将在内部调用`Watch`函数，并传入事件处理器`EnqueueRequestForOwner`。
  - 可以通过`WithPredicates`函数为此监听添加谓词，该函数返回的结果实现了`OwnsOption`接口。
- `Watches(src source.Source, eventhandler handler.EventHandler, opts ...WatchesOption) *Builder` – 此方法可用于添加`For`或`Owns`方法未涵盖的更多监听器——例如，使用`Channel`源的监听器。
  - 可以通过`WithPredicates`函数为此监听添加谓词，该函数返回的结果实现了`WatchesOption`接口。
- `WithEventFilter(p predicate.Predicate) *Builder` – 此方法可用于为所有通过`For`、`Owns`和`Watch`方法创建的监听器添加公共谓词。
- `WithOptions(options controller.Options) *Builder` – 此方法设置将内部传递给`controller.New`函数的选项。
- `WithLogConstructor(` `func(*reconcile.Request) logr.Logger,` `) *Builder` – 此方法设置`logConstructor`选项。
- `Named(name string) *Builder` – 此方法设置构造器的名称。它应该只使用下划线和字母数字字符。默认情况下，它是主要资源类型的全小写版本。
- `Build(` `r reconcile.Reconciler,` `) (controller.Controller, error)` – 此方法构建并返回控制器。
- `Complete(r reconcile.Reconciler) error` – 此方法构建控制器。您通常不需要直接访问控制器，因此可以使用此方法（不返回控制器值）来代替`Build`。



### 使用 `ControllerBuilder` 的第二个示例

在此示例中，你将使用 `ControllerBuilder` 构建控制器，而不是使用 `controller.New` 函数和 `Controller` 上的 `Watch` 方法。

```go
package main

import (
	"context"
	"fmt"
	corev1 "k8s.io/api/core/v1"
	"k8s.io/apimachinery/pkg/runtime"
	clientgoscheme "k8s.io/client-go/kubernetes/scheme"
	"sigs.k8s.io/controller-runtime/pkg/builder"
	"sigs.k8s.io/controller-runtime/pkg/client"
	"sigs.k8s.io/controller-runtime/pkg/client/config"
	"sigs.k8s.io/controller-runtime/pkg/manager"
	"sigs.k8s.io/controller-runtime/pkg/reconcile"
	mygroupv1alpha1 "github.com/feloy/myresource-crd/pkg/apis/mygroup.example.com/v1alpha1"
)

func main() {
	scheme := runtime.NewScheme()
	clientgoscheme.AddToScheme(scheme)
	mygroupv1alpha1.AddToScheme(scheme)
	mgr, err := manager.New(
		config.GetConfigOrDie(),
		manager.Options{
			Scheme: scheme,
		},
	)
	panicIf(err)

	err = builder.
		ControllerManagedBy(mgr).
		For(&mygroupv1alpha1.MyResource{}).
		Owns(&corev1.Pod{}).
		Complete(&MyReconciler{})
	panicIf(err)

	err = mgr.Start(context.Background())
	panicIf(err)
}

type MyReconciler struct{}

func (a *MyReconciler) Reconcile(
	ctx context.Context,
	req reconcile.Request,
) (reconcile.Result, error) {
	fmt.Printf("reconcile %v\n", req)
	return reconcile.Result{}, nil
}

func panicIf(err error) {
	if err != nil {
		panic(err)
	}
}
```

### 向 Reconciler 注入 Manager 资源

`Manager` 为控制器提供共享资源，包括用于读写 Kubernetes 资源的客户端（client）、用于从本地缓存读取资源的缓存（cache）以及用于解析资源的 Scheme（scheme）。`Reconcile` 函数需要访问这些共享资源。有两种方式可以实现共享：

#### 在创建 Reconciler 结构体时传递值

创建控制器时，你会传入一个实现了 `Reconciler` 接口的 `Reconcile` 结构体实例：

```go
type MyReconciler struct{}

err = builder.
	ControllerManagedBy(mgr).
	For(&mygroupv1alpha1.MyResource{}).
	Owns(&corev1.Pod{}).
	Complete(&MyReconciler{})
```

Manager 在此之前已经创建，你可以通过 Manager 上的 `Getter` 方法访问共享资源。

例如，以下是如何从新创建的 Manager 中获取客户端、缓存和 Scheme：

```go
mgr, err := manager.New(
	config.GetConfigOrDie(),
	manager.Options{
		manager.Options{
			Scheme: scheme,
		},
	},
)
// 处理错误
mgrClient := mgr.GetClient()
mgrCache := mgr.GetCache()
mgrScheme := mgr.GetScheme()
```

你可以向 `Reconciler` 结构体添加字段来传递这些值：

```go
type MyReconciler struct {
	client client.Client
	cache  cache.Cache
	scheme *runtime.Scheme
}
```

最后，在创建控制器时传递这些值：

```go
err = builder.
	ControllerManagedBy(mgr).
	For(&mygroupv1alpha1.MyResource{}).
	Owns(&corev1.Pod{}).
	Complete(&MyReconciler{
		client: mgr.GetClient(),
		cache:  mgr.GetCache(),
		scheme: mgr.GetScheme(),
	})
```

#### 使用注入器

`controller-runtime` 库提供了一套 *Injectors*（注入器）系统，用于将共享资源注入到 `Reconciler` 中，以及其他结构体（如你自定义的 `Sources`、`EventHandlers` 和 `Predicates` 实现）中。

`Reconciler` 的实现需要实现 `inject` 包中特定的注入器接口：`inject.Client`、`inject.Cache`、`inject.Scheme` 等。

这些方法将在初始化时（即调用 `controller.New` 或 `builder.Complete` 时）被调用。因此需要为每个接口创建一个方法，例如：

```go
type MyReconciler struct {
	client client.Client
	cache  cache.Cache
	scheme *runtime.Scheme
}

func (a *MyReconciler) InjectClient(
	c client.Client,
) error {
	a.client = c
	return nil
}

func (a *MyReconciler) InjectCache(
	c cache.Cache,
) error {
	a.cache = c
	return nil
}

func (a *MyReconciler) InjectScheme(
	s *runtime.Scheme,
) error {
	a.scheme = s
	return nil
}
```

## 使用客户端（Client）

客户端可用于读写集群上的资源，以及更新资源的状态。

`Read` 方法内部使用基于 *Informers* 和 *Listers* 的缓存系统，以限制对 API Server 的读取访问。通过使用此缓存，同一 Manager 下的所有控制器都可以访问资源，同时限制对 API Server 的请求。

你必须注意，读取操作返回的对象是指向缓存中值的指针。**切勿**直接修改这些对象。相反，在修改之前，你必须创建返回对象的深度副本。

客户端的方法是通用的：它们可以处理任何 Kubernetes 资源，无论是原生资源还是自定义资源，只要这些资源在传递给 Manager 的 Scheme 中已注册。

所有方法都会返回一个错误，其类型与 `Client-go` 的 ClientSet 方法返回的错误类型相同。更多信息请参考第 6 章的“错误和状态”部分。

### 获取资源信息

`Get` 方法用于获取资源的信息。

```go
Get(
	ctx context.Context,
	key ObjectKey,
	obj Object,
	opts ...GetOption,
) error
```

它接受一个 `ObjectKey` 值作为参数，用于指定资源的命名空间和名称；以及一个 `Object` 参数，用于指示要获取资源的 *Kind*（种类）并存储结果。`Object` 必须是指向类型化资源的指针——例如 `Pod` 或 `MyResource` 结构体。`ObjectKey` 类型是 `types.NamespacedName` 的别名，定义于 API Machinery 库中。

`NamespacedName` 也是作为参数传递给 `Reconcile` 函数的 `Request` 中嵌入的对象类型。你可以直接使用 `req.NamespacedName` 作为 `ObjectKey` 来获取需要调协的资源。例如，使用以下代码获取需要调协的资源：

```go
myresource := mygroupv1alpha1.MyResource{}
err := a.client.Get(
	ctx,
	req.NamespacedName,
	&myresource,
)
```

可以在 `Get` 请求中传递特定的 `resourceVersion` 值，只需将 `client.GetOptions` 结构体实例作为最后一个参数传入。

`GetOptions` 结构体实现了 `GetOption` 接口，包含一个单独的 `Raw` 字段，其值为 `metav1.GetOptions`。例如，指定 `resourceVersion` 为 "0" 以获取资源的任何版本：

```go
err := a.client.Get(
	ctx,
	req.NamespacedName,
	&myresource,
	&client.GetOptions{
		Raw: &metav1.GetOptions{
			ResourceVersion: "0",
		},
	},
)
```



### 列出资源

`List` 方法用于列出特定类型的资源。

```
List(
ctx context.Context,
list ObjectList,
opts ...ListOption,
) error
```

`list` 参数是一个 `ObjectList` 类型的值，用于指示要列出的资源的*种类*并存储结果。默认情况下，列表操作将跨所有命名空间进行。

`List` 方法接受零个或多个实现了 `ListOption` 接口的对象参数。支持以下这些类型：

*   `InNamespace`，`string` 的别名，用于返回特定命名空间的资源。

*   `MatchingLabels`，`map[string]string` 的别名，用于指定为返回资源而必须精确匹配的标签列表及其值。以下示例构建了一个 `MatchingLabels` 结构，用于过滤带有标签 `app=myapp` 的资源。

*   `HasLabels`，`[]string` 的别名，用于指定为返回资源而必须存在的标签列表，不考虑其值。以下示例构建了一个 `HasLabels` 结构，用于过滤带有 `app` 和 `debug` 标签的资源。

```
matchLabel := client.MatchingLabels{
"app": "myapp",
}
```

*   `MatchingLabelsSelector`，内嵌了 `labels.Selector` 接口，用于传递更高级的标签选择器。关于如何构建 `Selector` 的更多信息，请参阅第 6 章中的“过滤列表结果”一节。以下示例构建了一个 `MatchingLabelsSelector` 结构，该结构可作为 `List` 的选项，用于过滤那些标签 `mykey` 不等于 `ignore` 的资源。

```
hasLabels := client.HasLabels{"app", "debug"}
```

*   `MatchingFields`，`fields.Set` 的别名（其本身是 `map[string]string` 的别名），用于指定要匹配的字段及其值。以下示例构建了一个 `MatchingFields` 结构，用于过滤字段 `status.phase` 为 `Running` 的资源：

```
selector := labels.NewSelector()
require, err := labels.NewRequirement(
"mykey",
selection.NotEquals,
[]string{"ignore"},
)
// 断言 err 为 nil
selector = selector.Add(*require)
labSelOption := client.MatchingLabelsSelector{
Selector: selector,
}
```

*   `MatchingFieldsSelector`，内嵌了 `fields.Selector`，用于传递更高级的字段选择器。关于如何构建 `fields.Selector` 的更多信息，请参阅第 6 章中的“过滤列表结果”一节。以下示例构建了一个 `MatchingFieldsSelector` 结构，用于过滤字段 `status.phase` 不等于 `Running` 的资源：

```
matchFields := client.MatchingFields{
"status.phase": "Running",
}
```

*   `Limit`，`int64` 的别名，和 `Continue`，`string` 的别名，用于对结果进行分页。这些选项在第 2 章的“对结果进行分页”一节中有详细说明。

```
fieldSel := fields.OneTermNotEqualSelector(
"status.phase",
"Running",
)
fieldSelector := client.MatchingFieldsSelector{
Selector: fieldSel,
}
```

## 创建资源

`Create` 方法用于在集群中创建新资源。

```
Create(
ctx context.Context,
obj Object,
opts ...CreateOption,
) error
```

作为参数传递的 `obj` 定义了要创建对象的种类及其定义。以下示例将在集群中创建一个 Pod：

```
podToCreate := corev1.Pod{ [...] }
podToCreate.SetName("nginx")
podToCreate.SetNamespace("default")
err = a.client.Create(ctx, &podToCreate)
```

以下选项可以作为 `CreateOption` 传递，以参数化 `Create` 请求。

*   `DryRunAll` 值表示应执行所有操作，但将资源持久化到存储的操作除外。

*   `FieldOwner`，`string` 的别名，表示 `Create` 操作的字段管理器名称。此信息对于服务器端应用操作的正确工作很有用。

## 删除资源

`Delete` 方法用于从集群中删除资源。

```
Delete(
ctx context.Context,
obj Object,
opts ...DeleteOption,
) error
```

作为参数传递的 `obj` 定义了要删除的对象的种类，以及其命名空间（如果资源是命名空间范围的）和名称。以下示例可用于删除一个 Pod。

```
podToDelete := corev1.Pod{}
podToDelete.SetName("nginx")
podToDelete.SetNamespace("prj2")
err = a.client.Delete(ctx, &podToDelete)
```

以下选项可以作为 `DeleteOption` 传递，以参数化 `Delete` 请求。

*   `DryRunAll` – 此值表示应执行所有操作，但将资源持久化到存储的操作除外。

*   `GracePeriodSeconds`，`int64` 的别名 – 此值仅在删除 Pod 时有用。它表示在 Pod 被删除之前的持续时间（以秒为单位）。详见第 6 章中的“删除资源”一节。

*   `Preconditions`，`metav1.Preconditions` 的别名 – 它指示你期望删除哪个资源。详见第 6 章中的“删除资源”一节。

*   `PropagationPolicy`，`metav1.DeletionPropagation` 的别名 – 它指示是否以及如何执行垃圾回收。详见第 6 章中的“删除资源”一节。

## 删除资源集合

`DeleteAllOf` 方法用于从集群中删除指定类型的所有资源。

```
DeleteAllOf(
ctx context.Context,
obj Object,
opts ...DeleteAllOfOption,
) error
```

作为参数传递的 `obj` 定义了要删除对象的种类。

`DeleteAllOf` 操作可用的 `opts` 是 `List` 操作（参见“列出资源”一节）和 `Delete` 操作（参见“删除资源”一节）的选项组合。

例如，以下是如何删除给定命名空间中的所有部署：

```
err = a.client.DeleteAllOf(
ctx,
&appsv1.Deployment{},
client.InNamespace(aNamespace))
```

## 更新资源

`Update` 方法用于更新集群中已有的资源。

```
Update(
ctx context.Context,
obj Object,
opts ...UpdateOption,
) error
```

`obj` 参数用于指示要更新的资源及其新定义。如果该对象在集群中不存在，则 `Update` 操作会失败。

以下选项可以作为 `UpdateOption` 传递，以参数化 `Update` 请求。

*   `DryRunAll` 值表示应执行所有操作，但将资源持久化到存储的操作除外。

*   `FieldOwner`，`string` 的别名，表示 `Update` 操作的字段管理器名称。此信息对于服务器端应用操作的正确工作很有用。

## 修补资源

`Patch` 方法用于修补现有资源。它也可以用于运行服务器端应用操作。

```
Patch(
ctx context.Context,
obj Object,
patch Patch,
opts ...PatchOption,
) error
```

以下选项可以作为 `PatchOption` 传递，以参数化 `Patch` 请求。

*   `DryRunAll` 值表示应执行所有操作，但将资源持久化到存储的操作除外。

*   `FieldOwner`，`string` 的别名，表示 `Patch` 操作的字段管理器名称。

*   `ForceOwnership`，`struct{}` 的别名，表示调用者将重新获取其他管理器拥有的冲突字段。此选项仅在补丁类型为 `Apply` 时有效（参见下文）。


好的，作为一名高级文档工程师和翻译员，我将严格遵循您提供的格式要求和注意事项，将给定的英文文本翻译成中文。


### 服务端应用（Server-side Apply）

要使用服务端应用操作，你需要指定一个值为 `client.Apply` 的 `patch`。`obj` 值必须是新的对象定义。它必须包含对象的名称、命名空间（如果资源是按命名空间划分的）、组、版本和类型。

字段管理器的名称是必需的，必须通过选项 `client.FieldOwner(name)` 来指定。例如，以下是服务端应用一个 Deployment 的方法：

```
deployToApply := appsv1.Deployment{ [...] }
deployToApply.SetName("nginx")
deployToApply.SetNamespace("default")
deployToApply.SetGroupVersionKind(
appsv1.SchemeGroupVersion.WithKind("Deployment"),
)
err = a.client.Patch(
ctx,
&deployToApply,
client.Apply,
client.FieldOwner("mycontroller"),
client.ForceOwnership,
)
```

### 策略合并补丁（Strategic Merge Patch）

你在第 2 章的“使用策略合并补丁更新资源”一节中已经了解了如何使用策略合并补丁来修补资源。第 6 章的“使用策略合并补丁更新资源”一节展示了如何使用 `Client-go` 库执行此操作。

使用 `Client-go` 时，你必须首先从集群中读取一个资源（通过 `Get` 或 `List` 操作），然后使用此资源上的 `StrategicMergeFrom` 函数创建一个补丁，接着更新资源，最后调用补丁上的 `Data` 方法来计算 JSON 补丁数据。

使用 `controller-runtime` 客户端时，部分处理过程由客户端完成。你仍然需要从集群中读取资源（通过 `Get` 或 `List` 操作），使用 `StrategicMergeFrom` 函数创建一个补丁，然后更新资源。此时，你可以调用 `Patch` 方法，将更新后的对象和补丁作为参数传入。对 `Data` 方法的调用将在客户端内部完成。

请注意，此处使用的 `StrategicMergeFrom` 函数与 `Client-go` 库中使用的函数相同。你可以参考第 6 章的“使用策略合并补丁更新资源”一节，了解此方法的可用选项。

例如，以下是向 Deployment 的 `Pod` 模板的第一个容器添加一个环境变量的方法。

- ❶ 从集群中获取要修补的 Deployment
- ❷ 从该 Deployment 的一个副本创建一个 `Patch` 对象
- ❸ 修改 Deployment
- ❹ 使用修改后的 Deployment 和 `Patch` 对象执行 `Patch` 操作

```
var deploymentRead appsv1.Deployment
err = a.client.Get(                                                ❶
ctx,
key, // 一个用于定义命名空间和名称的 ObjectKey
&deploymentRead)
if err != nil {
return reconcile.Result{}, err
}
patch :=
client.StrategicMergeFrom(deploymentRead.DeepCopy())      ❷
depModified := deploymentRead.DeepCopy()
depModified.Spec.Template.Spec.Containers[0].Env =      ❸
append(
depModified.Spec.Template.Spec.Containers[0].Env,
corev1.EnvVar{
Name:  "newName",
Value: "newValue",
})
err = a.client.Patch(ctx, &depModified, patch)            ❹
```

### 合并补丁（Merge Patch）

合并补丁的工作方式与策略合并补丁类似，区别在于列表的合并方式。有关策略合并补丁如何合并列表的更多信息，请参见第 2 章的“修补数组字段”一节。对于合并补丁，不会考虑原始列表，新列表就是补丁中定义的列表。

使用 `controller-runtime` 库时，你只需在代码清单的步骤 ❷ 中将调用 `client.StrategicMergeFrom` 替换为调用 `client.MergeFrom`，即可指示执行合并补丁。

```
patch := client.MergeFrom(deploymentRead.DeepCopy())      ❷
```

## 更新资源的状态（Status）

在开发 Operator 时，你可能需要修改自定义资源（Custom Resource）的 Status 部分的值，以指示其当前状态。

```
Update(
ctx context.Context,
obj Object,
opts ...UpdateOption,
) error
```

请注意，你不需要为 Status 执行特定的 `Create`、`Get` 或 `Delete` 操作，因为当你创建自定义资源本身时，Status 会作为自定义资源的一部分一起创建。此外，对资源的 `Get` 操作会返回资源的 Status 部分，而删除资源也会删除其 Status 部分。

然而，API 服务器强制你为 Status 和资源的其余部分使用不同的 `Update` 方法，以防止你误操作同时修改这两个部分。

要使 Status 的更新生效，自定义资源定义（Custom Resource Definition）必须在其子资源列表中声明 `status` 字段。请参阅第 8 章的“资源版本定义”一节，了解如何声明此 `status` 子资源。

此方法的工作方式与资源本身的 `Update` 方法类似，但要调用它，你需要在 `client.Status()` 返回的对象上调用该方法：

```
err = client.Status().Update(ctx, obj)
```

### 修补资源的状态

与 `Update` 方法一样，修补资源的状态需要在 `client.Status()` 的结果上调用一个专用方法。

```
Patch(
ctx context.Context,
obj Object,
patch Patch,
opts ...PatchOption,
) error
```

用于 Status 的 `Patch` 方法与资源本身的 `Patch` 方法工作方式相同，区别在于它只会修补资源的 Status 部分。

```
err = client.Status().Patch(ctx, obj, patch)
```

## 日志记录（Logging）

Manager 利用 `controller-runtime` 库的 `log` 包中定义的日志系统。必须通过调用 `SetLogger` 来初始化日志记录器。之后，就可以使用 `log` 包中定义的 `Log` 变量。

```
import (
crlog "sigs.k8s.io/controller-runtime/pkg/log"
"sigs.k8s.io/controller-runtime/pkg/log/zap"
)
func main() {
log.SetLogger(zap.New())
log.Log.Info("starting")
[...]
}
```

`Log` 对象的类型是 `logr.Logger`（来自 `github.com/go-logr/logr`），它有两个主要方法：`Info` 和 `Error`。

```
func (l Logger) Info(
msg string,
keysAndValues ...interface{},
)
func (l Logger) Error(
err error,
msg string,
keysAndValues ...interface{},
)
```

`Logger` 是一个“结构化”的日志系统，其日志主要由键值对组成，而不是 `printf` 风格的格式化字符串。`printf` 风格的日志更易于人类阅读，而结构化日志更易于工具分析。日志条目的预定义键如下：

-   `level`：日志的详细程度级别；请参阅下一节了解日志详细程度的定义
-   `ts`：日志条目的时间戳
-   `logger`：此条目的日志记录器名称；请参阅后续部分了解日志记录器名称的定义
-   `msg`：与日志条目关联的消息
-   `error`：与日志条目关联的错误

`Info` 和 `Error` 方法接受一条消息以及零个或多个键值对。`Error` 方法还接受一个错误值。

### 详细程度（Verbosity）

可以通过使用 `log.V(n).Info(...)` 为 Info 消息指定详细程度。`n` 的详细程度值越大，消息的重要性越低。`log.Info(...)` 等同于 `log.V(0).Info(...)`。

### 预定义值（Predefined Values）

你可以通过使用 `WithValues` 构建一个带有预定义键值对的新日志记录器：

```
func (l Logger) WithValues(
keysAndValues ...interface{},
) Logger
ctrlLog := log.Log.WithValues("package", "controller")
```

### 日志记录器名称（Logger Name）

你可以通过使用 `WithName` 为日志记录器名称追加一个名称来构建一个新的日志记录器：

```
func (l Logger) WithName(name string) Logger
```

连续调用 `WithName` 会向日志记录器名称添加后缀。例如，以下代码将设置日志记录器名称为 `controller.main`：

```
ctrlLog := log.Log.WithName("controller")
[...]
ctrlMainLog = ctrlLog.Log.WithName("main")
```



### 从上下文中获取日志记录器

管理器会将一个日志记录器添加到传递给不同方法（特别是 `Reconcile` 函数）的 `Context` 中。

你可以使用 `FromContext` 函数从 `Context` 中提取日志记录器。可选的参数可用于向日志记录器中添加预定义的键值对：

```go
func FromContext(
ctx context.Context,
keysAndValues ...interface{},
) logr.Logger {
func (a *MyReconciler) Reconcile(
ctx context.Context,
req reconcile.Request,
) (reconcile.Result, error) {
log := log.FromContext(ctx).WithName("reconcile")
[...]
}
```

## 事件

Kubernetes API 提供了一个 `Event` 资源，`Event` 实例会附加到任何特定类型的实例上。事件由控制器发送，用于通知用户某个与对象相关的事件已发生。这些事件在执行 `kubectl describe` 时会显示出来。例如，你可以看到与 Pod 执行相关的事件：

```bash
$ kubectl describe pods nginx
[...]
Events:
Type    Reason     Age        From               Message
----    ------     ----       ----               -------
Normal  Scheduled  1s         default-scheduler  Successfully assigned...
Normal  Pulling    0s         kubelet            Pulling image "nginx"
Normal  Pulled       kubelet            Successfully pulled...
Normal  Created      kubelet            Created container nginx
Normal  Started      kubelet            Started container nginx
```

要从 `Reconcile` 函数发送此类事件，你需要能够访问由管理器提供的 `EventRecorder` 实例。在初始化期间，你可以通过管理器的 `GetEventRecorderFor` 方法获取此实例，然后在构建 `Reconcile` 结构体时传递这个值：

```go
type MyReconciler struct {
client        client.Client
EventRecorder record.EventRecorder
}
func main() {
[...]
eventRecorder := mgr.GetEventRecorderFor(
"MyResource",
)
err = builder.
ControllerManagedBy(mgr).
Named(controller.Name).
For(&mygroupv1alpha1.MyResource{}).
Owns(&appsv1.Deployment{}).
Complete(&controller.MyReconciler{
EventRecorder: eventRecorder,
})
[...]
}
```

然后，在 `Reconcile` 函数中，你可以调用 `Event`、`Eventf` 和 `AnnotatedEventf` 方法。

```go
func (record.EventRecorder) Event(
object runtime.Object,
eventtype, reason, message string,
)
func (record.EventRecorder) Eventf(
object runtime.Object,
eventtype, reason, messageFmt string,
args ...interface{},
)
func (record.EventRecorder) AnnotatedEventf(
object runtime.Object,
annotations map[string]string,
eventtype, reason, messageFmt string,
args ...interface{},
)
```

`object` 参数指明事件要附加到哪个对象上。你需要传递正在被调和的**自定义资源**。

`eventtype` 参数接受两个值，`corev1.EventTypeNormal` 和 `corev1.EventTypeWarning`。

`reason` 参数是一个采用 `UpperCamelCase` 格式的短值。`message` 参数是一个人类可读的文本。

`Event` 方法用于传递静态消息，`Eventf` 方法可以使用 `Sprintf` 创建消息，而 `AnnotatedEventf` 方法还可以向事件附加注解。

## 结论

在本章中，你使用了 `controller-runtime` 库来开始创建一个运算符。你已经了解了如何使用适当的方案创建管理器，如何创建控制器并声明 `Reconcile` 函数，并且探索了该库提供的客户端用于访问集群中资源的各种功能。

下一章将探讨如何编写 `Reconcile` 函数。

