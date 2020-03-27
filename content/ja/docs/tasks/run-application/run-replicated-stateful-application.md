---
title: レプリカを持つステートフルアプリケーションを実行する
content_template: templates/tutorial
weight: 30
---

{{% capture overview %}}

このページでは
、[StatefulSet](/ja/docs/concepts/workloads/controllers/statefulset/) コントロー
ラーを使用して、レプリカを持つステートフルアプリケーションを実行する方法を説明し
ます。ここでの例は、非同期レプリケーションを行う複数のスレーブを持つ、単一マスタ
ーの MySQL です。

**この例は本番環境向けの構成ではない**ことに注意してください。具体的には、MySQL
の設定が安全ではないデフォルトのままとなっています。これは Kubernetes でステート
フルアプリケーションを実行するための一般的なパターンに焦点を当てるためです。

{{% /capture %}}

{{% capture prerequisites %}}

- {{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}
- {{< include "default-storage-class-prereqs.md" >}}
- このチュートリアルは、あなた
  が[PersistentVolume](/docs/concepts/storage/persistent-volumes/)
  と[StatefulSet](/ja/docs/concepts/workloads/controllers/statefulset/)、さらに
  は[Pod](/ja/docs/concepts/workloads/pods/pod/)、
  [Service](/ja/docs/concepts/services-networking/service/)、
  [ConfigMap](/docs/tasks/configure-pod-container/configure-pod-configmap/)など
  の他のコアな概念に精通していることを前提としています。
- MySQL に関する知識は記事の理解に役立ちますが、このチュートリアルは他のシステム
  にも役立つ一般的なパターンを提示することを目的としています。

{{% /capture %}}

{{% capture objectives %}}

- StatefulSet コントローラーを使用して、レプリカを持つ MySQL トポロジーをデプロ
  イします。
- MySQL クライアントトラフィックを送信します。
- ダウンタイムに対する耐性を観察します。
- StatefulSet をスケールアップおよびスケールダウンします。

{{% /capture %}}

{{% capture lessoncontent %}}

## MySQL をデプロイする

この MySQL のデプロイの例は、1 つの ConfigMap、2 つの Service、および 1 つの
StatefulSet から構成されます。

### ConfigMap

次の YAML 設定ファイルから ConfigMap を作成します。

```shell
kubectl apply -f https://k8s.io/examples/application/mysql/mysql-configmap.yaml
```

{{< codenew file="application/mysql/mysql-configmap.yaml" >}}

この ConfigMap は、MySQL マスターとスレーブの設定を独立して制御するために、それ
ぞれの`my.cnf`を上書きする内容を提供します。この場合、マスターはスレーブにレプリ
ケーションログを提供するようにし、スレーブはレプリケーション以外の書き込みを拒否
するようにします。

ConfigMap 自体に特別なことはありませんが、ConfigMap の各部分は異なる Pod に適用
されます。各 Pod は、StatefulSet コントローラーから提供される情報に基づいて、初
期化時に ConfigMap のどの部分を見るかを決定します。

### Services

以下の YAML 設定ファイルから Service を作成します。

```shell
kubectl apply -f https://k8s.io/examples/application/mysql/mysql-services.yaml
```

{{< codenew file="application/mysql/mysql-services.yaml" >}}

ヘッドレスサービスは、StatefulSet コントローラーが StatefulSet の一部である Pod
ごとに作成する DNS エントリーのベースエントリーを提供します。この例ではヘッドレ
スサービスの名前は`mysql`なので、同じ Kubernetes クラスタの同じ名前空間内の他の
Pod は、`<pod-name>.mysql`を名前解決することで Pod にアクセスできます。

`mysql-read`と呼ばれるクライアントサービスは、独自のクラスタ IP を持つ通常の
Service であり、 Ready 状態のすべての MySQL Pod に接続を分散します。 Service の
エンドポイントには、MySQL マスターとすべてのスレーブが含まれる可能性があります。

読み込みクエリーのみが、負荷分散されるクライアントサービスを使用できることに注意
してください。 MySQL マスターは 1 つしかいないため、クライアントが書き込みを実行
するためには、 (ヘッドレスサービス内の DNS エントリーを介して)MySQL のマスター
Pod に直接接続する必要があります。

### StatefulSet

最後に、次の YAML 設定ファイルから StatefulSet を作成します。

```shell
kubectl apply -f https://k8s.io/examples/application/mysql/mysql-statefulset.yaml
```

{{< codenew file="application/mysql/mysql-statefulset.yaml" >}}

次のコマンドを実行して起動の進行状況を確認できます。

```shell
kubectl get pods -l app=mysql --watch
```

しばらくすると、3 つの Pod すべてが Running 状態になるはずです。

```
NAME      READY     STATUS    RESTARTS   AGE
mysql-0   2/2       Running   0          2m
mysql-1   2/2       Running   0          1m
mysql-2   2/2       Running   0          1m
```

**Ctrl+C**を押してウォッチをキャンセルします。起動が進行しない場合は
、[始める前に](#始める前に)で説明されているように、 PersistentVolume の動的プロ
ビジョニング機能が有効になっていることを確認してください。

このマニフェストでは、StatefulSet の一部としてステートフルな Pod を管理するため
にさまざまな手法を使用しています。次のセクションでは、これらの手法のいくつかに焦
点を当て、StatefulSet が Pod を作成するときに何が起こるかを説明します。

## ステートフルな Pod の初期化を理解する

StatefulSet コントローラーは、序数インデックスの順に Pod を一度に 1 つずつ起動し
ます。各 Pod が Ready 状態を報告するまで待機してから、その次の Pod の起動が開始
されます。

さらに、コントローラーは各 Pod に `<statefulset-name>-<ordinal-index>`という形式
の一意で不変の名前を割り当てます。この例の場合、Pod の名前
は`mysql-0`、`mysql-1`、そして`mysql-2`となります。

上記の StatefulSet マニフェスト内の Pod テンプレートは、これらのプロパティーを利
用して、 MySQL レプリケーションの起動を順序正しく実行します。

### 構成を生成する

Pod スペック内のコンテナを起動する前に、Pod は最初に
[初期化コンテナ](/ja/docs/concepts/workloads/pods/init-containers/)を定義された
順序で実行します。

最初の初期化コンテナは`init-mysql`という名前で、序数インデックスに基づいて特別な
MySQL 設定ファイルを生成します。

スクリプトは、`hostname`コマンドによって返される Pod 名の末尾から抽出することに
よって、自身の序数インデックスを特定します。それから、序数を(予約された値を避け
るために数値オフセット付きで)MySQL の`conf.d`ディレクトリーの`server-id.cnf`とい
うファイルに保存します。これは、StatefulSet コントローラーによって提供される一意
で不変の ID を、同じ特性を必要とする MySQL サーバー ID の領域に変換します。

さらに、`init-mysql`コンテナ内のスクリプトは、`master.cnf`または`slave.cnf`のい
ずれかを、 ConfigMap から内容を`conf.d`にコピーすることによって適用します。この
トポロジー例は単一の MySQL マスターと任意の数のスレーブで構成されているため、ス
クリプトは単に序数の`0`がマスターになるように、それ以外のすべてがスレーブになる
ように割り当てます。 StatefulSet コントローラーによる
[デプロイ順序の保証](/ja/docs/concepts/workloads/controllers/statefulset/#デプロイとスケーリングの保証)と
組み合わせると、スレーブが作成される前に MySQL マスターが Ready 状態になるため、
スレーブはレプリケーションを開始できます。

### 既存データをクローンする

一般に、新しい Pod がセットにスレーブとして参加するときは、 MySQL マスターにはす
でにデータがあるかもしれないと想定する必要があります。また、レプリケーションログ
が期間の先頭まで全て揃っていない場合も想定する必要があります。これらの控えめな仮
定は、実行中の StatefulSet のサイズを初期サイズに固定するのではなく、時間の経過
とともにスケールアップまたはスケールダウンできるようにするために重要です。

2 番目の初期化コンテナは`clone-mysql`という名前で、スレーブ Pod が空の
PersistentVolume で最初に起動したときに、クローン操作を実行します。つまり、実行
中の別の Pod から既存のデータをすべてコピーするので、そのローカル状態はマスター
からレプリケーションを開始するのに十分な一貫性があります。

MySQL 自体はこれを行うためのメカニズムを提供していないため、この例では Percona
XtraBackup という人気のあるオープンソースツールを使用しています。クローンの実行
中は、ソースとなる MySQL サーバーのパフォーマンスが低下する可能性があります。
MySQL マスターへの影響を最小限に抑えるために、スクリプトは各 Pod に序数インデッ
クスが自分より 1 低い Pod からクローンするように指示します。 StatefulSet コント
ローラーは、`N+1`の Pod を開始する前には必ず`N`の Pod が Ready 状態であることを
保証するので、この方法が機能します。

### レプリケーションを開始する

初期化コンテナが正常に完了すると、通常のコンテナが実行されます。 MySQL の Pod は
実際に`mysqld`サーバーを実行する`mysql`コンテナと、
[サイドカー](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns)
として機能する`xtrabackup`コンテナから成ります。

`xtrabackup`サイドカーはクローンされたデータファイルを見て、スレーブ上で MySQL
レプリケーションを初期化する必要があるかどうかを決定します。もし必要がある場合
、`mysqld`が準備できるのを待ってから、 XtraBackup クローンファイルから抽出された
レプリケーションパラメーターで`CHANGE MASTER TO`と`START SLAVE`コマンドを実行し
ます。

スレーブがレプリケーションを開始すると、スレーブは MySQL マスターを記憶し、サー
バーが再起動した場合または接続が停止した場合に、自動的に再接続します。また、スレ
ーブはその不変の DNS 名(`mysql-0.mysql`)でマスターを探すため、再スケジュールされ
たために新しい Pod IP を取得したとしても、自動的にマスターを見つけます。

最後に、レプリケーションを開始した後、`xtrabackup`コンテナはデータのクローンを要
求する他の Pod からの接続を待ち受けます。 StatefulSet がスケールアップした場合や
、次の Pod が PersistentVolumeClaim を失ってクローンをやり直す必要がある場合に備
えて、このサーバーは無期限に起動したままになります。

## クライアントトラフィックを送信する

テストクエリーを MySQL マスター(ホスト名 `mysql-0.mysql`)に送信するには、
`mysql:5.7`イメージを使って一時的なコンテナーを実行し、`mysql`クライアントバイナ
リーを実行します。

```shell
kubectl run mysql-client --image=mysql:5.7 -i --rm --restart=Never --\
  mysql -h mysql-0.mysql <<EOF
CREATE DATABASE test;
CREATE TABLE test.messages (message VARCHAR(250));
INSERT INTO test.messages VALUES ('hello');
EOF
```

Ready 状態を報告したいずれかのサーバーにテストクエリーを送信するには、ホスト
名`mysql-read`を使用します。

```shell
kubectl run mysql-client --image=mysql:5.7 -i -t --rm --restart=Never --\
  mysql -h mysql-read -e "SELECT * FROM test.messages"
```

次のような出力が得られるはずです。

```
Waiting for pod default/mysql-client to be running, status is Pending, pod ready: false
+---------+
| message |
+---------+
| hello   |
+---------+
pod "mysql-client" deleted
```

`mysql-read`サービスがサーバー間で接続を分散させることを実証するために、ループ
で`SELECT @@server_id`を実行することができます。

```shell
kubectl run mysql-client-loop --image=mysql:5.7 -i -t --rm --restart=Never --\
  bash -ic "while sleep 1; do mysql -h mysql-read -e 'SELECT @@server_id,NOW()'; done"
```

接続の試行ごとに異なるエンドポイントが選択される可能性があるため、報告され
る`@@server_id`はランダムに変更されるはずです。

```
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         100 | 2006-01-02 15:04:05 |
+-------------+---------------------+
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         102 | 2006-01-02 15:04:06 |
+-------------+---------------------+
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         101 | 2006-01-02 15:04:07 |
+-------------+---------------------+
```

ループを止めたいときは**Ctrl+C**を押すことができますが、別のウィンドウで実行した
ままにしておくことで、次の手順の効果を確認できます。

## Pod と Node のダウンタイムをシミュレーションする

単一のサーバーではなくスレーブのプールから読み取りを行うことによって可用性が高ま
っていることを実証するため、 Pod を強制的に Ready ではない状態にする間、上記
の`SELECT @@server_id`ループを実行したままにしてください。

### Readiness Probe を壊す

`mysql`コンテナに対する
[readiness probe](/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#define-readiness-probes)
は、`mysql -h 127.0.0.1 -e 'SELECT 1'`コマンドを実行することで、サーバーが起動し
ていてクエリーが実行できることを確認します。

この readiness probe を失敗させる 1 つの方法は、そのコマンドを壊すことです。

```shell
kubectl exec mysql-2 -c mysql -- mv /usr/bin/mysql /usr/bin/mysql.off
```

ここでは、`mysql-2` Pod の実際のコンテナのファイルシステムにアクセスし、
`mysql`コマンドの名前を変更して readiness probe がコマンドを見つけられないように
しています。数秒後、Pod はそのコンテナの 1 つが Ready ではないと報告するはずです
。以下を実行して確認できます。

```shell
kubectl get pod mysql-2
```

`READY`列の`1/2`を見てください。

```
NAME      READY     STATUS    RESTARTS   AGE
mysql-2   1/2       Running   0          3m
```

この時点で、`SELECT @@server_id`ループは実行され続け、しかしもう`102`が報告され
ないことが確認できるはずです。 `init-mysql`スクリプト
が`server-id`を`100+$ordinal`として定義したことを思い出して下さい。そのため、サ
ーバー ID`102`は Pod の`mysql-2`に対応します。

それでは Pod を修復しましょう。すると数秒後にループ出力に再び現れるはずです。

```shell
kubectl exec mysql-2 -c mysql -- mv /usr/bin/mysql.off /usr/bin/mysql
```

### Pod を削除する

StatefulSet は、Pod が削除された場合に Pod を再作成します。これは ReplicaSet が
ステートレスな Pod に対して行うのと同様です。

```shell
kubectl delete pod mysql-2
```

StatefulSet コントローラーは`mysql-2` Pod がもう存在しないことに気付き、同じ名前
で同じ PersistentVolumeClaim にリンクされた新しい Pod を作成します。サーバー
ID`102`がしばらくの間ループ出力から消えて、また元に戻るのが確認できるはずです。

### ノードを drain する

Kubernetes クラスタに複数のノードがある場合は、
[drain](/docs/reference/generated/kubectl/kubectl-commands/#drain)を発行してノー
ドのダウンタイム(例えばノードのアップグレード時など)をシミュレートできます。

まず、ある MySQL Pod がどのノード上にいるかを確認します。

```shell
kubectl get pod mysql-2 -o wide
```

ノード名が最後の列に表示されるはずです。

```
NAME      READY     STATUS    RESTARTS   AGE       IP            NODE
mysql-2   2/2       Running   0          15m       10.244.5.27   kubernetes-minion-group-9l2t
```

その後、次のコマンドを実行してノードを drain します。これにより、新しい Pod がそ
のノードにスケジュールされないように cordon され、そして既存の Pod は強制退去さ
れます。 `<node-name>`は前のステップで確認したノードの名前に置き換えてください。

この操作はノード上の他のアプリケーションに影響を与える可能性があるため、 **テス
トクラスタでのみこの操作を実行**するのが最善です。

```shell
kubectl drain <node-name> --force --delete-local-data --ignore-daemonsets
```

Pod が別のノードに再スケジュールされる様子を確認しましょう。

```shell
kubectl get pod mysql-2 -o wide --watch
```

次のような出力が見られるはずです。

```
NAME      READY   STATUS          RESTARTS   AGE       IP            NODE
mysql-2   2/2     Terminating     0          15m       10.244.1.56   kubernetes-minion-group-9l2t
[...]
mysql-2   0/2     Pending         0          0s        <none>        kubernetes-minion-group-fjlm
mysql-2   0/2     Init:0/2        0          0s        <none>        kubernetes-minion-group-fjlm
mysql-2   0/2     Init:1/2        0          20s       10.244.5.32   kubernetes-minion-group-fjlm
mysql-2   0/2     PodInitializing 0          21s       10.244.5.32   kubernetes-minion-group-fjlm
mysql-2   1/2     Running         0          22s       10.244.5.32   kubernetes-minion-group-fjlm
mysql-2   2/2     Running         0          30s       10.244.5.32   kubernetes-minion-group-fjlm
```

また、サーバー ID`102`が`SELECT @@server_id`ループの出力からしばらくの消えて、そ
して戻ることが確認できるはずです。

それでは、ノードを uncordon して正常な状態に戻しましょう。

```shell
kubectl uncordon <node-name>
```

## スレーブの数をスケーリングする

MySQL レプリケーションでは、スレーブを追加することで読み取りクエリーのキャパシテ
ィーをスケールできます。 StatefulSet を使用している場合、単一のコマンドでこれを
実行できます。

```shell
kubectl scale statefulset mysql  --replicas=5
```

次のコマンドを実行して、新しい Pod が起動してくるのを確認します。

```shell
kubectl get pods -l app=mysql --watch
```

新しい Pod が起動すると、サーバー ID`103`と`104`が`SELECT @@server_id`ループの出
力に現れます。

また、これらの新しいサーバーが、これらのサーバーが存在する前に追加したデータを持
っていることを確認することもできます。

```shell
kubectl run mysql-client --image=mysql:5.7 -i -t --rm --restart=Never --\
  mysql -h mysql-3.mysql -e "SELECT * FROM test.messages"
```

```
Waiting for pod default/mysql-client to be running, status is Pending, pod ready: false
+---------+
| message |
+---------+
| hello   |
+---------+
pod "mysql-client" deleted
```

元の状態へのスケールダウンもシームレスに可能です。

```shell
kubectl scale statefulset mysql --replicas=3
```

ただし、スケールアップすると新しい PersistentVolumeClaim が自動的に作成されます
が、スケールダウンしてもこれらの PVC は自動的には削除されないことに注意して下さ
い。このため、初期化された PVC をそのまま置いておいくことで再スケールアップを速
くしたり、 PV を削除する前にデータを抽出するといった選択が可能になります。

次のコマンドを実行してこのことを確認できます。

```shell
kubectl get pvc -l app=mysql
```

StatefulSet を 3 にスケールダウンしたにもかかわらず、5 つの PVC すべてがまだ存在
しています。

```
NAME           STATUS    VOLUME                                     CAPACITY   ACCESSMODES   AGE
data-mysql-0   Bound     pvc-8acbf5dc-b103-11e6-93fa-42010a800002   10Gi       RWO           20m
data-mysql-1   Bound     pvc-8ad39820-b103-11e6-93fa-42010a800002   10Gi       RWO           20m
data-mysql-2   Bound     pvc-8ad69a6d-b103-11e6-93fa-42010a800002   10Gi       RWO           20m
data-mysql-3   Bound     pvc-50043c45-b1c5-11e6-93fa-42010a800002   10Gi       RWO           2m
data-mysql-4   Bound     pvc-500a9957-b1c5-11e6-93fa-42010a800002   10Gi       RWO           2m
```

余分な PVC を再利用するつもりがないのであれば、削除することができます。

```shell
kubectl delete pvc data-mysql-3
kubectl delete pvc data-mysql-4
```

{{% /capture %}}

{{% capture cleanup %}}

1. `SELECT @@server_id`ループを実行している端末で**Ctrl+C**を押すか、別の端末か
   ら次のコマンドを実行して、ループをキャンセルします。

   ```shell
   kubectl delete pod mysql-client-loop --now
   ```

1. StatefulSet を削除します。これによって Pod の終了も開始されます。

   ```shell
   kubectl delete statefulset mysql
   ```

1. Pod が消えたことを確認します。 Pod が終了処理が完了するのには少し時間がかかる
   かもしれません。

   ```shell
   kubectl get pods -l app=mysql
   ```

   上記のコマンドから以下の出力が戻れば、Pod が終了したことがわかります。

   ```
   No resources found.
   ```

1. ConfigMap、Services、および PersistentVolumeClaim を削除します。

   ```shell
   kubectl delete configmap,service,pvc -l app=mysql
   ```

1. PersistentVolume を手動でプロビジョニングした場合は、それらを手動で削除し、ま
   た、下層にあるリソースも解放する必要があります。動的プロビジョニング機能を使
   用した場合は、PersistentVolumeClaim を削除すれば、自動的に PersistentVolume
   も削除されます。一部の動的プロビジョナー(EBS や PD など)は、PersistentVolume
   を削除すると同時に下層にあるリソースも解放します。

{{% /capture %}}

{{% capture whatsnext %}}

- その他のステートフルアプリケーションの例は
  、[Helm Charts repository](https://github.com/kubernetes/charts)を見てください
  。

{{% /capture %}}
