---
title: Service
feature:
  title: サービスディスカバリーと負荷分散
  description: >
    Kubernetesでは、なじみのないサービスディスカバリーのメカニズムを使用するためにユーザーがアプリケーションの修正をする必要はありません。KubernetesはPodにそれぞれのIPアドレス割り振りや、Podのセットに対する単一のDNS名を提供したり、それらのPodのセットに対する負荷分散が可能です。

content_template: templates/concept
weight: 10
---

{{% capture overview %}}

{{< glossary_definition term_id="service" length="short" >}}

Kubernetes では、なじみのないサービスディスカバリーのメカニズムを使用するために
ユーザーがアプリケーションの修正をする必要はありません。  
Kubernetes は Pod にそれぞれの IP アドレス割り振りや、Pod のセットに対する単一の
DNS 名を提供したり、それらの Pod のセットに対する負荷分散が可能です。

{{% /capture %}}

{{% capture body %}}

## Service を利用する動機

{{< glossary_tooltip term_id="pod" text="Pod" >}}は停止が想定して設計されていま
す。 Pod が作成され、もしそれらが停止する時、Pod は再作成されません。
{{< glossary_tooltip term_id="deployment" >}}をアプリケーションを稼働させるため
に使用すると、Pod を動的に作成・削除してくれます。

各 Pod はそれ自身の IP アドレスを持ちます。しかし Deployment では、ある時点にお
いて同時に稼働している Pod のセットは、その後のある時点において稼働している Pod
のセットとは異なる場合があります。

この仕組みはある問題を引き起こします。もし、ある Pod のセット(ここでは"バックエ
ンド"と呼びます)がクラスター内で他の Pod のセット(ここでは"フロントエンド"と呼び
ます)に対して機能を提供する場合、フロントエンドの Pod がワークロードにおけるバッ
クエンドを使用するために、バックエンドの Pod の IP アドレスを探し出したり、記録
し続けるためにはどうすればよいでしょうか?

ここで*Service* について説明します。

## Service リソース {#service-resource}

