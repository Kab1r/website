---
title: Podのライフサイクル
content_template: templates/concept
weight: 30
---

{{% capture overview %}}

このページでは Pod のライフサイクルについて説明します。

{{% /capture %}}

{{% capture body %}}

## Pod の Phase

Pod の`status`項目
は[PodStatus](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podstatus-v1-core)
オブジェクトで、それは`phase`のフィールドがあります。

Pod のフェーズは、その Pod がライフサイクルのどの状態にあるかを、簡単かつ高レベ
ルにまとめたものです。このフェーズはコンテナや Pod の状態を包括的にまとめること
を目的としたものではなく、また包括的なステートマシンでもありません。

Pod の各フェーズの値と意味は厳重に守られています。ここに記載されているもの以外
に`phase`の値は存在しないと思ってください。

これらが`phase`の取りうる値です。

| 値          | 概要                                                                                                                                                                                                                                                       |
| :---------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Pending`   | Pod が Kubernetes システムによって承認されましたが、1 つ以上のコンテナイメージが作成されていません。これには、スケジュールされるまでの時間と、ネットワーク経由でイメージをダウンロードするための時間などが含まれます。これには時間がかかることがあります。 |
| `Running`   | Pod が Node にバインドされ、すべてのコンテナが作成されました。少なくとも 1 つのコンテナがまだ実行されているか、開始または再起動中です。                                                                                                                    |
| `Succeeded` | Pod 内のすべてのコンテナが正常に終了し、再起動されません。                                                                                                                                                                                                 |
| `Failed`    | Pod 内のすべてのコンテナが終了し、少なくとも 1 つのコンテナが異常終了しました。つまり、コンテナはゼロ以外のステータスで終了したか、システムによって終了されました。                                                                                        |
| `Unknown`   | 何らかの理由により、通常は Pod のホストとの通信にエラーが発生したために、Pod の状態を取得できませんでした。                                                                                                                                                |

## Pod の conditions

Pod には PodStatus があります。それは Pod が成功したかどうかの情報を持
つ[PodConditions](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podcondition-v1-core)
の配列です。 PodCondition 配列の各要素には、次の 6 つのフィールドがあります。

- `lastProbeTime` は、Pod Condition が最後に確認されたときのタイムスタンプが表示
  されます。

- `lastTransitionTime` は、最後に Pod のステータスの遷移があった際のタイムスタン
  プが表示されます。

- `message` は、ステータスの遷移に関する詳細を示す人間向けのメッセージです。

- `reason` は、最後の状態遷移の理由を示す、一意のキャメルケースでの単語です。

- `status` は`True`と`False`、`Unknown`のうちのどれかです。

- `type` 次の値を取る文字列です。

  - `PodScheduled`: Pod が Node にスケジュールされました。
  - `Ready`: Pod はリクエストを処理でき、一致するすべてのサービスの負荷分散プー
    ルに追加されます。
  - `Initialized`: すべて
    の[init containers](/docs/concepts/workloads/pods/init-containers)が正常に実
    行されました。
  - `Unschedulable`: リソースの枯渇やその他の理由で、Pod がスケジュールできない
    状態です。
  - `ContainersReady`: Pod 内のすべてのコンテナが準備できた状態です。

## コンテナの Probe

[Probe](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#probe-v1-core)
は [kubelet](/docs/admin/kubelet/) により定期的に実行されるコンテナの診断です。
診断を行うために、kubelet はコンテナに実装された
[ハンドラー](https://godoc.org/k8s.io/kubernetes/pkg/api/v1#Handler)を呼びます。
Handler には次の 3 つの種類があります:

- [ExecAction](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#execaction-v1-core):
  コンテナ内で特定のコマンドを実行します。コマンドがステータス 0 で終了した場合
  に診断を成功と見まします。

- [TCPSocketAction](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#tcpsocketaction-v1-core):
  コンテナの IP の特定のポートに TCP チェックを行います。そのポートが空いていれ
  ば診断を成功とみなします。

- [HTTPGetAction](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#httpgetaction-v1-core):
  コンテナの IP の特定のポートとパスに対して、HTTP GET のリクエストを送信します
  。レスポンスのステータスコードが 200 以上 400 未満の際に診断を成功とみなします
  。

各 Probe 次の 3 つのうちの一つの結果を持ちます:

- Success: コンテナの診断が成功しました。
- Failure: コンテナの診断が失敗しました。
- Unknown: コンテナの診断が失敗し、取れるアクションがありません。

Kubelet は 2 種類の Probe を実行中のコンテナで行い、また反応することができます:

- `livenessProbe`: コンテナが動いているかを示します。 livenessProbe に失敗すると
  、kubelet はコンテナを殺します、そしてコンテナ
  は[restart policy](#restart-policy)に従います。 コンテナに livenessProbe が設
  定されていない場合、デフォルトの状態は`Success`です。

- `readinessProbe`: コンテナが Service のリクエストを受けることができるかを示し
  ます。 readinessProbe に失敗すると、エンドポイントコントローラーにより
  、Service からその Pod の IP アドレスが削除されます。 initial delay 前のデフォ
  ルトの readinessProbe の初期値は`Failure`です。 コンテナに readinessProbe が設
  定されていない場合、デフォルトの状態は`Success`です。

### livenessProbe と readinessProbe をいつ使うべきか?

コンテナ自体に問題が発生した場合や状態が悪くなった際にクラッシュすることができれ
ば livenessProbe は不要です。この場合 kubelet が自動で Pod の`restartPolicy`に基
づいたアクションを実行します。

Probe に失敗したときにコンテナを殺したり再起動させたりするには、 livenessProbe
を設定し`restartPolicy`を Always または OnFailure にします。

Probe が成功したときにのみ Pod にトラフィックを送信したい場合は、readinessProbe
を指定します。この場合 readinessProbe は livenessProbe と同じになる可能性があり
ますが、 readinessProbe が存在するということは、Pod がトラフィックを受けずに開始
され、Probe 成功が開始した後でトラフィックを受け始めることになります。コンテナが
起動時に大きなデータ、構成ファイル、またはマイグレーションを読み込む必要がある場
合は、readinessProbe を指定します。

コンテナがメンテナンスのために停止できるようにするには、 livenessProbe とは異な
る、特定のエンドポイントを確認する readinessProbe を指定することができます。

Pod が削除されたときにリクエストを来ないようにするためには必ずしも
readinessProbe が必要というわけではありません。 Pod の削除時には readinessProbe
が存在するかどうかに関係なく Pod は自動的に自身を unhealthy にします。 Pod 内の
コンテナが停止するのを待つ間 Pod は unhealthy のままです。

livenessProbe または readinessProbe を設定する方法の詳細については、
[Configure Liveness and Readiness Probes](/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/)を
参照してください

## Pod とコンテナのステータス

Pod と Container のステータスについての詳細の情報は、それぞ
れ[PodStatus](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podstatus-v1-core)
と
[ContainerStatus](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#containerstatus-v1-core)
を参照してください。 Pod のステータスとして報告される情報は、現在
の[ContainerState](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#containerstatus-v1-core)
に依存しています。

## コンテナのステータス

Pod がスケジューラによって Node に割り当てられると、 kubelet はコンテナのランタ
イムを使用してコンテナの作成を開始します。コンテナの状態は Waiting、Running また
は Terminated の 3 ついずれかです。コンテナの状態を確認するに
は`kubectl describe pod [POD_NAME]`のコマンドを使用します。 Pod 内のコンテナごと
に State の項目として表示されます。

- `Waiting`: コンテナのデフォルトの状態。コンテナが Running または Terminated の
  いずれの状態でもない場合コンテナは Waiting の状態になります。Waiting 状態のコ
  ンテナは引き続きイメージを取得したり Secrets を適用したりするなど必要な操作を
  実行します。この状態に加えてメッセージに関する情報と状態に関する理由が表示され
  ます。

  ```yaml
  ...
    State:          Waiting
     Reason:       ErrImagePull
  ...
  ```

- `Running`: コンテナが問題なく実行されていることを示します。コンテナが Running
  に入ると`postStart`フック（もしあれば）が実行されます。この状態にはコンテナが
  実行中状態に入った時刻も表示されます。

  ```yaml
  ...
     State:          Running
      Started:      Wed, 30 Jan 2019 16:46:38 +0530
  ...
  ```

- `Terminated`: コンテナの実行が完了しコンテナの実行が停止したことを示します。コ
  ンテナは実行が正常に完了したときまたは何らかの理由で失敗したときにこの状態にな
  ります。いずれにせよ理由と終了コード、コンテナの開始時刻と終了時刻が表示されま
  す。コンテナが Terminated に入る前に`preStop`フックがあればあれば実行されます
  。

  ```yaml
  ...
     State:          Terminated
       Reason:       Completed
       Exit Code:    0
       Started:      Wed, 30 Jan 2019 11:45:26 +0530
       Finished:     Wed, 30 Jan 2019 11:45:26 +0530
   ...
  ```

## PodReadinessGate

{{< feature-state for_k8s_version="v1.14" state="stable" >}}

追加のフィードバックやシグナルを`PodStatus`に注入できるようにして Pod の
Readiness に拡張性を持たせるため、 Kubernetes 1.11 で
は[Pod ready++](https://github.com/kubernetes/enhancements/blob/master/keps/sig-network/0007-pod-ready%2B%2B.md)と
いう機能が導入されました。 `PodSpec`の新しいフィールド`ReadinessGate`を使用して
、Pod の Rediness を評価する追加の状態を指定できます。 Kubernetes が Pod の
status.conditions フィールドでそのような状態を発見できない場合、ステータスはデフ
ォルトで`False`になります。以下はその例です。

```yaml
Kind: Pod
---
spec:
  readinessGates:
    - conditionType: "www.example.com/feature-1"
