---
title: サービスとアプリケーションの接続
content_template: templates/concept
weight: 30
---

{{% capture overview %}}

## コンテナを接続するための Kubernetes モデル

継続的に実行され、複製されたアプリケーションの準備ができたので、ネットワーク上で
公開することが可能になります。 Kubernetes のネットワークのアプローチについて説明
する前に、Docker の「通常の」ネットワーク手法と比較することが重要です。

デフォルトでは、Docker はホストプライベートネットワーキングを使用するため、コン
テナは同じマシン上にある場合にのみ他のコンテナと通信できます。 Docker コンテナが
ノード間で通信するには、マシンの IP アドレスにポートを割り当ててから、コンテナに
転送またはプロキシする必要があります。これは明らかに、コンテナが使用するポートを
非常に慎重に調整するか、ポートを動的に割り当てる必要があることを意味します。

複数の開発者間でポートを調整することは大規模に行うことは非常に難しく、ユーザーが
制御できないクラスターレベルの問題にさらされます。 Kubernetes では、どのホストで
稼働するかに関わらず、Pod が他の Pod と通信できると想定しています。すべての Pod
に独自のクラスタープライベート IP アドレスを付与するため、Pod 間のリンクを明示的
に作成したり、コンテナポートをホストポートにマップしたりする必要はありません。こ
れは、Pod 内のコンテナがすべて localhost の相互のポートに到達でき、クラスター内
のすべての Pod が NAT なしで相互に認識できることを意味します。このドキュメントの
残りの部分では、このようなネットワークモデルで信頼できるサービスを実行する方法に
ついて詳しく説明します。

