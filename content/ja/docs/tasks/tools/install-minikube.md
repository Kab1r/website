---
title: Minikubeのインストール
content_template: templates/task
weight: 20
card:
  name: tasks
  weight: 10
---

{{% capture overview %}}

このページでは[Minikube](/ja/docs/tutorials/hello-minikube)のインストール方法を
説明し、コンピューターの仮想マシン上で単一ノードの Kubernetes クラスターを実行し
ます。

{{% /capture %}}

{{% capture prerequisites %}}

{{< tabs name="minikube_before_you_begin" >}} {{% tab name="Linux" %}} Linux で
仮想化がサポートされているかどうかを確認するには、次のコマンドを実行して、出力が
空でないことを確認します:

```
grep -E --color 'vmx|svm' /proc/cpuinfo
```

{{% /tab %}}

{{% tab name="macOS" %}} 仮想化が macOS でサポートされているかどうかを確認するに
は、ターミナルで次のコマンドを実行します。

```
sysctl -a | grep -E --color 'machdep.cpu.features|VMX'
```

出力に`VMX`が表示されている場合（色付けされているはずです）、VT-x 機能がマシンで
有効になっています。 {{% /tab %}}

{{% tab name="Windows" %}} Windows 8 以降で仮想化がサポートされているかどうかを
確認するには、Windows ターミナルまたはコマンドプロンプトで次のコマンドを実行しま
す。

```
systeminfo
```

次の出力が表示される場合、仮想化は Windows でサポートされています。

```
Hyper-V Requirements:     VM Monitor Mode Extensions: Yes
                          Virtualization Enabled In Firmware: Yes
                          Second Level Address Translation: Yes
                          Data Execution Prevention Available: Yes
```

次の出力が表示される場合、システムにはすでに Hypervisor がインストールされており
、次の手順をスキップできます。

```
Hyper-V Requirements:     A hypervisor has been detected. Features required for Hyper-V will not be displayed.
```

{{% /tab %}} {{< /tabs >}}

{{% /capture %}}

{{% capture steps %}}

# minikube のインストール

{{< tabs name="tab_with_md" >}} {{% tab name="Linux" %}}

### kubectl のインストール

kubectl がインストールされていることを確認してください。
[kubectl のインストールとセットアップ](/ja/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux)の
指示に従って kubectl をインストールできます。

### ハイパーバイザーのインストール

ハイパーバイザーがまだインストールされていない場合は、これらのいずれかをインスト
ールしてください:

• [KVM](https://www.linux-kvm.org/)、ただし QEMU も使っているもの

• [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

{{< note >}} minikube は、VM ではなくホストで Kubernetes コンポーネントを実行す
る`--vm-driver=none`オプションもサポートしています。このドライバーを使用するには
、[Docker](https://www.docker.com/products/docker-desktop)と Linux 環境が必要で
すが、ハイパーバイザーは不要です。 none ドライバーを使用する場合は
、[Docker](https://www.docker.com/products/docker-desktop)から docker の apt イ
ンストールを使用することをおすすめします。 docker の snap インストールは
、minikube では機能しません。 {{< /note >}}

### パッケージを利用した Minikube のインストール

Minikube の*Experimental*パッケージが利用可能です。 GitHub の Minikube
の[リリース](https://github.com/kubernetes/minikube/releases)ページから
Linux(AMD64)パッケージを見つけることができます。

Linux のディストリビューションのパッケージツールを使用して、適切なパッケージをイ
ンストールしてください。

### 直接ダウンロードによる Minikube のインストール

パッケージ経由でインストールしない場合は、スタンドアロンバイナリをダウンロードし
て使用できます。

```shell
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
```

Minikube 実行可能バイナリをパスに追加する簡単な方法を次に示します:

```shell
sudo mkdir -p /usr/local/bin/
sudo install minikube /usr/local/bin/
```

{{% /tab %}} {{% tab name="macOS" %}}

### kubectl のインストール

kubectl がインストールされていることを確認してください。
[kubectl のインストールとセットアップ](/ja/docs/tasks/tools/install-kubectl/#install-kubectl-on-macos)の
指示に従って kubectl をインストールできます。

### ハイパーバイザーのインストール

ハイパーバイザーがまだインストールされていない場合は、これらのいずれかをインスト
ールしてください:

• [HyperKit](https://github.com/moby/hyperkit)

• [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

• [VMware Fusion](https://www.vmware.com/products/fusion)

### Minikube のインストール

[Homebrew](https://brew.sh)を使うことで macOS に Minikube を簡単にインストールで
きます:

```shell
brew install minikube
```

スタンドアロンのバイナリをダウンロードして、macOS にインストールすることもできま
す:

```shell
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64 \
  && chmod +x minikube
```

Minikube 実行可能バイナリをパスに追加する簡単な方法を次に示します:

```shell
sudo mv minikube /usr/local/bin
```

{{% /tab %}} {{% tab name="Windows" %}}

### kubectl のインストール

kubectl がインストールされていることを確認してください。
[kubectl のインストールとセットアップ](/ja/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows)の
指示に従って kubectl をインストールできます。

### ハイパーバイザーのインストール

ハイパーバイザーがまだインストールされていない場合は、これらのいずれかをインスト
ールしてください:

•
[Hyper-V](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install)

• [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

{{< note >}} Hyper-V は、Windows 10 Enterprise、Windows 10 Professional、Windows
10 Education の 3 つのバージョンの Windows 10 で実行できます。 {{< /note >}}

### Chocolatey を使用した Minikube のインストール

[Chocolatey](https://chocolatey.org/)を使うことで Windows に Minikube を簡単にイ
ンストールできます(管理者権限で実行する必要があります)。

```shell
choco install minikube
```

Minikube のインストールが終わったら、現在の CLI のセッションを終了して再起動しま
す。Minikube は自動的にパスに追加されます。

### インストーラーを使用した Minikube のインストール

[Windows インストーラー](https://docs.microsoft.com/en-us/windows/desktop/msi/windows-installer-portal)を
使用して Windows に Minikube を手動でインストールするには
、[`minikube-installer.exe`](https://github.com/kubernetes/minikube/releases/latest/download/minikube-installer.exe)を
ダウンロードしてインストーラーを実行します。

### 直接ダウンロードによる Minikube のインストール

Windows に Minikube を手動でインストールするには
、[`minikube-windows-amd64`](https://github.com/kubernetes/minikube/releases/latest)を
ダウンロードし、名前を`minikube.exe`に変更して、パスに追加します。

{{% /tab %}} {{< /tabs >}}

{{% /capture %}}

{{% capture whatsnext %}}

- [Minikube を使ってローカルで Kubernetes を実行する](/ja/docs/setup/learning-environment/minikube/)

{{% /capture %}}

## ローカル状態のクリーンアップ {#cleanup-local-state}

もし以前に　 Minikube をインストールしていたら、以下のコマンドを実行します。

```shell
minikube start
```

`minikube start`はエラーを返します。

```shell
machine does not exist
```

minikube のローカル状態をクリアする必要があります:

```shell
minikube delete
```
