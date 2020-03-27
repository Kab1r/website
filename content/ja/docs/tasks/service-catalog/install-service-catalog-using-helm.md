---
title: Helmを使用したサービスカタログのインストール
content_template: templates/task
---

{{% capture overview %}}
{{< glossary_definition term_id="service-catalog" length="all" prepend="サービスカタログは" >}}

[Helm](https://helm.sh/)を使用して Kubernetes クラスターにサービスカタログをイン
ストールします。手順の最新情報
は[kubernetes-sigs/service-catalog](https://github.com/kubernetes-sigs/service-catalog/blob/master/docs/install.md)リ
ポジトリーを参照してください。

{{% /capture %}}

{{% capture prerequisites %}}

- [サービスカタログ](/docs/concepts/service-catalog/)の基本概念を理解してくださ
  い。
- サービスカタログを使用するには、Kubernetes クラスターのバージョンが 1.7 以降で
  ある必要があります。
- Kubernetes クラスターのクラスター DNS を有効化する必要があります。
  - クラウド上の Kubernetes クラスター、また
    は{{< glossary_tooltip text="Minikube" term_id="minikube" >}}を使用している
    場合、クラスター DNS はすでに有効化されています。
  - `hack/local-up-cluster.sh`を使用している場合は、環境変
    数`KUBE_ENABLE_CLUSTER_DNS`が設定されていることを確認し、インストールスクリ
    プトを実行してください。
- [kubectl のインストールおよびセットアップ](/ja/docs/tasks/tools/install-kubectl/)を
  参考に、v1.7 以降の kubectl をインストールし、設定を行ってください。
- v2.7.0 以降の[Helm](http://helm.sh/)をインストールしてください。
  - [Helm install instructions](https://helm.sh/docs/intro/install/)を参考にして
    ください。
  - 上記のバージョンの Helm をすでにインストールしている場合は、`helm init`を実
    行し、Helm のサーバーサイドコンポーネントである Tiller をインストールしてく
    ださい。

{{% /capture %}}

{{% capture steps %}}

## Helm リポジトリーにサービスカタログを追�

Helm をインストールし、以下のコマンドを実行することでローカルマシン
に*service-catalog*の Helm リポジトリーを追加します。

```shell
helm repo add svc-cat https://svc-catalog-charts.storage.googleapis.com
```

以下のコマンドを実行し、インストールに成功していることを確認します。

```shell
helm search service-catalog
```

インストールが成功していれば、出力は以下のようになります:

```
NAME                	CHART VERSION	APP VERSION	DESCRIPTION
svc-cat/catalog     	0.2.1        	           	service-catalog API server and controller-manager helm chart
svc-cat/catalog-v0.2	0.2.2        	           	service-catalog API server and controller-manager helm chart
```

## RBAC の有効化

Kubernetes クラスターの RBAC を有効化することで、Tiller Pod に`cluster-admin`ア
クセスを持たせます。

v0.25 以前の Minikube を使用している場合は、明示的に RBAC を有効化して起動する必
要があります:

```shell
minikube start --extra-config=apiserver.Authorization.Mode=RBAC
```

v0.26 以降の Minikube を使用している場合は、以下のコマンドを実行してください。

```shell
minikube start
```

v0.26 以降の Minikube を使用している場合、`--extra-config`を指定しないでください
。このフラグは--extra-config=apiserver.authorization-mode を指定するものに変更さ
れており、現在 Minikube ではデフォルトで RBAC が有効化されています。古いフラグを
指定すると、スタートコマンドが応答しなくなることがあります。

`hack/local-up-cluster.sh`を使用している場合、環境変数`AUTHORIZATION_MODE`を以下
の値に設定してください:

```
AUTHORIZATION_MODE=Node,RBAC hack/local-up-cluster.sh -O
```

`helm init`は、デフォルトで`kube-system`の namespace に Tiller Pod をインストー
ルし、Tiller は`default`の ServiceAccount を使用するように設定されています。

{{< note >}} `helm init`を実行する際に`--tiller-namespace`また
は`--service-account`のフラグを使用する場合、以下のコマンド
の`--serviceaccount`フラグには適切な namespace と ServiceAccount を指定する必要
があります。 {{< /note >}}

Tiller に`cluster-admin`アクセスを設定する場合:

```shell
kubectl create clusterrolebinding tiller-cluster-admin \
    --clusterrole=cluster-admin \
    --serviceaccount=kube-system:default
```

## Kubernetes クラスターにサービスカタログをインストール

以下のコマンドを使用して、Helm リポジトリーの root からサービスカタログをインス
トールします:

{{< tabs name="helm-versions" >}} {{% tab name="Helm バージョン3" %}}

```shell
helm install catalog svc-cat/catalog --namespace catalog
```

{{% /tab %}} {{% tab name="Helm バージョン2" %}}

```shell
helm install svc-cat/catalog --name catalog --namespace catalog
```

{{% /tab %}} {{< /tabs >}} {{% /capture %}}

{{% capture whatsnext %}}

- [sample service brokers](https://github.com/openservicebrokerapi/servicebroker/blob/master/gettingStarted.md#sample-service-brokers)
- [kubernetes-sigs/service-catalog](https://github.com/kubernetes-sigs/service-catalog)

{{% /capture %}}
