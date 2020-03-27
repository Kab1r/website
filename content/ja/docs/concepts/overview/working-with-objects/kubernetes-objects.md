---
title: Kubernetesオブジェクトを理解する
content_template: templates/concept
weight: 10
card:
  name: concepts
  weight: 40
---

{{% capture overview %}} このページでは、Kubernetes オブジェクトが Kubernetes
API でどのように表現されているか、またそれらを`.yaml`フォーマットでどのように表
現するかを説明します。 {{% /capture %}}

{{% capture body %}}

## Kubernetes オブジェクトを理解する

_Kubernetes オブジェクト_ は、Kubernetes 上で永続的なエンティティです
。Kubernetes はこれらのエンティティを使い、クラスターの状態を表現します。具体的
に言うと、下記のような内容が表現出来ます:

- どのようなコンテナ化されたアプリケーションが稼働しているか（またそれらはどのノ
  ード上で動いているか）
- それらのアプリケーションから利用可能なリソース
- アプリケーションがどのように振る舞うかのポリシー、例えば再起動、アップグレード
  、耐障害性ポリシーなど

Kubernetes オブジェクトは"意図の記録"です。一度オブジェクトを作成すると
、Kubernetes は常にそのオブジェクトが存在し続けるように動きます。オブジェクトを
作成することで、Kubernetes に対し効果的にあなたのクラスターのワークロードがこの
ようになっていて欲しいと伝えているのです。これが、あなたのクラスターの**望ましい
状態**です。

Kubernetes オブジェクトを操作するには、作成、変更、または削除に関わら
ず[Kubernetes API](/docs/concepts/overview/kubernetes-api/)を使う必要があるでし
ょう。例えば`kubectl`コマンドラインインターフェースを使った場合、この CLI が処理
に必要な Kubernetes API 命令を、あなたに代わり発行します。あなたのプログラムか
ら[クライアントライブラリ](/docs/reference/using-api/client-libraries/)を利用し
、直接 Kubernetes API を利用することも可能です。

### オブジェクトの spec（仕様）と status（状態）

全ての Kubernetes オブジェクトは、オブジェクトの設定を管理する２つの入れ子になっ
たオブジェクトのフィールドを持っています。それは _spec_ オブジェクトと _status_
オブジェクトです。_spec_ オブジェクトはあなたが指定しなければならない項目で、オ
ブジェクトの _望ましい状態_ を記述し、オブジェクトに持たせたい特徴を表現します
。_status_ オブジェクトはオブジェクトの _現在の状態_ を示し、その情報は
Kubernetes から与えられ、更新されます。常に、Kubernetes コントロールプレーンは、
あなたから指定された望ましい状態と現在の状態が一致するよう積極的に管理をします。

例えば、Kubernetes の Deployment はクラスター上で稼働するアプリケーションを表現
するオブジェクトです。Deployment を作成するとき、アプリケーションの複製を３つ稼
働させるよう Deployment の spec で指定するかもしれません。Kubernetes は
Deployment の spec を読み取り、指定されたアプリケーションを３つ起動し、現在の状
態が spec に一致するようにします。もしこれらのインスタンスでどれかが落ちた場合
（status が変わる）、Kubernetes は spec と、status の違いに反応し、修正しようと
します。この場合は、落ちたインスタンスの代わりのインスタンスを立ち上げます。

spec、status、metadata に関するさらなる情報は
、[Kubernetes API Conventions](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md)を
ご確認ください。

### Kubernetes オブジェクトを記述する

Kubernetes でオブジェクトを作成する場合、オブジェクトの基本的な情報（例えば名前
）と共に、望ましい状態を記述したオブジェクトの spec を渡さなければいけません
。KubernetesAPI を利用しオブジェクトを作成する場合（直接 API を呼ぶか
、`kubectl`を利用するかに関わらず）、API リクエストはそれらの情報を JSON 形式で
リクエストの Body 部に含んでいなければなりません。

ここで、Kubernetes の Deployment に必要なフィールドとオブジェクトの spec を記載
した`.yaml`ファイルの例を示します:

{{< codenew file="application/deployment.yaml" >}}

上に示した`.yaml`ファイルを利用して Deployment を作成するには、`kubectl`コマンド
ラインインターフェースに含まれてい
る[`kubectl apply`](/docs/reference/generated/kubectl/kubectl-commands#apply)コ
マンドに`.yaml`ファイルを引数に指定し、実行します。ここで例を示します:

```shell
kubectl apply -f https://k8s.io/examples/application/deployment.yaml --record
```

出力結果は、下記に似た形になります:

```shell
deployment.apps/nginx-deployment created
```

### 必須フィールド

Kubernetes オブジェクトを`.yaml`ファイルに記載して作成する場合、下記に示すフィー
ルドに値をセットしておく必要があります:

- `apiVersion` - どのバージョンの KubernetesAPI を利用してオブジェクトを作成する
  か
- `kind` - どの種類のオブジェクトを作成するか
- `metadata` - オブジェクトを一意に特定するための情報、`name`、string、UID、また
  任意の`namespace`が該当する

またオブジェクトの`spec`の値も指定する必要があります。`spec`の正確なフォーマット
は、Kubernetes オブジェクトごとに異なり、オブジェクトごとに特有な入れ子のフィー
ルドを持っています。[Kubernetes API
リファレンス](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/)
が、Kubernetes で作成出来る全てのオブジェクトに関する spec のフォーマットを探す
のに役立ちます。例えば、`Pod`オブジェクトに関する`spec`のフォーマット
は[こちら](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podspec-v1-core)
を、また`Deployment`オブジェクトに関する`spec`のフォーマット
は[こちら](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#deploymentspec-v1-apps)
をご確認ください。

{{% /capture %}}

{{% capture whatsnext %}}

- 最も重要、かつ基本的な Kubernetes オブジェクト群を学びましょう、例えば
  、[Pod](/ja/docs/concepts/workloads/pods/pod-overview/)です。

{{% /capture %}}
