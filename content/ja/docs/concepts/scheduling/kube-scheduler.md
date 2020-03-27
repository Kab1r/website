---
title: Kubernetesのスケジューラー
content_template: templates/concept
weight: 60
---

{{% capture overview %}}

Kubernetes において、_スケジューリング_ とは
、{{< glossary_tooltip term_id="kubelet" >}}が{{< glossary_tooltip text="Pod" term_id="pod" >}}を
稼働させるために{{< glossary_tooltip text="Node" term_id="node" >}}に割り当てる
ことを意味します。

{{% /capture %}}

{{% capture body %}}

## スケジューリングの概要{#scheduling}

スケジューラーは新規に作成された Pod で、Node に割り当てられていないものを監視し
ます。スケジューラーは発見した各 Pod のために、稼働させるべき最適な Node を見つ
け出す責務を担っています。そのスケジューラーは下記で説明するスケジューリングの原
理を考慮に入れて、Node への Pod の割り当てを行います。

Pod が特定の Node に割り当てられる理由を理解したい場合や、カスタムスケジューラー
を自身で作ろうと考えている場合、このページはスケジューリングに関して学ぶのに役立
ちます。

## kube-scheduler

[kube-scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)は
Kubernetes におけるデフォルトのスケジューラーで
、{{< glossary_tooltip text="コントロールプレーン" term_id="control-plane" >}}の
一部分として稼働します。  
kube-scheduler は、もし希望するのであれば自分自身でスケジューリングのコンポーネ
ントを実装でき、それを代わりに使用できるように設計されています。

kube-scheduler は、新規に作成された各 Pod や他のスケジューリングされていない Pod
を稼働させるために最適な Node を選択します。  
しかし、Pod 内の各コンテナにはそれぞれ異なるリソースの要件があり、各 Pod 自体に
もそれぞれ異なる要件があります。そのため、既存の Node は特定のスケジューリング要
求によってフィルターされる必要があります。

クラスター内で Pod に対する割り当て要求を満たした Node は*割り当て可能* な Node
と呼ばれます。  
もし適切な Node が一つもない場合、スケジューラーが Node を割り当てることができる
まで、その Pod はスケジュールされずに残ります。

スケジューラーは Pod に対する割り当て可能な Node をみつけ、それらの割り当て可能
な Node にスコアをつけます。その中から最も高いスコアの Node を選択し、Pod に割り
当てるためのいくつかの関数を実行します。  
スケジューラーは*binding* と呼ばれる処理中において、API サーバーに対して割り当て
が決まった Node の情報を通知します。

スケジューリングを決定する上で考慮が必要な要素としては、個別または複数のリソース
要求や、ハードウェア/ソフトウェアのポリシー制約、affinity や anti-affinity の設
定、データの局所性や、ワークロード間での干渉などが挙げられます。

## kube-scheduler によるスケジューリング{#kube-scheduler-implementation}

kube-scheduler は 2 ステップの操作によって Pod に割り当てる Node を選択します。

1. フィルタリング

2. スコアリング

_フィルタリング_ ステップでは、Pod に割り当て可能な Node のセットを探します。例
えば PodFitsResources フィルターは、Pod のリソース要求を満たすのに十分なリソース
をもつ Node がどれかをチェックします。このステップの後、候補の Node のリストは、
要求を満たす Node を含みます。  
たいてい、リストの要素は複数となります。もしこのリストが空の場合、その Pod はス
ケジュール可能な状態とはなりません。

_スコアリング_ ステップでは、Pod を割り当てるのに最も適した Node を選択するため
に、スケジューラーはリストの中の Node をランク付けします。  
スケジューラーは、フィルタリングによって選ばれた各 Node に対してスコアを付けます
。このスコアはアクティブなスコア付けのルールに基づいています。

最後に、kube-scheduler は最も高いランクの Node に対して Pod を割り当てます。もし
同一のスコアの Node が複数ある場合は、kube-scheduler がランダムに 1 つ選択します
。

### デフォルトのポリシーについて

kube-scheduler は、デフォルトで用意されているスケジューリングポリシーのセットを
持っています。

### フィルタリング

- `PodFitsHostPorts`: Node に、Pod が要求するポートが利用可能かどうかをチェック
  します。

- `PodFitsHost`: Pod がそのホスト名において特定の Node を指定しているかをチェッ
  クします。

- `PodFitsResources`: Node に、Pod が要求するリソース(例: CPU とメモリー)が利用
  可能かどうかをチェックします。

- `PodMatchNodeSelector`: Pod の NodeSelector が、Node のラベルにマッチするかど
  うかをチェックします。

- `NoVolumeZoneConflict`: Pod が要求する Volume が Node 上で利用可能かを、障害が
  発生しているゾーンを考慮して評価します。