status:
  conditions:
    - type: Ready # これはビルトインのPodCondition
      status: "False"
      lastProbeTime: null
      lastTransitionTime: 2018-01-01T00:00:00Z
    - type: "www.example.com/feature-1" # 追加のPodCondition
      status: "False"
      lastProbeTime: null
      lastTransitionTime: 2018-01-01T00:00:00Z
  containerStatuses:
    - containerID: docker://abcd...
      ready: true
```

新しい Pod Condition は、Kubernetes
の[label key format](/docs/concepts/overview/working-with-objects/labels/#syntax-and-character-set)に
準拠している必要があります。 `kubectl patch`コマンドはオブジェクトステータスのパ
ッチ適用をまだサポートしていないので、新しい Pod Condition
は[KubeClient libraries](/docs/reference/using-api/client-libraries/)のどれかを
使用する必要があります。

新しい Pod Condition が導入されると Pod は次の両方の条件に当てはまる場合**の
み**準備できていると評価されます:

- Pod 内のすべてのコンテナが準備完了している。
- `ReadinessGates`で指定された条件が全て`True`である。

Pod の Readiness の評価へのこの変更を容易にするために、新しい Pod Condition であ
る`ContainersReady`が導入され、古い Pod の`Ready`条件を取得します。

K8s 1.1 では Alpha 機能のため"Pod Ready++" 機能は`PodReadinessGates`
[feature gate](/docs/reference/command-line-tools-reference/feature-gates/)にて
明示的に指定する必要があります。

K8s 1.12 ではこの機能はデフォルトで有効になっています。

## RestartPolicy

PodSpec には、Always、OnFailure、または Never のいずれかの値を持
つ`restartPolicy`フィールドがあります。デフォルト値は Always です
。`restartPolicy`は、Pod 内のすべてのコンテナに適用されます。 `restartPolicy`は
、同じ Node 上の kubelet によるコンテナの再起動のみを参照します。 kubelet によっ
て再起動される終了したコンテナは、5 分後にキャップされた指数バックオフ遅延（10
秒、20 秒、40 秒...）で再起動され、10 分間の実行後にリセットされます
。[Pods document](/docs/user-guide/pods/#durability-of-pods-or-lack-thereof)に書
かれているように、一度 Node にバインドされると Pod は別のポートにバインドされ直
すことはありません。

## Pod のライフタイム

一般に Pod は人間またはコントローラーが明示的に削除するまで存在します。コントロ
ールプレーンは終了状態の Pod(Succeeded または Failed の`phase`を持つ)の数が設定
された閾値（kube-controller-manager 内の`terminated-pod-gc-threshold`によって定
義される）を超えたとき、それらの Pod を削除します。これは Pod が作成されて時間と
ともに終了するため、リソースリークを避けます。

次の 3 種類のコントローラがあります。

- バッチ計算などのように終了が予想される Pod に対しては
  、[Job](/docs/concepts/jobs/run-to-completion-finite-workloads/)を使用します。
  Job は`restartPolicy`が OnFailure または Never になる Pod に対してのみ適切です
  。

- 停止することを期待しない Pod（たとえば Web サーバーなど）には
  、[ReplicationController](/docs/concepts/workloads/controllers/replicationcontroller/)、[ReplicaSet](/ja/docs/concepts/workloads/controllers/replicaset/)、
  または[Deployment](/ja/docs/concepts/workloads/controllers/deployment/)を使用
  します。ReplicationController は`restartPolicy`が Always の Pod に対してのみ適
  切です。

- マシン固有のシステムサービスを提供するため、マシンごとに 1 つずつ実行する必要
  がある Pod に
  は[DaemonSet](/ja/docs/concepts/workloads/controllers/daemonset/)を使用します
  。

3 種類のコントローラにはすべて PodTemplate が含まれます。 Pod を自分で直接作成す
るのではなく適切なコントローラを作成して Pod を作成させることをおすすめします。
これは Pod 単独ではマシンの障害に対して回復力がないためです。コントローラにはこ
の機能があります。

Node が停止したりクラスタの他の Node から切断された場合、 Kubernetes は失われた
ノード上のすべての Pod の`phase`を Failed に設定するためのポリシーを適用します。

## 例

### 高度な liveness probe の例

Liveness Probe は kubelet によって実行されるため、すべてのリクエストは kubelet
ネットワークの namespace で行われます。

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
    - args:
        - /server
      image: k8s.gcr.io/liveness
      livenessProbe:
        httpGet:
          # "host" が定義されていない場合、"PodIP"が使用されます
          # host: my-host
          # "scheme"が定義されていない場合、HTTPスキームが使用されます。"HTTP"と"HTTPS"のみ
          # scheme: HTTPS
          path: /healthz
          port: 8080
          httpHeaders:
            - name: X-Custom-Header
              value: Awesome
        initialDelaySeconds: 15
        timeoutSeconds: 1
      name: liveness
```

