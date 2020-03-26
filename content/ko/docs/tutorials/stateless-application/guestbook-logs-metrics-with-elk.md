---
title: "예제: PHP / Redis 방명록 예제에 로깅과 메트릭 추가"
reviewers:
content_template: templates/tutorial
weight: 21
card:
  name: tutorials
  weight: 31
  title: "예제: PHP / Redis 방명록 예제에 로깅과 메트릭 추가"
---

{{% capture overview %}} 이 튜토리얼은 [Redis를 이용한 PHP 방명록](../guestbook)
튜토리얼을 기반으로 한다. Elastic의 경량 로그, 메트릭, 네트워크 데이터 오픈소스
배송기인 _Beats_ 를 방명록과 동일한 쿠버네티스 클러스터에 배포한다. Beats는 데이
터를 수집하고 구문분석하여 Elasticsearch에 색인화하므로, Kibana에서 동작 정보를
결과로 보며 분석할 수 있다. 이 예시는 다음과 같이 구성되어 있다.

- [Redis를 이용한 PHP 방명록](../guestbook)을 실행한 인스턴스
- Elasticsearch와 Kibana
- Filebeat
- Metricbeat
- Packetbeat

{{% /capture %}}

{{% capture objectives %}}

- Redis를 이용한 PHP 방명록 시작.
- kube-state-metrics 설치.
- 쿠버네티스 시크릿 생성.
- Beats 배포.
- 로그와 메트릭의 대시보드 보기. {{% /capture %}}

