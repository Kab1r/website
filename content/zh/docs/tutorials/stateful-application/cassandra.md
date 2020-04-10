---
title: "示例：使用 Stateful Sets 部署 Cassandra"
---

<!--
title: "Example: Deploying Cassandra with Stateful Sets"
-->

## 目录

- [准备工作](#prerequisites)
- [Cassandra docker 镜像](#cassandra-docker)
- [快速入门](#quickstart)
- [步骤 1：创建 Cassandra Headless Service](#step-1-create-a-cassandra-headless-service)
- [步骤 2：使用 StatefulSet 创建 Cassandra Ring 环](#step-2-use-a-statefulset-to-create-cassandra-ring)
- [步骤 3：验证并修改 Cassandra StatefulSet](#step-3-validate-and-modify-the-cassandra-statefulset)
- [步骤 4：删除 Cassandra StatefulSet](#step-4-delete-cassandra-statefulset)
- [步骤 5：使用 Replication Controller 创建 Cassandra 节点 pods](#step-5-use-a-replication-controller-to-create-cassandra-node-pods)
- [步骤 6：Cassandra 集群扩容](#step-6-scale-up-the-cassandra-cluster)
- [步骤 7：删除 Replication Controller](#step-7-delete-the-replication-controller)
- [步骤 8：使用 DaemonSet 替换 Replication Controller](#step-8-use-a-daemonset-instead-of-a-replication-controller)
- [步骤 9：资源清理](#step-9-resource-cleanup)
- [Seed Provider Source](#seed-provider-source)

下文描述了在 Kubernetes 上部署一个*云原生*
[Cassandra](http://cassandra.apache.org/) 的过程。当我们说*云原生*时，指的是一个
应用能够理解它运行在一个集群管理器内部，并且使用这个集群的管理基础设施来帮助实现
这个应用。特别的，本例使用了一个自定义的 Cassandra `SeedProvider` 帮助 Cassandra
发现新加入集群 Cassandra 节点。

本示例也使用了 Kubernetes 的一些核心组件：

- [_Pods_](/zh/docs/user-guide/pods)
- [ _Services_](/zh/docs/user-guide/services)
- [_Replication Controllers_](/zh/docs/user-guide/replication-controller)
- [_Stateful Sets_](/zh/docs/concepts/workloads/controllers/statefulset/)
- [_Daemon Sets_](/zh/docs/admin/daemons)

## 准备工作

本示例假设你已经安装运行了一个 Kubernetes 集群（版本 >=1.2），并且还在某个路径下
安装了 [`kubectl`](/zh/docs/tasks/tools/install-kubectl/) 命令行工具。请查看
[getting started guides](/zh/docs/getting-started-guides/) 获取关于你的平台的安
装说明。

本示例还需要一些代码和配置文件。为了避免手动输入，你可以 `git clone` Kubernetes
源到你本地。

## Cassandra Docker 镜像

Pod 使用来自 Google
[容器仓库](https://cloud.google.com/container-registry/docs/) 的
[`gcr.io/google-samples/cassandra:v12`](https://github.com/kubernetes/examples/blob/master/cassandra/image/Dockerfile)
镜像。这个 docker 镜像基于 `debian:jessie` 并包含 OpenJDK 8。该镜像包含一个从
Apache Debian 源中安装的标准 Cassandra。你可以通过使用环境变量改变插入到
`cassandra.yaml` 文件中的参数值。

| ENV VAR                | DEFAULT VALUE  |
| ---------------------- | :------------: |
| CASSANDRA_CLUSTER_NAME | 'Test Cluster' |
| CASSANDRA_NUM_TOKENS   |       32       |
| CASSANDRA_RPC_ADDRESS  |    0.0.0.0     |

## 快速入门

{{< codenew file="application/cassandra/cassandra-service.yaml" >}}

如果你希望直接跳到我们使用的命令，以下是全部步骤：

<!--
# clone the example repository
# create a service to track all cassandra statefulset nodes
# create a statefulset
# validate the Cassandra cluster. Substitute the name of one of your pods.
# cleanup
-->

```sh

kubectl apply -f https://k8s.io/examples/application/cassandra/cassandra-service.yaml
```

{{< codenew file="application/cassandra/cassandra-statefulset.yaml" >}}

```
# 创建 statefulset
kubectl apply -f https://k8s.io/examples/application/cassandra/cassandra-statefulset.yaml

# 验证 Cassandra 集群。替换一个 pod 的名称。
kubectl exec -ti cassandra-0 -- nodetool status

# 清理
grace=$(kubectl get po cassandra-0 -o=jsonpath='{.spec.terminationGracePeriodSeconds}') \
  && kubectl delete statefulset,po -l app=cassandra \
  && echo "Sleeping $grace" \
  && sleep $grace \
  && kubectl delete pvc -l app=cassandra

#
# 资源控制器示例
#

# 创建一个副本控制器来复制 cassandra 节点
kubectl create -f cassandra/cassandra-controller.yaml

# 验证 Cassandra 集群。替换一个 pod 的名称。
kubectl exec -ti cassandra-xxxxx -- nodetool status

# 扩大 Cassandra 集群
kubectl scale rc cassandra --replicas=4

# 删除副本控制器
kubectl delete rc cassandra

#
# 创建一个 DaemonSet，在每个 kubernetes 节点上放置一个 cassandra 节点
#

kubectl create -f cassandra/cassandra-daemonset.yaml --validate=false

# 资源清理
kubectl delete service -l app=cassandra
kubectl delete daemonset cassandra
```

<!--
# Resource Controller Example
# create a replication controller to replicate cassandra nodes
# validate the Cassandra cluster. Substitute the name of one of your pods.
# scale up the Cassandra cluster
# delete the replication controller
# Create a DaemonSet to place a cassandra node on each kubernetes node
# resource cleanup
-->

## 步骤 1：创建 Cassandra Headless Service

Kubernetes _[Service](/zh/docs/user-guide/services)_ 描述一组执行同样任务的
[_Pod_](/zh/docs/user-guide/pods)。在 Kubernetes 中，一个应用的原子调度单位是一
个 Pod：一个或多个*必须*调度到相同主机上的容器。

这个 Service 用于在 Kubernetes 集群内部进行 Cassandra 客户端和 Cassandra Pod 之
间的 DNS 查找。

以下为这个 service 的描述：

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: cassandra
  name: cassandra
spec:
  clusterIP: None
  ports:
    - port: 9042
  selector:
    app: cassandra
```

Download
[`cassandra-service.yaml`](/examples/application/cassandra/cassandra-service.yaml)
and
[`cassandra-statefulset.yaml`](/examples/application/cassandra/cassandra-statefulset.yaml)

为 StatefulSet 创建 service

```console
kubectl apply -f https://k8s.io/examples/application/cassandra/cassandra-service.yaml
```

以下命令显示了 service 是否被成功创建。

```console
$ kubectl get svc cassandra
```

命令的响应应该像这样：

```console
NAME        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
cassandra   None         <none>        9042/TCP   45s
```

如果返回错误则表示 service 创建失败。

## 步骤 2：使用 StatefulSet 创建 Cassandra Ring 环

StatefulSets（以前叫做 PetSets）特性在 Kubernetes 1.5 中升级为一个 <i>Beta</i>
组件。在集群环境中部署类似于 Cassandra 的有状态分布式应用是一项具有挑战性的工作
。我们实现了 StatefulSet，极大的简化了这个过程。本示例使用了 StatefulSet 的多个
特性，但其本身超出了本文的范围
。[请参考 StatefulSet 文档](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)。

以下是 StatefulSet 的清单文件，用于创建一个由三个 pod 组成的 Cassandra ring 环。

本示例使用了 GCE Storage Class，请根据你运行的云平台做适当的修改。

```yaml
apiVersion: "apps/v1beta1"
kind: StatefulSet
metadata:
  name: cassandra
spec:
  serviceName: cassandra
  replicas: 3
  template:
    metadata:
      labels:
        app: cassandra
    spec:
      containers:
        - name: cassandra
          image: gcr.io/google-samples/cassandra:v12
          imagePullPolicy: Always
          ports:
            - containerPort: 7000
              name: intra-node
            - containerPort: 7001
              name: tls-intra-node
            - containerPort: 7199
              name: jmx
            - containerPort: 9042
              name: cql
          resources:
            limits:
              cpu: "500m"
              memory: 1Gi
            requests:
              cpu: "500m"
              memory: 1Gi
          securityContext:
            capabilities:
              add:
                - IPC_LOCK
          lifecycle:
            preStop:
              exec:
                command:
                  [
                    "/bin/sh",
                    "-c",
                    "PID=$(pidof java) && kill $PID && while ps -p $PID >
                    /dev/null; do sleep 1; done",
                  ]
          env:
            - name: MAX_HEAP_SIZE
              value: 512M
            - name: HEAP_NEWSIZE
              value: 100M
            - name: CASSANDRA_SEEDS
              value: "cassandra-0.cassandra.default.svc.cluster.local"
            - name: CASSANDRA_CLUSTER_NAME
              value: "K8Demo"
            - name: CASSANDRA_DC
              value: "DC1-K8Demo"
            - name: CASSANDRA_RACK
              value: "Rack1-K8Demo"
            - name: CASSANDRA_AUTO_BOOTSTRAP
              value: "false"
            - name: POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
          readinessProbe:
            exec:
              command:
                - /bin/bash
                - -c
                - /ready-probe.sh
            initialDelaySeconds: 15
            timeoutSeconds: 5
          # These volume mounts are persistent. They are like inline claims,
          # but not exactly because the names need to match exactly one of
          # the stateful pod volumes.
          volumeMounts:
            - name: cassandra-data
              mountPath: /cassandra_data
  # These are converted to volume claims by the controller
  # and mounted at the paths mentioned above.
  # do not use these in production until ssd GCEPersistentDisk or other ssd pd
  volumeClaimTemplates:
    - metadata:
        name: cassandra-data
        annotations:
          volume.beta.kubernetes.io/storage-class: fast
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
---
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```

创建 Cassandra StatefulSet 如下：

```console
kubectl apply -f https://k8s.io/examples/application/cassandra/cassandra-statefulset.yaml
```

## 步骤 3：验证和修改 Cassandra StatefulSet

这个 StatefulSet 的部署展示了 StatefulSets 提供的两个新特性：

1. Pod 的名称已知
2. Pod 以递增顺序部署

首先，运行下面的 `kubectl` 命令，验证 StatefulSet 已经被成功部署。

```console
$ kubectl get statefulset cassandra
```

这个命令的响应应该像这样：

```console
NAME        DESIRED   CURRENT   AGE
cassandra   3         3         13s
```

接下来观察 Cassandra pod 以一个接一个的形式部署。StatefulSet 资源按照数字序号的
模式部署 pod：1, 2, 3 等。如果在 pod 部署前执行下面的命令，你就能够看到这种顺序
的创建过程。

```console
$ kubectl get pods -l="app=cassandra"
NAME          READY     STATUS              RESTARTS   AGE
cassandra-0   1/1       Running             0          1m
cassandra-1   0/1       ContainerCreating   0          8s
```

上面的示例显示了三个 Cassandra StatefulSet pod 中的两个已经部署。一旦所有的 pod
都部署成功，相同的命令会显示一个完整的 StatefulSet。

```console
$ kubectl get pods -l="app=cassandra"
NAME          READY     STATUS    RESTARTS   AGE
cassandra-0   1/1       Running   0          10m
cassandra-1   1/1       Running   0          9m
cassandra-2   1/1       Running   0          8m
```

运行 Cassandra 工具 `nodetool` 将显示 ring 环的状态。

```console
$ kubectl exec cassandra-0 -- nodetool status
Datacenter: DC1-K8Demo
======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address   Load       Tokens       Owns (effective)  Host ID                               Rack
UN  10.4.2.4  65.26 KiB  32           63.7%             a9d27f81-6783-461d-8583-87de2589133e  Rack1-K8Demo
UN  10.4.0.4  102.04 KiB  32           66.7%             5559a58c-8b03-47ad-bc32-c621708dc2e4  Rack1-K8Demo
UN  10.4.1.4  83.06 KiB  32           69.6%             9dce943c-581d-4c0e-9543-f519969cc805  Rack1-K8Demo
```

你也可以运行 `cqlsh` 来显示集群的 keyspaces。

```console
$ kubectl exec cassandra-0 -- cqlsh -e 'desc keyspaces'

system_traces  system_schema  system_auth  system  system_distributed
```

你需要使用 `kubectl edit` 来增加或减小 Cassandra StatefulSet 的大小。你可以
在[文档](/zh/docs/user-guide/kubectl/kubectl_edit) 中找到更多关于 `edit` 命令的
信息。

使用以下命令编辑 StatefulSet。

```console
$ kubectl edit statefulset cassandra
```

这会在你的命令行中创建一个编辑器。你需要修改的行是 `replicas`。这个例子没有包含
终端窗口的所有内容，下面示例中的最后一行就是你希望改变的 replicas 行。

```console
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  creationTimestamp: 2016-08-13T18:40:58Z
  generation: 1
  labels:
    app: cassandra
  name: cassandra
  namespace: default
  resourceVersion: "323"
  uid: 7a219483-6185-11e6-a910-42010a8a0fc0
spec:
  replicas: 3
```

按下面的示例修改清单文件并保存。

```console
spec:
  replicas: 4
```

这个 StatefulSet 现在将包含四个 pod。

```console
$ kubectl get statefulset cassandra
```

这个 command 的响应应该像这样：

```console
NAME        DESIRED   CURRENT   AGE
cassandra   4         4         36m
```

对于 Kubernetes 1.5 发布版，beta StatefulSet 资源没有像 Deployment, ReplicaSet,
Replication Controller 或者 Job 一样，包含 `kubectl scale` 功能，

## 步骤 4：删除 Cassandra StatefulSet

删除或者缩容 StatefulSet 时不会删除与之关联的 volumes。这样做是为了优先保证安全
。你的数据比其它会被自动清除的 StatefulSet 关联资源更宝贵。删除 Persistent
Volume Claims 可能会导致关联的 volumes 被删除，这种行为依赖 storage class 和
reclaim policy。永远不要期望能在 claim 删除后访问一个 volume。

使用如下命令删除 StatefulSet。

```console
$ grace=$(kubectl get po cassandra-0 -o=jsonpath='{.spec.terminationGracePeriodSeconds}') \
  && kubectl delete statefulset -l app=cassandra \
  && echo "Sleeping $grace" \
  && sleep $grace \
  && kubectl delete pvc -l app=cassandra
```

## 步骤 5：使用 Replication Controller 创建 Cassandra 节点 pod

Kubernetes
_[Replication Controller](/zh/docs/user-guide/replication-controller)_ 负责复制
一个完全相同的 pod 集合。像 Service 一样，它具有一个 selector query，用来识别它
的集合成员。和 Service 不一样的是，它还具有一个期望的副本数，并且会通过创建或�
除 Pod 来保证 Pod 的数量满足它期望的状态。

和我们刚才定义的 Service 一起，Replication Controller 能够让我们轻松的构建一个复
制的、可扩展的 Cassandra 集群。

让我们创建一个具有两个初始副本的 replication controller。

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: cassandra
  # The labels will be applied automatically
  # from the labels in the pod template, if not set
  # labels:
  # app: cassandra
spec:
  replicas: 2
  # The selector will be applied automatically
  # from the labels in the pod template, if not set.
  # selector:
  # app: cassandra
  template:
    metadata:
      labels:
        app: cassandra
    spec:
      containers:
        - command:
            - /run.sh
          resources:
            limits:
              cpu: 0.5
          env:
            - name: MAX_HEAP_SIZE
              value: 512M
            - name: HEAP_NEWSIZE
              value: 100M
            - name: CASSANDRA_SEED_PROVIDER
              value: "io.k8s.cassandra.KubernetesSeedProvider"
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
            - name: POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
          image: gcr.io/google-samples/cassandra:v12
          name: cassandra
          ports:
            - containerPort: 7000
              name: intra-node
            - containerPort: 7001
              name: tls-intra-node
            - containerPort: 7199
              name: jmx
            - containerPort: 9042
              name: cql
          volumeMounts:
            - mountPath: /cassandra_data
              name: data
      volumes:
        - name: data
          emptyDir: {}
```

[下载示例](https://raw.githubusercontent.com/kubernetes/examples/master/cassandra-controller.yaml)

在这个描述中需要注意几件事情。

`selector` 属性包含了控制器的 selector query。它能够被显式指定，或者在没有设置时
，像此处一样从 pod 模板中的 labels 中自动应用。

Pod 模板的标签 `app:cassandra` 匹配步骤 1 中的 Service selector。这就是 Service
如何选择 replication controller 创建的 pod 的原理。

`replicas` 属性指明了期望的副本数量，在本例中最开始为 2。我们很快将要扩容更多数
量。

创建 Replication Controller：

```console

$ kubectl create -f cassandra/cassandra-controller.yaml

```

你可以列出新建的 controller：

```console

$ kubectl get rc -o wide
NAME        DESIRED   CURRENT   AGE       CONTAINER(S)   IMAGE(S)                             SELECTOR
cassandra   2         2         11s       cassandra      gcr.io/google-samples/cassandra:v12   app=cassandra

```

现在，如果你列出集群中的 pod，并且使用 `app=cassandra` 标签过滤，你应该能够看到
两个 Cassandra pod。（`wide` 参数使你能够看到 pod 被调度到了哪个 Kubernetes 节点
上）

```console
$ kubectl get pods -l="app=cassandra" -o wide
NAME              READY     STATUS    RESTARTS   AGE       NODE
cassandra-21qyy   1/1       Running   0          1m        kubernetes-minion-b286
cassandra-q6sz7   1/1       Running   0          1m        kubernetes-minion-9ye5
```

因为这些 pod 拥有 `app=cassandra` 标签，它们被映射给了我们在步骤 1 中创建的
service。

你可以使用下面的 service endpoint 查询命令来检查 Pod 是否对 Service 可用。

```console

$ kubectl get endpoints cassandra -o yaml
apiVersion: v1
kind: Endpoints
metadata:
  creationTimestamp: 2015-06-21T22:34:12Z
  labels:
    app: cassandra
  name: cassandra
  namespace: default
  resourceVersion: "944373"
  uid: a3d6c25f-1865-11e5-a34e-42010af01bcc
subsets:
- addresses:
  - ip: 10.244.3.15
    targetRef:
      kind: Pod
      name: cassandra
      namespace: default
      resourceVersion: "944372"
      uid: 9ef9895d-1865-11e5-a34e-42010af01bcc
  ports:
  - port: 9042
    protocol: TCP

```

为了显示 `SeedProvider` 逻辑是按设想在运行，你可以使用 `nodetool` 命令来检查
Cassandra 集群的状态。为此，请使用 `kubectl exec` 命令，这样你就能在一个
Cassandra pod 上运行 `nodetool`。同样的，请替换 `cassandra-xxxxx` 为任意一个
pods 的真实名字。

```console

$ kubectl exec -ti cassandra-xxxxx -- nodetool status
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens  Owns (effective)  Host ID                               Rack
UN  10.244.0.5  74.09 KB   256     100.0%            86feda0f-f070-4a5b-bda1-2eeb0ad08b77  rack1
UN  10.244.3.3  51.28 KB   256     100.0%            dafe3154-1d67-42e1-ac1d-78e7e80dce2b  rack1

```

## 步骤 6：Cassandra 集群扩容

现在，让我们把 Cassandra 集群扩展到 4 个 pod。我们通过告诉 Replication
Controller 现在我们需要 4 个副本来完成。

```sh

$ kubectl scale rc cassandra --replicas=4

```

你可以看到列出了新的 pod：

```console

$ kubectl get pods -l="app=cassandra" -o wide
NAME              READY     STATUS    RESTARTS   AGE       NODE
cassandra-21qyy   1/1       Running   0          6m        kubernetes-minion-b286
cassandra-81m2l   1/1       Running   0          47s       kubernetes-minion-b286
cassandra-8qoyp   1/1       Running   0          47s       kubernetes-minion-9ye5
cassandra-q6sz7   1/1       Running   0          6m        kubernetes-minion-9ye5

```

一会儿你就能再次检查 Cassandra 集群的状态，你可以看到新的 pod 已经被自定义的
`SeedProvider` 检测到：

```console

$ kubectl exec -ti cassandra-xxxxx -- nodetool status
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens  Owns (effective)  Host ID                               Rack
UN  10.244.0.6  51.67 KB   256     48.9%             d07b23a5-56a1-4b0b-952d-68ab95869163  rack1
UN  10.244.1.5  84.71 KB   256     50.7%             e060df1f-faa2-470c-923d-ca049b0f3f38  rack1
UN  10.244.1.6  84.71 KB   256     47.0%             83ca1580-4f3c-4ec5-9b38-75036b7a297f  rack1
UN  10.244.0.5  68.2 KB    256     53.4%             72ca27e2-c72c-402a-9313-1e4b61c2f839  rack1

```

## 步骤 7：删除 Replication Controller

在你开始步骤 5 之前， **删除**你在上面创建的 **replication controller**。

```sh

$ kubectl delete rc cassandra

```

## 步骤 8：使用 DaemonSet 替换 Replication Controller

在 Kubernetes 中，[_DaemonSet_](/zh/docs/admin/daemons) 能够将 pod 一对一的分布
到 Kubernetes 节点上。和 _ReplicationController_ 相同的是它也有一个用于识别它的
集合成员的 selector query。但和 _ReplicationController_ 不同的是，它拥有一个节点
selector，用于限制基于模板的 pod 可以调度的节点。并且 pod 的复制不是基于一个设置
的数量，而是为每一个节点分配一个 pod。

示范用例：当部署到云平台时，预期情况是实例是短暂的并且随时可能终止。Cassandra 被
搭建成为在各个节点间复制数据以便于实现数据冗余。这样的话，即使一个实例终止了，存
储在它上面的数据却没有，并且集群会通过重新复制数据到其它运行节点来作为响应。

`DaemonSet` 设计为在 Kubernetes 集群中的每个节点上放置一个 pod。那样就会给我们带
来数据冗余度。让我们创建一个 DaemonSet 来启动我们的存储集群：

```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  labels:
    name: cassandra
  name: cassandra
spec:
  template:
    metadata:
      labels:
        app: cassandra
    spec:
      # Filter to specific nodes:
      # nodeSelector:
      #  app: cassandra
      containers:
        - command:
            - /run.sh
          env:
            - name: MAX_HEAP_SIZE
              value: 512M
            - name: HEAP_NEWSIZE
              value: 100M
            - name: CASSANDRA_SEED_PROVIDER
              value: "io.k8s.cassandra.KubernetesSeedProvider"
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
            - name: POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
          image: gcr.io/google-samples/cassandra:v12
          name: cassandra
          ports:
            - containerPort: 7000
              name: intra-node
            - containerPort: 7001
              name: tls-intra-node
            - containerPort: 7199
              name: jmx
            - containerPort: 9042
              name: cql
              # If you need it, it will go away in C* 4.0.
              #- containerPort: 9160
              #  name: thrift
          resources:
            requests:
              cpu: 0.5
          volumeMounts:
            - mountPath: /cassandra_data
              name: data
      volumes:
        - name: data
          emptyDir: {}
```

[下载示例](https://raw.githubusercontent.com/kubernetes/examples/master/cassandra-daemonset.yaml)

这个 DaemonSet 绝大部分的定义和上面的 ReplicationController 完全相同；它只是简单
的给 daemonset 一个创建新的 Cassandra pod 的方法，并且以集群中所有的 Cassandra
节点为目标。

不同之处在于 `nodeSelector` 属性，它允许 DaemonSet 以全部节点的一个子集为目标（
你可以向其他资源一样标记节点），并且没有 `replicas` 属性，因为它使用 1 对 1 的
node-pod 关系。

创建这个 DaemonSet：

```console

$ kubectl create -f cassandra/cassandra-daemonset.yaml

```

你可能需要禁用配置文件检查，像这样：

```console

$ kubectl create -f cassandra/cassandra-daemonset.yaml --validate=false

```

你可以看到 DaemonSet 已经在运行：

```console

$ kubectl get daemonset
NAME        DESIRED   CURRENT   NODE-SELECTOR
cassandra   3         3         <none>

```

现在，如果你列出集群中的 pods，并且使用 `app=cassandra` 标签过滤，你应该能够看到
你的网络中的每一个节点上都有一个（且只有一个）新的 cassandra pod。

```console

$ kubectl get pods -l="app=cassandra" -o wide
NAME              READY     STATUS    RESTARTS   AGE       NODE
cassandra-ico4r   1/1       Running   0          4s        kubernetes-minion-rpo1
cassandra-kitfh   1/1       Running   0          1s        kubernetes-minion-9ye5
cassandra-tzw89   1/1       Running   0          2s        kubernetes-minion-b286

```

为了证明这是按设想的在工作，你可以再次使用 `nodetool` 命令来检查集群的状态。为此
，请使用 `kubectl exec` 命令在任何一个新建的 cassandra pod 上运行 `nodetool`。

```console

$ kubectl exec -ti cassandra-xxxxx -- nodetool status
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens  Owns (effective)  Host ID                               Rack
UN  10.244.0.5  74.09 KB   256     100.0%            86feda0f-f070-4a5b-bda1-2eeb0ad08b77  rack1
UN  10.244.4.2  32.45 KB   256     100.0%            0b1be71a-6ffb-4895-ac3e-b9791299c141  rack1
UN  10.244.3.3  51.28 KB   256     100.0%            dafe3154-1d67-42e1-ac1d-78e7e80dce2b  rack1

```

**注意**：这个示例让你在创建 DaemonSet 前删除了 cassandra 的 Replication
Controller。这是因为为了保持示例的简单，RC 和 DaemonSet 使用了相同的
`app=cassandra` 标签（如此它们的 pod 映射到了我们创建的 service，这样
SeedProvider 就能识别它们）。

如果我们没有预先删除 RC，这两个资源在需要运行多少 pod 上将会发生冲突。如果希望的
话，我们可以使用额外的标签和 selectors 来支持同时运行它们。

## 步骤 9：资源清理

当你准备删除你的资源时，按以下执行：

```console

$ kubectl delete service -l app=cassandra
$ kubectl delete daemonset cassandra

```

### Seed Provider Source

我们使用了一个自定义的
[`SeedProvider`](https://gitbox.apache.org/repos/asf?p=cassandra.git;a=blob;f=src/java/org/apache/cassandra/locator/SeedProvider.java;h=7efa9e050a4604c2cffcb953c3c023a2095524fe;hb=c2e11bd4224b2110abe6aa84c8882e85980e3491)
来在 Kubernetes 之上运行 Cassandra。仅当你通过 replication control 或者
daemonset 部署 Cassandra 时才需要使用自定义的 seed provider。在 Cassandra 中
，`SeedProvider` 引导 Cassandra 使用 gossip 协议来查找其它 Cassandra 节点。Seed
地址是被视为连接端点的主机。Cassandra 实例使用 seed 列表来查找彼此并学习 ring 环
拓扑
。[`KubernetesSeedProvider`](https://github.com/kubernetes/examples/blob/master/cassandra/java/src/main/java/io/k8s/cassandra/KubernetesSeedProvider.java)
通过 Kubernetes API 发现 Cassandra seeds IP 地址，那些 Cassandra 实例在
Cassandra Service 中定义。

请查阅自定义 seed provider 的
[README](https://git.k8s.io/examples/cassandra/java/README.md) 文档，获取
`KubernetesSeedProvider` 进阶配置。对于本示例来说，你应该不需要自定义 Seed
Provider 的配置。

查看本示例的
[image](https://github.com/kubernetes/examples/tree/master/cassandra/image) 目录
，了解如何构建容器的 docker 镜像及其内容。

你可能还注意到我们设置了一些 Cassandra 参数（`MAX_HEAP_SIZE`和`HEAP_NEWSIZE`），
并且增加了关于 [namespace](/zh/docs/user-guide/namespaces) 的信息。我们还告诉
Kubernetes 容器暴露了 `CQL` 和 `Thrift` API 端口。最后，我们告诉集群管理器我们需
要 0.1 cpu（0.1 核）。

[!Analytics](https://kubernetes-site.appspot.com/UA-36037335-10/GitHub/cassandra/README.md?pixel)]()
