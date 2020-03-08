## <!--

title: " Simple leader election with Kubernetes and Docker " date: 2016-01-11
slug: simple-leader-election-with-kubernetes

---

#### Overview

-->

title: "Kubernetes 和 Docker 简单的 leader election" date: 2016-01-11 slug:
simple-leader-election-with-kubernetes url:
/blog/2016/01/Simple-Leader-Election-With-Kubernetes

<!--
Kubernetes simplifies the deployment and operational management of services running on clusters. However, it also simplifies the development of these services. In this post we'll see how you can use Kubernetes to easily perform leader election in your distributed application. Distributed applications usually replicate the tasks of a service for reliability and scalability, but often it is necessary to designate one of the replicas as the leader who is responsible for coordination among all of the replicas.
-->

Kubernetes 简化了集群上运行的服务的部署和操作管理。但是，它也简化了这些服务的发
展。在本文中，我们将看到如何使用 Kubernetes 在分布式应用程序中轻松地执行 leader
election。分布式应用程序通常为了可靠性和可伸缩性而复制服务的任务，但通常需要指定
其中一个副本作为负责所有副本之间协调的负责人。

<!--
Typically in leader election, a set of candidates for becoming leader is identified. These candidates all race to declare themselves the leader. One of the candidates wins and becomes the leader. Once the election is won, the leader continually "heartbeats" to renew their position as the leader, and the other candidates periodically make new attempts to become the leader. This ensures that a new leader is identified quickly, if the current leader fails for some reason.
-->

通常在 leader election 中，会确定一组成为领导者的候选人。这些候选人都竞相宣布自
己为领袖。其中一位候选人获胜并成为领袖。一旦选举获胜，领导者就会不断地“信号”以表
示他们作为领导者的地位，其他候选人也会定期地做出新的尝试来成为领导者。这样可以确
保在当前领导因某种原因失败时，快速确定新领导。

<!--
Implementing leader election usually requires either deploying software such as ZooKeeper, etcd or Consul and using it for consensus, or alternately, implementing a consensus algorithm on your own. We will see below that Kubernetes makes the process of using leader election in your application significantly easier.

####  Implementing leader election in Kubernetes
-->

实现 leader election 通常需要部署 ZooKeeper、etcd 或 Consul 等软件并将其用于协商
一致，或者也可以自己实现协商一致算法。我们将在下面看到，Kubernetes 使在应用程序
中使用 leader election 的过程大大简化。

####在 Kubernetes 实施领导人选举

<!--
The first requirement in leader election is the specification of the set of candidates for becoming the leader. Kubernetes already uses _Endpoints_ to represent a replicated set of pods that comprise a service, so we will re-use this same object. (aside: You might have thought that we would use _ReplicationControllers_, but they are tied to a specific binary, and generally you want to have a single leader even if you are in the process of performing a rolling update)

To perform leader election, we use two properties of all Kubernetes API objects:
-->

Leader election 的首要条件是确定候选人的人选。Kubernetes 已经使用 _Endpoints_ 来
表示组成服务的一组复制 pod，因此我们将重用这个相同的对象。（旁白：您可能认为我们
会使用 _ReplicationControllers_，但它们是绑定到特定的二进制文件，而且通常您希望
只有一个领导者，即使您正在执行滚动更新）

要执行 leader election，我们使用所有 Kubernetes api 对象的两个属性：

<!--
* ResourceVersions - Every API object has a unique ResourceVersion, and you can use these versions to perform compare-and-swap on Kubernetes objects
* Annotations - Every API object can be annotated with arbitrary key/value pairs to be used by clients.

Given these primitives, the code to use master election is relatively straightforward, and you can find it [here][1]. Let's run it ourselves.
-->

- ResourceVersions - 每个 API 对象都有一个惟一的 ResourceVersion，您可以使用这些
  版本对 Kubernetes 对象执行比较和交换
- Annotations - 每个 API 对象都可以用客户端使用的任意键/值对进行注释。

给定这些原语，使用 master election 的代码相对简单，您可以在这里找到[here][1]。我
们自己来做吧。

<!--
```
$ kubectl run leader-elector --image=gcr.io/google_containers/leader-elector:0.4 --replicas=3 -- --election=example
```

This creates a leader election set with 3 replicas:

```
$ kubectl get pods
NAME                   READY     STATUS    RESTARTS   AGE
leader-elector-inmr1   1/1       Running   0          13s
leader-elector-qkq00   1/1       Running   0          13s
leader-elector-sgwcq   1/1       Running   0          13s
```
-->

