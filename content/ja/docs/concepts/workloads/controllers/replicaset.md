---
reviewers:
title: ReplicaSet
content_template: templates/concept
weight: 10
---

{{% capture overview %}}

ReplicaSet の目的は、どのような時でも安定したレプリカ Pod のセットを維持すること
です。これは、理想的なレプリカ数の Pod が利用可能であることを保証するものとして
使用されます。

{{% /capture %}}

{{% capture body %}}

## ReplicaSet がどのように動くか

ReplicaSet は、ReplicaSet が対象とする Pod をどう特定するかを示すためのセレクタ
ーや、稼働させたい Pod のレプリカ数、Pod テンプレート(理想のレプリカ数の条件を満
たすために作成される新しい Pod のデータを指定するために用意されるもの)といったフ
ィールドとともに定義されます。ReplicaSet は、指定された理想のレプリカ数にするた
めに Pod の作成と削除を行うことにより、その目的を達成します。ReplicaSet が新しい
Pod を作成するとき、ReplicaSet はその Pod テンプレートを使用します。

ReplicaSet がその Pod 群と連携するためのリンクは、Pod
の[metadata.ownerReferences](/docs/concepts/workloads/controllers/garbage-collection/#owners-and-dependents)と
いうフィールド(現在のオブジェクトが所有されているリソースを指定する)を介して作成
されます。ReplicaSet によって所持された全ての Pod は、それら
の`ownerReferences`フィールドに ReplicaSet を特定する情報を保持します。このリン
クを通じて、ReplicaSet は管理している Pod の状態を把握したり、その後の実行計画を
立てます。

ReplicaSet は、そのセレクターを使用することにより、所有するための新しい Pod を特
定します。もし`ownerReference`フィールドの値を持たない Pod か
、`ownerReference`フィールドの値がコントローラーでない Pod で、その Pod が
ReplicaSet のセレクターとマッチした場合に、その Pod は即座にその ReplicaSet によ
って所有されます。

## ReplicaSet を使うとき

ReplicaSet はどんな時でも指定された数の Pod のレプリカが稼働することを保証します
。しかし、Deployment は ReplicaSet を管理する、より上位レベルの概念で
、Deployment はその他の多くの有益な機能と共に、宣言的な Pod のアップデート機能を
提供します。それゆえ、我々はユーザーが独自のアップデートオーケストレーションを必
要としたり、アップデートを全く必要としないような場合を除いて、ReplicaSet を直接
使うよりも代わりに Deployment を使うことを推奨します。

これは、ユーザーが ReplicaSet のオブジェクトを操作する必要が全く無いことを意味し
ます。代わりに Deployment を使用して、`spec`セクションにユーザーのアプリケーショ
ンを定義してください。

## ReplicaSet の使用例

{{< codenew file="controllers/frontend.yaml" >}}

上記のマニフェストを`frontend.yaml`ファイルに保存し Kubernetes クラスターに適用
すると、マニフェストに定義された ReplicaSet とそれが管理する Pod 群を作成します
。

```shell
kubectl apply -f http://k8s.io/examples/controllers/frontend.yaml
```

ユーザーはデプロイされた現在の ReplicaSet の情報も取得できます。

```shell
kubectl get rs
```

そして、ユーザーが作成した frontend リソースについての情報も取得できます。

```shell
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         3       6s
```

ユーザーはまた ReplicaSet の状態も確認できます。

```shell
kubectl describe rs/frontend
```

その結果は以下のようになります。

```shell
Name:		frontend
Namespace:	default
Selector:	tier=frontend,tier in (frontend)
Labels:		app=guestbook
		tier=frontend
Annotations:	<none>
Replicas:	3 current / 3 desired
Pods Status:	3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:       app=guestbook
                tier=frontend
  Containers:
   php-redis:
    Image:      gcr.io/google_samples/gb-frontend:v3
    Port:       80/TCP
    Requests:
      cpu:      100m
      memory:   100Mi
    Environment:
      GET_HOSTS_FROM:   dns
    Mounts:             <none>
  Volumes:              <none>
Events:
  FirstSeen    LastSeen    Count    From                SubobjectPath    Type        Reason            Message
  ---------    --------    -----    ----                -------------    --------    ------            -------
  1m           1m          1        {replicaset-controller }             Normal      SuccessfulCreate  Created pod: frontend-qhloh
  1m           1m          1        {replicaset-controller }             Normal      SuccessfulCreate  Created pod: frontend-dnjpy
  1m           1m          1        {replicaset-controller }             Normal      SuccessfulCreate  Created pod: frontend-9si5l
```

そして最後に、ユーザーは ReplicaSet によって作成された Pod もチェックできます。

```shell
kubectl get Pods
```

表示される Pod に関する情報は以下のようになります。

```shell
NAME             READY     STATUS    RESTARTS   AGE
frontend-9si5l   1/1       Running   0          1m
frontend-dnjpy   1/1       Running   0          1m
frontend-qhloh   1/1       Running   0          1m
```

ユーザーはまた、それらの Pod の`ownerReferences`が`frontend`ReplicaSet に設定さ
れていることも確認できます。これを確認するためには、稼働している Pod の中のどれ
かの yaml ファイルを取得します。

```shell
kubectl get pods frontend-9si5l -o yaml
```

その表示結果は、以下のようになります。その`frontend`ReplicaSet の情報
が`metadata`の`ownerReferences`フィールドにセットされています。

```shell
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: 2019-01-31T17:20:41Z
  generateName: frontend-
  labels:
    tier: frontend
  name: frontend-9si5l
  namespace: default
  ownerReferences:
  - apiVersion: extensions/v1beta1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: frontend
    uid: 892a2330-257c-11e9-aecd-025000000001
...
```

## テンプレートなしの Pod の所有

ユーザーが問題なくベア Pod(Bare Pod: ここでは Pod テンプレート無しの Pod のこと)
を作成しているとき、そのベア Pod がユーザーの ReplicaSet の中のいずれのセレクタ
ーともマッチしないことを確認することを強く推奨します。この理由として、ReplicaSet
は、所有対象の Pod が ReplicaSet のテンプレートによって指定された Pod のみに限定
されていないからです(ReplicaSet は前のセクションで説明した方法によって他の Pod
も所有できます)。

前のセクションで取り上げた`frontend`ReplicaSet と、下記のマニフェストの Pod をみ
てみます。

{{< codenew file="pods/pod-rs.yaml" >}}

これらの Pod は`ownerReferences`に何のコントローラー(もしくはオブジェクト)も指定
されておらず、そして`frontend`ReplicaSet にマッチするセレクターをもっており、こ
れらの Pod は即座に`frontend`ReplicaSet によって所有されます。

この`frontend`ReplicaSet がデプロイされ、初期の Pod レプリカがレプリカ数の要求を
満たすためにセットアップされた後で、ユーザーがその Pod を作成することを考えます
。

```shell
kubectl apply -f http://k8s.io/examples/pods/pod-rs.yaml
```

新しい Pod はその ReplicaSet によって所有され、その ReplicaSet のレプリカ数が、
設定された理想のレプリカ数を超えた場合すぐにそれらの Pod は削除されます。

下記のコマンドで Pod を取得できます。

```shell
kubectl get Pods
```

その表示結果で、新しい Pod がすでに削除済みか、削除中のステータスになっているの
を確認できます。

```shell
NAME             READY   STATUS        RESTARTS   AGE
frontend-9si5l   1/1     Running       0          1m
frontend-dnjpy   1/1     Running       0          1m
frontend-qhloh   1/1     Running       0          1m
pod2             0/1     Terminating   0          4s
```

もしユーザーがその Pod を最初に作成する場合

```shell
kubectl apply -f http://k8s.io/examples/pods/pod-rs.yaml
```

そしてその後に`frontend`ReplicaSet を作成すると、

```shell
kubectl apply -f http://k8s.io/examples/controllers/frontend.yaml
```

ユーザーはその ReplicaSet が作成した Pod を所有し、さらにもともと存在していた
Pod と今回新たに作成された Pod の数が、理想のレプリカ数になるまで Pod を作成する
のを確認できます。ここでまた Pod の状態を取得します。

```shell
kubectl get Pods
```

取得結果は下記のようになります。

```shell
NAME             READY   STATUS    RESTARTS   AGE
frontend-pxj4r   1/1     Running   0          5s
pod1             1/1     Running   0          13s
pod2             1/1     Running   0          13s
```

この方法で、ReplicaSet はテンプレートで指定されたもの以外の Pod を所有することが
できます。

## ReplicaSet のマニフェストを記述する。

他の全ての Kubernetes API オブジェクトのように、ReplicaSet
は`apiVersion`、`kind`と`metadata`フィールドを必要とします。 ReplicaSet では
、`kind`フィールドの値は`ReplicaSet`です。 Kubernetes1.9 において、ReplicaSet
は`apps/v1`という API バージョンが現在のバージョンで、デフォルトで有効です
。`apps/v1beta2`という API バージョンは廃止されています。先ほど作成し
た`frontend.yaml`ファイルの最初の行を参考にしてください。

また、ReplicaSet
は[`.spec` セクション](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status)も
必須です。

### Pod テンプレート

`.spec.template`はラベルを持つことが必要
な[Pod テンプレート](/ja/docs/concepts/workloads/pods/pod-overview/#podテンプレート)
です。先ほど作成した`frontend.yaml`の例では、`tier: frontend`というラベルを 1 つ
持っています。他のコントローラーがこの Pod を所有しようとしないためにも、他のコ
ントローラーのセレクターでラベルを上書きしないように注意してください。

テンプレート
の[再起動ポリシー](/docs/concepts/workloads/Pods/pod-lifecycle/#restart-policy)の
ためのフィールドである`.spec.template.spec.restartPolicy`は`Always`のみ許可され
ていて、そしてそれがデフォルト値です。

### Pod セレクター

`.spec.selector`フィールド
は[ラベルセレクター](/docs/concepts/overview/working-with-objects/labels/)です。
[先ほど](#how-a-replicaset-works)議論したように、ReplicaSet が所有する Pod を指
定するためにそのラベルが使用されます。先ほどの`frontend.yaml`の例では、そのセレ
クターは下記のようになっていました

```shell
matchLabels:
	tier: frontend
```

その ReplicaSet において、`.spec.template.metadata.labels`フィールドの値
は`spec.selector`と一致しなくてはならず、一致しない場合は API によって拒否されま
す。

{{< note >}} 2 つの ReplicaSet が同じ`.spec.selector`の値を設定しているが、それ
ぞれ異なる`.spec.template.metadata.labels`と`.spec.template.spec`フィールドの値
を持っていたとき、それぞれの ReplicaSet はもう一方の ReplicaSet によって作成され
た Pod を無視します。 {{< /note >}}

### レプリカ数について

ユーザーは`.spec.replicas`フィールドの値を設定することにより、いくつの Pod を同
時に稼働させるか指定できます。そのとき ReplicaSet はレプリカ数がこの値に達するま
で Pod を作成、または削除します。

もしユーザーが`.spec.replicas`を指定しない場合、デフォルト値として 1 がセットさ
れます。

## ReplicaSet を利用する

### ReplicaSet と Pod の削除

ReplicaSet とそれが所有する全ての Pod 削除したいときは
、[`kubectl delete`](/docs/reference/generated/kubectl/kubectl-commands#delete)コ
マンドを使ってください。  
[ガーベージコレクター](/docs/concepts/workloads/controllers/garbage-collection/)が
デフォルトで自動的に全ての依存する Pod を削除します。

REST API もしくは`client-go`ライブラリーを使用するとき、ユーザーは`-d`オプション
で`propagationPolicy`を`Background`か`Foreground`と指定しなくてはなりません。例
えば下記のように実行します。

```shell
kubectl proxy --port=8080
curl -X DELETE  'localhost:8080/apis/extensions/v1beta1/namespaces/default/replicasets/frontend' \
> -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Foreground"}' \
> -H "Content-Type: application/json"
```

### ReplicaSet のみを削除する

ユーザー
は[`kubectl delete`](/docs/reference/generated/kubectl/kubectl-commands#delete)コ
マンドで`--cascade=false`オプションを付けることにより、所有する Pod に影響を与え
ることなく ReplicaSet を削除できます。 REST API もしくは`client-go`ライブラリー
を使用するとき、ユーザーは`-d`オプションで`propagationPolicy`を`Orphan`と指定し
なくてはなりません。

```shell
kubectl proxy --port=8080
curl -X DELETE  'localhost:8080/apis/extensions/v1beta1/namespaces/default/replicasets/frontend' \
> -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Orphan"}' \
> -H "Content-Type: application/json"
```

一度元の ReplicaSet が削除されると、ユーザーは新しいものに置き換えるため新しい
ReplicaSet を作ることができます。新旧の ReplicaSet の`.spec.selector`の値が同じ
である間、新しい ReplicaSet は古い ReplicaSet で稼働していた Pod を取り入れます
。しかし、存在する Pod が新しく異なる Pod テンプレートとマッチさせようとするとき
、この仕組みは機能しません。ユーザーのコントロール下において新しい spec の Pod
をアップデートしたい場合は、[ローリングアップデート](#rolling-updates)を使用して
ください。

### Pod を ReplicaSet から分離させる

ユーザーは Pod のラベルを変更することにより、ReplicaSet からその Pod を削除でき
ます。この手法はデバッグや、データ修復などのためにサービスから Pod を削除したい
ときに使用できます。この方法で削除された Pod は自動的に新しいものに置き換えられ
ます。(レプリカ数は変更されないものと仮定します。)

### ReplicaSet のスケーリング

ReplicaSet は、ただ`.spec.replicas`フィールドを更新することによって簡単にスケー
ルアップまたはスケールダウンできます。ReplicaSet コントローラーは、ラベルセレク
ターにマッチするような指定した数の Pod が利用可能であり、操作可能であることを保
証します。

### HorizontalPodAutoscaler(HPA)のターゲットとしての ReplicaSet

ReplicaSet はまた
、[Horizontal Pod Autoscalers (HPA)](/docs/tasks/run-application/horizontal-pod-autoscale/)の
ターゲットにもなることができます。これはつまり ReplicaSet が HPA によってオート
スケールされうることを意味します。ここでは HPA が、前の例で作成した ReplicaSet
をターゲットにする例を示します。

{{< codenew file="controllers/hpa-rs.yaml" >}}

このマニフェストを`hpa-rs.yaml`に保存し、Kubernetes クラスターに適用すると、レプ
リケートされた Pod の CPU 使用量にもとづいてターゲットの ReplicaSet をオートスケ
ールする HPA を作成します。

```shell
kubectl apply -f https://k8s.io/examples/controllers/hpa-rs.yaml
```

同様のことを行うための代替案として、`kubectl autoscale`コマンドも使用できます。(
こちらの方がより簡単です。)

```shell
kubectl autoscale rs frontend --max=10
```

## ReplicaSet の代替案

### Deployment (推奨)

[`Deployment`](/ja/docs/concepts/workloads/controllers/deployment/)は ReplicaSet
を所有することのできるオブジェクトで、宣言的なサーバサイドのローリングアップデー
トを介して ReplicaSet と Pod をアップデートできます。 ReplicaSet は単独で使用可
能ですが、現在では、ReplicaSet は主に Pod の作成、削除とアップデートを司るための
メカニズムとして Deployment によって使用されています。ユーザーが Deployment を使
用するとき、Deployment によって作成される ReplicaSet の管理について心配する必要
はありません。Deployment は ReplicaSet を所有し、管理します。このため、もしユー
ザーが ReplicaSet を必要とするとき、Deployment の使用を推奨します。

### ベア Pod(Bare Pods)

ユーザーが Pod を直接作成するケースとは異なり、ReplicaSet は Node の故障やカーネ
ルのアップグレードといった破壊的な Node のメンテナンスなど、どのような理由に限ら
ず削除または停止された Pod を置き換えます。このため、我々はもしユーザーのアプリ
ケーションが単一の Pod のみ必要とする場合でも ReplicaSet を使用することを推奨し
ます。プロセスのスーパーバイザーについても同様に考えると、それは単一 Node 上での
独立したプロセスの代わりに複数の Node にまたがった複数の Pod を監視します。
ReplicaSet は、Node 上のいくつかのエージェント(例えば、Kubelet や Docker）に対し
て、ローカルのコンテナ再起動を移譲します。

### Job

Pod を Pod それ自身で停止させたいような場合(例えば、バッチ用のジョブなど)は
、ReplicaSet の代わり
に[`Job`](/docs/concepts/jobs/run-to-completion-finite-workloads/)を使用してくだ
さい。

### DaemonSet

マシンの監視やロギングなど、マシンレベルの機能を提供したい場合は、ReplicaSet の
代わりに[`DaemonSet`](/ja/docs/concepts/workloads/controllers/daemonset/)を使用
してください。これらの Pod はマシン自体のライフタイムに紐づいています: その Pod
は他の Pod が起動する前に、そのマシン上で稼働される必要があり、マシンが再起動ま
たはシャットダウンされるときには、安全に停止されます。

### ReplicationController

ReplicaSet
は[_ReplicationControllers_](/docs/concepts/workloads/controllers/replicationcontroller/)の
後継となるものです。この 2 つは、ReplicationController
が[ラベルについてのユーザーガイド](/docs/concepts/overview/working-with-objects/labels/#label-selectors)に
書かれているように、集合ベース(set-based)のセレクター要求をサポートしていないこ
とを除いては、同じ目的を果たし、同じようにふるまいます。  
このように、ReplicaSet は ReplicationController よりも好まれます。

{{% /capture %}}