Kubernetes において、Service は Pod の論理的なセットや、その Pod のセットにアク
セスするためのポリシーを定義します(このパターンはよくマイクロサービスと呼ばるこ
とがあります)。  
Service によってターゲットとされた Pod のセットは、たいてい
{{< glossary_tooltip text="セレクター" term_id="selector" >}} (セレクターなしの
Service を利用したい場合は[下記](#services-without-selectors)を参照してください)
によって定義されます。

例えば、3 つのレプリカが稼働しているステートレスな画像処理用のバックエンドを考え
ます。これらのレプリカは代替可能です。&mdash; フロントエンドはバックエンドが何で
あろうと気にしません。バックエンドのセットを構成する実際の Pod のセットが変更さ
れた際、フロントエンドクライアントはその変更を気にしたり、バックエンドの Pod の
セットの情報を記録しておく必要はありません。

Service による抽象化は、クライアントからバックエンドの Pod の管理する責務を分離
することを可能にします。

### クラウドネイティブのサービスディスカバリー

アプリケーション内でサービスディスカバリーのために Kubernetes API が使える場合、
ユーザーはエンドポイント
を{{< glossary_tooltip text="API Server" term_id="kube-apiserver" >}}に問い合わ
せることができ、また Service 内の Pod のセットが変更された時はいつでも更新された
エンドポイントの情報を取得できます。

非ネイティブなアプリケーションのために、Kubernetes はアプリケーションとバックエ
ンド Pod の間で、ネットワークポートやロードバランサーを配置する方法を提供します
。

## Service の定義

Kubernetes の Service は Pod と同様に REST のオブジェクトです。他の REST オブジ
ェクトと同様に、ユーザーは Service の新しいインスタンスを作成するために API サー
バーに対して Service の定義を`POST`できます。

例えば、TCP で 9376 番ポートで待ち受けていて、`app=Myapp`というラベルをもつ Pod
のセットがあるとします。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

この定義では、"my-service"という名前のついた新しい Service オブジェクトを作成し
ます。これは`app=Myapp`ラベルのついた各 Pod 上で TCP の 9376 番ポートをターゲッ
トとします。

Kubernetes は、この Service に対して IP アドレス("clusterIP"とも呼ばれます)を割
り当てます。これは Service のプロキシーによって使用されます(下記
の[仮想 IP と Service プロキシー](#virtual-ips-and-service-proxies)を参照くださ
い)。

Service セレクターのコントローラーはセレクターに一致する Pod を継続的にスキャン
し、“my-service”という名前の Endpoints オブジェクトに対して変更を POST します。

{{< note >}} Service は`port`から`targetPort`へのマッピングを行います。デフォル
トでは、利便性のために`targetPort`フィールドは`port`フィールドと同じ値で設定され
ます。 {{< /note >}}

Pod 内のポートの定義は名前を設定でき、Service の`targetPort`属性にてその名前を参
照できます。これは単一の設定名をもつ Service 内で、複数の種類の Pod が混合してい
たとしても有効で、異なるポート番号を介することによって利用可能な、同一のネットワ
ークプロトコルを利用します。この仕組みは Service をデプロイしたり、設定を追加す
る場合に多くの点でフレキシブルです。例えば、バックエンドソフトウェアにおいて、次
のバージョンで Pod が公開するポート番号を変更するときに、クライアントの変更なし
に行えます。

Service のデフォルトプロトコルは TCP です。また、他
の[サポートされているプロトコル](#protocol-support)も利用可能です。

多くの Service が、1 つ以上のポートを公開する必要があるように、Kubernetes は 1
つの Service オブジェクトに対して複数のポートの定義をサポートしています。  
各ポート定義は同一の`protocol`または異なる値を設定できます。

### セレクターなしの Service {#services-without-selectors}

Service は多くの場合、Kubernetes の Pod に対するアクセスを抽象化しますが、他の種
類のバックエンドも抽象化できます。例えば:

- プロダクション環境で外部のデータベースクラスターを利用したいが、テスト環境では
  、自身のクラスターが持つデータベースを利用したい場合
- Service を、異なる Namespace 内の Service や他のクラスターの Service に向ける
  場合
- ワークロードを Kubernetes に移行するとき、アプリケーションに対する処理をしなが
  ら、バックエンドの一部を Kubernetes で実行する場合

このような場合において、ユーザーは Pod セレクター*なしで* Service を定義できます
。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

この Service はセレクターがないため、対応する Endpoints オブジェクトは自動的に作
成されません。  
ユーザーは Endpoints オブジェクトを手動で追加することにより、向き先のネットワー
クアドレスとポートを手動でマッピングできます。

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service
subsets:
  - addresses:
      - ip: 192.0.2.42
    ports:
      - port: 9376
```

{{< note >}} Endpoints の ip は、loopback (127.0.0.0/8 for IPv4, ::1/128 for
IPv6), や link-local (169.254.0.0/16 and 224.0.0.0/24 for IPv4, fe80::/64 for
IPv6)に設定することができません。

{{< glossary_tooltip term_id="kube-proxy" >}}が仮想 IP を最終的な到達先に設定す
ることをサポートしていないため、Endpoints の ip アドレスは他の Kubernetes
Service の ClusterIP にすることができません。 {{< /note >}}

セレクターなしの Service へのアクセスは、セレクターをもっている Service と同じよ
うにふるまいます。上記の例では、トラフィックは YAML ファイル内
で`192.0.2.42:9376` (TCP)で定義された単一のエンドポイントにルーティングされます
。

ExternalName Service はセレクターの代わりに DNS 名を使用する特殊なケースの
Service です。さらなる情報は、このドキュメントの後で紹介す
る[ExternalName](#externalname)を参照ください。

### エンドポイントスライス

{{< feature-state for_k8s_version="v1.17" state="beta" >}}

エンドポイントスライスは、Endpoints に対してよりスケーラブルな代替手段を提供でき
る API リソースです。概念的には Endpoints に非常に似ていますが、エンドポイントス
ライスを使用すると、ネットワークエンドポイントを複数のリソースに分割できます。デ
フォルトでは、エンドポイントスライスは、100 個のエンドポイントに到達すると「いっ
ぱいである」と見なされ、その時点で追加のエンドポイントスライスが作成され、追加の
エンドポイントが保存されます。

エンドポイントスライスは
、[エンドポイントスライスのドキュメント](/docs/concepts/services-networking/endpoint-slices/)に
て詳しく説明されている追加の属性と機能を提供します。

## 仮想 IP とサービスプロキシー {#virtual-ips-and-service-proxies}

Kubernetes クラスターの各 Node は`kube-proxy`を稼働させています
。`kube-proxy`は[`ExternalName`](#externalname)タイプ以外の`Service`用に仮想 IP
を実装する責務があります。

### なぜ、DNS ラウンドロビンを使わないのでしょうか。

ここで湧き上がる質問として、なぜ Kubernetes は内部のトラフィックをバックエンドへ
転送するためにプロキシーに頼るのでしょうか。  
他のアプローチはどうなのでしょうか。例えば、複数の A バリュー(もしくは IPv6 用に
AAAA バリューなど)をもつ DNS レコードを設定し、ラウンドロビン方式で名前を解決す
ることは可能でしょうか。

Service においてプロキシーを使う理由はいくつかあります。

- DNS の実装がレコードの TTL をうまく扱わず、期限が切れた後も名前解決の結果をキ
  ャッシュするという長い歴史がある。
- いくつかのアプリケーションでは DNS ルックアップを 1 度だけ行い、その結果を無期
  限にキャッシュする。
- アプリケーションとライブラリーが適切な DNS 名の再解決を行ったとしても、DNS レ
  コード上の 0 もしくは低い値の TTL が DNS に負荷をかけることがあり、管理が難し
  い。

### user-space プロキシーモード {#proxy-mode-userspace}

このモードでは、kube-proxy は Service や Endpoints オブジェクトの追加・削除をチ
ェックするために、Kubernetes Master を監視します。  
各 Service は、ローカルの Node 上でポート(ランダムに選ばれたもの)を公開します。
この"プロキシーポート"に対するどのようなリクエストも、その Service のバックエン
ド Pod のどれか 1 つにプロキシーされます(Endpoints を介して通知された Pod に対し
て)。  
kube-proxy は、どのバックエンド Pod を使うかを決める際に Service
の`SessionAffinity`項目の設定を考慮に入れます。

最後に、user-space プロキシーは Service の`clusterIP`(仮想 IP)と`port`に対するト
ラフィックをキャプチャする iptables ルールをインストールします。  
そのルールは、トラフィックをバックエンド Pod にプロキシーするためのプロキシーポ
ートにリダイレクトします。

デフォルトでは、user-space モードにおける kube-proxy はラウンドロビンアルゴリズ
ムによってバックエンド Pod を選択します。

![user-spaceプロキシーのService概要ダイアグラム](/images/docs/services-userspace-overview.svg)

### `iptables`プロキシーモード {#proxy-mode-iptables}

このモードでは、kube-proxy は Service や Endpoints オブジェクトの追加・削除のチ
ェックのために Kubernetes コントロールプレーンを監視します。  
各 Service では、その Service の`clusterIP`と`port`に対するトラフィックをキャプ
チャする iptables ルールをインストールし、そのトラフィックを Service のあるバッ
クエンドのセットに対してリダイレクトします。  
各 Endpoints オブジェクトは、バックエンドの Pod を選択する iptables ルールをイン
ストールします。

デフォルトでは、iptables モードにおける kube-proxy はバックエンド Pod をランダム
で選択します。

トラフィックのハンドリングのために iptables を使用すると、システムのオーバーヘッ
ドが少なくなります。これは、トラフィックが Linux の netfilter によって
user-space と kernel-space を切り替える必要がないためです。  
このアプローチは、オーバーヘッドが少ないことに加えて、より信頼できる方法でもあり
ます。

kube-proxy が iptables モードで稼働し、最初に選択された Pod が応答しない場合、そ
のコネクションは失敗します。  
これは user-space モードでの挙動と異なります: user-space モードにおいては
、kube-proxy は最初の Pod に対するコネクションが失敗したら、自動的に他のバックエ
ンド Pod に対して再接続を試みます。

iptables モードの kube-proxy が正常なバックエンド Pod のみをリダイレクト対象とす
るために、Pod
の[ReadinessProbe](/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)を
使用してバックエンド Pod が正常に動作しているか確認できます。これは、ユーザーが
kube-proxy を介して、コネクションに失敗した Pod に対してトラフィックをリダイレク
トするのを除外することを意味します。

![iptablesプロキシーのService概要ダイアグラム](/images/docs/services-iptables-overview.svg)

### IPVS プロキシーモード {#proxy-mode-ipvs}

{{< feature-state for_k8s_version="v1.11" state="stable" >}}

`ipvs`モードにおいて、kube-proxy は Service と Endpoints オブジェクトを監視し
、IPVS ルールを作成するために`netlink`インターフェースを呼び出し、定期的に
Kubernetes の Service と Endpoints と IPVS ルールを同期させます。  
このコントロールループは IPVS のステータスが理想的な状態になることを保証します
。  
Service にアクセスするとき、IPVS はトラフィックをバックエンドの Pod に向けます。

IPVS プロキシーモードは iptables モードと同様に、netfilter のフック関数に基づい
ています。ただし、基礎となるデータ構造としてハッシュテーブルを使っているのと
、kernel-space で動作します。  
これは、IPVS モードにおける kube-proxy は iptables モードに比べてより低いレイテ
ンシーでトラフィックをリダイレクトし、プロキシーのルールを同期する際にはよりパフ
ォーマンスがよいことを意味します。　　他のプロキシーモードと比較して、IPVS モー
ドはより高いネットワークトラフィックのスループットをサポートしています。

IPVS はバックエンド Pod に対するトラフィックのバランシングのために多くのオプショ
ンを下記のとおりに提供します。

- `rr`: ラウンドロビン
- `lc`: 最低コネクション数(オープンされているコネクション数がもっとも小さいもの)
- `dh`: 送信先 IP によって割り当てられたハッシュ値をもとに割り当てる(Destination
  Hashing)
- `sh`: 送信元 IP によって割り当てられたハッシュ値をもとに割り当てる(Source
  Hashing)
- `sed`: 見込み遅延が最小なもの
- `nq`: キューなしスケジューリング

{{< note >}} IPVS モードで kube-proxy を稼働させるためには、kube-proxy を稼働さ
せる前に Node 上で IPVS を有効にしなければなりません。

kube-proxy は IPVS モードで起動する場合、IPVS カーネルモジュールが利用可能かどう
かを確認します。  
もし IPVS カーネルモジュールが見つからなかった場合、kube-proxy は iptables モー
ドで稼働するようにフォールバックされます。 {{< /note >}}

![IPVSプロキシーのService概要ダイアグラム](/images/docs/services-ipvs-overview.svg)

このダイアグラムのプロキシーモデルにおいて、Service の IP:Port に対するトラフィ
ックは、クライアントが Kubernetes の Service や Pod について何も知ることなく適切
にバックエンドにプロキシーされています。

特定のクライアントからのコネクションが、毎回同一の Pod にリダイレクトされるよう
にするためには、`service.spec.sessionAffinity`を"ClientIP"にセットすることにより
、クライアントの IP アドレスに基づいた SessionAffinity を選択することができます(
デフォルトは"None")。また
、`service.spec.sessionAffinityConfig.clientIP.timeoutSeconds`を適切に設定するこ
とにより、セッションのタイムアウト時間を設定できます(デフォルトではこの値は
18,000 で、3 時間となります)。

## 複数のポートを公開する Service

いくつかの Service において、ユーザーは 1 つ以上のポートを公開する必要があります
。Kubernetes は、Service オブジェクト上で複数のポートを定義するように設定できま
す。  
Service で複数のポートを使用するとき、どのポートかを明確にするために、複数のポー
ト全てに対して名前をつける必要があります。例えば:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
    - name: https
      protocol: TCP
      port: 443
      targetPort: 9377
```

{{< note >}} Kubernetes の Pod 名と同様に、ポート名は小文字の英数字と`-`のみ含め
る必要があります。また、ポート名の最初と最後の文字は英数字である必要があります。

例えば、`123-abc`や`web`という名前は有効で、`123_abc`や`-web`は無効です。
{{< /note >}}

## ユーザー所有の IP アドレスを選択する

`Service`を作成するリクエストの一部として、ユーザー所有の clusterIP アドレスを指
定することができます。  
これを行うためには`.spec.clusterIP`フィールドにセットします。  
使用例として、もしすでに再利用したい DNS エントリーが存在していた場合や、特定の
IP アドレスを設定されたレガシーなシステムや、IP の再設定が難しい場合です。

ユーザーが指定した IP アドレスは、その API サーバーのために設定されてい
る`service-cluster-ip-range`という CIDR レンジ内の有効な IPv4 または IPv6 アドレ
スである必要があります。  
もし無効な clusterIP アドレスの値を設定して Service を作成した場合、問題があるこ
とを示すために API サーバーは HTTP ステータスコード 422 を返します。

## サービスディスカバリー

Kubernetes は、Service オブジェクトを見つけ出すために 2 つの主要なモードをサポー
トしています。 - それは環境変数と DNS です。

### 環境変数

Pod が Node 上で稼働するとき、kubelet はアクティブな各 Service に対して、環境変
数のセットを追加します。これ
は[Docker links 互換性](https://docs.docker.com/userguide/dockerlinks/)のある変
数( [makeLinkVariables 関数](http://releases.k8s.io/{{< param
"githubbranch" >}}/pkg/kubelet/envvars/envvars.go#L72)を確認してください)や、よ
り簡単な`{SVCNAME}_SERVICE_HOST`や、`{SVCNAME}_SERVICE_PORT`変数をサポートします
。この変数名で使われる Service 名は大文字に変換され、`-`は`_`に変換されます。

例えば、TCP ポート 6379 番を公開していて、さらに clusterIP が 10.0.0.11 に割り当
てられている`"redis-master"`という Service は、下記のような環境変数を生成します
。

```shell
REDIS_MASTER_SERVICE_HOST=10.0.0.11
REDIS_MASTER_SERVICE_PORT=6379
REDIS_MASTER_PORT=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP_PROTO=tcp
REDIS_MASTER_PORT_6379_TCP_PORT=6379
REDIS_MASTER_PORT_6379_TCP_ADDR=10.0.0.11
```

{{< note >}} Service にアクセスする必要のある Pod があり、クライアントであるその
Pod に対して環境変数を使ってポートと clusterIP を公開する場合、クライアントの
Pod が存在する*前に* Service を作成しなくてはなりません。  
そうでない場合、クライアントの Pod はそれらの環境変数を作成しません。

Service の clusterIP を発見するために DNS のみを使う場合、このような問題を心配す
る必要はありません。 {{< /note >}}

### DNS

ユーザーは[アドオン](/docs/concepts/cluster-administration/addons/)を使って
Kubernetes クラスターに DNS Service をセットアップできます(常にセットアップすべ
きです)。

CoreDNS などのクラスター対応の DNS サーバーは新しい Service や、各 Service 用の
DNS レコードのセットのために Kubernetes API を常に監視します。  
もしクラスターを通して DNS が有効になっている場合、全ての Pod は DNS 名によって
自動的に Service に対する名前解決をするようにできるはずです。

例えば、Kubernetes の`"my-ns"`という Namespace 内で`"my-service"`という Service
がある場合、Kubernetes コントロールプレーンと DNS Service が協調して動作し
、`"my-service.my-ns"`という DNS レコードを作成します。  
`"my-ns"`という Namespace 内の Pod は`my-service`という名前で簡単に名前解決でき
るはずです(`"my-service.my-ns"`でも動作します)。

他の Namespace 内での Pod は`my-service.my-ns`といった形で指定しなくてはなりませ
ん。これらの DNS 名は、その Service の clusterIP に名前解決されます。

Kubernetes は名前付きのポートに対する DNS SRV(Service)レコードもサポートしていま
す。もし`"my-service.my-ns"`という Service が`"http"`という名前の TCP ポートを持
っていた場合、IP アドレスと同様に、`"http"`のポート番号を探すため
に`_http._tcp.my-service.my-ns`という DNS SRV クエリを実行できます。

Kubernetes の DNS サーバーは`ExternalName` Service にアクセスする唯一の方法です
。  
[DNS Pods と Service](/docs/concepts/services-networking/dns-pod-service/)に
て`ExternalName`による名前解決に関するさらなる情報を確認できます。

## Headless Service {#headless-service}

場合によっては、負荷分散と単一の Service IP は不要です。このケースにおいて
、clusterIP(`.spec.clusterIP`)の値を`"None"`に設定することにより、"Headless"とよ
ばれる Service を作成できます。

ユーザーは、Kubernetes の実装と紐づくことなく、他のサービスディスカバリーのメカ
ニズムと連携するために Headless Service を使用できます。  
例えば、ユーザーはこの API 上でカスタ
ム{{< glossary_tooltip term_id="operator-pattern" text="オペレーター" >}}を実装
することができます。

この`Service`においては、clusterIP は割り当てられず、kube-proxy はこの Service
をハンドリングしないのと、プラットフォームによって行われるはずのロードバランシン
グやプロキシーとしての処理は行われません。DNS がどのように自動で設定されるかは、
定義された Service が定義されたラベルセレクターを持っているかどうかに依存します
。

### ラベルセレクターの利用

ラベルセレクターを定義した Headless Service において、Endpoints コントローラーは
API において`Endpoints`レコードを作成し、`Service`のバックエンドにある`Pod`への
IP を直接指し示すために DNS 設定を修正します。

### ラベルセレクターなしの場合

ラベルセレクターを定義しない Headless Service においては、Endpoints コントローラ
ーは`Endpoints`レコードを作成しません。  
しかし DNS のシステムは下記の 2 つ両方を探索し、設定します。

- [`ExternalName`](#externalname)タイプの Service に対する CNAME レコード
- 他の全ての Service タイプを含む、Service 名を共有している全ての`Endpoints`レコ
  ード

## Service の公開 (Service のタイプ) {#publishing-services-service-types}

ユーザーのアプリケーションのいくつかの部分において(例えば、frontends など)、ユー
ザーのクラスターの外部にある IP アドレス上で Service を公開したい場合があります
。

Kubernetes の`ServiceTypes`によって、ユーザーがどのような種類の Service を使いた
いかを指定することが可能です。  
デフォルトでは`ClusterIP`となります。

`Type`項目の値と、そのふるまいは以下のようになります。

- `ClusterIP`: クラスター内部の IP で Service を公開する。このタイプでは Service
  はクラスター内部からのみ疎通性があります。このタイプはデフォルト
  の`ServiceType`です。
- [`NodePort`](#nodeport): 各 Node の IP にて、静的なポート(`NodePort`)上で
  Service を公開します。その`NodePort` の Service が転送する先の`ClusterIP`
  Service が自動的に作成されます。`<NodeIP>:<NodePort>`にアクセスすることによっ
  て`NodePort` Service にアクセスできるようになります。
- [`LoadBalancer`](#loadbalancer): クラウドプロバイダーのロードバランサーを使用
  して、Service を外部に公開します。クラスター外部にあるロードバランサーが転送す
  る先の`NodePort`と`ClusterIP` Service は自動的に作成されます。
- [`ExternalName`](#externalname): `CNAME`レコードを返すことにより
  、`externalName`フィールドに指定したコンテンツ(例: `foo.bar.example.com`)と
  Service を紐づけます。しかし、いかなる種類のプロキシーも設定されません。

  {{< note >}} `ExternalName`タイプの Service を利用するためには、CoreDNS のバー
  ジョン 1.7 以上が必要となります。 {{< /note >}}

また、Service を公開するため
に[Ingress](/docs/concepts/services-networking/ingress/)も利用可能です。Ingress
は Service のタイプではありませんが、クラスターに対するエントリーポイントとして
動作します。  
Ingress は同一の IP アドレスにおいて、複数の Service を公開するように、ユーザー
の設定した転送ルールを 1 つのリソースにまとめることができます。

### NodePort タイプ {#nodeport}

もし`type`フィールドの値を`NodePort`に設定すると、Kubernetes コントロールプレー
ンは`--service-node-port-range`フラグによって指定されたレンジのポート(デフォルト
: 30000-32767)を割り当てます。  
各 Node はそのポート(各 Node で同じポート番号)への通信を Service に転送します
。  
作成した Service は、`.spec.ports[*].nodePort`フィールド内に割り当てられたポート
を記述します。

もしポートへの通信を転送する特定の IP を指定したい場合、特定の IP ブロックを
kube-proxy の`--nodeport-address`フラグで指定できます。これは Kubernetesv1.10 か
らサポートされています。  
このフラグは、コンマ区切りの IP ブロックのリスト(例: 10.0.0./8, 192.0.2.0/25)を
使用し、kube-proxy がこの Node に対してローカルとみなすべき IP アドレスの範囲を
指定します。

例えば、`--nodeport-addresses=127.0.0.0/8`というフラグによって kube-proxy を起動
した時、kube-proxy は NodePort Service のためにループバックインターフェースのみ
選択します。`--nodeport-addresses`のデフォルト値は空のリストになります。これは
kube-proxy が NodePort Service に対して全てのネットワークインターフェースを利用
可能とするべきということを意味します(これは以前の Kubernetes のバージョンとの互
換性があります)。

もしポート番号を指定したい場合、`nodePort`フィールドに値を指定できます。コントロ
ールプレーンは指定したポートを割り当てるか、API トランザクションが失敗したことを
知らせるかのどちらかになります。  
これは、ユーザーが自分自身で、ポート番号の衝突に関して気をつける必要があることを
意味します。  
また、ユーザーは有効なポート番号を指定する必要があり、NodePort の使用において、
設定された範囲のポートを指定する必要があります。

NodePort の使用は、Kubernetes によって完全にサポートされていないようなユーザー独
自の負荷分散を設定をするための有効な方法や、1 つ以上の Node の IP を直接公開する
ための方法となりえます。

注意点として、この Service は`<NodeIP>:spec.ports[*].nodePort`と
、`.spec.clusterIP:spec.ports[*].port`として疎通可能です。  
(もし kube-proxy において`--nodeport-addressses`が設定された場合、<NodeIP>はフィ
ルターされた NodeIP となります。)

### LoadBalancer タイプ {#loadbalancer}

外部のロードバランサーをサポートするクラウドプロバイダー上で、`type`フィールド
に`LoadBalancer`を設定すると、Service 用にロードバランサーがプロビジョニングされ
ます。  
実際のロードバランサーの作成は非同期で行われ、プロビジョンされたバランサーの情報
は、Service の`.status.loadBalancer`フィールドに記述されます。  
例えば:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
  clusterIP: 10.0.171.239
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
      - ip: 192.0.2.127
```

外部のロードバランサーからのトラフィックはバックエンドの Pod に直接転送されます
。クラウドプロバイダーはどのようにそのリクエストをバランシングするかを決めます。

いくつかのクラウドプロバイダーにおいて、`loadBalancerIP`の設定をすることができま
す。このようなケースでは、そのロードバランサーはユーザーが指定し
た`loadBalancerIP`に対してロードバランサーを作成します。  
もし`loadBalancerIP`フィールドの値が指定されていない場合、そのロードバランサーは
エフェメラルな IP アドレスに対して作成されます。もしユーザーが`loadBalancerIP`を
指定したが、使っているクラウドプロバイダーがその機能をサポートしていない場合、そ
の`loadBalancerIP`フィールドに設定された値は無視されます。

{{< note >}} もし SCTP を使っている場合、`LoadBalancer` タイプの Service に関す
る[使用上の警告](#caveat-sctp-loadbalancer-service-type)を参照してください。
{{< /note >}}

{{< note >}}

**Azure** において、もしユーザーが指定する`loadBalancerIP`を使用したい場合、最初
に静的なパブリック IP アドレスのリソースを作成する必要があります。  
このパブリック IP アドレスのリソースは、クラスター内で自動的に作成された他のリソ
ースと同じグループに作られるべきです。  
例: `MC_myResourceGroup_myAKSCluster_eastus`

割り当てられた IP アドレスを loadBalancerIP として指定してください。クラウドプロ
バイダーの設定ファイルにおいて securityGroupName を更新したことを確認してくださ
い。  
`CreatingLoadBalancerFailed`というパーミッションの問題に対するトラブルシューティ
ングの情報は
、[Azure Kubernetes Service(AKS)のロードバランサーで静的 IP アドレスを使用する](https://docs.microsoft.com/en-us/azure/aks/static-ip)
や
、[高度なネットワークを使用した AKS クラスターでの CreatingLoadBalancerFailed](https://github.com/Azure/AKS/issues/357)を
参照してください。 {{< /note >}}

#### 内部のロードバランサー

複雑な環境において、同一の(仮想)ネットワークアドレスブロック内の Service からの
トラフィックを転送する必要がでてきます。

Split-Horizon な DNS 環境において、ユーザーは 2 つの Service を外部と内部の両方
からのトラフィックをエンドポイントに転送させる必要がでてきます。

ユーザーは、Service に対して下記のアノテーションを 1 つ追加することでこれを実現
できます。  
追加するアノテーションは、ユーザーが使っているクラウドプロバイダーに依存していま
す。

{{< tabs name="service_tabs" >}} {{% tab name="Default" %}} タブを選択してくださ
い。  
{{% /tab %}} {{% tab name="GCP" %}}

```yaml
[...]
metadata:
    name: my-service
    annotations:
        cloud.google.com/load-balancer-type: "Internal"
[...]
```

{{% /tab %}} {{% tab name="AWS" %}}

```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/aws-load-balancer-internal: 0.0.0.0/0
[...]
```

{{% /tab %}} {{% tab name="Azure" %}}

```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/azure-load-balancer-internal: "true"
[...]
```

{{% /tab %}} {{% tab name="OpenStack" %}}

```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/openstack-internal-load-balancer: "true"
[...]
```

{{% /tab %}} {{% tab name="Baidu Cloud" %}}

```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/cce-load-balancer-internal-vpc: "true"
[...]
```

{{% /tab %}} {{% tab name="Tencent Cloud" %}}

```yaml
[...]
metadata:
  annotations:
    service.kubernetes.io/qcloud-loadbalancer-internal-subnetid: subnet-xxxxx
[...]
```

{{% /tab %}} {{< /tabs >}}

#### AWS における TLS のサポート {#ssl-support-on-aws}

AWS 上で稼働しているクラスターにおいて、部分的な TLS/SSL のサポートをするには
、`LoadBalancer` Service に対して 3 つのアノテーションを追加できます。

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: arn:aws:acm:us-east-1:123456789012:certificate/12345678-1234-1234-1234-123456789012
```

1 つ目は、使用する証明書の ARN です。これは IAM にアップロードされたサードパーテ
ィーが発行した証明書か、AWS Certificate Manager で作成された証明書になります。

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: (https|http|ssl|tcp)
```

2 つ目のアノテーションは Pod が利用するプロトコルを指定するものです。HTTPS と
SSL の場合、ELB はその Pod が証明書を使って暗号化されたコネクションを介して自分
自身の Pod を認証すると推測します。

HTTP と HTTPS では、レイヤー 7 でのプロキシーを選択します。ELB はユーザーとのコ
ネクションを切断し、リクエストを転送するときにリクエストヘッダーをパースして
、`X-Forwardef-For`ヘッダーにユーザーの IP を追加します(Pod は接続相手の ELB の
IP アドレスのみ確認可能です)。

TCP と SSL では、レイヤー 4 でのプロキシーを選択します。ELB はヘッダーの値を変更
せずにトラフィックを転送します。

いくつかのポートがセキュアに保護され、他のポートではセキュアでないような混合した
環境において、下記のようにアノテーションを使うことができます。

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: http
    service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "443,8443"
```

上記の例では、もし Service が`80`、`443`、`8443`と 3 つのポートを含んでいる場合
、`443`と`8443`は SSL 証明書を使いますが、`80`では単純に HTTP でのプロキシーとな
ります。

Kubernetes v1.9 以降のバージョンからは、Service のリスナー用に HTTPS や SSL
と[事前定義された AWS SSL ポリシー](http://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-security-policy-table.html)を
使用できます。どのポリシーが使用できるかを確認するために、`aws`コマンドラインツ
ールを使用できます。

```bash
aws elb describe-load-balancer-policies --query 'PolicyDescriptions[].PolicyName'
```

ユーザーは
"`service.beta.kubernetes.io/aws-load-balancer-ssl-negotiation-policy`"というア
ノテーションを使用することにより、複数のポリシーの中からどれか 1 つを指定できま
す。  
例えば:

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-ssl-negotiation-policy: "ELBSecurityPolicy-TLS-1-2-2017-01"
```

#### AWS 上での PROXY プロトコルのサポート

AWS 上で稼働するクラスター
で[PROXY protocol](https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt)の
サポートを有効にするために、下記の Service のアノテーションを使用できます。

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-proxy-protocol: "*"
```

Kubernetes バージョン 1.3.0 からは、このアノテーションを使用すると ELB によって
プロキシーされた全てのポートが対象になり、そしてそれ以外の場合は構成されません。

#### AWS 上での ELB のアクセスログ

AWS 上での ELB Service 用のアクセスログを管理するためにはいくつかのアノテーショ
ンが使用できます。

`service.beta.kubernetes.io/aws-load-balancer-access-log-enabled`というアノテー
ションはアクセスログを有効にするかを設定できます。

`service.beta.kubernetes.io/aws-load-balancer-access-log-emit-interval`というア
ノテーションはアクセスログをパブリッシュするためのインターバル(分)を設定できます
。  
ユーザーはそのインターバルで 5 分もしくは 60 分で設定できます。

`service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-name`というア
ノテーションはロードバランサーのアクセスログが保存される Amazon S3 のバケット名
を設定できます。

`service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-prefix`という
アノテーションはユーザーが作成した Amazon S3 バケットの論理的な階層を指定します
。

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-access-log-enabled: "true"
    # ロードバランサーのアクセスログが有効かどうか。
    service.beta.kubernetes.io/aws-load-balancer-access-log-emit-interval: "60"
    # アクセスログをパブリッシュするためのインターバル(分)。ユーザーはそのインターバルで5分もしくは60分で設定できます。
    service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-name: "my-bucket"
    # ロードバランサーのアクセスログが保存されるAmazon S3のバケット名。
    service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-prefix: "my-bucket-prefix/prod"
    # ユーザーが作成したAmazon S3バケットの論理的な階層。例えば: `my-bucket-prefix/prod`
```

#### AWS での接続の中断

古いタイプの ELB での接続の中断は
、`service.beta.kubernetes.io/aws-load-balancer-connection-draining-enabled`とい
うアノテーションを`"true"`に設定することで管理できます。  
`service.beta.kubernetes.io/aws-load-balancer-connection-draining-timeout`という
アノテーションで、インスタンスを登録解除するまえに既存の接続をオープンにし続ける
ための最大時間(秒)を指定できます。

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-connection-draining-enabled: "true"
    service.beta.kubernetes.io/aws-load-balancer-connection-draining-timeout: "60"
```

#### 他の ELB アノテーション

古いタイプの ELB を管理するためのアノテーションは他にもあり、下記で紹介します。

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: "60"
    # ロードバランサーによってクローズされる前にアイドル状態(コネクションでデータは送信されない)になれる秒数

    service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
    # ゾーンを跨いだロードバランシングが有効かどうか

    service.beta.kubernetes.io/aws-load-balancer-additional-resource-tags: "environment=prod,owner=devops"
    # ELBにおいて追加タグとして保存されるキー・バリューのペアのコンマ区切りのリスト

    service.beta.kubernetes.io/aws-load-balancer-healthcheck-healthy-threshold: ""
    # バックエンドへのトラフィックが正常になったと判断するために必要なヘルスチェックの連続成功数
    # デフォルトでは2 この値は2から10の間で設定可能

    service.beta.kubernetes.io/aws-load-balancer-healthcheck-unhealthy-threshold: "3"
    # バックエンドへのトラフィックが異常になったと判断するために必要なヘルスチェックの連続失敗数
    # デフォルトでは6 この値は2から10の間で設定可能

    service.beta.kubernetes.io/aws-load-balancer-healthcheck-interval: "20"
    # 各インスタンスのヘルスチェックのおよそのインターバル(秒)
    # デフォルトでは10 この値は5から300の間で設定可能

    service.beta.kubernetes.io/aws-load-balancer-healthcheck-timeout: "5"
    # ヘルスチェックが失敗したと判断されるレスポンスタイムのリミット(秒)
    # この値はservice.beta.kubernetes.io/aws-load-balancer-healthcheck-intervalの値以下である必要があります。
    # デフォルトでは5 この値は2から60の間で設定可能

    service.beta.kubernetes.io/aws-load-balancer-extra-security-groups: "sg-53fae93f,sg-42efd82e"
    # ELBに追加される予定のセキュリティーグループのリスト
```

#### AWS での Network Load Balancer のサポート {#aws-nlb-support}

{{< feature-state for_k8s_version="v1.15" state="beta" >}}

AWS で Network Load Balancer を使用するには、値を`nlb`に設定してアノテーショ
ン`service.beta.kubernetes.io/aws-load-balancer-type`を付与します。

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
```

{{< note >}} NLB は特定のインスタンスクラスでのみ稼働します。サポートされている
インスタンスタイプを確認するためには、ELB に関す
る[AWS documentation](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/target-group-register-targets.html#register-deregister-targets)を
参照してください。  
{{< /note >}}

古いタイプの Elastic Load Balancers とは異なり、Network Load Balancers (NLBs)は
クライアントの IP アドレスを Node に転送します。  
もし Service の`.spec.externalTrafficPolicy`の値が`Cluster`に設定されていた場合
、クライアントの IP アドレスは末端の Pod に伝播しません。

`.spec.externalTrafficPolicy`を`Local`に設定することにより、クライアント IP アド
レスは末端の Pod に伝播します。しかし、これにより、トラフィックの分配が不均等に
なります。  
特定の LoadBalancer Service に紐づいた Pod がない Node では、自動的に割り当てら
れた`.spec.healthCheckNodePort`に対する NLB のターゲットグループのヘルスチェック
が失敗し、トラフィックを全く受信しません。

均等なトラフィックの分配を実現するために、DaemonSet の使用や、同一の Node に配備
しないよう
に[Pod の anti-affinity](/ja/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity)を
設定します。

また
、[内部のロードバランサー](/ja/docs/concepts/services-networking/service/#internal-load-balancer)の
アノテーションと NLB Service を使用できます。

NLB の背後にあるインスタンスに対してクライアントのトラフィックを転送するために
、Node のセキュリティーグループは下記のような IP ルールに従って変更されます。

| Rule                             | Protocol | Port(s)                                                                             | IpRange(s)                                                 | IpRange Description                                |
| -------------------------------- | -------- | ----------------------------------------------------------------------------------- | ---------------------------------------------------------- | -------------------------------------------------- |
| ヘルスチェック                   | TCP      | NodePort(s) (`.spec.healthCheckNodePort` for `.spec.externalTrafficPolicy = Local`) | VPC CIDR                                                   | kubernetes.io/rule/nlb/health=\<loadBalancerName\> |
| クライアントのトラフィック       | TCP      | NodePort(s)                                                                         | `.spec.loadBalancerSourceRanges` (デフォルト: `0.0.0.0/0`) | kubernetes.io/rule/nlb/client=\<loadBalancerName\> |
| MTC によるサービスディスカバリー | ICMP     | 3,4                                                                                 | `.spec.loadBalancerSourceRanges` (デフォルト: `0.0.0.0/0`) | kubernetes.io/rule/nlb/mtu=\<loadBalancerName\>    |

どのクライアント IP が NLB にアクセス可能かを制限するためには
、`loadBalancerSourceRanges`を指定してください。

```yaml
spec:
  loadBalancerSourceRanges:
    - "143.231.0.0/16"
```

{{< note >}} もし`.spec.loadBalancerSourceRanges`が設定されていない場合
、Kubernetes は Node のセキュリティーグループに対して`0.0.0.0/0`からのトラフィッ
クを許可します。  
もし Node がパブリックな IP アドレスを持っていた場合、NLB でないトラフィックも修
正されたセキュリティーグループ内の全てのインスタンスにアクセス可能になってしまう
ので注意が必要です。

{{< /note >}}

#### Tencent Kubernetes Engine（TKE）におけるその他の CLB アノテーション

以下に示すように、TKE で Cloud Load Balancer を管理するためのその他のアノテーシ
ョンがあります。

```yaml
    metadata:
      name: my-service
      annotations:
        # 指定したノードでロードバランサーをバインドします
        service.kubernetes.io/qcloud-loadbalancer-backends-label: key in (value1, value2)
        # 既存のロードバランサーのID
        service.kubernetes.io/tke-existed-lbid：lb-6swtxxxx

        # ロードバランサー(LB)のカスタムパラメーターは、LBタイプの変更をまだサポートしていません
        service.kubernetes.io/service.extensiveParameters: ""

        # LBリスナーのカスタムパラメーター
        service.kubernetes.io/service.listenerParameters: ""

        # ロードバランサーのタイプを指定します
        # 有効な値: classic(Classic Cloud Load Balancer)またはapplication(Application Cloud Load Balancer)
        service.kubernetes.io/loadbalance-type: xxxxx
        # パブリックネットワーク帯域幅の課金方法を指定します
        # 有効な値: TRAFFIC_POSTPAID_BY_HOUR(bill-by-traffic)およびBANDWIDTH_POSTPAID_BY_HOUR(bill-by-bandwidth)
        service.kubernetes.io/qcloud-loadbalancer-internet-charge-type: xxxxxx
        # 帯域幅の値を指定します(値の範囲：[1-2000] Mbps)。
        service.kubernetes.io/qcloud-loadbalancer-internet-max-bandwidth-out: "10"
        # この注釈が設定されている場合、ロードバランサーはポッドが実行されているノードのみを登録します
        # そうでない場合、すべてのノードが登録されます
        service.kubernetes.io/local-svc-only-bind-node-with-pod: true
```

### ExternalName タイプ {#externalname}

ExternalName タイプの Service は、Service を DNS 名とマッピングし
、`my-service`や`cassandra`というような従来のラベルセレクターとはマッピングしま
せん。  
ユーザーはこれらの Service において`spec.externalName`フィールドの値を指定します
。

この Service の定義では、例えば`prod`という Namespace 内の`my-service`という
Service を`my.database.example.com`にマッピングします。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```

{{< note >}} ExternalName は Ipv4 のアドレスの文字列のみ受け付けますが、IP アド
レスではなく、数字で構成される DNS 名として受け入れます。  
IPv4 アドレスに似ている ExternalNames は CoreDNS もしくは Ingress-Nginx によって
名前解決されず、これは ExternalName は正規の DNS 名を指定することを目的としてい
るためです。  
IP アドレスをハードコードする場合、[Headless Service](#headless-service)の使用を
検討してください。 {{< /note >}}

`my-service.prod.svc.cluster.local`というホストをルックアップするとき、クラスタ
ーの DNS Service は`CNAME`レコードと`my.database.example.com`という値を返します
。  
`my-service`へのアクセスは、他の Service と同じ方法ですが、再接続する際はプロキ
シーや転送を介して行うよりも、DNS レベルで行われることが決定的に異なる点となりま
す。  
後にユーザーが使用しているデータベースをクラスター内に移行することになった後は
、Pod を起動させ、適切なラベルセレクターや Endpoints を追加し、Service
の`type`を変更します。

{{< warning >}} HTTP や HTTPS などの一般的なプロトコルで ExternalName を使用する
際に問題が発生する場合があります。ExternalName を使用する場合、クラスター内のク
ライアントが使用するホスト名は、ExternalName が参照する名前とは異なります。

ホスト名を使用するプロトコルの場合、この違いによりエラーまたは予期しない応答が発
生する場合があります。HTTP リクエストには、オリジンサーバーが認識しない`Host:`ヘ
ッダーがあります。TLS サーバーは、クライアントが接続したホスト名に一致する証明書
を提供できません。 {{< /warning >}}

{{< note >}} このセクションは、[Alen Komljen](https://akomljen.com/)によ
る[Kubernetes Tips - Part1](https://akomljen.com/kubernetes-tips-part-1/)という
ブログポストを参考にしています。

{{< /note >}}

### External IPs

もし 1 つ以上のクラスター Node に転送する externalIP が複数ある場合、Kubernetes
Service は`externalIPs`に指定した IP で公開されます。  
その externalIP(到達先の IP として扱われます)の Service のポートからトラフィック
がクラスターに入って来る場合、Service の Endpoints のどれか 1 つに対して転送され
ます。  
`externalIPs`は Kubernetes によって管理されず、それを管理する責任はクラスターの
管理者にあります。

Service の spec において、`externalIPs`は他のどの`ServiceTypes`と併用して設定で
きます。  
下記の例では、"`my-service`"は"`80.11.12.10:80`" (`externalIP:port`)のクライアン
トからアクセス可能です。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
  externalIPs:
    - 80.11.12.10
```

## Service のデメリット

仮想 IP 用に userspace モードのプロキシーを使用すると、小規模もしくは中規模のス
ケールでうまく稼働できますが、1000 以上の Service があるようなとても大きなクラス
ターではうまくスケールしません。  
これについては、[Service のデザインプロポーザル](http://issue.k8s.io/1107)にてさ
らなる詳細を確認できます。

userspace モードのプロキシーの使用は、Service にアクセスするパケットの送信元 IP
アドレスが不明瞭になります。  
これは、いくつかの種類のネットワークフィルタリング(ファイアウォールによるフィル
タリング)を不可能にします。  
iptables プロキシーモードはクラスター内の送信元 IP を不明瞭にはしませんが、依然
としてロードバランサーや NodePort へ疎通するクライアントに影響があります。

`Type`フィールドはネストされた機能としてデザインされています。 - 各レベルの値は
前のレベルに対して追加します。  
これは全てのクラウドプロバイダーにおいて厳密に要求されていません(例: Google
Compute Engine は`LoadBalancer`を動作させるために`NodePort`を割り当てる必要はあ
りませんが、AWS ではその必要があります)が、現在の API では要求しています。

## 仮想 IP の実装について {#the-gory-details-of-virtual-ips}

これより前の情報は、ただ Service を使いたいという多くのユーザーにとっては有益か
もしれません。しかし、その裏側では多くのことが行われており、理解する価値がありま
す。

### 衝突の回避

Kubernetes の主要な哲学のうちの一つは、ユーザーは、ユーザー自身のアクションによ
るミスでないものによって、ユーザーのアクションが失敗するような状況に晒されるべき
でないことです。  
Service リソースの設計のでは、これはユーザーの指定したポートが衝突する可能性があ
る場合は、そのポートの Service を作らないことを意味します。これは障害を分離する
こととなります。

Service のポート番号を選択できるようにするために、我々はどの 2 つの Service でも
ポートが衝突しないことを保証します。  
Kubernetes は各 Service に、それ自身の IP アドレスを割り当てることで実現していま
す。

各 Service が固有の IP を割り当てられるのを保証するために、内部のアロケーターは
、Service を作成する前に、etcd 内のグローバルの割り当てマップをアトミックに更新
します。  
そのマップオブジェクトは Service の IP アドレスの割り当てのためにレジストリー内
に存在しなくてはならず、そうでない場合は、Service の作成時に IP アドレスが割り当
てられなかったことを示すエラーメッセージが表示されます。

コントロールプレーンにおいて、バックグラウンドのコントローラーはそのマップを作成
する責務があります(インメモリーのロックが使われていた古いバージョンの Kubernetes
のマイグレーションも必要です)。また、Kubernetes は無効な割り当てがされているかを
チェックすることと、現時点でどの Service にも使用されていない割り当て済み IP ア
ドレスのクリーンアップのためにコントローラーを使用します。

### Service の IP アドレス {#ips-and-vips}

実際に固定された向き先である Pod の IP アドレスとは異なり、Service の IP は実際
には単一のホストによって応答されません。その代わり、kube-proxy は必要な時に透過
的にリダイレクトされる*仮想* IP アドレスを定義するため、iptables(Linux のパケッ
ト処理ロジック)を使用します。  
クライアントが VIP に接続する時、そのトラフィックは自動的に適切な Endpoints に転
送されます。 Service 用の環境変数と DNS は、Service の仮想 IP アドレス(とポート)
の面において、自動的に生成されます。

kube-proxy は 3 つの微妙に異なった動作をするプロキシーモード&mdash;
userspace、iptables と IPVS &mdash; をサポートしています。

#### Userspace

例として、上記で記述されている画像処理のアプリケーションを考えます。  
バックエンドの Service が作成されたとき、Kubernetes の Master は仮想 IP を割り当
てます。例えば 10.0.0.1 などです。  
その Service のポートが 1234 で、その Service はクラスター内の全ての kube-proxy
インスタンスによって監視されていると仮定します。  
kube-proxy が新しい Service を見つけた時、kube-proxy は新しいランダムポートをオ
ープンし、その仮想 IP アドレスの新しいポートにリダイレクトするように iptables を
更新し、そのポート上で新しい接続を待ち受けを開始します。

クライアントが Service の仮想 IP アドレスに接続したとき、iptables ルールが有効に
なり、そのパケットをプロキシー自身のポートにリダイレクトします。  
その"Service プロキシー"はバックエンド Pod の対象を選択し、クライアントのトラフ
ィックをバックエンド Pod に転送します。

これは Service のオーナーは、衝突のリスクなしに、求めるどのようなポートも選択で
きることを意味します。クライアントは単純にその IP とポートに対して接続すればよく
、実際にどの Pod にアクセスしているかを意識しません。

#### iptables

また画像処理のアプリケーションについて考えます。バックエンド Service が作成され
た時、その Kubernetes コントロールプレーンは仮想 IP アドレスを割り当てます。例え
ば 10.0.0.1 などです。  
Service のポートが 1234 で、その Service がクラスター内のすべての kube-proxy イ
ンスタンスによって監視されていると仮定します。  
kube-proxy が新しい Service を見つけた時、kube-proxy は仮想 IP から各 Service の
ルールにリダイレクトされるような、iptables ルールのセットをインストールします。
Service 毎のルールは、トラフィックをバックエンドにリダイレクト(Destination NAT
を使用)している Endpoints 毎のルールに対してリンクしています。

クライアントが Service の仮想 IP アドレスに対して接続しているとき、その iptables
ルールが有効になります。  
バックエンドの Pod が選択され(SessionAffinity に基づくか、もしくはランダムで選択
される)、パケットはバックエンドにリダイレクトされます。  
userspace モードのプロキシーとは異なり、パケットは決して userspace にコピーされ
ず、kube-proxy は仮想 IP のために稼働される必要はなく、また Node では変更されて
いないクライアント IP からトラフィックがきます。

このように同じ基本的なフローは、NodePort または LoadBalancer を介してトラフィッ
クがきた場合に、実行され、ただクライアント IP は変更されます。

#### IPVS

iptables の処理は、大規模なクラスターの場合劇的に遅くなります。例としては
Service が 10,000 ほどある場合です。 IPVS は負荷分散のために設計され、カーネル内
のハッシュテーブルに基づいています。そのため IPVS ベースの kube-proxy によって、
多数の Service がある場合でも一貫して高パフォーマンスを実現できます。  
次第に、IPVS ベースの kube-proxy は負荷分散のアルゴリズムはさらに洗練されていま
す(最小接続数、位置ベース、重み付け、永続性など)。

## API オブジェクト

Service は Kubernetes の REST API においてトップレベルのリソースです。ユーザーは
その API オブジェクトに関して、[Service API
object](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#service-v1-core)
でさらなる情報を確認できます。

## サポートされているプロトコル {#protocol-support}

### TCP

{{< feature-state for_k8s_version="v1.0" state="stable" >}}

ユーザーはどの種類の Service においても TCP を利用できます。これはデフォルトのネ
ットワークプロトコルです。

### UDP

{{< feature-state for_k8s_version="v1.0" state="stable" >}}

ユーザーは多くの Service において UDP を利用できます。 type=LoadBalancer の
Service においては、UDP のサポートはこの機能を提供しているクラウドプロバイダーに
依存しています。

### HTTP

{{< feature-state for_k8s_version="v1.1" state="stable" >}}

もしクラウドプロバイダーがサポートしている場合、Service の Endpoints に転送され
る外部の HTTP/HTTPS でのリバースプロキシーをセットアップするために、LoadBalancer
モードで Service を作成可能です。

{{< note >}} ユーザーはまた、HTTP / HTTPS Service を公開するために、Service の代
わりに{{< glossary_tooltip term_id="ingress" >}}を利用することもできます。
{{< /note >}}

### PROXY プロトコル

{{< feature-state for_k8s_version="v1.1" state="stable" >}}

もしクラウドプロバイダーがサポートしている場合(例:
[AWS](/docs/concepts/cluster-administration/cloud-providers/#aws))、Kubernetes
クラスターの外部のロードバランサーを設定するために LoadBalancer モードで Service
を利用できます。これ
は[PROXY protocol](https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt)が
ついた接続を転送します。

ロードバランサーは、最初の一連のオクテットを送信します。下記のような例となります
。

```
PROXY TCP4 192.0.2.202 10.0.42.7 12345 7\r\n
```

クライアントからのデータのあとに追加されます。

### SCTP

{{< feature-state for_k8s_version="v1.12" state="alpha" >}}

Kubernetse は Service、Endpoints、NetworkPolicy と Pod の定義において α 版の機能
として`protocol`フィールドの値で SCTP をサポートしています。この機能を有効にする
ために、クラスター管理者は API Server において`SCTPSupport`という Feature Gate
を有効にする必要があります。例えば、`--feature-gates=SCTPSupport=true,…`といった
ように設定します。

その Feature Gate が有効になった時、ユーザーは Service、Endpoints、NetworkPolicy
の`protocol`フィールドと、Pod の`SCTP`フィールドを設定できます。  
Kubernetes は、TCP 接続と同様に、SCTP アソシエーションに応じてネットワークをセッ
トアップします。

#### 警告 {#caveat-sctp-overview}

##### マルチホーム SCTP アソシエーションのサポート {#caveat-sctp-multihomed}

{{< warning >}} マルチホーム SCTP アソシエーションのサポートは、複数のインターフ
ェースと Pod の IP アドレスの割り当てをサポートできる CNI プラグインを要求します
。

マルチホーム SCTP アソシエーションにおける NAT は、対応するカーネルモジュール内
で特別なロジックを要求します。 {{< /warning >}}

##### type=LoadBalancer Service について {#caveat-sctp-loadbalancer-service-type}

{{< warning >}} クラウドプロバイダーのロードバランサーの実装がプロトコルとして
SCTP をサポートしている場合は、`type` が LoadBalancer で`protocol`が SCTP の場合
でのみサービスを作成できます。  
そうでない場合、Service の作成要求はリジェクトされます。現時点でのクラウドのロー
ドバランサーのプロバイダー(Azure、AWS、CloudStack、GCE、OpenStack)は全て SCTP の
サポートをしていません。 {{< /warning >}}

##### Windows {#caveat-sctp-windows-os}

{{< warning >}} SCTP は Windows ベースの Node ではサポートされていません。
{{< /warning >}}

##### Userspace kube-proxy {#caveat-sctp-kube-proxy-userspace}

{{< warning >}} kube-proxy は userspace モードにおいて SCTP アソシエーションの管
理をサポートしません。 {{< /warning >}}

## Future work

将来的に、Service のプロキシーポリシーはシンプルなラウンドロビンのバランシングだ
けでなく、もっと細かな設定が可能になります。例えば、Master によって選択されるも
のや、水平シャーディングされたりするようになります。  
我々もまた、いくつかの Service が"実際の"ロードバランサーを備えることを想定しま
す。その場合、仮想 IP は単純にパケットをそのロードバランサーに転送します。

Kubernetes プロジェクトは、L7 (HTTP) Service へのサポートをもっと発展させようと
しています。

Kubernetes プロジェクトは、現在利用可能な ClusterIP、NodePort や LoadBalancer タ
イプの Service に対して、より柔軟な Ingress のモードを追加する予定です。

{{% /capture %}}

{{% capture whatsnext %}}

- [Connecting Applications with Services](/docs/concepts/services-networking/connect-applications-service/)を
  参照してください。
- [Ingress](/docs/concepts/services-networking/ingress/)を参照してください。
- [Endpoint Slices](/docs/concepts/services-networking/endpoint-slices/)を参照し
  てください。

{{% /capture %}}
