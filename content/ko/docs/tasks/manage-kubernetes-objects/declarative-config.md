---
title: 구성 파일을 이용한 쿠버네티스 오브젝트의 선언형 관리
content_template: templates/task
weight: 10
---

{{% capture overview %}} 쿠버네티스 오브젝트는 여러 개의 오브젝트 구성 파일을디
렉터리에 저장하고 필요에 따라 `kubectl apply`를사용하여 재귀적으로 오브젝트를 생
성하고 업데이트함으로써 생성, 업데이트 및 삭제할 수 있다. 이 방식은 변경사항을
되돌려 오브젝트 구성 파일에 병합하지 않고활성 오브젝트에 가해진 기록을 유지한다.
`kubectl diff`는 또한 `apply`가 어떠한 변경사항을 이루어질지에 대한 프리뷰를 제
공한다. {{% /capture %}}

{{% capture prerequisites %}}

[`kubectl`](/docs/tasks/tools/install-kubectl/)를 설치한다.

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

{{% /capture %}}

{{% capture steps %}}

## 트레이드 오프

`kubectl` 툴은 세 가지 방식의 오브젝트 관리를 지원한다.

- 명령형 커맨드
- 명령형 오브젝트 구성
- 선언형 오브젝트 구성

오브젝트 관리 방식의 종류별 장단점에 대한 논의는
[쿠버네티스 오브젝트 관리](/ko/docs/concepts/overview/working-with-objects/object-management/)를
참고한다.

## 개요

선언형 오브젝트 구성은 쿠버네티스 오브젝트 정의와구성에 대한 확실한 이해가 필요
하다. 아직 그렇지 못하다면, 먼저 다음 문서를 읽고 이해한다.

- [명령형 커맨드를 사용한 쿠버네티스 오브젝트 관리하기](/ko/docs/tasks/manage-kubernetes-objects/imperative-command/)
- [구성 파일을 사용한 쿠버네티스 오브젝트 명령형 관리](/ko/docs/tasks/manage-kubernetes-objects/imperative-config/)

다음은 이 문서에서 사용되는 용어에 대한 정의이다.

- _오브젝트 구성 파일 / 구성 파일_: 쿠버네티스 오브젝트에 대한구성을 정의하는 하
  나의 파일. 이 주제는 어떻게 `kubectl apply`에 구성 파일을 전달하는지에 대해 보
  여준다. 구성 파일은 일반적으로 Git과 같은, 소스 컨트롤에 저장된다.
- _활성 오브젝트 구성 / 활성 구성_: 쿠버네티스 클러스터에 의해 관측된오브젝트에
  대한 활성 구성 값. 이것들은 쿠버네티스 클러스터 저장소에 유지된다. 일반적으로
  etcd가 사용된다.
- _선언형 구성 작성자 / 선언형 작성자_: 활성 오브젝트를 업데이트해 주는사람이나
  소프트웨어. 이 주제에서 언급하는 활성 작성자는 오브젝트 구성 파일에 변경을 가
  하고 `kubectl apply`를 실행하여 변경사항을 기록한다.

## 오브젝트 생성 방법

기존에 존재하는 것을 제외한, 지정한 디렉터리 내 구성 파일에 의해 정의된 모든 오
브젝트를 생성하기 위해 `kubectl apply`를사용한다.

```shell
kubectl apply -f <디렉터리>/
```

이것은  각 오브젝트에 대해
`kubectl.kubernetes.io/last-applied-configuration: '{...}'` 어노테이션을 설정한
다. 해당 어노테이션은 오브젝트를 생성하기 위해 사용했던오브젝트 구성 파일의 내용
을 포함한다.

{{< note >}} 재귀적으로 디렉터리를 처리하기 위해서 `-R` 플래그를 추가한다.�
{{< /note >}}

다음은 오브젝트 구성 파일에 대한 예시이다.

{{< codenew file="application/simple_deployment.yaml" >}}

생성될 오브젝트를 출력하려면 `kubectl diff`를 실행한다.

```shell
kubectl diff -f https://k8s.io/examples/application/simple_deployment.yaml
```

