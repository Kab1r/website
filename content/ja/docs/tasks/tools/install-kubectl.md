---
title: kubectlのインストールおよびセットアップ
content_template: templates/task
weight: 10
card:
  name: tasks
  weight: 20
  title: Install kubectl
---

{{% capture overview %}} Kubernetes のコマンドラインツールであ
る[kubectl](/docs/user-guide/kubectl/)を使用して、Kubernetes クラスターに対して
コマンドを実行することができます。kubectl によってアプリケーションのデプロイや、
クラスターのリソース管理および検査を行うことができます。kubectl の操作に関する完
全なリストは、[Overview of kubectl](/docs/reference/kubectl/overview/)を参照して
ください。 {{% /capture %}}

{{% capture prerequisites %}} kubectl のバージョンは、クラスターのマイナーバージ
ョンとの差分が 1 つ以内でなければなりません。たとえば、クライアントが v1.2 であ
れば、v1.1、v1.2、v1.3 のマスターで動作するはずです。最新バージョンの kubectl を
使うことで、不測の事態を避けることができるでしょう。 {{% /capture %}}

{{% capture steps %}}

## Linux へ kubectl をインストールする {#install-kubectl-on-linux}

### curl を使用して Linux へ kubectl のバイナリをインストールする

1. 次のコマンドにより、最新リリースをダウンロードしてください:

   ```
   curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
   ```

   特定のバージョンをダウンロードする場合、コマンド
   の`$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)`の
   部分を特定のバージョンに書き換えてください。

   たとえば、Linux へ{{< param "fullversion" >}}のバージョンをダウンロードするに
   は、次のコマンドを入力します:

   ```
   curl -LO https://storage.googleapis.com/kubernetes-release/release/{{< param "fullversion" >}}/bin/linux/amd64/kubectl
   ```

2. kubectl バイナリを実行可能にしてください。

   ```
   chmod +x ./kubectl
   ```

3. バイナリを PATH の中に移動させてください。

   ```
   sudo mv ./kubectl /usr/local/bin/kubectl
   ```

4. インストールしたバージョンが最新であることを確認してください:

   ```
   kubectl version
   ```

### ネイティブなパッケージマネージャーを使用してインストールする

{{< tabs name="kubectl_install" >}}
{{< tab name="Ubuntu、DebianまたはHypriotOS" codelang="bash" >}} sudo apt-get
update && sudo apt-get install -y apt-transport-https curl -s
https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - echo
"deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a
/etc/apt/sources.list.d/kubernetes.list sudo apt-get update sudo apt-get install
-y kubectl {{< /tab >}}
{{< tab name="CentOS、RHELまたはFedora" codelang="bash" >}}cat <<EOF >
/etc/yum.repos.d/kubernetes.repo [kubernetes] name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1 gpgcheck=1 repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg EOF yum install -y
kubectl {{< /tab >}} {{< /tabs >}}

### 他のパッケージマネージャーを使用してインストールする

