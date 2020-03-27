---
title: PodとReplicationControllerのデバッグ
content_template: templates/task
---

{{% capture overview %}}

このページでは、Pod と ReplicationController をデバッグする方法を説明します。

{{% /capture %}}

{{% capture prerequisites %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

- [Pod](/ja/docs/concepts/workloads/pods/pod/)と[Pod のライフサイクル](/ja/docs/concepts/workloads/pods/pod-lifecycle/)の
  基本を理解している必要があります。

{{% /capture %}}

{{% capture steps %}}

## Pod のデバッグ

Pod のデバッグの最初のステップは、Pod を調べることです。次のコマンドで、Pod の現
在の状態と最近のイベントを確認して下さい。

```shell
kubectl describe pods ${POD_NAME}
```

Pod 内のコンテナの状態を確認します。コンテナはすべて`Running`状態ですか？最近再
起動はしましたか？

Pod の状態に応じてデバッグを続けます。

### Pod が Pending 状態にとどまっている

Pod が`Pending`状態でスタックしている場合、ノードにスケジュールできていないこと
を意味します。一般的に、これは、何らかのタイプのリソースが不足しており、それによ
ってスケジューリングを妨げられているためです。上述の`kubectl describe...`コマン
ドの出力を確認してください。 Pod をスケジュールできない理由に関するスケジューラ
ーからのメッセージがあるはずです。理由としては以下のようなものがあります。

#### リソースが不十分

クラスター内の CPU またはメモリーの供給を使い果たした可能性があります。この場合
、いくつかのことを試すことができます。

- クラスター
  に[ノードを追加します](/docs/admin/cluster-management/#resizing-a-cluster)。

- [不要な Pod を終了](/docs/user-guide/pods/single-container/#deleting_a_pod)し
  て、 `Pending`状態の Pod のための空きリソースを作ります。

- Pod がノードよりも大きくないことを確認します。例えば、すべてのノードのキャパシ
  ティーが`cpu: 1`の場合、`cpu: 1.1`を要求する Pod は決してスケジュールされませ
  ん。

  `kubectl get nodes -o <format>`コマンドでノードのキャパシティーを確認できます
  。 必要な情報を抽出するコマンドラインの例を以下に示します。

  ```shell
  kubectl get nodes -o yaml | egrep '\sname:|cpu:|memory:'
  kubectl get nodes -o json | jq '.items[] | {name: .metadata.name, cap: .status.capacity}'
  ```

  [リソースクォータ](/docs/concepts/policy/resource-quotas/)機能では、消費できる
  リソースの合計量を制限するように構成できます。 Namespace と組み合わせて使用す
  ると、1 つのチームがすべてのリソースを占有することを防ぐことができます。

#### hostPort の使用

Pod を`hostPort`にバインドすると、Pod をスケジュールできる場所の数が制限されます
。ほとんどの場合、`hostPort`は不要です。Service オブジェクトを使用して Pod を公
開してください。どうしても`hostPort`が必要な場合は、コンテナクラスター内のノード
と同じ数の Pod のみをスケジュールできます。

### Pod が Waiting 状態にとどまっている

Pod が`Waiting`状態でスタックしている場合、Pod はワーカーノードにスケジュールさ
れていますが、そのマシンでは実行できない状態です。この場合も
、`kubectl describe ...`の情報が参考になるはずです。 Pod が`Waiting`状態となる最
も一般的な原因は、イメージをプルできないことです。確認すべき事項が 3 つあります
。

- イメージの名前が正しいことを確認して下さい。
- イメージはリポジトリーにプッシュしましたか？
- マシンで手動で`docker pull <image>`を実行し、イメージをプルできるかどうかを確
  認して下さい。

### Pod がクラッシュする、あるいは Unhealthy 状態

最初に、現在のコンテナのログを確認して下さい。

```shell
kubectl logs ${POD_NAME} ${CONTAINER_NAME}
```

以前にコンテナがクラッシュした場合、次のコマンドで以前のコンテナのクラッシュログ
にアクセスできます。

```shell
kubectl logs --previous ${POD_NAME} ${CONTAINER_NAME}
```

別の方法として、`exec`を使用してそのコンテナ内でコマンドを実行できます。

```shell
kubectl exec ${POD_NAME} -c ${CONTAINER_NAME} -- ${CMD} ${ARG1} ${ARG2} ... ${ARGN}
```

{{< note >}} `-c ${CONTAINER_NAME}`はオプションです。単一のコンテナのみを含む
Pod の場合は省略できます。 {{< /note >}}

例えば、実行中の Cassandra Pod のログを確認するには、次のコマンドを実行します。

```shell
kubectl exec cassandra -- cat /var/log/cassandra/system.log
```

これらのアプローチがいずれも機能しない場合、Pod が実行されているホストマシンを見
つけて、そのホストに SSH 接続することができます。

## ReplicationController のデバッグ

ReplicationController はかなり明快です。Pod を作成できるか、できないかのどちらか
です。 Pod を作成できない場合は、[上述の手順](#Podのデバッグ)を参照して Pod をデ
バッグしてください。

`kubectl describe rc ${CONTROLLER_NAME}`を使用して、レプリケーションコントローラ
ーに関連するイベントを調べることもできます。

{{% /capture %}}
