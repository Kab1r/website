---
title: 인그레스
content_template: templates/concept
weight: 40
---

{{% capture overview %}}
{{< feature-state for_k8s_version="v1.1" state="beta" >}}
{{< glossary_definition term_id="ingress" length="all" >}} {{% /capture %}}

{{% capture body %}}

## 용어

이 가이드는 용어의 명확성을 위해 다음과 같이 정의한다.

노드(Node): 클러스터의 일부이며, 쿠버네티스에 속한 워커 머신. 클러스터(Cluster):
쿠버네티스에서 관리되는 컨테이너화 된 애플리케이션을 실행하는 노드 집합. 이 예시
와 대부분의 일반적인 쿠버네티스 배포에서 클러스터에 속한 노드는 퍼블릭 인터넷의
일부가 아니다. 에지 라우터(Edge router): 클러스터에 방화벽 정책을 적용하는 라우
터. 이것은 클라우드 공급자 또는 물리적 하드웨어의 일부에서 관리하는 게이트웨이일
수 있다. 클러스터 네트워크(Cluster network): 쿠버네티스
[네트워킹 모델](/docs/concepts/cluster-administration/networking/)에 따라 클러스
터 내부에서 통신을 용이하게 하는 논리적 또는 물리적 링크 집합. 서비스(Service):
{{< glossary_tooltip text="레이블" term_id="label" >}} 셀렉터를 사용해서 파드 집
합을 식별하는 쿠버네티스 {{< glossary_tooltip term_id="service" >}}. 달리 언급하
지 않으면 서비스는 클러스터 네트워크 내에서만 라우팅 가능한 가상 IP를 가지고 있
다고 가정한다.

## 인그레스란?