### states の例

- Pod が実行中でその Pod には 1 つのコンテナがあります。コンテナは正常終了しまし
  た。

  - 完了のイベントを記録します。
  - `restartPolicy`が、
    - Always: コンテナを再起動します。Pod の`phase`は Running のままです。
    - OnFailure: Pod の`phase`は Succeeded になります。
    - Never: Pod の`phase`は Succeeded になります。

- Pod が実行中でその Pod には 1 つのコンテナがあります。コンテナは失敗終了しまし
  た。

  - 失敗イベントを記録します。
  - `restartPolicy`が、
    - Always: コンテナを再起動します。Pod の`phase`は Running のままです。
    - OnFailure: コンテナを再起動します。Pod の`phase`は Running のままです。
    - Never: Pod の`phase`は Failed になります。

- Pod が実行中で、その中には 2 つのコンテナがあります。コンテナ 1 は失敗終了しま
  した。

  - 失敗イベントを記録します。
  - `restartPolicy`が、
    - Always: コンテナを再起動します。Pod の`phase`は Running のままです。
    - OnFailure: コンテナを再起動します。Pod の`phase`は Running のままです。
    - Never: コンテナを再起動しません。Pod の`phase`は Running のままです。
  - コンテナ 1 が死んでいてコンテナ 2 は動いている場合
    - 失敗イベントを記録します。
    - `restartPolicy`が、
      - Always: コンテナを再起動します。Pod の`phase`は Running のままです。
      - OnFailure: コンテナを再起動します。Pod の`phase`は Running のままです。
      - Never: Pod の`phase`は Failed になります。