{{< note >}} `diff`는 `kube-apiserver`의 활성화가 필요한
[서버사이드 dry-run](/docs/reference/using-api/api-concepts/#dry-run)을 사용한다
.

`diff` 는 dry-run 모드에서 서버 측 적용 요청을 수행하므로, `PATCH`, `CREATE`, 그
리고 `UPDATE` 권한을 부여해야 한다. 자세한 것은
[Dry-Run 인증](/docs/reference/using-api/api-concepts#dry-run-authorization)을
본다.

{{< /note >}}

`kubectl apply`를 사용하여 오브젝트를 생성한다.

```shell
kubectl apply -f https://k8s.io/examples/application/simple_deployment.yaml
```

`kubectl get`을 사용하여 활성 구성을 출력한다.

```shell
kubectl get -f https://k8s.io/examples/application/simple_deployment.yaml -o yaml
```

출력은 `kubectl.kubernetes.io/last-applied-configuration` 어노테이션이활성 구성
에 기록된 것을 보여주며, 그것은 구성 파일과 일치한다.

```yaml
kind: Deployment
metadata:
  annotations:
    # ...
    # This is the json representation of simple_deployment.yaml
    # It was written by kubectl apply when the object was created
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment",
      "metadata":{"annotations":{},"name":"nginx-deployment","namespace":"default"},
      "spec":{"minReadySeconds":5,"selector":{"matchLabels":{"app":nginx}},"template":{"metadata":{"labels":{"app":"nginx"}},
      "spec":{"containers":[{"image":"nginx:1.14.2","name":"nginx",
      "ports":[{"containerPort":80}]}]}}}}
  # ...
spec:
  # ...
  minReadySeconds: 5
  selector:
    matchLabels:
      # ...
      app: nginx
  template:
    metadata:
      # ...
      labels:
        app: nginx
    spec:
      containers:
        - image: nginx:1.14.2
          # ...
          name: nginx
          ports:
            - containerPort: 80
          # ...
      # ...
    # ...
  # ...
```

## 오브젝트 업데이트 방법

또한 오브젝트가 기존에 존재하더라도 디렉터리 내 정의된 모든 오브젝트를 업데이트
하기 위해 `kubectl apply`를사용할 수 있다. 이러한 접근방식은 다음을 수행할 수 있
게 해준다.

1. 활성 구성 내 구성 파일에 나타나는 필드 설정
2. 활성 구성 내 구성 파일로부터 제거된 필드 정리

```shell
kubectl diff -f <디렉터리>/
kubectl apply -f <디렉터리>/
```

{{< note >}} 재귀적으로 디렉터리를 처리하기 위해서 `-R`플래그를 추가한다.
{{< /note >}}

다음은 구성 파일의 예시이다.

{{< codenew file="application/simple_deployment.yaml" >}}

`kubectl apply`를 사용하여 오브젝트를 생성한다.

```shell
kubectl apply -f https://k8s.io/examples/application/simple_deployment.yaml
```

{{< note >}} 설명을 위해, 앞선 명령은 디렉터리 대신하나의 구성 파일을 참조한다.
{{< /note >}}

`kubectl get`을 사용하여 활성 구성을 출력한다.

```shell
kubectl get -f https://k8s.io/examples/application/simple_deployment.yaml -o yaml
```

출력은 `kubectl.kubernetes.io/last-applied-configuration` 어노테이션이활성 구성
에 기록된 것을 보여주며, 그것은 구성 파일과 일치한다.

```yaml
kind: Deployment
metadata:
  annotations:
    # ...
    # This is the json representation of simple_deployment.yaml
    # It was written by kubectl apply when the object was created
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment",
      "metadata":{"annotations":{},"name":"nginx-deployment","namespace":"default"},
      "spec":{"minReadySeconds":5,"selector":{"matchLabels":{"app":nginx}},"template":{"metadata":{"labels":{"app":"nginx"}},
      "spec":{"containers":[{"image":"nginx:1.14.2","name":"nginx",
      "ports":[{"containerPort":80}]}]}}}}
  # ...
spec:
  # ...
  minReadySeconds: 5
  selector:
    matchLabels:
      # ...
      app: nginx
  template:
    metadata:
      # ...
      labels:
        app: nginx
    spec:
      containers:
        - image: nginx:1.14.2
          # ...
          name: nginx
          ports:
            - containerPort: 80
          # ...
      # ...
    # ...
  # ...
```

`kubectl scale`을 사용하여 활성 구성 내 `replicas` 필드를 직접 업데이트한다. 이
는 `kubectl apply`를 사용하지 않는다.

```shell
kubectl scale deployment/nginx-deployment --replicas=2
```

`kubectl get`을 사용하여 활성 구성을 출력한다.

```shell
kubectl get -f https://k8s.io/examples/application/simple_deployment.yaml -o yaml
```

출력은 `replicas` 필드가 2로 설정된 것을 보여주며, `last-applied-configuration`
어노테이션은 `replicas` 필드를 포함하지 않는다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    # ...
    # note that the annotation does not contain replicas
    # because it was not updated through apply
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment",
      "metadata":{"annotations":{},"name":"nginx-deployment","namespace":"default"},
      "spec":{"minReadySeconds":5,"selector":{"matchLabels":{"app":nginx}},"template":{"metadata":{"labels":{"app":"nginx"}},
      "spec":{"containers":[{"image":"nginx:1.14.2","name":"nginx",
      "ports":[{"containerPort":80}]}]}}}}
  # ...
spec:
  replicas: 2 # written by scale
  # ...
  minReadySeconds: 5
  selector:
    matchLabels:
      # ...
      app: nginx
  template:
    metadata:
      # ...
      labels:
        app: nginx
    spec:
      containers:
        - image: nginx:1.14.2
          # ...
          name: nginx
          ports:
            - containerPort: 80
      # ...
```

`nginx:1.14.2`에서 `nginx:1.16.1`로 이미지를 변경하기 위해
`simple_deployment.yaml` 구성 파일을 업데이트 하고, `minReadySeconds` 필드를 삭
제한다.

{{< codenew file="application/update_deployment.yaml" >}}

구성 파일에 이루어진 변경사항을 적용한다.

```shell
kubectl diff -f https://k8s.io/examples/application/update_deployment.yaml
kubectl apply -f https://k8s.io/examples/application/update_deployment.yaml
```

`kubectl get`을 사용하여 활성 구성을 출력한다.

```shell
kubectl get -f https://k8s.io/examples/application/simple_deployment.yaml -o yaml
```

출력은 활성 구성에 다음의 변경사항을 보여준다.

- `replicas` 필드는 `kubectl scale`에 의해 설정된 값 2를 유지한다.
  이는 구성 파일에서 생략되었기 때문에 가능하다.
- `image` 필드는 `nginx:1.14.2`에서 `nginx:1.16.1`로 업데이트되었다.
- `last-applied-configuration` 어노테이션은 새로운 이미지로 업데이트되었다.
- `minReadySeconds` 필드는 지워졌다.
- `last-applied-configuration` 어노테이션은 더 이상 `minReadySeconds` 필드를 포
  함하지 않는다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    # ...
    # The annotation contains the updated image to nginx 1.11.9,
    # but does not contain the updated replicas to 2
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment",
      "metadata":{"annotations":{},"name":"nginx-deployment","namespace":"default"},
      "spec":{"selector":{"matchLabels":{"app":nginx}},"template":{"metadata":{"labels":{"app":"nginx"}},
      "spec":{"containers":[{"image":"nginx:1.16.1","name":"nginx",
      "ports":[{"containerPort":80}]}]}}}}
    # ...
spec:
  replicas: 2 # Set by `kubectl scale`.  Ignored by `kubectl apply`.
  # minReadySeconds cleared by `kubectl apply`
  # ...
  selector:
    matchLabels:
      # ...
      app: nginx
  template:
    metadata:
      # ...
      labels:
        app: nginx
    spec:
      containers:
        - image: nginx:1.16.1 # Set by `kubectl apply`
          # ...
          name: nginx
          ports:
            - containerPort: 80
          # ...
      # ...
    # ...
  # ...
```

{{< warning >}} 명령형 오브젝트 구성 커맨드 `create`와 `replace`와 함께
`kubectl apply`를혼합하는 것은 지원하지 않는다. 이는 `kubectl apply`가 업데이트
사항을 계산하는데 사용하는 `kubectl.kubernetes.io/last-applied-configuration`을
`create`와 `replace`가유지하지 하지 않기 때문이다. {{< /warning >}}

## 오브젝트 삭제 방법

`kubectl apply`에 의해 관리되는 오브젝트를 삭제하는데 2가지 접근 방법이 있다.

### 권장 방법: `kubectl delete -f <파일명>`

명령형 커맨드를 사용하여 오브젝트를 수동으로 삭제하는 것이 권장되는 방식인데, 무
엇이 삭제되는지에 대해 더 명확하게 나타내므로 사용자가 의도하지 않게무언가를 삭
제할 가능성이 작아지기 때문이다.

```shell
kubectl delete -f <파일명>
```

### 대안: `kubectl apply -f <디렉터리/> --prune -l your=레이블`

무엇을 하는지 파악하는 경우에만 이를 사용한다.

{{< warning >}} `kubectl apply --prune`은 알파 상태이며, 후속 릴리스에서는하위
호환되지 않는 변경 사항이 도입될 수 있다. {{< /warning >}}

{{< warning >}} 이 명령을 사용할 때는 의도하지 않게 오브젝트를 삭제하지 않도록주
의해야만 한다. {{< /warning >}}

`kubectl delete`에 대한 대안으로, 디렉터리로부터 구성 파일이 삭제된 후에 삭제�
오브젝트를 식별하기 위해 `kubectl apply`를 사용할 수 있다. `--prune`을 사용하여
적용하면 일련의 레이블의 집합과 일치하는모든 오브젝트에 대해API 서버에 쿼리하고,
반환된 활성 오브젝트구성을 오브젝트 구성 파일에 일치시키려고 시도한다. 오브젝트
가 쿼리에 일치하고, 해당 디렉터리 내 구성 파일이 없�
`last-applied-configuration`어노테이션이 있는 경우, 삭제된다.

{{< comment >}} TODO(pwittrock): We need to change the behavior to prevent the
user from running apply on subdirectories unintentionally. {{< /comment >}}

```shell
kubectl apply -f <디렉터리/> --prune -l <레이블>
```

{{< warning >}} prune을 사용하여 적용하는 것은 오브젝트 구성 파일을포함하는 루트
디렉터리에 대해서만 실행해야 한다. 하위 디렉터리에 대해 실행하게 되면,
`-l <레이블>`로 지정된 레이블 셀렉터에 의해 반환되고 하위 디렉터리에 나타나지 않
는 경우, 오브젝트가 의도하지 않게 삭제될 수 있다. {{< /warning >}}

## 오브젝트 확인 방법

활성 오브젝트의 구성을 확인하기 위해 `-o yaml`과 함께 `kubectl get`을 사용할 수
있다.

```shell
kubectl get -f <파일명|url> -o yaml
```

## 어떻게 apply가 차이를 계산하고 변경을 병합하는가

{{< caution >}} _patch_ 는 전체 오브젝트 대신 오브젝트의 특정 필드 범위의 오퍼레
이션을 업데이트한다. 이는 먼저 오브젝트를 읽지 않고도 오브젝트의 특정 필드 집합
만을업데이트할 수 있도록 해준다. {{< /caution >}}

`kubectl apply`가 하나의 오브젝트에 대한 활성 구성을 업데이트할 때, API 서버에
패치 요청을 보냄으로써 그것을 수행한다. 그 패치는 활성 오브젝트 구성의 특정 필드
에 대한 범위의업데이트로 한정한다. `kubectl apply` 커맨드는구성 파일, 활성 구성,
그리고 활성 구성에 저장된 `last-applied-configuration`어노테이션을 사용하여 이
패치 요청을 계산한다.

### 패치 계산 병합

`kubectl apply` 명령은 `kubectl.kubernetes.io/last-applied-configuration` 어노테
이션에 구성 파일의 내용을 기록한다. 이것은 구성 파일로부터 제거되었고 활성 구성
으로부터 지워질 필요가 있는필드를 확인하는 데 사용된다. 다음은 어떤 필드가 삭제
또는 설정돼야 하는지계산하기 위해 사용되는 단계이다.

1. 삭제할 필드를 계산한다. 이것은 `last-applied-configuration` 내 존재하고 구성
   파일로부터 유실된 필드이다.
2. 추가 또는 설정되어야 할 필드를 계산한다. 이것은 활성 구성과 불일치하는 값을
   가지는 구성 파일 내 존재하는 필드이다.

다음은 예시이다. 디플로이먼트 오브젝트에 대한 구성 파일이라고 가정한다.

{{< codenew file="application/update_deployment.yaml" >}}

또한, 이것은 동일한 디플로이먼트 오브젝트에 대한 활성 구성이라고 가정한다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    # ...
    # note that the annotation does not contain replicas
    # because it was not updated through apply
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment",
      "metadata":{"annotations":{},"name":"nginx-deployment","namespace":"default"},
      "spec":{"minReadySeconds":5,"selector":{"matchLabels":{"app":nginx}},"template":{"metadata":{"labels":{"app":"nginx"}},
      "spec":{"containers":[{"image":"nginx:1.14.2","name":"nginx",
      "ports":[{"containerPort":80}]}]}}}}
  # ...
