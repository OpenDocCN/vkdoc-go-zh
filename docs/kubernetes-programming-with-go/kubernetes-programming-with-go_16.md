# 11. 编写调谐循环

在上一章中，我们已经了解了如何使用 `controller-runtime` 库来引导一个新项目以编写一个运算符。在本章中，我们将重点实现 `Reconcile` 函数，这是实现运算符的重要部分。

`Reconcile` 函数包含了运算符的所有业务逻辑。该函数将作用于单一的资源类型——可以说这个运算符*调谐*这种资源——并且当其他类型的对象触发事件时，可以通过使用*属主引用*将这些其他类型映射到被调谐的资源类型上，从而得到通知。

`Reconcile` 函数的作用是确保系统的状态与被调谐资源中指定的状态相匹配。为此，它将创建“底层”资源来实现要调谐的资源。这些资源依次由其他控制器或运算符进行调谐。当这些资源的调谐完成时，它们的状态会被更新以反映其新状态。此外，运算符将能够检测到这些变化并相应地调整被调谐资源的状态。

例如，你在上一章开始实现的运算符调谐了自定义资源 `MyResource`。该运算符将创建 `Deployment` 实例来实现 `MyResource` 实例，并且因此，该运算符也将希望监视 Deployment 资源。

当 Deployment 实例触发一个事件时，为其创建了 Deployment 的 `MyResource` 实例将被调谐。例如，在 Deployment 的 Pod 被创建后，Deployment 的状态将被更新以指示所有副本正在运行。此时，运算符可以修改被调谐资源的状态以指示其已就绪。

*调谐循环* 是一个过程，包括监视资源，在资源被创建、修改或删除时调用 `Reconcile` 函数，使用底层资源实现被调谐的资源，监视这些资源的状态，并相应更新被调谐资源的状态。

## 编写 Reconcile 函数

`Reconcile` 函数从队列中接收一个要调谐的资源。首先要进行的操作是获取关于这个要调谐的资源的信息。

事实上，只会接收到资源的命名空间和名称（其类型通过设计已知为运算符要调谐的类型），但你不会接收到资源的完整定义。

使用 `Get` 操作来获取资源的定义。

### 检查资源是否存在

资源可能因各种原因被加入队列：它已被创建、修改或删除（或者另一个拥有的资源已被创建、修改或删除）。在前两种情况（创建或修改）下，`Get` 操作将成功，并且 `Reconcile` 函数此时将知道资源的定义。在删除的情况下，`Get` 操作将失败并返回 `Notfound` 错误，因为该资源已被删除。

运算符的良好实践是在调谐资源时，为其创建的资源添加 `OwnerReferences`。主要目标是当这些创建的资源被修改时，能够调谐其属主，添加这些 `OwnerReferences` 的一个结果是，当属主资源被删除时，这些拥有的资源将由 Kubernetes 垃圾收集器删除。

因此，当被调谐的对象被删除时，在集群中无需执行任何操作，因为关联的已创建资源的删除将由集群处理。



### 实现已调和资源

如果在集群中找到了需要调和的资源，Operator 的下一步是创建“底层”资源来实现此调和资源。

由于 Kubernetes 是一个声明式平台，创建这些资源的一个好方法是声明底层资源应该是什么，而不依赖于它们在集群中是否存在，并依靠 Kubernetes 控制器接管并调和这些底层资源。

因此，不能盲目使用 `Create` 方法，因为我们不确定资源是否存在，如果资源已存在，操作将会失败。

你可以检查资源是否存在：如果不存在则创建，如果存在则修改。如之前章节所示，服务器端应用（Server-side Apply）方法在这种情况下非常完美：执行 Apply 操作时，如果资源不存在，则创建它；如果资源存在，则修补它，并处理可能因其他参与者修改资源而产生的冲突。

使用服务器端应用方法，Operator 无需检查资源是否存在，也无需检查资源是否被其他参与者修改。Operator 只需要从自己的角度使用资源的定义来执行服务器端应用操作。

应用底层资源后，应考虑两种可能性。

* **情况 1：** 如果底层资源已存在且未被 Apply 操作修改，则不会为这些资源触发 MODIFIED 事件，并且 `Reconcile` 函数将不会被再次调用（至少对于这次 Apply 操作来说）。

* **情况 2：** 如果底层资源被创建或修改，这将为这些资源触发 CREATED 或 MODIFIED 事件，并且由于这些事件，`Reconcile` 函数将被再次调用。函数的新执行将再次 Apply 底层资源，如果在此期间没有更新这些资源，Operator 将进入情况 1。

新的底层资源最终将由它们各自的 Operator 或控制器处理。反过来，它们将调和这些资源，并更新其状态以宣布当前状态。

一旦这些底层资源的状态被更新，将触发 MODIFIED 事件，并且 `Reconcile` 函数将被再次调用。Operator 将再次 Apply 资源，并且必须考虑情况 1 和情况 2。

Operator 需要在某个时刻读取它创建的底层资源的状态，以便计算已调和资源的状态。在简单的情况下，可以在执行底层资源的服务器端应用之后立即执行此操作。

### 简单实现示例

为了说明，这里是一个完整的 `Reconcile` 函数，用于一个 Operator，该 Operator 使用 `MyResource` 实例中提供的 `Image` 和 `Memory` 信息创建一个 Deployment。

* ❶ 获取要调和的资源的定义

* ❷ 如果资源不存在，立即返回

* ❸ 构建指向要调和资源的 `ownerReference`

