---
title: ラベル(Labels)とセレクター(Selectors)
content_template: templates/concept
weight: 40
---

{{% capture overview %}}

_ラベル(Labels)_ は Pod などのオブジェクトに割り当てられたキーとバリューのペアで
す。
ラベルはユーザーに関連した意味のあるオブジェクトの属性を指定するために使われるこ
とを目的としています。しかし Kubernetes のコアシステムに対して直接的にその意味を
暗示するものではありません。
ラベルはオブジェクトのサブセットを選択し、グルーピングするために使うことができま
す。また、ラベルはオブジェクトの作成時に割り当てられ、その後いつでも追加、修正が
できます。
各オブジェクトはキーとバリューのラベルのセットを定義できます。各キーは、単一のオ
ブジェクトに対してはユニークである必要があります。

```json
"metadata": {
  "labels": {
    "key1" : "value1",
    "key2" : "value2"
  }
}
```

ラベルは効率的な検索・閲覧を可能にし、UI や CLI 上での利用に最適です。 識別用途
でない情報は
、[アノテーション](/docs/concepts/overview/working-with-objects/annotations/)を
用いて記録されるべきです。

{{% /capture %}}

{{% capture body %}}

## ラベルを使う動機

ラベルは、クライアントにそのマッピング情報を保存することを要求することなく、ユー
ザー独自の組織構造をシステムオブジェクト上で疎結合にマッピングできます。

サービスデプロイメントとバッチ処理のパイプラインは多くの場合、多次元のエンティテ
ィとなります(例: 複数のパーティション、Deployment、リリーストラック、ティアー、
ティアー毎のマイクロサービスなど)
管理は分野横断的な操作が必要になることが多く、それによって厳密な階層表現、特にユ
ーザーによるものでなく、インフラストラクチャーによって定義された厳格な階層のカプ
セル化が破られます。

ラベルの例:

- `"release" : "stable"`, `"release" : "canary"`
- `"environment" : "dev"`, `"environment" : "qa"`,
  `"environment" : "production"`
- `"tier" : "frontend"`, `"tier" : "backend"`, `"tier" : "cache"`
- `"partition" : "customerA"`, `"partition" : "customerB"`
- `"track" : "daily"`, `"track" : "weekly"`

これらは単によく使われるラベルの例です。ユーザーは自由に規約を決めることができま
す。ラベルのキーは、ある 1 つのオブジェクトに対してユニークである必要があること
は覚えておかなくてはなりません。

## 構文と文字セット

ラベルは、キーとバリューのベアです。正しいラベルキーは 2 つのセグメントを持ちま
す。
それは`/`によって分割されたオプショナルなプレフィックスと名前です。
名前セグメントは必須で、63 文字以下である必要があり、文字列の最初と最後は英数字
(`[a-z0-9A-Z]`)で、文字列の間ではこれに加えてダッシュ(`-`)、アンダースコア
(`_`)、ドット(`.`)を使うことができます。
プレフィックスはオプションです。もしプレフィックスが指定されていた場合、プレフィ
ックスは DNS サブドメイン形式である必要があり、それはドット(`.`)で区切られた DNS
ラベルのセットで、253 文字以下である必要があり、最後にスラッシュ(`/`)が続きます
。

もしプレフィックスが省略された場合、ラベルキーはそのユーザーに対してプライベート
であると推定されます。
エンドユーザーのオブジェクトにラベルを追加するような自動化されたシステムコンポー
ネント(例: `kube-scheduler` `kube-controller-manager` `kube-apiserver`
`kubectl`やその他のサードパーティツール)は、プレフィックスを指定しなくてはなりま
せん。

`kubernetes.io/`と`k8s.io/`プレフィックスは、Kubernetes コアコンポーネントのため
に予約されています。

正しいラベル値は 63 文字以下の長さで、空文字か、もしくは開始と終了が英数字
(`[a-z0-9A-Z]`)で、文字列の間がダッシュ(`-`)、アンダースコア(`_`)、ドット(`.`)と
英数字である文字列を使うことができます。

## ラベルセレクター

[名前と UID](/docs/user-guide/identifiers)とは異なり、ラベルはユニーク性を提供し
ません。通常、多くのオブジェクトが同じラベルを保持することを想定します。

