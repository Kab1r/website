---
title: Serviceを使用してフロントエンドをバックエンドに接続する
content_template: templates/tutorial
weight: 70
---

{{% capture overview %}}

このタスクでは、フロントエンドとバックエンドのマイクロサービスを作成する方法を示
します。バックエンドのマイクロサービスは挨拶です。フロントエンドとバックエンドは
、Kubernetes {{< glossary_tooltip term_id="service" >}}オブジェクトを使用して接
続されます。

{{% /capture %}}

{{% capture objectives %}}

- {{< glossary_tooltip term_id="deployment" >}}オブジェクトを使用してマイクロサ
  ービスを作成および実行します。
- フロントエンドを経由してトラフィックをバックエンドにルーティングします。
- Service オブジェクトを使用して、フロントエンドアプリケーションをバックエンドア
  プリケーションに接続します。

{{% /capture %}}

{{% capture prerequisites %}}

- {{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

- このタスクで
  は[Service で外部ロードバランサー](/docs/tasks/access-application-cluster/create-external-load-balancer/)を
  使用しますが、外部ロードバランサーの使用がサポートされている環境である必要があ
  ります。ご使用の環境がこれをサポートしていない場合は、代わりにタイ
  プ[NodePort](/ja/docs/concepts/services-networking/service/#nodeport)の
  Service を使用できます。

{{% /capture %}}

{{% capture lessoncontent %}}

### Deployment を使用したバックエンドの作成

バックエンドは、単純な挨拶マイクロサービスです。バックエンドの Deployment の構成
ファイルは次のとおりです:

{{< codenew file="service/access/hello.yaml" >}}

バックエンドの Deployment を作成します:

```shell
kubectl apply -f https://k8s.io/examples/service/access/hello.yaml
```

バックエンドの Deployment に関する情報を表示します:

```shell
kubectl describe deployment hello
```

出力はこのようになります:

```
Name:                           hello
Namespace:                      default
CreationTimestamp:              Mon, 24 Oct 2016 14:21:02 -0700
Labels:                         app=hello
                                tier=backend
                                track=stable
Annotations:                    deployment.kubernetes.io/revision=1
Selector:                       app=hello,tier=backend,track=stable
Replicas:                       7 desired | 7 updated | 7 total | 7 available | 0 unavailable
StrategyType:                   RollingUpdate
MinReadySeconds:                0
RollingUpdateStrategy:          1 max unavailable, 1 max surge
Pod Template:
  Labels:       app=hello
                tier=backend
                track=stable
  Containers:
   hello:
    Image:              "gcr.io/google-samples/hello-go-gke:1.0"
    Port:               80/TCP
    Environment:        <none>
    Mounts:             <none>
  Volumes:              <none>
Conditions:
  Type          Status  Reason
  ----          ------  ------
  Available     True    MinimumReplicasAvailable
  Progressing   True    NewReplicaSetAvailable
OldReplicaSets:                 <none>
NewReplicaSet:                  hello-3621623197 (7/7 replicas created)
Events:
...
```

### バックエンド Service オブジェクトの作成

フロントエンドをバックエンドに接続する鍵は、バックエンド Service です。 Service
は、バックエンドマイクロサービスに常に到達できるように、永続的な IP アドレスと
DNS 名のエントリを作成します。 Service
は{{< glossary_tooltip text="セレクター" term_id="selector" >}}を使用して、トラ
フィックをルーティングする Pod を見つけます。

まず、Service 構成ファイルを調べます:

{{< codenew file="service/access/hello-service.yaml" >}}

設定ファイルで、Service が`app：hello`および`tier：backend`というラベルを持つ
Pod にトラフィックをルーティングしていることがわかります。

`hello` Service を作成します:

```shell
kubectl apply -f https://k8s.io/examples/service/access/hello-service.yaml
```

この時点で、バックエンドの Deployment が実行され、そちらにトラフィックをルーティ
ングできる Service があります。

### フロントエンドの作成

バックエンドができたので、バックエンドに接続するフロントエンドを作成できます。フ
ロントエンドは、バックエンド Service に指定された DNS 名を使用して、バックエンド
ワーカー Pod に接続します。 DNS 名は`hello`です。これは、前のサービス設定ファイ
ルの`name`フィールドの値です。

フロントエンド Deployment の Pod は、hello バックエンド Service を見つけるように
構成された nginx イメージを実行します。これは nginx 設定ファイルです:

{{< codenew file="service/access/frontend.conf" >}}

バックエンドと同様に、フロントエンドには Deployment と Service があります。
Service の設定には`type：LoadBalancer`があります。これは、Service がクラウドプロ
バイダーのデフォルトのロードバランサーを使用することを意味します。

{{< codenew file="service/access/frontend.yaml" >}}

フロントエンドの Deployment と Service を作成します:

```shell
kubectl apply -f https://k8s.io/examples/service/access/frontend.yaml
```

出力結果から両方のリソースが作成されたことを確認します:

```
deployment.apps/frontend created
service/frontend created
```

{{< note >}} nginx の構成は
、[コンテナイメージ](/examples/service/access/Dockerfile)に焼き付けられます。こ
れを行うためのより良い方法は
、[ConfigMap](/docs/tasks/configure-pod-container/configure-pod-configmap/)を使
用して、構成をより簡単に変更できるようにすることです。 {{< /note >}}

### フロントエンド Service と対話

LoadBalancer タイプの Service を作成したら、このコマンドを使用して外部 IP を見つ
けることができます:

```shell
kubectl get service frontend --watch
```

これにより`frontend` Service の設定が表示され、変更が監視されます。最初、外部 IP
は`<pending>`としてリストされます:

```
NAME       TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)  AGE
frontend   LoadBalancer   10.51.252.116   <pending>     80/TCP   10s
```

ただし、外部 IP がプロビジョニングされるとすぐに、`EXTERNAL-IP`という見出しの下
に新しい IP が含まれるように構成が更新されます:

```
NAME       TYPE           CLUSTER-IP      EXTERNAL-IP        PORT(S)  AGE
frontend   LoadBalancer   10.51.252.116   XXX.XXX.XXX.XXX    80/TCP   1m
```

この IP を使用して、クラスターの外部から`frontend` Service とやり取りできるよう
になりました。

### フロントエンドを介するトラフィック送信

フロントエンドとバックエンドが接続されました。フロントエンド Service の外部 IP
に対して curl コマンドを使用して、エンドポイントにアクセスできます。

```shell
curl http://${EXTERNAL_IP} # これを前に見たEXTERNAL-IPに置き換えます
```

出力には、バックエンドによって生成されたメッセージが表示されます:

```json
{ "message": "Hello" }
```

{{% /capture %}}

{{% capture whatsnext %}}

- [Service](/ja/docs/concepts/services-networking/service/)の詳細
- [ConfigMap](/docs/tasks/configure-pod-container/configure-pod-configmap/)の詳
  細

{{% /capture %}}
