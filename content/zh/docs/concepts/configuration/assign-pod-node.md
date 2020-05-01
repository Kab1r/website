---
title: 将 Pod 分配给节点
content_template: templates/concept
weight: 50
---

## <!--

reviewers:

- davidopp
- kevin-wangzefeng
- bsalamat title: Assigning Pods to Nodes content_template: templates/concept
  weight: 50

---

-->

{{% capture overview %}}

<!--
You can constrain a {{< glossary_tooltip text="Pod" term_id="pod" >}} to only be able to run on particular
{{< glossary_tooltip text="Node(s)" term_id="node" >}}, or to prefer to run on particular nodes.
There are several ways to do this, and the recommended approaches all use
[label selectors](/docs/concepts/overview/working-with-objects/labels/) to make the selection.
Generally such constraints are unnecessary, as the scheduler will automatically do a reasonable placement
(e.g. spread your pods across nodes, not place the pod on a node with insufficient free resources, etc.)
but there are some circumstances where you may want more control on a node where a pod lands, e.g. to ensure
that a pod ends up on a machine with an SSD attached to it, or to co-locate pods from two different
services that communicate a lot into the same availability zone.
-->

你可以约束一个 {{< glossary_tooltip text="Pod" term_id="pod" >}} 只能在特定的
{{< glossary_tooltip text="Node(s)" term_id="node" >}} 上运行，或者优先运行在特
定的节点上。有几种方法可以实现这点，推荐的方法都是
用[标签选择器](/docs/concepts/overview/working-with-objects/labels/)来进行选择。
通常这样的约束不是必须的，因为调度器将自动进行合理的放置（比如，将 pod 分散到节
点上，而不是将 pod 放置在可用资源不足的节点上等等），但在某些情况下，你可以需要
更多控制 pod 停靠的节点，例如，确保 pod 最终落在连接了 SSD 的机器上，或者将来自
两个不同的服务且有大量通信的 pod 放置在同一个可用区。

{{% /capture %}}

{{% capture body %}}

## nodeSelector

<!--
`nodeSelector` is the simplest recommended form of node selection constraint.
`nodeSelector` is a field of PodSpec. It specifies a map of key-value pairs. For the pod to be eligible
to run on a node, the node must have each of the indicated key-value pairs as labels (it can have
additional labels as well). The most common usage is one key-value pair.
-->

`nodeSelector` 是节点选择约束的最简单推荐形式。`nodeSelector` 是 PodSpec 的一个
字段。它指定键值对的映射。为了使 pod 可以在节点上运行，节点必须具有每个指定的键
值对作为标签（它也可以具有其他标签）。最常用的是一对键值对。

<!--
Let's walk through an example of how to use `nodeSelector`.
-->

让我们来看一个使用 `nodeSelector` 的例子。

<!--
### Step Zero: Prerequisites
-->

### 步骤零：先决条件

<!--
This example assumes that you have a basic understanding of Kubernetes pods and that you have [set up a Kubernetes cluster](/docs/setup/).
-->

本示例假设你已基本了解 Kubernetes 的 pod 并且已
经[建立一个 Kubernetes 集群](/docs/setup/)。

<!--
### Step One: Attach label to the node
-->

### 步骤一：添加标签到节点

<!--
Run `kubectl get nodes` to get the names of your cluster's nodes. Pick out the one that you want to add a label to, and then run `kubectl label nodes <node-name> <label-key>=<label-value>` to add a label to the node you've chosen. For example, if my node name is 'kubernetes-foo-node-1.c.a-robinson.internal' and my desired label is 'disktype=ssd', then I can run `kubectl label nodes kubernetes-foo-node-1.c.a-robinson.internal disktype=ssd`.
-->

执行 `kubectl get nodes` 命令获取集群的节点名称。选择一个你要增加标签的节点，然
后执行 `kubectl label nodes <node-name> <label-key>=<label-value>` 命令将标签添
加到你所选择的节点上。例如，如果你的节点名称为
'kubernetes-foo-node-1.c.a-robinson.internal' 并且想要的标签是 'disktype=ssd'，
则可以执行
`kubectl label nodes kubernetes-foo-node-1.c.a-robinson.internal disktype=ssd`
命令。

<!--
You can verify that it worked by re-running `kubectl get nodes --show-labels` and checking that the node now has a label. You can also use `kubectl describe node "nodename"` to see the full list of labels of the given node.
-->

你可以通过重新运行 `kubectl get nodes --show-labels` 并且查看节点当前具有了一个
标签来验证它是否有效。你也可以使用 `kubectl describe node "nodename"` 命令查看指
定节点的标签完整列表。

<!--
### Step Two: Add a nodeSelector field to your pod configuration
-->

### 步骤二：添加 nodeSelector 字段到 pod 配置中

