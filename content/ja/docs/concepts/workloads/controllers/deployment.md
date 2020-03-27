---
title: Deployment
feature:
  title: 自動化されたロールアウトとロールバック
  description: >
    Kubernetesはアプリケーションや設定への変更を段階的に行い、アプリケーションの状態を監視しながら、全てのインスタンスが同時停止しないようにします。更新に問題が起きたとき、Kubernetesは変更のロールバックを行います。進化を続けるDeploymnetのエコシステムを活用してください。

content_template: templates/concept
weight: 30
---

{{% capture overview %}}

_Deployment_ コントローラー
は[Pod](/ja/docs/concepts/workloads/pods/pod/)と[ReplicaSet](/ja/docs/concepts/workloads/controllers/replicaset/)の
宣言的なアップデート機能を提供します。

ユーザーは Deployment において*理想的な状態* を定義し、Deployment コントローラー
は指定された頻度で現在の状態を理想的な状態に変更させます。ユーザーは Deployment
を定義して、新しい ReplicaSet を作成したり、既存の Deployment を削除して新しい
Deployment で全てのリソースを適用できます。

{{< note >}} Deployment によって作成された ReplicaSet を管理しないでください。ユ
ーザーのユースケースが下記の項目をカバーできていない場合はメインの Kubernetes リ
ポジトリーにイシューを作成することを検討してください。 {{< /note >}}

{{% /capture %}}

{{% capture body %}}

## ユースケース

下記の項目は Deployment の典型的なユースケースです。

