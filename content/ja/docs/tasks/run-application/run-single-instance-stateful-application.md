---
title: 単一レプリカのステートフルアプリケーションを実行する
content_template: templates/tutorial
weight: 20
---

{{% capture overview %}}

このページでは、PersistentVolume と Deployment を使用して、Kubernetes で単一レプ
リカのステートフルアプリケーションを実行する方法を説明します。アプリケーションは
MySQL です。

{{% /capture %}}

{{% capture objectives %}}

- 自身の環境のディスクを参照する PersistentVolume を作成します。
- MySQL の Deployment を作成します。
- MySQL を DNS 名でクラスター内の他の Pod に公開します。

{{% /capture %}}

{{% capture prerequisites %}}

- {{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

- {{< include "default-storage-class-prereqs.md" >}}

{{% /capture %}}

{{% capture lessoncontent %}}

## MySQL をデプロイする

Kubernetes Deployment を作成し、PersistentVolumeClaim を使用して既存の
PersistentVolume に接続することで、ステートフルアプリケーションを実行できます。
たとえば、以下の YAML ファイルは MySQL を実行し、PersistentVolumeClaim を参照す
る Deployment を記述しています。このファイルは/var/lib/mysql のボリュームマウン
トを定義してから、20G のボリュームを要求する PersistentVolumeClaim を作成します
。この要求は、要件を満たす既存のボリューム、または動的プロビジョナーによって満た
されます。

注：パスワードは YAML ファイル内に定義されており、これは安全ではありません。安全
な解決策については[Kubernetes Secret](/docs/concepts/configuration/secret/)を参
照してください 。

{{< codenew file="application/mysql/mysql-deployment.yaml" >}}
{{< codenew file="application/mysql/mysql-pv.yaml" >}}

1.  YAML ファイルに記述された PV と PVC をデプロイします。

        kubectl create -f https://k8s.io/examples/application/mysql/mysql-pv.yaml

1.  YAML ファイルの内容をデプロイします。

        kubectl create -f https://k8s.io/examples/application/mysql/mysql-deployment.yaml

1.  作成した Deployment の情報を表示します。

        kubectl describe deployment mysql

        Name:                 mysql
        Namespace:            default
        CreationTimestamp:    Tue, 01 Nov 2016 11:18:45 -0700
        Labels:               app=mysql
        Annotations:          deployment.kubernetes.io/revision=1
        Selector:             app=mysql
        Replicas:             1 desired | 1 updated | 1 total | 0 available | 1 unavailable
        StrategyType:         Recreate
        MinReadySeconds:      0
        Pod Template:
          Labels:       app=mysql
          Containers:
           mysql:
            Image:      mysql:5.6
            Port:       3306/TCP
            Environment:
              MYSQL_ROOT_PASSWORD:      password
            Mounts:
              /var/lib/mysql from mysql-persistent-storage (rw)
          Volumes:
           mysql-persistent-storage:
            Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
            ClaimName:  mysql-pv-claim
            ReadOnly:   false
        Conditions:
          Type          Status  Reason
          ----          ------  ------
          Available     False   MinimumReplicasUnavailable
          Progressing   True    ReplicaSetUpdated
        OldReplicaSets:       <none>
        NewReplicaSet:        mysql-63082529 (1/1 replicas created)
        Events:
          FirstSeen    LastSeen    Count    From                SubobjectPath    Type        Reason            Message
          ---------    --------    -----    ----                -------------    --------    ------            -------
          33s          33s         1        {deployment-controller }             Normal      ScalingReplicaSet Scaled up replica set mysql-63082529 to 1

1.  Deployment によって作成された Pod を一覧表示します。

        kubectl get pods -l app=mysql

        NAME                   READY     STATUS    RESTARTS   AGE
        mysql-63082529-2z3ki   1/1       Running   0          3m

1.  PersistentVolumeClaim を確認します。

        kubectl describe pvc mysql-pv-claim

        Name:         mysql-pv-claim
        Namespace:    default
        StorageClass:
        Status:       Bound
        Volume:       mysql-pv-volume
        Labels:       <none>
        Annotations:    pv.kubernetes.io/bind-completed=yes
                        pv.kubernetes.io/bound-by-controller=yes
        Capacity:     20Gi
        Access Modes: RWO
        Events:       <none>

## MySQL インスタンスにアクセスする

前述の YAML ファイルは、クラスター内の他の Pod がデータベースにアクセスできるよ
うにする Service を作成します。 Service のオプションで`clusterIP: None`を指定す
ると、Service の DNS 名が Pod の IP アドレスに直接解決されます。このオプションは
、Service のバックエンドの Pod が 1 つのみであり、Pod の数を増やす予定がない場合
に適しています。

MySQL クライアントを実行してサーバーに接続します。

```
kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -ppassword
```

このコマンドは、クラスター内に MySQL クライアントを実行する新しい Pod を作成し
、Service を通じて MySQL サーバーに接続します。接続できれば、ステートフルな
MySQL データベースが稼働していることが確認できます。

```
Waiting for pod default/mysql-client-274442439-zyp6i to be running, status is Pending, pod ready: false
If you don't see a command prompt, try pressing enter.

mysql>
```

## アップデート

イメージまたは Deployment の他の部分は、`kubectl apply`コマンドを使用して通常ど
おりに更新できます。ステートフルアプリケーションに固有のいくつかの注意事項を以下
に記載します。

- アプリケーションをスケールしないでください。このセットアップは単一レプリカのア
  プリケーション専用です。下層にある PersistentVolume は 1 つの Pod にしかマウン
  トできません。クラスター化されたステートフルアプリケーションについては
  、[StatefulSet のドキュメント](/ja/docs/concepts/workloads/controllers/statefulset/)を
  参照してください。
- Deployment を定義する YAML ファイルでは`strategy: type: Recreate`を使用して下
  さい。この設定は Kubernetes にローリングアップデートを使用 _しない_ ように指示
  します。同時に複数の Pod を実行することはできないため、ローリングアップデート
  は使用できません。 `Recreate`戦略は、更新された設定で新しい Pod を作成する前に
  、最初の Pod を停止します。

## Deployment の削除

名前を指定してデプロイしたオブジェクトを削除します。

```
kubectl delete deployment,svc mysql
kubectl delete pvc mysql-pv-claim
kubectl delete pv mysql-pv-volume
```

PersistentVolume を手動でプロビジョニングした場合は、PersistentVolume を手動で削
除し、また、下層にあるリソースも解放する必要があります。動的プロビジョニング機能
を使用した場合は、PersistentVolumeClaim を削除すれば、自動的に PersistentVolume
も削除されます。一部の動的プロビジョナー(EBS や PD など)は、PersistentVolume を
削除すると同時に下層にあるリソースも解放します。

{{% /capture %}}

{{% capture whatsnext %}}

- [Deployment オブジェクト](/ja/docs/concepts/workloads/controllers/deployment/)に
  ついてもっと学ぶ

- [アプリケーションのデプロイ](/ja/docs/tasks/run-application/run-stateless-application-deployment/)に
  ついてもっと学ぶ

- [kubectl run のドキュメント](/docs/reference/generated/kubectl/kubectl-commands/#run)

- [Volumes](/docs/concepts/storage/volumes/)と[Persistent Volumes](/docs/concepts/storage/persistent-volumes/)

{{% /capture %}}
