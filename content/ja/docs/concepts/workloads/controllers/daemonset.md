---
reviewers:
title: DaemonSet
content_template: templates/concept
weight: 50
---

{{% capture overview %}}

_DaemonSet_ は全て(またはいくつか)の Node が単一の Pod のコピーを稼働させること
を保証します。Node がクラスターに追加されるとき、Pod が Node 上に追加されます
。Node がクラスターから削除されたとき、それらの Pod はガーベージコレクターにより
除去されます。DaemonSet の削除により、DaemonSet が作成した Pod もクリーンアップ
します。

DaemonSet のいくつかの典型的な使用例は以下の通りです。

- `glusterd`や`ceph`のようなクラスターのストレージデーモンを各 Node 上で稼働させ
  る。
- `fluentd`や`logstash`のようなログ集計デーモンを各 Node 上で稼働させる。
- [Prometheus Node Exporter](https://github.com/prometheus/node_exporter)や[Flowmill](https://github.com/Flowmill/flowmill-k8s/)、[Sysdig Agent](https://docs.sysdig.com)、`collectd`、[Dynatrace OneAgent](https://www.dynatrace.com/technologies/kubernetes-monitoring/)、
  [AppDynamics Agent](https://docs.appdynamics.com/display/CLOUD/Container+Visibility+with+Kubernetes)、
  [Datadog agent](https://docs.datadoghq.com/agent/kubernetes/daemonset_setup/)、
  [New Relic agent](https://docs.newrelic.com/docs/integrations/kubernetes-integration/installation/kubernetes-installation-configuration)、Ganglia
  の`gmond`や Instana agent などのような Node のモニタリングデーモンを各 Node 上
  で稼働させる。

シンプルなケースとして、各タイプのデーモンにおいて、全ての Node をカバーする 1
つの DaemonSet が使用されるケースがあります。さらに複雑な設定では、単一のタイプ
のデーモン用ですが、異なるフラグや、異なるハードウェアタイプに対するメモリー
、CPU リクエストを要求する複数の DaemonSet を使用するケースもあります。

{{% /capture %}}

{{% capture body %}}

## DaemonSet Spec の記述

### DaemonSet の作成

ユーザーは YAML ファイル内で DaemonSet の設定を記述することができます。
例えば、下記の`daemonset.yaml`ファイルでは`fluentd-elasticsearch`という Docker
イメージを稼働させる DaemonSet の設定を記述します。

{{< codenew file="controllers/daemonset.yaml" >}}

- YAML ファイルに基づいて DaemonSet を作成します。

```
kubectl apply -f https://k8s.io/examples/controllers/daemonset.yaml
```

### 必須のフィールド

他の全ての Kubernetes の設定と同様に、DaemonSet
は`apiVersion`、`kind`と`metadata`フィールドが必須となります。
設定ファイルの活用法に関する一般的な情報は
、[アプリケーションのデプロイ](/ja/docs/tasks/run-application/run-stateless-application-deployment/)、[コンテナの設定](/ja/docs/tasks/)、[kubectl を用いたオブジェクトの管理](/ja/docs/concepts/overview/working-with-objects/object-management/)と
いったドキュメントを参照ください。

また、DaemonSet におい
て[`.spec`](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status)セ
クションも必須となります。

### Pod テンプレート

`.spec.template`は`.spec`内での必須のフィールドの 1 つです。

`.spec.template`は[Pod テンプレート](/ja/docs/concepts/workloads/pods/pod-overview/#podテンプレート)と
なります。これはフィールドがネストされていて、`apiVersion`や`kind`をもたないこと
を除いては、[Pod](/ja/docs/concepts/workloads/pods/pod/)のテンプレートと同じスキ
ーマとなります。

Pod に対する必須のフィールドに加えて、DaemonSet 内の Pod テンプレートは適切なラ
ベルを指定しなくてはなりません([Pod セレクター](#pod-selector)の項目を参照くださ
い)。

DaemonSet 内の Pod テンプレートでは
、[`RestartPolicy`](/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy)フ
ィールドを指定せずにデフォルトの`Always`を使用するか、明示的に`Always`を設定する
かのどちらかである必要があります。

### Pod セレクター

`.spec.selector`フィールドは Pod セレクターとなります。これ
は[Job](/docs/concepts/jobs/run-to-completion-finite-workloads/)の`.spec.selector`と
同じものです。

Kubernetes1.8 のように、ユーザーは`.spec.template`のラベルにマッチする Pod セレ
クターを指定しなくてはいけません。Pod セレクターは、値を空のままにしてもデフォル
ト設定にならなくなりました。セレクターのデフォルト化は`kubectl apply`と互換性は
ありません。また、一度 DaemonSet が作成されると、その`.spec.selector`は変更不可
能になります。Pod セレクターの変更は、意図しない Pod の孤立を引き起こし、ユーザ
ーにとってやっかいなものとなります。

`.spec.selector`は 2 つのフィールドからなるオブジェクトです。

- `matchLabels` -
  [ReplicationController](/docs/concepts/workloads/controllers/replicationcontroller/)の`.spec.selector`と
  同じように機能します。
- `matchExpressions` - キーと、値のリストとさらにはそれらのキーとバリューに関連
  したオペレーターを指定することにより、より洗練された形式のセレクターを構成でき
  ます。

上記の 2 つが指定された場合は、2 つの条件を AND でどちらも満たすものを結果として
返します。

もし`spec.selector`が指定されたとき、`.spec.template.metadata.labels`とマッチし
なければなりません。この 2 つの値がマッチしない設定をした場合、API によってリジ
ェクトされます。

また、ユーザーは通常、別の DaemonSet や ReplicaSet などの別のワークロードリソー
スを使用する場合であっても直接であっても、このセレクターマッチするラベルを持つ
Pod を作成すべきではありません。さもないと、DaemonSet
{{<glossary_tooltip term_id = "controller">}}は、それらの Pod が作成されたものと
みなすためです。Kubernetes はこれを行うことを止めません。ユーザーがこれを行いた
い 1 つのケースとしては、テスト用にノード上に異なる値を持つ Pod を手動で作成する
ような場合があります。

### 特定のいくつかの Node 上のみに Pod を稼働させる

もしユーザーが`.spec.template.spec.nodeSelector`を指定したとき、DaemonSet コント
ローラーは、そ
の[node selector](/ja/docs/concepts/configuration/assign-pod-node/)にマッチする
Pod を Node 上に作成します。
同様に、もし`.spec.template.spec.affinity`を指定したとき、DaemonSet コントローラ
ーは[node affinity](/ja/docs/concepts/configuration/assign-pod-node/)マッチする
Pod を Node 上に作成します。もしユーザーがどちらも指定しないとき、DaemonSet コン
トローラーは全ての Node 上に Pod を作成します。

## Daemon Pod がどのようにスケジューリングされるか

### デフォルトスケジューラーによってスケジューリングされる場合(Kubernetes1.12 からデフォルトで有効)

{{< feature-state state="stable" for-kubernetes-version="1.17" >}}

DaemonSet は全ての利用可能な Node が単一の Pod のコピーを稼働させることを保証し
ます。通常、Pod が稼働する Node は Kubernetes スケジューラーによって選択されます
。しかし、DaemonSet の Pod は代わりに DaemonSet コントローラーによって作成され、
スケジューリングされます。
下記の問題について説明します:

- 矛盾する Pod のふるまい: スケジューリングされるのを待っている通常の Pod は、作
  成されているが`Pending`状態となりますが、DaemonSet の Pod は`Pending`状態で作
  成されません。これはユーザーにとって困惑するものです。
- [Pod プリエンプション(Pod preemption)](/docs/concepts/configuration/pod-priority-preemption/)は
  デフォルトスケジューラーによってハンドルされます。もしプリエンプションが有効な
  場合、その DaemonSet コントローラーは Pod の優先順位とプリエンプションを考慮す
  ることなくスケジューリングの判断を行います。

`ScheduleDaemonSetPods`は、DaemonSet の Pod に対して`NodeAffinity`項目を追加する
ことにより、DaemonSet コントローラーの代わりにデフォルトスケジューラーを使って
DaemonSet のスケジュールを可能にします。その際に、デフォルトスケジューラーは Pod
をターゲットのホストにバインドします。もし DaemonSet の NodeAffinity が存在する
とき、それは新しいものに置き換えられます。DaemonSet コントローラーは DaemonSet
の Pod の作成や修正を行うときのみそれらの操作を実施します。そして DaemonSet
の`.spec.template`フィールドに対しては何も変更が加えられません。

```yaml
nodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
      - matchFields:
          - key: metadata.name
            operator: In
            values:
              - target-host-name
```

さらに、`node.kubernetes.io/unschedulable:NoSchedule`という tolaration が
DaemonSet の Pod に自動的に追加されます。デフォルトスケジューラーは、DaemonSet
の Pod のスケジューリングのときに、`unschedulable`な Node を無視します。

### Taints と Tolerations

DaemonSet の Pod
は[Taints と Tolerations](/docs/concepts/configuration/taint-and-toleration)の設
定を尊重します。
下記の Tolerations は、関連する機能によって自動的に DaemonSet の Pod に追加され
ます。

| Toleration Key                           | Effect     | Version | Description                                                                                                                      |
| ---------------------------------------- | ---------- | ------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `node.kubernetes.io/not-ready`           | NoExecute  | 1.13+   | DaemonSet の Pod はネットワーク分割のような Node の問題が発生したときに除外されません。                                          |
| `node.kubernetes.io/unreachable`         | NoExecute  | 1.13+   | DaemonSet の Pod はネットワーク分割のような Node の問題が発生したときに除外されません。                                          |
| `node.kubernetes.io/disk-pressure`       | NoSchedule | 1.8+    |                                                                                                                                  |
| `node.kubernetes.io/memory-pressure`     | NoSchedule | 1.8+    |                                                                                                                                  |
| `node.kubernetes.io/unschedulable`       | NoSchedule | 1.12+   | DaemonSet の Pod はデフォルトスケジューラーによってスケジュール不可能な属性を許容(tolerate)します。                              |
| `node.kubernetes.io/network-unavailable` | NoSchedule | 1.12+   | ホストネットワークを使う DaemonSet の Pod はデフォルトスケジューラーによってネットワーク利用不可能な属性を許容(tolerate)します。 |

## Daemon Pod とのコミュニケーション

DaemonSet 内の Pod とのコミュニケーションをする際に考えられるパターンは以下の通
りです。:

- **Push**: DaemonSet 内の Pod は他のサービスに対して更新情報を送信するように設
  定されます。
- **NodeIP と Known Port**: Pod が NodeIP を介して疎通できるようにするため
  、DaemonSet 内の Pod は`hostPort`を使用できます。慣例により、クライアントは
  NodeIP のリストとポートを知っています。
- **DNS**: 同じ Pod セレクターを持
  つ[HeadlessService](/ja/docs/concepts/services-networking/service/#headless-service)を
  作成し、`endpoints`リソースを使って DaemonSet を探すか、DNS から複数の A レコ
  ードを取得します。
- **Service**: 同じ Pod セレクターを持つ Service を作成し、複数のうちのいずれか
  の Node 上の Daemon に疎通させるためにその Service を使います。

## DaemonSet の更新

もし Node ラベルが変更されたとき、その DaemonSet は直ちに新しくマッチした Node
に Pod を追加し、マッチしなくなった Node から Pod を削除します。

ユーザーは DaemonSet が作成した Pod を修正可能です。しかし、Pod は全てのフィール
ドの更新を許可していません。また、DaemonSet コントローラーは次の Node(同じ名前で
も)が作成されたときにオリジナルのテンプレートを使って Pod を作成します。

ユーザーは DaemonSet を削除可能です。`kubectl`コマンドで`--cascade=false`を指定
すると DaemonSet の Pod は Node 上に残り続けます。その後、同じセレクターで新しい
DaemonSet を作成すると、新しい DaemonSet は既存の Pod を再利用します。Pod で
DaemonSet を置き換える必要がある場合は、`updateStrategy`に従ってそれらを置き換え
ます。

ユーザーは DaemonSet 上
で[ローリングアップデートの実施](/docs/tasks/manage-daemon/update-daemon-set/)が
可能です。

## DaemonSet の代替案

### Init スクリプト

Node 上で直接起動することにより(例: `init`、`upstartd`、`systemd`を使用する)、デ
ーモンプロセスを稼働することが可能です。この方法は非常に良いですが、このようなプ
ロセスを DaemonSet を介して起動することはいくつかの利点があります。

- アプリケーションと同じ方法でデーモンの監視とログの管理ができる。
- デーモンとアプリケーションで同じ設定用の言語とツール(例: Pod テンプレート
  、`kubectl`)を使える。
- リソースリミットを使ったコンテナ内でデーモンを稼働させることにより、デーモンと
  アプリケーションコンテナの分離を促進します。しかし、これは Pod 内でなく、コン
  テナ内でデーモンを稼働させることにより可能です(Docker を介して直接起動する)。

### ベア Pod

特定の Node 上で稼働するように指定した Pod を直接作成することは可能です。しかし
、DaemonSet は Node の故障や Node の破壊的なメンテナンスやカーネルのアップグレー
ドなど、どのような理由に限らず、削除されたもしくは停止された Pod を置き換えます
。このような理由で、ユーザーは Pod 単体を作成するよりもむしろ DaemonSet を使うべ
きです。

### 静的 Pod Pods

Kubelet によって監視されているディレクトリに対してファイルを書き込むことによって
、Pod を作成することが可能です。これ
は[静的 Pod](/docs/concepts/cluster-administration/static-pod/)と呼ばれます。
DaemonSet と違い、静的 Pod は kubectl や他の Kubernetes API クライアントで管理で
きません。静的 Pod は ApiServer に依存しておらず、クラスターの自立起動時に最適で
す。また、静的 Pod は将来的には廃止される予定です。

### Deployment

DaemonSet は、Pod の作成し、その Pod が停止されることのないプロセスを持つことに
おいて[Deployment](/ja/docs/concepts/workloads/controllers/deployment/)と同様で
す(例: web サーバー、ストレージサーバー)。

フロントエンドのような Service のように、どのホスト上に Pod が稼働するか制御する
よりも、レプリカ数をスケールアップまたはスケールダウンしたりローリングアップデー
トする方が重要であるような、状態をもたない Service に対して Deployment を使って
ください。 Pod のコピーが全てまたは特定のホスト上で常に稼働していることが重要な
場合や、他の Pod の前に起動させる必要があるときに DaemonSet を使ってください。

{{% /capture %}}
