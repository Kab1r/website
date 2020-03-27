---
title: コンテナライフサイクルフック
content_template: templates/concept
weight: 30
---

{{% capture overview %}}

このページでは、kubelet により管理されるコンテナがコンテナライフサイクルフックフ
レームワークを使用して、管理ライフサイクル中にイベントによって引き起こされたコー
ドを実行する方法について説明します。

{{% /capture %}}

{{% capture body %}}

## 概要

Angular などのコンポーネントライフサイクルフックを持つ多くのプログラミング言語フ
レームワークと同様に、Kubernetes はコンテナにライフサイクルフックを提供します。
フックにより、コンテナは管理ライフサイクル内のイベントを認識し、対応するライフサ
イクルフックが実行されたときにハンドラーに実装されたコードを実行できます。

## コンテナフック

コンテナに公開されている 2 つのフックがあります。

`PostStart`

このフックはコンテナが作成された直後に実行されます。しかし、フックがコンテナの
ENTRYPOINT の前に実行されるという保証はありません。ハンドラーにパラメーターは渡
されません。

`PreStop`

このフックは、liveness probe の失敗、プリエンプション、リソース競合などの API 要
求または管理イベントが原因でコンテナが終了する直前に呼び出されます。コンテナが既
に終了状態または完了状態にある場合、preStop フックの呼び出しは失敗します。これは
ブロッキング、つまり同期的であるため、コンテナを削除するための呼び出しを送信する
前に完了する必要があります。ハンドラーにパラメーターは渡されません。

終了動作の詳細な説明は
、[Termination of Pods](/ja/docs/concepts/workloads/pods/pod/#podの終了)にありま
す。

### フックハンドラーの実装

コンテナは、フックのハンドラーを実装して登録することでそのフックにアクセスできま
す。コンテナに実装できるフックハンドラーは 2 種類あります。

- Exec - コンテナの cgroups と名前空間の中で、 `pre-stop.sh`のような特定のコマン
  ドを実行します。コマンドによって消費されたリソースはコンテナに対してカウントさ
  れます。
- HTTP - コンテナ上の特定のエンドポイントに対して HTTP 要求を実行します。

### フックハンドラーの実行

コンテナライフサイクル管理フックが呼び出されると、Kubernetes 管理システムはその
フック用に登録されたコンテナ内のハンドラーを実行します。

フックハンドラーの呼び出しは、コンテナを含む Pod のコンテキスト内で同期していま
す。これは、`PostStart`フックの場合、コンテナの ENTRYPOINT とフックは非同期に起
動することを意味します。しかし、フックの実行に時間がかかりすぎたりハングしたりす
ると、コンテナは`running`状態になることができません。

その振る舞いは `PreStop`フックに似ています。実行中にフックがハングした場合、Pod
フェーズは`Terminating`状態に留まり、Pod の`terminationGracePeriodSeconds`が終了
した後に終了します。 `PostStart`または`PreStop`フックが失敗した場合、コンテナを
強制終了します。

ユーザーはフックハンドラーをできるだけ軽量にするべきです。ただし、コンテナを停止
する前に状態を保存する場合など、長時間実行されるコマンドが意味をなす場合がありま
す。

### フック配送保証

フックの配送は _少なくとも 1 回_ を意図しています。これはフック
が`PostStart`や`PreStop`のような任意のイベントに対して複数回呼ばれることがあるこ
とを意味します。これを正しく処理するのはフックの実装次第です。

通常、単一の配送のみが行われます。たとえば、HTTP フックレシーバーがダウンしてい
てトラフィックを受け取れない場合、再送信は試みられません。ただし、まれに二重配送
が発生することがあります。たとえば、フックの送信中に kubelet が再起動した場合
、kubelet が起動した後にフックが再送信される可能性があります。

### フックハンドラーのデバッグ

フックハンドラーのログは、Pod のイベントには表示されません。ハンドラーが何らかの
理由で失敗した場合は、イベントをブロードキャストします。 `PostStart`の場合、これ
は`FailedPostStartHook`イベントで、`PreStop`の場合、これは`FailedPreStopHook`イ
ベントです。これらのイベントは `kubectl describe pod <pod_name>`を実行することで
見ることができます。このコマンドの実行によるイベントの出力例をいくつか示します。

```
Events:
  FirstSeen  LastSeen  Count  From                                                   SubobjectPath          Type      Reason               Message
  ---------  --------  -----  ----                                                   -------------          --------  ------               -------
  1m         1m        1      {default-scheduler }                                                          Normal    Scheduled            Successfully assigned test-1730497541-cq1d2 to gke-test-cluster-default-pool-a07e5d30-siqd
  1m         1m        1      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}  spec.containers{main}  Normal    Pulling              pulling image "test:1.0"
  1m         1m        1      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}  spec.containers{main}  Normal    Created              Created container with docker id 5c6a256a2567; Security:[seccomp=unconfined]
  1m         1m        1      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}  spec.containers{main}  Normal    Pulled               Successfully pulled image "test:1.0"
  1m         1m        1      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}  spec.containers{main}  Normal    Started              Started container with docker id 5c6a256a2567
  38s        38s       1      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}  spec.containers{main}  Normal    Killing              Killing container with docker id 5c6a256a2567: PostStart handler: Error executing in Docker Container: 1
  37s        37s       1      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}  spec.containers{main}  Normal    Killing              Killing container with docker id 8df9fdfd7054: PostStart handler: Error executing in Docker Container: 1
  38s        37s       2      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}                         Warning   FailedSync           Error syncing pod, skipping: failed to "StartContainer" for "main" with RunContainerError: "PostStart handler: Error executing in Docker Container: 1"
  1m         22s       2      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}  spec.containers{main}  Warning   FailedPostStartHook
```

{{% /capture %}}

{{% capture whatsnext %}}

- [コンテナ環境](/docs/concepts/containers/container-environment-variables/)の詳
  細
- [コンテナライフサイクルイベントへのハンドラー紐付け](/docs/tasks/configure-pod-container/attach-handler-lifecycle-event/)の
  ハンズオン

{{% /capture %}}
