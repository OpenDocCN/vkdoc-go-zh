# 12. 测试调谐循环

前一章描述了如何为调谐自定义资源的 Operator 实现一个简单但完整的 `Reconcile` 函数。

为了测试你在前一章编写的 `Reconcile` 函数，你将使用 `ginkgo`（一个用于 Go 的测试框架）以及 `controller-runtime` 库中的 `envtest` 包，后者提供了一个用于测试的 Kubernetes 环境。

## envtest 包

`controller-runtime` 库提供了一个 `envtest` 包。该包通过启动一个简单的本地控制平面来提供 Kubernetes 环境。

默认情况下，该包使用位于 `/usr/local/kubebuilder/bin` 的 `etcd` 和 `kube-apiserver` 本地二进制文件，你也可以提供自己的路径来查找这些二进制文件。你可以安装 `setup-envtest` 来获取不同 Kubernetes 版本的这些二进制文件。

### 安装 envtest 二进制文件

`setup-envtest` 工具可用于安装 `envtest` 使用的二进制文件。要安装该工具，你必须运行：

```
$ go install sigs.k8s.io/controller-runtime/tools/setup-envtest@latest
```

然后，你可以使用以下命令安装特定 Kubernetes 版本的二进制文件：

```
$ setup-envtest use 1.23
Version: 1.23.5
OS/Arch: linux/amd64
Path: /path/to/kubebuilder-envtest/k8s/1.23.5-linux-amd64
```

命令的输出将告知你二进制文件安装的目录。如果你想从默认目录 `/usr/local/kubebuilder/bin` 使用这些二进制文件，可以创建一个符号链接来访问它们：

```
$ sudo mkdir /usr/local/kubebuilder
$ sudo ln -s /path/to/kubebuilder-envtest/k8s/1.23.5-linux-amd64 /usr/local/kubebuilder/bin
```

或者，如果你更倾向于使用 `KUBEBUILDER_ASSETS` 环境变量来定义包含二进制文件的目录，可以执行：

```
$ source <(setup-envtest use -i -p env 1.23.5)
$ echo $KUBEBUILDER_ASSETS
/path/to/kubebuilder-envtest/k8s/1.23.5-linux-amd64
```

这将定义并导出 `KUBEBUILDER_ASSETS` 变量，其值指向包含 Kubernetes 1.23.5 二进制文件的路径。

### 使用 envtest

控制平面只会运行 API Server 和 `etcd`，但不会运行任何控制器。这意味着当你想要测试的 Operator 创建 Kubernetes 资源时，不会有任何控制器做出响应。例如，如果 Operator 创建了一个 Deployment，不会有 Pod 被创建，且 Deployment 的状态永远不会更新。

这起初可能会让人感到惊讶，但这将有助于你只测试自己的 Operator，而不测试 Kubernetes 控制器。要创建测试环境，你首先需要创建一个 `envtest.Environment` 结构体的实例。

使用 `Environment` 结构体的默认值将启动一个本地控制平面，该控制平面会使用 `/usr/local/kubebuilder/bin` 或 `KUBEBUILDER_ASSETS` 环境变量所定义目录中的二进制文件。

如果你正在编写一个调谐自定义资源的 Operator，则需要为该自定义资源添加自定义资源定义 (CRD)。为此，你可以使用 `CRDDirectoryPaths` 字段来传递包含 YAML 或 JSON 格式 CRD 定义的目录列表。在初始化环境时，所有这些定义都将被应用到本地集群中。

`ErrorIfCRDPathMissing` 字段在你希望在 CRD 目录不存在时收到警告时非常有用。以下是一个示例，展示了如何创建 `Environment` 结构体，其中 CRD 的 YAML 或 JSON 文件位于 `../../crd` 目录中：

```
import (
"path/filepath"
"sigs.k8s.io/controller-runtime/pkg/envtest"
)
testEnv = &envtest.Environment{
CRDDirectoryPaths:     []string{
filepath.Join("..", "..", "crd"),
},
ErrorIfCRDPathMissing: true,
}
```

要启动环境，你可以使用此 `Environment` 上的 `Start` 方法：

```
cfg, err := testEnv.Start()
```

该方法返回一个 `rest.Config` 值，该 `Config` 值用于连接到由 `Environment` 启动的本地集群。在测试结束时，可以使用 `Stop` 方法停止 `Environment`：

