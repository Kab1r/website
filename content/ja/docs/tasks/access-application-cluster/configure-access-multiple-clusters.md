---
title: 複数のクラスターへのアクセスを設定する
content_template: templates/task
weight: 30
card:
  name: tasks
  weight: 40
---

{{% capture overview %}}

ここでは、設定ファイルを使って複数のクラスターにアクセスする方法を紹介します。ク
ラスター、ユーザー、context の情報を一つ以上の設定ファイルにまとめることで
、`kubectl config use-context`のコマンドを使ってクラスターを素早く切り替えること
ができます。

{{< note >}} クラスターへのアクセスを設定するファイルを、_kubeconfig_ ファイルと
呼ぶことがあります。これは設定ファイルの一般的な呼び方です。`kubeconfig`という名
前のファイルが存在するわけではありません。 {{< /note >}}

{{% /capture %}}

{{% capture prerequisites %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

{{% /capture %}}

{{% capture steps %}}

## クラスター、ユーザー、context を設定する

 例として、開発用のクラスターが一つ、実験用のクラスターが一つ、計二つのクラスタ
ーが存在する場合を考えます。`development`と呼ばれる開発用のクラスター内では、フ
ロントエンドの開発者は`frontend`という namespace 内で、ストレージの開発者
は`storage`という namespace 内で作業をします。`scratch`と呼ばれる実験用のクラス
ター内では、開発者はデフォルトの namespace で作業をするか、状況に応じて追加の
namespace を作成します。開発用のクラスターは証明書を通しての認証を必要とします。
実験用のクラスターはユーザーネームとパスワードを通しての認証を必要とします。

`config-exercise`というディレクトリを作成してください。`config-exercise`ディレク
トリ内に 、以下を含む`config-demo`というファイルを作成してください:

```shell
apiVersion: v1
kind: Config
preferences: {}

clusters:
- cluster:
  name: development
- cluster:
  name: scratch

users:
- name: developer
- name: experimenter

contexts:
- context:
  name: dev-frontend
- context:
  name: dev-storage
- context:
  name: exp-scratch
```

設定ファイルには、クラスター、ユーザー、context の情報が含まれています。上記
の`config-demo`設定ファイルには、二つのクラスター、二人のユーザー、三つの
context の情報が含まれています。

`config-exercise`ディレクトリに移動してください。クラスター情報を設定ファイルに
追加するために、以下のコマンドを  実行してください:

```shell
kubectl config --kubeconfig=config-demo set-cluster development --server=https://1.2.3.4 --certificate-authority=fake-ca-file
kubectl config --kubeconfig=config-demo set-cluster scratch --server=https://5.6.7.8 --insecure-skip-tls-verify
```

ユーザー情報を設定ファイルに追加してください:

```shell
kubectl config --kubeconfig=config-demo set-credentials developer --client-certificate=fake-cert-file --client-key=fake-key-seefile
kubectl config --kubeconfig=config-demo set-credentials experimenter --username=exp --password=some-password
```

{{< note >}} `kubectl config unset users.<name>`を実行すると、ユーザーを削除する
ことができます。 {{< /note >}}

context 情報を設定ファイルに追加してください:

```shell
kubectl config --kubeconfig=config-demo set-context dev-frontend --cluster=development --namespace=frontend --user=developer
kubectl config --kubeconfig=config-demo set-context dev-storage --cluster=development --namespace=storage --user=developer
kubectl config --kubeconfig=config-demo set-context exp-scratch --cluster=scratch --namespace=default --user=experimenter
```

追加した情報を確認するために、`config-demo`ファイルを開いてください
。`config-demo`ファイルを開く代わりに、`config view`のコマンドを使うこともできま
す。

```shell
kubectl config --kubeconfig=config-demo view
```

出力には、二つのクラスター、二人のユーザー、三つの context が表示されます:

```shell
apiVersion: v1
clusters:
- cluster:
    certificate-authority: fake-ca-file
    server: https://1.2.3.4
  name: development
- cluster:
    insecure-skip-tls-verify: true
    server: https://5.6.7.8
  name: scratch
contexts:
- context:
    cluster: development
    namespace: frontend
    user: developer
  name: dev-frontend
- context:
    cluster: development
    namespace: storage
    user: developer
  name: dev-storage
- context:
    cluster: scratch
    namespace: default
    user: experimenter
  name: exp-scratch
current-context: ""
kind: Config
preferences: {}
users:
- name: developer
  user:
    client-certificate: fake-cert-file
    client-key: fake-key-file
- name: experimenter
  user:
    password: some-password
    username: exp
```

上記の`fake-ca-file`、`fake-cert-file`、`fake-key-file`は、証明書ファイルの実際
のパスのプレースホルダーです。環境内にある証明書ファイルの実際のパスに変更してく
ださい。

証明書ファイルのパスの代わりに base64 にエンコードされたデータを使用したい場合は
、キーに`-data`の接尾辞を加えてください。例えば
、`certificate-authority-data`、`client-certificate-data`、`client-key-data`とで
きます。

それぞれの context は、クラスター、ユーザー、namespace の三つ組からなっています
。例えば、`dev-frontend`context は、`developer`ユーザーの認証情報を使っ
て`development`クラスターの`frontend`namespace へのアクセスを意味しています。

現在の context を設定してください:

```shell
kubectl config --kubeconfig=config-demo use-context dev-frontend
```

これ以降実行される`kubectl`コマンドは、`dev-frontend`context に設定されたクラス
ターと namespace に適用されます。また、`dev-frontend`context に設定されたユーザ
ーの認証情報を使用します。

現在の context の設定情報のみを確認するには、`--minify`フラグを使用してください
。

```shell
kubectl config --kubeconfig=config-demo view --minify
```

 出力には、`dev-frontend`context の設定情報が表示されます:

```shell
apiVersion: v1
clusters:
- cluster:
    certificate-authority: fake-ca-file
    server: https://1.2.3.4
  name: development
contexts:
- context:
    cluster: development
    namespace: frontend
    user: developer
  name: dev-frontend
current-context: dev-frontend
kind: Config
preferences: {}
users:
- name: developer
  user:
    client-certificate: fake-cert-file
    client-key: fake-key-file
```

今度は、実験用のクラスター内でしばらく作業する場合を考えます。

現在の context を`exp-scratch`に切り替えてください:

```shell
kubectl config --kubeconfig=config-demo use-context exp-scratch
```

これ以降実行される`kubectl`コマンドは、`scratch`クラスター内のデフォルト
namespace に適用されます。また、`exp-scratch`context に設定されたユーザーの認証
情報を使用します。

新しく切り替えた`exp-scratch`context の設定を確認してください。

```shell
kubectl config --kubeconfig=config-demo view --minify
```

最後に、`development`クラスター内の`storage`namespace でしばらく作業する場合を考
えます。

現在の context を`dev-storage`に切り替えてください:

```shell
kubectl config --kubeconfig=config-demo use-context dev-storage
```

新しく切り替えた`dev-storage`context の設定を確認してください。

```shell
kubectl config --kubeconfig=config-demo view --minify
```

## 二つ目の設定ファイルを作成する

`config-exercise`ディレクトリ内に、以下を含む`config-demo-2`というファイルを作成
してください:

```shell
apiVersion: v1
kind: Config
preferences: {}

contexts:
- context:
    cluster: development
    namespace: ramp
    user: developer
  name: dev-ramp-up
```

上記の設定ファイルは、`dev-ramp-up`という context を表します。

## KUBECONFIG 環境変数を設定する

`KUBECONFIG`という環境変数が存在するかを確認してください。もし存在する場合は、後
で復元できるようにバックアップしてください。例えば:

### Linux

```shell
export KUBECONFIG_SAVED=$KUBECONFIG
```

### Windows PowerShell

```shell
$Env:KUBECONFIG_SAVED=$ENV:KUBECONFIG
```

`KUBECONFIG`環境変数は、設定ファイルのパスのリストです。リスト内のパスは Linux
と Mac ではコロンで区切られ、Windows ではセミコロンで区切られます
。`KUBECONFIG`環境変数が存在する場合は、リスト内の設定ファイルの内容を確認してく
ださい。

一時的に`KUBECONFIG`環境変数に以下の二つのパスを追加してください。例えば:<br>

### Linux

```shell
export KUBECONFIG=$KUBECONFIG:config-demo:config-demo-2
```

### Windows PowerShell

```shell
$Env:KUBECONFIG=("config-demo;config-demo-2")
```

`config-exercise`ディレクトリ内から、以下のコマンドを実行してください:

```shell
kubectl config view
```

出力には、`KUBECONFIG`環境変数に含まれる全てのファイルの情報がまとめて表示されま
す。`config-demo-2`ファイルに設定された`dev-ramp-up`context の情報と
、`config-demo`ファイルに設定された三つの context の情報がまとめてあることに注目
してください:

```shell
contexts:
- context:
    cluster: development
    namespace: frontend
    user: developer
  name: dev-frontend
- context:
    cluster: development
    namespace: ramp
    user: developer
  name: dev-ramp-up
- context:
    cluster: development
    namespace: storage
    user: developer
  name: dev-storage
- context:
    cluster: scratch
    namespace: default
    user: experimenter
  name: exp-scratch
```

kubeconfig ファイルに関するさらなる情報を参照するには
、[kubeconfig ファイルを使ってクラスターへのアクセスを管理する](/docs/concepts/configuration/organize-cluster-access-kubeconfig/)を
参照してください。

## \$HOME/.kube ディレクトリの内容を確認する

既にクラスターを所持していて、`kubectl`を使ってクラスターを操作できる場合は
、`$HOME/.kube`ディレクトリ内に`config`というファイルが存在する可能性が高いです
。

`$HOME/.kube`に移動して、そこに存在するファイルを確認してください。`config`とい
う設定ファイルが存在するはずです。他の設定ファイルも存在する可能性があります。全
てのファイルの中身を確認してください。

## \$HOME/.kube/config を KUBECONFIG 環境変数に追加する

もし`$HOME/.kube/config`ファイルが存在していて、既に`KUBECONFIG`環境変数に追加さ
れていない場合は、`KUBECONFIG`環境変数に追加してください。例えば:

### Linux

```shell
export KUBECONFIG=$KUBECONFIG:$HOME/.kube/config
```

### Windows Powershell

```shell
$Env:KUBECONFIG=($Env:KUBECONFIG;$HOME/.kube/config)
```

`KUBECONFIG`環境変数内のファイルからまとめられた設定情報を確認してください
。`config-exercise`ディレクトリ内から、以下のコマンドを実行してください:

```shell
kubectl config view
```

## クリーンアップ

`KUBECONFIG`環境変数を元に戻してください。例えば:

Linux:

```shell
export KUBECONFIG=$KUBECONFIG_SAVED
```

Windows PowerShell

```shell
$Env:KUBECONFIG=$ENV:KUBECONFIG_SAVED
```

{{% /capture %}}

{{% capture whatsnext %}}

- [kubeconfig ファイルを使ってクラスターへのアクセスを管理する](/docs/concepts/configuration/organize-cluster-access-kubeconfig/)
- [kubectl config](/docs/reference/generated/kubectl/kubectl-commands#config)

{{% /capture %}}