spec:
  replicas: 2 # written by scale
  # ...
  minReadySeconds: 5
  selector:
    matchLabels:
      # ...
      app: nginx
  template:
    metadata:
      # ...
      labels:
        app: nginx
    spec:
      containers:
        - image: nginx:1.14.2
          # ...
          name: nginx
          ports:
            - containerPort: 80
      # ...
```

다음은 `kubectl apply`에 의해 수행될 병합 계산이다.

1. `last-applied-configuration`으로부터 값을 읽어구성 파일의 값과 비교하여 삭제
   할 필드를계산한다. `last-applied-configuration`에 보이는 것과는 무관하게로컬
   의 오브젝트 구성 파일 내 null이라고 명시적으로 설정된 필드를 지운다. 이 예시
   에서, `minReadySeconds`은 `last-applied-configuration` 어노테이션 내 나타나지
   만, 구성 파일 내에는 보여지지 않는다. **조치:** 활성 구성으로부터
   `minReadySeconds`을 지운다.
2. 구성 파일로부터 값을 읽어 활성 구성 내 값과비교하여 설정할 필드를 계산한다.
   이 예시에서, 구성 파일 내 `image` 값은 활성 구성 내 값과 불일치한다. **조치
   :** 활성 구성 내 `image` 값을 설정한다.
3. 구성 파일의 값과 일치시키기 위해 `last-applied-configuration` 어노테이션을 설
   정한다.
4. 1, 2, 3으로부터의 결과를 API 서버에 단일 패치 요청으로 병합한다.

다음은 병합의 결과인 활성 구성이다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    # ...
    # The annotation contains the updated image to nginx 1.11.9,
    # but does not contain the updated replicas to 2
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment",
      "metadata":{"annotations":{},"name":"nginx-deployment","namespace":"default"},
      "spec":{"selector":{"matchLabels":{"app":nginx}},"template":{"metadata":{"labels":{"app":"nginx"}},
      "spec":{"containers":[{"image":"nginx:1.16.1","name":"nginx",
      "ports":[{"containerPort":80}]}]}}}}
    # ...
spec:
  selector:
    matchLabels:
      # ...
      app: nginx
  replicas: 2 # Set by `kubectl scale`.  Ignored by `kubectl apply`.
  # minReadySeconds cleared by `kubectl apply`
  # ...
  template:
    metadata:
      # ...
      labels:
        app: nginx
    spec:
      containers:
        - image: nginx:1.16.1 # Set by `kubectl apply`
          # ...
          name: nginx
          ports:
            - containerPort: 80
          # ...
      # ...
    # ...
  # ...
```

