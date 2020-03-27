---
title: Minikubeを使用してローカル環境でKubernetesを動かす
content_template: templates/concept
---

{{% capture overview %}}

Minikube はローカル環境で Kubernetes を簡単に実行するためのツールです
。Kubernetes を試したり日々の開発への使用を検討するユーザー向けに、PC 上の VM 内
でシングルノードの Kubernetes クラスタを実行することができます。

{{% /capture %}}

{{% capture body %}}

## Minikube の機能

- Minikube のサポートする Kubernetes の機能:
  - DNS
  - NodePorts
  - ConfigMaps と Secrets
  - ダッシュボード
  - コンテナランタイム: Docker, [rkt](https://github.com/rkt/rkt),
    [CRI-O](https://cri-o.io/),
    [containerd](https://github.com/containerd/containerd)
  - CNI (Container Network Interface) の有効化
  - Ingress

## インストール

[Minikube のインストール](/ja/docs/tasks/tools/install-minikube/) を参照

## クイックスタート

これは Minikube の使い方の簡単なデモです。もし VM ドライバを変更したい場合は、適
切な `--vm-driver=xxx` フラグを `minikube start` に設定してください。Minikube は
以下のドライバをサポートしています。

- virtualbox
- vmwarefusion
- kvm2
  ([driver installation](https://git.k8s.io/minikube/docs/drivers.md#kvm2-driver))
- kvm
  ([driver installation](https://git.k8s.io/minikube/docs/drivers.md#kvm-driver))
- hyperkit
  ([driver installation](https://git.k8s.io/minikube/docs/drivers.md#hyperkit-driver))
- xhyve
  ([driver installation](https://git.k8s.io/minikube/docs/drivers.md#xhyve-driver))
  (非推奨)
- hyperv
  ([driver installation](https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#hyperv-driver))
  注意: 以下の IP は動的であり、変更される可能性があります。IP は `minikube ip`
  で取得することができます。
- none (VM ではなくホスト上で Kubernetes コンポーネントを起動する。このドライバ
  を使用するには Docker
  ([docker install](https://docs.docker.com/install/linux/docker-ce/ubuntu/)) と
  Linux 環境を必要とします)

```shell
minikube start
```

```
Starting local Kubernetes cluster...
Running pre-create checks...
Creating machine...
Starting local Kubernetes cluster...
```

```shell
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.10
```

```
deployment.apps/hello-minikube created
```

```shell
kubectl expose deployment hello-minikube --type=NodePort --port=8080
```

```
service/hello-minikube exposed
```

```
# We have now launched an echoserver pod but we have to wait until the pod is up before curling/accessing it
# via the exposed service.
# To check whether the pod is up and running we can use the following:
kubectl get pod
```

```
NAME                              READY     STATUS              RESTARTS   AGE
hello-minikube-3383150820-vctvh   0/1       ContainerCreating   0          3s
```

```shell
# We can see that the pod is still being created from the ContainerCreating status
kubectl get pod
```

```
NAME                              READY     STATUS    RESTARTS   AGE
hello-minikube-3383150820-vctvh   1/1       Running   0          13s
```

```shell
# We can see that the pod is now Running and we will now be able to curl it:
curl $(minikube service hello-minikube --url)
```

```

Hostname: hello-minikube-7c77b68cff-8wdzq

Pod Information:
	-no pod information available-

Server values:
	server_version=nginx: 1.13.3 - lua: 10008

Request Information:
	client_address=172.17.0.1
	method=GET
	real path=/
	query=
	request_version=1.1
	request_scheme=http
	request_uri=http://192.168.99.100:8080/

Request Headers:
	accept=*/*
	host=192.168.99.100:30674
	user-agent=curl/7.47.0

Request Body:
	-no body in request-
```

```shell
kubectl delete services hello-minikube
```

```
service "hello-minikube" deleted
```

```shell
kubectl delete deployment hello-minikube
```

```
deployment.extensions "hello-minikube" deleted
```

```shell
minikube stop
```

```
Stopping local Kubernetes cluster...
Stopping "minikube"...
```

### コンテナランタイムの代替

#### containerd

[containerd](https://github.com/containerd/containerd) をコンテナランタイムとし
て使用するには以下を実行してください:

```bash
minikube start \
    --network-plugin=cni \
    --enable-default-cni \
    --container-runtime=containerd \
    --bootstrapper=kubeadm
```

もしくは拡張バージョンを使用することもできます:

```bash
minikube start \
    --network-plugin=cni \
    --enable-default-cni \
    --extra-config=kubelet.container-runtime=remote \
    --extra-config=kubelet.container-runtime-endpoint=unix:///run/containerd/containerd.sock \
    --extra-config=kubelet.image-service-endpoint=unix:///run/containerd/containerd.sock \
    --bootstrapper=kubeadm
```

#### CRI-O

[CRI-O](https://github.com/kubernetes-incubator/cri-o) をコンテナランタイムとし
て使用するには以下を実行してください:

```bash
minikube start \
    --network-plugin=cni \
    --enable-default-cni \
    --container-runtime=cri-o \
    --bootstrapper=kubeadm
```

もしくは拡張バージョンを使用することもできます:

```bash
minikube start \
    --network-plugin=cni \
    --enable-default-cni \
    --extra-config=kubelet.container-runtime=remote \
    --extra-config=kubelet.container-runtime-endpoint=/var/run/crio.sock \
    --extra-config=kubelet.image-service-endpoint=/var/run/crio.sock \
    --bootstrapper=kubeadm
```

#### rkt コンテナエンジン

[rkt](https://github.com/rkt/rkt) をコンテナランタイムとして使用するには以下を実
行してください:

```shell
minikube start \
    --network-plugin=cni \
    --enable-default-cni \
    --container-runtime=rkt
```

これは rkt と Docker の両方を含んだ代替の Minikube の ISO イメージを使用し、CNI
ネットワークを有効にします。

### ドライバープラグイン

サポートされているドライバとプラグインのインストールの詳細については
[DRIVERS](https://git.k8s.io/minikube/docs/drivers.md) を参照してください。

### Docker デーモンの再利用によるローカルイメージの使用

Kubernetes の単一の VM を使用する場合、Minikube 組み込みの Docker デーモンの再利
用がおすすめです。ホストマシン上に Docker レジストリを構築してイメージをプッシュ
する必要がなく、ローカルでの実験を加速させる Minikube と同じ Docker デーモンの中
に構築することができます。ただ Docker イメージに'latest'以外のタグを付け、そのタ
グを使用してイメージをプルしてください。イメージのバージョンを指定しなければ
、`Always` のプルイメージポリシーにより `:latest` と仮定され、もしデフォルトの
Docker レジストリ(通常は DockerHub)にどのバージョンの Docker イメージもまだ存在
しない場合には、`ErrImagePull` になる恐れがあります。

Mac/Linux のホストで Docker デーモンを操作できるようにするには、shell 内で
`docker-env command` を使います:

```shell
eval $(minikube docker-env)
```

これにより、Minikube の VM 内の Docker デーモンと通信しているホストの Mac/Linux
マシンのコマンドラインで Docker を使用できるようになっているはずです。

```shell
docker ps
```

CentOS 7 では、Docker が以下のエラーを出力することがあります:

```shell
Could not read CA certificate "/etc/docker/ca.pem": open /etc/docker/ca.pem: no such file or directory
```

修正方法としては、/etc/sysconfig/docker を更新して Minikube 環境の変更が確実に反
映されるようにすることです:

```shell
< DOCKER_CERT_PATH=/etc/docker
---
> if [ -z "${DOCKER_CERT_PATH}" ]; then
>   DOCKER_CERT_PATH=/etc/docker
> fi
```

imagePullPolicy:Always をオフにすることを忘れないでください: さもなければ
Kubernetes はローカルに構築したイメージを使用しません。

## クラスターの管理

### クラスターの起動

`minikube start` コマンドはクラスターを起動することができます。このコマンドはシ
ングルノードの Kubernetes クラスターを実行する仮想マシンを作成・設定します。また
、このクラスターと通信する [kubectl](/docs/user-guide/kubectl-overview/) のイン
ストールも設定します。

もし Web プロキシーを通している場合、そのプロキシー情報を `minikube start` コマ
ンドに渡す必要があります:

```shell
https_proxy=<my proxy> minikube start --docker-env http_proxy=<my proxy> --docker-env https_proxy=<my proxy> --docker-env no_proxy=192.168.99.0/24
```

残念なことに、ただ環境変数を設定するだけではうまく動作しません。

Minikube は "minikube" コンテキストも作成し、そのコンテキストをデフォルト設定と
して kubectl に設定します。あとでコンテキストを切り戻すには、このコマンドを実行
してください: `kubectl config use-context minikube`

#### Kubernetes バージョンの指定

`minikube start` コマンドに `--kubernetes-version` 文字列を追加することで、
Minikube に Kubernetes の特定のバージョンを指定することができます。例えば
、`v1.7.3` のバージョンを実行するには以下を実行します:

```
minikube start --kubernetes-version v1.7.3
```

### Kubernetes の設定

Minikube にはユーザーが任意の値で Kubenetes コンポーネントを設定することを可能に
する "configurator" 機能があります。この機能を使うには、`minikube start` コマン
ドに `--extra-config` フラグを使うことができます。

このフラグは繰り返されるので、複数のオプションを設定するためにいくつかの異なる値
を使って何度も渡すことができます。

このフラグは `component.key=value` 形式の文字列を取ります。`component` は下記の
リストの文字列の 1 つです。 `key`は設定構造体上の値で、 `value` は設定する値です
。

各コンポーネントの Kubernetes `componentconfigs` のドキュメントを調べることで有
効なキーを見つけることができます。サポートされている各設定のドキュメントは次のと
おりです:

- [kubelet](https://godoc.org/k8s.io/kubernetes/pkg/kubelet/apis/config#KubeletConfiguration)
- [apiserver](https://godoc.org/k8s.io/kubernetes/cmd/kube-apiserver/app/options#ServerRunOptions)
- [proxy](https://godoc.org/k8s.io/kubernetes/pkg/proxy/apis/config#KubeProxyConfiguration)
- [controller-manager](https://godoc.org/k8s.io/kubernetes/pkg/controller/apis/config#KubeControllerManagerConfiguration)
- [etcd](https://godoc.org/github.com/coreos/etcd/etcdserver#ServerConfig)
- [scheduler](https://godoc.org/k8s.io/kubernetes/pkg/scheduler/apis/config#KubeSchedulerConfiguration)

#### 例

Kubelet の `MaxPods` 設定を 5 に変更するには、このフラグを渡します:
`--extra-config=kubelet.MaxPods=5`

この機能はネストした構造体もサポートします。スケジューラーの
`LeaderElection.LeaderElect` を `true` に設定するには、このフラグを渡します:
`--extra-config=scheduler.LeaderElection.LeaderElect=true`

`apiserver` の `AuthorizationMode` を `RABC` に設定するには、このフラグを使いま
す: `--extra-config=apiserver.authorization-mode=RBAC`.

### クラスターの停止

`minikube stop` コマンドを使ってクラスターを停止することができます。このコマンド
は Minikube 仮想マシンをシャットダウンしますが、すべてのクラスターの状態とデータ
を保存します。クラスターを再起動すると、以前の状態に復元されます。

### クラスターの削除

`minikube delete` コマンドを使ってクラスターを削除することができます。このコマン
ドは Minikube 仮想マシンをシャットダウンして削除します。データや状態は保存されま
せん。

## クラスターに触れてみよう

### Kubectl

`minikube start` コマンドは "minikube" とい
う[kubectl context](/docs/reference/generated/kubectl/kubectl-commands#-em-set-context-em-)を
作成します。このコンテキストは Minikube クラスターと通信するための設定が含まれて
います。

Minikube はこのコンテキストを自動的にデフォルトに設定しますが、将来的に設定を切
り戻す場合には次のコマンドを実行してください:

`kubectl config use-context minikube`,

もしくは各コマンドにコンテキストを次のように渡します:
`kubectl get pods --context=minikube`

### ダッシュボード

[Kubernetes Dashboard](/docs/tasks/access-application-cluster/web-ui-dashboard/)に
アクセスするには、Minikube を起動してアドレスを取得した後、シェルでこのコマンド
を実行してください:

```shell
minikube dashboard
```

### サービス

ノードポート経由で公開されているサービスにアクセスするには、Minikube を起動して
アドレスを取得した後、シェルでこのコマンドを実行してください:

```shell
minikube service [-n NAMESPACE] [--url] NAME
```

## ネットワーク

Minikube の VM は `minikube ip`コマンドで取得できるホストオンリー IP アドレスを
介してホストシステムに公開されます。 NodePort 上では、 `NodePort` タイプのどのサ
ービスもその IP アドレスを介してアクセスできます。

サービスの NodePort を決定するには、`kubectl` コマンドを次のように使用します:

`kubectl get service $SERVICE --output='jsonpath="{.spec.ports[0].nodePort}"'`

## 永続ボリューム

Minikube は `hostPath` タイプ
の[PersistentVolumes](/docs/concepts/storage/persistent-volumes/)をサポートしま
す。この PersistentVolumes は Minikube の VM 内のディレクトリーにマッピングされ
ます。

Minikube の VM は tmpfs で起動するため、ほとんどのディレクトリーは再起動しても持
続しません (`minikube stop`)。しかし、Minikube は以下のホストディレクトリーに保
存されているファイルを保持するように設定されています:

- `/data`
- `/var/lib/minikube`
- `/var/lib/docker`

以下は `/data` ディレクトリのデータを永続化する PersistentVolume の設定例です:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0001
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 5Gi
  hostPath:
    path: /data/pv0001/
```

## ホストフォルダーのマウント

一部のドライバーは VM 内にホストフォルダーをマウントするため、VM とホストの間で
ファイルを簡単に共有できます。これらは現時点では設定可能ではなく、使用しているド
ライバーと OS によって異なります。

{{< note >}} ホストフォルダーの共有は KVM ドライバーにはまだ実装されていません。
{{< /note >}}

| Driver        | OS      | HostFolder | VM        |
| ------------- | ------- | ---------- | --------- |
| VirtualBox    | Linux   | /home      | /hosthome |
| VirtualBox    | macOS   | /Users     | /Users    |
| VirtualBox    | Windows | C://Users  | /c/Users  |
| VMware Fusion | macOS   | /Users     | /Users    |
| Xhyve         | macOS   | /Users     | /Users    |

## プライベートコンテナレジストリ

プライベートコンテナレジストリにアクセスするには
、[このページ](/docs/concepts/containers/images/)の手順に従ってください。

`ImagePullSecrets` を使用することをおすすめしますが、Minikube の VM 内でアクセス
設定したい場合には、`/home/docker` ディレクトリに `.dockercfg` を置くか、または
`/home/docker/.docker` ディレクトリに `config.json` を置いてください。

## アドオン

カスタムアドオンを正しく起動または再起動させるには、 Minikube で起動したいアドオ
ンを `~/.minikube/addons` ディレクトリに置きます。このフォルダ内のアドオンは
Minikube の VM に移動され、Minikube が起動または再起動されるたびにアドオンが起動
されます。

## HTTP プロキシ経由の Minikube 利用

Minikube は Kubernetes と Docker デーモンを含む仮想マシンを作成します。
Kubernetes が Docker を使用してコンテナをスケジュールしようとする際、Docker デー
モンはコンテナをプルするために外部ネットワークを必要とする場合があります。

HTTP プロキシーを通している場合には、プロキシー設定を Docker に提供する必要があ
ります。これを行うには、`minikube start` に必要な環境変数をフラグとして渡します
。

例:

```shell
minikube start --docker-env http_proxy=http://$YOURPROXY:PORT \
               --docker-env https_proxy=https://$YOURPROXY:PORT
```

仮想マシンのアドレスが 192.168.99.100 の場合、プロキシーの設定により `kubectl`
が直接アクセスできない可能性があります。この IP アドレスのプロキシー設定を迂回す
るには、以下のように no_proxy 設定を変更する必要があります。

```shell
export no_proxy=$no_proxy,$(minikube ip)
```

## 既知の問題

- クラウドプロバイダーを必要とする機能は Minikube では動作しません
  - ロードバランサー
- 複数ノードを必要とする機能
  - 高度なスケジューリングポリシー

## 設計

Minikube は VM のプロビジョニング
に[libmachine](https://github.com/docker/machine/tree/master/libmachine)を使用し
、[kubeadm](https://github.com/kubernetes/kubeadm)を Kubernetes クラスターのプロ
ビジョニングに使用します。

Minikube の詳細については
、[proposal](https://git.k8s.io/community/contributors/design-proposals/cluster-lifecycle/local-cluster-ux.md)を
参照してください。

## 追加リンク集

- **目標と非目標**: Minikube プロジェクトの目標と非目標については
  、[ロードマップ](https://git.k8s.io/minikube/docs/contributors/roadmap.md)を参
  照してください。
- **開発ガイド**: プルリクエストを送る方法の概要については
  、[CONTRIBUTING.md](https://git.k8s.io/minikube/CONTRIBUTING.md)を参照してくだ
  さい。
- **Minikube のビルド**: Minikube をソースからビルド/テストする方法については
  、[ビルドガイド](https://git.k8s.io/minikube/docs/contributors/build_guide.md)を
  参照してください。
- **新しい依存性の追加**: Minikube に新しい依存性を追加する方法については
  、[依存性追加ガイド](https://git.k8s.io/minikube/docs/contributors/adding_a_dependency.md)を
  参照してください。
- **新しいアドオンの追加**: Minikube に新しいアドオンを追加する方法については
  、[アドオン追加ガイド](https://git.k8s.io/minikube/docs/contributors/adding_an_addon.md)を
  参照してください。
- **MicroK8s**: 仮想マシンを実行したくない Linux ユーザーは代わり
  に[MicroK8s](https://microk8s.io/)を検討してみてください。

## コミュニティ

コントリビューションや質問、コメントは歓迎・奨励されています! Minikube の開発者
は[Slack](https://kubernetes.slack.com)の#minikube チャンネルにいます(Slack への
招待状
は[こちら](http://slack.kubernetes.io/))。[kubernetes-dev Google Groups メーリングリスト](https://groups.google.com/forum/#!forum/kubernetes-dev)も
あります。メーリングリストに投稿する際は件名の最初に "minikube: " をつけてくださ
い。

{{% /capture %}}