このガイドでは、シンプルな nginx サーバーを使用して概念実証を示します。同じ原則
が、より完全
な[Jenkins CI アプリケーション](https://kubernetes.io/blog/2015/07/strong-simple-ssl-for-kubernetes)で
具体化されています。

{{% /capture %}}

{{% capture body %}}

## Pod をクラスターに公開する

前の例でネットワークモデルを紹介しましたが、再度ネットワークの観点に焦点を当てま
しょう。 nginx Pod を作成し、コンテナポートの仕様を指定していることに注意してく
ださい。

{{< codenew file="service/networking/run-my-nginx.yaml" >}}

これにより、クラスター内のどのノードからでもアクセスできるようになります。 Pod
が実行されているノードを確認します:

```shell
kubectl apply -f ./run-my-nginx.yaml
kubectl get pods -l run=my-nginx -o wide
```

```
NAME                        READY     STATUS    RESTARTS   AGE       IP            NODE
my-nginx-3800858182-jr4a2   1/1       Running   0          13s       10.244.3.4    kubernetes-minion-905m
my-nginx-3800858182-kna2y   1/1       Running   0          13s       10.244.2.5    kubernetes-minion-ljyd
```

Pod の IP を確認します:

```shell
kubectl get pods -l run=my-nginx -o yaml | grep podIP
    podIP: 10.244.3.4
    podIP: 10.244.2.5
```

クラスター内の任意のノードに SSH 接続し、両方の IP に curl 接続できるはずです。
コンテナはノードでポート 80 を使用**していない**ことに注意してください。また
、Pod にトラフィックをルーティングする特別な NAT ルールもありません。つまり、同
じ containerPort を使用して同じノードで複数の nginx Pod を実行し、IP を使用して
クラスター内の他の Pod やノードからそれらにアクセスできます。 Docker と同様に、
ポートは引き続きホストノードのインターフェイスに公開できますが、ネットワークモデ
ルにより、この必要性は根本的に減少します。

興味があれば、これ
を[どのように達成するか](/docs/concepts/cluster-administration/networking/#how-to-achieve-this)に
ついて詳しく読むことができます。

## Service を作成する

そのため、フラットでクラスター全体のアドレス空間で nginx を実行する Pod がありま
す。理論的には、これらの Pod と直接通信することができますが、ノードが停止すると
どうなりますか？ Pod はそれで死に、Deployment は異なる IP を持つ新しいものを作成
します。これは、Service が解決する問題です。

Kubernetes Service は、クラスター内のどこかで実行される Pod の論理セットを定義す
る抽象化であり、すべて同じ機能を提供します。作成されると、各 Service には一意の
IP アドレス(clusterIP とも呼ばれます)が割り当てられます。このアドレスは Service
の有効期間に関連付けられており、Service が動作している間は変更されません。 Pod
は、Service と通信するように構成でき、Service への通信は、Service のメンバーであ
る Pod に自動的に負荷分散されることを認識できます。

2 つの nginx レプリカのサービスを`kubectl exposed`で作成できます:

```shell
kubectl expose deployment/my-nginx
```

```
service/my-nginx exposed
```

これは次の yaml を`kubectl apply -f`することと同等です:

{{< codenew file="service/networking/nginx-svc.yaml" >}}

この仕様は、`run：my-nginx`ラベルを持つ任意の Pod の TCP ポート 80 をターゲット
とするサービスを作成し、抽象化されたサービスポートで Pod を公開します
(`targetPort`:はコンテナがトラフィックを受信するポート、`port`:は抽象化された
Service のポートであり、他の Pod が Service へのアクセスに使用する任意のポートに
することができます)。サービス定義でサポートされているフィールドのリスト
は[Service](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#service-v1-core)
API オブジェクトを参照してください。

Service を確認します:

```shell
kubectl get svc my-nginx
```

```
NAME       TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
my-nginx   ClusterIP   10.0.162.149   <none>        80/TCP    21s
```

前述のように、Service は Pod のグループによってサポートされています。これらの
Pod はエンドポイントを通じて公開されます。 Service のセレクターは継続的に評価さ
れ、結果は`my-nginx`という名前の Endpoints オブジェクトに POST されます。 Pod が
終了すると、エンドポイントから自動的に削除され、Service のセレクターに一致する新
しい Pod が自動的にエンドポイントに追加されます。エンドポイントを確認し、IP が最
初のステップで作成された Pod と同じであることを確認します:

```shell
kubectl describe svc my-nginx
```

```
Name:                my-nginx
Namespace:           default
Labels:              run=my-nginx
Annotations:         <none>
Selector:            run=my-nginx
Type:                ClusterIP
IP:                  10.0.162.149
Port:                <unset> 80/TCP
Endpoints:           10.244.2.5:80,10.244.3.4:80
Session Affinity:    None
Events:              <none>
```

```shell
kubectl get ep my-nginx
```

```
NAME       ENDPOINTS                     AGE
my-nginx   10.244.2.5:80,10.244.3.4:80   1m
```

クラスター内の任意のノードから、`<CLUSTER-IP>:<PORT>`で nginx Service に curl 接
続できるようになりました。 Service IP は完全に仮想的なもので、ホスト側のネットワ
ークには接続できないことに注意してください。この仕組みに興味がある場合は
、[サービスプロキシー](/ja/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies)の
詳細をお読みください。

## Service にアクセスする

Kubernetes は、環境変数と DNS の 2 つの主要な Service 検索モードをサポートしてい
ます。前者はそのまま使用でき、後者は[CoreDNS
クラスタアドオン](http://releases.k8s.io/{{< param
"githubbranch" >}}/cluster/addons/dns/coredns)を必要とします。 {{< note >}} サー
ビス環境変数が望ましくない場合(予想されるプログラム変数と衝突する可能性がある、
処理する変数が多すぎる、DNS のみを使用するなど)、[Pod
仕様](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#pod-v1-core)
で`enableServiceLinks`フラグを`false`に設定することでこのモードを無効にできます
。 {{< /note >}}

### 環境変数

ノードで Pod が実行されると、kubelet はアクティブな各サービスの環境変数のセット
を追加します。これにより、順序付けの問題が発生します。理由を確認するには、実行中
の nginx Pod の環境を調べます(Pod 名は環境によって異なります):

```shell
kubectl exec my-nginx-3800858182-jr4a2 -- printenv | grep SERVICE
```

```
KUBERNETES_SERVICE_HOST=10.0.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
```

サービスに言及がないことに注意してください。これは、サービスの前にレプリカを作成
したためです。これのもう 1 つの欠点は、スケジューラーが両方の Pod を同じマシンに
配置し、サービスが停止した場合にサービス全体がダウンする可能性があることです。 2
つの Pod を強制終了し、Deployment がそれらを再作成するのを待つことで、これを正し
い方法で実行できます。今回は、サービスはレプリカの「前」に存在します。これにより
、スケジューラーレベルのサービスが Pod に広がり(すべてのノードの容量が等しい場合
)、適切な環境変数が提供されます:

```shell
kubectl scale deployment my-nginx --replicas=0; kubectl scale deployment my-nginx --replicas=2;

kubectl get pods -l run=my-nginx -o wide
```

```
NAME                        READY     STATUS    RESTARTS   AGE     IP            NODE
my-nginx-3800858182-e9ihh   1/1       Running   0          5s      10.244.2.7    kubernetes-minion-ljyd
my-nginx-3800858182-j4rm4   1/1       Running   0          5s      10.244.3.8    kubernetes-minion-905m
```

Pod は強制終了されて再作成されるため、異なる名前が付いていることに気付くでしょう
。

```shell
kubectl exec my-nginx-3800858182-e9ihh -- printenv | grep SERVICE
```

```
KUBERNETES_SERVICE_PORT=443
MY_NGINX_SERVICE_HOST=10.0.162.149
KUBERNETES_SERVICE_HOST=10.0.0.1
MY_NGINX_SERVICE_PORT=80
KUBERNETES_SERVICE_PORT_HTTPS=443
```

### DNS

Kubernetes は、DNS 名を他の Service に自動的に割り当てる DNS クラスターアドオン
サービスを提供します。クラスターで実行されているかどうかを確認できます:

```shell
kubectl get services kube-dns --namespace=kube-system
```

```
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
kube-dns   ClusterIP   10.0.0.10    <none>        53/UDP,53/TCP   8m
```

実行されていない場合は、[有効にする](http://releases.k8s.io/{{< param
"githubbranch" >}}/cluster/addons/dns/kube-dns/README.md#how-do-i-configure-it)
ことができます。このセクションの残りの部分では、寿命の長い IP(my-nginx)を持つ
Service と、その IP に名前を割り当てた DNS サーバー(CoreDNS クラスターアドオン)
があることを前提としているため、標準的な方法(gethostbyname など)を使用してクラス
ター内の任意の Pod から Service に通信できます。 curl アプリケーションを実行して
、これをテストしてみましょう:

```shell
kubectl run curl --image=radial/busyboxplus:curl -i --tty
```

```
Waiting for pod default/curl-131556218-9fnch to be running, status is Pending, pod ready: false
Hit enter for command prompt
```

次に、Enter キーを押して`nslookup my-nginx`を実行します:

```shell
[ root@curl-131556218-9fnch:/ ]$ nslookup my-nginx
Server:    10.0.0.10
Address 1: 10.0.0.10

Name:      my-nginx
Address 1: 10.0.162.149
```

## Service を安全にする

これまでは、クラスター内から nginx サーバーにアクセスしただけでした。サービスを
インターネットに公開する前に、通信チャネルが安全であることを確認する必要がありま
す。これには、次のものが必要です:

- https 用の自己署名証明書(既に ID 証明書を持っている場合を除く)
- 証明書を使用するように構成された nginx サーバー
- Pod が証明書にアクセスできるようにす
  る[Secret](/docs/concepts/configuration/secret/)

これらはすべて[nginx https の例](https://github.com/kubernetes/examples/tree/{{<
param "githubbranch" >}}/staging/https-nginx/)から取得できます。これにはツールを
インストールする必要があります。これらをインストールしたくない場合は、後で手動の
手順に従ってください。つまり:

```shell
make keys KEY=/tmp/nginx.key CERT=/tmp/nginx.crt
kubectl create secret tls nginxsecret --key /tmp/nginx.key --cert /tmp/nginx.crt
```

```
secret/nginxsecret created
```

```shell
kubectl get secrets
```

```
NAME                  TYPE                                  DATA      AGE
default-token-il9rc   kubernetes.io/service-account-token   1         1d
nginxsecret           Opaque                                2         1m
```

以下は、(Windows 上など)make の実行で問題が発生した場合に実行する手動の手順です:

```shell
# 公開秘密鍵ペアを作成します
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /d/tmp/nginx.key -out /d/tmp/nginx.crt -subj "/CN=my-nginx/O=my-nginx"
# キーをbase64エンコードに変換します
cat /d/tmp/nginx.crt | base64
cat /d/tmp/nginx.key | base64
```

前のコマンドの出力を使用して、次のように yaml ファイルを作成します。 base64 でエ
ンコードされた値はすべて 1 行である必要があります。

```yaml
apiVersion: "v1"
kind: "Secret"
metadata:
  name: "nginxsecret"
  namespace: "default"
data:
  nginx.crt: "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURIekNDQWdlZ0F3SUJBZ0lKQUp5M3lQK0pzMlpJTUEwR0NTcUdTSWIzRFFFQkJRVUFNQ1l4RVRBUEJnTlYKQkFNVENHNW5hVzU0YzNaak1SRXdEd1lEVlFRS0V3aHVaMmx1ZUhOMll6QWVGdzB4TnpFd01qWXdOekEzTVRKYQpGdzB4T0RFd01qWXdOekEzTVRKYU1DWXhFVEFQQmdOVkJBTVRDRzVuYVc1NGMzWmpNUkV3RHdZRFZRUUtFd2h1CloybHVlSE4yWXpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBSjFxSU1SOVdWM0IKMlZIQlRMRmtobDRONXljMEJxYUhIQktMSnJMcy8vdzZhU3hRS29GbHlJSU94NGUrMlN5ajBFcndCLzlYTnBwbQppeW1CL3JkRldkOXg5UWhBQUxCZkVaTmNiV3NsTVFVcnhBZW50VWt1dk1vLzgvMHRpbGhjc3paenJEYVJ4NEo5Ci82UVRtVVI3a0ZTWUpOWTVQZkR3cGc3dlVvaDZmZ1Voam92VG42eHNVR0M2QURVODBpNXFlZWhNeVI1N2lmU2YKNHZpaXdIY3hnL3lZR1JBRS9mRTRqakxCdmdONjc2SU90S01rZXV3R0ljNDFhd05tNnNTSzRqYUNGeGpYSnZaZQp2by9kTlEybHhHWCtKT2l3SEhXbXNhdGp4WTRaNVk3R1ZoK0QrWnYvcW1mMFgvbVY0Rmo1NzV3ajFMWVBocWtsCmdhSXZYRyt4U1FVQ0F3RUFBYU5RTUU0d0hRWURWUjBPQkJZRUZPNG9OWkI3YXc1OUlsYkROMzhIYkduYnhFVjcKTUI4R0ExVWRJd1FZTUJhQUZPNG9OWkI3YXc1OUlsYkROMzhIYkduYnhFVjdNQXdHQTFVZEV3UUZNQU1CQWY4dwpEUVlKS29aSWh2Y05BUUVGQlFBRGdnRUJBRVhTMW9FU0lFaXdyMDhWcVA0K2NwTHI3TW5FMTducDBvMm14alFvCjRGb0RvRjdRZnZqeE04Tzd2TjB0clcxb2pGSW0vWDE4ZnZaL3k4ZzVaWG40Vm8zc3hKVmRBcStNZC9jTStzUGEKNmJjTkNUekZqeFpUV0UrKzE5NS9zb2dmOUZ3VDVDK3U2Q3B5N0M3MTZvUXRUakViV05VdEt4cXI0Nk1OZWNCMApwRFhWZmdWQTRadkR4NFo3S2RiZDY5eXM3OVFHYmg5ZW1PZ05NZFlsSUswSGt0ejF5WU4vbVpmK3FqTkJqbWZjCkNnMnlwbGQ0Wi8rUUNQZjl3SkoybFIrY2FnT0R4elBWcGxNSEcybzgvTHFDdnh6elZPUDUxeXdLZEtxaUMwSVEKQ0I5T2wwWW5scE9UNEh1b2hSUzBPOStlMm9KdFZsNUIyczRpbDlhZ3RTVXFxUlU9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K"
  nginx.key: "LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV2UUlCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQktjd2dnU2pBZ0VBQW9JQkFRQ2RhaURFZlZsZHdkbFIKd1V5eFpJWmVEZWNuTkFhbWh4d1NpeWF5N1AvOE9ta3NVQ3FCWmNpQ0RzZUh2dGtzbzlCSzhBZi9WemFhWm9zcApnZjYzUlZuZmNmVUlRQUN3WHhHVFhHMXJKVEVGSzhRSHA3VkpMcnpLUC9QOUxZcFlYTE0yYzZ3MmtjZUNmZitrCkU1bEVlNUJVbUNUV09UM3c4S1lPNzFLSWVuNEZJWTZMMDUrc2JGQmd1Z0ExUE5JdWFubm9UTWtlZTRuMG4rTDQKb3NCM01ZUDhtQmtRQlAzeE9JNHl3YjREZXUraURyU2pKSHJzQmlIT05Xc0RadXJFaXVJMmdoY1kxeWIyWHI2UAozVFVOcGNSbC9pVG9zQngxcHJHclk4V09HZVdPeGxZZmcvbWIvNnBuOUYvNWxlQlkrZStjSTlTMkQ0YXBKWUdpCkwxeHZzVWtGQWdNQkFBRUNnZ0VBZFhCK0xkbk8ySElOTGo5bWRsb25IUGlHWWVzZ294RGQwci9hQ1Zkank4dlEKTjIwL3FQWkUxek1yall6Ry9kVGhTMmMwc0QxaTBXSjdwR1lGb0xtdXlWTjltY0FXUTM5SjM0VHZaU2FFSWZWNgo5TE1jUHhNTmFsNjRLMFRVbUFQZytGam9QSFlhUUxLOERLOUtnNXNrSE5pOWNzMlY5ckd6VWlVZWtBL0RBUlBTClI3L2ZjUFBacDRuRWVBZmI3WTk1R1llb1p5V21SU3VKdlNyblBESGtUdW1vVlVWdkxMRHRzaG9reUxiTWVtN3oKMmJzVmpwSW1GTHJqbGtmQXlpNHg0WjJrV3YyMFRrdWtsZU1jaVlMbjk4QWxiRi9DSmRLM3QraTRoMTVlR2ZQegpoTnh3bk9QdlVTaDR2Q0o3c2Q5TmtEUGJvS2JneVVHOXBYamZhRGR2UVFLQmdRRFFLM01nUkhkQ1pKNVFqZWFKClFGdXF4cHdnNzhZTjQyL1NwenlUYmtGcVFoQWtyczJxWGx1MDZBRzhrZzIzQkswaHkzaE9zSGgxcXRVK3NHZVAKOWRERHBsUWV0ODZsY2FlR3hoc0V0L1R6cEdtNGFKSm5oNzVVaTVGZk9QTDhPTm1FZ3MxMVRhUldhNzZxelRyMgphRlpjQ2pWV1g0YnRSTHVwSkgrMjZnY0FhUUtCZ1FEQmxVSUUzTnNVOFBBZEYvL25sQVB5VWs1T3lDdWc3dmVyClUycXlrdXFzYnBkSi9hODViT1JhM05IVmpVM25uRGpHVHBWaE9JeXg5TEFrc2RwZEFjVmxvcG9HODhXYk9lMTAKMUdqbnkySmdDK3JVWUZiRGtpUGx1K09IYnRnOXFYcGJMSHBzUVpsMGhucDBYSFNYVm9CMUliQndnMGEyOFVadApCbFBtWmc2d1BRS0JnRHVIUVV2SDZHYTNDVUsxNFdmOFhIcFFnMU16M2VvWTBPQm5iSDRvZUZKZmcraEppSXlnCm9RN3hqWldVR3BIc3AyblRtcHErQWlSNzdyRVhsdlhtOElVU2FsbkNiRGlKY01Pc29RdFBZNS9NczJMRm5LQTQKaENmL0pWb2FtZm1nZEN0ZGtFMXNINE9MR2lJVHdEbTRpb0dWZGIwMllnbzFyb2htNUpLMUI3MkpBb0dBUW01UQpHNDhXOTVhL0w1eSt5dCsyZ3YvUHM2VnBvMjZlTzRNQ3lJazJVem9ZWE9IYnNkODJkaC8xT2sybGdHZlI2K3VuCnc1YytZUXRSTHlhQmd3MUtpbGhFZDBKTWU3cGpUSVpnQWJ0LzVPbnlDak9OVXN2aDJjS2lrQ1Z2dTZsZlBjNkQKckliT2ZIaHhxV0RZK2Q1TGN1YSt2NzJ0RkxhenJsSlBsRzlOZHhrQ2dZRUF5elIzT3UyMDNRVVV6bUlCRkwzZAp4Wm5XZ0JLSEo3TnNxcGFWb2RjL0d5aGVycjFDZzE2MmJaSjJDV2RsZkI0VEdtUjZZdmxTZEFOOFRwUWhFbUtKCnFBLzVzdHdxNWd0WGVLOVJmMWxXK29xNThRNTBxMmk1NVdUTThoSDZhTjlaMTltZ0FGdE5VdGNqQUx2dFYxdEYKWSs4WFJkSHJaRnBIWll2NWkwVW1VbGc9Ci0tLS0tRU5EIFBSSVZBVEUgS0VZLS0tLS0K"
```