### 어떻게 상이한 필드 타입이 병합되는가

구성 파일 내 특정 필드가 필드의 타입에 따라어떻게 활성 구성과 함께 병합되는가.
여러 가지 필드 타입이 있다.

- _기본(primitives)_: 문자열, 숫자 또는 불리언 타입의 필드. 예를 들어, `image`와
  `replicas`는 기본 필드다. **조치:** 교체.

- _맵_, 또한 _오브젝트_ 라 칭함: 맵 타입 또는 서브필드를 포함하는 복합 타입의 필
  드. 예를 들어, `레이블`, `어노테이션`,`스펙` 및 `메타데이터`는 모두 맵이다.
  **조치:** 구성요소 또는 서브필드 병합.

- _리스트_: 기본타입 또는 맵이 될 수 있는 아이템의 리스트를 포함하는 필드. 예를
  들어, `컨테이너`, `포트`, 그리고 `args`는 리스트다. **조치:** 다양함.

`kubectl apply`가 맵 또는 리스트 필드를 업데이트하는 경우, 일반적으로 전체 필드
를 교체하는 대신, 개별 부 구성요소를 업데이트한다, 예를 들어, 디플로이먼트에 대
한 `spec`을 병합할 경우, 전체 `spec`이교체되지 않는다. 대신 `replicas`와 같은
`spec`의 서브필드가비교되고 병합된다.

