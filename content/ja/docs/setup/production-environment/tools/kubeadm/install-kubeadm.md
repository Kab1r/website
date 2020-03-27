---
title: kubeadmのインストール
content_template: templates/task
weight: 20
card:
  name: setup
  weight: 20
  title: kubeadmセットアップツールのインストール
---

{{% capture overview %}}

<img src="https://raw.githubusercontent.com/kubernetes/kubeadm/master/logos/stacked/color/kubeadm-stacked-color.png" align="right" width="150px">
このページでは`kubeadm`コマンドをインストールする方法を示します。このインストール処理実行後にkubeadmを使用してクラスターを作成する方法については、[kubeadmを使用したシングルマスタークラスターの作成](/ja/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)を参照してください。

{{% /capture %}}

{{% capture prerequisites %}}

- 次のいずれかが動作しているマシンが必要です
  - Ubuntu 16.04+
  - Debian 9+
  - CentOS 7
  - Red Hat Enterprise Linux (RHEL) 7
  - Fedora 25+
  - HypriotOS v1.0.1+
  - Container Linux (tested with 1800.6.0)
- 1 台あたり 2GB 以上のメモリ(2GB の場合、アプリ用のスペースはほとんどありません
  )
- 2 コア以上の CPU
- クラスター内のすべてのマシン間で通信可能なネットワーク(パブリックネットワーク
  でもプライベートネットワークでも構いません)
