---
reviewers:
title: StatefulSet
content_template: templates/concept
weight: 40
---

{{% capture overview %}}

StatefulSet はステートフルなアプリケーションを管理するためのワークロード API で
す。

{{< note >}} StatefulSet は Kubernetes1.9 において利用可能(GA)です。
{{< /note >}}

{{< glossary_definition term_id="statefulset" length="all" >}} {{% /capture %}}

{{% capture body %}}

## StatefulSet の使用

StatefulSet は下記の 1 つ以上の項目を要求するアプリケーションにおいて最適です。

- 安定した一意のネットワーク識別子
- 安定した永続ストレージ
- 規則的で安全なデプロイとスケーリング
- 規則的で自動化されたローリングアップデート

上記において安定とは、Pod のスケジュール(または再スケジュール)をまたいでも永続的
であることと同義です。もしアプリケーションが安定したネットワーク識別子と規則的な
デプロイや削除、スケーリングを全く要求しない場合、ユーザーはステートレスなレプリ
カのセットを提供するコントローラーを使ってアプリケーションをデプロイするべきです
。
[Deployment](/ja/docs/concepts/workloads/controllers/deployment/)や[ReplicaSet](/ja/docs/concepts/workloads/controllers/replicaset/)の
ようなコントローラーはこのようなステートレスな要求に対して最適です。

## 制限事項

- StatefuleSet は Kubernetes1.9 より以前のバージョンでは β 版のリソースであり
  、1.5 より前のバージョンでは利用できません。
- 提供された Pod のストレージは、要求された`storage class`にもとづい
  て[PersistentVolume
  Provisioner](https://github.com/kubernetes/examples/tree/{{< param
  "githubbranch" >}}/staging/persistent-volume-provisioning/README.md)によってプ
  ロビジョンされるか、管理者によって事前にプロビジョンされなくてはなりません。
- StatefulSet の削除もしくはスケールダウンをすることにより、StatefulSet に関連し
  たボリュームは削除*されません* 。 これはデータ安全性のためで、関連する
  StatefulSet のリソース全てを自動的に削除するよりもたいてい有効です。
