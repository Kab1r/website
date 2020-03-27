---
reviewers:
title: ServiceとPodに対するDNS
content_template: templates/concept
weight: 20
---

{{% capture overview %}} このページでは Kubernetes による DNS サポートについて概
観します。 {{% /capture %}}

{{% capture body %}}

## イントロダクション

Kubernetes の DNS はクラスター上で DNS Pod と Service をスケジュールし、DNS の名
前解決をするために各コンテナに対して DNS Service の IP を使うように Kubelet を設
定します。

### 何が DNS 名を取得するか

クラスター内(DNS サーバーそれ自体も含む)で定義された全ての Service は DNS 名を割
り当てられます。デフォルトでは、クライアント Pod の DNS サーチリストは Pod 自身
のネームスペースと、クラスターのデフォルトドメインを含みます。  
下記の例でこの仕組みを説明します。

Kubernetes の`bar`というネームスペース内で`foo`という名前の Service があると仮定
します。`bar`ネームスペース内で稼働している Pod は、`foo`に対して DNS クエリを実
行するだけでこの Service を探すことができます。`bar`とは別の`quux`ネームスペース
内で稼働している Pod は、`foo.bar`に対して DNS クエリを実行するだけでこの
Service を探すことができます。

下記のセクションでは、サポートされているレコードタイプとレイアウトについて詳しく
まとめています。うまく機能する他のレイアウト、名前、またはクエリーは、実装の詳細
を考慮し、警告なしに変更されることがあります。  
最新の仕様に関する詳細は
、[Kubernetes における DNS ベースの Service ディスカバリ](https://github.com/kubernetes/dns/blob/master/docs/specification.md)を
参照ください。

## Service

### A レコード

"通常の"(Headless でない)Service は、`my-svc.my-namespace.svc.cluster.local`とい
う形式の DNS A レコードを割り当てられます。この A レコードはその Service の
ClusterIP へと名前解決されます。

"Headless"(ClusterIP なしの)Service もま
た`my-svc.my-namespace.svc.cluster.local`という形式の DNS A レコードを割り当てら
れます。通常の Service とは異なり、この A レコードは Service によって選択された
Pod の IP の一覧へと名前解決されます。クライアントはこの一覧の IP を使うか、その
一覧から標準のラウンドロビン方式によって選択された IP を使います…

### SRV レコード

SRV レコードは、通常の Service もしく
は[Headless Services](/ja/docs/concepts/services-networking/service/#headless-service)の
一部である名前付きポート向けに作成されます。それぞれの名前付きポートに対して、そ
の SRV レコード
は`_my-port-name._my-port-protocol.my-svc.my-namespace.svc.cluster.local`という
形式となります。  
通常の Service に対しては、この SRV レコード
は`my-svc.my-namespace.svc.cluster.local`という形式のドメイン名とポート番号へ名
前解決します。  
Headless Service に対しては、この SRV レコードは複数の結果を返します。それは
Service の背後にある各 Pod の 1 つを返すのと
、`auto-generated-name.my-svc.my-namespace.svc.cluster.local`という形式の Pod の
ドメイン名とポート番号を含んだ結果を返します。

## Pod

### A レコード

DNS が有効なとき、Pod は"`pod-ip-address.my-namespace.pod.cluster.local`"という
形式の A レコードを割り当てられます。

例えば、`default`ネームスペース内で`cluster.local`という DNS 名を持ち
、`1.2.3.4`という IP を持った Pod は次の形式のエントリーを持ちます。:
`1-2-3-4.default.pod.cluster.local`。

### Pod の hostname と subdomain フィールド

現在、Pod が作成されたとき、その Pod のホスト名は Pod の`metadata.name`フィール
ドの値となります。

Pod Spec は、オプションである`hostname`フィールドを持ち、Pod のホスト名を指定す
るために使うことができます。`hostname`が指定されたとき、`hostname`はその Pod の
名前よりも優先されます。例えば、`hostname`フィールドが"`my-host`"にセットされた
Pod を考えると、Pod はその hostname が"`my-host`"に設定されます。

Pod Spec はまた、オプションである`subdomain`フィールドも持ち、Pod のサブドメイン
名を指定するために使うことができます。例えば、"`my-namespace`"というネームスペー
ス内で`hostname`が`foo`とセットされていて、`subdomain`が`bar`とセットされている
Pod の場合、その Pod は"`foo.bar.my-namespace.svc.cluster.local`"という名前の完
全修飾ドメイン名(FQDN)を持つことになります。

例:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: default-subdomain
spec:
  selector:
    name: busybox
  clusterIP: None
  ports:
    - name: foo # 実際は、portは必要ありません。
      port: 1234
      targetPort: 1234
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox1
  labels:
    name: busybox
spec:
  hostname: busybox-1
  subdomain: default-subdomain
  containers:
    - image: busybox:1.28
      command:
        - sleep
        - "3600"
      name: busybox
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox2
  labels:
    name: busybox
spec:
  hostname: busybox-2
  subdomain: default-subdomain
  containers:
    - image: busybox:1.28
      command:
        - sleep
        - "3600"
      name: busybox
```

もしその Pod と同じネームスペース内で、同じサブドメインを持った Headless Service
が存在していた場合、クラスターの KubeDNS サーバーもまた、その Pod の完全修飾ドメ
イン名(FQDN)に対する A レコードを返します。例えば、"`busybox-1`"というホスト名で
、"`default-subdomain`"というサブドメインを持った Pod と、その Pod と同じネーム
スペース内にある"`default-subdomain`"という名前の Headless Service があると考え
ると、その Pod は自身の完全修飾ドメイン名(FQDN)を
"`busybox-1.default-subdomain.my-namespace.svc.cluster.local`"として扱います
。DNS はその Pod の IP を指し示す A レコードを返します。"`busybox1`"と
"`busybox2`"の両方の Pod はそれぞれ独立した A レコードを持ちます。

そのエンドポイントオブジェクトはその IP に加えて`hostname`を任意のエンドポイント
アドレスに対して指定できます。

{{< note >}} A レコードは Pod の名前に対して作成されないため、`hostname`は Pod
の A レコードが作成されるために必須となります。`hostname`を持たない
が`subdomain`を持つような Pod は、その Pod の IP アドレスを指し示す Headless
Service(`default-subdomain.my-namespace.svc.cluster.local`)に対する A レコードの
み作成します。 {{< /note >}}

### Pod の DNS ポリシー

DNS ポリシーは Pod 毎に設定できます。現在の Kubernetes では次のような Pod 固有の
DNS ポリシーをサポートしています。これらのポリシーは Pod Spec の`dnsPolicy`フィ
ールドで指定されます。

- "`Default`": その Pod は Pod が稼働している Node から名前解決の設定を継承しま
  す。詳細に関しては
  、[関連する議論](/docs/tasks/administer-cluster/dns-custom-nameservers/#inheriting-dns-from-the-node)を
  参照してください。
- "`ClusterFirst`": "`www.kubernetes.io`"のようなクラスタードメインのサフィック
  スにマッチしないような DNS クエリーは、Node から継承された上流のネームサーバー
  にフォワーディングされます。クラスター管理者は、追加の stub ドメインと上流の
  DNS サーバーを設定できます。
- "`ClusterFirstWithHostNet`": hostNetwork によって稼働している Pod に対しては、
  ユーザーは明示的に DNS ポリシーを"`ClusterFirstWithHostNet`"とセットするべきで
  す。
- "`None`": この設定では、Kubernetes の環境から DNS 設定を無視することができます
  。全ての DNS 設定は、Pod Spec 内の`dnsConfig`フィールドを指定して提供すること
  になっています。下記のセクションの[Pod's DNS config](#pod-s-dns-config)を参照
  ください。

{{< note >}} "Default"は、デフォルトの DNS ポリシーではありません。も
し`dnsPolicy`が明示的に指定されていない場合、"ClusterFirst"が使用されます。
{{< /note >}}

下記の例では、`hostNetwork`フィールドが`true`にセットされているため
、`dnsPolicy`が"`ClusterFirstWithHostNet`"とセットされている Pod を示します。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  containers:
    - image: busybox:1.28
      command:
        - sleep
        - "3600"
      imagePullPolicy: IfNotPresent
      name: busybox
  restartPolicy: Always
  hostNetwork: true
  dnsPolicy: ClusterFirstWithHostNet
```

### Pod の DNS 設定

Pod の DNS 設定は、ユーザーが Pod に対してその DNS 設定上でさらに制御するための
手段を提供します。

`dnsConfig`フィールドはオプションで、どのような設定の`dnsPolicy`でも共に機能する
ことができます。しかし、Pod の`dnsPolicy`が"`None`"にセットされていたとき
、`dnsConfig`フィールドは必ず指定されなくてはなりません。

下記の項目は、ユーザーが`dnsConfig`フィールドに指定可能なプロパティーとなります
。

- `nameservers`: その Pod に対する DNS サーバーとして使われる IP アドレスのリス
  トです。これは最大で 3 つの IP アドレスを指定することができます。Pod
  の`dnsPolicy`が"`None`"に指定されていたとき、そのリストは最低 1 つの IP アドレ
  スを指定しなければならず、もし指定されていなければ、それ以外の`dnsPolicy`の値
  の場合は、このプロパティーはオプションとなります。
- `searches`: Pod 内のホスト名のルックアップのための DNS サーチドメインのリスト
  です。このプロパティーはオプションです。指定されていたとき、このリストは選択さ
  れた DNS ポリシーから生成されたサーチドメイン名のベースとなるリストにマージさ
  れます。重複されているドメイン名は削除されます。Kubernetes では最大 6 つのサー
  チドメインの設定を許可しています。
- `options`: `name`プロパティー(必須)と`value`プロパティー(オプション)を持つよう
  な各オプジェクトのリストで、これはオプションです。このプロパティー内の内容は指
  定された DNS ポリシーから生成されたオプションにマージされます。重複されたエン
  トリーは削除されます。

下記のファイルはカスタム DNS 設定を持った Pod の例です。

{{< codenew file="service/networking/custom-dns.yaml" >}}

上記の Pod が作成されたとき、`test`コンテナは、コンテナ内の`/etc/resolv.conf`フ
ァイル内にある下記の内容を取得します。

```
nameserver 1.2.3.4
search ns1.svc.cluster.local my.dns.search.suffix
options ndots:2 edns0
```

IPv6 用のセットアップのためには、サーチパスと name server は下記のようにセットア
ップするべきです。

```
$ kubectl exec -it dns-example -- cat /etc/resolv.conf
nameserver fd00:79:30::a
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

### DNS 機能を利用可用なバージョン

Pod の DNS 設定と"`None`"という DNS ポリシーの利用可能なバージョンに関しては下記
の通りです。

| k8s version |     Feature support     |
| :---------: | :---------------------: |
|    1.14     |       ステーブル        |
|    1.10     | β 版 (デフォルトで有効) |
|     1.9     |          α 版           |

{{% /capture %}}

{{% capture whatsnext %}}

DNS 設定の管理方法に関しては
、[DNS Service の設定](/docs/tasks/administer-cluster/dns-custom-nameservers/)
を確認してください。

{{% /capture %}}
