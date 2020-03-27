---
title: Podについての概観(Pod Overview)
content_template: templates/concept
weight: 10
card:
  name: concepts
  weight: 60
---

{{% capture overview %}} このページでは、Kubernetes のオブジェクトモデルにおいて
、デプロイ可能な最小単位のオブジェクトである`Pod`に関して概観します。
{{% /capture %}}

{{% capture body %}}

## Pod について理解する

_Pod_ は、Kubernetes アプリケーションの基本的な実行単位です。これは、作成または
デプロイする Kubernetes オブジェクトモデルの中で最小かつ最も単純な単位です。Pod
は、{{< glossary_tooltip term_id="cluster" >}}で実行されているプロセスを表します
。

Pod は、アプリケーションのコンテナ(いくつかの場合においては複数のコンテナ)、スト
レージリソース、ユニークなネットワーク IP、およびコンテナの実行方法を管理するオ
プションをカプセル化します。Pod はデプロイメントの単位、すなわち*Kubernetes のア
プリケーションの単一インスタンス* で、単一
の{{< glossary_tooltip term_id="container" >}}または密結合なリソースを共有する少
数のコンテナで構成される場合があります。

[Docker](https://www.docker.com)は Kubernetes の Pod 内で使われる最も一般的なコ
ンテナランタイムですが、Pod は他
の[コンテナランタイム](/ja/docs/setup/production-environment/container-runtimes/)も
同様にサポートしています。

Kubernetes クラスター内での Pod は 2 つの主な方法で使うことができます。

- **単一のコンテナを稼働させる Pod** : いわゆる*「1Pod1 コンテナ」* 構成のモデル
  は、最も一般的な Kubernetes のユースケースです。
  このケースでは、ユーザーは Pod を単一のコンテナのラッパーとして考えることがで
  き、Kubernetes はコンテナを直接扱うというよりは、Pod を管理することになります
  。
- **協調して稼働させる必要がある複数のコンテナを稼働させる Pod** : 単一の Pod は
  、リソースを共有する必要があるような、密接に連携した複数の同じ環境にあるコンテ
  ナからなるアプリケーションをカプセル化することもできます。 これらの同じ環境に
  あるコンテナ群は、サービスの結合力の強いユニットを構成することができます。
  -- 1 つのコンテナが、共有されたボリュームからファイルをパブリックな場所に送信
  し、一方では分割された*サイドカー* コンテナがそれらのファイルを更新します。そ
  の Pod はそれらのコンテナとストレージリソースを、単一の管理可能なエンティティ
  としてまとめます。

[Kubernetes Blog](http://kubernetes.io/blog)にて、Pod のユースケースに関するいく
つかの追加情報を見ることができます。さらなる情報を得たい場合は、下記のページを参
照ください。

- [The Distributed System Toolkit: Patterns for Composite Containers](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns)
- [Container Design Patterns](https://kubernetes.io/blog/2016/06/container-design-patterns)

各 Pod は、与えられたアプリケーションの単一のインスタンスを稼働するためのもので
す。もしユーザーのアプリケーションを水平にスケールさせたい場合(例: 複数インスタ
ンスを稼働させる)、複数の Pod を使うべきです。1 つの Pod は各インスタンスに対応
しています。 Kubernetes において、これは一般的に*レプリケーション* と呼ばれます
。
レプリケーションされた Pod は、通常コントローラーと呼ばれる抽象概念によって単一
のグループとして作成、管理されます。
さらなる情報に関しては[Pod とコントローラー](#pods-and-controllers)を参照して下
さい。

### Pod がどのように複数のコンテナを管理しているか

Pod は凝集性の高いサービスのユニットを構成するような複数の協調プロセス(コンテナ
）をサポートするためにデザインされました。単一の Pod 内のコンテナ群は、クラスタ
ー内において同一の物理マシンもしくは仮想マシン上において自動で同じ環境に配備され
、スケジュールされます。コンテナはリソースや依存関係を共有し、お互いにコミュニケ
ートし、それらがいつ、どのように削除されるかを調整できます。

注意点として、単一の Pod 内で同じ環境に配備され、同時管理される複数のコンテナを
グルーピングするのは、比較的に発展的なユースケースとなります。ユーザーは、コンテ
ナ群が密接に連携するような、特定のインスタンスにおいてのみこのパターンを使用する
べきです。例えば、ユーザーが共有ボリューム内にあるファイル用の Web サーバとして
稼働するコンテナと、下記のダイアグラムにあるような、リモートのソースからファイル
を更新するような分離された*サイドカー* コンテナを持っているような場合です。

{{< figure src="/images/docs/pod.svg" alt="Podのダイアグラム" width="50%" >}}

Pod は、Pod によって構成されたコンテナ群のために 2 種類の共有リソースを提供しま
す。 _ネットワーキング_ と*ストレージ* です。

#### ネットワーキング

各 Pod は固有の IP アドレスを割り当てられます。単一の Pod 内の各コンテナは、IP
アドレスやネットワークポートを含む、そのネットワークの名前空間を共有します。_Pod
内の_ コンテナは`localhost`を使用してお互いに疎通できます。単一の Pod 内のコンテ
ナが*Pod 外* のエンティティと疎通する場合、共有されたネットワークリソース(ポート
など）をどのように使うかに関して調整しなければなりません。

#### ストレージ

単一の Pod は共有されたストレージ{{< glossary_tooltip term_id="volume" >}}のセッ
トを指定できます。Pod 内の全てのコンテナは、その共有されたボリュームにアクセスで
き、コンテナ間でデータを共有することを可能にします。ボリュームもまた、もし Pod
内のコンテナの 1 つが再起動が必要になった場合に備えて、データを永続化できます
。
単一の Pod 内での共有ストレージを Kubernetes がどう実装しているかについてのさら
なる情報については、[Volumes](/docs/concepts/storage/volumes/)を参照してください
。

## Pod を利用する

ユーザーはまれに、Kubenetes 内で独立した Pod を直接作成する場合があります(シング
ルトン Pod など)。これは Pod が比較的、一時的な使い捨てエンティティとしてデザイ
ンされているためです。Pod が作成された時（ユーザーによって直接的、またはコントロ
ーラーによって間接的に作成された場合）、ユーザーのクラスター内の単一
の{{< glossary_tooltip term_id="node" >}}上で稼働するようにスケジューリングされ
ます。その Pod はプロセスが停止されたり、Pod オブジェクトが削除されたり、Pod が
リソースの欠如のために*追い出され* たり、ノードが故障するまでノード上に残り続け
ます。

{{< note >}} 単一の Pod 内でのコンテナを再起動することと、その Pod を再起動する
ことを混同しないでください。Pod はそれ自体は実行されませんが、コンテナが実行され
る環境であり、削除されるまで存在し続けます。 {{< /note >}}

Pod は、Pod それ自体によって自己修復しません。もし、稼働されていないノード上に
Pod がスケジュールされた場合や、スケジューリング操作自体が失敗した場合、Pod が削
除されます。同様に、Pod はリソースの欠如や、ノードのメンテナンスによる追い出しが
あった場合はそこで停止します。Kubernetes は*コントローラー* と呼ばれる高レベルの
抽象概念を使用し、それは比較的使い捨て可能な Pod インスタンスの管理を行います
。
このように、Pod を直接使うのは可能ですが、コントローラーを使用した Pod を管理す
る方がより一般的です。Kubernetes が Pod のスケーリングと修復機能を実現するために
コントローラーをどのように使うかに関する情報
は[Pod とコントローラー](#pods-and-controllers)を参照してください。

### Pod とコントローラー

単一のコントローラーは、ユーザーのために複数の Pod を作成・管理し、レプリケーシ
ョンやロールアウト、クラスターのスコープ内で自己修復の機能をハンドリングします。
例えば、もしノードが故障した場合、コントローラーは異なるノード上に Pod を置き換
えるようにスケジューリングすることで、自動的にリプレース可能となります。

1 つまたはそれ以上の Pod を含むコントローラーの例は下記の通りです。

- [Deployment](/ja/docs/concepts/workloads/controllers/deployment/)
- [StatefulSet](/ja/docs/concepts/workloads/controllers/statefulset/)
- [DaemonSet](/ja/docs/concepts/workloads/controllers/daemonset/)

通常は、コントローラーはユーザーが作成した Pod テンプレートを使用して、担当する
Pod を作成します。

## Pod テンプレート

Pod テンプレートは
、[ReplicationController](/docs/concepts/workloads/controllers/replicationcontroller/)、
[Job](/docs/concepts/jobs/run-to-completion-finite-workloads/)や、
[DaemonSet](/ja/docs/concepts/workloads/controllers/daemonset/)のような他のオブ
ジェクト内で含まれる Pod の仕様となります。コントローラーは実際の Pod を作成する
ために Pod テンプレートを使用します。
下記のサンプルは、メッセージを表示する単一のコンテナを含んだ、シンプルな Pod の
マニフェストとなります。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
    - name: myapp-container
      image: busybox
      command: ["sh", "-c", "echo Hello Kubernetes! && sleep 3600"]
```

全てのレプリカの現在の理想的な状態を指定するというよりも、Pod テンプレートはクッ
キーの抜き型のようなものです。一度クッキーがカットされると、そのクッキーは抜き型
から離れて関係が無くなります。そこにはいわゆる”量子もつれ”といったものはありませ
ん。テンプレートに対するその後の変更や新しいテンプレートへの切り替えは、すでに作
成された Pod 上には直接的な影響はありません。同様に、ReplicationController によ
って作成された Pod は、変更後に直接更新されます。これは Pod との意図的な違いとな
り、その Pod に属する全てのコンテナの現在の理想的な状態を指定します。このアプロ
ーチは根本的にシステムのセマンティクスを単純化し、機能の柔軟性を高めます。

{{% /capture %}}

{{% capture whatsnext %}}

- [Pod](/ja/docs/concepts/workloads/pods/pod/)について更に学びましょう
- Pod の振る舞いに関して学ぶには下記を参照してください
  - [Pod の停止](/ja/docs/concepts/workloads/pods/pod/#podの終了)
  - [Pod のライフサイクル](/ja/docs/concepts/workloads/pods/pod-lifecycle/)
    {{% /capture %}}