```
$ kubectl run leader-elector --image=gcr.io/google_containers/leader-elector:0.4 --replicas=3 -- --election=example
```

这将创建一个包含 3 个副本的 leader election 集合：

```
$ kubectl get pods
NAME                   READY     STATUS    RESTARTS   AGE
leader-elector-inmr1   1/1       Running   0          13s
leader-elector-qkq00   1/1       Running   0          13s
leader-elector-sgwcq   1/1       Running   0          13s
```

<!--
To see which pod was chosen as the leader, you can access the logs of one of the pods, substituting one of your own pod's names in place of

```
${pod_name}, (e.g. leader-elector-inmr1 from the above)

$ kubectl logs -f ${name}
leader is (leader-pod-name)
```
… Alternately, you can inspect the endpoints object directly:
-->

要查看哪个 pod 被选为领导，您可以访问其中一个 pod 的日志，用您自己的一个 pod 的
名称替换

```
${pod_name}, (e.g. leader-elector-inmr1 from the above)

$ kubectl logs -f ${name}
leader is (leader-pod-name)
```

…或者，可以直接检查 endpoints 对象：

<!--
_'example' is the name of the candidate set from the above kubectl run … command_
```
$ kubectl get endpoints example -o yaml
```
Now to validate that leader election actually works, in a different terminal, run:

```
$ kubectl delete pods (leader-pod-name)
```
-->

*'example' 是上面 kubectl run … 命令*中候选集的名称

```
$ kubectl get endpoints example -o yaml
```

现在，要验证 leader election 是否实际有效，请在另一个终端运行：

```
$ kubectl delete pods (leader-pod-name)
```

<!--
This will delete the existing leader. Because the set of pods is being managed by a replication controller, a new pod replaces the one that was deleted, ensuring that the size of the replicated set is still three. Via leader election one of these three pods is selected as the new leader, and you should see the leader failover to a different pod. Because pods in Kubernetes have a _grace period_ before termination, this may take 30-40 seconds.