* ❹ 为“底层” Deployment 使用服务器端应用

* ❺ 基于“底层” Deployment 计算资源状态

* ❻ 更新要调和资源的状态

```go
func (a *MyReconciler) Reconcile(
ctx context.Context,
req reconcile.Request,
) (reconcile.Result, error) {
log := log.FromContext(ctx)
log.Info("获取 myresource 实例")
myresource := mygroupv1alpha1.MyResource{}
err := a.client.Get(                                        ❶
ctx,
req.NamespacedName,
&myresource,
&client.GetOptions{},
)
if err != nil {
if errors.IsNotFound(err) {                              ❷
log.Info("资源未找到")
return reconcile.Result{}, nil
}
return reconcile.Result{}, err
}
ownerReference := metav1.NewControllerRef(               ❸
&myresource,
mygroupv1alpha1.SchemeGroupVersion.
WithKind("MyResource"),
)
err = a.applyDeployment(                                   ❹
ctx,
&myresource,
ownerReference,
)
if err != nil {
return reconcile.Result{}, err
}
status, err := a.computeStatus(ctx, &myresource)          ❺
if err != nil {
return reconcile.Result{}, err
}
myresource.Status = *status
log.Info("更新状态", "state", status.State)
err = a.client.Status().Update(ctx, &myresource)          ❻
if err != nil {
return reconcile.Result{}, err
}
return reconcile.Result{}, nil
}
```

以下是 Operator 创建的 Deployment 的服务器端应用操作的实现示例：

* ❼ 使用 `Patch` 方法执行服务器端应用操作

* ❽ 使用待调和资源中定义的 `Image`

* ❾ 使用待调和资源中定义的 `Memory`

* ❿ 设置 `OwnerReference` 以指向待调和资源

```go
func (a *MyReconciler) applyDeployment(
ctx context.Context,
myres *mygroupv1alpha1.MyResource,
ownerref *metav1.OwnerReference,
) error {
deploy := createDeployment(myres, ownerref)
err := a.client.Patch(                                   ❼
ctx,
deploy,
client.Apply,
client.FieldOwner(Name),
client.ForceOwnership,
)
return err
}
func createDeployment(
myres *mygroupv1alpha1.MyResource,
ownerref *metav1.OwnerReference,
) *appsv1.Deployment {
deploy := &appsv1.Deployment{
ObjectMeta: metav1.ObjectMeta{
Labels: map[string]string{
"myresource": myres.GetName(),
},
},
Spec: appsv1.DeploymentSpec{
Selector: &metav1.LabelSelector{
MatchLabels: map[string]string{
"myresource": myres.GetName(),
},
},
Template: corev1.PodTemplateSpec{
ObjectMeta: metav1.ObjectMeta{
Labels: map[string]string{
"myresource": myres.GetName(),
},
},
Spec: corev1.PodSpec{
Containers: []corev1.Container{
{
Name:  "main",
Image: myres.Spec.Image,                    ❽
Resources: corev1.ResourceRequirements{
Requests: corev1.ResourceList{
corev1.ResourceMemory:      myres.Spec.Memory,          ❾
},
},
},
},
},
},
},
}
deploy.SetName(myres.GetName() + "-deployment")
deploy.SetNamespace(myres.GetNamespace())
deploy.SetGroupVersionKind(
appsv1.SchemeGroupVersion.WithKind("Deployment"),
)
deploy.SetOwnerReferences([]metav1.OwnerReference{     ❿
*ownerref,
})
return deploy
}
```

然后，这是一个如何实现状态计算和更新的示例：

* ⓫ 获取为此调和资源创建的 Deployment

* ⓬ 获取找到的唯一 Deployment 的状态

* ⓭ 当副本数为 `1` 时，为已调和资源设置状态为 `Ready`

```go
const (
_buildingState = "Building"
_readyState    = "Ready"
)
func (a *MyReconciler) computeStatus(
ctx context.Context,
myres *mygroupv1alpha1.MyResource,
) (*mygroupv1alpha1.MyResourceStatus, error) {
logger := log.FromContext(ctx)
result := mygroupv1alpha1.MyResourceStatus{
State: _buildingState,
}
deployList := appsv1.DeploymentList{}
err := a.client.List(                                        ⓫
ctx,
&deployList,
client.InNamespace(myres.GetNamespace()),
client.MatchingLabels{
"myresource": myres.GetName(),
},
)
if err != nil {
return nil, err
}
if len(deployList.Items) == 0 {
logger.Info("未找到 Deployment")
return &result, nil
}
if len(deployList.Items) > 1 {
logger.Info(
"找到过多 Deployment", "count",
len(deployList.Items),
)
return nil, fmt.Errorf(
"找到 %d 个 Deployment，预期为 1",
len(deployList.Items),
)
}
status := deployList.Items[0].Status                    ⓬
logger.Info(
"获取 Deployment 状态",
"status", status,
)
if status.ReadyReplicas == 1 {
result.State = _readyState                              ⓭
}
return &result, nil
}
```



## 结论

本章展示了如何为一个简单的 Operator 实现 `Reconcile` 函数，该 Operator 利用自定义资源中的信息创建了一个 Deployment。

真实的 Operator 通常比这个简单示例更复杂，可能会创建多个具有更复杂生命周期的资源，但本章演示了开始编写 Operator 时需要了解的几个要点：`Reconcile` 循环的声明式特性、**所有者引用**的使用、**服务端应用**的使用，以及状态更新方式。

下一章将展示如何测试 `Reconcile` 循环。

