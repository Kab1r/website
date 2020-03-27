---
title: クラスター内のアプリケーションにアクセスするために外部IPアドレスを公開する
content_template: templates/tutorial
weight: 10
---

{{% capture overview %}}

このページでは、外部 IP アドレスを公開する Kubernetes の Service オブジェクトを
作成する方法を示します。

{{% /capture %}}

{{% capture prerequisites %}}

- [kubectl](/ja/docs/tasks/tools/install-kubectl/)をインストールしてください。

- Kubernetes クラスターを作成する際に、Google Kubernetes Engine や Amazon Web
  Services のようなクラウドプロバイダーを使用します。このチュートリアルでは、ク
  ラウドプロバイダーを必要とす
  る[外部ロードバランサー](/docs/tasks/access-application-cluster/create-external-load-balancer/)を
  作成します。

- Kubernetes API サーバーと通信するために、`kubectl`を設定してください。手順につ
  いては、各クラウドプロバイダーのドキュメントを参照してください。

{{% /capture %}}

{{% capture objectives %}}

- 5 つのインスタンスで実際のアプリケーションを起動します。
- 外部 IP アドレスを公開する Service オブジェクトを作成します。
- 起動中のアプリケーションにアクセスするために Service オブジェクトを使用します
  。

{{% /capture %}}

{{% capture lessoncontent %}}

## 5 つの Pod で起動しているアプリケーションへの Service の作成

1. クラスターにて Hello World アプリケーションを実行してください。

{{< codenew file="service/load-balancer-example.yaml" >}}

```shell
kubectl apply -f https://k8s.io/examples/service/load-balancer-example.yaml
```

上記のコマンドにより
、[Deployment](/ja/docs/concepts/workloads/controllers/deployment/)オブジェクト
を作成し、[ReplicaSet](/ja/docs/concepts/workloads/controllers/replicaset/)オブ
ジェクトを関連づけます。ReplicaSet には 5 つ
の[Pod](/ja/docs/concepts/workloads/pods/pod/)があり、それぞれ Hello World アプ
リケーションが起動しています。

1.  Deployment に関する情報を表示します:

        kubectl get deployments hello-world
        kubectl describe deployments hello-world

1.  ReplicaSet オブジェクトに関する情報を表示します:

        kubectl get replicasets
        kubectl describe replicasets

1.  Deployment を公開する Service オブジェクトを作成します。

        kubectl expose deployment hello-world --type=LoadBalancer --name=my-service

1.  Service に関する情報を表示します:

        kubectl get services my-service

    出力は次のようになります:

        NAME         TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)    AGE
        my-service   LoadBalancer   10.3.245.137   104.198.205.71   8080/TCP   54s

    注意: 外部 IP アドレスが\<pending\>と表示されている場合は、しばらく待ってか
    ら同じコマンドを実行してください。

1.  Service に関する詳細な情報を表示します:

        kubectl describe services my-service

    出力は次のようになります:

        Name:           my-service
        Namespace:      default
        Labels:         app.kubernetes.io/name=load-balancer-example
        Annotations:    <none>
        Selector:       app.kubernetes.io/name=load-balancer-example
        Type:           LoadBalancer
        IP:             10.3.245.137
        LoadBalancer Ingress:   104.198.205.71
        Port:           <unset> 8080/TCP
        NodePort:       <unset> 32377/TCP
        Endpoints:      10.0.0.6:8080,10.0.1.6:8080,10.0.1.7:8080 + 2 more...
        Session Affinity:   None
        Events:         <none>

    Service によって公開された外部 IP アドレス(`LoadBalancer Ingress`)を記録して
    おいてください。この例では、外部 IP アドレスは 104.198.205.71 です。また
    、`Port`および`NodePort`の値も控えてください。この例では、`Port`は
    8080、`NodePort`は 32377 です。

1.  先ほどの出力にて、Service にはいくつかのエンドポイントがあることを確認できま
    す: 10.0.0.6:8080、 10.0.1.6:8080、10.0.1.7:8080、その他 2 つです。これらは
    Hello World アプリケーションが動作している Pod の内部 IP アドレスです。これ
    らの Pod のアドレスを確認するには、次のコマンドを実行します:

         kubectl get pods --output=wide

    出力は次のようになります:

         NAME                         ...  IP         NODE
         hello-world-2895499144-1jaz9 ...  10.0.1.6   gke-cluster-1-default-pool-e0b8d269-1afc
         hello-world-2895499144-2e5uh ...  10.0.1.8   gke-cluster-1-default-pool-e0b8d269-1afc
         hello-world-2895499144-9m4h1 ...  10.0.0.6   gke-cluster-1-default-pool-e0b8d269-5v7a
         hello-world-2895499144-o4z13 ...  10.0.1.7   gke-cluster-1-default-pool-e0b8d269-1afc
         hello-world-2895499144-segjf ...  10.0.2.5   gke-cluster-1-default-pool-e0b8d269-cpuc

1.  Hello World アプリケーションにアクセスするために、外部 IP アドレス
    (`LoadBalancer Ingress`)を使用します:

        curl http://<external-ip>:<port>

    ここで、`<external-ip>`は Service の外部 IP アドレス(`LoadBalancer Ingress`)
    で、 `<port>`は Service の詳細出力における`Port`です。minikube を使用してい
    る場合、`minikube service my-service`を実行することで Hello World アプリケー
    ションをブラウザで自動的に開かれます。

    正常なリクエストに対するレスポンスは、hello メッセージです:

        Hello Kubernetes!

{{% /capture %}}

{{% capture cleanup %}}

Service を削除する場合、次のコマンドを実行します:

        kubectl delete services my-service

Deployment、ReplicaSet、および Hello World アプリケーションが動作している Pod を
削除する場合、次のコマンドを実行します:

        kubectl delete deployment hello-world

{{% /capture %}}

{{% capture whatsnext %}}

[connecting applications with services](/docs/concepts/services-networking/connect-applications-service/)に
て詳細を学ぶことができます。 {{% /capture %}}