The leader-election container provides a simple webserver that can serve on any address (e.g. http://localhost:4040). You can test this out by deleting the existing leader election group and creating a new one where you additionally pass in a --http=(host):(port) specification to the leader-elector image. This causes each member of the set to serve information about the leader via a webhook.
-->

这将删除现有领导。由于 pod 集由 replication controller 管理，因此新的 pod 将替换
已删除的 pod，确保复制集的大小仍为 3。通过 leader election，这三个 pod 中的一个
被选为新的领导者，您应该会看到领导者故障转移到另一个 pod。因为 Kubernetes 的吊舱
在终止前有一个 _grace period_，这可能需要 30-40 秒。

Leader-election container 提供了一个简单的 web 服务器，可以服务于任何地址(e.g.
http://localhost:4040)。您可以通过删除现有的 leader election 组并创建一个新的
leader elector 组来测试这一点，在该组中，您还可以向 leader elector 映像传递
--http=(host):(port) 规范。这将导致集合中的每个成员通过 webhook 提供有关领导者的
信息。

<!--
```
# delete the old leader elector group
$ kubectl delete rc leader-elector

# create the new group, note the --http=localhost:4040 flag
$ kubectl run leader-elector --image=gcr.io/google_containers/leader-elector:0.4 --replicas=3 -- --election=example --http=0.0.0.0:4040

# create a proxy to your Kubernetes api server
$ kubectl proxy
```
-->

```
# delete the old leader elector group
$ kubectl delete rc leader-elector

# create the new group, note the --http=localhost:4040 flag
$ kubectl run leader-elector --image=gcr.io/google_containers/leader-elector:0.4 --replicas=3 -- --election=example --http=0.0.0.0:4040

# create a proxy to your Kubernetes api server
$ kubectl proxy
```

<!--
You can then access:


http://localhost:8001/api/v1/proxy/namespaces/default/pods/(leader-pod-name):4040/


And you will see:

```
{"name":"(name-of-leader-here)"}
```
####  Leader election with sidecars
-->

然后您可以访问：

http://localhost:8001/api/v1/proxy/namespaces/default/pods/(leader-pod-name):4040/

你会看到：

```
{"name":"(name-of-leader-here)"}
```

#### 有副手的 leader election

<!--
Ok, that's great, you can do leader election and find out the leader over HTTP, but how can you use it from your own application? This is where the notion of sidecars come in. In Kubernetes, Pods are made up of one or more containers. Often times, this means that you add sidecar containers to your main application to make up a Pod. (for a much more detailed treatment of this subject see my earlier blog post).

The leader-election container can serve as a sidecar that you can use from your own application. Any container in the Pod that's interested in who the current master is can simply access http://localhost:4040 and they'll get back a simple JSON object that contains the name of the current master. Since all containers in a Pod share the same network namespace, there's no service discovery required!
-->

好吧，那太好了，你可以通过 HTTP 进行 leader election 并找到 leader，但是你如何从
自己的应用程序中使用它呢？这就是 sidecar 的由来。Kubernetes 中，Pods 由一个或多
个容器组成。通常情况下，这意味着您将 sidecar containers 添加到主应用程序中以组成
pod。（关于这个主题的更详细的处理，请参阅我之前的博客文章）。 Leader-election
container 可以作为一个 sidecar，您可以从自己的应用程序中使用。Pod 中任何对当前
master 感兴趣的容器都可以简单地访问http://localhost:4040，它们将返回一个包含当前
master 名称的简单 json 对象。由于 pod 中 的所有容器共享相同的网络命名空间，因此
不需要服务发现！

<!--
For example, here is a simple Node.js application that connects to the leader election sidecar and prints out whether or not it is currently the master. The leader election sidecar sets its identifier to `hostname` by default.

```
var http = require('http');
// This will hold info about the current master
var master = {};

  // The web handler for our nodejs application
  var handleRequest = function(request, response) {
    response.writeHead(200);
    response.end("Master is " + master.name);
  };

  // A callback that is used for our outgoing client requests to the sidecar
  var cb = function(response) {
    var data = '';
    response.on('data', function(piece) { data = data + piece; });
    response.on('end', function() { master = JSON.parse(data); });
  };

  // Make an async request to the sidecar at http://localhost:4040
  var updateMaster = function() {
    var req = http.get({host: 'localhost', path: '/', port: 4040}, cb);
    req.on('error', function(e) { console.log('problem with request: ' + e.message); });
    req.end();
  };

  / / Set up regular updates
  updateMaster();
  setInterval(updateMaster, 5000);

  // set up the web server
  var www = http.createServer(handleRequest);
  www.listen(8080);
  ```
  Of course, you can use this sidecar from any language that you choose that supports HTTP and JSON.
-->

例如，这里有一个简单的 Node.js 应用程序，它连接到 leader election sidecar 并打印
出它当前是否是 master。默认情况下，leader election sidecar 将其标识符设置为
`hostname`。

```
var http = require('http');
// This will hold info about the current master
var master = {};

  // The web handler for our nodejs application
  var handleRequest = function(request, response) {
    response.writeHead(200);
    response.end("Master is " + master.name);
  };

  // A callback that is used for our outgoing client requests to the sidecar
  var cb = function(response) {
    var data = '';
    response.on('data', function(piece) { data = data + piece; });
    response.on('end', function() { master = JSON.parse(data); });
  };

  // Make an async request to the sidecar at http://localhost:4040
  var updateMaster = function() {
    var req = http.get({host: 'localhost', path: '/', port: 4040}, cb);
    req.on('error', function(e) { console.log('problem with request: ' + e.message); });
    req.end();
  };

  / / Set up regular updates
  updateMaster();
  setInterval(updateMaster, 5000);

  // set up the web server
  var www = http.createServer(handleRequest);
  www.listen(8080);
```

当然，您可以从任何支持 HTTP 和 JSON 的语言中使用这个 sidecar。

<!--
#### Conclusion


  Hopefully I've shown you how easy it is to build leader election for your distributed application using Kubernetes. In future installments we'll show you how Kubernetes is making building distributed systems even easier. In the meantime, head over to [Google Container Engine][2] or [kubernetes.io][3] to get started with Kubernetes.

  [1]: https://github.com/kubernetes/contrib/pull/353
  [2]: https://cloud.google.com/container-engine/
  [3]: http://kubernetes.io/
-->

#### 结论

希望我已经向您展示了使用 Kubernetes 为您的分布式应用程序构建 leader election 是
多么容易。在以后的部分中，我们将向您展示 Kubernetes 如何使构建分布式系统变得更�
容易。同时，转到[Google Container Engine][2]或[kubernetes.io][3]开始使用
Kubernetes。

[1]: https://github.com/kubernetes/contrib/pull/353
[2]: https://cloud.google.com/container-engine/
[3]: http://kubernetes.io/