### 기본 필드에 대한 변경사항 병합하기

기본 필드는 교체되거나 지워진다.

{{< note >}} `-` 는 값이 사용되지 않기 때문에 "해당 없음"으로 사용된다.
{{< /note >}}

| Field in object configuration file | Field in live object configuration | Field in last-applied-configuration | Action                      |
| ---------------------------------- | ---------------------------------- | ----------------------------------- | --------------------------- |
| Yes                                | Yes                                | -                                   | 구성 파일 값 활성으로 설정. |
| Yes                                | No                                 | -                                   | 활성을 로컬 구성으로 설정.  |
| No                                 | -                                  | Yes                                 | 활성 구성으로부터 지움.     |
| No                                 | -                                  | No                                  | 아무것도 안함. 활성값 유지. |

### 맵 필드에 변경사항 병합하기

맵을 요청하는 필드는 서브필드의 각각 또는 맵의 구성요소를 비교함으로써 병합된다.

{{< note >}} `-` 는 값이 사용되지 않기 때문에 "해당 없음"으로 사용된다.
{{< /note >}}

| Key in object configuration file | Key in live object configuration | Field in last-applied-configuration | Action                      |
| -------------------------------- | -------------------------------- | ----------------------------------- | --------------------------- |
| Yes                              | Yes                              | -                                   | 서브필드 값 비교.           |
| Yes                              | No                               | -                                   | 활성을 로컬 구성으로 설정.  |
| No                               | -                                | Yes                                 | 활성 구성으로부터 삭제.     |
| No                               | -                                | No                                  | 아무것도 안함. 활성값 유지. |

### 타입 리스트의 필드에 대한 변경사항 병합하기

리스트에 대한 변경사항을 병합하는 것은 세 가지 전략 중 하나를 사용한다.

- 구성요소가 모두 기본형인 경우 리스트를 교체한다.
- 복합 구성요소의 리스트에서 개별 구성요소를 병합한다.
- 기초 구성요소의 리스트를 병합한다.

전략에 대한 선택은 필드별로 이루어진다.

#### 구성요소가 모두 기본형인 경우 리스트 교체

기초 필드와 동일한 리스트로 취급한다. 전체 리스트를 교체 또는 삭제한다. 이것은
순서를 유지한다.

