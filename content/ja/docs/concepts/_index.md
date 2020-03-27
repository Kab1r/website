---
title: コンセプト
main_menu: true
content_template: templates/concept
weight: 40
---

{{% capture overview %}}

本セクションは、Kubernetes システムの各パートと
、{{< glossary_tooltip text="クラスター" term_id="cluster" length="all" >}}を表
現するために Kubernetes が使用する抽象概念について学習し、Kubernetes の仕組みを
より深く理解するのに役立ちます。

{{% /capture %}}

{{% capture body %}}

## 概要

Kubernetes を機能させるには、_Kubernetes API オブジェクト_ を使用して、実行した
いアプリケーションやその他のワークロード、使用するコンテナイメージ、レプリカ(複
製)の数、どんなネットワークやディスクリソースを利用可能にするかなど、クラスター
の _desired state_ (望ましい状態)を記述します。desired sate (望ましい状態)をセッ
トするには、Kubernetes API を使用してオブジェクトを作成します。通常はコマンドラ
インインターフェイス `kubectl` を用いて Kubernetes API を操作しますが
、Kubernetes API を直接使用してクラスターと対話し、desired state (望ましい状態)
を設定、または変更することもできます。

一旦 desired state (望ましい状態)を設定すると、Pod Lifecycle Event
Generator([PLEG](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/node/pod-lifecycle-event-generator.md))
を使用した*Kubernetes コントロールプレーン*が機能し、クラスターの現在の状態を
desired state (望ましい状態)に一致させます。そのために Kubernetes はさまざまなタ
スク(たとえば、コンテナの起動または再起動、特定アプリケーションのレプリカ数のス
ケーリング等)を自動的に実行します。Kubernetes コントロールプレーンは、クラスター
で実行されている以下のプロセスで構成されています。

- **Kubernetes Master**
  :[kube-apiserver](/docs/admin/kube-apiserver/)、[kube-controller-manager](/docs/admin/kube-controller-manager/)、[kube-scheduler](/docs/admin/kube-scheduler/)
  の 3 プロセスの集合です。これらのプロセスはクラスター内の一つのノード上で実行
  されます。実行ノードはマスターノードとして指定します。
- クラスター内の個々の非マスターノードは、それぞれ 2 つのプロセスを実行します。
  - **[kubelet](/docs/admin/kubelet/)**, Kubernetes Master と通信します。
  - **[kube-proxy](/docs/admin/kube-proxy/)**, 各ノードの Kubernetes ネットワー
    クサービスを反映するネットワークプロキシです。

## Kubernetes オブジェクト

Kubernetes には、デプロイ済みのコンテナ化されたアプリケーションやワークロード、
関連するネットワークとディスクリソース、クラスターが何をしているかに関するその他
の情報といった、システムの状態を表現する抽象が含まれています。これらの抽象は
、Kubernetes API のオブジェクトによって表現されます。詳細については
、[Kubernetes オブジェクトについて知る](/docs/concepts/overview/working-with-objects/kubernetes-objects/)を
ご覧ください。

基本的な Kubernetes のオブジェクトは次のとおりです。

- [Pod](/ja/docs/concepts/workloads/pods/pod-overview/)
- [Service](/ja/docs/concepts/services-networking/service/)
- [Volume](/docs/concepts/storage/volumes/)
- [Namespace](/ja/docs/concepts/overview/working-with-objects/namespaces/)

Kubernetes には、[コントローラー](/docs/concepts/architecture/controller/)に依存
して基本オブジェクトを構築し、追加の機能と便利な機能を提供する高レベルの抽象化も
含まれています。これらには以下のものを含みます:

- [Deployment](/ja/docs/concepts/workloads/controllers/deployment/)
- [DaemonSet](/ja/docs/concepts/workloads/controllers/daemonset/)
- [StatefulSet](/ja/docs/concepts/workloads/controllers/statefulset/)
- [ReplicaSet](/ja/docs/concepts/workloads/controllers/replicaset/)
- [Job](/docs/concepts/workloads/controllers/jobs-run-to-completion/)

## Kubernetes コントロールプレーン

Kubernetes マスターや kubelet プロセスといった Kubernetes コントロールプレーンの
さまざまなパーツは、Kubernetes がクラスターとどのように通信するかを統制します。
コントロールプレーンはシステム内のすべての Kubernetes オブジェクトの記録を保持し
、それらのオブジェクトの状態を管理するために継続的制御ループを実行します。コント
ロールプレーンの制御ループは常にクラスターの変更に反応し、システム内のすべてのオ
ブジェクトの実際の状態が、指定した状態に一致するように動作します。

たとえば、Kubernetes API を使用して Deployment を作成する場合、システムには新し
い desired state (望ましい状態)が提供されます。Kubernetes コントロールプレーンは
、そのオブジェクトの作成を記録します。そして、要求されたアプリケーションの開始、
およびクラスターノードへのスケジューリングにより指示を完遂します。このようにして
クラスターの実際の状態を望ましい状態に一致させます。

### Kubernetes マスター

Kubernetes のマスターは、クラスターの望ましい状態を維持する責務を持ちます
。`kubectl` コマンドラインインターフェイスを使用するなどして Kubernetes とやり取
りするとき、ユーザーは実際にはクラスターにある Kubernetes のマスターと通信してい
ます。

「マスター」とは、クラスター状態を管理するプロセスの集合を指します。通常これらの
プロセスは、すべてクラスター内の単一ノードで実行されます。このノードはマスターと
も呼ばれます。マスターは、可用性と冗長性のために複製することもできます。

### Kubernetes ノード

クラスターのノードは、アプリケーションとクラウドワークフローを実行するマシン
(VM、物理サーバーなど)です。Kubernetes のマスターは各ノードを制御します。運用者
自身がノードと直接対話することはほとんどありません。

{{% /capture %}}

{{% capture whatsnext %}}

コンセプトページを追加したい場合は、
[ページテンプレートの使用](/docs/home/contribute/page-templates/) のコンセプトペ
ージタイプとコンセプトテンプレートに関する情報を確認してください。

{{% /capture %}}