- ReplicaSet をロールアウトするため
  に[Deployment の作成](#creating-a-deployment)を行う: ReplicaSet はバックグラウ
  ンドで Pod を作成します。Pod の作成が完了したかどうかは、ロールアウトのステー
  タスを確認してください。
- Deployment の PodTemplateSpec を更新することによ
  り[Pod の新しい状態を宣言する](#updating-a-deployment): 新しい ReplicaSet が作
  成され、Deployment は指定された頻度で古い ReplicaSet から新しい ReplicaSet へ
  の Pod の移行を管理します。新しい ReplicaSet は Deployment のリビジョンを更新
  します。
- Deployment の現在の状態が不安定な場合
  、[Deployment のロールバック](#rolling-back-a-deployment)をする: ロールバック
  による各更新作業は、Deployment のリビジョンを更新します。
- より多くの負荷をさばけるように
  、[Deployment をスケールアップ](#scaling-a-deployment)する
- PodTemplateSpec に対する複数の修正を適用するため
  に[Deployment を停止(Pause)し](#pausing-and-resuming-a-deployment)、それを再開
  して新しいロールアウトを開始する。
- 今後必要としない[古い ReplicaSet のクリーンアップ](#clean-up-policy)

## Deployment の作成 {#creating-a-deployment}

下記の内容は Deployment の例です。これは`nginx`Pod のレプリカを 3 つ持つ
ReplicaSet を作成します。

{{< codenew file="controllers/nginx-deployment.yaml" >}}

この例において、

- `nginx-deployment`という名前の Deployment が作成され、`.metadata.name`フィール
  ドで名前を指定します。
- Deployment は 3 つのレプリカ Pod を作成し、`replicas`フィールドによってレプリ
  カ数を指定します。
- `selector`フィールドは、Deployment が管理する Pod のラベルを定義します。このケ
  ースにおいて、ユーザーは Pod テンプレートにて定義されたラベル(`app: nginx`)を
  選択します。しかし、PodTemplate 自体がそのルールを満たす限り、さらに洗練された
  方法でセレクターを指定することができます。 {{< note >}} `matchLabels`フィール
  ドは、キーとバリューのペアのマップとなります。`matchLabels`マップにおいて
  、{key, value}というペアは、key というフィールドの値が"key"で、その演算子が
  "In"で、値の配列が"value"のみ含むような`matchExpressions`の要素と等しいです。
  `matchLabels`と`matchExpressions`の両方が設定された場合、条件に一致するには両
  方とも満たす必要があります。 {{< /note >}}
- `template`フィールドは、下記のサブフィールドを持ちます。:

  - Pod は`labels`フィールドによって指定された`app: nginx`というラベルがつけられ
    る
  - PodTemplate の仕様もしくは、`.template.spec`フィールドは、この Pod
    は`nginx`という名前のコンテナーを 1 つ稼働させ、それは`nginx`というさせ
    、[Docker Hub](https://hub.docker.com/)にある`nginx`のバージョン 1.14.2 を使
    うことを示します
  - 1 つのコンテナを作成し、`name`フィールドを使って`nginx`という名前をつけます

  上記の Deployment を作成するために、以下に示すステップにしたがってください。作
  成を始める前に、ユーザーの Kubernetes クラスターが稼働していることを確認してく
  ださい。

  1. 下記のコマンドを実行して Deployment を作成してください。

     {{< note >}} 実行したコマンドを`kubernetes.io/change-cause`というアノテーシ
     ョンに記録するために`--record`フラグを指定できます。これは将来的な問題の調
     査のために有効です。例えば、各 Deployment のリビジョンにおいて実行されたコ
     マンドを見るときに便利です。 {{< /note >}}

  ```shell
  kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
  ```

  2. Deployment が作成されたことを確認するために、`kubectl get deployment`を実行
     してください。Deployment がまだ作成中の場合、コマンドの実行結果は下記のとお
     りです。

  ```shell
  NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
  nginx-deployment   3         0         0            0           1s
  ```

  ユーザーのクラスターにおいて Deployment を調査するとき、下記のフィールドが出力
  されます。

      * `NAME` クラスター内のDeploymentの名前を表示する
      * `DESIRED` アプリケーションの理想的な_replicas_ の値を表示する: これはDeploymentを作成したときに定義したもので、これが_理想的な状態_ と呼ばれるものです。
      * `CURRENT` 現在稼働中のレプリカ数
      * `UP-TO-DATE` 理想的な状態にするために、アップデートが完了したレプリカ数
      * `AVAILABLE` ユーザーが利用可能なレプリカ数
      * `AGE` アプリケーションが稼働してからの時間

  上記の yaml の例だと、`.spec.replicas`フィールドの値によると、理想的なレプリカ
  数は 3 です。

  3. Deployment のロールアウトステータスを確認するために
     、`kubectl rollout status deployment.v1.apps/nginx-deployment`を実行してく
     ださい。コマンドの実行結果は下記のとおりです。

  ```shell
  Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
  deployment.apps/nginx-deployment successfully rolled out
  ```

  4. 数秒後、再度`kubectl get deployments`を実行してください。コマンドの実行結果
     は下記のとおりです。

  ```shell
  NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
  nginx-deployment   3         3         3            3           18s
  ```

  Deployment が 3 つ全てのレプリカを作成して、全てのレプリカが最新(Pod が最新の
  Pod テンプレートを含んでいる)になり、利用可能となっていることを確認してくださ
  い。

  5. Deployment によって作成された ReplicaSet (`rs`)を確認するに
     は`kubectl get rs`を実行してください。コマンドの実行結果は下記のとおりです
     。

  ```shell
  NAME                          DESIRED   CURRENT   READY   AGE
  nginx-deployment-75675f5897   3         3         3       18s
  ```

  ReplicaSet の名前は`[Deployment名]-[ランダム文字列]`という形式になることに注意
  してください。ランダム文字列はランダムに生成され、pod-template-hash をシードと
  して使用します。

  6. 各 Pod にラベルが自動的に付けられるのを確認するに
     は`kubectl get pods --show-labels`を実行してください。コマンドの実行結果は
     下記のとおりです。

  ```shell
  NAME                                READY     STATUS    RESTARTS   AGE       LABELS
  nginx-deployment-75675f5897-7ci7o   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
  nginx-deployment-75675f5897-kzszj   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
  nginx-deployment-75675f5897-qqcnn   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
  ```

  作成された ReplicaSet は`nginx`Pod を 3 つ作成することを保証します。

  {{< note >}} Deployment に対して適切なセレクターと Pod テンプレートのラベルを
  設定する必要があります(このケースでは`app: nginx`)。ラベルやセレクターを他のコ
  ントローラーと重複させないでください(他の Deployment や StatefulSet を含む
  )。Kubernetes はユーザがラベルを重複させることを止めないため、複数のコントロー
  ラーでセレクターの重複が発生すると、コントローラー間で衝突し予期せぬふるまいを
  することになります。 {{< /note >}}

### pod-template-hash ラベル

{{< note >}} このラベルを変更しないでください。 {{< /note >}}

`pod-template-hash`ラベルは Deployment コントローラーによって Deployment が作成
し適用した各 ReplicaSet に対して追加されます。

このラベルは Deployment が管理する ReplicaSet が重複しないことを保証します。この
ラベルは ReplicaSet の`PodTemplate`をハッシュ化することにより生成され、生成され
たハッシュ値はラベル値として ReplicaSet セレクター、Pod テンプレートラベル
、ReplicaSet が作成した全ての Pod に対して追加されます。

## Deployment の更新

{{< note >}} Deployment のロールアウトは、Deployment の Pod テンプレート(この場
合`.spec.template`)が変更された場合にのみトリガーされます。例えばテンプレートの
ラベルもしくはコンテナーイメージが更新された場合です。Deployment のスケールのよ
うな更新では、ロールアウトはトリガーされません。 {{< /note >}}

Deployment を更新するには下記のステップに従ってください。

1. nginx の Pod で、`nginx:1.14.2`イメージの代わりに`nginx:1.16.1`を使うように更
   新します。

   ```shell
   kubectl --record deployment.apps/nginx-deployment set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1
   ```

   実行結果は下記のとおりです。

   ```
   deployment.apps/nginx-deployment image updated
   ```

   また、Deployment を`編集`して
   、`.spec.template.spec.containers[0].image`を`nginx:1.14.2`か
   ら`nginx:1.16.1`に変更することができます。

   ```shell
   kubectl edit deployment.v1.apps/nginx-deployment
   ```

   実行結果は下記のとおりです。

   ```
   deployment.apps/nginx-deployment edited
   ```

2. ロールアウトのステータスを確認するには、下記のコマンドを実行してください。

   ```shell
   kubectl rollout status deployment.v1.apps/nginx-deployment
   ```

   実行結果は下記のとおりです。

   ```
   Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
   ```

   もしくは

   ```
   deployment.apps/nginx-deployment successfully rolled out
   ```

更新された Deployment のさらなる情報を取得するには、下記を確認してください。

- ロールアウトが成功したあと、`kubectl get deployments`を実行して Deployment を
  確認できます。 実行結果は下記のとおりです。

  ```
  NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
  nginx-deployment   3         3         3            3           36s
  ```

- Deployment が新しい ReplicaSet を作成して Pod を更新させたり、新しい
  ReplicaSet のレプリカを 3 にスケールアップさせたり、古い ReplicaSet のレプリカ
  を 0 にスケールダウンさせるのを確認するには`kubectl get rs`を実行してください
  。

  ```shell
  kubectl get rs
  ```

  実行結果は下記のとおりです。

  ```
  NAME                          DESIRED   CURRENT   READY   AGE
  nginx-deployment-1564180365   3         3         3       6s
  nginx-deployment-2035384211   0         0         0       36s
  ```

- `get pods`を実行させると、新しい Pod のみ確認できます。

  ```shell
  kubectl get pods
  ```

  実行結果は下記のとおりです。

  ```
  NAME                                READY     STATUS    RESTARTS   AGE
  nginx-deployment-1564180365-khku8   1/1       Running   0          14s
  nginx-deployment-1564180365-nacti   1/1       Running   0          14s
  nginx-deployment-1564180365-z9gth   1/1       Running   0          14s
  ```

  次に Pod を更新させたいときは、Deployment の Pod テンプレートを再度更新する�
  けです。

  Deployment は、Pod が更新されている間に特定の数の Pod のみ停止状態になることを
  保証します。デフォルトでは、目標とする Pod 数の少なくとも 25%が停止状態になる
  ことを保証します(25% max unavailable)。

  また、Deployment は Pod が更新されている間に、目標とする Pod 数を特定の数まで
  超えて Pod を稼働させることを保証します。デフォルトでは、目標とする Pod 数に対
  して最大でも 25%を超えて Pod を稼働させることを保証します(25% max surge)。

  例えば、上記で説明した Deployment の状態を注意深く見ると、最初に新しい Pod が
  作成され、次に古い Pod が削除されるのを確認できます。十分な数の新しい Pod が稼
  働するまでは、Deployment は古い Pod を削除しません。また十分な数の古い Pod が
  削除しない限り新しい Pod は作成されません。少なくとも 2 つの Pod が利用可能で
  、最大でもトータルで 4 つの Pod が利用可能になっていることを保証します。

- Deployment の詳細情報を取得します。
  ```shell
  kubectl describe deployments
  ```
  実行結果は下記のとおりです。
  ```
  Name:                   nginx-deployment
  Namespace:              default
  CreationTimestamp:      Thu, 30 Nov 2017 10:56:25 +0000
  Labels:                 app=nginx
  Annotations:            deployment.kubernetes.io/revision=2
  Selector:               app=nginx
  Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
  StrategyType:           RollingUpdate
  MinReadySeconds:        0
  RollingUpdateStrategy:  25% max unavailable, 25% max surge
  Pod Template:
    Labels:  app=nginx
     Containers:
      nginx:
        Image:        nginx:1.16.1
        Port:         80/TCP
        Environment:  <none>
        Mounts:       <none>
      Volumes:        <none>
    Conditions:
      Type           Status  Reason
      ----           ------  ------
      Available      True    MinimumReplicasAvailable
      Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   nginx-deployment-1564180365 (3/3 replicas created)
    Events:
      Type    Reason             Age   From                   Message
      ----    ------             ----  ----                   -------
      Normal  ScalingReplicaSet  2m    deployment-controller  Scaled up replica set nginx-deployment-2035384211 to 3
      Normal  ScalingReplicaSet  24s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 1
      Normal  ScalingReplicaSet  22s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 2
      Normal  ScalingReplicaSet  22s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 2
      Normal  ScalingReplicaSet  19s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 1
      Normal  ScalingReplicaSet  19s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 3
      Normal  ScalingReplicaSet  14s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 0
  ```
  最初に Deployment を作成した時、ReplicaSet(nginx-deployment-2035384211)を作成
  してすぐにレプリカ数を 3 にスケールするのを確認できます。Deployment を更新する
  と新しい ReplicaSet(nginx-deployment-1564180365)を作成してレプリカ数を 1 にス
  ケールアップし、古い ReplicaSeet を 2 にスケールダウンさせます。これは常に最低
  でも 2 つの Pod が利用可能で、かつ最大 4 つの Pod が作成されている状態にするた
  めです。Deployment は同じローリングアップ戦略に従って新しい ReplicaSet のスケ
  ールアップと古い ReplicaSet のスケールダウンを続けます。最終的に新しい
  ReplicaSet を 3 にスケールアップさせ、古い ReplicaSet を 0 にスケールダウンさ
  せます。

### ロールオーバー (リアルタイムでの複数の Pod の更新)

Deployment コントローラーにより、新しい Deployment が観測される度に ReplicaSet
が作成され、理想とするレプリカ数の Pod を作成します。Deployment が更新されると、
既存の ReplicaSet が管理する Pod のラベルが`.spec.selector`にマッチするが、テン
プレートが`.spec.template`にマッチしない場合はスケールダウンされます。最終的に、
新しい ReplicaSet は`.spec.replicas`の値にスケールアップされ、古い ReplicaSet は
0 にスケールダウンされます。

Deployment のロールアウトが進行中に Deployment を更新すると、Deployment は更新す
る毎に新しい ReplicaSet を作成してスケールアップさせ、以前にスケールアップした
ReplicaSet のロールオーバーを行います。Deployment は更新前の ReplicaSet を古い
ReplicaSet のリストに追加し、スケールダウンを開始します。

例えば、5 つのレプリカを持つ`nginx:1.14.2`の Deployment を作成し
、`nginx:1.14.2`の 3 つのレプリカが作成されているときに 5 つのレプリカを持
つ`nginx:1.16.1`に更新します。このケースでは Deployment は作成済み
の`nginx:1.14.2`の 3 つの Pod をすぐに削除し、`nginx:1.16.1`の Pod の作成を開始
します。`nginx:1.14.2`の 5 つのレプリカを全て作成するのを待つことはありません。

### ラベルセレクターの更新

通常、ラベルセレクターを更新することは推奨されません。事前にラベルセレクターの使
い方を計画しておきましょう。いかなる場合であっても更新が必要なときは十分に注意を
払い、変更時の影響範囲を把握しておきましょう。

{{< note >}} `apps/v1`API バージョンにおいて、Deployment のラベルセレクターは作
成後に不変となります。 {{< /note >}}

- セレクターの追加は、Deployment Spec のテンプレートラベルも新しいラベルで更新す
  る必要があります。そうでない場合はバリデーションエラーが返されます。この変更は
  重複がない更新となります。これは新しいセレクターは古いセレクターを持つ
  ReplicaSet と Pod を選択せず、結果として古い全ての ReplicaSet がみなし子状態に
  なり、新しい ReplicaSet を作成することを意味します。
- セレクターの更新により、セレクターキー内の既存の値が変更されます。これにより、
  セレクターの追加と同じふるまいをします。
- セレクターの削除により、Deployment のセレクターから存在している値を削除します
  。これは Pod テンプレートのラベルに関する変更を要求しません。既存の ReplicaSet
  はみなし子状態にならず、新しい ReplicaSet は作成されませんが、削除されたラベル
  は既存の Pod と ReplicaSet では残り続けます。

## Deployment のロールバック {#rolling-back-a-deployment}

Deployment のロールバックを行いたい場合があります。例えば、Deployment がクラッシ
ュ状態になりそれがループしたりする不安定なときです。デフォルトではユーザーがいつ
でもロールバックできるように Deployment の全てのロールアウト履歴がシステムに保持
されます(リビジョン履歴の上限は設定することで変更可能です)。

{{< note >}} Deployment のリビジョンは、Deployment のロールアウトがトリガーされ
た時に作成されます。これは Deployment の Pod テンプレート(`.spec.template`)が変
更されたときのみ新しいリビジョンが作成されることを意味します。Deployment のスケ
ーリングなど、他の種類の更新においては Deployment のリビジョンは作成されません。
これは手動もしくはオートスケーリングを同時に行うことができるようにするためです。
これは過去のリビジョンにロールバックするとき、Deployment の Pod テンプレートの箇
所のみロールバックされることを意味します。 {{< /note >}}

- `nginx:1.16.1`の代わりに`nginx:1.161`というイメージに更新して、Deployment の更
  新中にタイプミスをしたと仮定します。

  ```shell
  kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.161 --record=true
  ```

  実行結果は下記のとおりです。

  ```
  deployment.apps/nginx-deployment image updated
  ```

- このロールアウトはうまくいきません。ユーザーロールアウトのステータスを見ること
  でロールアウトがうまくいくか確認できます。

  ```shell
  kubectl rollout status deployment.v1.apps/nginx-deployment
  ```

  実行結果は下記のとおりです。

  ```
  Waiting for rollout to finish: 1 out of 3 new replicas have been updated...
  ```

- ロールアウトのステータスの確認は、Ctrl-C を押すことで停止できます。ロールアウ
  トがうまく行かないときは、[Deployment のステータス](#deployment-status)を読ん
  でさらなる情報を得てください。

- 古いレプリカ数(`nginx-deployment-1564180365` and
  `nginx-deployment-2035384211`)が 2 になっていることを確認でき、新しいレプリカ
  数(nginx-deployment-3066724191)は 1 になっています。

  ```shell
  kubectl get rs
  ```

  実行結果は下記のとおりです。

  ```
  NAME                          DESIRED   CURRENT   READY   AGE
  nginx-deployment-1564180365   3         3         3       25s
  nginx-deployment-2035384211   0         0         0       36s
  nginx-deployment-3066724191   1         1         0       6s
  ```

- 作成された Pod を確認していると、新しい ReplicaSet によって作成された 1 つの
  Pod はコンテナイメージの pull に失敗し続けているのがわかります。

  ```shell
  kubectl get pods
  ```

  実行結果は下記のとおりです。

  ```
  NAME                                READY     STATUS             RESTARTS   AGE
  nginx-deployment-1564180365-70iae   1/1       Running            0          25s
  nginx-deployment-1564180365-jbqqo   1/1       Running            0          25s
  nginx-deployment-1564180365-hysrc   1/1       Running            0          25s
  nginx-deployment-3066724191-08mng   0/1       ImagePullBackOff   0          6s
  ```

  {{< note >}} Deployment コントローラーは、この悪い状態のロールアウトを自動的に
  停止し、新しい ReplicaSet のスケールアップを止めます。これはユーザーが指定した
  ローリングアップデートに関するパラメータ(特に`maxUnavailable`)に依存します。デ
  フォルトでは Kubernetes がこの値を 25%に設定します。 {{< /note >}}

- Deployment の詳細情報を取得します。

  ```shell
  kubectl describe deployment
  ```

  実行結果は下記のとおりです。

  ```
  Name:           nginx-deployment
  Namespace:      default
  CreationTimestamp:  Tue, 15 Mar 2016 14:48:04 -0700
  Labels:         app=nginx
  Selector:       app=nginx
  Replicas:       3 desired | 1 updated | 4 total | 3 available | 1 unavailable
  StrategyType:       RollingUpdate
  MinReadySeconds:    0
  RollingUpdateStrategy:  25% max unavailable, 25% max surge
  Pod Template:
    Labels:  app=nginx
    Containers:
     nginx:
      Image:        nginx:1.161
      Port:         80/TCP
      Host Port:    0/TCP
      Environment:  <none>
      Mounts:       <none>
    Volumes:        <none>
  Conditions:
    Type           Status  Reason
    ----           ------  ------
    Available      True    MinimumReplicasAvailable
    Progressing    True    ReplicaSetUpdated
  OldReplicaSets:     nginx-deployment-1564180365 (3/3 replicas created)
  NewReplicaSet:      nginx-deployment-3066724191 (1/1 replicas created)
  Events:
    FirstSeen LastSeen    Count   From                    SubobjectPath   Type        Reason              Message
    --------- --------    -----   ----                    -------------   --------    ------              -------
    1m        1m          1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-2035384211 to 3
    22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 1
    22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 2
    22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 2
    21s       21s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 1
    21s       21s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 3
    13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 0
    13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-3066724191 to 1
  ```

  これを修正するために、Deployment を安定した状態の過去のリビジョンに更新する必
  要があります。

### Deployment のロールアウト履歴の確認

ロールアウトの履歴を確認するには、下記の手順に従って下さい。

1. 最初に、Deployment のリビジョンを確認します。

   ```shell
   kubectl rollout history deployment.v1.apps/nginx-deployment
   ```

   実行結果は下記のとおりです。

   ```
   deployments "nginx-deployment"
   REVISION    CHANGE-CAUSE
   1           kubectl apply --filename=https://k8s.io/examples/controllers/nginx-deployment.yaml --record=true
   2           kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1 --record=true
   3           kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.161 --record=true
   ```

   `CHANGE-CAUSE`はリビジョンの作成時に Deployment
   の`kubernetes.io/change-cause`アノテーションからリビジョンにコピーされます。
   下記の手段により`CHANGE-CAUSE`メッセージを指定できます。

   - `kubectl annotate deployment.v1.apps/nginx-deployment kubernetes.io/change-cause="image updated to 1.16.1"`の
     実行によりアノテーションを追加する。
   - リソースの変更時に`kubectl`コマンドの内容を記録するために`--record`フラグを
     追加する。
   - リソースのマニフェストを手動で編集する。

2. 各リビジョンの詳細を確認するためには下記のコマンドを実行してください。

   ```shell
   kubectl rollout history deployment.v1.apps/nginx-deployment --revision=2
   ```

   実行結果は下記のとおりです。

   ```
   deployments "nginx-deployment" revision 2
     Labels:       app=nginx
             pod-template-hash=1159050644
     Annotations:  kubernetes.io/change-cause=kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1 --record=true
     Containers:
      nginx:
       Image:      nginx:1.16.1
       Port:       80/TCP
        QoS Tier:
           cpu:      BestEffort
           memory:   BestEffort
       Environment Variables:      <none>
     No volumes.
   ```

### 過去のリビジョンにロールバックする {#rolling-back-to-a-previous-revision}

現在のリビジョンから過去のリビジョン(リビジョン番号 2)にロールバックさせるには、
下記の手順に従ってください。

1. 現在のリビジョンから過去のリビジョンにロールバックします。

   ```shell
   kubectl rollout undo deployment.v1.apps/nginx-deployment
   ```

   実行結果は下記のとおりです。

   ```
   deployment.apps/nginx-deployment
   ```

   その他に、`--to-revision`を指定することにより特定のリビジョンにロールバックで
   きます。

   ```shell
   kubectl rollout undo deployment.v1.apps/nginx-deployment --to-revision=2
   ```

   実行結果は下記のとおりです。

   ```
   deployment.apps/nginx-deployment
   ```

   ロールアウトに関連したコマンドのさらなる情報
   は[`kubectl rollout`](/docs/reference/generated/kubectl/kubectl-commands#rollout)を
   参照してください。

   Deployment が過去の安定したリビジョンにロールバックされました。Deployment コ
   ントローラーによって、リビジョン番号 2 にロールバックす
   る`DeploymentRollback`イベントが作成されたのを確認できます。

2. ロールバックが成功し、Deployment が正常に稼働していることを確認するために、下
   記のコマンドを実行してください。

   ```shell
   kubectl get deployment nginx-deployment
   ```

   実行結果は下記のとおりです。

   ```
   NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
   nginx-deployment   3         3         3            3           30m
   ```

3. Deployment の詳細情報を取得します。
   ```shell
   kubectl describe deployment nginx-deployment
   ```
   実行結果は下記のとおりです。
   ```
   Name:                   nginx-deployment
   Namespace:              default
   CreationTimestamp:      Sun, 02 Sep 2018 18:17:55 -0500
   Labels:                 app=nginx
   Annotations:            deployment.kubernetes.io/revision=4
                           kubernetes.io/change-cause=kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1 --record=true
   Selector:               app=nginx
   Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
   StrategyType:           RollingUpdate
   MinReadySeconds:        0
   RollingUpdateStrategy:  25% max unavailable, 25% max surge
   Pod Template:
     Labels:  app=nginx
     Containers:
      nginx:
       Image:        nginx:1.16.1
       Port:         80/TCP
       Host Port:    0/TCP
       Environment:  <none>
       Mounts:       <none>
     Volumes:        <none>
   Conditions:
     Type           Status  Reason
     ----           ------  ------
     Available      True    MinimumReplicasAvailable
     Progressing    True    NewReplicaSetAvailable
   OldReplicaSets:  <none>
   NewReplicaSet:   nginx-deployment-c4747d96c (3/3 replicas created)
   Events:
     Type    Reason              Age   From                   Message
     ----    ------              ----  ----                   -------
     Normal  ScalingReplicaSet   12m   deployment-controller  Scaled up replica set nginx-deployment-75675f5897 to 3
     Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 1
     Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 2
     Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 2
     Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 1
     Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 3
     Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 0
     Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-595696685f to 1
     Normal  DeploymentRollback  15s   deployment-controller  Rolled back deployment "nginx-deployment" to revision 2
     Normal  ScalingReplicaSet   15s   deployment-controller  Scaled down replica set nginx-deployment-595696685f to 0
   ```

## Deployment のスケーリング {#scaling-a-deployment}

下記のコマンドを実行させて Deployment をスケールできます。

```shell
kubectl scale deployment.v1.apps/nginx-deployment --replicas=10
```

実行結果は下記のとおりです。

```
deployment.apps/nginx-deployment scaled
```

クラスター内
で[水平 Pod オートスケーラー](/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)が
有効になっていると仮定します。ここで Deployment のオートスケーラーを設定し、稼働
している Pod の CPU 使用量に基づいて、ユーザーが稼働させたい Pod のレプリカ数の
最小値と最大値を設定できます。

```shell
kubectl autoscale deployment.v1.apps/nginx-deployment --min=10 --max=15 --cpu-percent=80
```

実行結果は下記のとおりです。

```
deployment.apps/nginx-deployment scaled
```

### 比例スケーリング

Deployment のローリングアップデートは、同時に複数のバージョンのアプリケーション
の稼働をサポートします。ユーザーやオートスケーラーがロールアウト中(更新中もしく
は一時停止中)の Deployment のローリングアップデートを行うとき、Deployment コント
ローラーはリスクを削減するために既存のアクティブな ReplicaSet のレプリカのバラン
シングを行います。これを*比例スケーリング* と呼びます。

レプリカ数が 10、[maxSurge](#max-surge)=3、[maxUnavailable](#max-unavailable)=2
である Deployment が稼働している例です。

- Deployment 内で 10 のレプリカが稼働していることを確認します。

  ```shell
  kubectl get deploy
  ```

  実行結果は下記のとおりです。

  ```
  NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
  nginx-deployment     10        10        10           10          50s
  ```

- クラスター内で、解決できない新しいイメージに更新します。
- You update to a new image which happens to be unresolvable from inside the
  cluster.

  ```shell
  kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:sometag
  ```

  実行結果は下記のとおりです。 The output is similar to this:

  ```
  deployment.apps/nginx-deployment image updated
  ```

- イメージの更新は新しい ReplicaSet nginx-deployment-1989198191 へのロールアウト
  を開始させます。しかしロールアウトは、上述した`maxUnavailable`の要求によりブロ
  ックされます。ここでロールアウトのステータスを確認します。

  ```shell
  kubectl get rs
  ```

      実行結果は下記のとおりです。

  ```
  NAME                          DESIRED   CURRENT   READY     AGE
  nginx-deployment-1989198191   5         5         0         9s
  nginx-deployment-618515232    8         8         8         1m
  ```

- 次に Deployment のスケーリングをするための新しい要求が発生します。オートスケー
  ラーは Deployment のレプリカ数を 15 に増やします。Deployment コントローラーは
  新しい 5 つのレプリカをどこに追加するか決める必要がでてきます。比例スケーリン
  グを使用していない場合、5 つのレプリカは全て新しい ReplicaSet に追加されます。
  比例スケーリングでは、追加されるレプリカは全ての ReplicaSet に分散されます。比
  例割合が大きいものはレプリカ数の大きい ReplicaSet となり、比例割合が低いときは
  レプリカ数の小さい ReplicaSet となります。残っているレプリカはもっとも大きいレ
  プリカ数を持つ ReplicaSet に追加されます。レプリカ数が 0 の ReplicaSet はスケ
  ールアップされません。

上記の例では、3 つのレプリカが古い ReplicaSet に追加され、2 つのレプリカが新しい
ReplicaSet に追加されました。ロールアウトの処理では、新しいレプリカ数の Pod が正
常になったと仮定すると、最終的に新しい ReplicaSet に全てのレプリカを移動させます
。これを確認するためには下記のコマンドを実行して下さい。

    ```shell
    kubectl get deploy
    ```
    実行結果は下記のとおりです。
    ```
    NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment     15        18        7            8           7m
    ```

ロールアウトのステータスでレプリカがどのように各 ReplicaSet に追加されるか確認で
きます。 `shell kubectl get rs` 実行結果は下記のとおりです。
`NAME DESIRED CURRENT READY AGE nginx-deployment-1989198191 7 7 0 7m nginx-deployment-618515232 11 11 11 7m`

## Deployment 更新の一時停止と再開 {#pausing-and-resuming-a-deployment}

ユーザーは 1 つ以上の更新処理をトリガーする前に更新の一時停止と再開ができます。
これにより、不必要なロールアウトを実行することなく一時停止と再開を行う間に複数の
修正を反映できます。

- 例えば、作成直後の Deployment を考えます。
  Deployment の詳細情報を確認します。

  ```shell
  kubectl get deploy
  ```

  実行結果は下記のとおりです。

  ```
  NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
  nginx     3         3         3            3           1m
  ```

  ロールアウトのステータスを確認します。

  ```shell
  kubectl get rs
  ```

  実行結果は下記のとおりです。

  ```
  NAME               DESIRED   CURRENT   READY     AGE
  nginx-2142116321   3         3         3         1m
  ```

- 下記のコマンドを実行して更新処理の一時停止を行います。

  ```shell
  kubectl rollout pause deployment.v1.apps/nginx-deployment
  ```

  実行結果は下記のとおりです。

  ```
  deployment.apps/nginx-deployment paused
  ```

- 次に Deployment のイメージを更新します。

  ```shell
  kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1
  ```

  実行結果は下記のとおりです。

  ```
  deployment.apps/nginx-deployment image updated
  ```

- 新しいロールアウトが開始されていないことを確認します。

  ```shell
  kubectl rollout history deployment.v1.apps/nginx-deployment
  ```

  実行結果は下記のとおりです。

  ```
  deployments "nginx"
  REVISION  CHANGE-CAUSE
  1   <none>
  ```

- Deployment の更新に成功したことを確認するためにロールアウトのステータスを確認
  します。

  ```shell
  kubectl get rs
  ```

  実行結果は下記のとおりです。

  ```
  NAME               DESIRED   CURRENT   READY     AGE
  nginx-2142116321   3         3         3         2m
  ```

- ユーザーは何度も更新を行えます。例えば Deployment が使用するリソースを更新しま
  す。

  ```shell
  kubectl set resources deployment.v1.apps/nginx-deployment -c=nginx --limits=cpu=200m,memory=512Mi
  ```

  実行結果は下記のとおりです。

  ```
  deployment.apps/nginx-deployment resource requirements updated
  ```

  一時停止する前の初期状態では更新処理は機能しますが、Deployment が一時停止され
  ている間は新しい更新処理は反映されません。

- 最後に、Deployment の稼働を再開させ、新しい ReplicaSet が更新内容を全て反映さ
  せているのを確認します。
  ```shell
  kubectl rollout resume deployment.v1.apps/nginx-deployment
  ```
  実行結果は下記のとおりです。
  ```
  deployment.apps/nginx-deployment resumed
  ```
- 更新処理が完了するまでロールアウトのステータスを確認します。
  ```shell
  kubectl get rs -w
  ```
  実行結果は下記のとおりです。
  ```
  NAME               DESIRED   CURRENT   READY     AGE
  nginx-2142116321   2         2         2         2m
  nginx-3926361531   2         2         0         6s
  nginx-3926361531   2         2         1         18s
  nginx-2142116321   1         2         2         2m
  nginx-2142116321   1         2         2         2m
  nginx-3926361531   3         2         1         18s
  nginx-3926361531   3         2         1         18s
  nginx-2142116321   1         1         1         2m
  nginx-3926361531   3         3         1         18s
  nginx-3926361531   3         3         2         19s
  nginx-2142116321   0         1         1         2m
  nginx-2142116321   0         1         1         2m
  nginx-2142116321   0         0         0         2m
  nginx-3926361531   3         3         3         20s
  ```
- 最新のロールアウトのステータスを確認します。 `shell kubectl get rs`

      実行結果は下記のとおりです。
      ```
      NAME               DESIRED   CURRENT   READY     AGE
      nginx-2142116321   0         0         0         2m
      nginx-3926361531   3         3         3         28s
      ```

  {{< note >}} 一時停止した Deployment の稼働を再開させない限り、ユーザーは
  Deployment のロールバックはできません。 {{< /note >}}

## Deployment のステータス {#deployment-status}

Deployment は、そのライフサイクルの間に様々な状態に遷移します。新しい ReplicaSet
へのロールアウト中は[進行中](#progressing-deployment)になり、その後
は[完了](#complete-deployment)し、また[失敗](#failed-deployment)にもなります。

### Deployment の更新処理 {#progressing-deployment}

下記のタスクが実行中のとき、Kubernetes は Deployment の状態を*progressing* にし
ます。

- Deployment が新しい ReplicaSet を作成する。
- Deployment が新しい ReplicaSet をスケールアップさせている。
- Deployment が古い ReplicaSet をスケールダウンさせている。
- 新しい Pod が準備中もしくは利用可能な状態になる(少なくと
  も[MinReadySeconds](#min-ready-seconds)の間は準備中になります)。

ユーザーは`kubectl rollout status`を実行して Deployment の進行状態を確認できます
。

### Deployment の更新処理の完了 {#complete-deployment}

Deployment が下記の状態になったとき、Kubernetes は Deployment のステータス
を*complete* にします。

- Deployment の全てのレプリカが、指定された最新のバージョンに更新される。これは
  ユーザーが指定した更新処理が完了したことを意味します。
- Deployment の全てのレプリカが利用可能になる。
- Deployment の古いレプリカが 1 つも稼働していない。

`kubectl rollout status`を実行して、Deployment の更新が完了したことを確認できま
す。ロールアウトが正常に完了すると`kubectl rollout status`の終了コードが 0 で返
されます。

```shell
kubectl rollout status deployment.v1.apps/nginx-deployment
```

実行結果は下記のとおりです。

```
Waiting for rollout to finish: 2 of 3 updated replicas are available...
deployment.apps/nginx-deployment successfully rolled out
$ echo $?
0
```

### Deployment の更新処理の失敗 {#failed-deployment}

新しい ReplicaSet のデプロイが完了せず、更新処理が止まる場合があります。これは主
に下記の要因によるものです。

- 不十分なリソースの割り当て
- ReadinessProbe の失敗
- コンテナイメージの取得ができない
- 不十分なパーミッション
- リソースリミットのレンジ
- アプリケーションランタイムの設定の不備

このような状況を検知する 1 つの方法として、Deployment のリソース定義でデッドライ
ンのパラメータを指定します
([`.spec.progressDeadlineSeconds`](#progress-deadline-seconds))。`.spec.progressDeadlineSeconds`は
Deployment の更新が停止したことを示す前に Deployment コントローラーが待つ秒数を
示します。

下記の`kubectl`コマンドでリソース定義に`progressDeadlineSeconds`を設定します。こ
れは Deployment の更新が止まってから 10 分後に、コントローラーが失敗を通知させる
ためです。

```shell
kubectl patch deployment.v1.apps/nginx-deployment -p '{"spec":{"progressDeadlineSeconds":600}}'
```

実行結果は下記のとおりです。

```
deployment.apps/nginx-deployment patched
```

一度デッドラインを超過すると、Deployment コントローラーは Deployment
の`.status.conditions`に下記の DeploymentCondition を追加します。

- Type=Progressing
- Status=False
- Reason=ProgressDeadlineExceeded

ステータスの状態に関するさらなる情報
は[Kubernetes API の規則](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#typical-status-properties)を
参照してください。

{{< note >}} Kubernetes は停止状態の Deployment に対して、ステータス状態を報告す
る以外のアクションを実行しません。高レベルのオーケストレーターはこれを利用して、
状態に応じて行動できます。例えば、前のバージョンへの Deployment のロールバックが
挙げられます。 {{< /note >}}

{{< note >}} Deployment を停止すると、Kubernetes はユーザーが指定したデッドライ
ンを超えたかどうかチェックしません。ユーザーはロールアウトのとゆうでも
Deployment を安全に一時停止でき、デッドラインを超えたイベントをトリガーすること
なく再開できます。 {{< /note >}}

設定したタイムアウトの秒数が小さかったり、一時的なエラーとして扱える他の種類のエ
ラーが原因となり、Deployment で一時的なエラーが出る場合があります。例えば、リソ
ースの割り当てが不十分な場合を考えます。Deployment の詳細情報を確認すると、下記
のセクションが表示されます。

```shell
kubectl describe deployment nginx-deployment
```

実行結果は下記のとおりです。

```
<...>
Conditions:
  Type            Status  Reason
  ----            ------  ------
  Available       True    MinimumReplicasAvailable
  Progressing     True    ReplicaSetUpdated
  ReplicaFailure  True    FailedCreate
<...>
```

`kubectl get deployment nginx-deployment -o yaml`を実行すると、Deployment のステ
ータスは下記のようになります。

```
status:
  availableReplicas: 2
  conditions:
  - lastTransitionTime: 2016-10-04T12:25:39Z
    lastUpdateTime: 2016-10-04T12:25:39Z
    message: Replica set "nginx-deployment-4262182780" is progressing.
    reason: ReplicaSetUpdated
    status: "True"
    type: Progressing
  - lastTransitionTime: 2016-10-04T12:25:42Z
    lastUpdateTime: 2016-10-04T12:25:42Z
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: 2016-10-04T12:25:39Z
    lastUpdateTime: 2016-10-04T12:25:39Z
    message: 'Error creating: pods "nginx-deployment-4262182780-" is forbidden: exceeded quota:
      object-counts, requested: pods=1, used: pods=3, limited: pods=2'
    reason: FailedCreate
    status: "True"
    type: ReplicaFailure
  observedGeneration: 3
  replicas: 2
  unavailableReplicas: 2
```

最後に、一度 Deployment の更新処理のデッドラインを越えると、Kubernetes は
Deployment のステータスと進行中の状態を更新します。

```
Conditions:
  Type            Status  Reason
  ----            ------  ------
  Available       True    MinimumReplicasAvailable
  Progressing     False   ProgressDeadlineExceeded
  ReplicaFailure  True    FailedCreate
```

Deployment か他のリソースコントローラーのスケールダウンを行うか、使用している名
前空間内でリソースの割り当てを増やすことで、リソースの割り当て不足の問題に対処で
きます。割り当て条件を満たすと、Deployment コントローラーは Deployment のロール
アウトを完了させ、Deployment のステータスが成功状態になるのを確認できます
(`Status=True`と`Reason=NewReplicaSetAvailable`)。

```
Conditions:
  Type          Status  Reason
  ----          ------  ------
  Available     True    MinimumReplicasAvailable
  Progressing   True    NewReplicaSetAvailable
```

`Status=True`の`Type=Available`は、Deployment が最小可用性の状態であることを意味
します。最小可用性は、Deployment の更新戦略において指定されているパラメータによ
り決定されます。`Status=True`の`Type=Progressing`は、Deployment のロールアウトの
途中で、更新処理が進行中であるか、更新処理が完了し、必要な最小数のレプリカが利用
可能であることを意味します(各 Type の Reason 項目を確認してください。このケース
では、`Reason=NewReplicaSetAvailable`は Deployment の更新が完了したことを意味し
ます)。

`kubectl rollout status`を実行して Deployment が更新に失敗したかどうかを確認でき
ます。`kubectl rollout status`は Deployment が更新処理のデッドラインを超えたとき
に 0 以外の終了コードを返します。

```shell
kubectl rollout status deployment.v1.apps/nginx-deployment
```

実行結果は下記のとおりです。

```
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
error: deployment "nginx" exceeded its progress deadline
$ echo $?
1
```

### 失敗した Deployment の操作

更新完了した Deployment に適用した全てのアクションは、更新失敗した Deployment に
対しても適用されます。スケールアップ、スケールダウンができ、前のリビジョンへのロ
ールバックや、Deployment のテンプレートに複数の更新を適用させる必要があるときは
一時停止もできます。

## 古いリビジョンのクリーンアップポリシー {#clean-up-policy}

Deployment が管理する古い ReplicaSet をいくつ保持するかを指定するために
、`.spec.revisionHistoryLimit`フィールドを設定できます。この値を超えた古い
ReplicaSet はバックグラウンドでガーベージコレクションの対象となって削除されます
。デフォルトではこの値は 10 です。

{{< note >}} このフィールドを明示的に 0 に設定すると、Deployment の全ての履歴を
削除します。従って、Deployment はロールバックできません。 {{< /note >}}

## カナリアパターンによるデプロイ

Deployment を使って一部のユーザーやサーバーに対してリリースのロールアウトをした
いとき
、[リソースの管理](/docs/concepts/cluster-administration/manage-deployment/#canary-deployments)に
記載されているカナリアパターンに従って、リリース毎に 1 つずつ、複数の Deployment
を作成できます。

## Deployment Spec の記述

他の全ての Kubernetes の設定と同様に、Deployment
は`apiVersion`、`kind`や`metadata`フィールドを必要とします。設定ファイルの利用に
関する情報
は[アプリケーションのデプロイ](/docs/tutorials/stateless-application/run-stateless-application-deployment/)を
参照してください。コンテナーの設定に関して
は[リソースを管理するための kubectl の使用](/docs/concepts/overview/working-with-objects/object-management/)を
参照してください。

Deployment
は[`.spec`セクション](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status)も
必要とします。

### Pod テンプレート

`.spec.template`と`.spec.selector`は`.spec`における必須のフィールドです。

`.spec.template`は[Pod テンプレート](/ja/docs/concepts/workloads/pods/pod-overview/#podテンプレート)で
す。これは.spec 内でネストされていないことと、`apiVersion`や`kind`を持たないこと
を除いては[Pod](/ja/docs/concepts/workloads/pods/pod/)と同じスキーマとなります。

Pod の必須フィールドに加えて、Deployment 内の Pod テンプレートでは適切なラベルと
再起動ポリシーを設定しなくてはなりません。ラベルは他のコントローラーと重複しない
ようにしてください。ラベルについては、[セレクター](#selector)を参照してください
。

[`.spec.template.spec.restartPolicy`](/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy)が`Always`に
等しいときのみ許可されます。これはテンプレートで指定されていない場合のデフォルト
値です。

### レプリカ数

`.spec.replias`は理想的な Pod の数を指定するオプションのフィールドです。デフォル
トは 1 です。

### セレクター {#selector}

`.spec.selector`は必須フィールドで、Deployment によって対象とされる Pod
の[ラベルセレクター](/docs/concepts/overview/working-with-objects/labels/)を指定
します。

`.spec.selector`は`.spec.template.metadata.labels`と一致している必要があり、一致
しない場合は API によって拒否されます。

`apps/v1`バージョンにおいて、`.spec.selector`と`.metadata.labels`が指定されてい
ない場合、`.spec.template.metadata.labels`の値に初期化されません。そのた
め`.spec.selector`と`.metadata.labels`を明示的に指定する必要があります。ま
た`apps/v1`の Deployment において`.spec.selector`は作成後に不変になります。

Deployment のテンプレートが`.spec.template`と異なる場合や、`.spec.replicas`の値
を超えて Pod が稼働している場合、Deployment はセレクターに一致するラベルを持つ
Pod を削除します。Pod の数が理想状態より少ない場合 Deployment
は`.spec.template`をもとに新しい Pod を作成します。

{{< note >}} ユーザーは、Deployment のセレクターに一致するラベルを持つ Pod を、
直接作成したり他の Deployment や ReplicaSet や ReplicationController によって作
成するべきではありません。作成した場合は最初の Deployment が、ラベルに一致する新
しい Pod を作成したとみなしてしまいます。Kubernetes はユーザーがこれを行ってもエ
ラーなどを出さず、処理を止めません。 {{< /note >}}

セレクターが重複する複数のコントローラーを持つとき、そのコントローラーは互いに競
合状態となり、正しくふるまいません。

### 更新戦略

`.spec.strategy`は古い Pod から新しい Pod に置き換える際の更新戦略を指定します
。`.spec.strategy.type`は"Recreate"もしくは"RollingUpdate"を指定できます。デフォ
ルトは"RollingUpdate"です。

#### Deployment の再作成

`.spec.strategy.type==Recreate`と指定されているとき、既存の全ての Pod は新しい
Pod が作成される前に削除されます。

#### Deployment のローリングアップデート

`.spec.strategy.type==RollingUpdate`と指定されているとき、Deployment
は[ローリングアップデート](/docs/tasks/run-application/rolling-update-replication-controller/)に
より Pod を更新します。ローリングアップデートの処理をコントロールするため
に`maxUnavailable`と`maxSurge`を指定できます。

##### maxUnavailable

`.spec.strategy.rollingUpdate.maxUnavailable`はオプションのフィールドで、更新処
理において利用不可となる最大の Pod 数を指定します。値は絶対値(例: 5)を指定するか
、理想状態の Pod のパーセンテージを指定します(例: 10%)。パーセンテージを指定した
場合、絶対値は小数切り捨てされて計算されます
。`.spec.strategy.rollingUpdate.maxSurge`が 0 に指定されている場合、この値を 0
にできません。デフォルトでは 25%です。

例えば、この値が 30%と指定されているとき、ローリングアップデートが開始すると古い
ReplicaSet はすぐに理想状態の 70%にスケールダウンされます。一度新しい Pod が稼働
できる状態になると、古い ReplicaSet はさらにスケールダウンされ、続いて新しい
ReplicaSet がスケールアップされます。この間、利用可能な Pod の総数は理想状態の
Pod の少なくとも 70%以上になるように保証されます。

##### maxSurge

`.spec.strategy.rollingUpdate.maxSurge`はオプションのフィールドで、理想状態の
Pod 数を超えて作成できる最大の Pod 数を指定します。値は絶対値(例: 5)を指定するか
、理想状態の Pod のパーセンテージを指定します(例: 10%)。パーセンテージを指定した
場合、絶対値は小数切り上げで計算されます。`MaxUnavailable`が 0 に指定されている
場合、この値を 0 にできません。デフォルトでは 25%です。

例えば、この値が 30%と指定されているとき、ローリングアップデートが開始すると新し
い ReplicaSet はすぐに更新されます。このとき古い Pod と新しい Pod の総数は理想状
態の 130%を超えないように更新されます。一度古い Pod が削除されると、新しい
ReplicaSet はさらにスケールアップされます。この間、利用可能な Pod の総数は理想状
態の Pod に対して最大 130%になるように保証されます。

### progressDeadlineSeconds

`.spec.progressDeadlineSeconds`はオプションのフィールドで、システムが Deployment
の[更新に失敗](#failed-deployment)したと判断するまでに待つ秒数を指定します。更新
に失敗したと判断されたとき、リソースのステータス
は`Type=Progressing`、`Status=False`かつ`Reason=ProgressDeadlineExceeded`となる
のを確認できます。Deployment コントローラーは Deployment の更新のリトライし続け
ます。今後、自動的なロールバックが実装されたとき、更新失敗状態になるとすぐに
Deployment コントローラーがロールバックを行うようになります。

この値が指定されているとき、`.spec.minReadySeconds`より大きい値を指定する必要が
あります。

### minReadySeconds {#min-ready-seconds}

`.spec.minReadySeconds`はオプションのフィールドで、新しく作成された Pod が利用可
能となるために、最低どれくらいの秒数コンテナーがクラッシュすることなく稼働し続け
ればよいかを指定するものです。デフォルトでは 0 です(Pod は作成されるとすぐに利用
可能と判断されます)。Pod が利用可能と判断された場合についてさらに学ぶため
に[Container Probes](/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)を
参照してください。

### rollbackTo

`.spec.rollbackTo`は、`extensions/v1beta1`と`apps/v1beta1`の API バージョンにお
いて非推奨で、`apps/v1beta2`以降の API バージョンではサポートされません。かわり
に、[前のリビジョンへのロールバック](#rolling-back-to-a-previous-revision)で説明
されているように`kubectl rollout undo`を使用するべきです。

### リビジョン履歴の保持上限

Deployment のリビジョン履歴は、Deployment が管理する ReplicaSet に保持されていま
す。

`.spec.revisionHistoryLimit`はオプションのフィールドで、ロールバック可能な古い
ReplicaSet の数を指定します。この古い ReplicaSet は`etcd`内のリソースを消費し
、`kubectl get rs`の出力結果を見にくくします。Deployment の各リビジョンの設定は
ReplicaSet に保持されます。このため一度古い ReplicaSet が削除されると、そのリビ
ジョンの Deployment にロールバックすることができなくなります。デフォルトでは 10
もの古い ReplicaSet が保持されます。しかし、この値の最適値は新しい Deploymnet の
更新頻度と安定性に依存します。

さらに詳しく言うと、この値を 0 にすると、0 のレプリカを持つ古い全ての ReplicaSet
が削除されます。このケースでは、リビジョン履歴が完全に削除されているため新しい
Deployment のロールアウトを完了することができません。

### paused

`.spec.paused`はオプションの boolean 値で、Deployment の一時停止と再開のための値
です。一時停止されているものと、そうでないものとの違いは、一時停止されている
Deployment は PodTemplateSpec のいかなる変更があってもロールアウトがトリガーされ
ないことです。デフォルトでは Deployment は一時停止していない状態で作成されます。

## Deployment の代替案

### kubectl rolling update

[`kubectl rolling update`](/docs/reference/generated/kubectl/kubectl-commands#rolling-update)に
よって、同様の形式で Pod と ReplicationController を更新できます。しかし
Deployment の使用が推奨されます。なぜなら Deployment の作成は宣言的であり、ロー
リングアップデートが更新された後に過去のリビジョンにロールバックできるなど、いく
つかの追加機能があります。

{{% /capture %}}