Ubuntu または[snap](https://snapcraft.io/docs/core/install)パッケージマネージャ
ーをサポートする別の Linux ディストリビューションを使用している場合、kubectl
は[snap](https://snapcraft.io/)アプリケーションとして使用できます。

Linux で[Homebrew](https://docs.brew.sh/Homebrew-on-Linux)パッケージマネージャー
を使用している場合は、kubectl
を[インストール](https://docs.brew.sh/Homebrew-on-Linux#install)することが可能で
す。

{{< tabs name="other_kubectl_install" >}}
{{< tab name="Snap" codelang="bash" >}} sudo snap install kubectl --classic

kubectl version {{< /tab >}} {{< tab name="Homebrew" codelang="bash" >}} brew
install kubectl

kubectl version {{< /tab >}} {{< /tabs >}}

## macOS へ kubectl をインストールする {#install-kubectl-on-macos}

### curl を使用して macOS へ kubectl のバイナリをインストールする

1. 最新リリースをダウンロードしてください:

   ```
   curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl"
   ```

   特定のバージョンをダウンロードする場合、コマンド
   の`$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)`の
   部分を特定のバージョンに書き換えてください。

   たとえば、macOS へ{{< param "fullversion" >}}のバージョンをダウンロードするに
   は、次のコマンドを入力します:

   ```
   curl -LO https://storage.googleapis.com/kubernetes-release/release/{{< param "fullversion" >}}/bin/darwin/amd64/kubectl
   ```

2. kubectl バイナリを実行可能にしてください。

   ```
   chmod +x ./kubectl
   ```

3. バイナリを PATH の中に移動させてください。

   ```
   sudo mv ./kubectl /usr/local/bin/kubectl
   ```

4. インストールしたバージョンが最新であることを確認してください:

   ```
   kubectl version
   ```

### Homebrew を使用して macOS へインストールする

macOS で[Homebrew](https://brew.sh/)パッケージマネージャーを使用していれば
、Homebrew で kubectl をインストールすることもできます。

1. インストールコマンドを実行してください:

   ```
   brew install kubectl
   ```

   または

   ```
   brew install kubernetes-cli
   ```

2. インストールしたバージョンが最新であることを確認してください:

   ```
   kubectl version
   ```

### MacPorts を使用して macOS へインストールする

macOS で[MacPorts](https://macports.org/)パッケージマネージャーを使用していれば
、MacPorts で kubectl をインストールすることもできます。

1. インストールコマンドを実行してください:

   ```
   sudo port selfupdate
   sudo port install kubectl
   ```

2. インストールしたバージョンが最新であることを確認してください:

   ```
   kubectl version
   ```

## Windows へ kubectl をインストールする {#install-kubectl-on-windows}

### curl を使用して Windows へ kubectl のバイナリをインストールする

1.  [こちらのリンク](https://storage.googleapis.com/kubernetes-release/release/{{<
    param "fullversion" >}}/bin/windows/amd64/kubectl.exe)から、最新リリースであ
    る{{< param "fullversion" >}}をダウンロードしてください。

    または、`curl`をインストールされていれば、次のコマンドも使用できます:

    ```
    curl -LO https://storage.googleapis.com/kubernetes-release/release/{{< param "fullversion" >}}/bin/windows/amd64/kubectl.exe
    ```

    最新の安定版を入手する際は（たとえばスクリプトで使用する場合）
    、[https://storage.googleapis.com/kubernetes-release/release/stable.txt](https://storage.googleapis.com/kubernetes-release/release/stable.txt)を
    参照してください。

2.  バイナリを PATH に追加します
3.  `kubectl`のバージョンがダウンロードしたものと同じであることを確認してくださ
    い:

        ```
        kubectl version
        ```

    {{< note >}}
    [Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/#kubernetes)は
    、それ自身のバージョンの`kubectl`を PATH に追加します。Docker Desktop をすで
    にインストールしている場合、Docker Desktop インストーラーによって追加された
    PATH の前に追加するか、Docker Desktop の`kubectl`を削除してください。
    {{< /note >}}

### PSGallery から PowerShell を使用してインストールする

Windows で[Powershell Gallery](https://www.powershellgallery.com/)パッケージマネ
ージャーを使用していれば、Powershell で kubectl をインストールおよびアップデート
することもできます。

1. インストールコマンドを実行してください（必ず`DownloadLocation`を指定してく�
   さい）:

   ```
   Install-Script -Name install-kubectl -Scope CurrentUser -Force
   install-kubectl.ps1 [-DownloadLocation <path>]
   ```

   {{< note >}}`DownloadLocation`を指定しない場合、`kubectl`はユーザの Temp ディ
   レクトリにインストールされます。{{< /note >}}

   インストーラーは`$HOME/.kube`を作成し、設定ファイルを作成します。

2. インストールしたバージョンが最新であることを確認してください:

   ```
   kubectl version
   ```

   {{< note >}}アップデートする際は、手順 1 に示した 2 つのコマンドを再実行して
   ください。{{< /note >}}

### Chocolatey または Scoop を使用して Windows へインストールする

Windows へ kubectl をインストールするために
、[Chocolatey](https://chocolatey.org)パッケージマネージャー
や[Scoop](https://scoop.sh)コマンドラインインストーラーを使用することもできます
。 {{< tabs name="kubectl_win_install" >}} {{% tab name="choco" %}}

    choco install kubernetes-cli

{{% /tab %}} {{% tab name="scoop" %}}

    scoop install kubectl

{{% /tab %}} {{< /tabs >}}

2. インストールしたバージョンが最新であることを確認してください:

   ```
   kubectl version
   ```

3. ホームディレクトリへ移動してください:

   ```
   cd %USERPROFILE%
   ```

4. `.kube`ディレクトリを作成してください:

   ```
   mkdir .kube
   ```

5. 作成した`.kube`ディレクトリへ移動してください:

   ```
   cd .kube
   ```

6. リモートの Kubernetes クラスターを使うために、kubectl を設定してください:

   ```
   New-Item config -type file
   ```

   {{< note >}}Notepad などの選択したテキストエディターから設定ファイルを編集し
   てください。{{< /note >}}

## Google Cloud SDK の一部としてダウンロードする

Google Cloud SDK の一部として、kubectl をインストールすることもできます。

1. [Google Cloud SDK](https://cloud.google.com/sdk/)をインストールしてください。
2. `kubectl`のインストールコマンドを実行してください:

   ```
   gcloud components install kubectl
   ```

3. インストールしたバージョンが最新であることを確認してください:

   ```
   kubectl version
   ```

## kubectl の設定を検証する

kubectl が Kubernetes クラスターを探索し接続するために
、[kubeconfig ファイル](/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)が
必要になります。これは
、[kube-up.sh](https://github.com/kubernetes/kubernetes/blob/master/cluster/kube-up.sh)に
よりクラスターを作成した際や、Minikube クラスターを正常にデプロイした際に自動生
成されます。デフォルトでは、kubectl の設定は`~/.kube/config`に格納されています。

クラスターの状態を取得し、kubectl が適切に設定されていることを確認してください:

```shell
kubectl cluster-info
```

URL のレスポンスが表示されている場合は、kubectl はクラスターに接続するよう正しく
設定されています。

以下のようなメッセージが表示されている場合は、kubectl は正しく設定されていないか
、Kubernetes クラスターに接続できていません。

```shell
The connection to the server <server-name:port> was refused - did you specify the right host or port?
```

たとえば、ラップトップ上（ローカル環境）で Kubernetes クラスターを起動するような
場合、Minikube などのツールを最初にインストールしてから、上記のコマンドを再実行
する必要があります。

kubectl cluster-info が URL レスポンスを返したにもかかわらずクラスターにアクセス
できない場合は、次のコマンドで設定が正しいことを確認してください:

```shell
kubectl cluster-info dump
```

## kubectl の任意の設定

### シェルの自動補完を有効にする

kubectl は Bash および Zsh の自動補完を提供しています。これにより、入力を大幅に
削減することができます。

以下に Bash（Linux と macOS の違いも含む）および Zsh の自動補完の設定手順を示し
ます。

{{< tabs name="kubectl_autocompletion" >}}

{{% tab name="LinuxでのBash" %}}

### はじめに

Bash における kubectl の補完スクリプトは`kubectl completion bash`コマンドで生成
できます。シェル内で補完スクリプトを source することで kubectl の自動補完が有効
になります。

ただし、補完スクリプト
は[**bash-completion**](https://github.com/scop/bash-completion)に依存しているた
め、このソフトウェアを最初にインストールしておく必要があります
（`type _init_completion`を実行することで、bash-completion がすでにインストール
されていることを確認できます）。

### bash-completion をインストールする

bash-completion は多くのパッケージマネージャーから提供されています
（[こちら](https://github.com/scop/bash-completion#installation)を参照してくださ
い）。`apt-get install bash-completion`または`yum install bash-completion`などで
インストールできます。

上記のコマンドで bash-completion の主要スクリプトであ
る`/usr/share/bash-completion/bash_completion`が作成されます。パッケージマネージ
ャーによっては、このファイルを`~/.bashrc`にて手動で source する必要があります。

これを調べるには、シェルをリロードしてから`type _init_completion`を実行してく�
さい。コマンドが成功していればすでに設定済みです。そうでなければ、`~/.bashrc`に
以下を追記してください:

```shell
source /usr/share/bash-completion/bash_completion
```

シェルをリロードし、`type _init_completion`を実行して bash-completion が正しくイ
ンストールされていることを検証してください。

### kubectl の自動補完を有効にする

すべてのシェルセッションにて kubectl の補完スクリプトを source できるようにしな
ければなりません。これを行うには 2 つの方法があります:

- 補完スクリプトを`~/.bashrc`内で source してください:

  ```shell
  echo 'source <(kubectl completion bash)' >>~/.bashrc
  ```

- 補完スクリプトを`/etc/bash_completion.d`ディレクトリに追加してください:

  ```shell
  kubectl completion bash >/etc/bash_completion.d/kubectl
  ```

- kubectl にエイリアスを張っている場合は、以下のようにシェルの補完を拡張して使う
  ことができます:

  ```shell
  echo 'alias k=kubectl' >>~/.bashrc
  echo 'complete -F __start_kubectl k' >>~/.bashrc
  ```

{{< note >}} bash-completion は`/etc/bash_completion.d`内のすべての補完スクリプ
トを source します。 {{< /note >}}

どちらも同様の手法です。シェルをリロードしたあとに、kubectl の自動補完が機能する
はずです。

{{% /tab %}}

{{% tab name="macOSでのBash" %}}

### はじめに

Bash における kubectl の補完スクリプトは`kubectl completion bash`コマンドで生成
できます。シェル内で補完スクリプトを source することで kubectl の自動補完が有効
になります。

ただし、補完スクリプト
は[**bash-completion**](https://github.com/scop/bash-completion)に依存しているた
め、事前にインストールする必要があります。

{{< warning>}} bash-completion には v1 と v2 のバージョンがあり、v1 は Bash
3.2（macOS のデフォルト）用で、v2 は Bash 4.1 以降向けです。kubectl の補完スクリ
プトは bash-completion の v1 と Bash 3.2 では正しく**動作しませ
ん**。**bash-completion v2**および**Bash 4.1**が必要になります。したがって
、macOS で正常に kubectl の補完を使用するには、Bash 4.1 以降をインストールする必
要があります([_手順_](https://itnext.io/upgrading-bash-on-macos-7138bd1066ba))。
以下の手順では、Bash4.1 以降（Bash のバージョンが 4.1 またはそれより新しいことを
指します）を使用することを前提とします。 {{< /warning >}}

### bash-completion をインストールする

{{< note >}} 前述のとおり、この手順では Bash 4.1 以降であることが前提のため
、bash-completion v2 をインストールすることになります（これとは逆に、Bash 3.2 お
よび bash-completion v1 の場合では kubectl の補完は動作しません）。
{{< /note >}}

`type _init_completion`を実行することで、bash-completion がすでにインストールさ
れていることを確認できます。ない場合は、Homebrew を使用してインストールすること
もできます:

```shell
brew install bash-completion@2
```

このコマンドの出力で示されたように、`~/.bashrc`に以下を追記してください:

```shell
export BASH_COMPLETION_COMPAT_DIR="/usr/local/etc/bash_completion.d"
[[ -r "/usr/local/etc/profile.d/bash_completion.sh" ]] && . "/usr/local/etc/profile.d/bash_completion.sh"
```

シェルをリロードし、`type _init_completion`を実行して bash-completion v2 が正し
くインストールされていることを検証してください。

### kubectl の自動補完を有効にする

すべてのシェルセッションにて kubectl の補完スクリプトを source できるようにしな
ければなりません。これを行うには複数の方法があります:

- 補完スクリプトを`~/.bashrc`内で source する:

  ```shell
  echo 'source <(kubectl completion bash)' >>~/.bashrc
  ```

- 補完スクリプトを`/usr/local/etc/bash_completion.d`ディレクトリに追加する:

  ```shell
  kubectl completion bash >/usr/local/etc/bash_completion.d/kubectl
  ```

- kubectl にエイリアスを張っている場合は、以下のようにシェルの補完を拡張して使う
  ことができます:

  ```shell
  echo 'alias k=kubectl' >>~/.bashrc
  echo 'complete -F __start_kubectl k' >>~/.bashrc
  ```

- kubectl を Homwbrew でインストールした場合
  （[前述](#homebrewを使用してmacosへインストールする)のとおり）、kubectl の補完
  スクリプトはすでに`/usr/local/etc/bash_completion.d/kubectl`に格納されているで
  しょう。この場合、なにも操作する必要はありません。

{{< note >}} Homebrew でインストールした bash-completion v2
は`BASH_COMPLETION_COMPAT_DIR`ディレクトリ内のすべてのファイルを source するため
、後者の 2 つの方法が機能します。 {{< /note >}}

どの場合でも、シェルをリロードしたあとに、kubectl の自動補完が機能するはずです。
{{% /tab %}}

{{% tab name="Zsh" %}}

Zsh における kubectl の補完スクリプトは`kubectl completion zsh`コマンドで生成で
きます。シェル内で補完スクリプトを source することで kubectl の自動補完が有効に
なります。

すべてのシェルセッションで使用するには、`~/.bashrc`に以下を追記してください:

```shell
source <(kubectl completion zsh)
```

kubectl にエイリアスを張っている場合は、以下のようにシェルの補完を拡張して使うこ
とができます:

```shell
echo 'alias k=kubectl' >>~/.zshrc
echo 'complete -F __start_kubectl k' >>~/.zshrc
```

シェルをリロードしたあとに、kubectl の自動補完が機能するはずです。

`complete:13: command not found: compdef`のようなエラーが出力された場合は、以下
を`~/.zshrc`の先頭に追記してください:

```shell
autoload -Uz compinit
compinit
```

{{% /tab %}} {{< /tabs >}}

{{% /capture %}}

{{% capture whatsnext %}}

- [Minikube をインストールする](/ja/docs/tasks/tools/install-minikube/)
- クラスターの作成に関する詳細を[スタートガイド](/docs/setup/)で確認する
- [アプリケーションを起動して公開する方法を学ぶ](/docs/tasks/access-application-cluster/service-access-application-cluster/)
- あなたが作成していないクラスターにアクセスする必要がある場合は
  、[クラスターアクセスドキュメントの共有](/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)を
  参照してください
- [kubectl リファレンスドキュメント](/docs/reference/kubectl/kubectl/)を参照する
  {{% /capture %}}