- StatefulSet は現在、Pod のネットワークアイデンティティーに責務をもつため
  に[Headless Service](/ja/docs/concepts/services-networking/service/#headless-service)を
  要求します。ユーザーはこの Service を作成する責任があります。
- StatefulSet は、StatefulSet が削除されたときに Pod の停止を行うことを保証して
  いません。StatefulSet において、規則的で安全な Pod の停止を行う場合、削除のた
  めに事前にその StatefulSet の数を 0 にスケールダウンさせることが可能です。
- デフォルト設定の[Pod 管理ポリシー](#pod-management-policies) (`OrderedReady`)
  によって[ローリングアップデート](#rolling-updates)を行う場合
  、[修復のための手動介入](#forced-rollback)を要求するようなブロークンな状態に遷
  移させることが可能です。

## コンポーネント

下記の例は、StatefulSet のコンポーネントのデモンストレーションとなります。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
    - port: 80
      name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # .spec.template.metadata.labelsの値と一致する必要があります
  serviceName: "nginx"
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: nginx # .spec.selector.matchLabelsの値と一致する必要があります
    spec:
      terminationGracePeriodSeconds: 10
      containers:
        - name: nginx
          image: k8s.gcr.io/nginx-slim:0.8
          ports:
            - containerPort: 80
              name: web
          volumeMounts:
            - name: www
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
    - metadata:
        name: www
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: "my-storage-class"
        resources:
          requests:
            storage: 1Gi
```

上記の例では、

- nginx という名前の HeadlessService は、ネットワークドメインをコントロールする
  ために使われます。
- web という名前の StatefulSet は、spec で 3 つの nginx コンテナのレプリカを持ち
  、そのコンテナはそれぞれ別の Pod で稼働するように設定されています。
- volumeClaimTemplates は、PersistentVolume プロビジョナーによってプロビジョンさ
  れた[PersistentVolume](/docs/concepts/storage/persistent-volumes/)を使って安定
  したストレージを提供します。

## Pod セレクター

ユーザーは、StatefulSet の`.spec.template.metadata.labels`のラベルと一致させるた
め、StatefulSet の`.spec.selector`フィールドをセットしなくてはなりません
。Kubernetes1.8 以前では、`.spec.selector`フィールドは省略された場合デフォルト値
になります。Kubernetes1.8 とそれ以降のバージョンでは、ラベルに一致する Pod セレ
クターの指定がない場合は StatefulSet の作成時にバリデーションエラーになります。

## Pod アイデンティティー

StatefulSet の Pod は、順番を示す番号、安定したネットワークアイデンティティー、
安定したストレージからなる一意なアイデンティティーを持ちます。  
そのアイデンティティーはどの Node 上にスケジュール(もしくは再スケジュール)される
かに関わらず、その Pod に紐付きます。

### 順序インデックス

N 個のレプリカをもった StatefulSet において、StatefulSet 内の各 Pod は、0 からは
じまり N-1 までの整数値を順番に割り当てられ、その StatefulSet においては一意とな
ります。

### 安定したネットワーク ID

StatefulSet 内の各 Pod は、その StatefulSet 名と Pod の順序番号から派生してホス
トネームが割り当てられます。作成されたホストネームの形式
は`$(StatefulSet名)-$(順序番号)`となります。先ほどの上記の例では
、`web-0,web-1,web-2`という 3 つの Pod が作成されます。 StatefulSet は、Pod のド
メインをコントロールするため
に[Headless Service](/ja/docs/concepts/services-networking/service/#headless-service)を
使うことができます。  
この Headless Service によって管理されたドメイン
は`$(Service名).$(ネームスペース).svc.cluster.local`形式となり、"cluster.local"
というのはそのクラスターのドメインとなります。  
各 Pod が作成されると、Pod は`$(Pod名).$(管理するServiceドメイン名)`に一致する
DNS サブドメインを取得し、管理する Service は StatefulSet の`serviceName`で定義
されます。

[制限事項](#制限事項)セクションで言及したように、ユーザーは Pod のネットワークア
イデンティティーのため
に[Headless Service](/ja/docs/concepts/services-networking/service/#headless-service)を
作成する責任があります。

ここで、クラスタードメイン、Service 名、StatefulSet 名の選択と、それらが
StatefulSet の Pod の DNS 名にどう影響するかの例をあげます。

| Cluster Domain | Service (ns/name) | StatefulSet (ns/name) | StatefulSet Domain              | Pod DNS                                      | Pod Hostname |
| -------------- | ----------------- | --------------------- | ------------------------------- | -------------------------------------------- | ------------ |
| cluster.local  | default/nginx     | default/web           | nginx.default.svc.cluster.local | web-{0..N-1}.nginx.default.svc.cluster.local | web-{0..N-1} |
| cluster.local  | foo/nginx         | foo/web               | nginx.foo.svc.cluster.local     | web-{0..N-1}.nginx.foo.svc.cluster.local     | web-{0..N-1} |
| kube.local     | foo/nginx         | foo/web               | nginx.foo.svc.kube.local        | web-{0..N-1}.nginx.foo.svc.kube.local        | web-{0..N-1} |

{{< note >}} クラスタードメイン
は[その他の設定](/docs/concepts/services-networking/dns-pod-service/#how-it-works)が
されない限り、`cluster.local`にセットされます。 {{< /note >}}

### 安定したストレージ

Kubernetes は各 VolumeClaimTemplate に対して、1 つ
の[PersistentVolume](/docs/concepts/storage/persistent-volumes/)を作成します。上
記の nginx の例において、各 Pod は`my-storage-class`という StorageClass をもち
、1Gib のストレージ容量を持った単一の PersistentVolume を受け取ります。もし
StorageClass が指定されていない場合、デフォルトの StorageClass が使用されます
。Pod が Node 上にスケジュール(もしくは再スケジュール)されたとき、そ
の`volumeMounts`は PersistentVolume Claim に関連した PersistentVolume をマウント
します。  
注意点として、Pod の PersistentVolume Claim と関連した PersistentVolume は、Pod
や StatefulSet が削除されたときに削除されません。削除する場合は手動で行わなけれ
ばなりません。

### Pod のネームラベル

StatefulSet のコントローラーが Pod を作成したとき、Pod の名前として
、`statefulset.kubernetes.io/pod-name`にラベルを追加します。このラベルによってユ
ーザーは Service に StatefulSet 内の指定した Pod を割り当てることができます。

## デプロイとスケーリングの保証

- N 個のレプリカをもつ StatefulSet において、Pod がデプロイされるとき、それらの
  Pod は{0..N-1}の番号で順番に作成されます。
- Pod が削除されるとき、それらの Pod は{N-1..0}の番号で降順に削除されます。
- Pod に対してスケーリングオプションが適用される前に、その Pod の前の順番の全て
  の Pod が Running かつ Ready 状態になっていなくてはなりません。
- Pod が停止される前に、その Pod の番号より大きい番号を持つの全ての Pod は完全に
  シャットダウンされていなくてはなりません。

StatefulSet は`pod.Spec.TerminationGracePeriodSeconds`を 0 に指定すべきではあり
ません。これは不安全で、やらないことを強く推奨します。さらなる説明としては
、[StatefulSet の Pod の強制削除](/docs/tasks/run-application/force-delete-stateful-set-pod/)を
参照してください。

上記の例の nginx が作成されたとき、3 つの Pod は`web-0`、`web-1`、`web-2`の順番
でデプロイされます
。`web-1`は`web-0`が[Running かつ Ready 状態](/docs/user-guide/pod-states/)にな
るまでは決してデプロイされないのと、同様に`web-2`は`web-1`が Running かつ Ready
状態にならないとデプロイされません。もし`web-0`が`web-1`が Running かつ Ready 状
態になった後だが、`web-2`が起動する前に失敗した場合、`web-2`は`web-0`の再起動が
成功し、Running かつ Ready 状態にならないと再起動されません。

もしユーザーが`replicas=1`といったように StatefulSet にパッチをあてることにより
、デプロイされたものをスケールすることになった場合、`web-2`は最初に停止されます
。`web-1`は`web-2`が完全にシャットダウンされ削除されるまでは、停止されません。も
し`web-0`が、`web-2`が完全に停止され削除された後だが、`web-1`の停止の前に失敗し
た場合、`web-1`は`web-0`が Running かつ Ready 状態になるまでは停止されません。

### Pod の管理ポリシー

Kubernetes1.7 とそれ以降のバージョンでは、StatefulSet
は`.spec.podManagementPolicy`フィールドを介して、Pod の一意性とアイデンティティ
ーを保証します。

#### OrderedReady な Pod 管理

`OrderedReady`な Pod 管理は StatefulSet においてデフォルトです。これ
は[デプロイとスケーリングの保証](#deployment-and-scaling-guarantees)に記載されて
いる項目の振る舞いを実装します。

#### 並行な Pod 管理

`Parallel`な Pod 管理は、StatefulSet コントローラーに対して、他の Pod が起動や停
止される前にその Pod が完全に起動し準備完了になるか停止するのを待つことなく、Pod
が並行に起動もしくは停止するように指示します。

## アップデートストラテジー

Kubernetes1.7 とそれ以降のバージョンにおいて、StatefulSet
の`.spec.updateStarategy`フィールドで、コンテナの自動のローリングアップデートの
設定やラベル、リソースのリクエストとリミットや、StatefulSet 内の Pod のアノテー
ションを指定できます。

### OnDelete

`OnDelete`というアップデートストラテジーは、レガシーな(Kubernetes1.6 以前)振る舞
いとなります。StatefulSet の`.spec.updateStrategy.type`が`OnDelete`にセットされ
ていたとき、その StatefulSet コントローラーは StatefulSet 内で Pod を自動的に更
新しません。StatefulSet の`.spec.template`項目の修正を反映した新しい Pod の作成
をコントローラーに支持するためには、ユーザーは手動で Pod を削除しなければなりま
せん。

### RollinUpdate

`RollinUpdate`というアップデートストラテジーは、StatefulSet 内の Pod に対する自
動化されたローリングアップデートの機能を実装します。これ
は`.spec.updateStrategy`フィールドが未指定の場合のデフォルトのストラテジーです
。StatefulSet の`.spec.updateStrategy.type`が`RollingUpdate`にセットされたとき、
その StatefulSet コントローラーは、StatefulSet 内の Pod を削除し、再作成します。
これは Pod の停止(Pod の番号の降順)と同じ順番で、一度に 1 つの Pod を更新します
。コントローラーは、その前の Pod の状態が Running かつ Ready 状態になるまで次の
Pod の更新を待ちます。

#### パーティション

`RollingUpdate`というアップデートストラテジーは
、`.spec.updateStrategy.rollingUpdate.partition`を指定することにより、パーティシ
ョンに分けることができます。もしパーティションが指定されていたとき、そのパーティ
ションの値と等しいか、大きい番号を持つ Pod が更新されます。パーティションの値よ
り小さい番号を持つ Pod は更新されず、たとえそれらの Pod が削除されたとしても、そ
れらの Pod は以前のバージョンで再作成されます。もし StatefulSet
の`.spec.updateStrategy.rollingUpdate.partition`が、`.spec.replicas`より大きい場
合、`.spec.template`への更新は Pod に反映されません。  
多くのケースの場合、ユーザーはパーティションを使う必要はありませんが、もし一部の
更新を行う場合や、カナリー版のバージョンをロールアウトする場合や、段階的ロールア
ウトを行う場合に最適です。

#### 強制ロールバック

デフォルトの[Pod 管理ポリシー](#pod-management-policies)(`OrderedReady`)によ
る[ローリングアップデート](#rolling-updates)を行う際、修復のために手作業が必要な
状態にすることが可能です。

もしユーザーが、決して Running かつ Ready 状態にならないような設定になるように
Pod テンプレートを更新した場合(例えば、不正なバイナリや、アプリケーションレベル
の設定エラーなど)、StatefulSet はロールアウトを停止し、待機します。

この状態では、Pod テンプレートを正常な状態に戻すだけでは不十分です
。[既知の問題](https://github.com/kubernetes/kubernetes/issues/67250)によって
、StatefulSet は元の正常な状態へ戻す前に、壊れた Pod が Ready 状態(決して起こり
えない)に戻るのを待ち続けます。

そのテンプレートを戻したあと、ユーザーはまた StatefulSet が異常状態で稼働しよう
としていた Pod をすべて削除する必要があります。StatefulSet はその戻されたテンプ
レートを使って Pod の再作成を始めます。

{{% /capture %}} {{% capture whatsnext %}}

- [ステートフルなアプリケーションのデプロイ](/docs/tutorials/stateful-application/basic-stateful-set/)の
  例を参考にしてください。
- [StatefulSet を使った Cassandra のデプロイ](/docs/tutorials/stateful-application/cassandra/)の
  例を参考にしてください。

{{% /capture %}}
