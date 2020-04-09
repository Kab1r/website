---
title: 리소스 쿼터
content_template: templates/concept
weight: 10
---

{{% capture overview %}}

여러 사용자나 팀이 정해진 수의 노드로 클러스터를 공유할 때한 팀이 공정하게 분배
된 리소스보다 많은 리소스를 사용할 수 있다는 우려가 있다.

리소스 쿼터는 관리자가 이 문제를 해결하기 위한 도구이다.

{{% /capture %}}

{{% capture body %}}

`ResourceQuota` 오브젝트로 정의된 리소스 쿼터는 네임스페이스별 총 리소스 사용을
제한하는제약 조건을 제공한다. 유형별로 네임스페이스에서 만들 수 있는 오브젝트 수
와해당 프로젝트의 리소스가 사용할 수 있는 총 컴퓨트 리소스의 양을제한할 수 있다.

리소스 쿼터는 다음과 같이 작동한다.

- 다른 팀은 다른 네임스페이스에서 작동한다. 현재 이것은 자발적이지만 ACL을 통해
  이 필수 사항을적용하기 위한 지원이 계획되어 있다.
- 관리자는 각 네임스페이스에 대해 하나의 `ResourceQuota`를 생성한다.
- 사용자는 네임스페이스에서 리소스(파드, 서비스 등)를 생성하고 쿼터 시스템은사용
  량을 추적하여 `ResourceQuota`에 정의된 하드(hard) 리소스 제한을 초과하지 않도
  록 한다.
- 리소스를 생성하거나 업데이트할 때 쿼터 제약 조건을 위반하면 위반된 제약 조건을
  설명하는메시지와 함께 HTTP 상태 코드 `403 FORBIDDEN`으로 요청이 실패한다.
- `cpu`, `memory`와 같은 컴퓨트 리소스에 대해 네임스페이스에서 쿼터가 활성화된
  경우사용자는 해당값에 대한 요청 또는 제한을 지정해야 한다. 그렇지 않으면 쿼터
  시스템이파드 생성을 거부할 수 있다. 힌트: 컴퓨트 리소스 요구 사항이 없는 파드
  를 기본값으로 설정하려면 `LimitRanger` 어드미션 컨트롤러를 사용하자. 이 문제를
  회피하는 방법에 대한 예제는
  [연습](/docs/tasks/administer-cluster/quota-memory-cpu-namespace/)을 참고하길
  바란다.

