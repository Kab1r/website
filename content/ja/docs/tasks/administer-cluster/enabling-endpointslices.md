---
title: EndpointSliceの有効化
content_template: templates/task
---

{{% capture overview %}} このページは Kubernetes の EndpointSlice の有効化の概要
を説明します。 {{% /capture %}}

{{% capture prerequisites %}} {{< include "task-tutorial-prereqs.md" >}}
{{< version-check >}} {{% /capture %}}

{{% capture steps %}}

## 概要

EndpointSlice は、Kubernetes の Endpoints に対してスケーラブルで拡張可能な代替手
段を提供します。Endpoints が提供する機能のベースの上に構築し、スケーラブルな方法
で拡張します。Service が多数(100 以上)のネットワークエンドポイントを持つ場合、そ
れらは単一の大きな Endpoints リソースではなく、複数の小さな EndpointSlice に分割
されます。

## EndpointSlice の有効化

{{< feature-state for_k8s_version="v1.17" state="beta" >}}

{{< note >}} EndpointSlice は、最終的には既存の Endpoints を置き換える可能性があ
りますが、多くの Kubernetes コンポーネントはまだ既存の Endpoints に依存していま
す。現時点では EndpointSlice を有効化することは、Endpoints の置き換えではなく、
クラスター内の Endpoints への追加とみなされる必要があります。 {{< /note >}}

EndpointSlice はベータ版の機能とみなされますが、デフォルトでは API のみが有効で
す。kube-proxy による EndpointSlice コントローラーと EndpointSlice の使用は、デ
フォルトでは有効になっていません。

EndpointSlice コントローラーはクラスター内に EndpointSlice を作成し、管理します
。これは
、{{< glossary_tooltip text="kube-apiserver" term_id="kube-apiserver" >}}と{{< glossary_tooltip text="kube-controller-manager" term_id="kube-controller-manager" >}}の`EndpointSlice`の[フィーチャーゲート](/docs/reference/command-line-tools-reference/feature-gates/)で
有効にできます(`--feature-gates=EndpointSlice=true`)。

スケーラビリティ向上のため
、{{<glossary_tooltip text="kube-proxy" term_id="kube-proxy" >}}でフィーチャーゲ
ートを有効にして、Endpoints の代わりに EndpointSlice をデータソースとして使用す
ることもできます。

## EndpointSlice の使用

クラスター内で EndpointSlice を完全に有効にすると、各 Endpoints リソースに対応す
る EndpointSlice リソースが表示されます。既存の Endpoints の機能をサポートするこ
とに加えて、EndpointSlice はトポロジーなどの新しい情報を含める必要があります。こ
れらにより、クラスター内のネットワークエンドポイントのスケーラビリティと拡張性が
大きく向上します。

{{% capture whatsnext %}}

- [EndpointSlice](/docs/concepts/services-networking/endpoint-slices/)を参照して
  ください。
- [サービスとアプリケーションの接続](/ja/docs/concepts/services-networking/connect-applications-service/)を
  参照してください。

{{% /capture %}}