**예시:** 파드 내 컨테이너의 `args` 필드를 업데이트하기 위해 `kubectl apply`를
사용한다. 이것은 활성 구성 내 `args`의 값을 구성 파일 내 값으로 설정한다. 활성
구성에 추가했던 이전의 모든 `args`구성요소들은 유실된다. 구성 파일 내 정의한
`args` 구성요소의 순서는활성 구성 내 유지된다.

```yaml
# last-applied-configuration value
    args: ["a", "b"]

# configuration file value
    args: ["a", "c"]

# live configuration
    args: ["a", "b", "d"]

# result after merge
    args: ["a", "c"]
```

**설명:** 병합은 새로운 리스트 값으로 구성 파일 값을 사용했다.

#### 복합 구성요소 리스트에 대한 개별 구성요소 병합

리스트를 맵으로 취급하고 각 구성요소의 특정 필드를 키로 취급한다. 개별 구성요소
를 추가, 삭제, 또는 업데이트 한다. 이것은 순서를 보존하지 않는다.

이 병합 전략은 각 필드에 `patchMergeKey`라 칭하는 특별한 태그를 사용한다.
`patchMergeKey`는 쿠버네티스 소스 코드:
[types.go](https://github.com/kubernetes/api/blob/d04500c8c3dda9c980b668c57abc2ca61efcf5c4/core/v1/types.go#L2747)
의 각 필드에 대해 정의한다. 맵 리스트를 병합할 때, 주어진 구성요소에 대한
`patchMergeKey`로지정한 필드는 해당 구성요소에 대한 맵키와 같이 사용된다.

**예시:** `kubectl apply`를 사용하여 PodSpec에 대한 `containers`필드를 업데이트
한다.
이렇게 하면 각 구성요소가 `name`별로 키로 되어 있는 맵인 것처럼 리스트를 병합한
다.

```yaml
# last-applied-configuration value
    containers:
    - name: nginx
      image: nginx:1.10
    - name: nginx-helper-a # key: nginx-helper-a; will be deleted in result
      image: helper:1.3
    - name: nginx-helper-b # key: nginx-helper-b; will be retained
      image: helper:1.3

# configuration file value
    containers:
    - name: nginx
      image: nginx:1.10
    - name: nginx-helper-b
      image: helper:1.3
    - name: nginx-helper-c # key: nginx-helper-c; will be added in result
      image: helper:1.3

# live configuration
    containers:
    - name: nginx
      image: nginx:1.10
    - name: nginx-helper-a
      image: helper:1.3
    - name: nginx-helper-b
      image: helper:1.3
      args: ["run"] # Field will be retained
    - name: nginx-helper-d # key: nginx-helper-d; will be retained
      image: helper:1.3

# result after merge
    containers:
    - name: nginx
      image: nginx:1.10
      # Element nginx-helper-a was deleted
    - name: nginx-helper-b
      image: helper:1.3
      args: ["run"] # Field was retained
    - name: nginx-helper-c # Element was added
      image: helper:1.3
    - name: nginx-helper-d # Element was ignored
      image: helper:1.3
```

**설명:**

- 구성 파일에 "nginx-helper-a"라는 이름을 가진 컨테이너가 나타나지 않았기 때문에
  "nginx-helper-a"라는 컨테이너는 삭제되었다.
- "nginx-helper-b"라는 컨테이너는 활성 구성에 `args`에
  대한 변경사항을 유지했다. `kubectl apply`는
  필드 값이 다름에도 불구하고(구성 파일에 `args`가 없음) 활성 구성에
  "nginx-helper-b"가 구성 파일과 동일한 "nginx-helper-b"임을 식별할 수 있었다.
  이것은 `patchMergeKey` 필드 값(이름)이 둘 다 같았기 때문이다..
- "nginx-helper-c"라는 이름의 컨테이너가 활성 구성에 나타나지않았지만, 구성 파일
  에 그 이름을 가진 컨테이너가 나타났기 때문에추가되었다.
- last-applied-configuration에 그 이름을 가진 구성요소가 없었기 때문에
  "nginx-helper-d"라는 이름의 컨테이너는 유지되었다.

#### 기초 구성요소 리스트 병합

쿠버네티스 1.5로부터 기초 구성요소 병합하기는 지원되지 않는다.

{{< note >}} 주어진 필드에 대해 위 전략 중 어떤 것을 선택할지에 대해서는
[types.go](https://github.com/kubernetes/api/blob/d04500c8c3dda9c980b668c57abc2ca61efcf5c4/core/v1/types.go#L2748)의
`patchStrategy` 태그에 의해 제어된다. 타입 필드에 대해 `patchStrategy`가 지정되
지 않으면, 리스트는 대체된다. {{< /note >}}

{{< comment >}} TODO(pwittrock): Uncomment this for 1.6

- Treat the list as a set of primitives. Replace or delete individual elements.
  Does not preserve ordering. Does not preserve duplicates.

**Example:** Using apply to update the `finalizers` field of ObjectMeta keeps
elements added to the live configuration. Ordering of finalizers is lost.
{{< /comment >}}

## 기본 필드값

오브젝트가 생성될 때 값이 지정되지 않는 경우, API 서버는 활성 구성 내특정 필드를
기본값으로 설정한다.

다음은 디플로이먼트에 대한 구성 파일이다. 파일에는 `strategy`가 지정되지 않았다.

{{< codenew file="application/simple_deployment.yaml" >}}

`kubectl apply`를 사용하여 오브젝트를 생성한다.

```shell
kubectl apply -f https://k8s.io/examples/application/simple_deployment.yaml
```

`kubectl get`을 사용하여 활성 구성을 출력한다.

```shell
kubectl get -f https://k8s.io/examples/application/simple_deployment.yaml -o yaml
```

출력은 API 서버가 활성 구성 내 여러 필드를 기본값으로 설정한 것을 보여준다. 이
필드들은 구성 파일에 지정되지 않았다.

```yaml
apiVersion: apps/v1
kind: Deployment
# ...
spec:
  selector:
    matchLabels:
      app: nginx
  minReadySeconds: 5
  replicas: 1 # defaulted by apiserver
  strategy:
    rollingUpdate: # defaulted by apiserver - derived from strategy.type
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate # defaulted by apiserver
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
        - image: nginx:1.14.2
          imagePullPolicy: IfNotPresent # defaulted by apiserver
          name: nginx
          ports:
            - containerPort: 80
              protocol: TCP # defaulted by apiserver
          resources: {} # defaulted by apiserver
          terminationMessagePath: /dev/termination-log # defaulted by apiserver
      dnsPolicy: ClusterFirst # defaulted by apiserver
      restartPolicy: Always # defaulted by apiserver
      securityContext: {} # defaulted by apiserver
      terminationGracePeriodSeconds: 30 # defaulted by apiserver
# ...
```

패치 요청에서, 패치 요청의 부분으로서 명시적으로 지워지지 않은 경우기본 처리된
필드는 다시 기본으로 설정되지 않는다. 이것은 다른 필드에 대한 값에 따라 기본 처
리된 필드에 대해예상하지 못한 동작을 유발할 수 있다. 다른 필드가 나중에 변경되면
, 그로부터 기본 처리된 것이 명시적으로 지워지지 않은 한업데이트되지 않을 것이다.

이러한 사유로, 의도한 값이 서버의 기본값과 일치하더라도, 서버에 의해 기본 처리된
특정 필드는 구성 파일 내명시적으로 정의할 것을 권고한다. 이렇게 하면서버에 의해
다시 기본 처리되지 않게 될 충돌하는 값을 보다 쉽게인식할 수 있도록 해준다.

**Example:**

```yaml
# last-applied-configuration
spec:
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

# configuration file
spec:
  strategy:
    type: Recreate # updated value
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

# live configuration
spec:
  strategy:
    type: RollingUpdate # defaulted value
    rollingUpdate: # defaulted value derived from type
      maxSurge : 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

# result after merge - ERROR!
spec:
  strategy:
    type: Recreate # updated value: incompatible with rollingUpdate
    rollingUpdate: # defaulted value: incompatible with "type: Recreate"
      maxSurge : 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

**설명:**

1. 사용자가 `strategy.type`을 정의하지 않고 디플로이먼트를 생성한다.
2. 서버는 `strategy.type`을 `RollingUpdate`로 기본 설정하�
   `strategy.rollingUpdate`값을 기본 값으로 처리한다.
3. 사용자가 `strategy.type`를 `Recreate`로 변경한다. 서버에서 해당 값이 삭제�
   거라 예상하지만 `strategy.rollingUpdate`값은 기본값으로 남아 있다.
   `strategy.rollingUpdate`값이 처음에 구성 파일에서 지정되었다면, 이것을 삭제해
   야 한다는 것이 더 분명했을 것이다.
4. `strategy.rollingUpdate`가 지워지지 않았기 때문에 적용은 실패한다.
   `strategy.rollingupdate` 필드는 `Recreate`의 `strategy.type`으로 정의될 수 없
   다.

권고: 이들 필드는 오브젝트 구성 파일 내 명시적으로 정의돼야 한다.

- 디플로이먼트, 스테이트풀셋, 잡, 데몬셋, 레플리카셋 및 레플리케이션컨트롤러와
  같은워크로드에 대한 셀렉터와 파드템플릿 레이블
- 디플로이먼트 롤아웃 전략

### 서버 기본 필드 또는 다른 작성자에 의해 설정된 필드 지우는 방법

구성 파일 내 나타나지 않는 필드는 그 값을 `null`로 설정하고 나서 구성 파일을 적
용함으로써 지워질 수 있다. 서버가 기본 값을 할당했던 필드에 대해서, 이는 다시 기
본 값을할당하도록 한다.

## 구성 파일과 직접 명령형 작성자 간의 필드 소유권을 변경시키는 방법

개별 오브젝트 필드를 변경시키는 데 사용해야 하는 유일한 방법은 다음과 같다.

- `kubectl apply`를 사용한다.
- 구성 파일을 수정하지 않고 활성 구성을 직접 작성한다. 예를 들어,
  `kubectl scale`을 사용한다.

### 직접 명령형 작성자에서 구성 파일로 소유자 변경하기

구성 파일에 필드를 추가한다. 해당 필드의 경우 `kubectl apply`를 거치지 않는 활성
구성에 대해 직접 업데이트를 적용하지 않는다.

### 구성 파일에서 직접 명령형 작성자로 소유자 변경하기

쿠버네티스 1.5로부터 구성 파일에서 명령형 작성자로 소유권을 변경하는데수동 단계
필요하다.

- 구성 파일에서 필드를 제거한다.
- 활성 오브젝트 상의 `kubectl.kubernetes.io/last-applied-configuration` 어노테이
  션에서 필드를 제거한다.

## 관리 방법 변경하기

쿠버네티스 오브젝트는 한 번에 오직 하나의 방법을 사용하여 관리돼야 한다. 하나의
방법에서 다른 방법으로 전환하는 것은 가능하나, 수동 프로세스이다.

{{< note >}} 선언형 관리와 함께 명령형 삭제를 사용하는 것은 괜찮다.
{{< /note >}}

{{< comment >}} TODO(pwittrock): We need to make using imperative commands with
declarative object configuration work so that it doesn't write the fields to the
annotation, and instead. Then add this bullet point.

- using imperative commands with declarative configuration to manage where each
  manages different fields. {{< /comment >}}

### 명령형 커맨드 관리에서 오브젝트 구성으로 이전하기

명령형 커맨드 관리에서 오브젝트 구성으로 이전하는 것은여러 수동 단계를 포함한다.

1. 활성 오브젝트를 로컬 구성 파일로 내보낸다.

   ```shell
   kubectl get <종류>/<이름> -o yaml > <종류>_<이름>.yaml
   ```

1. 구성 파일에서 수동으로 `status` 필드를 제거한다.

   {{< note >}} `kubectl apply` 구성 파일에 존재한다고 하더라도 상태 필드가 업데
   이트되지 않기 때문에, 이 단계는 선택적이다. {{< /note >}}

1. 오브젝트의 `kubectl.kubernetes.io/last-applied-configuration` 어노테이션을 설
   정한다.

   ```shell
   kubectl replace --save-config -f <종류>_<이름>.yaml
   ```

1. 오직 오브젝트를 관리하기 위해 `kubectl apply`를 사용하도록 프로세스를 변경한
   다.

{{< comment >}} TODO(pwittrock): Why doesn't export remove the status field?
Seems like it should. {{< /comment >}}

### 명령형 오브젝트 구성에서 선언형 오브젝트 구성으로 이전하기

1. 오브젝트의 `kubectl.kubernetes.io/last-applied-configuration` 어노테이션을 설
   정한다.

   ```shell
   kubectl replace --save-config -f <종류>_<이름>.yaml
   ```

1. 오직 오브젝트를 관리하기 위해 `kubectl apply`를 사용하도록 프로세스를 변경한
   다.

## 컨트롤러 셀렉터와 파드템플릿 레이블 정의하기

{{< warning >}} 컨트롤러에서 셀렉터를 업데이트하는 것은 추천되지 않는다.
{{< /warning >}}

권고되는 접근 방법은 다른 의미론적 의미를 가지지 않고 컨트롤러에 의해서만 사용되
는단일, 불변의 파드템플릿 레이블을 정의하는 것이다.

**예시:**

```yaml
selector:
  matchLabels:
    controller-selector: "apps/v1/deployment/nginx"
template:
  metadata:
    labels:
      controller-selector: "apps/v1/deployment/nginx"
```

{{% capture whatsnext %}}

- [명령형 커맨드 사용하여 쿠버네티스 오브젝트 관리하기](/ko/docs/tasks/manage-kubernetes-objects/imperative-command/)
- [구성 파일 사용하여 쿠버네티스 오브젝트 관리하기](/ko/docs/tasks/manage-kubernetes-objects/imperative-config/)
- [Kubectl 명령어 참조](/docs/reference/generated/kubectl/kubectl/)
- [쿠버네티스 API
  참조](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/)
  {{% /capture %}}
