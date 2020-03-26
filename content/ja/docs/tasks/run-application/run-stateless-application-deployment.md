---
title: Deploymentを使用してステートレスアプリケーションを実行する
min-kubernetes-server-version: v1.9
content_template: templates/tutorial
weight: 10
---

{{% capture overview %}}

このページでは、Kubernetes Deployment オブジェクトを使用してアプリケーションを実
行する方法を説明します。

{{% /capture %}}

{{% capture objectives %}}

- nginx deployment を作成します。
- kubectl を使って deployment に関する情報を一覧表示します。
- deployment を更新します。

{{% /capture %}}

{{% capture prerequisites %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

{{% /capture %}}

{{% capture lessoncontent %}}

## nginx deployment の作成と探検

Kubernetes Deployment オブジェクトを作成することでアプリケーションを実行できます
。また、YAML ファイルで Deployment を記述できます。例えば、この YAML ファイルは
nginx:1.14.2 Docker イメージを実行するデプロイメントを記述しています:

{{< codenew file="application/deployment.yaml" >}}

1.  YAML ファイルに基づいて Deployment を作成します:

        kubectl apply -f https://k8s.io/examples/application/deployment.yaml

1.  Deployment に関する情報を表示します:

        kubectl describe deployment nginx-deployment

    出力はこのようになります:

        user@computer:~/website$ kubectl describe deployment nginx-deployment
        Name:     nginx-deployment
        Namespace:    default
        CreationTimestamp:  Tue, 30 Aug 2016 18:11:37 -0700
        Labels:     app=nginx
        Annotations:    deployment.kubernetes.io/revision=1
        Selector:   app=nginx
        Replicas:   2 desired | 2 updated | 2 total | 2 available | 0 unavailable
        StrategyType:   RollingUpdate
        MinReadySeconds:  0
        RollingUpdateStrategy:  1 max unavailable, 1 max surge
        Pod Template:
          Labels:       app=nginx
          Containers:
           nginx:
            Image:              nginx:1.14.2
            Port:               80/TCP
            Environment:        <none>
            Mounts:             <none>
          Volumes:              <none>
        Conditions:
          Type          Status  Reason
          ----          ------  ------
          Available     True    MinimumReplicasAvailable
          Progressing   True    NewReplicaSetAvailable
        OldReplicaSets:   <none>
        NewReplicaSet:    nginx-deployment-1771418926 (2/2 replicas created)
        No events.

1.  Deployment によって作成された Pod を一覧表示します:

        kubectl get pods -l app=nginx

    出力はこのようになります:

        NAME                                READY     STATUS    RESTARTS   AGE
        nginx-deployment-1771418926-7o5ns   1/1       Running   0          16h
        nginx-deployment-1771418926-r18az   1/1       Running   0          16h

1.  Pod に関する情報を表示します:

        kubectl describe pod <pod-name>

    ここで`<pod-name>`は Pod の 1 つの名前を指定します。

## Deployment の更新

新しい YAML ファイルを適用して Deployment を更新できます。この YAML ファイルは
、Deployment を更新して nginx 1.8 を使用するように指定しています。

{{< codenew file="application/deployment-update.yaml" >}}

1.  新しい YAML ファイルを適用します:

         kubectl apply -f https://k8s.io/examples/application/deployment-update.yaml

1.  Deployment が新しい名前で Pod を作成し、古い Pod を削除するのを監視します:

         kubectl get pods -l app=nginx

## レプリカ数を増やすことによるアプリケーションのスケール

新しい YAML ファイルを適用することで、Deployment 内の Pod の数を増やすことができ
ます。この YAML ファイルは`replicas`を 4 に設定します。これは Deployment が 4 つ
の Pod を持つべきであることを指定します:

{{< codenew file="application/deployment-scale.yaml" >}}

1.  新しい YAML ファイルを適用します:

        kubectl apply -f https://k8s.io/examples/application/deployment-scale.yaml

1.  Deployment に 4 つの Pod があることを確認します:

        kubectl get pods -l app=nginx

    出力はこのようになります:

        NAME                               READY     STATUS    RESTARTS   AGE
        nginx-deployment-148880595-4zdqq   1/1       Running   0          25s
        nginx-deployment-148880595-6zgi1   1/1       Running   0          25s
        nginx-deployment-148880595-fxcez   1/1       Running   0          2m
        nginx-deployment-148880595-rwovn   1/1       Running   0          2m

## Deployment の削除

Deployment を名前を指定して削除します:

    kubectl delete deployment nginx-deployment

## ReplicationControllers -- 昔のやり方

複製アプリケーションを作成するための好ましい方法は Deployment を使用することです
。そして、Deployment は ReplicaSet を使用します。 Deployment と ReplicaSet が
Kubernetes に追加される前は
、[ReplicationController](/docs/concepts/workloads/controllers/replicationcontroller/)を
使用して複製アプリケーションを構成していました。

{{% /capture %}}

{{% capture whatsnext %}}

- [Deployment オブジェクト](/docs/concepts/workloads/controllers/deployment/)の
  詳細

{{% /capture %}}
