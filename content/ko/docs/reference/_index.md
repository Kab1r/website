---
title: 레퍼런스
linkTitle: "레퍼런스"
main_menu: true
weight: 70
content_template: templates/concept
---

{{% capture overview %}}

쿠버네티스 문서의 본 섹션에서는 레퍼런스를 다룬다.

{{% /capture %}}

{{% capture body %}}

## API 레퍼런스

- [쿠버네티스 API 개요](/ko/docs/reference/using-api/api-overview/) - 쿠버네티스
  API에 대한 개요
- [Kubernetes API 레퍼런스 {{< latest-version
  >}}](/docs/reference/generated/kubernetes-api/{{< latest-version >}}/)

## API 클라이언트 라이브러리

프로그래밍 언어에서 쿠버네티스 API를 호출하기 위해서,
[클라이언트 라이브러리](/ko/docs/reference/using-api/client-libraries/)를 사용�
수 있다. 공식적으로 지원되는 클라이언트 라이브러리는 다음과 같다.

- [쿠버네티스 Go 클라이언트 라이브러리](https://github.com/kubernetes/client-go/)
- [쿠버네티스 Python 클라이언트 라이브러리](https://github.com/kubernetes-client/python)
- [쿠버네티스 Java 클라이언트 라이브러리](https://github.com/kubernetes-client/java)
- [쿠버네티스 JavaScript 클라이언트 라이브러리](https://github.com/kubernetes-client/javascript)

## CLI 레퍼런스

- [kubectl](/docs/reference/kubectl/overview/) - 명령어를 실행하거나 쿠버네티스
  클러스터를 관리하기 위해 사용하는 주된 CLI 도구.
  - [JSONPath](/docs/reference/kubectl/jsonpath/) - kubectl에서
    [JSONPath 표현](http://goessner.net/articles/JsonPath/)을 사용하기 위한 문법
    가이드.
- [kubeadm](/docs/reference/setup-tools/kubeadm/kubeadm/) - 안정적인 쿠버네티스
  클러스터를 쉽게 프로비전하기 위한 CLI 도구.

## 설정 레퍼런스

- [kubelet](/docs/reference/command-line-tools-reference/kubelet/) - 각 노드에서
  구동되는 주요한 _노드 에이전트_. kubelet은 PodSpecs 집합을 가지며 기술된 컨테
  이너가 구동되고 있는지, 정상 작동하는지를 보장한다.
- [kube-apiserver](/docs/reference/command-line-tools-reference/kube-apiserver/) -
  파드, 서비스, 레플리케이션 컨트롤러와 같은 API 오브젝트에 대한 검증과 구성을
  수행하는 REST API.
- [kube-controller-manager](/docs/reference/command-line-tools-reference/kube-controller-manager/) -
  쿠버네티스에 탑재된 핵심 제어 루프를 포함하는 데몬.
- [kube-proxy](/docs/reference/command-line-tools-reference/kube-proxy/) - 간단
  한 TCP/UDP 스트림 포워딩이나 백-엔드 집합에 걸쳐서 라운드-로빈 TCP/UDP 포워딩
  을 할 수 있다.
- [kube-scheduler](/docs/reference/command-line-tools-reference/kube-scheduler/) -
  가용성, 성능 및 용량을 관리하는 스케줄러.

## 설계 문서

쿠버네티스 기능에 대한 설계 문서의 아카이브.
[쿠버네티스 아키텍처](https://git.k8s.io/community/contributors/design-proposals/architecture/architecture.md)와
[쿠버네티스 디자인 개요](https://git.k8s.io/community/contributors/design-proposals)가
좋은 출발점이다.

{{% /capture %}}
