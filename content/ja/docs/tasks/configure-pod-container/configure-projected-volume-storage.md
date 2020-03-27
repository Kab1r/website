---
title: ストレージにProjectedボリュームを使用するようPodを設定する
content_template: templates/task
weight: 70
---

{{% capture overview %}} このページでは
、[`projected`](/docs/concepts/storage/volumes/#projected)（投影）ボリュームを使
用して、既存の複数のボリュームソースを同一ディレクトリ内にマウントする方法を説明
します。現在、`secret`、`configMap`、`downwardAPI`および`serviceAccountToken`ボ
リュームを投影できます。

{{< note >}} `serviceAccountToken`はボリュームタイプではありません。
{{< /note >}} {{% /capture %}}

{{% capture prerequisites %}} {{< include "task-tutorial-prereqs.md" >}}
{{< version-check >}} {{% /capture %}}

{{% capture steps %}}

## Projected ボリュームを Pod に設定する

この課題では、ローカルファイルからユーザーネームおよびパスワード
の{{< glossary_tooltip text="Secret" term_id="secret" >}}を作成します。次に、単
一のコンテナを実行する Pod を作成し
、[`projected`](/docs/concepts/storage/volumes/#projected)ボリュームを使用してそ
れぞれの Secret を同じ共有ディレクトリにマウントします。

以下に Pod の設定ファイルを示します:

{{< codenew file="pods/storage/projected.yaml" >}}

1. Secret を作成します:

   ```shell
   # ユーザーネームおよびパスワードを含むファイルを作成します:
   echo -n "admin" > ./username.txt
   echo -n "1f2d1e2e67df" > ./password.txt

   # これらのファイルからSecretを作成します:
   kubectl create secret generic user --from-file=./username.txt
   kubectl create secret generic pass --from-file=./password.txt
   ```

1. Pod を作成します:

   ```shell
   kubectl apply -f https://k8s.io/examples/pods/storage/projected.yaml
   ```

1. Pod 内のコンテナが実行されていることを確認するため、Pod の変更を監視します:

   ```shell
   kubectl get --watch pod test-projected-volume
   ```

   出力は次のようになります:

   ```
   NAME                    READY     STATUS    RESTARTS   AGE
   test-projected-volume   1/1       Running   0          14s
   ```

1. 別の端末にて、実行中のコンテナへのシェルを取得します:

   ```shell
   kubectl exec -it test-projected-volume -- /bin/sh
   ```

1. シェル内にて、投影されたソースを含む`projected-volume`ディレクトリが存在する
   ことを確認します:

   ```shell
   ls /projected-volume/
   ```

## クリーンアップ

Pod および Secret を削除します:

```shell
kubectl delete pod test-projected-volume
kubectl delete secret user pass
```

{{% /capture %}}

{{% capture whatsnext %}}

- [`projected`](/docs/concepts/storage/volumes/#projected)ボリュームについてさら
  に学ぶ
- [all-in-one ボリューム](https://github.com/kubernetes/community/blob/{{< param
  "githubbranch" >}}/contributors/design-proposals/node/all-in-one-volume.md)の
  デザインドキュメントを読む {{% /capture %}}
