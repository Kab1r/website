---
title: Initコンテナ(Init Containers)
content_template: templates/concept
weight: 40
---

{{% capture overview %}} このページでは、Init コンテナについて概観します。Init
コンテナとは、アプリケーションコンテナの前に実行され、アプリケーションコンテナの
イメージに存在しないセットアップスクリプトやユーティリティーを含んだ特別なコンテ
ナです。 {{% /capture %}}

この機能は Kubernetes1.6 から β 版の機能として存在しています。Init コンテナは
PodSpec 内で、アプリケーションの`containers`という配列と並べて指定されます。その
ベータ版のアノテーション値はまだ扱われ、PodSpec のフィールド値を上書きします。し
かしながら、それらは Kubernetes バージョン 1.6 と 1.7 において廃止されました
。Kubernetes バージョン 1.8 からはそのアノテーション値はサポートされず、PodSpec
フィールドの値に変換する必要があります。

{{% capture body %}}

## Init コンテナを理解する

単一の[Pod](/docs/concepts/workloads/pods/pod-overview/)は、Pod 内に複数のコンテ
ナを稼働させることができますが、Init コンテナもまた、アプリケーションコンテナが
稼働する前に 1 つまたは複数稼働できます。

Init コンテナは下記の項目をのぞいて、通常のコンテナと全く同じものとなります。

- Init コンテナは常に完了するまで稼働します。
- 各 Init コンテナは、次の Init コンテナが稼働する前に正常に完了しなくてはなりま
  せん。

もしある Pod の単一の Init コンテナが失敗した場合、Kubernetes は Init コンテナが
成功するまで何度もその Pod を再起動します。しかし、もしその Pod
の`restartPolicy`が`Never`の場合、再起動されません。