_ラベルセレクター_ を介して、クライアントとユーザーはオブジェクトのセットを指定
できます。ラベルセレクターは Kubernetes においてコアなグルーピング機能となります
。

Kubernetes API は現在 2 タイプのセレクターをサポートしています。
それは*等価ベース(equality-based)* と*集合ベース(set-based)* です。
単一のラベルセレクターは、コンマ区切りの複数の*要件(requirements)* で構成されて
います。
複数の要件がある場合、コンマセパレーターは論理積 _AND_(`&&`)オペレーターと同様に
ふるまい、全ての要件を満たす必要があります。

空文字の場合や、指定なしのセレクターに関するセマンティクスは、コンテキストに依存
します。そしてセレクターを使う API タイプは、それらのセレクターの妥当性とそれら
が示す意味をドキュメントに記載するべきです。

{{< note >}} ReplicaSet など、いくつかの API タイプにおいて、2 つのインスタンス
のラベルセレクターは単一の名前空間において重複してはいけません。重複していると、
コントローラがそれらのラベルセレクターがコンフリクトした操作とみなし、どれだけの
数のレプリカを稼働させるべきか決めることができなくなります。 {{< /note >}}

### _等価ベース(Equality-based)_ の要件(requirement)

_等価ベース(Equality-based)_ もしくは*不等ベース(Inequality-based)* の要件は、ラ
ベルキーとラベル値によるフィルタリングを可能にします。
要件に一致したオブジェクトは、指定されたラベルの全てを満たさなくてはいけませんが
、それらのオブジェクトはさらに追加のラベルも持つことができます。
そして等価ベースの要件においては、3 つの種類のオペレーターの利用が許可されていま
す。`=`、`==`、`!=`となります。
最初の 2 つのオペレーター(`=`、`==`)は*等価(Equality)* を表現し(この 2 つは単な
る同義語)、最後の 1 つ(`!=`)は*不等(Inequality)* を意味します。
例えば

```
environment = production
tier != frontend
```

最初の例は、キーが`environment`で、値が`production`である全てのリソースを対象に
します。
次の例は、キーが`tier`で、値が`frontend`とは異なるリソースと、`tier`という名前の
キーを持たない全てのリソースを対象にします。
コンマセパレーター`,`を使って、`production`の中から、`frontend`のものを除外する
ようにフィルターすることもできます。
`environment=production,tier!=frontend`

等価ベースのラベル要件の 1 つの使用シナリオとして、Pod における Node の選択要件
を指定するケースがあります。
例えば、下記のサンプル Pod は、ラベル`accelerator=nvidia-tesla-p100`をもった
Node を選択します。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cuda-test
spec:
  containers:
    - name: cuda-test
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
      resources:
        limits:
          nvidia.com/gpu: 1
  nodeSelector:
    accelerator: nvidia-tesla-p100
