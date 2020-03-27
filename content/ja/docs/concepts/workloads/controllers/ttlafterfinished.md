---
reviewers:
title:
  終了したリソースのためのTTLコントローラー(TTL Controller for Finished
  Resources)
content_template: templates/concept
weight: 65
---

{{% capture overview %}}

{{< feature-state for_k8s_version="v1.12" state="alpha" >}}

TTL コントローラーは実行を終えたリソースオブジェクトのライフタイムを制御するため
の TTL メカニズムを提供します。  
TTL コントローラーは現
在[Job](/docs/concepts/workloads/controllers/jobs-run-to-completion/)のみ扱って
いて、将来的に Pod やカスタムリソースなど、他のリソースの実行終了を扱えるように
拡張される予定です。

α 版の免責事項: この機能は現在 α 版の機能で
、[Feature Gate](/docs/reference/command-line-tools-reference/feature-gates/)の`TTLAfterFinished`を
有効にすることで使用可能です。

{{% /capture %}}

{{% capture body %}}

## TTL コントローラー

TTL コントローラーは現在 Job に対してのみサポートされています。クラスターオペレ
ーターはこ
の[例](/docs/concepts/workloads/controllers/jobs-run-to-completion/#clean-up-finished-jobs-automatically)の
ように、Job の`.spec.ttlSecondsAfterFinished`フィールドを指定することにより、終
了した Job(`完了した`もしくは`失敗した`)を自動的に削除するためにこの機能を使うこ
とができます。  
TTL コントローラーは、そのリソースが終了したあと指定した TTL の秒数後に削除でき
るか推定します。言い換えると、その TTL が期限切れになると、TTL コントローラーが
リソースをクリーンアップするときに、そのリソースに紐づく従属オブジェクトも一緒に
連続で削除します。注意点として、リソースが削除されるとき、ファイナライザーのよう
なライフサイクルに関する保証は尊重されます。

TTL 秒はいつでもセット可能です。下記は Job の`.spec.ttlSecondsAfterFinished`フィ
ールドのセットに関するいくつかの例です。

- Job がその終了後にいくつか時間がたった後に自動的にクリーンアップできるように、
  そのリソースマニフェストにこの値を指定します。
- この新しい機能を適用させるために、存在していて既に終了したリソースに対してこの
  フィールドをセットします。
- リソース作成時に、このフィールドを動的にセットするために
  、[管理 webhook の変更](/docs/reference/access-authn-authz/extensible-admission-controllers/#admission-webhooks)を
  させます。クラスター管理者は、終了したリソースに対して、この TTL ポリシーを強
  制するために使うことができます。
- リソースが終了した後に、このフィールドを動的にセットしたり、リソースステータス
  やラベルなどの値に基づいて異なる TTL 値を選択するために
  、[管理 webhook の変更](/docs/reference/access-authn-authz/extensible-admission-controllers/#admission-webhooks)を
  させます。

## 注意

### TTL 秒の更新

注意点として、Job の`.spec.ttlSecondsAfterFinished`フィールドといった TTL 期間は
リソースが作成された後、もしくは終了した後に変更できます。しかし、一度 Job が削
除可能(TTL の期限が切れたとき)になると、それがたとえ TTL を伸ばすような更新に対
して API のレスポンスで成功したと返されたとしても、そのシステムは Job が稼働し続
けることをもはや保証しません。

### タイムスキュー(Time Skew)

TTL コントローラーが、TTL 値が期限切れかそうでないかを決定するために Kubernetes
リソース内に保存されたタイムスタンプを使うため、この機能はクラスター内のタイムス
キュー(時刻のずれ)に対してセンシティブとなります。タイムスキューは、誤った時間に
TTL コントローラーに対してリソースオブジェクトのクリーンアップしてしまうことを引
き起こすものです。

Kubernetes においてタイムスキューを避けるために、全ての Node 上で NTP の稼働を必
須とします
([#6159](https://github.com/kubernetes/kubernetes/issues/6159#issuecomment-93844058)を
参照してください)。クロックは常に正しいものではありませんが、Node 間におけるその
差はとても小さいものとなります。TTL に 0 でない値をセットするときにこのリスクに
対して注意してください。

{{% /capture %}}

{{% capture whatsnext %}}

[Job の自動クリーンアップ](/docs/concepts/workloads/controllers/jobs-run-to-completion/#clean-up-finished-jobs-automatically)

[設計ドキュメント](https://github.com/kubernetes/community/blob/master/keps/sig-apps/0026-ttl-after-finish.md)

{{% /capture %}}
