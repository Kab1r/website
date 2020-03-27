---
title: Serviceを利用したクラスター内のアプリケーションへのアクセス
content_template: templates/tutorial
weight: 60
---

{{% capture overview %}}

ここでは、クラスター内で稼働しているアプリケーションに外部からアクセスするために
、Kubernetes の Service オブジェクトを作成する方法を紹介します。例として、2 つの
インスタンスから成るアプリケーションへのロードバランシングを扱います。

{{% /capture %}}

{{% capture prerequisites %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

{{% /capture %}}

{{% capture objectives %}}

- 2 つの Hellow World アプリケーションを稼働させる。
- Node のポートを公開する Service オブジェクトを作成する。
- 稼働しているアプリケーションにアクセスするために Service オブジェクトを使用す
  る。

{{% /capture %}}

{{% capture lessoncontent %}}

## 2 つの Pod から成るアプリケーションの Service を作成

1. クラスタで Hello World アプリケーションを稼働させます:

   ```shell
   kubectl run hello-world --replicas=2 --labels="run=load-balancer-example" --image=gcr.io/google-samples/node-hello:1.0  --port=8080
   ```

   このコマンドは
   [Deployment](/ja/docs/concepts/workloads/controllers/deployment/) オブジェク
   トとそれに紐付く
   [ReplicaSet](/ja/docs/concepts/workloads/controllers/replicaset/) オブジェク
   トを作成します。ReplicaSet は、Hello World アプリケーションが稼働している 2
   つの [Pod](/ja/docs/concepts/workloads/pods/pod/) から構成されます。

1. Deployment の情報を表示します:

   ```shell
   kubectl get deployments hello-world
   kubectl describe deployments hello-world
   ```

1. ReplicaSet オブジェクトの情報を表示します:

   ```shell
   kubectl get replicasets
   kubectl describe replicasets
   ```

1. Deployment を公開する Service オブジェクトを作成します:

   ```shell
   kubectl expose deployment hello-world --type=NodePort --name=example-service
   ```

1. Service に関する情報を表示します:

   ```shell
   kubectl describe services example-service
   ```

   出力例は以下の通りです:

   ```shell
   Name:                   example-service
   Namespace:              default
   Labels:                 run=load-balancer-example
   Annotations:            <none>
   Selector:               run=load-balancer-example
   Type:                   NodePort
   IP:                     10.32.0.16
   Port:                   <unset> 8080/TCP
   TargetPort:             8080/TCP
   NodePort:               <unset> 31496/TCP
   Endpoints:              10.200.1.4:8080,10.200.2.5:8080
   Session Affinity:       None
   Events:                 <none>
   ```

   NodePort の値を記録しておきます。上記の例では、31496 です。

1. Hello World アプリーションが稼働している Pod を表示します:
   ```shell
   kubectl get pods --selector="run=load-balancer-example" --output=wide
   ```
   出力例は以下の通りです:
   ```shell
   NAME                           READY   STATUS    ...  IP           NODE
   hello-world-2895499144-bsbk5   1/1     Running   ...  10.200.1.4   worker1
   hello-world-2895499144-m1pwt   1/1     Running   ...  10.200.2.5   worker2
   ```
1. Hello World pod が稼働する Node のうち、いずれか 1 つのパブリック IP アドレス
   を確認します。確認方法は、使用している環境により異なります。例として
   、Minikube の場合は`kubectl cluster-info`、Google Compute Engine の場合
   は`gcloud compute instances list`によって確認できます。

1. 選択したノード上で、NodePort の値での TCP 通信を許可するファイヤーウォールを
   作成します。 NodePort の値が 31568 の場合、31568 番のポートを利用した TCP 通
   信を許可するファイヤーウォールを作成します。クラウドプロバイダーによって設定
   方法が異なります。

1. Hello World application にアクセスするために、Node のアドレスとポート番号を使
   用します:
   ```shell
   curl http://<public-node-ip>:<node-port>
   ```
   ここで `<public-node-ip>` は Node のパブリック IP アドレス、 `<node-port>` は
   NodePort Service のポート番号の値を表しています。リクエストが成功すると、下記
   のメッセージが表示されます:
   ```shell
   Hello Kubernetes!
   ```

## service configuration file の利用

`kubectl expose`コマンドの代わりに、
[service configuration file](/ja/docs/concepts/services-networking/service/) を
使用して Service を作成することもできます。

{{% /capture %}}

{{% capture cleanup %}}

Service を削除するには、以下のコマンドを実行します:

    kubectl delete services example-service

Hello World アプリケーションが稼働している Deployment、ReplicaSet、Pod を削除す
るには、以下のコマンドを実行します:

    kubectl delete deployment hello-world

{{% /capture %}}

{{% capture whatsnext %}}

詳細は
[service を利用してアプリケーションと接続する](/docs/concepts/services-networking/connect-applications-service/)
を確認してください。 {{% /capture %}}
