---
reviewers:
title: クラスター管理の概要
content_template: templates/concept
weight: 10
---

{{% capture overview %}} このページは Kubernetes クラスターの作成や管理者向けの
内容です。Kubernetes のコア[コンセプト](/ja/docs/concepts/)についてある程度精通
していることを前提とします。 {{% /capture %}}

{{% capture body %}}

## クラスターのプランニング

Kubernetes クラスターの計画、セットアップ、設定の例を知るに
は[設定](/ja/docs/setup/)のガイドを参照してください。この記事で列挙されているソ
リューションは*ディストリビューション* と呼ばれます。

ガイドを選択する前に、いくつかの考慮事項を挙げます。

- ユーザーのコンピューター上で Kubernetes を試したいでしょうか、それとも高可用性
  のあるマルチノードクラスターを構築したいでしょうか? あなたのニーズにあったディ
  ストリビューションを選択してください。
- **もしあなたが高可用性を求める場合**、
  [複数ゾーンにまたがるクラスター](/docs/concepts/cluster-administration/federation/)の
  設定について学んでください。
- [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/)のよう
  な**ホストされている Kubernetes クラスター**を使用するのか、それとも**自分自身
  でクラスターをホストするのでしょうか**?
- 使用するクラスターは**オンプレミス**なのか、それとも**クラウド (IaaS)**でしょ
  うか? Kubernetes はハイブリッドクラスターを直接サポートしていません。その代わ
  りユーザーは複数のクラスターをセットアップできます。
- Kubernetes を**"ベアメタル"なハードウェア** 上で稼働させるますか? それとも**仮
  想マシン (VMs)** 上で稼働させますか?
- **もしオンプレミスで Kubernetes を構築する場合**、ど
  の[ネットワークモデル](/ja/docs/concepts/cluster-administration/networking/)が
  最適か検討してください。
- **ただクラスターを稼働させたいだけ**でしょうか、それとも**Kubernetes プロジェ
  クトのコードの開発**を行いたいでしょうか? もし後者の場合、開発が進行中のディス
  トリビューションを選択してください。いくつかのディストリビューションはバイナリ
  リリースのみ使用していますが、多くの選択肢があります。
- クラスターを稼働させるのに必要
  な[コンポーネント](/ja/docs/concepts/overview/components/)についてよく理解して
  ください。

注意: 全てのディストリビューションがアクティブにメンテナンスされている訳ではあり
ません。最新バージョンの Kubernetes でテストされたディストリビューションを選択し
てください。

## クラスターの管理

- [クラスターの管理](/docs/tasks/administer-cluster/cluster-management/)では、ク
  ラスターのライフサイクルに関するいくつかのトピックを紹介しています。例えば、新
  規クラスターの作成、クラスターのマスターやワーカーノードのアップグレード、ノー
  ドのメンテナンスの実施(例: カーネルのアップグレード)、稼働中のクラスターの
  Kubernetes API バージョンのアップグレードについてです。

- [ノードの管理](/ja/docs/concepts/architecture/nodes/)方法について学んでくださ
  い。

- 共有クラスターにおけ
  る[リソースクォータ](/docs/concepts/policy/resource-quotas/)のセットアップと管
  理方法について学んでください。

## クラスターをセキュアにする

- [Certificates](/docs/concepts/cluster-administration/certificates/)では、異な
  るツールチェインを使用して証明書を作成する方法を説明します。

- [Kubernetes コンテナの環境](/ja/docs/concepts/containers/container-environment-variables/)で
  は、Kubernetes ノード上での Kubelet が管理するコンテナの環境について説明します
  。

- [Kubernetes API へのアクセス制御](/docs/reference/access-authn-authz/controlling-access/)で
  は、ユーザーとサービスアカウントの権限の設定方法について説明します。

- [認証](/docs/reference/access-authn-authz/authentication/)では、様々な認証オプ
  ションを含む Kubernetes での認証について説明します。

- [認可](/docs/reference/access-authn-authz/authorization/)では、認証とは別に
  、HTTP リクエストの処理方法を制御します。

- [アドミッションコントローラーの使用](/docs/reference/access-authn-authz/admission-controllers/)で
  は、認証と認可の後に Kubernetes API に対するリクエストをインターセプトするプラ
  グインについて説明します。

- [Kubernetes クラスターでの sysctl の使用](/docs/concepts/cluster-administration/sysctl-cluster/)で
  は、管理者向けにカーネルパラメーターを設定するため`sysctl`コマンドラインツール
  の使用方法について解説します。

- [クラスターの監査](/docs/tasks/debug-application-cluster/audit/)では
  、Kubernetes の監査ログの扱い方について解説します。

### kubelet をセキュアにする

- [マスターとノードのコミュニケーション](/ja/docs/concepts/architecture/master-node-communication/)
- [TLS のブートストラップ](/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/)
- [Kubelet の認証/認可](/docs/admin/kubelet-authentication-authorization/)

## オプションのクラスターサービス

- [DNS のインテグレーション](/ja/docs/concepts/services-networking/dns-pod-service/)で
  は、DNS 名を Kubernetes Service に直接名前解決する方法を解説します。

- [クラスターアクティビィのロギングと監視](/docs/concepts/cluster-administration/logging/)で
  は、Kubernetes におけるロギングがどのように行われ、どう実装されているかについ
  て解説します。

{{% /capture %}}
