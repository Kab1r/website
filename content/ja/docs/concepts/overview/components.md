---
title: Kubernetesのコンポーネント
content_template: templates/concept
weight: 20
card:
  name: concepts
  weight: 20
---

{{% capture overview %}} Kubernetes をデプロイすると、クラスターが展開されます。
{{< glossary_definition term_id="cluster" length="all" prepend="クラスターは、">}}

このドキュメントでは、Kubernetes クラスターが機能するために必要となるさまざまな
コンポーネントの概要を説明します。

すべてのコンポーネントが結び付けられた Kubernetes クラスターの図を次に示します。

![Kubernetesのコンポーネント](/images/docs/components-of-kubernetes.png)

{{% /capture %}}

{{% capture body %}}

## マスターコンポーネント

マスターコンポーネントは、クラスターのコントロールプレーンを提供します。マスター
コンポーネントは、クラスターに関する全体的な決定(スケジューリングなど)を行います
。また、クラスターイベントの検出および応答を行います(たとえば、deployment
の`replicas`フィールドが満たされていない場合に、新しい
{{< glossary_tooltip text="pod" term_id="pod">}} を起動する等)。

マスターコンポーネントはクラスター内のどのマシンでも実行できますが、シンプルにす
るため、セットアップスクリプトは通常、すべてのマスターコンポーネントを同じマシン
で起動し、そのマシンではユーザーコンテナを実行しません。マルチマスター VM セット
アップの例については、[高可用性クラスターの構築](/docs/admin/high-availability/)
を参照してください。

### kube-apiserver

{{< glossary_definition term_id="kube-apiserver" length="all" >}}

### etcd

{{< glossary_definition term_id="etcd" length="all" >}}

### kube-scheduler

{{< glossary_definition term_id="kube-scheduler" length="all" >}}

### kube-controller-manager

{{< glossary_definition term_id="kube-controller-manager" length="all" >}}

コントローラーには以下が含まれます。

- ノードコントローラー：ノードがダウンした場合の通知と対応を担当します。
- レプリケーションコントローラー：システム内の全レプリケーションコントローラーオ
  ブジェクトについて、Pod の数を正しく保つ役割を持ちます。
- エンドポイントコントローラー：エンドポイントオブジェクトを注入します(つまり
  、Service と Pod を紐付けます)。
- サービスアカウントとトークンコントローラー：新規の名前空間に対して、デフォルト
  アカウントと API アクセストークンを作成します。

### cloud-controller-manager

[cloud-controller-manager](/docs/tasks/administer-cluster/running-cloud-controller/)
は、基盤であるクラウドプロバイダーと対話するコントローラーを実行します。
cloud-controller-manager バイナリは、Kubernetes リリース 1.6 で導入された機能で
す。

cloud-controller-manager は、クラウドプロバイダー固有のコントローラーループのみ
を実行します。これらのコントローラーループは kube-controller-manager で無効にす
る必要があります。 kube-controller-manager の起動時に `--cloud-provider` フラグ
を `external` に設定することで、コントローラーループを無効にできます。

cloud-controller-manager を使用すると、クラウドベンダーのコードと Kubernetes コ
ードを互いに独立して進化させることができます。以前のリリースでは、コア
Kubernetes コードは、機能的にクラウドプロバイダー固有のコードに依存していました
。将来のリリースでは、クラウドベンダーに固有のコードはクラウドベンダー自身で管理
し、Kubernetes の実行中に cloud-controller-manager にリンクする必要があります。

次のコントローラーには、クラウドプロバイダーへの依存関係があります。

- ノードコントローラー：ノードが応答を停止した後、クラウドで削除されたかどうかを
  判断するため、クラウドプロバイダーをチェックします。
- ルーティングコントローラー：基盤であるクラウドインフラでルーティングを設定しま
  す。
- サービスコントローラー：クラウドプロバイダーのロードバランサーの作成、更新、削
  除を行います。
- ボリュームコントローラー：ボリュームを作成、アタッチ、マウントしたり、クラウド
  プロバイダーとやり取りしてボリュームを調整したりします。

## ノードコンポーネント

ノードコンポーネントはすべてのノードで実行され、稼働中の Pod の管理や Kubernetes
の実行環境を提供します。

### kubelet

{{< glossary_definition term_id="kubelet" length="all" >}}

### kube-proxy

{{< glossary_definition term_id="kube-proxy" length="all" >}}

### コンテナランタイム {#container-runtime}

{{< glossary_definition term_id="container-runtime" length="all" >}}

## アドオン

アドオンはクラスター機能を実装するために Kubernetes リソース
({{< glossary_tooltip term_id="daemonset" >}}、{{< glossary_tooltip term_id="deployment" >}}な
ど)を使用します。アドオンはクラスターレベルの機能を提供しているため、アドオンの
リソースで名前空間が必要なものは`kube-system`名前空間に属します。

いくつかのアドオンについて以下で説明します。より多くの利用可能なアドオンのリスト
は、[アドオン](/docs/concepts/cluster-administration/addons/) をご覧ください。

### DNS

クラスター DNS 以外のアドオンは必須ではありませんが、すべての Kubernetes クラス
ターは[クラスター DNS](/docs/concepts/services-networking/dns-pod-service/)を持
つべきです。多くの使用例がクラスター DNS を前提としています。

クラスター DNS は、環境内の他の DNS サーバーに加えて、Kubernetes サービスの DNS
レコードを提供する DNS サーバーです。

Kubernetes によって開始されたコンテナは、DNS 検索にこの DNS サーバーを自動的に含
めます。

### Web UI (ダッシュボード)

[ダッシュボード](/docs/tasks/access-application-cluster/web-ui-dashboard/)は
、Kubernetes クラスター用の汎用 Web ベース UI です。これによりユーザーはクラスタ
ーおよびクラスター内で実行されているアプリケーションについて、管理およびトラブル
シューティングを行うことができます。

### コンテナリソース監視

[コンテナリソース監視](/docs/tasks/debug-application-cluster/resource-usage-monitoring/)は
、コンテナに関する一般的な時系列メトリックを中央データベースに記録します。また、
そのデータを閲覧するための UI を提供します。

### クラスターレベルログ

[クラスターレベルログ](/docs/concepts/cluster-administration/logging/)メカニズム
は、コンテナのログを、検索／参照インターフェイスを備えた中央ログストアに保存しま
す。

{{% /capture %}} {{% capture whatsnext %}}

- [ノード](/ja/docs/concepts/architecture/nodes/)について学ぶ
- [コントローラー](/docs/concepts/architecture/controller/)について学ぶ
- [kube-scheduler](/ja/docs/concepts/scheduling/kube-scheduler/)について学ぶ
- etcd の公式 [ドキュメント](https://etcd.io/docs/)を読む {{% /capture %}}
