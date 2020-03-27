---
title: Pod内のコンテナ間でプロセス名前空間を共有する
min-kubernetes-server-version: v1.10
content_template: templates/task
weight: 160
---

{{% capture overview %}}

{{< feature-state state="stable" for_k8s_version="v1.17" >}}

このページでは、プロセス名前空間を共有する Pod を構成する方法を示します。プロセ
ス名前空間の共有が有効になっている場合、コンテナ内のプロセスは、その Pod 内の他
のすべてのコンテナに表示されます。

この機能を使用して、ログハンドラーサイドカーコンテナなどの協調コンテナを構成した
り、シェルなどのデバッグユーティリティを含まないコンテナイメージをトラブルシュー
ティングしたりできます。

{{% /capture %}}

{{% capture prerequisites %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

{{% /capture %}}

{{% capture steps %}}

## Pod を構成する

プロセス名前空間の共有は、`v1.PodSpec`の`shareProcessNamespace`フィールドを使用
して有効にします。例:

{{< codenew file="pods/share-process-namespace.yaml" >}}

1. クラスターに Pod `nginx`を作成します:

   ```shell
   kubectl apply -f https://k8s.io/examples/pods/share-process-namespace.yaml
   ```

1. `shell`コンテナにアタッチして`ps`を実行します:

   ```shell
   kubectl attach -it nginx -c shell
   ```

   コマンドプロンプトが表示されない場合は、Enter キーを押してみてください。

   ```
   / # ps ax
   PID   USER     TIME  COMMAND
       1 root      0:00 /pause
       8 root      0:00 nginx: master process nginx -g daemon off;
      14 101       0:00 nginx: worker process
      15 root      0:00 sh
      21 root      0:00 ps ax
   ```

他のコンテナのプロセスにシグナルを送ることができます。たとえば、ワーカープロセス
を再起動するには、`SIGHUP`を nginx に送信します。この操作には`SYS_PTRACE`機能が
必要です。

```
/ # kill -HUP 8
/ # ps ax
PID   USER     TIME  COMMAND
    1 root      0:00 /pause
    8 root      0:00 nginx: master process nginx -g daemon off;
   15 root      0:00 sh
   22 101       0:00 nginx: worker process
   23 root      0:00 ps ax
```

`/proc/$pid/root`リンクを使用して別のコンテナイメージにアクセスすることもできま
す。

```
/ # head /proc/8/root/etc/nginx/nginx.conf

user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
```

{{% /capture %}}

{{% capture discussion %}}

## プロセス名前空間の共有について理解する

Pod は多くのリソースを共有するため、プロセスの名前空間も共有することになります。
ただし、一部のコンテナイメージは他のコンテナから分離されることが期待されるため、
これらの違いを理解することが重要です:

1. **コンテナプロセスは PID 1 ではなくなります。** 一部のコンテナイメージは、PID
   1 なしで起動することを拒否し（たとえば、`systemd`を使用するコンテナ）
   、`kill -HUP 1`などのコマンドを実行してコンテナプロセスにシグナルを送信します
   。共有プロセス名前空間を持つ Pod では、`kill -HUP 1`は Pod サンドボックスにシ
   グナルを送ります。(上の例では`/pause`)

1. **プロセスは Pod 内の他のコンテナに表示されます。** これには、引数または環境
   変数として渡されたパスワードなど、`/proc`に表示されるすべての情報が含まれます
   。これらは、通常の Unix アクセス許可によってのみ保護されます。

1. **コンテナファイルシステムは、`/proc/$pid/root`リンクを介して Pod 内の他のコ
   ンテナに表示されます。** これによりデバッグが容易になりますが、ファイルシステ
   ム内の秘密情報はファイルシステムのアクセス許可によってのみ保護されることも意
   味します。

{{% /capture %}}
