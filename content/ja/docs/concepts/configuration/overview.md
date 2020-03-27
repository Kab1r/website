---
title: 設定のベストプラクティス
content_template: templates/concept
weight: 10
---

{{% capture overview %}} このドキュメントでは、ユーザーガイド、入門マニュアル、
および例を通して紹介されている設定のベストプラクティスを中心に説明します。

このドキュメントは生ものです。このリストには載っていないが他の人に役立つかもしれ
ない何かについて考えている場合、Issue または PR を遠慮なく作成してください。
{{% /capture %}}

{{% capture body %}}

## 一般的な設定の Tips

- 構成を定義する際には、最新の安定した API バージョンを指定してください。

- 設定ファイルは、クラスターに反映される前にバージョン管理システムに保存されるべ
  きです。これによって、必要に応じて設定変更を迅速にロールバックできます。また、
  クラスターの再作成や復元時にも役立ちます。

- JSON ではなく YAML を使って設定ファイルを書いてください。これらのフォーマット
  はほとんどすべてのシナリオで互換的に使用できますが、YAML はよりユーザーフレン
  ドリーになる傾向があります。

- 意味がある場合は常に、関連オブジェクトを単一ファイルにグループ化します。多くの
  場合、1 つのファイルの方が管理が簡単です。例とし
  て[guestbook-all-in-one.yaml](https://github.com/kubernetes/examples/tree/{{<
  param "githubbranch" >}}/guestbook/all-in-one/guestbook-all-in-one.yaml)ファイ
  ルを参照してください。

- 多くの`kubectl`コマンドがディレクトリに対しても呼び出せることも覚えておきまし
  ょう。たとえば、設定ファイルのディレクトリで `kubectl apply`を呼び出すことがで
  きます。

- 不必要にデフォルト値を指定しないでください。シンプルかつ最小限の設定のほうがエ
  ラーが発生しにくくなります。

- よりよいイントロスペクションのために、オブジェクトの説明をアノテーションに入れ
  ましょう。

## "真っ裸"の Pod に対する ReplicaSet、Deployment、および Job

- 可能な限り、"真っ裸"の
  Pod([ReplicaSet](/ja/docs/concepts/workloads/controllers/replicaset/)や[Deployment](/ja/docs/concepts/workloads/controllers/deployment/)に
  バインドされていない Pod)は使わないでください。Node に障害が発生した場合、これ
  らの Pod は再スケジュールされません。

  明示的
  に[`restartPolicy: Never`](/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy)を
  使いたいシーンを除いて、Deployment は Pod を直接作成するよりもほとんど常に望ま
  しい方法です。Deployment には、希望する数の Pod が常に使用可能であることを確認
  するために ReplicaSet を作成したり、Pod を置き換えるための戦略（RollingUpdate
  など）を指定したりできます
  。[Job](/docs/concepts/workloads/controllers/jobs-run-to-completion/)のほうが
  適切な場合もあるかもしれません。

## Service

- 対応するバックエンドワークロード（Deployment または ReplicaSet）の前、およびそ
  れにアクセスする必要があるワークロードの前
  に[Service](/ja/docs/concepts/services-networking/service/)を作成します
  。Kubernetes がコンテナを起動すると、コンテナ起動時に実行されていたすべての
  Service を指す環境変数が提供されます。たとえば、foo という名前の Service が存
  在する場合、すべてのコンテナは初期環境で次の変数を取得します。

  ```shell
  FOO_SERVICE_HOST=<the host the Service is running on>
  FOO_SERVICE_PORT=<the port the Service is running on>
  ```

  _これは順序付けの必要性を意味します_ - `Pod`がアクセスしたい`Service`は`Pod`自
  身の前に作らなければならず、そうしないと環境変数は注入されません。DNS にはこの
  制限はありません。

- （強くお勧めしますが
  ）[クラスターアドオン](/docs/concepts/cluster-administration/addons/)の 1 つの
  選択肢は DNS サーバーです。DNS サーバーは、新しい`Service`について Kubernetes
  API を監視し、それぞれに対して一連の DNS レコードを作成します。クラスタ全体で
  DNS が有効になっている場合は、すべての`Pod`が自動的に`Services`の名前解決を行
  えるはずです。

