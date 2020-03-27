---
title: Hello Minikube
content_template: templates/tutorial
weight: 5
menu:
  main:
    title: "Get Started"
    weight: 10
    post: >
      <p>手を動かす準備はできていますか？本チュートリアルでは、Node.jsを使った簡単な"Hello
      World"を実行するKubernetesクラスタをビルドします。</p>
card:
  name: tutorials
  weight: 10
---

{{% capture overview %}}

このチュートリアルでは、[Minikube](/docs/getting-started-guides/minikube)と
Katacoda を使用して、Kubernetes 上でシンプルな Hello World の Node.js アプリケー
ションを動かす方法を紹介します。Katacoda はブラウザで無償の Kubernetes 環境を提
供します。

{{< note >}}
[Minikube をローカルにインストール](/ja/docs/tasks/tools/install-minikube/)して
いる場合もこのチュートリアルを進めることが可能です。 {{< /note >}}

{{% /capture %}}

{{% capture objectives %}}

- Minikube への hello world アプリケーションのデプロイ
- アプリケーションの実行
- アプリケーションログの確認

{{% /capture %}}

{{% capture prerequisites %}}

このチュートリアルは下記のファイルからビルドされるコンテナーイメージを提供します
:

{{< codenew language="js" file="minikube/server.js" >}}

{{< codenew language="conf" file="minikube/Dockerfile" >}}