<!--
Take whatever pod config file you want to run, and add a nodeSelector section to it, like this. For example, if this is my pod config:
-->

拿任意一个你想运行的 pod 的配置文件，并且在其中添加一个 nodeSelector 部分。例如
，如果下面是我的 pod 配置：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
    - name: nginx
      image: nginx
```

<!--
Then add a nodeSelector like so:
-->

然后像下面这样添加 nodeSelector：

{{< codenew file="pods/pod-nginx.yaml" >}}

<!--
When you then run `kubectl apply -f https://k8s.io/examples/pods/pod-nginx.yaml`,
the Pod will get scheduled on the node that you attached the label to. You can
verify that it worked by running `kubectl get pods -o wide` and looking at the
"NODE" that the Pod was assigned to.
-->

当你之后运行 `kubectl apply -f https://k8s.io/examples/pods/pod-nginx.yaml` 命令
，pod 将会调度到将标签添加到的节点上。你可以通过运行 `kubectl get pods -o wide`
并查看分配给 pod 的 “NODE” 来验证其是否有效。

<!--
## Interlude: built-in node labels {#built-in-node-labels}
-->

## 插曲：内置的节点标签 {#内置的节点标签}

<!--
In addition to labels you [attach](#step-one-attach-label-to-the-node), nodes come pre-populated
with a standard set of labels. These labels are
-->

除了你[附加](#添加标签到节点)的标签外，节点还预先填充了一组标准标签。这些标签是

- [`kubernetes.io/hostname`](/docs/reference/kubernetes-api/labels-annotations-taints/#kubernetes-io-hostname)
- [`failure-domain.beta.kubernetes.io/zone`](/docs/reference/kubernetes-api/labels-annotations-taints/#failure-domainbetakubernetesiozone)
- [`failure-domain.beta.kubernetes.io/region`](/docs/reference/kubernetes-api/labels-annotations-taints/#failure-domainbetakubernetesioregion)
- [`topology.kubernetes.io/zone`](/docs/reference/kubernetes-api/labels-annotations-taints/#topologykubernetesiozone)
- [`topology.kubernetes.io/region`](/docs/reference/kubernetes-api/labels-annotations-taints/#topologykubernetesiozone)
- [`beta.kubernetes.io/instance-type`](/docs/reference/kubernetes-api/labels-annotations-taints/#beta-kubernetes-io-instance-type)
- [`node.kubernetes.io/instance-type`](/docs/reference/kubernetes-api/labels-annotations-taints/#nodekubernetesioinstance-type)
- [`kubernetes.io/os`](/docs/reference/kubernetes-api/labels-annotations-taints/#kubernetes-io-os)
- [`kubernetes.io/arch`](/docs/reference/kubernetes-api/labels-annotations-taints/#kubernetes-io-arch)

{{< note >}}

<!--
The value of these labels is cloud provider specific and is not guaranteed to be reliable.
For example, the value of `kubernetes.io/hostname` may be the same as the Node name in some environments
and a different value in other environments.
-->

这些标签的值是特定于云供应商的，因此不能保证可靠。例如，`kubernetes.io/hostname`
的值在某些环境中可能与节点名称相同，但在其他环境中可能是一个不同的值。
{{< /note >}}

<!--
## Node isolation/restriction
-->

## 节点隔离/限制

<!--
Adding labels to Node objects allows targeting pods to specific nodes or groups of nodes.
This can be used to ensure specific pods only run on nodes with certain isolation, security, or regulatory properties.
When using labels for this purpose, choosing label keys that cannot be modified by the kubelet process on the node is strongly recommended.
This prevents a compromised node from using its kubelet credential to set those labels on its own Node object,
and influencing the scheduler to schedule workloads to the compromised node.
-->

向 Node 对象添加标签可以将 pod 定位到特定的节点或节点组。这可以用来确保指定的
pod 只能运行在具有一定隔离性，安全性或监管属性的节点上。当为此目的使用标签时，强
烈建议选择节点上的 kubelet 进程无法修改的标签键。这可以防止受感染的节点使用其
kubelet 凭据在自己的 Node 对象上设置这些标签，并影响调度器将工作负载调度到受感染
的节点。

<!--
The `NodeRestriction` admission plugin prevents kubelets from setting or modifying labels with a `node-restriction.kubernetes.io/` prefix.
To make use of that label prefix for node isolation:
-->

`NodeRestriction` 准入插件防止 kubelet 使用 `node-restriction.kubernetes.io/` 前
缀设置或修改标签。要使用该标签前缀进行节点隔离：

<!--
1. Check that you're using Kubernetes v1.11+ so that NodeRestriction is available.
2. Ensure you are using the [Node authorizer](/docs/reference/access-authn-authz/node/) and have _enabled_ the [NodeRestriction admission plugin](/docs/reference/access-authn-authz/admission-controllers/#noderestriction).
3. Add labels under the `node-restriction.kubernetes.io/` prefix to your Node objects, and use those labels in your node selectors.
For example, `example.com.node-restriction.kubernetes.io/fips=true` or `example.com.node-restriction.kubernetes.io/pci-dss=true`.
-->

1. 检查是否在使用 Kubernetes v1.11+，以便 NodeRestriction 功能可用。
2. 确保你在使用[节点授权](/docs/reference/access-authn-authz/node/)并且已经*启
   用*
   [NodeRestriction 准入插件](/docs/reference/access-authn-authz/admission-controllers/#noderestriction)。
3. 将 `node-restriction.kubernetes.io/` 前缀下的标签添加到 Node 对象，然后在节点
   选择器中使用这些标签。例如
   ，`example.com.node-restriction.kubernetes.io/fips=true` 或
   `example.com.node-restriction.kubernetes.io/pci-dss=true`。

<!--
## Affinity and anti-affinity
-->

## 亲和与反亲和

<!--
`nodeSelector` provides a very simple way to constrain pods to nodes with particular labels. The affinity/anti-affinity
feature, greatly expands the types of constraints you can express. The key enhancements are
-->

`nodeSelector` 提供了一种非常简单的方法来将 pod 约束到具有特定标签的节点上。亲和
/反亲和功能极大地扩展了你可以表达约束的类型。关键的增强点是

<!--
1. the language is more expressive (not just "AND of exact match")
2. you can indicate that the rule is "soft"/"preference" rather than a hard requirement, so if the scheduler
   can't satisfy it, the pod will still be scheduled
3. you can constrain against labels on other pods running on the node (or other topological domain),
   rather than against labels on the node itself, which allows rules about which pods can and cannot be co-located
-->

1. 语言更具表现力（不仅仅是“完全匹配的 AND”）
2. 你可以发现规则是“软”/“偏好”，而不是硬性要求，因此，如果调度器无法满足该要求，
   仍然调度该 pod
3. 你可以使用节点上（或其他拓扑域中）的 pod 的标签来约束，而不是使用节点本身的标
   签，来允许哪些 pod 可以或者不可以被放置在一起。

<!--
The affinity feature consists of two types of affinity, "node affinity" and "inter-pod affinity/anti-affinity".
Node affinity is like the existing `nodeSelector` (but with the first two benefits listed above),
while inter-pod affinity/anti-affinity constrains against pod labels rather than node labels, as
described in the third item listed above, in addition to having the first and second properties listed above.
-->

亲和功能包含两种类型的亲和，即“节点亲和”和“pod 间亲和/反亲和”。节点亲和就像现有
的 `nodeSelector`（但具有上面列出的前两个好处），然而 pod 间亲和/反亲和约束 pod
标签而不是节点标签（在上面列出的第三项中描述，除了具有上面列出的第一和第二属性）
。

<!--
### Node affinity
-->

### 节点亲和

<!--
Node affinity is conceptually similar to `nodeSelector` -- it allows you to constrain which nodes your
pod is eligible to be scheduled on, based on labels on the node.
-->

节点亲和概念上类似于 `nodeSelector`，它使你可以根据节点上的标签来约束 pod 可以调
度到哪些节点。

<!--
There are currently two types of node affinity, called `requiredDuringSchedulingIgnoredDuringExecution` and
`preferredDuringSchedulingIgnoredDuringExecution`. You can think of them as "hard" and "soft" respectively,
in the sense that the former specifies rules that *must* be met for a pod to be scheduled onto a node (just like
`nodeSelector` but using a more expressive syntax), while the latter specifies *preferences* that the scheduler
will try to enforce but will not guarantee. The "IgnoredDuringExecution" part of the names means that, similar
to how `nodeSelector` works, if labels on a node change at runtime such that the affinity rules on a pod are no longer
met, the pod will still continue to run on the node. In the future we plan to offer
`requiredDuringSchedulingRequiredDuringExecution` which will be just like `requiredDuringSchedulingIgnoredDuringExecution`
except that it will evict pods from nodes that cease to satisfy the pods' node affinity requirements.
-->

目前有两种类型的节点亲和，分别为
`requiredDuringSchedulingIgnoredDuringExecution` 和
`preferredDuringSchedulingIgnoredDuringExecution`。你可以视它们为“硬”和“软”，意
思是，前者指定了将 pod 调度到一个节点上*必须*满足的规则（就像 `nodeSelector` 但
使用更具表现力的语法），后者指定调度器将尝试执行但不能保证的*偏好*。名称的
“IgnoredDuringExecution”部分意味着，类似于 `nodeSelector` 的工作原理，如果节点的
标签在运行时发生变更，从而不再满足 pod 上的亲和规则，那么 pod 将仍然继续在该节点
上运行。将来我们计划提供 `requiredDuringSchedulingRequiredDuringExecution`，它将
类似于 `requiredDuringSchedulingIgnoredDuringExecution`，除了它会将 pod 从不再满
足 pod 的节点亲和要求的节点上驱逐。

<!--
Thus an example of `requiredDuringSchedulingIgnoredDuringExecution` would be "only run the pod on nodes with Intel CPUs"
and an example `preferredDuringSchedulingIgnoredDuringExecution` would be "try to run this set of pods in failure
zone XYZ, but if it's not possible, then allow some to run elsewhere".
-->

因此，`requiredDuringSchedulingIgnoredDuringExecution` 的示例将是“仅将 pod 运行
在具有 Intel CPU 的节点上”，而 `preferredDuringSchedulingIgnoredDuringExecution`
的示例为“尝试将这组 pod 运行在 XYZ 故障区域，如果这不可能的话，则允许一些 pod 在
其他地方运行”。

<!--
Node affinity is specified as field `nodeAffinity` of field `affinity` in the PodSpec.
-->

节点亲和通过 PodSpec 的 `affinity` 字段下的 `nodeAffinity` 字段进行指定。

<!--
Here's an example of a pod that uses node affinity:
-->

下面是一个使用节点亲和的 pod 的实例：

{{< codenew file="pods/pod-with-node-affinity.yaml" >}}

<!--
This node affinity rule says the pod can only be placed on a node with a label whose key is
`kubernetes.io/e2e-az-name` and whose value is either `e2e-az1` or `e2e-az2`. In addition,
among nodes that meet that criteria, nodes with a label whose key is `another-node-label-key` and whose
value is `another-node-label-value` should be preferred.
-->

此节点亲和规则表示，pod 只能放置在具有标签键为 `kubernetes.io/e2e-az-name` 且 标
签值为 `e2e-az1` 或 `e2e-az2` 的节点上。另外，在满足这些标准的节点中，具有标签键
为 `another-node-label-key` 且标签值为 `another-node-label-value` 的节点应该优先
使用。

<!--
You can see the operator `In` being used in the example. The new node affinity syntax supports the following operators: `In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt`, `Lt`.
You can use `NotIn` and `DoesNotExist` to achieve node anti-affinity behavior, or use
[node taints](/docs/concepts/configuration/taint-and-toleration/) to repel pods from specific nodes.
-->

你可以在上面的例子中看到 `In` 操作符的使用。新的节点亲和语法支持下面的操作符：
`In`，`NotIn`，`Exists`，`DoesNotExist`，`Gt`，`Lt`。你可以使用 `NotIn` 和
`DoesNotExist` 来实现节点反亲和行为，或者使
用[节点污点](/docs/concepts/configuration/taint-and-toleration/)将 pod 从特定节
点中驱逐。

<!--
If you specify both `nodeSelector` and `nodeAffinity`, *both* must be satisfied for the pod
to be scheduled onto a candidate node.
-->

如果你同时指定了 `nodeSelector` 和 `nodeAffinity`，*两者*必须都要满足，才能将
pod 调度到候选节点上。

<!--
If you specify multiple `nodeSelectorTerms` associated with `nodeAffinity` types, then the pod can be scheduled onto a node **if one of** the `nodeSelectorTerms` is satisfied.
-->

如果你指定了多个与 `nodeAffinity` 类型关联的 `nodeSelectorTerms`，则**如果其中一
个** `nodeSelectorTerms` 满足的话，pod 将可以调度到节点上。

<!--
If you specify multiple `matchExpressions` associated with `nodeSelectorTerms`, then the pod can be scheduled onto a node **only if all** `matchExpressions` can be satisfied.
-->

如果你指定了多个与 `nodeSelectorTerms` 关联的 `matchExpressions`，则**只有当所
有** `matchExpressions` 满足的话，pod 才会可以调度到节点上。

<!--
If you remove or change the label of the node where the pod is scheduled, the pod won't be removed. In other words, the affinity selection works only at the time of scheduling the pod.
-->

如果你修改或删除了 pod 所调度到的节点的标签，pod 不会被删除。换句话说，亲和选择
只在 pod 调度期间有效。

<!--
The `weight` field in `preferredDuringSchedulingIgnoredDuringExecution` is in the range 1-100. For each node that meets all of the scheduling requirements (resource request, RequiredDuringScheduling affinity expressions, etc.), the scheduler will compute a sum by iterating through the elements of this field and adding "weight" to the sum if the node matches the corresponding MatchExpressions. This score is then combined with the scores of other priority functions for the node. The node(s) with the highest total score are the most preferred.
-->

`preferredDuringSchedulingIgnoredDuringExecution` 中的 `weight` 字段值的范围是
1-100。对于每个符合所有调度要求（资源请求，RequiredDuringScheduling 亲和表达式等
）的节点，调度器将遍历该字段的元素来计算总和，并且如果节点匹配对应的
MatchExpressions，则添加“权重”到总和。然后将这个评分与该节点的其他优先级函数的评
分进行组合。总分最高的节点是最优选的。

<!--
### Inter-pod affinity and anti-affinity
-->

### pod 间亲和与反亲和

<!--
Inter-pod affinity and anti-affinity allow you to constrain which nodes your pod is eligible to be scheduled *based on
labels on pods that are already running on the node* rather than based on labels on nodes. The rules are of the form
"this pod should (or, in the case of anti-affinity, should not) run in an X if that X is already running one or more pods that meet rule Y".
Y is expressed as a LabelSelector with an optional associated list of namespaces; unlike nodes, because pods are namespaced
(and therefore the labels on pods are implicitly namespaced),
a label selector over pod labels must specify which namespaces the selector should apply to. Conceptually X is a topology domain
like node, rack, cloud provider zone, cloud provider region, etc. You express it using a `topologyKey` which is the
key for the node label that the system uses to denote such a topology domain, e.g. see the label keys listed above
in the section [Interlude: built-in node labels](#built-in-node-labels).
-->

pod 间亲和与反亲和使你可以*基于已经在节点上运行的 pod 的标签*来约束 pod 可以调度
到的节点，而不是基于节点上的标签。规则的格式为“如果 X 节点上已经运行了一个或多个
满足规则 Y 的 pod，则这个 pod 应该（或者在非亲和的情况下不应该）运行在 X 节点
”。Y 表示一个具有可选的关联命令空间列表的 LabelSelector；与节点不同，因为 pod 是
命名空间限定的（因此 pod 上的标签也是命名空间限定的），因此作用于 pod 标签的标签
选择器必须指定选择器应用在哪个命名空间。从概念上讲，X 是一个拓扑域，如节点，机架
，云供应商地区，云供应商区域等。你可以使用 `topologyKey` 来表示它，`topologyKey`
是节点标签的键以便系统用来表示这样的拓扑域。请参阅上
面[插曲：内置的节点标签](#内置的节点标签)部分中列出的标签键。

{{< note >}}

<!--
Inter-pod affinity and anti-affinity require substantial amount of
processing which can slow down scheduling in large clusters significantly. We do
not recommend using them in clusters larger than several hundred nodes.
-->

Pod 间亲和与反亲和需要大量的处理，这可能会显著减慢大规模集群中的调度。我们不建议
在超过数百个节点的集群中使用它们。 {{< /note >}}

{{< note >}}

<!--
Pod anti-affinity requires nodes to be consistently labelled, i.e. every node in the cluster must have an appropriate label matching `topologyKey`. If some or all nodes are missing the specified `topologyKey` label, it can lead to unintended behavior.
-->

Pod 反亲和需要对节点进行一致的标记，即集群中的每个节点必须具有适当的标签能够匹配
`topologyKey`。如果某些或所有节点缺少指定的 `topologyKey` 标签，可能会导致意外行
为。 {{< /note >}}

<!--
As with node affinity, there are currently two types of pod affinity and anti-affinity, called `requiredDuringSchedulingIgnoredDuringExecution` and
`preferredDuringSchedulingIgnoredDuringExecution` which denote "hard" vs. "soft" requirements.
See the description in the node affinity section earlier.
An example of `requiredDuringSchedulingIgnoredDuringExecution` affinity would be "co-locate the pods of service A and service B
in the same zone, since they communicate a lot with each other"
and an example `preferredDuringSchedulingIgnoredDuringExecution` anti-affinity would be "spread the pods from this service across zones"
(a hard requirement wouldn't make sense, since you probably have more pods than zones).
-->

与节点亲和一样，当前有两种类型的 pod 亲和与反亲和，即
`requiredDuringSchedulingIgnoredDuringExecution` 和
`preferredDuringSchedulingIgnoredDuringExecution`，分表表示“硬性”与“软性”要求。
请参阅前面节点亲和部分中的描述。`requiredDuringSchedulingIgnoredDuringExecution`
亲和的一个示例是“将服务 A 和服务 B 的 pod 放置在同一区域，因为它们之间进行大量交
流”，而 `preferredDuringSchedulingIgnoredDuringExecution` 反亲和的示例将是“将此
服务的 pod 跨区域分布”（硬性要求是说不通的，因为你可能拥有的 pod 数多于区域数）
。

<!--
Inter-pod affinity is specified as field `podAffinity` of field `affinity` in the PodSpec.
And inter-pod anti-affinity is specified as field `podAntiAffinity` of field `affinity` in the PodSpec.
-->

Pod 间亲和通过 PodSpec 中 `affinity` 字段下的 `podAffinity` 字段进行指定。而 pod
间反亲和通过 PodSpec 中 `affinity` 字段下的 `podAntiAffinity` 字段进行指定。

<!--
#### An example of a pod that uses pod affinity:
-->

### Pod 使用 pod 亲和 的示例：

{{< codenew file="pods/pod-with-pod-affinity.yaml" >}}

<!--
The affinity on this pod defines one pod affinity rule and one pod anti-affinity rule. In this example, the
`podAffinity` is `requiredDuringSchedulingIgnoredDuringExecution`
while the `podAntiAffinity` is `preferredDuringSchedulingIgnoredDuringExecution`. The
pod affinity rule says that the pod can be scheduled onto a node only if that node is in the same zone
as at least one already-running pod that has a label with key "security" and value "S1". (More precisely, the pod is eligible to run
on node N if node N has a label with key `failure-domain.beta.kubernetes.io/zone` and some value V
such that there is at least one node in the cluster with key `failure-domain.beta.kubernetes.io/zone` and
value V that is running a pod that has a label with key "security" and value "S1".) The pod anti-affinity
rule says that the pod prefers not to be scheduled onto a node if that node is already running a pod with label
having key "security" and value "S2". (If the `topologyKey` were `failure-domain.beta.kubernetes.io/zone` then
it would mean that the pod cannot be scheduled onto a node if that node is in the same zone as a pod with
label having key "security" and value "S2".) See the
[design doc](https://git.k8s.io/community/contributors/design-proposals/scheduling/podaffinity.md)
for many more examples of pod affinity and anti-affinity, both the `requiredDuringSchedulingIgnoredDuringExecution`
flavor and the `preferredDuringSchedulingIgnoredDuringExecution` flavor.
-->

在这个 pod 的 affinity 配置定义了一条 pod 亲和规则和一条 pod 反亲和规则。在此示
例中，`podAffinity` 配置为 `requiredDuringSchedulingIgnoredDuringExecution`，然
而 `podAntiAffinity` 配置为
`preferredDuringSchedulingIgnoredDuringExecution`。pod 亲和规则表示，仅当节点和
至少一个已运行且有键为“security”且值为“S1”的标签的 pod 处于同一区域时，才可以将
该 pod 调度到节点上。（更确切的说，如果节点 N 具有带有键
`failure-domain.beta.kubernetes.io/zone` 和某个值 V 的标签，则 pod 有资格在节点
N 上运行，以便集群中至少有一个节点具有键
`failure-domain.beta.kubernetes.io/zone` 和值为 V 的节点正在运行具有键“security”
和值“S1”的标签的 pod。）pod 反亲和规则表示，如果节点已经运行了一个具有键
“security”和值“S2”的标签的 pod，则该 pod 不希望将其调度到该节点上。（如果
`topologyKey` 为 `failure-domain.beta.kubernetes.io/zone`，则意味着当节点和具有
键“security”和值“S2”的标签的 pod 处于相同的区域，pod 不能被调度到该节点上。）查
阅[设计文档](https://git.k8s.io/community/contributors/design-proposals/scheduling/podaffinity.md)来
获取更多 pod 亲和与反亲和的样例，包括
`requiredDuringSchedulingIgnoredDuringExecution` 和
`preferredDuringSchedulingIgnoredDuringExecution` 两种配置。

<!--
The legal operators for pod affinity and anti-affinity are `In`, `NotIn`, `Exists`, `DoesNotExist`.
-->

Pod 亲和与反亲和的合法操作符有 `In`，`NotIn`，`Exists`，`DoesNotExist`。

<!--
In principle, the `topologyKey` can be any legal label-key. However,
for performance and security reasons, there are some constraints on topologyKey:
-->

原则上，`topologyKey` 可以是任何合法的标签键。然而，出于性能和安全原�
，topologyKey 受到一些限制：

<!--
1. For affinity and for `requiredDuringSchedulingIgnoredDuringExecution` pod anti-affinity,
empty `topologyKey` is not allowed.
2. For `requiredDuringSchedulingIgnoredDuringExecution` pod anti-affinity, the admission controller `LimitPodHardAntiAffinityTopology` was introduced to limit `topologyKey` to `kubernetes.io/hostname`. If you want to make it available for custom topologies, you may modify the admission controller, or simply disable it.
3. For `preferredDuringSchedulingIgnoredDuringExecution` pod anti-affinity, empty `topologyKey` is interpreted as "all topologies" ("all topologies" here is now limited to the combination of `kubernetes.io/hostname`, `failure-domain.beta.kubernetes.io/zone` and `failure-domain.beta.kubernetes.io/region`).
4. Except for the above cases, the `topologyKey` can be any legal label-key.
-->

1. 对于亲和与 `requiredDuringSchedulingIgnoredDuringExecution` 要求的 pod 反亲和
   ，`topologyKey` 不允许为空。
2. 对于 `requiredDuringSchedulingIgnoredDuringExecution` 要求的 pod 反亲和，准入
   控制器 `LimitPodHardAntiAffinityTopology` 被引入来限制 `topologyKey` 不为
   `kubernetes.io/hostname`。如果你想使它可用于自定义拓扑结构，你必须修改准入控
   制器或者禁用它。
3. 对于 `preferredDuringSchedulingIgnoredDuringExecution` 要求的 pod 反亲和，空
   的 `topologyKey` 被解释为“所有拓扑结构”（这里的“所有拓扑结构”限制为
   `kubernetes.io/hostname`，`failure-domain.beta.kubernetes.io/zone` 和
   `failure-domain.beta.kubernetes.io/region` 的组合）。
4. 除上述情况外，`topologyKey` 可以是任何合法的标签键。

<!--
In addition to `labelSelector` and `topologyKey`, you can optionally specify a list `namespaces`
of namespaces which the `labelSelector` should match against (this goes at the same level of the definition as `labelSelector` and `topologyKey`).
If omitted or empty, it defaults to the namespace of the pod where the affinity/anti-affinity definition appears.
-->

除了 `labelSelector` 和 `topologyKey`，你也可以指定表示命名空间的 `namespaces`
队列，`labelSelector` 也应该匹配它（这个与 `labelSelector` 和 `topologyKey` 的定
义位于相同的级别）。如果忽略或者为空，则默认为 pod 亲和/反亲和的定义所在的命名空
间。

<!--
All `matchExpressions` associated with `requiredDuringSchedulingIgnoredDuringExecution` affinity and anti-affinity
must be satisfied for the pod to be scheduled onto a node.
-->

所有与 `requiredDuringSchedulingIgnoredDuringExecution` 亲和与反亲和关联的
`matchExpressions` 必须满足，才能将 pod 调度到节点上。

<!--
#### More Practical Use-cases
-->

#### 更实际的用例

<!--
Interpod Affinity and AntiAffinity can be even more useful when they are used with higher
level collections such as ReplicaSets, StatefulSets, Deployments, etc.  One can easily configure that a set of workloads should
be co-located in the same defined topology, eg., the same node.
-->

Pod 间亲和与反亲和在与更高级别的集合（例如
ReplicaSets，StatefulSets，Deployments 等）一起使用时，它们可能更加有用。可以轻
松配置一组应位于相同定义拓扑（例如，节点）中的工作负载。

<!--
##### Always co-located in the same node
-->

##### 始终放置在相同节点上

<!--
In a three node cluster, a web application has in-memory cache such as redis. We want the web-servers to be co-located with the cache as much as possible.
-->

在三节点集群中，一个 web 应用程序具有内存缓存，例如 redis。我们希望 web 服务器尽
可能与缓存放置在同一位置。

<!--
Here is the yaml snippet of a simple redis deployment with three replicas and selector label `app=store`. The deployment has `PodAntiAffinity` configured to ensure the scheduler does not co-locate replicas on a single node.
-->

下面是一个简单 redis deployment 的 yaml 代码段，它有三个副本和选择器标签
`app=store`。Deployment 配置了 `PodAntiAffinity`，用来确保调度器不会将副本调度到
单个节点上。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-cache
spec:
  selector:
    matchLabels:
      app: store
  replicas: 3
  template:
    metadata:
      labels:
        app: store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - store
              topologyKey: "kubernetes.io/hostname"
      containers:
        - name: redis-server
          image: redis:3.2-alpine
```

<!--
The below yaml snippet of the webserver deployment has `podAntiAffinity` and `podAffinity` configured. This informs the scheduler that all its replicas are to be co-located with pods that have selector label `app=store`. This will also ensure that each web-server replica does not co-locate on a single node.
-->

下面 webserver deployment 的 yaml 代码段中配置了 `podAntiAffinity` 和
`podAffinity`。这将通知调度器将它的所有副本与具有 `app=store` 选择器标签的 pod
放置在一起。这还确保每个 web 服务器副本不会调度到单个节点上。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server
spec:
  selector:
    matchLabels:
      app: web-store
  replicas: 3
  template:
    metadata:
      labels:
        app: web-store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - web-store
              topologyKey: "kubernetes.io/hostname"
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - store
              topologyKey: "kubernetes.io/hostname"
      containers:
        - name: web-app
          image: nginx:1.16-alpine
```

<!--
If we create the above two deployments, our three node cluster should look like below.
-->

如果我们创建了上面的两个 deployment，我们的三节点集群将如下表所示。

|    node-1     |    node-2     |    node-3     |
| :-----------: | :-----------: | :-----------: |
| _webserver-1_ | _webserver-2_ | _webserver-3_ |
|   _cache-1_   |   _cache-2_   |   _cache-3_   |

<!--
As you can see, all the 3 replicas of the `web-server` are automatically co-located with the cache as expected.
-->

如你所见，`web-server` 的三个副本都按照预期那样自动放置在同一位置。

```
kubectl get pods -o wide
```

<!--
The output is similar to this:
-->

输出类似于如下内容：

```
NAME                           READY     STATUS    RESTARTS   AGE       IP           NODE
redis-cache-1450370735-6dzlj   1/1       Running   0          8m        10.192.4.2   kube-node-3
redis-cache-1450370735-j2j96   1/1       Running   0          8m        10.192.2.2   kube-node-1
redis-cache-1450370735-z73mh   1/1       Running   0          8m        10.192.3.1   kube-node-2
web-server-1287567482-5d4dz    1/1       Running   0          7m        10.192.2.3   kube-node-1
web-server-1287567482-6f7v5    1/1       Running   0          7m        10.192.4.3   kube-node-3
web-server-1287567482-s330j    1/1       Running   0          7m        10.192.3.2   kube-node-2
```

<!--
##### Never co-located in the same node
-->

##### 永远不放置在相同节点

<!--
The above example uses `PodAntiAffinity` rule with `topologyKey: "kubernetes.io/hostname"` to deploy the redis cluster so that
no two instances are located on the same host.
See [ZooKeeper tutorial](/docs/tutorials/stateful-application/zookeeper/#tolerating-node-failure)
for an example of a StatefulSet configured with anti-affinity for high availability, using the same technique.
-->

上面的例子使用 `PodAntiAffinity` 规则和 `topologyKey: "kubernetes.io/hostname"`
来部署 redis 集群以便在同一主机上没有两个实例。参阅
[ZooKeeper 教程](/docs/tutorials/stateful-application/zookeeper/#tolerating-node-failure)，
以获取配置反亲和来达到高可用性的 StatefulSet 的样例（使用了相同的技巧）。

## nodeName

<!--
`nodeName` is the simplest form of node selection constraint, but due
to its limitations it is typically not used.  `nodeName` is a field of
PodSpec.  If it is non-empty, the scheduler ignores the pod and the
kubelet running on the named node tries to run the pod.  Thus, if
`nodeName` is provided in the PodSpec, it takes precedence over the
above methods for node selection.
-->

`nodeName` 是节点选择约束的最简单方法，但是由于其自身限制，通常不使用它
。`nodeName` 是 PodSpec 的一个字段。如果它不为空，调度器将忽略 pod，并且运行在它
指定节点上的 kubelet 进程尝试运行该 pod。因此，如果 `nodeName` 在 PodSpec 中指定
了，则它优先于上面的节点选择方法。

<!--
Some of the limitations of using `nodeName` to select nodes are:
-->

使用 `nodeName` 来选择节点的一些限制：

<!--
-   If the named node does not exist, the pod will not be run, and in
    some cases may be automatically deleted.
-   If the named node does not have the resources to accommodate the
    pod, the pod will fail and its reason will indicate why,
    e.g. OutOfmemory or OutOfcpu.
-   Node names in cloud environments are not always predictable or
    stable.
-->

- 如果指定的节点不存在，
- 如果指定的节点没有资源来容纳 pod，pod 将会调度失败并且其原因将显示为，比如
  OutOfmemory 或 OutOfcpu。
- 云环境中的节点名称并非总是可预测或稳定的。

<!--
Here is an example of a pod config file using the `nodeName` field:
-->

下面的是使用 `nodeName` 字段的 pod 配置文件的例子：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
  nodeName: kube-01
```

<!--
The above pod will run on the node kube-01.
-->

上面的 pod 将运行在 kube-01 节点上。

{{% /capture %}}

{{% capture whatsnext %}}

<!--
[Taints](/docs/concepts/configuration/taint-and-toleration/) allow a Node to *repel* a set of Pods.
-->

[污点](/docs/concepts/configuration/taint-and-toleration/)允许节点*排斥*一组
pod。

<!--
The design documents for
[node affinity](https://git.k8s.io/community/contributors/design-proposals/scheduling/nodeaffinity.md)
and for [inter-pod affinity/anti-affinity](https://git.k8s.io/community/contributors/design-proposals/scheduling/podaffinity.md) contain extra background information about these features.
-->

[节点亲和](https://git.k8s.io/community/contributors/design-proposals/scheduling/nodeaffinity.md)与
[pod 间亲和/反亲和](https://git.k8s.io/community/contributors/design-proposals/scheduling/podaffinity.md)的
设计文档包含这些功能的其他背景信息。

<!--
Once a Pod is assigned to a Node, the kubelet runs the Pod and allocates node-local resources.
The [topology manager](/docs/tasks/administer-cluster/topology-manager/) can take part in node-level
resource allocation decisions.
-->

一旦 pod 分配给 节点，kubelet 应用将运行该 pod 并且分配节点本地资源
。[拓扑管理](/docs/tasks/administer-cluster/topology-manager/)

{{% /capture %}}
