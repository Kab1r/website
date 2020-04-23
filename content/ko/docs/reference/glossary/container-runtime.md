---
title: 컨테이너 런타임
id: container-runtime
date: 2019-06-05
full_link: /docs/setup/production-environment/container-runtimes
short_description: >
  컨테이너 런타임은 컨테이너 실행을 담당하는 소프트웨어이다.

aka:
tags:
  - fundamental
  - workload
---

컨테이너 런타임은 컨테이너 실행을 담당하는 소프트웨어이다.

<!--more-->

쿠버네티스는 여러 컨테이너 런타임을 지원한다.
{{< glossary_tooltip term_id="docker">}},
{{< glossary_tooltip term_id="containerd" >}},
{{< glossary_tooltip term_id="cri-o" >}} 그리�
[Kubernetes CRI (컨테이너 런타임인터페이스)](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md)를
구현한 모든 소프트웨어.