`docker build`コマンドについての詳細な情報は
、[Docker のドキュメント](https://docs.docker.com/engine/reference/commandline/build/)を
参照してください。

{{% /capture %}}

{{% capture lessoncontent %}}

## Minikube クラスタの作成

1. **Launch Terminal** をクリックしてください

   {{< kat-button >}}

   {{< note >}}Minikube をローカルにインストール済みの場合は、`minikube start`を
   実行してください。{{< /note >}}

2. ブラウザーで Kubernetes ダッシュボードを開いてください:

   ```shell
   minikube dashboard
   ```

3. Katacoda 環境のみ：ターミナルペーン上部の+ボタンをクリックしてから **Select
   port to view on Host 1** をクリックしてください。

4. Katacoda 環境のみ：`30000`を入力し、**Display Port**をクリックしてください。

## Deployment の作成

Kubernetes の[_Pod_](/ja/docs/concepts/workloads/pods/pod/) は、コンテナの管理や
ネットワーキングの目的でまとめられた、1 つ以上のコンテナのグループです。このチュ
ートリアルの Pod がもつコンテナは 1 つのみです。Kubernetes の
[_Deployment_](/ja/docs/concepts/workloads/controllers/deployment/) は Pod の状
態を確認し、Pod のコンテナが停止した場合には再起動します。Deployment は Pod の作
成やスケールを管理するために推奨される方法(手段)です。

1. `kubectl create` コマンドを使用して Pod を管理する Deployment を作成してく�
   さい。Pod は提供された Docker イメージを元にコンテナを実行します。

   ```shell
   kubectl create deployment hello-node --image=gcr.io/hello-minikube-zero-install/hello-node
   ```

2. Deployment を確認します:

   ```shell
   kubectl get deployments
   ```

   出力:

   ```shell
   NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
   hello-node   1         1         1            1           1m
   ```

3. Pod を確認します:

   ```shell
   kubectl get pods
   ```

   出力:

   ```shell
   NAME                          READY     STATUS    RESTARTS   AGE
   hello-node-5f76cf6ccf-br9b5   1/1       Running   0          1m
   ```

4. クラスタイベントを確認します:

   ```shell
   kubectl get events
   ```

5. `kubectl` で設定を確認します:

   ```shell
   kubectl config view
   ```

   {{< note >}} `kubectl`コマンドの詳細な情報
   は[kubectl overview](/docs/user-guide/kubectl-overview/)を参照してください
   。{{< /note >}}

## Service の作成

通常、Pod は Kubernetes クラスタ内部の IP アドレスからのみアクセスすることができ
ます。`hello-node`コンテナを Kubernetes の仮想ネットワークの外部からアクセスする
ためには、Kubernetes
の[_Service_](/ja/docs/concepts/services-networking/service/)としてポッドを公開
する必要があります。

1. `kubectl expose` コマンドを使用して Pod をインターネットに公開します:

   ```shell
   kubectl expose deployment hello-node --type=LoadBalancer --port=8080
   ```

   `--type=LoadBalancer`フラグは Service をクラスタ外部に公開したいことを示して
   います。

2. 作成した Service を確認します:

   ```shell
   kubectl get services
   ```

   出力:

   ```shell
   NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
   hello-node   LoadBalancer   10.108.144.78   <pending>     8080:30369/TCP   21s
   kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          23m
   ```

   ロードバランサーをサポートするクラウドプロバイダーでは、Service にアクセスす
   るための外部 IP アドレスが提供されます。 Minikube では、`LoadBalancer`タイプ
   は`minikube service`コマンドを使用した接続可能な Service を作成します。

3. 次のコマンドを実行します:

   ```shell
   minikube service hello-node
   ```

4. Katacoda 環境のみ：ターミナル画面上部の+ボタンをクリックして **Select port to
   view on Host 1** をクリックしてください。

5. Katacoda 環境のみ：`30369`(Service 出力に表示されている`8080`の反対側のポート
   を参照)を入力し、クリックしてください。

   "Hello World"メッセージが表示されるアプリケーションのブラウザウィンドウが開き
   ます。

## アドオンの有効化

Minikube はビルトインのアドオンがあり、有効化、無効化、あるいはローカルの
Kubernetes 環境に公開することができます。

1. サポートされているアドオンをリストアップします:

   ```shell
   minikube addons list
   ```

   出力:

   ```shell
   addon-manager: enabled
   coredns: disabled
   dashboard: enabled
   default-storageclass: enabled
   efk: disabled
   freshpod: disabled
   heapster: disabled
   ingress: disabled
   kube-dns: enabled
   metrics-server: disabled
   nvidia-driver-installer: disabled
   nvidia-gpu-device-plugin: disabled
   registry: disabled
   registry-creds: disabled
   storage-provisioner: enabled
   ```

2. ここでは例として`heapster`のアドオンを有効化します:

   ```shell
   minikube addons enable heapster
   ```

   出力:

   ```shell
   heapster was successfully enabled
   ```

3. 作成されたポッドとサービスを確認します:

   ```shell
   kubectl get pod,svc -n kube-system
   ```

   出力:

   ```shell
   NAME                                        READY     STATUS    RESTARTS   AGE
   pod/heapster-9jttx                          1/1       Running   0          26s
   pod/influxdb-grafana-b29w8                  2/2       Running   0          26s
   pod/kube-addon-manager-minikube             1/1       Running   0          34m
   pod/kube-dns-6dcb57bcc8-gv7mw               3/3       Running   0          34m
   pod/kubernetes-dashboard-5498ccf677-cgspw   1/1       Running   0          34m
   pod/storage-provisioner                     1/1       Running   0          34m

   NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
   service/heapster               ClusterIP   10.96.241.45    <none>        80/TCP              26s
   service/kube-dns               ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP       34m
   service/kubernetes-dashboard   NodePort    10.109.29.1     <none>        80:30000/TCP        34m
   service/monitoring-grafana     NodePort    10.99.24.54     <none>        80:30002/TCP        26s
   service/monitoring-influxdb    ClusterIP   10.111.169.94   <none>        8083/TCP,8086/TCP   26s
   ```

4. `heapster`を無効化します:

   ```shell
   minikube addons disable heapster
   ```

   出力:

   ```shell
   heapster was successfully disabled
   ```

## クリーンアップ

クラスタに作成したリソースをクリーンアップします:

```shell
kubectl delete service hello-node
kubectl delete deployment hello-node
```

(オプション)Minikube の仮想マシン(VM)を停止します:

```shell
minikube stop
```

(オプション)Minikube の VM を削除します:

```shell
minikube delete
```

{{% /capture %}}

{{% capture whatsnext %}}

- [Deployment オブジェクト](/ja/docs/concepts/workloads/controllers/deployment/)に
  ついて学ぶ.
- [アプリケーションのデプロイ](/ja/docs/tasks/run-application/run-stateless-application-deployment/)に
  ついて学ぶ.
- [Service オブジェクト](/ja/docs/concepts/services-networking/service/)について
  学ぶ.

{{% /capture %}}