```
err := testEnv.Stop()
```

一旦环境启动并且你有了一个 `Config`，你就可以创建 Manager 和 Controller 并启动 Manager，如第 10 章所述。



### 定义 Ginkgo 测试套件

要开始测试 `Reconcile` 函数，可以使用一个 `go` 测试函数来启动 `ginkgo` 的规格说明：

```go
import (
    "testing"
    . "github.com/onsi/ginkgo/v2"
    . "github.com/onsi/gomega"
)

func TestMyReconciler_Reconcile(t *testing.T) {
    RegisterFailHandler(Fail)
    RunSpecs(t,
        "Controller Suite",
    )
}
```

然后，你可以声明 `BeforeSuite` 和 `AfterSuite` 函数，用于启动和停止环境（`Environment`）以及管理器（Manager）。

以下是这些函数的示例，它将创建一个加载了 `MyResource` 的 CRD 的环境。管理器在 `BeforeSuite` 函数的末尾通过 `Go` 协程启动，因此测试可以在主 `Go` 协程上执行。

请注意，你将创建一个*可取消的上下文（Cancelable Context）*，它会在启动管理器时使用，因此你可以通过从 `AfterSuite` 函数中取消该上下文来停止管理器。

- ❶ `testEnv`、`ctx` 和 `cancel` 将在 `BeforeSuite` 和 `AfterSuite` 中使用
- ❷ `k8sClient` 将在测试中使用
- ❸ 创建一个可取消的上下文
- ❹ 创建 `testEnv` 环境
- ❺ 启动 `testEnv` 环境
- ❻ 构建传递给管理器的方案（Scheme）
- ❼ 构建管理器
- ❽ 从管理器中获取 `k8sClient` 以供测试使用
- ❾ 构建控制器
- ❿ 通过 `Go` 协程启动管理器
- ⓫ 取消上下文，这将终止管理器
- ⓬ 终止 `testEnv` 环境

```go
import (
    "context"
    "path/filepath"
    "testing"

    . "github.com/onsi/ginkgo/v2"
    . "github.com/onsi/gomega"
    appsv1 "k8s.io/api/apps/v1"
    "k8s.io/apimachinery/pkg/runtime"
    clientgoscheme "k8s.io/client-go/kubernetes/scheme"
    "sigs.k8s.io/controller-runtime/pkg/builder"
    "sigs.k8s.io/controller-runtime/pkg/client"
    "sigs.k8s.io/controller-runtime/pkg/envtest"
    "sigs.k8s.io/controller-runtime/pkg/log"
    "sigs.k8s.io/controller-runtime/pkg/log/zap"
    "sigs.k8s.io/controller-runtime/pkg/manager"

    mygroupv1alpha1 "github.com/feloy/myresource-crd/pkg/apis/mygroup.example.com/v1alpha1"
)

var (
    testEnv   *envtest.Environment                    ❶
    ctx       context.Context
    cancel    context.CancelFunc
    k8sClient client.Client                           ❷
)

var _ = BeforeSuite(func() {
    log.SetLogger(zap.New(
        zap.WriteTo(GinkgoWriter),
        zap.UseDevMode(true),
    ))

    ctx, cancel = context.WithCancel(                 ❸
        context.Background(),
    )

    testEnv = &envtest.Environment{                   ❹
        CRDDirectoryPaths:     []string{
            filepath.Join("..", "..", "crd"),
        },
        ErrorIfCRDPathMissing: true,
    }

    var err error
    // cfg 在此文件中全局定义。
    cfg, err := testEnv.Start()                       ❺
    Expect(err).NotTo(HaveOccurred())
    Expect(cfg).NotTo(BeNil())

    scheme := runtime.NewScheme()                     ❻
    err = clientgoscheme.AddToScheme(scheme)
    Expect(err).NotTo(HaveOccurred())
    err = mygroupv1alpha1.AddToScheme(scheme)
    Expect(err).NotTo(HaveOccurred())

    mgr, err := manager.New(cfg, manager.Options{     ❼
        Scheme: scheme,
    })
    Expect(err).ToNot(HaveOccurred())

    k8sClient = mgr.GetClient()                       ❽

    err = builder.                                    ❾
        ControllerManagedBy(mgr).
        Named(Name).
        For(&mygroupv1alpha1.MyResource{}).
        Owns(&appsv1.Deployment{}).
        Complete(&MyReconciler{})

    go func() {
        defer GinkgoRecover()
        err = mgr.Start(ctx)                          ❿
        Expect(err).ToNot(
            HaveOccurred(),
            "failed to run manager",
        )
    }()
})

var _ = AfterSuite(func() {
    cancel()                                          ⓫
    err := testEnv.Stop()                             ⓬
    Expect(err).NotTo(HaveOccurred())
})
```