{{% capture prerequisites %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

추가로 다음이 필요하다.

- 실행 중인 [Redis를 이용한 PHP 방명록](../guestbook) 튜토리얼의 배포본.

- 실행 중인 Elasticsearch와 Kibana 디플로이먼트.
  [Elastic Cloud의 Elasticsearch 서비스](https://cloud.elastic.co)를 사용하거나,
  [파일을 내려받아](https://www.elastic.co/guide/en/elastic-stack-get-started/current/get-started-elastic-stack.html)
  워크스테이션이나 서버에서 운영하거나,
  [Elastic의 Helm 차트](https://github.com/elastic/helm-charts)를 이용한다.

{{% /capture %}}

{{% capture lessoncontent %}}

## Redis를 이용한 PHP 방명록 시작

이 튜토리얼은 [Redis를 이용한 PHP 방명록](../guestbook)을 기반으로 한다. 방명록
애플리케이션을 실행 중이라면, 이를 모니터링할 수 있다. 실행되지 않은 경우라면 지
침을 따라 방명록을 배포하고 **정리하기** 단계는 수행하지 말자. 방명록을 실행�
때 이 페이지로 돌아오자.

## 클러스터 롤 바인딩 추가

[클러스터 단위 롤 바인딩](/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding)을
생성하여, 클러스터 수준(kube-system 안에)으로 kube-state-metrics와 Beats를 배포
할 수 있게 한다.

```shell
kubectl create clusterrolebinding cluster-admin-binding \
 --clusterrole=cluster-admin --user=<your email associated with the k8s provider account>
```

## kube-state-metrics 설치

[_kube-state-metrics_](https://github.com/kubernetes/kube-state-metrics)는 쿠버
네티스 API 서버를 모니터링하며 오브젝트 상태에 대한 메트릭을 생성하는 간단한 서
비스이다. 이런 메트릭을 Metricbeat이 보고한다. 방명록이 실행된 쿠버네티스 클러스
터에서 kube-state-metrics을 추가한다.

### kube-state-metrics 실행 여부 확인

```shell
kubectl get pods --namespace=kube-system | grep kube-state
```

### 필요하면 kube-state-metrics 설치

```shell
git clone https://github.com/kubernetes/kube-state-metrics.git kube-state-metrics
kubectl create -f examples/standard
kubectl get pods --namespace=kube-system | grep kube-state-metrics
```

kube-state-metrics이 실행 중이고 준비되었는지 확인한다.

```shell
kubectl get pods -n kube-system -l app.kubernetes.io/name=kube-state-metrics
```

출력

```shell
NAME                                 READY   STATUS    RESTARTS   AGE
kube-state-metrics-89d656bf8-vdthm   2/2     Running     0          21s
```

## Elastic의 예제를 GitHub 리포지터리에 클론한다.

```shell
git clone https://github.com/elastic/examples.git
```

나머지 커맨드는 `examples/beats-k8s-send-anywhere` 디렉터리의 파일을 참조할 것이
라서, 그쪽으로 현재 디렉터리를 변경한다.

```shell
cd examples/beats-k8s-send-anywhere
```

## 쿠버네티스 시크릿 만들기

쿠버네티스 {{< glossary_tooltip text="시크릿" term_id="secret" >}}은 암호나 토큰
, 키 같이 소량의 민감한 데이터를 포함하는 오브젝트이다. 이러한 정보는 다른 방식
으로도 파드 스펙이나 이미지에 넣을 수 있을 것이다. 시크릿 오브젝트에 넣으면 이것
이 어떻게 사용되는지 다양하게 제어할 수 있고, 우발적인 노출 사고의 위험이 줄일
수 있다.

{{< note >}} 여기에는 방식이 나뉘는데, 하나는 _자체 관리(Self managed)_ 로
Elasticsearch와 Kibana(Elastic의 Helm 차트를 이용하여 사용자 서버를 구동하는)를
사용하는 경우이고 다른 경우는 Elastic Cloud의 Elasticsearch 서비스의 _관리 서비
스(Managed service)_ 를 사용하는 방식이다. 이 튜토리얼에서는 사용�
Elasticsearch와 Kibana 시스템의 종류에 따라 시크릿을 만들어야 한다.
{{< /note >}}

{{< tabs name="tab_with_md" >}} {{% tab name="자체 관리(Self Managed)" %}}

### 자체 관리

Elastic Cloud의 Elasticsearch 서비스로 연결한다면 **관리 서비스** 탭으로 전환한
다.

### 자격증명(credentials) 설정

자체 관리 Elasticsearch와 Kibana(자체 관리는 사실상 Elastic Cloud의 관리 서비스
Elasticsearch와 다르다) 서비스에 접속할 때에 4개 파일을 수정하여 쿠버네티스 시크
릿을 생성한다. 파일은 다음과 같다.

1. ELASTICSEARCH_HOSTS
1. ELASTICSEARCH_PASSWORD
1. ELASTICSEARCH_USERNAME
1. KIBANA_HOST

이 정보를 Elasticsearch 클러스터와 Kibana 호스트에 지정한다. 여기 예시(또는
[_이 구성_](https://stackoverflow.com/questions/59892896/how-to-connect-from-minikube-to-elasticsearch-installed-on-host-local-developme/59892897#59892897)을
본다)가 있다.

#### `ELASTICSEARCH_HOSTS`

1.  Elastic의 Elasticsearch Helm 차트에서 노드 그룹(nodeGroup).

    ```shell
    ["http://elasticsearch-master.default.svc.cluster.local:9200"]
    ```

1.  Mac을 위한 Docker에서 Beats를 운영 중인 Mac에서 운영하는 단일 Elasticsearch
    노드.

    ```shell
    ["http://host.docker.internal:9200"]
    ```

1.  VM이나 물리 장비에서 운영 중인 두 개의 ELASTICSEARCH 노드.

        ```shell
        ["http://host1.example.com:9200", "http://host2.example.com:9200"]
        ```

    `ELASTICSEARCH_HOSTS` 수정한다.

```shell
vi ELASTICSEARCH_HOSTS
```

#### `ELASTICSEARCH_PASSWORD`

화이트 스페이스나 인용 부호나 <> 도 없는 암호이다.

    <사용자의 시크릿 암호>

`ELASTICSEARCH_PASSWORD` 수정한다.

```shell
vi ELASTICSEARCH_PASSWORD
```

#### `ELASTICSEARCH_USERNAME`

화이트 스페이스나 인용 부호나 <> 도 없는 이름이다.

    <Elasticsearch를 위한 수집 사용자 이름>

`ELASTICSEARCH_USERNAME` 수정한다.

```shell
vi ELASTICSEARCH_USERNAME
```

#### `KIBANA_HOST`

1.Elastic의 Kibana Helm 차트의 인스턴스이다. 하위 도메인 `default`는 기본 네임스
페이스를 참조한다. 다른 네임스페이스를 사용하여 Helm 차트를 배포한 경우 하위 도
메인이 다릅니다.

    ```shell
    "kibana-kibana.default.svc.cluster.local:5601"
    ```

1.  Mac 용 Docker에서 실행하는 Beats가 있는 Mac에서 실행하는 Kibana 인스턴스

    ```shell
    "host.docker.internal:5601"
    ```

1.  가상머신이나 물리적 하드웨어에서 실행 중인 두 개의 Elasticsearch 노드

        ```shell
        "host1.example.com:5601"
        ```

    `KIBANA_HOST`를 편집한다.

```shell
vi KIBANA_HOST
```

### 쿠버네티스 시크릿 만들기

이 커맨드는 방금 편집한 파일을 기반으로 쿠버네티스의 시스템 수준의 네임스페이스
(kube-system)에 시크릿을 만든다.

    kubectl create secret generic dynamic-logging \
      --from-file=./ELASTICSEARCH_HOSTS \
      --from-file=./ELASTICSEARCH_PASSWORD \
      --from-file=./ELASTICSEARCH_USERNAME \
      --from-file=./KIBANA_HOST \
      --namespace=kube-system

{{% /tab %}} {{% tab name="관리 서비스(Managed service)" %}}

## 관리 서비스

이 탭은 Elastic Cloud에서 Elasticsearch 서비스 만에 대한 것으로, 이미 자체 관리
Elasticsearch와 Kibana 배포로 시크릿을 생성했다면,
[Beats 배포](#deploy-the-beats)를 계속한다.

### 자격증명(credentials) 설정

Elastic Cloud에서 관리되는 Elastic 서비스에 연결할 때, 쿠버네티스 시크릿을 생성
하기 위해 편집할 두 파일이 있다. 파일은 다음과 같다.

1. ELASTIC_CLOUD_AUTH
1. ELASTIC_CLOUD_ID

디플로이먼트를 생성할 때에 Elasticsearch 콘솔에서 제공한 정보로 이를 설정한다.
여기 예시들이 있다.

#### ELASTIC_CLOUD_ID

```shell
devk8s:ABC123def456ghi789jkl123mno456pqr789stu123vwx456yza789bcd012efg345hijj678klm901nop345zEwOTJjMTc5YWQ0YzQ5OThlN2U5MjAwYTg4NTIzZQ==
```

#### ELASTIC_CLOUD_AUTH

사용자 이름, 콜론(`:`) 및 비밀번호인데, 공백 또는 따옴표는 없다.

```shell
elastic:VFxJJf9Tjwer90wnfTghsn8w
```

### 필요 파일 편집하기

```shell
vi ELASTIC_CLOUD_ID
vi ELASTIC_CLOUD_AUTH
```

### 쿠버네티스 시크릿 생성하기

이 커맨드는 방금 편집한 파일을 기반으로 쿠버네티스의 시스템 수준의 네임스페이스
(kube-system)에 시크릿을 생성한다.

    kubectl create secret generic dynamic-logging \
      --from-file=./ELASTIC_CLOUD_ID \
      --from-file=./ELASTIC_CLOUD_AUTH \
      --namespace=kube-system

{{% /tab %}} {{< /tabs >}}

## Beats 배포하기 {#deploy-the-beats}

각 Beat마다 메니페스트 파일을 제공한다. 이 메니페스트 파일은 앞서 생성한 시크릿
을 사용하여, Elasticsearch 및 Kibana 서버에 연결하도록 Beats를 구성한다.

### Filebeat에 대해

Filebeat는 쿠버네티스 노드와 해당 노두에서 실행되는 각 파드에서 실행되는 컨테이
너의 로그를 수집한다. Filebeat는
{{< glossary_tooltip text="데몬 셋" term_id="daemonset" >}}으로 배포한다.
Filebeat는 쿠버네티스 클러스터에서 실행 중인 애플리케이션을 자동 검색할 수 있다.
시작시에 Filebeat는 기존 컨테이너를 검색하고 이에 적절한 구성을 시작하고 새 시작
/종료 이벤트를 감시한다.

아래 내용은 Filebeat가 방명록 애플리케이션과 함께 배포된 Redis 컨테이너에서
Redis 로그를 찾아 구문분석할 수 있게 하는 자동 검색 구성이다. 이 구성은
`filebeat-kubernetes.yaml`파일에 있다.

```yaml
- condition.contains:
    kubernetes.labels.app: redis
  config:
    - module: redis
      log:
        input:
          type: docker
          containers.ids:
            - ${data.kubernetes.container.id}
      slowlog:
        enabled: true
        var.hosts: ["${data.host}:${data.port}"]
```

이것은 `redis` 컨테이너가 `app` 문자열을 포함하는 레이블로 감지될 때에 Filebeat
모듈 `redis`를 적용하도록 Filebeat를 구성한다. Redis 모듈은 Docker 입력 유형을
사용하여 컨테이너에서 `로그` 스트림을 수집할 수 있다(이 Redis 컨테이너의 STDOUT
스트림과 연관된 쿠버네티스 노드에서 파일 읽기). 또한 이 모듈은 컨테이너 메타 데
이터에 제공되는 적절한 파드 호스트와 포트에 연결하여 Redis의 `slowlog` 항목을 수
집할 수 있다.

### Filebeat 배포

```shell
kubectl create -f filebeat-kubernetes.yaml
```

#### 확인

```shell
kubectl get pods -n kube-system -l k8s-app=filebeat-dynamic
```

### Metricbeat에 대해

Metricbeat 자동 검색은 Filebeat와 같은 방식으로 구성된다. 다음은 Redis 컨테이너
에 대한 Metricbeat의 자동 검색 구성이다. 이 구성은
`metricbeat-kubernetes.yaml`에 있다.

```yaml
- condition.equals:
    kubernetes.labels.tier: backend
  config:
    - module: redis
      metricsets: ["info", "keyspace"]
      period: 10s

      # Redis hosts
      hosts: ["${data.host}:${data.port}"]
```

이것은 컨테이너가 `tier` 레이블이 `backend` 문자열과 같은 레이블로 감지될 때에
Metricbeat 모듈 `redis`를 적용하도록 Metricbeat를 구성한다. `redis` 모듈은 컨테
이너 메타데이터에 제공되는 적절한 파드 호스트와 포트에 연결하여 컨테이너에서
`info` 및 `keyspace` 메트릭을 수집할 수 있다.

### Metricbeat 배포

```shell
kubectl create -f metricbeat-kubernetes.yaml
```

#### 확인

```shell
kubectl get pods -n kube-system -l k8s-app=metricbeat
```

### Packetbeat에 대해

Packetbeat 구성은 Filebeat와 Metricbeat와는 다르다. 컨테이너 레이블과 일치시킬
패턴을 지정하지 않고, 구성은 관련 프로토콜 및 포트 번호를 기반으로 한다. 아래는
포트 번호의 하위 집합이다.

{{< note >}} 비표준 포트로 서비스를 실행했다면 해당 포트를 `filebeat.yaml`에 적
절한 유형에 추가하고, Packetbeat 데몬 셋을 삭제하고 생성한다. {{< /note >}}

```yaml
packetbeat.interfaces.device: any

packetbeat.protocols:
  - type: dns
    ports: [53]
    include_authorities: true
    include_additionals: true

  - type: http
    ports: [80, 8000, 8080, 9200]

  - type: mysql
    ports: [3306]

  - type: redis
    ports: [6379]

packetbeat.flows:
  timeout: 30s
  period: 10s
```

#### Packetbeat 배포하기

```shell
kubectl create -f packetbeat-kubernetes.yaml
```

#### 확인하기

```shell
kubectl get pods -n kube-system -l k8s-app=packetbeat-dynamic
```

## Kibana에서 보기

브라우저에서 Kibana를 열고, **대시보드** 애플리케이션을 열어보자. 검색창에
kubernetes를 입력하고 쿠버네티스를 위한 Metricbeat 대시보드를 클릭한다. 이 대시
보드에는 노드 상태, 배포 등의 보고 내용이 있다.

대시보드 페이지에 Packetbeat를 검색하고 Packetbeat의 개요 페이지를 살펴보자.

마찬가지로 Apache와 Redis를 위한 대시보드를 확인한다. 각 로그와 메트릭에 대한 대
시보드가 표시된다. 이 Apache Metricbeat 대시보드는 비어 있다. Apache Filebeat 대
시보드를 보고, 맨 아래로 스크롤하여 Apache 오류 로그를 확인한다. Apache에서 보여
줄 메트릭이 없는 이유를 알려줄 것이다.

Metricbeat에서 Apache 메트릭을 가져올 수 있게 하려면, mod-status 구성 파일을 포
함한 컨피그맵을 추가하고 방명록을 재배포하여 서버 상태를 활성화해야 한다.

## 디플로이먼트를 확장하고 모니터링중인 새 파드를 확인하기

기존 디플로이먼트를 확인한다.

```shell
kubectl get deployments
```

출력

```shell
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
frontend        3/3     3            3           3h27m
redis-master    1/1     1            1           3h27m
redis-slave     2/2     2            2           3h27m
```

front의 디플로이먼트를 두 개의 파드로 축소한다.

```shell
kubectl scale --replicas=2 deployment/frontend
```

출력

```shell
deployment.extensions/frontend scaled
```

## Kibana에서 변화 확인하기

스크린 캡처를 확인하여, 표시된 필터를 추가하고 해당 열을 뷰에 추가한다.
ScalingReplicaSet 항목이 표시되고, 여기에서 이벤트 목록의 맨 위에 풀링되는 이미
지, 마운트된 볼륨, 파드 시작 등을 보여준다.
![Kibana 디스커버리](https://raw.githubusercontent.com/elastic/examples/master/beats-k8s-send-anywhere/scaling-discover.png)

{{% /capture %}}

{{% capture cleanup %}} 디플로이먼트와 서비스를 삭제하면 실행중인 파드도 삭제된
다. 한 커맨드로 여러 개의 리소스를 삭제하기 위해 레이블을 이용한다.

1. 다음 커맨드를 실행하여 모든 파드, 디플로이먼트, 서비스를 삭제한다.

   ```shell
   kubectl delete deployment -l app=redis
   kubectl delete service -l app=redis
   kubectl delete deployment -l app=guestbook
   kubectl delete service -l app=guestbook
   kubectl delete -f filebeat-kubernetes.yaml
   kubectl delete -f metricbeat-kubernetes.yaml
   kubectl delete -f packetbeat-kubernetes.yaml
   kubectl delete secret dynamic-logging -n kube-system
   ```

1. 실행 중인 파드가 없음을 확인하기 위해 파드 목록을 조회한다.

   ```shell
   kubectl get pods
   ```

   커맨드의 출력은 다음과 같아야 한다.

   ```
   No resources found.
   ```

{{% /capture %}}

{{% capture whatsnext %}}

- [리소스 모니터링 도구](/ko/docs/tasks/debug-application-cluster/resource-usage-monitoring/)를
  공부한다.
- [로깅 아키텍처](/docs/concepts/cluster-administration/logging/)를 더 읽어본다.
- [애플리케이션 검사 및 디버깅](/ko/docs/tasks/debug-application-cluster/)을 더
  읽어본다.
- [애플리케이션 문제 해결](/ko/docs/tasks/debug-application-cluster/resource-usage-monitoring/)을
  더 읽어본다. {{% /capture %}}