- どうしても必要な場合以外は、Pod に`hostPort`を指定しないでください。Pod
  を`hostPort`にバインドすると、Pod がスケジュールできる場所の数を制限します、そ
  れぞれの<`hostIP`、 `hostPort`、`protocol`>の組み合わせはユニークでなければな
  らないからです。`hostIP`と`protocol`を明示的に指定しないと、Kubernetes はデフ
  ォルトの`hostIP`として`0.0.0.0`を、デフォルトの `protocol`として`TCP`を使いま
  す。

  デバッグ目的でのみポートにアクセスする必要がある場合は
  、[apiserver proxy](/docs/tasks/access-application-cluster/access-cluster/#manually-constructing-apiserver-proxy-urls)ま
  た
  は[`kubectl port-forward`](/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)を
  使用できます。

  ノード上で Pod のポートを明示的に公開する必要がある場合は、hostPort に頼る前
  に[NodePort](/ja/docs/concepts/services-networking/service/#nodeport)の使用を
  検討してください。

- `hostPort`の理由と同じくして、`hostNetwork`の使用はできるだけ避けてください。

- `kube-proxy`のロードバランシングが不要な場合は
  、[headless Service](/ja/docs/concepts/services-networking/service/#headless-service)（`ClusterIP`が`None`）
  を使用して Service を簡単に検出できます。

## ラベルの使用

- `{ app: myapp, tier: frontend, phase: test, deployment: v3 }`のように、アプリ
  ケーションまたはデプロイメントの**セマンティック属性**を識別す
  る[ラベル](/docs/concepts/overview/working-with-objects/labels/)を定義して使い
  ましょう。これらのラベルを使用して、他のリソースに適切なポッドを選択できます。
  例えば、すべての`tier：frontend`を持つ Pod を選択する Service や
  、`app：myapp`に属するすべての`phase：test`コンポーネント、などです。このアプ
  ローチの例を知るには
  、[ゲストブック](https://github.com/kubernetes/examples/tree/{{< param
  "githubbranch" >}}/guestbook/)アプリも合わせてご覧ください。

セレクターからリリース固有のラベルを省略することで、Service を複数の Deployment
にまたがるように作成できます。
[Deployment](/ja/docs/concepts/workloads/controllers/deployment/)により、ダウン
タイムなしで実行中のサービスを簡単に更新できます。

オブジェクトの望ましい状態は Deployment によって記述され、その仕様への変更が*適
用*されると、Deployment コントローラは制御された速度で実際の状態を望ましい状態に
変更します。

- デバッグ用にラベルを操作できます。Kubernetes コントローラー（ReplicaSet など）
  と Service はセレクターラベルを使用して Pod とマッチするため、Pod から関連ラベ
  ルを削除すると、コントローラーによって考慮されたり、Service によってトラフィッ
  クを処理されたりすることがなくなります。既存の Pod のラベルを削除すると、その
  コントローラーはその代わりに新しい Pod を作成します。これは、「隔離」環境で以
  前の「ライブ」Pod をデバッグするのに便利な方法です。対話的にラベルを削除または
  追加するには
  、[`kubectl label`](/docs/reference/generated/kubectl/kubectl-commands#label)を
  使います。

## コンテナイメージ

[imagePullPolicy](/docs/concepts/containers/images/#updating-images)とイメージの
タグは、[kubelet](/docs/admin/kubelet/)が特定のイメージを pull しようとしたとき
に作用します。

- `imagePullPolicy: IfNotPresent`: ローカルでイメージが見つからない場合にのみイ
  メージを pull します。

- `imagePullPolicy: Always`: Pod の起動時に常にイメージを pull します。

- `imagePullPolicy` のタグが省略されていて、利用してるイメージのタグ
  が`:latest`の場合や省略されている場合、`Always`が適用されます。

- `imagePullPolicy` のタグが省略されていて、利用してるイメージのタグはある
  が`:latest`でない場合、`IfNotPresent`が適用されます。

- `imagePullPolicy: Never`: 常にローカルでイメージを探そうとします。ない場合にも
  イメージは pull しません。

{{< note >}} コンテナが常に同じバージョンのイメージを使用するようにするためには
、そのコンテナイメージ
の[ダイジェスト](https://docs.docker.com/engine/reference/commandline/pull/#pull-an-image-by-digest-immutable-identifier)を
指定することができます(例
:`sha256:45b23dee08af5e43a7fea6c4cf9c25ccf269ee113168c19722f87876677c5cb2`)。こ
のダイジェストはイメージの特定のバージョンを一意に識別するため、ダイジェスト値を
変更しない限り、Kubernetes によって更新されることはありません。 {{< /note >}}

{{< note >}} どのバージョンのイメージが実行されているのかを追跡するのが難しく、
適切にロールバックするのが難しいため、本番環境でコンテナをデプロイするときは
`：latest`タグを使用しないでください。 {{< /note >}}

{{< note >}} ベースイメージのプロバイダーのキャッシュセマンティクスにより
、`imagePullPolicy：Always`もより効率的になります。たとえば、Docker では、イメー
ジが既に存在する場合すべてのイメージレイヤーがキャッシュされ、イメージのダウンロ
ードが不要であるため、pull が高速になります。 {{< /note >}}

## kubectl の使い方

- `kubectl apply -f <directory>`を使いましょう。これを使うと、ディレクトリ内のす
  べての`.yaml`、`.yml`、および`.json`ファイルが`apply`に渡されます。

- `get`や`delete`を行う際は、特定のオブジェクト名を指定するのではなくラベルセレ
  クターを使いましょう
  。[ラベルセレクター](/docs/concepts/overview/working-with-objects/labels/#label-selectors)と[ラベルの効果的な使い方](/docs/concepts/cluster-administration/manage-deployment/#using-labels-effectively)の
  セクションを参照してください。

{{% /capture %}}