## 编写测试

现在你已经有了一个可以启动和停止环境的测试套件，可以开始编写测试了。测试将首先创建一个 `MyResource` 实例，以验证 `Reconcile` 函数是否按照预期定义创建了预期的“底层”资源。

然后，当底层资源被创建后，测试将更新底层资源的状态（请记住集群中没有部署任何控制器，因此你需要从测试中执行这些更改），以验证 `MyResource` 实例的状态是否相应地更新。

测试计划如下。请注意，每个 `It` 测试将单独执行，并且对于每个 `It`，所有前置的 `BeforeEach` 将在测试之前执行，所有前置的 `AfterEach` 将在测试之后执行。

```go
var _ = Describe("MyResource controller", func() {
    When("creating a MyResource instance", func() {
        BeforeEach(func() {
            // 创建 MyResource 实例
        })
        AfterEach(func() {
            // 删除 MyResource 实例
        })

        It("should create a deployment", func() {
            // 检查 deployment 是否最终被创建
        })

        When("deployment is found", func() {
            BeforeEach(func() {
                // 等待 deployment 最终被创建
            })

            It("should be owned by the MyResource instance", func() {
                // 检查 Deployment 中的 ownerReference
                // 是否引用了 MyResource 实例
            })

            It("should use the image specified in MyResource instance", func() {
            })

            When("deployment ReadyReplicas is 1", func() {
                BeforeEach(func() {
                    // 更新 Deployment 状态为 ReadyReplicas=1
                })

                It("should set status ready for MyResource instance", func() {
                    // 检查 MyResource 实例的状态
                    // 是否最终变为 Ready
                })
            })
        })
    })
})
```

为了清晰展示测试的执行过程，以下是四个 `It` 测试将如何执行的流程，以及 `BeforeEach` 和 `AfterEach` 的展示。

### 测试 1

- **When**：创建一个 MyResource 实例
- **Before**：// 创建 MyResource 实例
- **It**：会创建一个 deployment
- **After**：// 删除 MyResource 实例

### 测试 2

- **When**：创建一个 MyResource 实例
- **Before**：// 创建 MyResource 实例
- **When**：deployment 被找到
- **Before**：// 等待 deployment 最终被创建
- **It**：应由 MyResource 实例拥有
- **After**：// 删除 MyResource 实例

### 测试 3

- **When**：创建一个 MyResource 实例
- **Before**：// 创建 MyResource 实例
- **When**：deployment 被找到
- **Before**：// 等待 deployment 最终被创建
- **It**：应使用 MyResource 实例中指定的镜像
- **After**：// 删除 MyResource 实例

### 测试 4

- **When**：创建一个 MyResource 实例
- **Before**：// 创建 MyResource 实例
- **When**：deployment 被找到
- **Before**：// 等待 deployment 最终被创建
- **When**：deployment 的 ReadyReplicas 为 1
- **Before**：// 更新 Deployment 状态为 ReadyReplicas=1
- **It**：应为 MyResource 实例设置状态为就绪（Ready）
- **After**：// 删除 MyResource 实例

最后，完整的测试源代码在接下来的段落之后。请注意，你正在使用一个随机名称创建 `MyResource` 实例，这样创建的资源就不会在测试之间冲突。

事实上，测试是一个接一个执行的，因此前一个测试遗留的资源可能会干扰后续测试。

在这种实现中，你可以通过删除 `MyResource` 实例来结束测试，但是因为集群中没有运行控制器，所以 Deployment 不会被垃圾回收器删除，尽管它引用了已删除的 `MyResource` 实例作为所有者。因此，如果你使用相同的 `MyResource` 实例名称执行所有测试，第一个测试中的 Deployment 将被用于后续的测试。


