---
title: Service
id: service
date: 2018-04-12
full_link: /ja/docs/concepts/services-networking/service/
short_description: >
  Podの集合で実行されているアプリケーションをネットワークサービスとして公開する方法。

aka:
tags:
  - fundamental
  - core-object
---

{{< glossary_tooltip text="Pods" term_id="pod" >}}の集合で実行されているアプリケ
ーションをネットワークサービスとして公開する抽象的な方法。

<!--more-->

Service が対象とする Pod の集合は、(通常
){{< glossary_tooltip text="セレクター" term_id="selector" >}}によって決定されま
す。 Pod を追加または削除するとセレクターにマッチしている Pod の集合は変更されま
す。 Service は、ネットワークトラフィックが現在そのワークロードを処理する Pod の
集合に向かうことを保証します。
