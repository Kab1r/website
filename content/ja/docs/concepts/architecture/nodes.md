---
title: ノード
content_template: templates/concept
weight: 10
---

{{% capture overview %}}

ノードは、以前には `ミニオン` としても知られていた、Kubernetes におけるワーカー
マシンです。1 つのノードはクラスターの性質にもよりますが、1 つの VM または物理的
なマシンです。各ノードには[Pod](/ja/docs/concepts/workloads/pods/pod/)を動かすた
めに必要なサービスが含まれており、マスターコンポーネントによって管理されています
。ノード上のサービスに
は[コンテナランタイム](/ja/docs/concepts/overview/components/#container-runtime)、kubelet、kube-proxy
が含まれています。詳細については、設計ドキュメント
の[Kubernetes Node](https://git.k8s.io/community/contributors/design-proposals/architecture/architecture.md#the-kubernetes-node)セ
クションをご覧ください。

{{% /capture %}}

{{% capture body %}}

## ノードのステータス

ノードのステータスには以下のような情報が含まれます:

- [Addresses](#addresses)
- [Conditions](#condition)
- [Capacity と Allocatable](#capacity)
- [Info](#info)

ノードのステータスや、ノードに関するその他の詳細は、下記のコマンドを使うことで表
示できます:

```shell
kubectl describe node <ノード名>
```

各セクションについては、下記で説明します。

### Addresses

これらのフィールドの使い方は、お使いのクラウドプロバイダーやベアメタルの設定内容
によって異なります。

- HostName: ノードのカーネルによって伝えられたホスト名です。kubelet
  の`--hostname-override`パラメーターによって上書きすることができます。
- ExternalIP: 通常は、外部にルーティング可能(クラスターの外からアクセス可能)なノ
  ードの IP アドレスです。
- InternalIP: 通常は、クラスター内でのみルーティング可能なノードの IP アドレスで
  す。

### Conditions {#condition}

`conditions`フィールドは全ての`Running`なノードのステータスを表します。例として
、以下のような状態を含みます:

| ノードの Condition   | 概要                                                                                                                                                                                                                                                                                    |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `OutOfDisk`          | 新しい Pod を追加するために必要なディスク容量が足りない場合に`True`になります。それ以外のときは`False`です。                                                                                                                                                                            |
| `Ready`              | ノードの状態が Healthy で Pod を配置可能な場合に`True`になります。ノードの状態に問題があり、Pod が配置できない場合に`False`になります。ノードコントローラーが、`node-monitor-grace-period`で設定された時間内(デフォルトでは 40 秒)に該当ノードと疎通できない場合、`Unknown`になります。 |
| `MemoryPressure`     | ノードのメモリが圧迫されているときに`True`になります。圧迫とは、メモリの空き容量が少ないことを指します。それ以外のときは`False`です。                                                                                                                                                   |
| `PIDPressure`        | プロセスが圧迫されているときに`True`になります。圧迫とは、プロセス数が多すぎることを指します。それ以外のときは`False`です。                                                                                                                                                             |
| `DiskPressure`       | ノードのディスク容量が圧迫されているときに`True`になります。圧迫とは、ディスクの空き容量が少ないことを指します。それ以外のときは`False`です。                                                                                                                                           |
| `NetworkUnavailable` | ノードのネットワークが適切に設定されていない場合に`True`になります。それ以外のときは`False`です。                                                                                                                                                                                       |

ノードの Condition は JSON オブジェクトで表現されます。例えば、正常なノードの場
合は以下のようなレスポンスが表示されます。

```json
"conditions": [
  {
    "type": "Ready",
    "status": "True",
    "reason": "KubeletReady",
    "message": "kubelet is posting ready status",
    "lastHeartbeatTime": "2019-06-05T18:38:35Z",
    "lastTransitionTime": "2019-06-05T11:41:27Z"
  }
]
```

Ready condition が`pod-eviction-timeout`に設定された時間を超えて
も`Unknown`や`False`のままになっている場合
、[kube-controller-manager](/docs/admin/kube-controller-manager/)に引数が渡され
、該当ノード上にある Pod はノードコントローラーによって削除がスケジュールされま
す。デフォルトの退役のタイムアウトの時間は**5 分**です。ノードが到達不能ないくつ
かの場合においては、API サーバーが該当ノードの kubelet と疎通できない状態になっ
ています。その場合、API サーバーが kubelet と再び通信を確立するまでの間、Pod の
削除を行うことはできません。削除がスケジュールされるまでの間、削除対象の Pod た
ちは切り離されたノードの上で稼働を続けることになります。

バージョン 1.5 よりも前の Kubernetes では、ノードコントローラーは API サーバーか
ら到達不能なそれらの Pod
を[強制削除](/ja/docs/concepts/workloads/pods/pod/#podの強制削除)していました。
しかしながら、1.5 以降では、ノードコントローラーはクラスター内で Pod が停止する
のを確認するまでは強制的に削除しないようになりました。到達不能なノード上で動いて
いる Pod は`Terminating`または`Unknown`のステータスになります。Kubernetes が基盤
となるインフラストラクチャーを推定できない場合、クラスター管理者は手動で Node オ
ブジェクトを削除する必要があります。Kubernetes から Node オブジェクトを削除する
と、そのノードで実行されているすべての Pod オブジェクトが API サーバーから削除さ
れ、それらの名前が解放されます。

バージョン 1.12 において、`TaintNodesByCondition`機能が Beta に昇格し、それによ
ってノードのライフサイクルコントローラーが condition を表し
た[taint](/docs/concepts/configuration/taint-and-toleration/)を自動的に生成する
ようになりました。同様に、スケジューラーが Pod を配置するノードを検討する際、ノ
ードの taint と Pod の tolerations を見るかわりに condition を無視するようになり
ました。

ユーザーは、古いスケジューリングモデルか、新しくてより柔軟なスケジューリングモデ
ルのどちらかを選択できるようになりました。上記の toleration がない Pod は古いス
ケジュールモデルに従ってスケジュールされます。しかし、特定のノードの taint を許
容する Pod については、条件に合ったノードにスケジュールすることができます。

{{< caution >}}

この機能を有効にすると、condition が観測されてから taint が作成されるまでの間に
わずかな遅延が発生します。この遅延は通常 1 秒未満ですが、正常にスケジュールされ
ているが、kubelet によって配置を拒否された Pod の数が増える可能性があります。

{{< /caution >}}

### Capacity と Allocatable {#capacity}

ノードで利用可能なリソース（CPU、メモリ、およびノードでスケジュールできる最大
Pod 数）について説明します。

capacity ブロック内のフィールドは、ノードが持っているリソースの合計量を示します
。 allocatable ブロックは、通常の Pod によって消費されるノード上のリソースの量を
示します。

Capacity と Allocatable について深く知りたい場合は、ノード上でどのよう
に[コンピュートリソースが予約されるか](/docs/tasks/administer-cluster/reserve-compute-resources/#node-allocatable)を
読みながら学ぶことができます。

### Info

カーネルのバージョン、Kubernetes のバージョン（kubelet および kube-proxy のバー
ジョン）、（使用されている場合）Docker のバージョン、OS 名など、ノードに関する一
般的な情報です。この情報はノードから kubelet を通じて取得されます。

## 管理 {#management}

[Pod](/ja/docs/concepts/workloads/pods/pod/)や[Service](/ja/docs/concepts/services-networking/service/)と
違い、ノードは本質的には Kubernetes によって作成されません。GCP のようなクラウド
プロバイダーによって外的に作成されるか、VM や物理マシンのプールに存在するもので
す。そのため、Kubernetes がノードを作成すると、そのノードを表すオブジェクトが作
成されます。作成後、Kubernetes はそのノードが有効かどうかを確認します。 たとえば
、次の内容からノードを作成しようとしたとします:

```json
{
  "kind": "Node",
  "apiVersion": "v1",
  "metadata": {
    "name": "10.240.79.157",
    "labels": {
      "name": "my-first-k8s-node"
    }
  }
}
```

Kubernetes は内部的に Node オブジェクトを作成し、 `metadata.name`フィールドに基
づくヘルスチェックによってノードを検証します。ノードが有効な場合、つまり必要なサ
ービスがすべて実行されている場合は、Pod を実行する資格があります。それ以外の場合
、該当ノードが有効になるまではいかなるクラスターの活動に対しても無視されます。

{{< note >}} Kubernetes は無効なノードのためにオブジェクトを保存し、それをチェッ
クし続けます。このプロセスを停止するには、Node オブジェクトを明示的に削除する必
要があります。 {{< /note >}}

現在、Kubernetes のノードインターフェースと相互作用する 3 つのコンポーネントがあ
ります。ノードコントローラー、kubelet、および kubectl です。

### ノードコントローラー

ノードコントローラーは、ノードのさまざまな側面を管理する Kubernetes のマスターコ
ンポーネントです。

ノードコントローラーは、ノードの存続期間中に複数の役割を果たします。1 つ目は、ノ
ードが登録されたときに CIDR ブロックをノードに割り当てることです（CIDR 割り当て
がオンになっている場合）。

2 つ目は、ノードコントローラーの内部ノードリストをクラウドの利用可能なマシンのリ
ストと一致させることです。クラウド環境で実行している場合、ノードに異常があると、
ノードコントローラーはクラウドプロバイダーにその Node の VM がまだ使用可能かどう
かを問い合わせます。使用可能でない場合、ノードコントローラーはノードのリストから
該当ノードを削除します。

3 つ目は、ノードの状態を監視することです。ノードが到達不能(例えば、ノードがダウ
ンしているなどので理由で、ノードコントローラーがハートビートの受信を停止した場合
)になると、ノードコントローラーは、NodeStatus の NodeReady condition を
ConditionUnknown に変更する役割があります。その後も該当ノードが到達不能のままで
あった場合、Graceful Termination を使って全ての Pod を退役させます。デフォルトの
タイムアウトは、ConditionUnknown の報告を開始するまで 40 秒、その後 Pod の追い出
しを開始するまで 5 分に設定されています。ノードコントローラーは
、`--node-monitor-period`に設定された秒数ごとに各ノードの状態をチェックします。

バージョン 1.13 よりも前の Kubernetes において、NodeStatus はノードからのハート
ビートでした。Kubernetes 1.13 から、NodeLease がアルファ機能として導入されました
（Feature Gate `NodeLease`,
[KEP-0009](https://github.com/kubernetes/community/blob/master/keps/sig-node/0009-node-heartbeat.md)）
。

NodeLease が有効になっている場合、各ノードは `kube-node-lease`という Namespace
に関連付けられた`Lease`オブジェクトを持ち、ノードによって定期的に更新されます
。NodeStatus と NodeLease の両方がノードからのハートビートとして扱われます
。NodeLease は頻繁に更新されますが、NodeStatus はノードからマスターへの変更があ
るか、または十分な時間が経過した場合にのみ報告されます（デフォルトは 1 分で、到
達不能の場合のデフォルトタイムアウトである 40 秒よりも長いです）。NodeLease は
NodeStatus よりもはるかに軽量であるため、スケーラビリティとパフォーマンスの両方
の観点においてノードのハートビートのコストを下げます。

Kubernetes 1.4 では、マスターに問題が発生した場合の対処方法を改善するように、ノ
ードコントローラーのロジックをアップデートしています（マスターのネットワークに問
題があるため）バージョン 1.4 以降、ノードコントローラーは、Pod の退役について決
定する際に、クラスター内のすべてのノードの状態を調べます。

ほとんどの場合、排除の速度は 1 秒あたり`--node-eviction-rate`に設定された数値（
デフォルトは秒間 0.1）です。つまり、10 秒間に 1 つ以上の Pod をノードから追い出
すことはありません。

特定のアベイラビリティーゾーン内のノードのステータスが異常になると、ノード排除の
挙動が変わります。ノードコントローラーは、ゾーン内のノードの何%が異常（NodeReady
条件が ConditionUnknown または ConditionFalse である）であるかを同時に確認します
。異常なノードの割合が少なくとも `--healthy-zone-threshold`に設定した値を下回る
場合（デフォルトは 0.55）であれば、退役率は低下します。クラスターが小さい場合（
すなわち、 `--large-cluster-size-threshold`の設定値よりもノード数が少ない場合。
デフォルトは 50）、退役は停止し、そうでない場合、退役率は秒間
で`--secondary-node-eviction-rate`の設定値（デフォルトは 0.01）に減少します。こ
れらのポリシーがアベイラビリティーゾーンごとに実装されているのは、1 つのアベイラ
ビリティーゾーンがマスターから分割される一方、他のアベイラビリティーゾーンは接続
されたままになる可能性があるためです。クラスターが複数のクラウドプロバイダーのア
ベイラビリティーゾーンにまたがっていない場合、アベイラビリティーゾーンは 1 つ�
けです（クラスター全体）。

ノードを複数のアベイラビリティゾーンに分散させる主な理由は、1 つのゾーン全体が停
止したときにワークロードを正常なゾーンに移動できることです。したがって、ゾーン内
のすべてのノードが異常である場合、ノードコントローラーは通常のレート
`--node-eviction-rate`で退役します。コーナーケースは、すべてのゾーンが完全に
Unhealthy である（すなわち、クラスタ内に Healthy なノードがない）場合です。この
ような場合、ノードコントローラーはマスター接続に問題があると見なし、接続が回復す
るまですべての退役を停止します。

Kubernetes 1.6 以降では、ノードコントローラーは、Pod が taint を許容しない場合、
`NoExecute`の taint を持つノード上で実行されている Pod を排除する責務もあります
。さらに、デフォルトで無効になっているアルファ機能として、ノードコントローラーは
ノードに到達できない、または準備ができていないなどのノードの問題に対応する taint
を追加する責務があります。 `NoExecute`の taint 及び上述のアルファ機能に関する詳
細は
、[こちらのドキュメント](/docs/concepts/configuration/taint-and-toleration/)をご
覧ください。

バージョン 1.8 以降、ノードコントローラーに対してノードの状態を表す taint を作成
する責務を持たせることができます。これはバージョン 1.8 のアルファ機能です。

### ノードの自己登録

kubelet のフラグ `--register-node`が true（デフォルト）のとき、kubelet は自分自
身を API サーバーに登録しようとします。これはほとんどのディストリビューションで
使用されている推奨パターンです。

自己登録については、kubelet は以下のオプションを伴って起動されます:

- `--kubeconfig` - 自分自身を API サーバーに対して認証するための資格情報へのパス
- `--cloud-provider` - 自身に関するメタデータを読むためにクラウドプロバイダーと
  会話する方法
- `--register-node` - 自身を API サーバーに自動的に登録
- `--register-with-taints` - 与えられた taint のリストでノードを登録します (カン
  マ区切りの `<key>=<value>:<effect>`). `register-node`が false の場合、このオプ
  ションは機能しません
- `--node-ip` - ノードの IP アドレス
- `--node-labels` - ノードをクラスターに登録するときに追加するラベル（1.13 以降
  の[NodeRestriction 許可プラグイン](/docs/reference/access-authn-authz/admission-controllers/#noderestriction)に
  よって適用されるラベルの制限を参照）
- `--node-status-update-frequency` - kubelet がノードのステータスをマスターに
  POST する頻度の指定

[ノード認証モード](/docs/reference/access-authn-authz/node/)およ
び[NodeRestriction 許可プラグイン](/docs/reference/access-authn-authz/admission-controllers/#noderestriction)が
有効になっている場合、kubelet は自分自身のノードリソースを作成/変更することのみ
許可されています。

#### 手動によるノード管理 {#manual-node-administration}

クラスター管理者は Node オブジェクトを作成および変更できます。

管理者が手動で Node オブジェクトを作成したい場合は、kubelet フラグ
`--register-node = false`を設定してください。

管理者は`--register-node`の設定に関係なく Node リソースを変更することができます
。変更には、ノードにラベルを設定し、それを unschedulable としてマークすることが
含まれます。

ノード上のラベルは、スケジューリングを制御するために Pod 上のノードセレクタと組
み合わせて使用できます。例えば、Pod をノードのサブセットでのみ実行する資格がある
ように制限します。

ノードを unschedulable としてマークすると、新しい Pod がそのノードにスケジュール
されるのを防ぎますが、ノード上の既存の Pod には影響しません。これは、ノードの再
起動などの前の準備ステップとして役立ちます。たとえば、ノードにスケジュール不可能
のマークを付けるには、次のコマンドを実行します:

```shell
kubectl cordon $ノード名
```

{{< note >}} DaemonSet コントローラーによって作成された Pod は Kubernetes スケジ
ューラーをバイパスし、ノード上の unschedulable 属性を考慮しません。これは、再起
動の準備中にアプリケーションからアプリケーションが削除されている場合でも、デーモ
ンがマシンに属していることを前提としているためです。 {{< /note >}}

### ノードのキャパシティ

ノードのキャパシティ（CPU の数とメモリの量）は Node オブジェクトの一部です。通常
、ノードは自分自身を登録し、Node オブジェクトを作成するときにキャパシティを報告
します。 [手動によるノード管理](#manual-node-administration)を実行している場合は
、ノードを追加するときにキャパシティを設定する必要があります。

Kubernetes スケジューラーは、ノード上のすべての Pod に十分なリソースがあることを
確認します。ノード上のコンテナが要求するリソースの合計がノードキャパシティ以下で
あることを確認します。これは、kubelet によって開始されたすべてのコンテナを含みま
すが
、[コンテナランタイム](/ja/docs/concepts/overview/components/#container-runtime)に
よって直接開始されたコンテナやコンテナの外で実行されているプロセスは含みません。

Pod 以外のプロセス用にリソースを明示的に予約したい場合は、このチュートリアルに従
っ
て[System デーモン用にリソースを予約](/docs/tasks/administer-cluster/reserve-compute-resources/#system-reserved)し
てください。

## API オブジェクト

Node は Kubernetes の REST API におけるトップレベルのリソースです。API オブジェ
クトに関する詳細は以下の記事にてご覧いただけます: [Node API
オブジェクト](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#node-v1-core).

{{% /capture %}}