ファイルを使用して Secret を作成します:

```shell
kubectl apply -f nginxsecrets.yaml
kubectl get secrets
```

```
NAME                  TYPE                                  DATA      AGE
default-token-il9rc   kubernetes.io/service-account-token   1         1d
nginxsecret           Opaque                                2         1m
```

次に、nginx レプリカを変更して、シークレットの証明書と Service を使用して https
サーバーを起動し、両方のポート(80 と 443)を公開します:

{{< codenew file="service/networking/nginx-secure-app.yaml" >}}

nginx-secure-app マニフェストに関する注目すべき点:

- 同じファイルに Deployment と Service の両方が含まれています。
- [nginx サーバー](https://github.com/kubernetes/examples/tree/{{< param
  "githubbranch" >}}/staging/https-nginx/default.conf)はポート 80 の HTTP トラフ
  ィックと 443 の HTTPS トラフィックを処理し、nginx Service は両方のポートを公開
  します。
- 各コンテナは`/etc/nginx/ssl`にマウントされたボリュームを介してキーにアクセスで
  きます。これは、nginx サーバーが起動する*前に*セットアップされます。

```shell
kubectl delete deployments,svc my-nginx; kubectl create -f ./nginx-secure-app.yaml
```

この時点で、任意のノードから nginx サーバーに到達できます。

