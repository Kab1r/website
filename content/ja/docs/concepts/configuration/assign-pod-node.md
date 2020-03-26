---
title: Node上へのPodのスケジューリング
content_template: templates/concept
weight: 30
---

{{% capture overview %}}

[Pod](/docs/concepts/workloads/pods/pod/)が稼働す
る[Node](/docs/concepts/architecture/nodes/)を特定のものに指定したり、優先条件を
指定して制限することができます。これを実現するためにはいくつかの方法がありますが
、推奨されている方法
は[ラベルでの選択](/docs/concepts/overview/working-with-objects/labels/)です。ス
ケジューラーが最適な配置を選択するため、一般的にはこのような制限は不要です(例え
ば、複数の Pod を別々の Node へデプロイしたり、Pod を配置する際にリソースが不十
分な Node にはデプロイされないことが挙げられます)が、 SSD が搭載されている Node
に Pod をデプロイしたり、同じアベイラビリティーゾーン内で通信する異なるサービス
の Pod を同じ Node にデプロイする等、柔軟な制御が必要なこともあります。

{{% /capture %}}

{{% capture body %}}

## nodeSelector

`nodeSelector`は、Node を選択するための、最も簡単で推奨されている手法です。
`nodeSelector`は PodSpec のフィールドです。これは key-value ペアのマップを特定し
ます。あるノードで Pod を稼働させるためには、そのノードがラベルとして指定された
key-value ペアを保持している必要があります(複数のラベルを保持することも可能です
)。最も一般的な使用方法は、1 つの key-value ペアを付与する方法です。

以下に、`nodeSelector`の使用例を紹介します。

### ステップ 0: 前提条件