[인그레스](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#ingress-v1beta1-networking-k8s-io)
는 클러스터 외부에서 클러스터 내부
{{< link text="서비스" url="/docs/concepts/services-networking/service/" >}}로
HTTP와 HTTPS 경로를 노출한다. 트래픽 라우팅은 인그레스 리소스에 정의된 규칙에 의
해 컨트롤된다.

```none
    internet
        |
   [ Ingress ]
   --|-----|--
   [ Services ]
```

인그레스는 외부에서 서비스로 접속이 가능한 URL, 로드 밸런스 트래픽, SSL / TLS 종
료 그리고 이름 기반의 가상 호스팅을 제공하도록 구성할 수 있다.
[인그레스 컨트롤러](/ko/docs/concepts/services-networking/ingress-controllers)는
일반적으로 로드 밸런서를 사용해서 인그레스를 수행할 책임이 있으며, 트래픽을 처리
하는데 도움이 되도록 에지 라우터 또는 추가 프런트 엔드를 구성할 수도 있다.

인그레스는 임의의 포트 또는 프로토콜을 노출시키지 않는다. HTTP와 HTTPS 이외의 서
비스를 인터넷에 노출하려면 보통
[Service.Type=NodePort](/ko/docs/concepts/services-networking/service/#nodeport)
또는
[Service.Type=LoadBalancer](/ko/docs/concepts/services-networking/service/#loadbalancer)
유형의 서비스를 사용한다.

## 전제 조건들

[인그레스 컨트롤러](/ko/docs/concepts/services-networking/ingress-controllers)가
있어야 인그레스를 충족할 수 있다. 인그레스 리소스만 생성한다면 효과가 없다.

[ingress-nginx](https://kubernetes.github.io/ingress-nginx/deploy/)와 같은 인그
레스 컨트롤러를 배포해야 할 수도 있다. 여러
[인그레스 컨트롤러](/ko/docs/concepts/services-networking/ingress-controllers)
중에서 선택할 수도 있다.

이상적으로, 모든 인그레스 컨트롤러는 참조 사양이 맞아야 한다. 실제로, 다양한 인
그레스컨트롤러는 조금 다르게 작동한다.

{{< note >}} 인그레스 컨트롤러의 설명서를 검토하여 선택 시 주의 사항을 이해해야
한다. {{< /note >}}

## 인그레스 리소스

최소한의 인그레스 리소스 예제:

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - http:
        paths:
          - path: /testpath
            backend:
              serviceName: test
              servicePort: 80
```

다른 모든 쿠버네티스 리소스와 마찬가지로 인그레스에는 `apiVersion`, `kind`, 그리
고 `metadata` 필드가 필요하다. 인그레스 오브젝트의 이름은 유효한
[DNS 서브도메인 이름](/ko/docs/concepts/overview/working-with-objects/names/#dns-서브도메인-이름들)이
어야 한다. 설정 파일의 작성에 대한 일반적인 내용은
[애플리케이션 배포하기](/docs/tasks/run-application/run-stateless-application-deployment/),
[컨테이너 구성하기](/docs/tasks/configure-pod-container/configure-pod-configmap/),
[리소스 관리하기](/docs/concepts/cluster-administration/manage-deployment/)를 참
조한다. 인그레스는 종종 어노테이션을 이용해서 인그레스 컨트롤러에 따라 몇 가지
옵션을 구성하는데, 그 예시는
[재작성-타겟 어노테이션](https://github.com/kubernetes/ingress-nginx/blob/master/docs/examples/rewrite/README.md)이
다. 다른
[인그레스 컨트롤러](/ko/docs/concepts/services-networking/ingress-controllers)는
다른 어노테이션을 지원한다. 지원되는 어노테이션을 확인하려면 선택한 인그레스 컨
트롤러의 설명서를 검토한다.

인그레스
[사양](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status)
에는 로드 밸런서 또는 프록시 서버를 구성하는데 필요한 모든 정보가 있다. 가장 중
요한 것은, 들어오는 요청과 일치하는 규칙 목록을 포함하는 것이다. 인그레스 리소스
는 HTTP 트래픽을지시하는 규칙만 지원한다.

### 인그레스 규칙

각 HTTP 규칙에는 다음의 정보가 포함된다.

- 선택적 호스트. 이 예시에서는, 호스트가 지정되지 않기에 지정된 IP 주소를 통해
  모든 인바운드 HTTP 트래픽에 규칙이 적용 된다. 만약 호스트가 제공되면(예,
  foo.bar.com), 규칙이 해당 호스트에 적용된다.
- 경로 목록 (예, `/testpath`)에는 각각 `serviceName` 과 `servicePort` 가 정의되
  어있는 관련백엔드를 가지고 있다. 로드 밸런서가 트래픽을 참조된 서비스로 보내기
  전에 호스트와 경로가모두 수신 요청의 내용과 일치해야 한다.
- 백엔드는 [서비스 문서](/ko/docs/concepts/services-networking/service/)에 설명
  된 바와 같이서비스와 포트 이름의 조합이다. 호스트와 규칙 경로가 일치하는 인그
  레스에 대한 HTTP(와 HTTPS) 요청은 백엔드 목록으로 전송된다.

기본 백엔드는 종종 사양의 경로와 일치하지 않는 서비스에 대한 모든 요청을 처리하
도록 인그레스컨트롤러에 구성되는 경우가 많다.

### 기본 벡엔드

규칙이 없는 인그레스는 모든 트래픽을 단일 기본 백엔드로 전송한다. 기본백엔드는
일반적으로
[인그레스 컨트롤러](/ko/docs/concepts/services-networking/ingress-controllers)의
구성 옵션이며, 인그레스 리소스에 지정되어 있지 않다.

만약 인그레스 오브젝트의 HTTP 요청과 일치하는 호스트 또는 경로가 없으면, 트래픽
은기본 백엔드로 라우팅 된다.

## 인그레스 유형들

### 단일 서비스 인그레스

단일 서비스를 노출할 수 있는 기존 쿠버네티스 개념이 있다 ([대안](#대안)을 본다).
인그레스에 규칙 없이 _기본 백엔드_ 를 지정해서이를 수행할 수 있다.

{{< codenew file="service/networking/ingress.yaml" >}}

만약 `kubectl apply -f` 를 사용해서 생성한다면 방금 추가한 인그레스의상태를 볼
수 있어야 한다.

```shell
kubectl get ingress test-ingress
```

```
NAME           HOSTS     ADDRESS           PORTS     AGE
test-ingress   *         203.0.113.123   80        59s
```

여기서 `203.0.113.123` 는 인그레스 컨트롤러가 인그레스를 충족시키기 위해 할당한
IP 이다.

{{< note >}} 인그레스 컨트롤러와 로드 밸런서는 IP 주소를 할당하는데 1~2분이 걸릴
수 있다. 할당될 때 까지는 주소는 종종 `<pending>` 으로 표시된다. {{< /note >}}

### 간단한 팬아웃(fanout)

팬아웃 구성은 HTTP URI에서 요청된 것을 기반으로 단일 IP 주소에서 1개 이상의 서비
스로트래픽을 라우팅 한다. 인그레스를 사용하면 로드 밸런서의 수를최소로 유지할 수
있다. 예를 들어 다음과 같은 설정을 한다.

```none
foo.bar.com -> 178.91.123.132 -> / foo    service1:4200
                                 / bar    service2:8080
```

다음과 같은 인그레스가 필요하다.

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: simple-fanout-example
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: foo.bar.com
      http:
        paths:
          - path: /foo
            backend:
              serviceName: service1
              servicePort: 4200
          - path: /bar
            backend:
              serviceName: service2
              servicePort: 8080
```

`kubectl apply -f` 를 사용해서 인그레스를 생성 할 때 다음과 같다.

```shell
kubectl describe ingress simple-fanout-example
```

```
Name:             simple-fanout-example
Namespace:        default
Address:          178.91.123.132
Default backend:  default-http-backend:80 (10.8.2.3:8080)
Rules:
  Host         Path  Backends
  ----         ----  --------
  foo.bar.com
               /foo   service1:4200 (10.8.0.90:4200)
               /bar   service2:8080 (10.8.0.91:8080)
Annotations:
  nginx.ingress.kubernetes.io/rewrite-target:  /
Events:
  Type     Reason  Age                From                     Message
  ----     ------  ----               ----                     -------
  Normal   ADD     22s                loadbalancer-controller  default/test
```

인그레스 컨트롤러는 서비스(`service1`, `service2`)가 존재하는 한, 인그레스를 만
족시키는 특정한 로드 밸런서를 프로비저닝한다. 이렇게 하면, 주소 필드에서 로드 밸
런서의 주소를볼 수 있다.

{{< note >}} 사용중인
[인그레스 컨트롤러](/ko/docs/concepts/services-networking/ingress-controllers)
에 따라 default-http-backend
[서비스](/ko/docs/concepts/services-networking/service/)를 만들어야 할 수도 있다
. {{< /note >}}

### 이름 기반의 가상 호스팅

이름 기반의 가상 호스트는 동일한 IP 주소에서 여러 호스트 이름으로 HTTP 트래픽을
라우팅하는 것을 지원한다.

```none
foo.bar.com --|                 |-> foo.bar.com service1:80
              | 178.91.123.132  |
bar.foo.com --|                 |-> bar.foo.com service2:80
```

다음 인그레스는 [호스트 헤더](https://tools.ietf.org/html/rfc7230#section-5.4)에
기반한 요청을라우팅 하기 위해 뒷단의 로드 밸런서를 알려준다.

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: name-virtual-host-ingress
spec:
  rules:
    - host: foo.bar.com
      http:
        paths:
          - backend:
              serviceName: service1
              servicePort: 80
    - host: bar.foo.com
      http:
        paths:
          - backend:
              serviceName: service2
              servicePort: 80
```

만약 규칙에 정의된 호스트 없이 인그레스 리소스를 생성하는 경우, 이름 기반 가상
호스트가 없어도 인그레스 컨트롤러의 IP 주소에 대한 웹트래픽을 일치 시킬 수 있다.

예를 들어, 다음 인그레스 리소스는 `first.bar.com`에 요청된 트래픽을 `service1`로
, `second.foo.com`는 `service2`로, 호스트 이름이 정의되지않은(즉, 요청 헤더가 표
시 되지 않는) IP 주소로의 모든트래픽은 `service3`로 라우팅 한다.

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: name-virtual-host-ingress
spec:
  rules:
    - host: first.bar.com
      http:
        paths:
          - backend:
              serviceName: service1
              servicePort: 80
    - host: second.foo.com
      http:
        paths:
          - backend:
              serviceName: service2
              servicePort: 80
    - http:
        paths:
          - backend:
              serviceName: service3
              servicePort: 80
```

### TLS

TLS 개인 키 및 인증서가 포함된 {{< glossary_tooltip term_id="secret" >}} 을 지정
해서 인그레스를 보호할 수 있다. 현재 인그레스는단일 TLS 포트인 443만 지원하며
TLS 종료를 가정한다. 만약 인그레스의 TLS 구성 섹션에서 다른 호스트를 지정하면,
SNI TLS 확장을 통해지정된 호스트이름에 따라 동일한 포트에서 멀티플렉싱된다(인그
레스 컨트롤러가 SNI를 지원하는 경우). TLS secret에는 `tls.crt` 와 `tls.key` 라는
이름의 키가 있어야 하고, 여기에는 TLS에 사용할 인증서와개인 키가 있다. 예를 들어
다음과 같다.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: testsecret-tls
  namespace: default
data:
  tls.crt: base64 encoded cert
  tls.key: base64 encoded key
type: kubernetes.io/tls
```

인그레스에서 시크릿을 참조하면 인그레스 컨트롤러가 TLS를 사용하여클라이언트에서
로드 밸런서로 채널을 보호하도록 지시한다. 생성한 TLS 시크릿이
`sslexample.foo.com` 의 정규화 된 도메인 이름(FQDN)이라고하는 일반 이름(CN)을 포
함하는 인증서에서 온 것인지 확인해야 한다.

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: tls-example-ingress
spec:
  tls:
    - hosts:
        - sslexample.foo.com
      secretName: testsecret-tls
  rules:
    - host: sslexample.foo.com
      http:
        paths:
          - path: /
            backend:
              serviceName: service1
              servicePort: 80
```

{{< note >}} TLS 기능을 제공하는 다양한 인그레스 컨트롤러간의 기능차이가 있다.
사용자 환경에서의 TLS의 작동 방식을 이해하려면
[nginx](https://kubernetes.github.io/ingress-nginx/user-guide/tls/),
[GCE](https://git.k8s.io/ingress-gce/README.md#frontend-https) 또는 기타플랫폼의
특정 인그레스 컨트롤러에 대한 설명서를 참조한다. {{< /note >}}

### 로드밸런싱

인그레스 컨트롤러는 로드 밸런싱 알고리즘, 백엔드 가중치 구성표 등모든 인그레스에
적용되는 일부 로드 밸런싱정책 설정으로 부트스트랩된다. 보다 진보된 로드 밸런싱
개념 (예: 지속적인 세션, 동적 가중치)은 아직 인그레스를 통해노출되지 않는다. 대
신 서비스에 사용되는 로드 밸런서를 통해 이러한 기능을얻을 수 있다.

또한, 헬스 체크를 인그레스를 통해 직접 노출되지 않더라도, 쿠버네티스에는
[준비 상태 프로브](/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/)와
같은 동일한 최종 결과를 얻을 수 있는 병렬 개념이있다는 점도 주목할 가치가 있다.
컨트롤러 별설명서를 검토하여 헬스 체크를 처리하는 방법을 확인한다(
[nginx](https://git.k8s.io/ingress-nginx/README.md),
[GCE](https://git.k8s.io/ingress-gce/README.md#health-checks)).

## 인그레스 업데이트

기존 인그레스를 업데이트해서 새 호스트를 추가하려면, 리소스를 편집해서 호스트를
업데이트 할 수 있다.

```shell
kubectl describe ingress test
```

```
Name:             test
Namespace:        default
Address:          178.91.123.132
Default backend:  default-http-backend:80 (10.8.2.3:8080)
Rules:
  Host         Path  Backends
  ----         ----  --------
  foo.bar.com
               /foo   service1:80 (10.8.0.90:80)
Annotations:
  nginx.ingress.kubernetes.io/rewrite-target:  /
Events:
  Type     Reason  Age                From                     Message
  ----     ------  ----               ----                     -------
  Normal   ADD     35s                loadbalancer-controller  default/test
```

```shell
kubectl edit ingress test
```

YAML 형식의 기존 구성이 있는 편집기가 나타난다. 새 호스트를 포함하도록 수정한다.

```yaml
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - backend:
          serviceName: service1
          servicePort: 80
        path: /foo
  - host: bar.baz.com
    http:
      paths:
      - backend:
          serviceName: service2
          servicePort: 80
        path: /foo
..
```

변경사항을 저장한 후, kubectl은 API 서버의 리소스를 업데이트하며, 인그레스컨트롤
러에게도 로드 밸런서를 재구성하도록 지시한다.

이것을 확인한다.

```shell
kubectl describe ingress test
```

```
Name:             test
Namespace:        default
Address:          178.91.123.132
Default backend:  default-http-backend:80 (10.8.2.3:8080)
Rules:
  Host         Path  Backends
  ----         ----  --------
  foo.bar.com
               /foo   service1:80 (10.8.0.90:80)
  bar.baz.com
               /foo   service2:80 (10.8.0.91:80)
Annotations:
  nginx.ingress.kubernetes.io/rewrite-target:  /
Events:
  Type     Reason  Age                From                     Message
  ----     ------  ----               ----                     -------
  Normal   ADD     45s                loadbalancer-controller  default/test
```

수정된 인그레스 YAML 파일을 `kubectl replace -f` 를 호출해서 동일한 결과를 얻을
수 있다.

## 가용성 영역에 전체에서의 실패

장애 도메인에 트래픽을 분산시키는 기술은 클라우드 공급자마다 다르다. 자세한 내용
은
[인그레스 컨트롤러](/ko/docs/concepts/services-networking/ingress-controllers)
설명서를 확인한다. 페더레이션 클러스터에서 인그레스 배포에 대한 자세한 내용은
[페더레이션 설명서](https://github.com/kubernetes-sigs/federation-v2) 를 참조�
수 있다.

## 앞으로의 할일

[SIG Network](https://github.com/kubernetes/community/tree/master/sig-network)
를 추적하여 인그레스와 진행중인 리소스의 발전에 대한 자세한 내용을 알아 본다. 다
양한인그레스 컨트롤러의 발전에 대한 자세한 내용은
[인그레스 리포지터리](https://github.com/kubernetes/ingress/tree/master)에서 추
적할 수 있다.

## 대안

사용자는 인그레스 리소스를 직접적으로 포함하지 않는 여러가지 방법으로 서비스를
노출할 수 있다.

- [Service.Type=LoadBalancer](/ko/docs/concepts/services-networking/service/#loadbalancer)
  사용.
- [Service.Type=NodePort](/ko/docs/concepts/services-networking/service/#nodeport)
  사용.

{{% /capture %}}

{{% capture whatsnext %}}

- [인그레스]
  API](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#ingress-v1beta1-networking-k8s-io)
  에 대해 배우기
- [인그레스 컨트롤러](/ko/docs/concepts/services-networking/ingress-controllers/)에
  대해 배우기
- [NGINX 컨트롤러로 Minikube에서 인그레스 구성하기](/docs/tasks/access-application-cluster/ingress-minikube)
  {{% /capture %}}