```go
import (
    "fmt"
    "math/rand"
    . "github.com/onsi/ginkgo/v2"
    . "github.com/onsi/gomega"
    appsv1 "k8s.io/api/apps/v1"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/apimachinery/pkg/types"
    "sigs.k8s.io/controller-runtime/pkg/client"
    mygroupv1alpha1 "github.com/feloy/myresource-crd/pkg/apis/mygroup.example.com/v1alpha1"
)

var _ = Describe("MyResource 控制器", func() {
    When("当创建一个 MyResource 实例时", func() {
        var (
            myres      mygroupv1alpha1.MyResource
            ownerref   *metav1.OwnerReference
            name       string
            namespace  = "default"
            deployName string
            image      string
        )

        BeforeEach(func() {
            image = fmt.Sprintf("myimage%d", rand.Intn(1000))
            myres = mygroupv1alpha1.MyResource{
                Spec: mygroupv1alpha1.MyResourceSpec{
                    Image: image,
                },
            }
            name = fmt.Sprintf("myres%d", rand.Intn(1000))
            myres.SetName(name)
            myres.SetNamespace(namespace)
            err := k8sClient.Create(ctx, &myres)
            Expect(err).NotTo(HaveOccurred())
            ownerref = metav1.NewControllerRef(
                &myres,
                mygroupv1alpha1.SchemeGroupVersion.
                    WithKind("MyResource"),
            )
            deployName = fmt.Sprintf("%s-deployment", name)
        })

        AfterEach(func() {
            k8sClient.Delete(ctx, &myres)
        })

        It("应该创建一个 deployment", func() {
            var dep appsv1.Deployment
            Eventually(
                deploymentExists(deployName, namespace, &dep),
                10, 1,
            ).Should(BeTrue())
        })

        When("当 deployment 被找到时", func() {
            var dep appsv1.Deployment
            BeforeEach(func() {
                Eventually(
                    deploymentExists(deployName, namespace, &dep),
                    10, 1,
                ).Should(BeTrue())
            })

            It("应该由 MyResource 实例所拥有", func() {
                Expect(dep.GetOwnerReferences()).
                    To(ContainElement(*ownerref))
            })

            It("应该使用 MyResource 实例中指定的镜像", func() {
                Expect(
                    dep.Spec.Template.Spec.Containers[0].Image,
                ).To(Equal(image))
            })

            When("当 deployment 的 ReadyReplicas 为 1 时", func() {
                BeforeEach(func() {
                    dep.Status.Replicas = 1
                    dep.Status.ReadyReplicas = 1
                    err := k8sClient.Status().Update(ctx, &dep)
                    Expect(err).NotTo(HaveOccurred())
                })

                It("应该为 MyResource 实例设置状态为 ready", func() {
                    Eventually(
                        getMyResourceState(name, namespace), 10, 1,
                    ).Should(Equal("Ready"))
                })
            })
        })
    })
})

func deploymentExists(
    name, namespace string, dep *appsv1.Deployment,
) func() bool {
    return func() bool {
        err := k8sClient.Get(ctx, client.ObjectKey{
            Namespace: namespace,
            Name:      name,
        }, dep)
        return err == nil
    }
}

func getMyResourceState(
    name, namespace string,
) func() (string, error) {
    return func() (string, error) {
        myres := mygroupv1alpha1.MyResource{}
        err := k8sClient.Get(ctx, types.NamespacedName{
            Namespace: namespace,
            Name:      name,
        }, &myres)
        if err != nil {
            return "", err
        }
        return myres.Status.State, nil
    }
}
```

## 总结

本章节总结了可用于编写 Kubernetes Operator 的各种概念和库的介绍。

第 8 章介绍了**自定义资源**，通过向所提供的资源列表中添加新资源来扩展 Kubernetes API。第 9 章介绍了使用 Go 语言处理自定义资源的各种方式，包括为此资源生成`Clientset`，或使用`DynamicClient`。

第 10 章介绍了`controller-runtime`库，该库对于实现管理自定义资源生命周期的 Operator 非常有用。第 11 章专注于编写 Operator 的业务逻辑，第 12 章则用于测试该逻辑。

下一章将介绍`kubebuilder` SDK，它是一个使用前几章介绍的工具的框架。该框架通过为新自定义资源定义和关联的 Operator 生成代码，并提供构建和部署这些自定义资源定义及 Operator 到集群的工具，从而简化了 Operator 的开发。