- `NoDiskConflict`: Node の Volume が Pod の要求を満たし、すでにマウントされてい
  るかどうかを評価します。

- `MaxCSIVolumeCount`: CSI Volume をいくつ割り当てるべきか決定し、それが設定され
  た上限を超えるかどうかを評価します。

- `CheckNodeMemoryPressure`: もし Node がメモリーの容量が逼迫している場合、また
  設定された例外がない場合はその Pod はその Node にスケジュールされません。

- `CheckNodePIDPressure`: もし Node のプロセス ID が枯渇しそうになっていた場合や
  、設定された例外がない場合はその Pod はその Node にスケジュールされません。

- `CheckNodeDiskPressure`: もし Node のストレージが逼迫している場合(ファイルシス
  テムの残り容量がほぼない場合)や、設定された例外がない場合はその Pod はその
  Node にスケジュールされません。

- `CheckNodeCondition`: Node は、ファイルシステムの空き容量が完全になくなった場
  合、ネットワークが利用不可な場合、kubelet が Pod を稼働させる準備をできていな
  い場合などに、その状況を通知できます。Node がこの状況下かつ設定された例外がな
  い場合、Pod は該当の Node にスケジュールされません。

- `PodToleratesNodeTaints`: Pod の Toleration が Node の Taint を許容できるかチ
  ェックします。

- `CheckVolumeBinding`: Pod が要求する Volume の要求を満たすか評価します。これは
  PersistentVolumeClaim がバインドされているかに関わらず適用されます。

### スコアリング

- `SelectorSpreadPriority`:　同一の Service、StatefulSet や、ReplicaSet に属する
  Pod を複数のホストをまたいで稼働させます。

- `InterPodAffinityPriority`: weightedPodAffinityTerm の要素をイテレートして合計
  を計算したり、もし一致する PodAffinityTerm が Node に適合している場合は、"重み
  "を合計値に足したりします。:最も高い合計値を持つ Node(複数もあり)が候補となり
  ます。

- `LeastRequestedPriority`: 要求されたリソースがより低い Node を優先するものです
  。言い換えると、Node に多くの Pod が稼働しているほど、Pod が使用するリソースが
  多くなり、その要求量が低い Node が選択されます。

- `MostRequestedPriority`: 要求されたリソースがより多い Node を優先するものです
  。このポリシーは、ワークロードの全体セットを実行するために必要な最小数の Node
  に対して、スケジュールされた Pod を適合させます。

- `RequestedToCapacityRatioPriority`: デフォルトのリソーススコアリング関数を使用
  して、requestedToCapacity ベースの ResourceAllocationPriority を作成します。

- `BalancedResourceAllocation`: バランスのとれたリソース使用量になるように Node
  を選択します。

- `NodePreferAvoidPodsPriority`: Node
  の`scheduler.alpha.kubernetes.io/preferAvoidPods`というアノテーションに基づい
  て Node の優先順位づけをします。この設定により、2 つの異なる Pod が同じ Node
  上で実行しないことを示唆できます。

- `NodeAffinityPriority`: "PreferredDuringSchedulingIgnoredDuringExecution"の値
  によって示された Node Affinity のスケジューリング性向に基づいて Node の優先順
  位づけを行います。詳細
  は[Node への Pod の割り当て](https://kubernetes.io/ja/docs/concepts/configuration/assign-pod-node/)に
  て確認できます。

- `TaintTolerationPriority`: Node 上における許容できない Taints の数に基づいて、
  全ての Node の優先順位リストを準備します。このポリシーでは優先順位リストを考慮
  して Node のランクを調整します。

- `ImageLocalityPriority`: すでに Pod に対するコンテナイメージをローカルにキャッ
  シュしている Node を優先します。

- `ServiceSpreadingPriority`: このポリシーの目的は、特定の Service に対するバッ
  クエンドの Pod が、それぞれ異なる Node で実行されるようにすることです。このポ
  リシーでは Service のバックエンドの Pod が既に実行されていない Node 上にスケジ
  ュールするように優先します。これによる結果として、Service は単体の Node 障害に
  対してより耐障害性が高まります。

- `CalculateAntiAffinityPriorityMap`: このポリシー
  は[Pod の Anti-Affinity](https://kubernetes.io/ja/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity)の
  実装に役立ちます。

- `EqualPriorityMap`: 全ての Node に対して等しい重みを与えます。

{{% /capture %}} {{% capture whatsnext %}}

- [スケジューラーのパフォーマンスチューニング](/docs/concepts/scheduling/scheduler-perf-tuning/)を
  参照してください。
- kube-scheduler
  の[リファレンスドキュメント](/docs/reference/command-line-tools-reference/kube-scheduler/)を
  参照してください。
- [複数のスケジューラーの設定](https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/)に
  ついて学んでください。 {{% /capture %}}