```shell
kubectl get pods -o yaml | grep -i podip
    podIP: 10.244.3.5
node $ curl -k https://10.244.3.5
...
<h1>Welcome to nginx!</h1>
```

最後の手順で curl に`-k`パラメーターを指定したことに注意してください。これは、証
明書の生成時に nginx を実行している Pod について何も知らないためです。 CName の
不一致を無視するよう curl に指示する必要があります。 Service を作成することによ
り、証明書で使用される CName を、Service 検索中に Pod で使用される実際の DNS 名
にリンクしました。これを Pod からテストしましょう(簡単にするために同じシークレッ
トを再利用しています。Pod は Service にアクセスするために nginx.crt のみを必要と
します):

{{< codenew file="service/networking/curlpod.yaml" >}}

```shell
kubectl apply -f ./curlpod.yaml
kubectl get pods -l app=curlpod
```

```
NAME                               READY     STATUS    RESTARTS   AGE
curl-deployment-1515033274-1410r   1/1       Running   0          1m
```

```shell
kubectl exec curl-deployment-1515033274-1410r -- curl https://my-nginx --cacert /etc/nginx/ssl/nginx.crt
...
<title>Welcome to nginx!</title>
...
```

## Service を公開する

アプリケーションの一部では、Service を外部 IP アドレスに公開したい場合があります
。 Kubernetes は、NodePort と LoadBalancer の 2 つの方法をサポートしています。前
のセクションで作成した Service はすでに`NodePort`を使用しているため、ノードにパ
ブリック IP があれば、nginx HTTPS レプリカはインターネット上のトラフィックを処理
する準備ができています。

