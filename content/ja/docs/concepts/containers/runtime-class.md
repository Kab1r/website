---
reviewers:
title: ランタイムクラス(Runtime Class)
content_template: templates/concept
weight: 20
---

{{% capture overview %}}

{{< feature-state for_k8s_version="v1.14" state="beta" >}}

このページでは RuntimeClass リソースと、runtime セクションのメカニズムについて説
明します。

{{< warning >}} RuntimeClass は Kubernetes1.14 の β 版アップグレードにおいて*破
壊的な* 変更を含んでいます。もしユーザーが Kubernetes1.14 以前のバージョンを使っ
ていた場合
、[RuntimeClass の α 版から β 版へのアップグレード](#upgrading-runtimeclass-from-alpha-to-beta)を
参照してください。 {{< /warning >}}

{{% /capture %}}

{{% capture body %}}

## RuntimeClass について

RuntimeClass はコンテナランタイムの設定を選択するための機能です。そのコンテナラ
ンタイム設定は Pod のコンテナを稼働させるために使われます。

### セットアップ

RuntimeClass 機能の Feature Gate が有効になっていることを確認してください(デフォ
ルトで有効です)。Feature Gate を有効にする方法については
、[Feature Gates](/docs/reference/command-line-tools-reference/feature-gates/)を
参照してください。その`RuntimeClass`の Feature Gate は ApiServer と kubelet のど
ちらも有効になっていなければなりません。

1. ノード上で CRI 実装を設定する。（ランタイムに依存）
2. 対応する RuntimeClass リソースを作成する。

#### 1. ノード上で CRI 実装を設定する。

RuntimeClass を通じて利用可能な設定は Container Runtime Interface (CRI)の実装依
存となります。ユーザーの環境の CRI 実装の設定方法は、対応するドキュメント
([下記](#cri-configuration))を参照ください。

{{< note >}} RuntimeClass は現時点において、クラスター全体で同じ種類の Node 設定
であることを仮定しています。(これは全ての Node がコンテナランタイムに関して同じ
方法で構成されていることを意味します)。設定が異なる Node に関しては、スケジュー
リング機能を通じて RuntimeClass とは独立して管理されなくてはなりません
。([Pod を Node に割り当てる方法](/ja/docs/concepts/configuration/assign-pod-node/)を
参照して下さい)。 {{< /note >}}

RuntimeClass の設定は、RuntimeClass によって参照される`ハンドラー`名を持ちます。
そのハンドラーは正式な DNS-1123 に準拠する形式のラベルでなくてはなりません(英数
字 + `-`の文字で構成されます)。

#### 2. 対応する RuntimeClass リソースを作成する

ステップ 1 にて設定する各項目は、関連する`ハンドラー` 名を持ちます。それはどの設
定かを指定するものです。各ハンドラーにおいて、対応する RuntimeClass オブジェクト
が作成されます。

その RuntimeClass リソースは現時点で 2 つの重要なフィールドを持ちます。それは
RuntimeClass の名前(`metadata.name`)とハンドラー(`handler`)です。そのオブジェク
トの定義は下記のようになります。

```yaml
apiVersion: node.k8s.io/v1beta1 # RuntimeClassはnode.k8s.ioというAPIグループで定義されます。
kind: RuntimeClass
metadata:
  name: myclass # RuntimeClass名
  # RuntimeClassはネームスペースなしのリソースです。
handler: myconfiguration # 対応するCRI設定
```

{{< note >}} RuntimeClass の書き込み操作(create/update/patch/delete)はクラスター
管理者のみに制限されることを推奨します。これはたいていデフォルトで有効となってい
ます。さらなる詳細に関して
は[Authorization Overview](/docs/reference/access-authn-authz/authorization/)を
参照してください。 {{< /note >}}

### 使用例

一度 RuntimeClass がクラスターに対して設定されると、それを使用するのは非常に簡単
です。PodSpec の`runtimeClassName`を指定してください。
例えば

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  runtimeClassName: myclass
  # ...
```

これは、Kubelet に対して Pod を稼働させるための RuntimeClass を使うように指示し
ます。もし設定された RuntimeClass が存在しない場合や、CRI が対応するハンドラーを
実行できない場合、その Pod は`Failed`とい
う[フェーズ](/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase)になります。
エラーメッセージに関しては対応す
る[イベント](/docs/tasks/debug-application-cluster/debug-application-introspection/)を
参照して下さい。

もし`runtimeClassName`が指定されていない場合、デフォルトの RuntimeHandler が使用
され、これは RuntimeClass の機能が無効であるときのふるまいと同じものとなります。

### CRI の設定

CRI ランタイムのセットアップに関するさらなる詳細は
、[CRI のインストール](/docs/setup/cri/)を参照してください。

#### dockershim

Kubernetes のビルトインの dockershim CRI は、ランタイムハンドラーをサポートして
いません。

#### [containerd](https://containerd.io/)

ランタイムハンドラーは、`/etc/containerd/config.toml`にある containerd の設定フ
ァイルにより設定されます。正しいハンドラーは、その`runtime`セクションで設定され
ます。

```
[plugins.cri.containerd.runtimes.${HANDLER_NAME}]
```

containerd の設定に関する詳細なドキュメントは下記を参照してください。
https://github.com/containerd/cri/blob/master/docs/config.md

#### [cri-o](https://cri-o.io/)

ランタイムハンドラーは、`/etc/crio/crio.conf`にある cri-o の設定ファイルにより設
定されます。正しいハンドラー
は[crio.runtime table](https://github.com/kubernetes-sigs/cri-o/blob/master/docs/crio.conf.5.md#crioruntime-table)で
設定されます。

```
[crio.runtime.runtimes.${HANDLER_NAME}]
  runtime_path = "${PATH_TO_BINARY}"
```

cri-o の設定に関する詳細なドキュメントは下記を参照してください。
https://github.com/kubernetes-sigs/cri-o/blob/master/cmd/crio/config.go

### RutimeClass を α 版から β 版にアップグレードする

RuntimeClass の β 版の機能は、下記の変更点を含みます。

- `node.k8s.io`API グループと`runtimeclasses.node.k8s.io`リソースは
  CustomResourceDefinition からビルトイン API へとマイグレーションされました。
- `spec`は RuntimeClass の定義内にインライン化されました(RuntimeClassSpec はすで
  にありません)。
- `runtimeHandler`フィールドは`handler`にリネームされました。
- `handler`フィールドは、全ての API バージョンにおいて必須となりました。これは α
  版の API での`runtimeHandler`フィールドもまた必須であることを意味します。
- `handler`フィールドは正しい DNS ラベルの形式である必要があり
  ([RFC 1123](https://tools.ietf.org/html/rfc1123))、これは`.`文字はもはや含むこ
  とができないことを意味します(全てのバージョンにおいて)。有効なハンドラー名は、
  次の正規表現に従います。`^[a-z0-9]([-a-z0-9]*[a-z0-9])?$`

**Action Required:** 次のアクションは RuntimeClass の α 版から β 版へのアップグ
レードにおいて対応が必須です。

- RuntimeClass リソースは Kubernetes v1.14 にアップグレードされた*後に* 再作成さ
  れなくてはなりません。そして`runtimeclasses.node.k8s.io`という CRD は手動で削
  除されるべきです。
  ```
  kubectl delete customresourcedefinitions.apiextensions.k8s.io runtimeclasses.node.k8s.io
  ```
- `runtimeHandler`の指定がないか、もしくは空文字の場合や、ハンドラー名に`.`文字
  列が使われている場合は α 版の RuntimeClass においてもはや有効ではありません。
  正しい形式のハンドラー設定に変更しなくてはなりません(先ほど記載した内容を確認
  ください)。

{{% /capture %}}