単一のコンテナを Init コンテナとして指定するためには、PodSpec にそのアプリケーシ
ョンの`containers`配列と並べて、`initContainers`フィールド
を[Container](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#container-v1-core)
型のオブジェクトの JSON 配列として指定してください。 Init コンテナのステータスは
、`.status.initContainerStatuses`フィールドにコンテナのステータスの配列として返
されます(`.status.containerStatuses`と同様)。

### 通常のコンテナとの違い

Init コンテナは、リソースリミット、ボリューム、セキュリティ設定などのアプリケー
ションコンテナの全てのフィールドと機能をサポートしています。しかし、Init コンテ
ナに対するリソースリクエストやリソースリミットの扱いは微妙に異なり、下記
の[Resources](#resources)にて説明します。また、Init コンテナはその Pod の準備が
できる前に完了しなくてはならないため、`readinessProbe`をサポートしていません。

もし複数の Init コンテナが単一の Pod に対して指定された場合、それらの Init コン
テナは 1 つずつ順番に実行されます。各 Init コンテナは次のコンテナが完了できる前
に完了しなくてはなりません。全ての Init コンテナが実行完了した時、Kubernetes は
Pod を初期化し、通常通りアプリケーションコンテナを稼働させます。

## Init コンテナは何に使用できるか?

Init コンテナはアプリケーションコンテナのイメージとは分離されているため、コンテ
ナの起動に関連したコードにおいていくつかの利点があります。

- セキュリティの理由からアプリケーションコンテナのイメージに含めたくないユーティ
  リティーを含んだり実行できます。
- アプリケーションのイメージに存在していないセットアップ用のユーティリティーやカ
  スタムコードを含むことができます。例えば、セットアップ中
  に`sed`、`awk`、`python`や、`dig`のようなツールを使うための他のイメージから、
  アプリケーションのイメージを作る必要がなくなります。
- アプリケーションイメージをひとまとめにしてビルドすることなく、アプリケーション
  のイメージ作成と、デプロイ処理を独立して行うことができます。
- アプリケーションコンテナと別のファイルシステムビューを持つために、Linux の名前
  空間を使用します。その結果、アプリケーションコンテナがアクセスできない箇所への
  シークレットなアクセス権限を得ることができます。
- Init コンテナはアプリケーションコンテナの実行の前に完了しますが、その一方で、
  複数のアプリケーションコンテナは並列に実行されます。そのため Init コンテナはい
  くつかの前提条件をセットされるまで、アプリケーションコンテナの起動をブロックし
  たり遅らせることができます。

### 使用例

ここでは Init コンテナの使用例を挙げます。

- シェルコマンドを使って単一の Service が作成されるのを待機します。

      for i in {1..100}; do sleep 1; if dig myservice; then exit 0; fi; done; exit 1

- コマンドを使って下位の API からこの Pod をリモートサーバに登録します。

      `curl -X POST http://$MANAGEMENT_SERVICE_HOST:$MANAGEMENT_SERVICE_PORT/register -d 'instance=$(<POD_NAME>)&ip=$(<POD_IP>)'`

- `sleep 60`のようなコマンドを使ってアプリケーションコンテナが起動する前に待機し
  ます。
- ボリュームにある git リポジトリをクローンします。
- メインのアプリケーションコンテナのための設定ファイルを動的に生成するために、い
  くつかの値を設定ファイルに移してテンプレートツールを稼働させます。例えば、設定
  ファイルにその Pod の POD_IP を移して、Jinja を使ってメインのアプリケーション
  コンテナの設定ファイルを生成します。

さらに詳細な使用例は
、[StatefulSets のドキュメント](/docs/concepts/workloads/controllers/statefulset/)と[Production Pods guide](/docs/tasks/configure-pod-container/configure-pod-initialization/)に
まとまっています。

### Init コンテナの使用

下記の Kubernetes1.5 用の yaml ファイルは、2 つの Init コンテナを含む単一のシン
プルなポッドの概要となります。最初の Init コンテナの例は、`myservies`、2 つ目の
Init コンテナは`mydb`の起動をそれぞれ待ちます。2 つのコンテナの実行が完了すると
Pod の起動が始まります。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
  annotations:
    pod.beta.kubernetes.io/init-containers: '[
        {
            "name": "init-myservice",
            "image": "busybox:1.28",
            "command": ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
        },
        {
            "name": "init-mydb",
            "image": "busybox:1.28",
            "command": ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
        }
    ]'
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
```

古いアノテーション構文が Kubernetes1.6 と 1.7 において有効ですが、1.6 では新しい
構文にも対応しています。Kubernetes1.8 以降では新しい構文を使用する必要があります
。Kubernetes では Init コンテナの宣言を`spec`に移行させました。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
    - name: myapp-container
      image: busybox:1.28
      command: ["sh", "-c", "echo The app is running! && sleep 3600"]
  initContainers:
    - name: init-myservice
      image: busybox:1.28
      command:
        [
          "sh",
          "-c",
          "until nslookup myservice; do echo waiting for myservice; sleep 2;
          done;",
        ]
    - name: init-mydb
      image: busybox:1.28
      command:
        [
          "sh",
          "-c",
          "until nslookup mydb; do echo waiting for mydb; sleep 2; done;",
        ]
```

Kubernetes1.5 での構文は 1.6 においても稼働しますが、1.6 の構文の使用を推奨しま
す。Kubernetes1.6 において、API 内で Init コンテナのフィールド作成されます。ベー
タ版のアノテーションは Kubernetes1.6 と 1.7 において有効ですが、1.8 以降ではサポ
ートされません。

下記の yaml ファイルは`mydb`と`myservice`という Service の概要です。

```yaml
kind: Service
apiVersion: v1
metadata:
  name: myservice
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
---
kind: Service
apiVersion: v1
metadata:
  name: mydb
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9377
```

この Pod は、下記のコマンドによって起動とデバッグすることが可能です。

```shell
kubectl apply -f myapp.yaml
```

```
pod/myapp-pod created
```

```shell
kubectl get -f myapp.yaml
```

```
NAME        READY     STATUS     RESTARTS   AGE
myapp-pod   0/1       Init:0/2   0          6m
```

```shell
kubectl describe -f myapp.yaml
```

```
Name:          myapp-pod
Namespace:     default
[...]
Labels:        app=myapp
Status:        Pending
[...]
Init Containers:
  init-myservice:
[...]
    State:         Running
[...]
  init-mydb:
[...]
    State:         Waiting
      Reason:      PodInitializing
    Ready:         False
[...]
Containers:
  myapp-container:
[...]
    State:         Waiting
      Reason:      PodInitializing
    Ready:         False
[...]
Events:
  FirstSeen    LastSeen    Count    From                      SubObjectPath                           Type          Reason        Message
  ---------    --------    -----    ----                      -------------                           --------      ------        -------
  16s          16s         1        {default-scheduler }                                              Normal        Scheduled     Successfully assigned myapp-pod to 172.17.4.201
  16s          16s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Pulling       pulling image "busybox"
  13s          13s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Pulled        Successfully pulled image "busybox"
  13s          13s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Created       Created container with docker id 5ced34a04634; Security:[seccomp=unconfined]
  13s          13s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Started       Started container with docker id 5ced34a04634
```

```shell
kubectl logs myapp-pod -c init-myservice # 1つ目のInitコンテナを調査する
kubectl logs myapp-pod -c init-mydb      # 2つ目のInitコンテナを調査する
```

一度`mydq`と`myservice` Service を起動させると、Init コンテナが完了し
て`myapp-pod`が作成されるのを確認できます。

```shell
kubectl apply -f services.yaml
```

```
service/myservice created
service/mydb created
```

```shell
kubectl get -f myapp.yaml
NAME        READY     STATUS    RESTARTS   AGE
myapp-pod   1/1       Running   0          9m
```

この例は非常にシンプルですが、ユーザー独自の Init コンテナを作成するためのインス
ピレーションを提供するでしょう。

## Init コンテナのふるまいに関する詳細

単一の Pod が起動している間、ネットワークとボリュームが初期化されたのちに、Init
コンテナは順番に起動されます。各 Init コンテナは次の Init コンテナが起動する前に
完了しなくてはなりません。もしある Init コンテナがランタイムもしくはエラーにより
起動失敗した場合、その Pod の`restartPolicy`の値をもとにリトライされます。しかし
、もし Pod の`restartPolicy`が`Always`に設定されていた場合、その Init コンテナ
の`restartPolicy`は`OnFailure`となります。

Pod は全ての Init コンテナが完了するまで`Ready`状態となりません。Init コンテナ上
のポートは Service によって集約されません。初期化中の Pod のステータス
は`Pending`となりますが、`Initializing`という値は true となります。

もしその Pod が[再起動](#pod-restart-reasons)されたとき、全ての Init コンテナは
再度実行されなくてはなりません。

Init コンテナのスペックに対する変更はコンテナのイメージフィールドのみに限定され
ます。 Init コンテナのイメージフィールド値の変更は、その Pod の再起動することと
等しいです。

Init コンテナは何度も再起動、リトライ可能なため、べき等(Idempotent)である必要が
あります。特に、`EmptyDirs`にファイルを書き込むコードは、書き込み先のファイルが
すでに存在している可能性を考慮に入れるべきです。

Init コンテナはアプリケーションコンテナの全てのフィールドを持っています。しかし
Kubernetes は、Init コンテナが完了と異なる状態を定義できないた
め`readinessProbe`が使用されることを禁止しています。これはバリデーションの際に強
要されます。

Init コンテナがずっと失敗し続けたままの状態を防ぐために、Pod
に`activeDeadlineSeconds`、コンテナに`livenessProbe`の設定をそれぞれ使用してく�
さい。`activeDeadlineSeconds`の設定は Init コンテナにも適用されます。

ある Pod 内の各アプリケーションコンテナと Init コンテナの名前はユニークである必
要があります。他のコンテナと同じ名前を共有していた場合、バリデーションエラーが返
されます。

### リソース

Init コンテナの順序と実行を考えるとき、リソースの使用に関して下記のルールが適用
されます。

- 全ての Init コンテナの中で定義された最も高いリソースリクエストとリソースリミッ
  トが、_有効な Init リクエストとリミット_ になります。
- Pod のリソースの*有効なリクエストとリミット* は、下記の 2 つの中のどちらか高い
  方となります。
  - そのリソースの全てのアプリケーションコンテナのリクエストとリミットの合計
  - そのリソースの有効な Init リクエストとリミット
- スケジューリングは有効なリクエストとリミットに基づいて実行されます。これは
  Init コンテナがその Pod の生存中に使われない初期化のためのリソースを保持するこ
  とができることを意味しています。
- Pod の*有効な Qos ティアー* は、Init コンテナとアプリケーションコンテナで同様
  です。

クォータとリミットは有効な Pod リクエストとリミットに基づいて適用されます。

Pod レベルの cgroups は、スケジューラーと同様に、有効な Pod リクエストとリミット
に基づいています。

### Pod の再起動の理由

単一の Pod は再起動可能で、Init コンテナの再実行も引き起こします。それらは下記の
理由によるものです。

- あるユーザーが、その Pod の Init コンテナのイメージを変更するように PodSpec を
  更新する場合。アプリケーションコンテナのイメージの変更はそのアプリケーションコ
  ンテナの再起動のみ行われます。
- その Pod のインフラストラクチャーコンテナが再起動された場合。これはあまり起き
  るものでなく、Node に対するルート権限を持ったユーザーにより行われることがあり
  ます。
- `restartPolicy`が`Always`と設定されているとき、単一 Pod 内の全てのコンテナが停
  止され、再起動が行われた時と、ガーベージコレクションにより Init コンテナの完了
  記録が失われた場合。

## サポートと互換性

ApiServer のバージョン 1.6.0 かそれ以上のバージョンのクラスターは
、`.spec.initContainers`フィールドを使った Init コンテナの機能をサポートしていま
す。
それ以前のバージョンでは、α 版か β 版のアノテーションを使って Init コンテナを使
用できます。また、`.spec.initContainers`フィールドは、Kubernetes1.3.0 かそれ以上
のバージョンで Init コンテナを使用できるようにするためと、ApiServer バージョン
1.6 において、1.5.x などの古いバージョンにロールバックできるようにするために、α
版か β 版のアノテーションにミラーされ、存在する Pod の Init コンテナの機能が失わ
れることが無いように安全にロールバックできるようにします。

ApiServer と Kubelet バージョン 1.8.0 かそれ以上のバージョンでは、α 版と β 版の
アノテーションは削除されており、廃止されたアノテーション
は`.spec.initContainers`フィールドへの移行が必須となります。

{{% /capture %}}

{{% capture whatsnext %}}

- [Init コンテナを持っている Pod の作成](/docs/tasks/configure-pod-container/configure-pod-initialization/#creating-a-pod-that-has-an-init-container)

{{% /capture %}}