この例では、Kubernetes の Pod に関して基本的な知識を有していることと
、[Kubernetes クラスターのセットアップ](https://github.com/kubernetes/kubernetes#documentation)が
されていることが前提となっています。

### ステップ 1: Node へのラベルの付与

`kubectl get nodes`で、クラスターのノードの名前を取得してください。そして、ラベ
ルを付与する Node を選び
、`kubectl label nodes <node-name> <label-key>=<label-value>`で選択した Node に
ラベルを付与します。例えば、Node の名前が
'kubernetes-foo-node-1.c.a-robinson.internal'、付与するラベルが'disktype=ssd'の
場合
、`kubectl label nodes kubernetes-foo-node-1.c.a-robinson.internal disktype=ssd`に
よってラベルが付与されます。

`kubectl get nodes --show-labels`によって、ノードにラベルが付与されたかを確認す
ることができます。また、`kubectl describe node "nodename"`から、その Node の全て
のラベルを表示することもできます。

### ステップ 2: Pod への nodeSelector フィールドの追�

該当の Pod の config ファイルに、nodeSelector のセクションを追加します: 例として
以下の config ファイルを扱います:

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

nodeSelector を以下のように追加します:

{{< codenew file="pods/pod-nginx.yaml" >}}

`kubectl apply -f https://k8s.io/examples/pods/pod-nginx.yaml`により、Pod は先ほ
どラベルを付与した Node へスケジュールされます。 `kubectl get pods -o wide`で表
示される"NODE"の列から、Pod がデプロイされている Node を確認することができます。

## 補足: ビルトイン Node ラベル

明示的に[付与](#step-one-attach-label-to-the-node)するラベルの他に、事前に Node
へ付与されているものもあります。以下のようなラベルが該当します。

- `kubernetes.io/hostname`
- `failure-domain.beta.kubernetes.io/zone`
- `failure-domain.beta.kubernetes.io/region`
- `beta.kubernetes.io/instance-type`
- `kubernetes.io/os`
- `kubernetes.io/arch`

{{< note >}} これらのラベルは、クラウドプロバイダー固有であり、確実なものではあ
りません。例えば、`kubernetes.io/hostname`の値は Node の名前と同じである環境もあ
れば、異なる環境もあります。 {{< /note >}}

## Node の隔離や制限

Node にラベルを付与することで、Pod は特定の Node や Node グループにスケジュール
されます。これにより、 特定の Pod を、確かな隔離性や安全性、特性を持った Node で
稼働させることができます。この目的でラベルを使用する際に、Node 上の kubelet プロ
セスに上書きされないラベルキーを選択することが強く推奨されています。これは、安全
性が損なわれた Node が kubelet の認証情報を Node のオブジェクトに設定したり、ス
ケジューラーがそのような Node にデプロイすることを防ぎます。

`NodeRestriction`プラグインは、kubelet が`node-restriction.kubernetes.io/`プレフ
ィックスを有するラベルの設定や上書きを防ぎます。 Node の隔離にラベルのプレフィッ
クスを使用するためには、以下の３点を確認してください。

1. NodeRestriction を使用するため、Kubernetes のバージョンが v1.11 以上であるこ
   と。
2. [Node authorizer](/docs/reference/access-authn-authz/node/)を使用していること
   と
   、[NodeRestriction admission plugin](/docs/reference/access-authn-authz/admission-controllers/#noderestriction)が
   有効になっていること。
3. Node に`node-restriction.kubernetes.io/` プレフィックスのラベルを付与し、その
   ラベルが node selector に指定されていること。例えば
   、`example.com.node-restriction.kubernetes.io/fips=true` または
   `example.com.node-restriction.kubernetes.io/pci-dss=true`のようなラベルです。

## Affinity と Anti-Affinity

`nodeSelector`は Pod の稼働を特定のラベルが付与された Node に制限する最も簡単な
方法です。 Affinity/Anti-Affinity では、より柔軟な指定方法が提供されています。拡
張機能は以下の通りです。

1. 様々な指定方法がある ("AND 条件"に限らない)
2. 必須条件ではなく優先条件を指定でき、条件を満たさない場合でも Pod をスケジュー
   ルさせることができる
3. Node 自体のラベルではなく、Node(または他のトポロジカルドメイン)上で稼働してい
   る他の Pod のラベルに対して条件を指定することができ、その Pod と同じ、または
   異なるドメインで稼働させることができる

Affinity は"Node Affinity"と"Inter-Pod Affinity/Anti-Affinity"の 2 種類から成り
ます。 Node affinity は`nodeSelector`(前述の 2 つのメリットがあります)に似ていま
すが、Inter-Pod Affinity/Anti-Affinity は、上記の 3 番目の機能に記載している通り
、Node のラベルではなく Pod のラベルに対して制限をかけます。

`nodeSelector`は問題なく使用することができますが、Node affinity
は`nodeSelector`で指定できる条件を全て実現できるため、将来的には推奨されなくなり
ます。

### Node Affinity

Node Affinity は α 機能として Kubernetes の v1.2 から導入されました。 Node
Affinity は概念的には、Node のラベルによって Pod がどの Node にスケジュールされ
るかを制限する`nodeSelector`と同様です。

現在は 2 種類の Node Affinity があり
、`requiredDuringSchedulingIgnoredDuringExecution`と`preferredDuringSchedulingIgnoredDuringExecution`で
す。前者は Node にスケジュールされる Pod が条件を満たすことが必須
(`nodeSelector`に似ていますが、より柔軟に条件を指定できます)であり、後者は条件を
指定できますが保証されるわけではなく、優先的に考慮されます。
"IgnoredDuringExecution"の意味するところは、`nodeSelector`の機能と同様であり
、Node のラベルが変更され、Pod がその条件を満たさなくなった場合でも Pod はその
Node で稼働し続けるということです。将来的には
、`requiredDuringSchedulingIgnoredDuringExecution`に、Pod の Node Affinity に記
された必須要件を満たさなくなった Node からその Pod を退避させることができる機能
を備えた`requiredDuringSchedulingRequiredDuringExecution`が提供される予定です。

それぞれの使用例として、 `requiredDuringSchedulingIgnoredDuringExecution` は、"
インテル CPU を供えた Node 上で Pod を稼働させる"、
`preferredDuringSchedulingIgnoredDuringExecution`は、"ゾーン XYZ で Pod の稼働を
試みますが、実現不可能な場合には他の場所で稼働させる" といった方法が挙げられます
。

Node Affinity は、PodSpec の`affinity`フィールドにある`nodeAffinity`フィールドで
特定します。

Node Affinity を使用した Pod の例を以下に示します:

{{< codenew file="pods/pod-with-node-affinity.yaml" >}}

この Node Affinity では、Pod はキーが`kubernetes.io/e2e-az-name`、値
が`e2e-az1`または`e2e-az2`のラベルが付与された Node にしか配置されません。加えて
、キーが`another-node-label-key`、値が`another-node-label-value`のラベルが付与さ
れた Node が優先されます。

この例ではオペレーター`In`が使われています。 Node Affinity では
、`In`、`NotIn`、`Exists`、`DoesNotExist`、`Gt`、`Lt`のオペレーターが使用できま
す。 `NotIn`と`DoesNotExist`は Node Anti-Affinity、または Pod を特定の Node にス
ケジュールさせない場合に使われ
る[Taints](/docs/concepts/configuration/taint-and-toleration/)に使用します。

`nodeSelector`と`nodeAffinity`の両方を指定した場合、Pod は**両方の**条件を満たす
Node にスケジュールされます。

`nodeAffinity`内で複数の`nodeSelectorTerms`を指定した場合、Pod は**いずれか
の**`nodeSelectorTerms`を満たした Node へスケジュールされます。

`nodeSelectorTerms`内で複数の`matchExpressions`を指定した場合には Pod は**全て
の**`matchExpressions`を満たした Node へスケジュールされます。

Pod がスケジュールされた Node のラベルを削除したり変更しても、Pod は削除されませ
ん。言い換えると、Affinity は Pod をスケジュールする際にのみ考慮されます。

`preferredDuringSchedulingIgnoredDuringExecution`内の`weight`フィールドは、1 か
ら 100 の範囲で指定します。全ての必要条件(リソースや RequiredDuringScheduling
Affinity 等)を満たした Node に対して、スケジューラーはその Node が
MatchExpressions を満たした場合に、このフィルードの"weight"を加算して合計を計算
します。このスコアが Node の他の優先機能のスコアと組み合わせれ、最も高いスコアを
有した Node が優先されます。

### Inter-Pod Affinity/Anti-Affinity

Inter-Pod Affinity と Anti-Affinity は、Node のラベルではなく、すでに Node で稼
働している Pod のラベルに従って Pod がスケジュールされる Node を制限します。この
ポリシーは、"X にてルール Y を満たす Pod がすでに稼働している場合、この Pod も X
で稼働させる(Anti-Affinity の場合は稼働させない)"という形式です。 Y は namespace
のリストで指定した LabelSelector で表されます。 Node と異なり、Pod は namespace
で区切られているため(それゆえ Pod のラベルも暗黙的に namespace で区切られます
)、Pod のラベルを指定する label selector は、どの namespace に selector を適用す
るかを指定する必要があります。概念的に、X は Node や、ラック、クラウドプロバイダ
ゾーン、クラウドプロバイダのリージョン等を表すトポロジードメインです。これらを表
すためにシステムが使用する Node Label のキーである`topologyKey`を使うことで、ト
ポロジードメインを指定することができます。先述のセクショ
ン[補足: ビルトイン Node ラベル](#interlude-built-in-node-labels)にてラベルの例
が紹介されています。

{{< note >}} Inter-Pod Affinity と Anti-Affinity は、大規模なクラスター上で使用
する際にスケジューリングを非常に遅くする恐れのある多くの処理を要します。そのため
、数百台以上の Node から成るクラスターでは使用することを推奨されません。
{{< /note >}}

{{< note >}} Pod Anti-Affinity は、Node に必ずラベルが付与されている必要がありま
す。例えば、クラスターの全ての Node が、`topologyKey`で指定されたものに合致する
適切なラベルが必要になります。それらが付与されていない Node が存在する場合、意図
しない挙動を示すことがあります。 {{< /note >}}

Node Affinity と同様に、Pod Affinity と Pod Anti-Affinity にも必須条件と優先条件
を示
す`requiredDuringSchedulingIgnoredDuringExecution`と`preferredDuringSchedulingIgnoredDuringExecution`が
あります。前述の Node Affinity のセクションを参照してください。
`requiredDuringSchedulingIgnoredDuringExecution`を指定する Affinity の使用例は
、"Service A の Pod と Service B の Pod が密に通信する際、それらを同じゾーンで稼
働させる場合"です。また、`preferredDuringSchedulingIgnoredDuringExecution`を指定
する Anti-Affinity の使用例は、"ゾーンをまたいで Pod のサービスを稼働させる場合
"(Pod の数はゾーンの数よりも多いため、必須条件を指定すると合理的ではありません)
です。

Inter-Pod Affinity は、PodSpec の`affinity`フィールド内に`podAffinity`で指定し
、Inter-Pod Anti-Affinity は、`podAntiAffinity`で指定します。

#### Pod Affinity を使用した Pod の例

{{< codenew file="pods/pod-with-pod-affinity.yaml" >}}

この Pod の Affifnity は、Pod Affinity と Pod Anti-Affinity を 1 つずつ定義して
います。この例では
、`podAffinity`に`requiredDuringSchedulingIgnoredDuringExecution`、`podAntiAffinity`に`preferredDuringSchedulingIgnoredDuringExecution`が
設定されています。 Pod Affinity は、「キーが"security"、値が"S1"のラベルが付与さ
れた Pod が少なくとも 1 つは稼働している Node が同じゾーンにあれば、Pod はその
Node にスケジュールされる」という条件を指定しています(より正確には、キーが
"security"、値が"S1"のラベルが付与された Pod が稼働しており、キー
が`failure-domain.beta.kubernetes.io/zone`、値が V である Node が少なくとも 1 つ
はある状態で、 Node N がキー`failure-domain.beta.kubernetes.io/zone`、値 V のラ
ベルを持つ場合に、Pod は Node N で稼働させることができます)。 Pod Anti-Affinity
は、「すでにある Node 上で、キーが"security"、値が"S2"である Pod が稼働している
場合に、Pod を可能な限りその Node 上で稼働させない」という条件を指定しています
(`topologyKey`が`failure-domain.beta.kubernetes.io/zone`であった場合、キーが
"security"、値が"S2"であるである Pod が稼働しているゾーンと同じゾーン内の Node
にはスケジュールされなくなります)。 Pod Affinity と Pod Anti-Affinity や
、`requiredDuringSchedulingIgnoredDuringExecution`と`preferredDuringSchedulingIgnoredDuringExecution`に
関する他の使用例
は[デザインドック](https://git.k8s.io/community/contributors/design-proposals/scheduling/podaffinity.md)を
参照してください。

Pod Affinity と Pod Anti-Affinity で使用できるオペレーターは、`In`、`NotIn`、
`Exists`、 `DoesNotExist`です。

原則として、`topologyKey`には任意のラベルとキーが使用できます。しかし、パフォー
マンスやセキュリティの観点から、以下の制約があります:

1. Affinity と、`requiredDuringSchedulingIgnoredDuringExecution`を指定した Pod
   Anti-Affinity では、`topologyKey`を指定しないことは許可されていません。
2. `requiredDuringSchedulingIgnoredDuringExecution`を指定した Pod Anti-Affinity
   では、`kubernetes.io/hostname`の`topologyKey`を制限するため、アドミッションコ
   ントローラー`LimitPodHardAntiAffinityTopology`が導入されました。トポロジーを
   カスタマイズする場合には、アドミッションコントローラーを修正または無効化する
   必要があります。
3. `preferredDuringSchedulingIgnoredDuringExecution`を指定した Pod Anti-Affinity
   では、`topologyKey`を指定しなかった場合、"全てのトポロジー"と解釈されます("全
   てのトポロジー"とは、ここで
   は`kubernetes.io/hostname`、`failure-domain.beta.kubernetes.io/zone`、`failure-domain.beta.kubernetes.io/region`を
   合わせたものを意味します)。
4. 上記の場合を除き、`topologyKey` は任意のラベルとキーを指定することができあま
   す。

`labelSelector`と`topologyKey`に加え、`labelSelector`が合致すべき`namespaces`の
リストを特定することも可能です(これは`labelSelector`と`topologyKey`を定義するこ
とと同等です)。省略した場合や空の場合は、Affinity と Anti-Affinity が定義された
Pod の namespace がデフォルトで設定されます。

`requiredDuringSchedulingIgnoredDuringExecution`が指定された Affinity と
Anti-Affinity では、`matchExpressions`に記載された全ての条件が満たされる Node に
Pod がスケジュールされます。

#### 実際的なユースケース

Inter-Pod Affinity と Anti-Affinity は、ReplicaSet、StatefulSet、Deployment など
のより高レベルなコレクションと併せて使用すると更に有用です。 Workload が、Node
等の定義された同じトポロジーに共存させるよう、簡単に設定できます。

##### 常に同じ Node で稼働させる場合

３つのノードから成るクラスターでは、ウェブアプリケーションは redis のようにイン
メモリキャッシュを保持しています。このような場合、ウェブサーバーは可能な限りキャ
ッシュと共存させることが望ましいです。

ラベル`app=store`を付与した 3 つのレプリカから成る redis の deployment を記述し
た yaml ファイルを示します。 Deployment には、1 つの Node にレプリカを共存させな
いために`PodAntiAffinity`を付与しています。

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

ウェブサーバーの Deployment を記載した以下の yaml ファイルには
、`podAntiAffinity` と`podAffinity`が設定されています。全てのレプリカ
が`app=store`のラベルが付与された Pod と同じゾーンで稼働するよう、スケジューラー
に設定されます。また、それぞれのウェブサーバーは 1 つのノードで稼働されないこと
も保証されます。

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

上記 2 つの Deployment が生成されると、3 つのノードは以下のようになります。

|    node-1     |    node-2     |    node-3     |
| :-----------: | :-----------: | :-----------: |
| _webserver-1_ | _webserver-2_ | _webserver-3_ |
|   _cache-1_   |   _cache-2_   |   _cache-3_   |

このように、3 つの`web-server`は期待通り自動的にキャッシュと共存しています。

```
kubectl get pods -o wide
```

出力は以下のようになります:

```
NAME                           READY     STATUS    RESTARTS   AGE       IP           NODE
redis-cache-1450370735-6dzlj   1/1       Running   0          8m        10.192.4.2   kube-node-3
redis-cache-1450370735-j2j96   1/1       Running   0          8m        10.192.2.2   kube-node-1
redis-cache-1450370735-z73mh   1/1       Running   0          8m        10.192.3.1   kube-node-2
web-server-1287567482-5d4dz    1/1       Running   0          7m        10.192.2.3   kube-node-1
web-server-1287567482-6f7v5    1/1       Running   0          7m        10.192.4.3   kube-node-3
web-server-1287567482-s330j    1/1       Running   0          7m        10.192.3.2   kube-node-2
```

##### 同じ Node に共存させない場合

上記の例では `PodAntiAffinity`を`topologyKey: "kubernetes.io/hostname"`と合わせ
て指定することで、redis クラスター内の 2 つのインスタンスが同じホストにデプロイ
されない場合を扱いました。同様の方法で、Anti-Affinity を用いて高可用性を実現した
StatefulSet の使用例
は[ZooKeeper tutorial](/docs/tutorials/stateful-application/zookeeper/#tolerating-node-failure)を
参照してください。

## nodeName

`nodeName`は Node の選択を制限する最も簡単な方法ですが、制約があることからあまり
使用されません。 `nodeName`は PodSpec のフィールドです。ここに値が設定されると
、scheduler はその Pod を考慮しなくなり、その名前が付与されている Node の
kubelet は Pod を稼働させようとします。そのため、PodSpec に`nodeName`が指定され
ると、上述の Node の選択方法よりも優先されます。

`nodeName`を使用することによる制約は以下の通りです:

- その名前の Node が存在しない場合、Pod は起動されす、自動的に削除される場合があ
  ります。
- その名前の Node に Pod を稼働させるためのリソースがない場合、Pod の起動は失敗
  し、理由は OutOfmemory や OutOfcpu になります。
- クラウド上の Node の名前は予期できず、変更される可能性があります。

`nodeName`を指定した Pod の設定ファイルの例を示します:

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

上記の Pod は kube-01 という名前の Node で稼働します。

{{% /capture %}}

{{% capture whatsnext %}}

[Taints](/docs/concepts/configuration/taint-and-toleration/)を使うことで、Node
は Pod を追い出すことができます。

[Node Affinity](https://git.k8s.io/community/contributors/design-proposals/scheduling/nodeaffinity.md)と
[Inter-Pod Affinity/Anti-Affinity](https://git.k8s.io/community/contributors/design-proposals/scheduling/podaffinity.md)
には、Taints の要点に関して様々な背景が紹介されています。

{{% /capture %}}