```shell
kubectl get svc my-nginx -o yaml | grep nodePort -C 5
  uid: 07191fb3-f61a-11e5-8ae5-42010af00002
spec:
  clusterIP: 10.0.162.149
  ports:
  - name: http
    nodePort: 31704
    port: 8080
    protocol: TCP
    targetPort: 80
  - name: https
    nodePort: 32453
    port: 443
    protocol: TCP
    targetPort: 443
  selector:
    run: my-nginx
```

```shell
kubectl get nodes -o yaml | grep ExternalIP -C 1
    - address: 104.197.41.11
      type: ExternalIP
    allocatable:
--
    - address: 23.251.152.56
      type: ExternalIP
    allocatable:
...

$ curl https://<EXTERNAL-IP>:<NODE-PORT> -k
...
<h1>Welcome to nginx!</h1>
```

クラウドロードバランサーを使用するようにサービスを再作成しましょう。
`my-nginx`サービスの`Type`を`NodePort`から`LoadBalancer`に変更するだけです:

```shell
kubectl edit svc my-nginx
kubectl get svc my-nginx
```

```
NAME       TYPE        CLUSTER-IP     EXTERNAL-IP        PORT(S)               AGE
my-nginx   ClusterIP   10.0.162.149   162.222.184.144    80/TCP,81/TCP,82/TCP  21s
```

```
curl https://<EXTERNAL-IP> -k
...
<title>Welcome to nginx!</title>
```

`EXTERNAL-IP`列の IP アドレスは、パブリックインターネットで利用可能なものです。
`CLUSTER-IP`は、クラスター/プライベートクラウドネットワーク内でのみ使用できます
。

AWS では、type `LoadBalancer`は IP ではなく(長い)ホスト名を使用する ELB が作成さ
れます。実際、標準の`kubectl get svc`の出力に収まるには長すぎるので、それを確認
するには`kubectl describe service my-nginx`を実行する必要があります。次のような
ものが表示されます:

```shell
kubectl describe service my-nginx
...
LoadBalancer Ingress:   a320587ffd19711e5a37606cf4a74574-1142138393.us-east-1.elb.amazonaws.com
...
```

{{% /capture %}}

{{% capture whatsnext %}}

Kubernetes は、複数のクラスターおよびクラウドプロバイダーにまたがるフェデレーシ
ョンサービスもサポートし、可用性の向上、フォールトトレランスの向上、サービスのス
ケーラビリティの向上を実現します。詳細について
は[フェデレーションサービスユーザーガイド](/docs/concepts/cluster-administration/federation-service-discovery/)を
参照してください。

{{% /capture %}}
