---
title: StatefulSet
id: statefulset
date: 2018-04-12
full_link: /ja/docs/concepts/workloads/controllers/statefulset/
short_description: >
  Manages the deployment and scaling of a set of Pods, *and provides guarantees
  about the ordering and uniqueness* of these Pods.

aka:
tags:
  - fundamental
  - core-object
  - workload
  - storage
---

StatefulSet は Deployment と{{< glossary_tooltip text="Pod" term_id="pod" >}}の
セットのスケーリングの管理をし、それらの Pod の*順序とユニーク性を保証* します。

<!--more-->

{{< glossary_tooltip term_id="deployment" >}}のように、StatefulSet は指定したコ
ンテナの spec に基づいて Pod を管理します。Deployment とは異なり、StatefulSet は
各 Pod において管理が大変な同一性を維持します。これらの Pod は同一の spec から作
成されますが、それらは交換可能ではなく、リスケジュール処理をまたいで維持される永
続的な識別子を持ちます。

StatefulSet は他のコントローラーと同様のパターンで動作します。ユーザーは
StatefulSet*オブジェクト* の理想的な状態を定義し、StatefulSet*コントローラー* は
現在の状態から、理想状態になるために必要なアップデートを行います。