```

### _集合ベース(Set-based)_ の要件(requirement)

_集合ベース(Set-based)_ のラベルの要件は値のセットによってキーをフィルタリングし
ます。
`in`、`notin`、`exists`の 3 つのオペレーターをサポートしています(キーを特定する
のみ)。

例えば:

```
environment in (production, qa)
tier notin (frontend, backend)
partition
!partition
```

最初の例では、キーが`environment`で、値が`production`か`qa`に等しいリソースを全
て選択します。
第 2 の例では、キーが`tier`で、値が`frontend`と`backend`以外のもの、そし
て`tier`キーを持たないリソースを全て選択します。
第 3 の例では、`partition`というキーをもつラベルを全て選択し、値はチェックしませ
ん。
第 4 の例では、`partition`というキーを持たないラベルを全て選択し、値はチェックし
ません。
同様に、コンマセパレーターは、_AND_ オペレーターと同様にふるまいます。そのため
、`partition`と`environment`キーの値がともに`qa`でないラベルを選択するには
、`partition,environment notin (qa)`と記述することで可能です。
_集合ベース_ のラベルセレクターは、`environment=production`という記述
が`environment in (production)`と等しいため、一般的な等価形式となります。
`!=`と`notin`も同様に等価となります。

_集合ベース_ の要件は、_等価ベース_ の要件と混在できます。
例えば:
`partition in (customerA, customerB),environment!=qa`.

## API

### LIST と WATCH によるフィルタリング

LIST と WATCH オペレーションは、単一のクエリパラメータを使うことによって返される
オブジェクトのセットをフィルターするためのラベルセレクターを指定できます。
_集合ベース_ と*等価ベース* のどちらの要件も許可されています(ここでは、URL クエ
リストリング内で出現します)。

- _等価ベース_ での要件:
  `?labelSelector=environment%3Dproduction,tier%3Dfrontend`
- _集合ベース_ での要件:
  `?labelSelector=environment+in+%28production%2Cqa%29%2Ctier+in+%28frontend%29`

上記の 2 つの形式のラベルセレクターは REST クライアントを介してリストにしたり、
もしくは確認するために使われます。
例えば、`kubectl`によって`apiserver`をターゲットにし、_等価ベース_ の要件でフィ
ルターすると以下のように書けます。

```shell
kubectl get pods -l environment=production,tier=frontend
```

もしくは、_集合ベース_ の要件を指定すると以下のようになります。

```shell
kubectl get pods -l 'environment in (production),tier in (frontend)'
```

すでに言及したように、_集合ベース_ の要件は、_等価ベース_ の要件より表現力があり
ます。
例えば、値に対する*OR* オペレーターを実装して以下のように書けます。

```shell
kubectl get pods -l 'environment in (production, qa)'
```

もしくは、_exists_ オペレーターを介して、否定マッチングによる制限もできます。

```shell
kubectl get pods -l 'environment,environment notin (frontend)'
```

### API オブジェクトに参照を設定する

[`Service`](/docs/user-guide/services) と
[`ReplicationController`](/docs/user-guide/replication-controller)のような、いく
つかの Kubernetes オブジェクトでは、ラベルセレクター
を[Pod](/docs/user-guide/pods)のような他のリソースのセットを指定するのにも使われ
ます。

#### Service と ReplicationController

`Service`が対象とする Pod の集合は、ラベルセレクターによって定義されます。
同様に、`ReplicationController`が管理するべき Pod 数についてもラベルセレクターを
使って定義されます。

それぞれのオブジェクトに対するラベルセレクターはマップを使って`json`もしく
は`yaml`形式のファイルで定義され、_等価ベース_ のセレクターのみサポートされてい
ます。

```json
"selector": {
    "component" : "redis",
}
```

もしくは

```yaml
selector:
  component: redis
```

このセレクター(それぞれ`json`または`yaml`形式)は、`component=redis`また
は`component in (redis)`と等価です。

#### _集合ベース_ の要件指定をサポートするリソース

[`Job`](/docs/concepts/workloads/controllers/jobs-run-to-completion/)や[`Deployment`](/ja/docs/concepts/workloads/controllers/deployment/)、[`ReplicaSet`](/ja/docs/concepts/workloads/controllers/replicaset/)や[`DaemonSet`](/ja/docs/concepts/workloads/controllers/daemonset/)な
どの比較的新しいリソースは、_集合ベース_ での要件指定もサポートしています。

```yaml
selector:
  matchLabels:
    component: redis
  matchExpressions:
    - { key: tier, operator: In, values: [cache] }
    - { key: environment, operator: NotIn, values: [dev] }
```

`matchLabels`は、`{key,value}`ペアのマップです。`matchLabels`内の単一
の`{key,value}`は、`matchExpressions`の要素と等しく、それは、`key`フィールドがキ
ー名で、`operator`が"In"で、`values`配列は単に"値"を保持します。
`matchExpressions`は Pod セレクター要件のリストです。対応しているオペレーター
は`In`、`NotIn`、`Exists`と`DoesNotExist`です。`values`のセットは
、`In`と`NotIn`オペレーターにおいては空文字を許容しません。
`matchLabels`と`matchExpressions`の両方によって指定された全ての要件指定は AND で
判定されます。つまり要件にマッチするには指定された全ての要件を満たす必要がありま
す。

#### Node のセットを選択する

ラベルを選択するための 1 つのユースケースは Pod がスケジュールできる Node のセッ
トを制限することです。
さらなる情報に関しては
、[Node 選定](/ja/docs/concepts/configuration/assign-pod-node/) のドキュメントを
参照してください。

{{% /capture %}}
