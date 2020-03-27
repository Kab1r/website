---
reviewers:
title: Kubernetes API
content_template: templates/concept
weight: 30
card:
  name: concepts
  weight: 30
---

{{% capture overview %}}

全般的な API の規則は
、[API 規則ドキュメント](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md)に
記載されています。

API エンドポイント、リソースタイプ、そしてサンプル
は[API リファレンス](/docs/reference)に記載されています。

API への外部からのアクセスは
、[API アクセス制御ドキュメント](/docs/reference/access-authn-authz/controlling-access/)に
記載されています。

Kubernetes API は、システムの宣言的設定スキーマの基礎としても機能します
。[kubectl](/docs/reference/kubectl/overview/)コマンドラインツールから、API オブ
ジェクトを作成、更新、削除、取得することが出来ます。

また、Kubernetes は、シリアライズされた状態を(現在
は[etcd](https://coreos.com/docs/distributed-configuration/getting-started-with-etcd/)に
)API リソースの単位で保存しています。

Kubernetes それ自身は複数のコンポーネントから構成されており、API を介して連携し
ています。

{{% /capture %}}

{{% capture body %}}

## API の変更

我々の経験上、成功を収めているどのようなシステムも、新しいユースケースへの対応、
既存の変更に合わせ、成長し変わっていく必要があります。したがって、Kubernetes に
も継続的に変化、成長することを期待しています。一方で、長期間にわたり、既存のクラ
イアントとの互換性を損なわないようにする予定です。一般的に、新しい API リソース
とリソースフィールドは頻繁に追加されることが予想されます。リソース、フィールドの
削除は、[API 廃止ポリシー](/docs/reference/using-api/deprecation-policy/)への準
拠を必要とします。

何が互換性のある変更を意味するか、また API をどのように変更するかは
、[API 変更ドキュメント](https://git.k8s.io/community/contributors/devel/api_changes.md)に
詳解されています。

## OpenAPI と Swagger の定義

完全な API の詳細は、[OpenAPI](https://www.openapis.org/)に記載されています。

Kubernetes 1.10 から、KubernetesAPI サーバーは`/openapi/v2`のエンドポイントを通
じて、OpenAPI 仕様を提供しています。リクエストフォーマットは、HTTP ヘッダーを下
記のように設定することで指定されます:

| ヘッダ          | 設定可能な値                                                                                                                                                                             |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Accept          | `application/json`, `application/com.github.proto-openapi.spec.v2@v1.0+protobuf` （デフォルトの content-type は、`*/*`に対して`application/json`か、もしくはこのヘッダーを送信しません） |
| Accept-Encoding | `gzip` （このヘッダーを送信しないことも許容されています）                                                                                                                                |

1.14 より前のバージョンでは、フォーマット分離エンドポイント（`/swagger.json`,
`/swagger-2.0.0.json`, `/swagger-2.0.0.pb-v1`, `/swagger-2.0.0.pb-v1.gz`）が
、OpenAPI 仕様を違うフォーマットで提供しています。これらのエンドポイントは非推奨
となっており、Kubernetes1.14 で削除される予定です。

**OpenAPI 仕様の取得サンプル**:

| 1.10 より前                 | Kubernetes1.10 以降                                                                                              |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| GET /swagger.json           | GET /openapi/v2 **Accept**: application/json                                                                     |
| GET /swagger-2.0.0.pb-v1    | GET /openapi/v2 **Accept**: application/com.github.proto-openapi.spec.v2@v1.0+protobuf                           |
| GET /swagger-2.0.0.pb-v1.gz | GET /openapi/v2 **Accept**: application/com.github.proto-openapi.spec.v2@v1.0+protobuf **Accept-Encoding**: gzip |

Kubernetes は、他の手段として主にクラスター間の連携用途向けの API に、Protocol
buffers をベースにしたシリアライズフォーマットを実装しており、そのフォーマットの
概要
は[デザイン提案](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/api-machinery/protobuf.md)に
記載されています。また各スキーマの IDF ファイルは、API オブジェクトを定義してい
る Go パッケージ内に配置されています。

また、1.14 より前のバージョンの KubernetesAPI サーバーでは
、[Swagger v1.2](http://swagger.io/)をベースにした Kubernetes 仕様を
、`/swaggerapi`で公開しています。このエンドポイントは非推奨となっており
、Kubernetes1.14 で削除される予定です。

## API バージョニング

フィールドの削除やリソース表現の再構成を簡単に行えるようにするため、Kubernetes
は複数の API バージョンをサポートしており
、`/api/v1`や`/apis/extensions/v1beta1`のように、それぞれ異なる API のパスが割り
当てられています。

API が、システムリソースと動作について明確かつ一貫したビューを提供し、サポート終
了、実験的な API へのアクセス制御を有効にするために、リソースまたはフィールドレ
ベルではなく、API レベルでバージョンを付けることを選択しました。JSON と Protocol
Buffers のシリアライズスキーマも、スキーマ変更に関して同じガイドラインに従います
。ここから以下の説明は、双方のフォーマットをカバーしています。

API とソフトウエアのバージョニングは、間接的にしか関連していないことに注意してく
ださい
。[API とリリースバージョニング提案](https://git.k8s.io/community/contributors/design-proposals/release/versioning.md)で
、API とソフトウェアのバージョニングの関連について記載しています。

異なるバージョンの API は、異なるレベル（版）の安定性とサポートを持っています。
それぞれのレベル（版）の基準は
、[API 変更ドキュメント](https://git.k8s.io/community/contributors/devel/sig-architecture/api_changes.md#alpha-beta-and-stable-versions)に
詳細が記載されています。下記に簡潔にまとめます:

- アルファレベル（版）:
  - バージョン名に`alpha`を含みます（例、`v1alpha1`）。
  - バグが多いかもしれません。アルファ機能の有効化がバグを顕在化させるかもしれま
    せん。デフォルトでは無効となっています。
  - アルファ機能のサポートは、いつでも通知無しに取りやめられる可能性があります。
  - ソフトウェアリリース後、API が通知無しに互換性が無い形で変更される可能性があ
    ります。
  - バグが増えるリスク、また長期サポートが無いことから、短期間のテスト用クラスタ
    ーでの利用を推奨します。
- ベータレベル（版）:
  - バージョン名に`beta`を含みます（例、`v2beta3`）。
  - コードは十分にテストされています。ベータ機能の有効化は安全だと考えられます。
    デフォルトで有効化されています。
  - 全体的な機能のサポートは取りやめられませんが、詳細は変更される可能性がありま
    す。
  - オブジェクトのスキーマ、意味はその後のベータ、安定版リリースで互換性が無い形
    で変更される可能性があります。その場合、次のバージョンへアップデートするため
    の手順を提供します。その手順では API オブジェクトの削除、修正、再作成が必要
    になるかもしれません。修正のプロセスは多少の検討が必要になるかもしれません。
    これは、この機能を利用しているアプリケーションでダウンタイムが必要になる可能
    性があるためです。
  - 今後のリリースで、互換性の無い変更が行われる可能性があるため、ビジネスクリテ
    ィカルな場面以外での利用を推奨します。もし複数のクラスターを持っており、それ
    ぞれ個別にアップグレードが可能な場合、この制限の影響を緩和できるかもしれませ
    ん。
  - **是非ベータ機能を試して、フィードバックをください！ベータから安定版になって
    しまうと、より多くの変更を加えることが難しくなってしまいます。**
- 安定版:
  - バージョン名は`vX`のようになっており、`X`は整数です。
  - 安定版の機能は、今後のリリースバージョンにも適用されます。

## API グループ

KubernetesAPI の拡張を簡易に行えるようにするため
、[_API グループ_](https://git.k8s.io/community/contributors/design-proposals/api-machinery/api-group.md)を
実装しました。 API グループは、REST のパスとシリアライズされたオブジェクト
の`apiVersion`フィールドで指定されます。

現在、いくつかの API グループが利用されています:

1. _core_ グループ（度々、_legacy group_ と呼ばれます）は、`/api/v1`という REST
   のパスで、`apiVersion: v1`を使います。

1. 名前付きのグループは、`/apis/$GROUP_NAME/$VERSION`という REST のパスで
   、`apiVersion: $GROUP_NAME/$VERSION`（例、`apiVersion: batch/v1`）を使います
   。サポートされている API グループの全リストは
   、[Kubernetes API リファレンス](/docs/reference/)を参照してください。

[カスタムリソース](/docs/concepts/api-extension/custom-resources/)で API を拡張
するために、2 つの方法がサポートされています:

1. [カスタムリソース定義](/docs/tasks/access-kubernetes-api/extend-api-custom-resource-definitions/)は
   、とても基本的な CRUD が必要なユーザー向けです。
1. 独自の API サーバーを実装可能な、フルセットの Kubernetes API が必要なユーザー
   は
   、[アグリゲーター](/docs/tasks/access-kubernetes-api/configure-aggregation-layer/)を
   使い、クライアントにシームレスな形で拡張を行います。

## API グループの有効化

いくつかのリソースと API グループはデフォルトで有効になっています。それらは、API
サーバーの`--runtime-config`設定で、有効化、無効化できます。`--runtime-config`は
、カンマ区切りの複数の値を設定可能です。例えば、batch/v1 を無効化する場合
、`--runtime-config=batch/v1=false`をセットし、batch/v2alpha1 を有効化する場合
、`--runtime-config=batch/v2alpha1`をセットします。このフラグは、API サーバーの
ランタイム設定を表す key=value のペアを、カンマ区切りで指定したセットを指定可能
です。

重要: API グループ、リソースの有効化、無効化は、`--runtime-config`の変更を反映す
るため、API サーバーとコントローラーマネージャーの再起動が必要です。

## API グループのリソースの有効化

DaemonSets、Deployments、HorizontalPodAutoscalers、Ingresses、JobsReplicaSets、
そして ReplicaSets はデフォルトで有効です。その他の拡張リソースは、API サーバー
の`--runtime-config`を設定することで有効化できます。`--runtime-config`はカンマ区
切りの複数の値を設定可能です。例えば、deployments と ingress を無効化する場合
、`--runtime-config=extensions/v1beta1/deployments=false,extensions/v1beta1/ingresses=false`と
設定します。

{{% /capture %}}