`ResourceQuota` 오브젝트의 이름은 유효한
[DNS 서브도메인 이름](/ko/docs/concepts/overview/working-with-objects/names#dns-서브도메인-이름)이
어야 한다.

네임스페이스와 쿼터를 사용하여 만들 수 있는 정책의 예는 다음과 같다.

- 용량이 32GiB RAM, 16 코어인 클러스터에서 A 팀이 20GiB 및 10 코어를 사용하고 B
  팀은 10GiB 및 4 코어를 사용하게 하고 2GiB 및 2 코어를 향후 할당을 위해 보유하
  도록 한다.
- "testing" 네임스페이스를 1 코어 및 1GiB RAM을 사용하도록 제한한다.
  "production" 네임스페이스에는 원하는 양을 사용하도록 한다.

클러스터의 총 용량이 네임스페이스의 쿼터 합보다 작은 경우 리소스에 대한 경합이
있을 수 있다. 이것은 선착순으로 처리된다.

경합이나 쿼터 변경은 이미 생성된 리소스에 영향을 미치지 않는다.

## 리소스 쿼터 활성화

많은 쿠버네티스 배포판에 기본적으로 리소스 쿼터 지원이 활성화되어 있다. API 서버
`--enable-admission-plugins=` 플래그의 인수 중 하나로 `ResourceQuota`가 있는 경
우 활성화된다.

해당 네임스페이스에 `ResourceQuota`가 있는 경우 특정 네임스페이스에 리소스 쿼터
가 적용된다.

## 컴퓨트 리소스 쿼터

지정된 네임스페이스에서 요청할 수 있는 총
[컴퓨트 리소스](/docs/user-guide/compute-resources) 합을 제한할 수 있다.

다음과 같은 리소스 유형이 지원된다.

| 리소스 이름       | 설명                                                                         |
| ----------------- | ---------------------------------------------------------------------------- |
| `limits.cpu`      | 터미널이 아닌 상태의 모든 파드에서 CPU 제한의 합은 이 값을 초과할 수 없음    |
| `limits.memory`   | 터미널이 아닌 상태의 모든 파드에서 메모리 제한의 합은 이 값을 초과할 수 없음 |
| `requests.cpu`    | 터미널이 아닌 상태의 모든 파드에서 CPU 요청의 합은 이 값을 초과할 수 없음    |
| `requests.memory` | 터미널이 아닌 상태의 모든 파드에서 메모리 요청의 합은 이 값을 초과할 수 없음 |

### 확장된 리소스에 대한 리소스 쿼터

위에서 언급한 리소스 외에도 릴리스 1.10에서는
[확장된 리소스](/docs/concepts/configuration/manage-compute-resources-container/#extended-resources)에
대한 쿼터 지원이 추가되었다.

확장된 리소스에는 오버커밋(overcommit)이 허용되지 않으므로 하나의 쿼터에서동일한
확장된 리소스에 대한 `requests`와 `limits`을 모두 지정하는 것은 의미가 없다. 따
라서 확장된리소스의 경우 지금은 접두사 `requests.`이 있는 쿼터 항목만 허용된다.

예를 들어, 리소스 이름이 `nvidia.com/gpu`이고 네임스페이스에서 요청된 총 GPU 수
를 4개로 제한하려는 경우, GPU 리소스를 다음과 같이 쿼터를 정의할 수 있다.

- `requests.nvidia.com/gpu: 4`

자세한 내용은 [쿼터 보기 및 설정](#쿼터-보기-및-설정)을 참고하길 바란다.

## 스토리지 리소스 쿼터

지정된 네임스페이스에서 요청할 수 있는 총
[스토리지 리소스](/ko/docs/concepts/storage/persistent-volumes/) 합을 제한할 수
있다.

또한 연관된 ​​스토리지 클래스를 기반으로 스토리지 리소스 사용을 제한할 수 있다.

| 리소스 이름                                                               | 설명                                                                                                                                                                                         |
| ------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `requests.storage`                                                        | 모든 퍼시스턴트 볼륨 클레임에서 스토리지 요청의 합은 이 값을 초과할 수 없음                                                                                                                  |
| `persistentvolumeclaims`                                                  | 네임스페이스에 존재할 수 있는 총 [퍼시스턴트 볼륨 클레임](/ko/docs/concepts/storage/persistent-volumes/#퍼시스턴트볼륨클레임) 수                                                             |
| `<storage-class-name>.storageclass.storage.k8s.io/requests.storage`       | storage-class-name과 관련된 모든 퍼시스턴트 볼륨 클레임에서 스토리지 요청의 합은 이 값을 초과할 수 없음                                                                                      |
| `<storage-class-name>.storageclass.storage.k8s.io/persistentvolumeclaims` | storage-class-name과 관련된 모든 퍼시스턴트 볼륨 클레임에서 네임스페이스에 존재할 수 있는 총 [퍼시스턴트 볼륨 클레임](/ko/docs/concepts/storage/persistent-volumes/#퍼시스턴트볼륨클레임) 수 |

예를 들어, 운영자가 `bronze` 스토리지 클래스와 별도로 `gold` 스토리지 클래스를
사용하여 스토리지에 쿼터를 지정하려는 경우 운영자는 다음과 같이쿼터를 정의할 수
있다.

- `gold.storageclass.storage.k8s.io/requests.storage: 500Gi`
- `bronze.storageclass.storage.k8s.io/requests.storage: 100Gi`

릴리스 1.8에서는 로컬 임시 스토리지에 대한 쿼터 지원이 알파 기능으로 추가되었다.

| 리소스 이름                  | 설명                                                                               |
| ---------------------------- | ---------------------------------------------------------------------------------- |
| `requests.ephemeral-storage` | 네임스페이스의 모든 파드에서 로컬 임시 스토리지 요청의 합은 이 값을 초과할 수 없음 |
| `limits.ephemeral-storage`   | 네임스페이스의 모든 파드에서 로컬 임시 스토리지 제한의 합은 이 값을 초과할 수 없음 |

## 오브젝트 수 쿼터

1.9 릴리스는 다음 구문을 사용하여 모든 표준 네임스페이스 리소스 유형에 쿼터를 지
정하는 지원을 추가했다.

- `count/<resource>.<group>`

다음은 사용자가 오브젝트 수 쿼터 아래에 배치하려는 리소스 셋의 예이다.

- `count/persistentvolumeclaims`
- `count/services`
- `count/secrets`
- `count/configmaps`
- `count/replicationcontrollers`
- `count/deployments.apps`
- `count/replicasets.apps`
- `count/statefulsets.apps`
- `count/jobs.batch`
- `count/cronjobs.batch`
- `count/deployments.extensions`

  1.15 릴리스는 동일한 구문을 사용하여 사용자 정의 리소스에 대한 지원을 추가했다
  . 예를 들어 `example.com` API 그룹에서 `widgets` 사용자 정의 리소스에 대한 쿼
  터를 생성하려면 `count/widgets.example.com`을 사용한다.

`count/*` 리소스 쿼터를 사용할 때 서버 스토리지 영역에 있다면 오브젝트는 쿼터에
대해 과금된다. 이러한 유형의 쿼터는 스토리지 리소스 고갈을 방지하는 데 유용하다.
예를 들어, 크기가 큰 서버에서 시크릿 수에 쿼터를 지정할 수 있다. 클러스터에 시크
릿이 너무 많으면 실제로 서버와컨트롤러가 시작되지 않을 수 있다! 네임스페이스에
너무 많은 작업을 생성하는잘못 구성된 크론 잡으로 인해 서비스 거부를 유발하는 것
으로부터 보호하기 위해 작업의 쿼터를 지정하도록 선택할 수 있다.

1.9 릴리스 이전에는 제한된 리소스 셋에서 일반 오브젝트 수 쿼터를 적용할 수 있었
다. 또한, 특정 리소스에 대한 쿼터를 유형별로 추가로 제한할 수 있다.

다음 유형이 지원된다.

| 리소스 이름              | 설명                                                                                                                                       |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `configmaps`             | 네임스페이스에 존재할 수 있는 총 구성 맵 수                                                                                                |
| `persistentvolumeclaims` | 네임스페이스에 존재할 수 있는 총 [퍼시스턴트 볼륨 클레임](/ko/docs/concepts/storage/persistent-volumes/#퍼시스턴트볼륨클레임) 수           |
| `pods`                   | 네임스페이스에 존재할 수 있는 터미널이 아닌 상태의 파드의 총 수. `.status.phase in (Failed, Succeeded)`가 true인 경우 파드는 터미널 상태임 |
| `replicationcontrollers` | 네임스페이스에 존재할 수 있는 총 레플리케이션 컨트롤러 수                                                                                  |
| `resourcequotas`         | 네임스페이스에 존재할 수 있는 총 [리소스 쿼터](/docs/reference/access-authn-authz/admission-controllers/#resourcequota) 수                 |
| `services`               | 네임스페이스에 존재할 수 있는 총 서비스 수                                                                                                 |
| `services.loadbalancers` | 네임스페이스에 존재할 수 있는 로드 밸런서 유형의 총 서비스 수                                                                              |
| `services.nodeports`     | 네임스페이스에 존재할 수 있는 노드 포트 유형의 총 서비스 수                                                                                |
| `secrets`                | 네임스페이스에 존재할 수 있는 총 시크릿 수                                                                                                 |

예를 들어, `pods` 쿼터는 터미널이 아닌 단일 네임스페이스에서 생성된 `pods` 수를
계산하고 최대값을 적용한다. 사용자가 작은 파드를 많이 생성하여 클러스터의 파드
IP 공급이 고갈되는 경우를 피하기 위해네임스페이스에 `pods` 쿼터를 설정할 수 있다
.

## 쿼터 범위

각 쿼터에는 연결된 범위 셋이 있을 수 있다. 쿼터는 열거된 범위의 교차 부분과 일치
하는 경우에만리소스 사용량을 측정한다.

범위가 쿼터에 추가되면 해당 범위와 관련된 리소스를 지원하는 리소스 수가 제한된다
. 허용된 셋 이외의 쿼터에 지정된 리소스는 유효성 검사 오류가 발생한다.

| 범위             | 설명                                                 |
| ---------------- | ---------------------------------------------------- |
| `Terminating`    | `.spec.activeDeadlineSeconds >= 0`에 일치하는 파드   |
| `NotTerminating` | `.spec.activeDeadlineSeconds is nil`에 일치하는 파드 |
| `BestEffort`     | 최상의 서비스 품질을 제공하는 파드                   |
| `NotBestEffort`  | 서비스 품질이 나쁜 파드                              |

`BestEffort` 범위는 다음의 리소스(파드)를 추적하도록 쿼터를 제한한다.

`Terminating`, `NotTerminating` 및 `NotBestEffort` 범위는 쿼터를 제한하여 다음의
리소스를 추적한다.

- `cpu`
- `limits.cpu`
- `limits.memory`
- `memory`
- `pods`
- `requests.cpu`
- `requests.memory`

### PriorityClass별 리소스 쿼터

{{< feature-state for_k8s_version="1.12" state="beta" >}}

특정
[우선 순위](/docs/concepts/configuration/pod-priority-preemption/#pod-priority)로
파드를 생성할 수 있다. 쿼터 스펙의 `scopeSelector` 필드를 사용하여 파드의 우�
순위에 따라 파드의 시스템 리소스 사용을제어할 수 있다.

쿼터 스펙의 `scopeSelector`가 파드를 선택한 경우에만 쿼터가 일치하고 사용된다.

이 예에서는 쿼터 오브젝트를 생성하여 특정 우선 순위의 파드와 일치시킨다. 예제는
다음과 같이 작동한다.

- 클러스터의 파드는 "low(낮음)", "medium(중간)", "high(높음)"의 세 가지 우선 순
  위 클래스 중 하나를 가진다.
- 각 우선 순위마다 하나의 쿼터 오브젝트가 생성된다.

다음 YAML을 `quota.yml` 파일에 저장한다.

```yaml
apiVersion: v1
kind: List
items:
  - apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: pods-high
    spec:
      hard:
        cpu: "1000"
        memory: 200Gi
        pods: "10"
      scopeSelector:
        matchExpressions:
          - operator: In
            scopeName: PriorityClass
            values: ["high"]
  - apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: pods-medium
    spec:
      hard:
        cpu: "10"
        memory: 20Gi
        pods: "10"
      scopeSelector:
        matchExpressions:
          - operator: In
            scopeName: PriorityClass
            values: ["medium"]
  - apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: pods-low
    spec:
      hard:
        cpu: "5"
        memory: 10Gi
        pods: "10"
      scopeSelector:
        matchExpressions:
          - operator: In
            scopeName: PriorityClass
            values: ["low"]
```

`kubectl create`를 사용하여 YAML을 적용한다.

```shell
kubectl create -f ./quota.yml
```

```shell
resourcequota/pods-high created
resourcequota/pods-medium created
resourcequota/pods-low created
```

`kubectl describe quota`를 사용하여 `Used` 쿼터가 `0`인지 확인하자.

```shell
kubectl describe quota
```

```shell
Name:       pods-high
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         0     1k
memory      0     200Gi
pods        0     10


Name:       pods-low
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         0     5
memory      0     10Gi
pods        0     10


Name:       pods-medium
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         0     10
memory      0     20Gi
pods        0     10
```

우선 순위가 "high"인 파드를 생성한다. 다음 YAML을 `high-priority-pod.yml` 파일에
저장한다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: high-priority
spec:
  containers:
    - name: high-priority
      image: ubuntu
      command: ["/bin/sh"]
      args: ["-c", "while true; do echo hello; sleep 10;done"]
      resources:
        requests:
          memory: "10Gi"
          cpu: "500m"
        limits:
          memory: "10Gi"
          cpu: "500m"
  priorityClassName: high
```

`kubectl create`로 적용하자.

```shell
kubectl create -f ./high-priority-pod.yml
```

"high" 우선 순위 쿼터가 적용된 `pods-high`에 대한 "Used" 통계가 변경되었고다른
두 쿼터는 변경되지 않았는지 확인한다.

```shell
kubectl describe quota
```

```shell
Name:       pods-high
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         500m  1k
memory      10Gi  200Gi
pods        1     10


Name:       pods-low
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         0     5
memory      0     10Gi
pods        0     10


Name:       pods-medium
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         0     10
memory      0     20Gi
pods        0     10
```

`scopeSelector`는 `operator` 필드에서 다음 값을 지원한다.

- `In`
- `NotIn`
- `Exist`
- `DoesNotExist`

## 요청과 제한의 비교 {#requests-vs-limits}

컴퓨트 리소스를 할당할 때 각 컨테이너는 CPU 또는 메모리에 대한 요청과 제한값을
지정할 수 있다. 쿼터는 값에 대한 쿼터를 지정하도록 구성할 수 있다.

쿼터에 `requests.cpu`나 `requests.memory`에 지정된 값이 있으면 들어오는 모든컨테
이너가 해당 리소스에 대한 명시적인 요청을 지정해야 한다. 쿼터에 `limits.cpu`나
`limits.memory`에 지정된 값이 있으면 들어오는 모든 컨테이너가 해당 리소스에 대한
명시적인 제한을 지정해야 한다.

## 쿼터 보기 및 설정

Kubectl은 쿼터 생성, 업데이트 및 보기를 지원한다.

```shell
kubectl create namespace myspace
```

```shell
cat <<EOF > compute-resources.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    requests.nvidia.com/gpu: 4
EOF
```

```shell
kubectl create -f ./compute-resources.yaml --namespace=myspace
```

```shell
cat <<EOF > object-counts.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-counts
spec:
  hard:
    configmaps: "10"
    persistentvolumeclaims: "4"
    pods: "4"
    replicationcontrollers: "20"
    secrets: "10"
    services: "10"
    services.loadbalancers: "2"
EOF
```

```shell
kubectl create -f ./object-counts.yaml --namespace=myspace
```

```shell
kubectl get quota --namespace=myspace
```

```shell
NAME                    AGE
compute-resources       30s
object-counts           32s
```

```shell
kubectl describe quota compute-resources --namespace=myspace
```

```shell
Name:                    compute-resources
Namespace:               myspace
Resource                 Used  Hard
--------                 ----  ----
limits.cpu               0     2
limits.memory            0     2Gi
requests.cpu             0     1
requests.memory          0     1Gi
requests.nvidia.com/gpu  0     4
```

```shell
kubectl describe quota object-counts --namespace=myspace
```

```shell
Name:                   object-counts
Namespace:              myspace
Resource                Used    Hard
--------                ----    ----
configmaps              0       10
persistentvolumeclaims  0       4
pods                    0       4
replicationcontrollers  0       20
secrets                 1       10
services                0       10
services.loadbalancers  0       2
```

Kubectl은 `count/<resource>.<group>` 구문을 사용하여 모든 표준 네임스페이스 리소
스에 대한오브젝트 수 쿼터를 지원한다.

```shell
kubectl create namespace myspace
```

```shell
kubectl create quota test --hard=count/deployments.extensions=2,count/replicasets.extensions=4,count/pods=3,count/secrets=4 --namespace=myspace
```

```shell
kubectl run nginx --image=nginx --replicas=2 --namespace=myspace
```

```shell
kubectl describe quota --namespace=myspace
```

```shell
Name:                         test
Namespace:                    myspace
Resource                      Used  Hard
--------                      ----  ----
count/deployments.extensions  1     2
count/pods                    2     3
count/replicasets.extensions  1     4
count/secrets                 1     4
```

## 쿼터 및 클러스터 용량

`ResourceQuotas`는 클러스터 용량과 무관하다. 그것들은 절대 단위로 표현된다. 따라
서 클러스터에 노드를 추가해도 각 네임스페이스에 더 많은 리소스를사용할 수 있는
기능이 자동으로 부여되지는 _않는다_.

가끔 다음과 같은 보다 복잡한 정책이 필요할 수 있다.

- 여러 팀으로 전체 클러스터 리소스를 비례적으로 나눈다.
- 각 테넌트가 필요에 따라 리소스 사용량을 늘릴 수 있지만, 실수로 리소스가 고갈되
  는 것을막기 위한 충분한 제한이 있다.
- 하나의 네임스페이스에서 요구를 감지하고 노드를 추가하며 쿼터를 늘린다.

이러한 정책은 쿼터 사용을 감시하고 다른 신호에 따라 각 네임스페이스의 쿼터 하드
제한을조정하는 "컨트롤러"를 작성하여 `ResourceQuotas`를 구성 요소로사용하여 구현
할 수 있다.

리소스 쿼터는 통합된 클러스터 리소스를 분할하지만 노드에 대한 제한은 없다. 여러
네임스페이스의 파드가 동일한 노드에서 실행될 수 있다.

## 기본적으로 우선 순위 클래스 소비 제한

파드가 특정 우선 순위, 예를 들어 일치하는 쿼터 오브젝트가 존재하는 경우에만
"cluster-services"가 네임스페이스에 허용되어야 힌다.

이 메커니즘을 통해 운영자는 특정 우선 순위가 높은 클래스의 사용을 제한된 수의 네
임스페이스로 제한할 수 있으며 모든 네임스페이스가 기본적으로 이러한 우선 순위 클
래스를 사용할 수 있는 것은 아니다.

이를 적용하려면 kube-apiserver 플래그 `--admission-control-config-file`을 사용하
여 다음 구성 파일의 경로를 전달해야 한다.

{{< tabs name="example1" >}} {{% tab name="apiserver.config.k8s.io/v1" %}}

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
  - name: "ResourceQuota"
    configuration:
      apiVersion: apiserver.config.k8s.io/v1
      kind: ResourceQuotaConfiguration
      limitedResources:
        - resource: pods
          matchScopes:
            - scopeName: PriorityClass
              operator: In
              values: ["cluster-services"]
```

{{% /tab %}} {{% tab name="apiserver.k8s.io/v1alpha1" %}}

```yaml
# Deprecated in v1.17 in favor of apiserver.config.k8s.io/v1
apiVersion: apiserver.k8s.io/v1alpha1
kind: AdmissionConfiguration
plugins:
  - name: "ResourceQuota"
    configuration:
      # Deprecated in v1.17 in favor of apiserver.config.k8s.io/v1, ResourceQuotaConfiguration
      apiVersion: resourcequota.admission.k8s.io/v1beta1
      kind: Configuration
      limitedResources:
        - resource: pods
          matchScopes:
            - scopeName: PriorityClass
              operator: In
              values: ["cluster-services"]
```

{{% /tab %}} {{< /tabs >}}

이제 "cluster-services" 파드는 `scopeSelector`와 일치하는 쿼터 오브젝트가 있는
네임스페이스에서만 허용된다. 예를 들면 다음과 같다.

```yaml
scopeSelector:
  matchExpressions:
    - scopeName: PriorityClass
      operator: In
      values: ["cluster-services"]
```

자세한 내용은
[LimitedResources](https://github.com/kubernetes/kubernetes/pull/36765)와
[우선 순위 클래스에 대한 쿼터 지원 디자인 문서](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/pod-priority-resourcequota.md)를
참고하길 바란다.

## 예제

[리소스 쿼터를 사용하는 방법에 대한 자세한 예](/docs/tasks/administer-cluster/quota-api-object/)를
참고하길 바란다.

{{% /capture %}}

{{% capture whatsnext %}}

자세한 내용은
[리소스쿼터 디자인 문서](https://git.k8s.io/community/contributors/design-proposals/resource-management/admission_control_resource_quota.md)를
참고하길 바란다.

{{% /capture %}}