- Pod が実行中でその Pod には 1 つのコンテナがあります。コンテナはメモリーを使い
  果たしました。

  - コンテナは失敗で終了します。
  - OOM イベントを記録します。
  - `restartPolicy`が、
    - Always: コンテナを再起動します。Pod の`phase`は Running のままです。
    - OnFailure: コンテナを再起動します。Pod の`phase`は Running のままです。
    - Never: 失敗イベントを記録します。Pod の`phase`は Failed になります。

- Pod が実行中ですがディスクは死んでいます。

  - すべてのコンテナを殺します。
  - 適切なイベントを記録します。
  - Pod の`phase`は Failed になります。
  - Pod がコントローラで作成されていた場合は、別の場所で再作成されます。

- Pod が実行中ですが Node が切り離されました。
  - Node コントローラがタイムアウトを待ちます。
  - Node コントローラが Pod の`phase`を Failed にします。
  - Pod がコントローラで作成されていた場合は、別の場所で再作成されます。

{{% /capture %}}

{{% capture whatsnext %}}

- [attaching handlers to Container lifecycle events](/docs/tasks/configure-pod-container/attach-handler-lifecycle-event/)の
  ハンズオンをやってみる

- [configuring liveness and readiness probes](/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/)の
  ハンズオンをやってみる

- [Container lifecycle hooks](/docs/concepts/containers/container-lifecycle-hooks/)に
  ついてもっと学ぶ

{{% /capture %}}
