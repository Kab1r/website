---
title: リファレンス
linkTitle: "リファレンス"
main_menu: true
weight: 70
content_template: templates/concept
---

{{% capture overview %}}

本セクションには、Kubernetes のドキュメントのリファレンスが含まれています。

{{% /capture %}}

{{% capture body %}}

## API リファレンス

- [Kubernetes API 概要](/docs/reference/using-api/api-overview/) - Kubernetes
  API の概要です。
- Kubernetes API バージョン
  - [1.17](/docs/reference/generated/kubernetes-api/v1.17/)
  - [1.16](/docs/reference/generated/kubernetes-api/v1.16/)
  - [1.15](/docs/reference/generated/kubernetes-api/v1.15/)
  - [1.14](/docs/reference/generated/kubernetes-api/v1.14/)
  - [1.13](/docs/reference/generated/kubernetes-api/v1.13/)

## API クライアントライブラリー

プログラミング言語から Kubernetes の API を呼ぶためには
、[クライアントライブラリー](/docs/reference/using-api/client-libraries/)を使う
ことができます。公式にサポートしているクライアントライブラリー:

- [Kubernetes Go client library](https://github.com/kubernetes/client-go/)
- [Kubernetes Python client library](https://github.com/kubernetes-client/python)
- [Kubernetes Java client library](https://github.com/kubernetes-client/java)
- [Kubernetes JavaScript client library](https://github.com/kubernetes-client/javascript)

## CLI リファレンス

- [kubectl](/docs/user-guide/kubectl-overview) - コマンドの実行や Kubernetes ク
  ラスターの管理に使う主要な CLI ツールです。
  - [JSONPath](/docs/user-guide/jsonpath/) - kubectl
    で[JSONPath 記法](http://goessner.net/articles/JsonPath/)を使うための構文ガ
    イドです。
- [kubeadm](/docs/admin/kubeadm/) - セキュアな Kubernetes クラスターを簡単にプロ
  ビジョニングするための CLI ツールです。
- [kubefed](/docs/admin/kubefed/) - 連合型クラスターを管理するのに役立つ CLI ツ
  ールです。

## 設定リファレンス

- [kubelet](/docs/admin/kubelet/) - 各ノード上で動作する最も重要なノードエージェ
  ントです。kubelet は一通りの PodSpec を受け取り、コンテナーが実行中で正常であ
  ることを確認します。
- [kube-apiserver](/docs/admin/kube-apiserver/) - Pod、Service、Replication
  Controller 等、API オブジェクトのデータを検証・設定する REST API サーバーです
  。
- [kube-controller-manager](/docs/admin/kube-controller-manager/) - Kubernetes
  に同梱された、コアのコントロールループを埋め込むデーモンです。
- [kube-proxy](/docs/admin/kube-proxy/) - 単純な TCP/UDP ストリームのフォワーデ
  ィングや、一連のバックエンド間で TCP/UDP のラウンドロビンでのフォワーディング
  を実行できます。
- [kube-scheduler](/docs/admin/kube-scheduler/) - 可用性、パフォーマンス、および
  キャパシティを管理するスケジューラーです。

## 設計のドキュメント

Kubernetes の機能に関する設計ドキュメントのアーカイブです
。[Kubernetes アーキテクチャ](https://git.k8s.io/community/contributors/design-proposals/architecture/architecture.md)
と[Kubernetes デザイン概要](https://git.k8s.io/community/contributors/design-proposals)か
ら読み始めると良いでしょう。

{{% /capture %}}