- ユニークな hostname、MAC アドレス、と product_uuid が各ノードに必要です。詳細
  は[ここ](#MACアドレスとproduct_uuidが全てのノードでユニークであることの検証)を
  参照してください。
- マシン内の特定のポートが開いていること。詳細は[ここ](#必須ポートの確認)を参照
  してください。
- Swap がオフであること。kubelet が正常に動作するためには swap は**必ず**オフで
  なければなりません。

{{% /capture %}}

{{% capture steps %}}

## MAC アドレスと product_uuid が全てのノードでユニークであることの検証

- ネットワークインターフェースの MAC アドレスは`ip link`もしくは`ifconfig -a`コ
  マンドで取得できます。
- product_uuid は`sudo cat /sys/class/dmi/id/product_uuid`コマンドで確認できます
  。

ハードウェアデバイスではユニークなアドレスが割り当てられる可能性が非常に高いです
が、VM では同じになることがあります。Kubernetes はこれらの値を使用して、クラスタ
ー内のノードを一意に識別します。これらの値が各ノードに固有ではない場合、インスト
ール処理が[失敗](https://github.com/kubernetes/kubeadm/issues/31)することもあり
ます。

## ネットワークアダプタの確認

複数のネットワークアダプターがあり、Kubernetes コンポーネントにデフォルトで到達
できない場合、IP ルートを追加して、Kubernetes クラスターのアドレスが適切なアダプ
ターを経由するように設定することをお勧めします。

## iptables が nftables バックエンドを使用しないようにする

Linux では、カーネルの iptables サブシステムの最新の代替品として nftables が利用
できます。`iptables`ツールは互換性レイヤーとして機能し、iptables のように動作し
ますが、実際には nftables を設定します。この nftables バックエンドは現在の
kubeadm パッケージと互換性がありません。(ファイアウォールルールが重複し
、`kube-proxy`を破壊するためです。)

もしあなたのシステムの`iptables`ツールが nftables バックエンドを使用している場合
、これらの問題を避けるために`iptables`ツールをレガシーモードに切り替える必要があ
ります。これは、少なくとも Debian 10(Buster)、Ubuntu 19.04、Fedora 29、およびこ
れらのディストリビューションの新しいリリースでのデフォルトです。RHEL 8 はレガシ
ーモードへの切り替えをサポートしていないため、現在の kubeadm パッケージと互換性
がありません。

{{< tabs name="iptables_legacy" >}} {{% tab name="Debian or Ubuntu" %}}

```bash
# レガシーバイナリがインストールされていることを確認してください
sudo apt-get install -y iptables arptables ebtables

sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
sudo update-alternatives --set arptables /usr/sbin/arptables-legacy
sudo update-alternatives --set ebtables /usr/sbin/ebtables-legacy
```

{{% /tab %}} {{% tab name="Fedora" %}}

```bash
update-alternatives --set iptables /usr/sbin/iptables-legacy
```

{{% /tab %}} {{< /tabs >}}

## 必須ポートの確認

### マスターノード

| プロトコル | 通信の向き | ポート範囲 | 目的                    | 使用者               |
| ---------- | ---------- | ---------- | ----------------------- | -------------------- |
| TCP        | Inbound    | 6443\*     | Kubernetes API server   | All                  |
| TCP        | Inbound    | 2379-2380  | etcd server client API  | kube-apiserver, etcd |
| TCP        | Inbound    | 10250      | Kubelet API             | Self, Control plane  |
| TCP        | Inbound    | 10251      | kube-scheduler          | Self                 |
| TCP        | Inbound    | 10252      | kube-controller-manager | Self                 |

### ワーカーノード

| プロトコル | 通信の向き | ポート範囲  | 目的                  | 使用者              |
| ---------- | ---------- | ----------- | --------------------- | ------------------- |
| TCP        | Inbound    | 10250       | Kubelet API           | Self, Control plane |
| TCP        | Inbound    | 30000-32767 | NodePort Services\*\* | All                 |

\*\* [NodePort Services](/ja/docs/concepts/services-networking/service/)のデフォ
ルトのポートの範囲

\*の項目は書き換え可能です。そのため、あなたが指定したカスタムポートも開いている
ことを確認する必要があります。

etcd ポートはコントロールプレーンノードに含まれていますが、独自の etcd クラスタ
ーを外部またはカスタムポートでホストすることもできます。

使用する Pod ネットワークプラグイン（以下を参照）のポートも開く必要があります。
これは各 Pod ネットワークプラグインによって異なるため、必要なポートについてはプ
ラグインのドキュメントを参照してください。

## ランタイムのインストール

v1.6.0 以降、Kubernetes はデフォルトで CRI(Container Runtime Interface)の使用を
有効にしています。

また、v1.14.0 以降、kubeadm は既知のドメインソケットのリストをスキャンして
、Linux ノード上のコンテナランタイムを自動的に検出しようとします。検出可能なラン
タイムとソケットパスは、以下の表に記載されています。

| ランタイム | ドメインソケット                |
| ---------- | ------------------------------- |
| Docker     | /var/run/docker.sock            |
| containerd | /run/containerd/containerd.sock |
| CRI-O      | /var/run/crio/crio.sock         |

Docker と containerd の両方が同時に検出された場合、Docker が優先されます。Docker
18.09 には containerd が同梱されており、両方が検出可能であるため、この仕様が必要
です。他の 2 つ以上のランタイムが検出された場合、kubeadm は適切なエラーメッセー
ジで終了します。

Linux 以外のノードでは、デフォルトで使用されるコンテナランタイムは Docker です。

もしコンテナランタイムとして Docker を選択した場合、`kebelet`内に組み込まれ
た`dockershim` CRI が使用されます。

その他の CRI に基づくランタイムでは以下を使用します

- [containerd](https://github.com/containerd/cri) (CRI plugin built into
  containerd)
- [cri-o](https://cri-o.io/)
- [frakti](https://github.com/kubernetes/frakti)

詳細
は[CRI のインストール](/ja/docs/setup/production-environment/container-runtimes/)を
参照してください。

## kubeadm、kubelet、kubectl のインストール

以下のパッケージをマシン上にインストールしてください

- `kubeadm`: クラスターを起動するコマンドです。

- `kubelet`: クラスター内のすべてのマシンで実行されるコンポーネントです。 Pod や
  コンテナの起動などを行います。

- `kubectl`: クラスターにアクセスするためのコマンドラインツールです。

kubeadm は`kubelet`や`kubectl`をインストールまたは管理**しない**ため、kubeadm に
インストールする Kubernetes コントロールプレーンのバージョンと一致させる必要があ
ります。そうしないと、予期しないバグのある動作につながる可能性のあるバージョン差
異(version skew)が発生するリスクがあります。ただし、kubelet とコントロールプレー
ン間のマイナーバージョン差異(minor version skew)は*1 つ*サポートされていますが
、kubelet バージョンが API サーバーのバージョンを超えることはできません。たとえ
ば、1.7.0 を実行する kubelet は 1.8.0 API サーバーと完全に互換性がありますが、そ
の逆はできません。

`kubectl`のインストールに関する詳細情報は
、[kubectl のインストールおよびセットアップ](/ja/docs/tasks/tools/install-kubectl/)を
参照してください。

{{< warning >}} これらの手順はシステムアップグレードによるすべての Kubernetes パ
ッケージの更新を除きます。これは kubeadm と Kubernetes
が[アップグレードにおける特別な注意](docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)を
必要とするからです。 {{</ warning >}}

バージョン差異(version skew)に関しては下記を参照してください。

- Kubernetes
  [Kubernetes バージョンとバージョンスキューサポートポリシー](/ja/docs/setup/release/version-skew-policy/)
- Kubeadm-specific
  [バージョン互換ポリシー](/ja/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#version-skew-policy)

{{< tabs name="k8s_install" >}} {{% tab name="Ubuntu, Debian or HypriotOS" %}}

```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

{{% /tab %}} {{% tab name="CentOS, RHEL or Fedora" %}}

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

# Set SELinux in permissive mode (effectively disabling it)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

systemctl enable --now kubelet
```

**Note:**

- Setting SELinux in permissive mode by running `setenforce 0` and `sed ...`
  effectively disables it. This is required to allow containers to access the
  host filesystem, which is needed by pod networks for example. You have to do
  this until SELinux support is improved in the kubelet.
- Some users on RHEL/CentOS 7 have reported issues with traffic being routed
  incorrectly due to iptables being bypassed. You should ensure
  `net.bridge.bridge-nf-call-iptables` is set to 1 in your `sysctl` config, e.g.

  ```bash
  cat <<EOF > /etc/sysctl.d/k8s.conf
  net.bridge.bridge-nf-call-ip6tables = 1
  net.bridge.bridge-nf-call-iptables = 1
  EOF
  sysctl --system
  ```

- Make sure that the `br_netfilter` module is loaded before this step. This can
  be done by running `lsmod | grep br_netfilter`. To load it explicitly call
  `modprobe br_netfilter`. {{% /tab %}} {{% tab name="Container Linux" %}}
  Install CNI plugins (required for most pod network):

```bash
CNI_VERSION="v0.8.2"
mkdir -p /opt/cni/bin
curl -L "https://github.com/containernetworking/plugins/releases/download/${CNI_VERSION}/cni-plugins-linux-amd64-${CNI_VERSION}.tgz" | tar -C /opt/cni/bin -xz
```

Install crictl (required for kubeadm / Kubelet Container Runtime Interface
(CRI))

```bash
CRICTL_VERSION="v1.16.0"
mkdir -p /opt/bin
curl -L "https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz" | tar -C /opt/bin -xz
```

Install `kubeadm`, `kubelet`, `kubectl` and add a `kubelet` systemd service:

```bash
RELEASE="$(curl -sSL https://dl.k8s.io/release/stable.txt)"

mkdir -p /opt/bin
cd /opt/bin
curl -L --remote-name-all https://storage.googleapis.com/kubernetes-release/release/${RELEASE}/bin/linux/amd64/{kubeadm,kubelet,kubectl}
chmod +x {kubeadm,kubelet,kubectl}

curl -sSL "https://raw.githubusercontent.com/kubernetes/kubernetes/${RELEASE}/build/debs/kubelet.service" | sed "s:/usr/bin:/opt/bin:g" > /etc/systemd/system/kubelet.service
mkdir -p /etc/systemd/system/kubelet.service.d
curl -sSL "https://raw.githubusercontent.com/kubernetes/kubernetes/${RELEASE}/build/debs/10-kubeadm.conf" | sed "s:/usr/bin:/opt/bin:g" > /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

Enable and start `kubelet`:

```bash
systemctl enable --now kubelet
```

{{% /tab %}} {{< /tabs >}}

kubeadm が何をすべきか指示するまで、kubelet はクラッシュループで数秒ごとに再起動
します。

## マスターノードの kubelet によって使用される cgroup ドライバーの設定

Docker を使用した場合、kubeadm は自動的に kubelet 向けの cgroup ドライバーを検出
し、それを実行時に`/var/lib/kubelet/kubeadm-flags.env`ファイルに設定します。

もしあなたが異なる CRI を使用している場合
、`/etc/default/kubelet`(CentOS、RHEL、Fedora では`/etc/sysconfig/kubelet`)ファ
イル内の`cgroup-driver`の値を以下のように変更する必要があります。

```bash
KUBELET_EXTRA_ARGS=--cgroup-driver=<value>
```

このファイルは、kubelet の追加のユーザー定義引数を取得するために
、`kubeadm init`および`kubeadm join`によって使用されます。

CRI の cgroup ドライバーが`cgroupfs`でない場合に**のみ**それを行う必要があること
に注意してください。なぜなら、これは既に kubelet のデフォルト値であるためです。

kubelet をリスタートする方法:

```bash
systemctl daemon-reload
systemctl restart kubelet
```

CRI-O や containerd といった他のコンテナランタイムの cgroup driver は実行中に自
動的に検出されます。

## トラブルシュート

kubeadm で問題が発生した場合は
、[トラブルシューティング](/ja/docs/setup/production-environment/tools/kubeadm/troubleshooting-kubeadm/)を
参照してください。

{{% capture whatsnext %}}

- [kubeadm を使用したシングルコントロールプレーンクラスターの作成](/ja/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

{{% /capture %}}
