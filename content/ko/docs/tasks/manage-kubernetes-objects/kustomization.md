---
title: Kustomize를 이용한 쿠버네티스 오브젝트의 선언형 관리
content_template: templates/task
weight: 20
---

{{% capture overview %}}

[Kustomize](https://github.com/kubernetes-sigs/kustomize)는
[kustomization 파일](https://github.com/kubernetes-sigs/kustomize/blob/master/docs/glossary.md#kustomization)을
통해 쿠버네티스 오브젝트를 사용자가 원하는 대로 변경하는(customize) 독립형 도구
이다.

1.14 이후로, kubectl도 kustomization 파일을 사용한 쿠버네티스 오브젝트의 관리를
지원한다. kustomization 파일을 포함하는 디렉터리 내의 리소스를 보려면 다음 명령
어를 실행한다.

```shell
kubectl kustomize <kustomization_directory>
```

이 리소스를 적용하려면 `kubectl apply`를 `--kustomize` 또는 `-k` 플래그와 함께
실행한다.

```shell
kubectl apply -k <kustomization_directory>
```

{{% /capture %}}

{{% capture prerequisites %}}

[`kubectl`](/docs/tasks/tools/install-kubectl/)을 설치한다.

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

{{% /capture %}}

{{% capture steps %}}

## Kustomize 개요

Kustomize는 쿠버네티스 구성을 사용자 정의화하는 도구이다. 이는 애플리케이션 구성
파일을 관리하기 위해 다음 기능들을 가진다.

- 다른 소스에서 리소스 생성
- 리소스에 대한 교차 편집 필드 설정
- 리소스 집합을 구성하고 사용자 정의

### 리소스 생성

컨피그 맵과 시크릿은 파드같은 다른 쿠버네티스 오브젝트에서 사용되는 설정이나 민
감한 데이터를 가지고 있다. 컨피그 맵이나 시크릿의 실질적인 소스는 일반적으로
`.properties` 파일이나 ssh key 파일같이 다른 곳에서 가져온다. Kustomize는 시크릿
과 컨피그 맵을 파일이나 문자열에서 생성하는 `secretGenerator`와
`configMapGenerator`를 가지고 있다.

#### configMapGenerator

파일에서 컨피그 맵을 생성하려면 `configMapGenerator` 내의 `files` 리스트에 항목
을 추가한다. 다음은 하나의 파일 콘텐츠에서 데이터 항목으로 컨피그 맵을 생성하는
예제이다.

```shell
# application.properties 파일을 생성
cat <<EOF >application.properties
FOO=Bar
EOF

cat <<EOF >./kustomization.yaml
configMapGenerator:
- name: example-configmap-1
  files:
  - application.properties
EOF
```

생성된 컨피그 맵은 다음 명령어를 통해 확인할 수 있다.

```shell
kubectl kustomize ./
```

생성된 컨피그 맵은 다음과 같다.

```yaml
apiVersion: v1
data:
  application.properties: |
    FOO=Bar
kind: ConfigMap
metadata:
  name: example-configmap-1-8mbdf7882g
```

컨피그 맵은 문자로된 키-값 쌍들로도 생성할 수 있다. 문자로된 키-값 쌍에서 컨피그
맵을 생성하려면, configMapGenerator 내의 `literals` 리스트에 항목을 추가한다. 다
음은 키-값 쌍을 데이터 항목으로 받는 컨피그 맵을 생성하는 예제이다.

```shell
cat <<EOF >./kustomization.yaml
configMapGenerator:
- name: example-configmap-2
  literals:
  - FOO=Bar
EOF
```

생성된 컨피그 맵은 다음 명령어로 확인할 수 있다.

```shell
kubectl kustomize ./
```

생성된 컨피그 맵은 다음과 같다.

```yaml
apiVersion: v1
data:
  FOO: Bar
kind: ConfigMap
metadata:
  name: example-configmap-2-g2hdhfc6tk
```

#### secretGenerator

파일 또는 문자로된 키-값 쌍들로 시크릿을 생성할 수 있다. 파일에서 시크릿을 생성
하려면 `secretGenerator` 내의 `files` 리스트에 항목을 추가한다. 다음은 파일을 데
이터 항목으로 받는 시크릿을 생성하는 예제이다.

```shell
# password.txt 파일을 생성
cat <<EOF >./password.txt
username=admin
password=secret
EOF

cat <<EOF >./kustomization.yaml
secretGenerator:
- name: example-secret-1
  files:
  - password.txt
EOF
```

생성된 시크릿은 다음과 같다.

```yaml
apiVersion: v1
data:
  password.txt: dXNlcm5hbWU9YWRtaW4KcGFzc3dvcmQ9c2VjcmV0Cg==
kind: Secret
metadata:
  name: example-secret-1-t2kt65hgtb
type: Opaque
```

문자로된 키-값 쌍으로 시크릿을 생성하려면, `secretGenerator` 내의 `literals` 리
스트에 항목을 추가한다. 다음은 키-값 쌍을 데이터 항목으로 받는 시크릿을 생성하는
예제이다.

```shell
cat <<EOF >./kustomization.yaml
secretGenerator:
- name: example-secret-2
  literals:
  - username=admin
  - password=secret
EOF
```

생성된 시크릿은 다음과 같다.

```yaml
apiVersion: v1
data:
  password: c2VjcmV0
  username: YWRtaW4=
kind: Secret
metadata:
  name: example-secret-2-t52t6g96d8
type: Opaque
```

#### generatorOptions

생성된 컨피그 맵과 시크릿은 콘텐츠를 해쉬화하여 덧붙이는 접미사를 가진다. 이는
콘텐츠가 변경될 때 새로운 컨피그 맵 이나 시크릿이 생성되는 것을 보장한다. 접미사
를 추가하는 동작을 비활성화하는 방법으로 `generatorOptions`를 사용할 수 있다. 그
밖에, 생성된 컨피그 맵과 시크릿에 교차 편집 옵션들을 지정해주는 것도 가능하다.

```shell
cat <<EOF >./kustomization.yaml
configMapGenerator:
- name: example-configmap-3
  literals:
  - FOO=Bar
generatorOptions:
  disableNameSuffixHash: true
  labels:
    type: generated
  annotations:
    note: generated
EOF
```

생성된 컨피그 맵을 보려면 `kubectl kustomize ./`를 실행한다.

```yaml
apiVersion: v1
data:
  FOO: Bar
kind: ConfigMap
metadata:
  annotations:
    note: generated
  labels:
    type: generated
  name: example-configmap-3
```

### 교차 편집 필드 설정

프로젝트 내 모든 쿠버네티스 리소스에 교차 편집 필드를 설정하는 것은 꽤나 일반적
이다. 교차 편집 필드를 설정하는 몇 가지 사용 사례는 다음과 같다.

- 모든 리소스에 동일한 네임스페이스를 설정
- 동일한 네임 접두사 또는 접미사를 추가
- 동일한 레이블들을 추가
- 동일한 어노테이션들을 추가

다음은 예제이다.

```shell
# deployment.yaml을 생성
cat <<EOF >./deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
EOF

cat <<EOF >./kustomization.yaml
namespace: my-namespace
namePrefix: dev-
nameSuffix: "-001"
commonLabels:
  app: bingo
commonAnnotations:
  oncallPager: 800-555-1212
resources:
- deployment.yaml
EOF
```

이 필드들이 디플로이먼트 리소스에 모두 설정되었는지 보려면
`kubectl kustomize ./`를 실행한다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    oncallPager: 800-555-1212
  labels:
    app: bingo
  name: dev-nginx-deployment-001
  namespace: my-namespace
spec:
  selector:
    matchLabels:
      app: bingo
  template:
    metadata:
      annotations:
        oncallPager: 800-555-1212
      labels:
        app: bingo
    spec:
      containers:
        - image: nginx
          name: nginx
```

### 리소스 구성과 사용자 정의

프로젝트 내 리소스의 집합을 구성하여 이들을 동일한 파일이나 디렉터리 내에서 관리
하는 것은 일반적이다. Kustomize는 서로 다른 파일들로 리소스를 구성하고 패치나 다
른 사용자 정의를 이들에 적용하는 것을 제공한다.

#### 구성

Kustomize는 서로 다른 리소스들의 구성을 지원한다. `kustomization.yaml` 파일 내
`resources` 필드는 구성 내에 포함하려는 리소스들의 리스트를 정의한다.
`resources` 리스트 내에 리소스의 구성 파일의 경로를 설정한다. 다음 예제는 디플로
이먼트와 서비스를 가지는 nginx 애플리케이션이다.

```shell
# deployment.yaml 파일 생성
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EOF

# service.yaml 파일 생성
cat <<EOF > service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: my-nginx
EOF

# 이들을 구성하는 kustomization.yaml 생성
cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml
- service.yaml
EOF
```

`kubectl kustomize ./`의 리소스는 디플로이먼트와 서비스 오브젝트 모두를 포함한다
.

#### 사용자 정의

리소스 위에 패치를 적용하는 것으로 서로 다른 사용자 정의들을 적용할 수 있다.
Kustomize는 `patchesStrategicMerge`와 `patchesJson6902`를 통해 서로 다른 패치 메
커니즘을 지원한다. `patchesStrategicMerge`는 파일 경로들의 리스트이다. 각각의 파
일은
[전략적 병합 패치](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-api-machinery/strategic-merge-patch.md)로
분석될 수 있어야 한다. 패치 내부의 네임은 반드시 이미 읽혀진 리소스 네임과 일치
해야 한다. 한 가지 일을 하는 작은 패치가 권장된다. 예를 들기 위해 디플로이먼트
레플리카 숫자를 증가시키는 하나의 패치와 메모리 상한을 설정하는 다른 패치를 생성
한다.

```shell
# deployment.yaml 파일 생성
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EOF

# increase_replicas.yaml 패치 생성
cat <<EOF > increase_replicas.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 3
EOF

# 다른 패치로 set_memory.yaml 생성
cat <<EOF > set_memory.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  template:
    spec:
      containers:
      - name: my-nginx
        resources:
        limits:
          memory: 512Mi
EOF

cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml
patchesStrategicMerge:
- increase_replicas.yaml
- set_memory.yaml
EOF
```

디플로이먼트를 보려면 `kubectl kustomize ./`를 실행한다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      run: my-nginx
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
        - image: nginx
          limits:
            memory: 512Mi
          name: my-nginx
          ports:
            - containerPort: 80
```

모든 리소스 또는 필드가 전략적 병합 패치를 지원하는 것은 아니다. 임의의 리소스
내 임의의 필드의 수정을 지원하기 위해, Kustomize는 `patchesJson6902`를 통한
[JSON 패치](https://tools.ietf.org/html/rfc6902) 적용을 제공한다. Json 패치의 정
확한 리소스를 찾기 위해, 해당 리소스의 group, version, kind, name이
`kustomization.yaml` 내에 명시될 필요가 있다. 예를 들면, `patchesJson6902`를 통
해 디플로이먼트 오브젝트의 레플리카 개수를 증가시킬 수 있다.

```shell
# deployment.yaml 파일 생성
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EOF

# json 패치 생성
cat <<EOF > patch.yaml
- op: replace
  path: /spec/replicas
  value: 3
EOF

# kustomization.yaml 생성
cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml

patchesJson6902:
- target:
    group: apps
    version: v1
    kind: Deployment
    name: my-nginx
  path: patch.yaml
EOF
```

`kubectl kustomize ./`를 실행하여 `replicas` 필드가 갱신되었는지 확인한다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      run: my-nginx
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
        - image: nginx
          name: my-nginx
          ports:
            - containerPort: 80
```

패치 기능에 추가로 Kustomize는 패치를 생성하지 않고 컨테이너 이미지를 사용자 정
의하거나 다른 오브젝트의 필드 값을 컨테이너에 주입하는 기능도 제공한다. 예를 들
어 `kustomization.yaml`의 `images` 필드에 신규 이미지를 지정하여 컨테이너에서 사
용되는 이미지를 변경할 수 있다.

```shell
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EOF

cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml
images:
- name: nginx
  newName: my.image.registry/nginx
  newTag: 1.4.0
EOF
```

사용된 이미지가 갱신되었는지 확인하려면 `kubectl kustomize ./`를 실행한다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      run: my-nginx
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
        - image: my.image.registry/nginx:1.4.0
          name: my-nginx
          ports:
            - containerPort: 80
```

가끔, 파드 내에서 실행되는 애플리케이션이 다른 오브젝트의 설정 값을 사용해야 �
수도 있다. 예를 들어, 디플로이먼트 오브젝트의 파드는 Env 또는 커맨드 인수로 해당
서비스 네임을 읽어야 한다고 하자. `kustomization.yaml` 파일에 `namePrefix` 또는
`nameSuffix`가 추가되면 서비스 네임이 변경될 수 있다. 커맨드 인수 내에 서비스 네
임을 하드 코딩하는 것을 권장하지 않는다. 이 용도에서 Kustomize는 `vars`를 통해
containers에 서비스 네임을 삽입할 수 있다.

```shell
# deployment.yaml 파일 생성
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        command: ["start", "--host", "\$(MY_SERVICE_NAME)"]
EOF

# service.yaml 파일 생성
cat <<EOF > service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: my-nginx
EOF

cat <<EOF >./kustomization.yaml
namePrefix: dev-
nameSuffix: "-001"

resources:
- deployment.yaml
- service.yaml

vars:
- name: MY_SERVICE_NAME
  objref:
    kind: Service
    name: my-nginx
    apiVersion: v1
EOF
```

`kubectl kustomize ./`를 실행하면 `dev-my-nginx-001`로 컨테이너에 삽입된 서비스
네임을 볼 수 있다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dev-my-nginx-001
spec:
  replicas: 2
  selector:
    matchLabels:
      run: my-nginx
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
        - command:
            - start
            - --host
            - dev-my-nginx-001
          image: nginx
          name: my-nginx
```

## Base와 Overlay

Kustomize는 **base**와 **overlay**의 개념을 가지고 있다. **base**는
`kustomization.yaml`과 함께 사용되는 디렉터리다. 이는 사용자 정의와 관련된 리소
스들의 집합을 포함한다. `kustomization.yaml`의 내부에 표시되는 base는 로컬 디렉
터리이거나 원격 리포지터리의 디렉터리가 될 수 있다. **overlay**는
`kustomization.yaml`이 있는 디렉터리로 다른 kustomization 디렉터리들을 `bases`로
참조한다. **base**는 overlay에 대해서 알지 못하며 여러 overlay들에서 사용될 수
있다. 한 overlay는 다수의 base들을 가질 수 있고, base들에서 모든 리소스를 구성�
수 있으며, 이들의 위에 사용자 정의도 가질 수 있다.

다음은 base에 대한 예이다.

```shell
# base를 가지는 디렉터리 생성
mkdir base
# base/deployment.yaml 생성
cat <<EOF > base/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
EOF

# base/service.yaml 파일 생성
cat <<EOF > base/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: my-nginx
EOF
# base/kustomization.yaml 생성
cat <<EOF > base/kustomization.yaml
resources:
- deployment.yaml
- service.yaml
EOF
```

이 base는 다수의 overlay에서 사용될 수 있다. 다른 `namePrefix` 또는 다른 교차 편
집 필드들을 서로 다른 overlay에 추가할 수 있다. 다음 예제는 동일한 base를 사용하
는 두 overlay들이다.

```shell
mkdir dev
cat <<EOF > dev/kustomization.yaml
bases:
- ../base
namePrefix: dev-
EOF

mkdir prod
cat <<EOF > prod/kustomization.yaml
bases:
- ../base
namePrefix: prod-
EOF
```

## Kustomize를 이용하여 오브젝트를 적용/확인/삭제하는 방법

`kustomization.yaml`에서 관리되는 리소스를 인식하려면 `kubectl` 명령어에
`--kustomize` 나 `-k`를 사용한다. `-k`는 다음과 같이 kustomization 디렉터리를 가
리키고 있어야 한다는 것을 주의한다.

```shell
kubectl apply -k <kustomization directory>/
```

다음 `kustomization.yaml`이 주어지고,

```shell
# deployment.yaml 파일 생성
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EOF

# kustomization.yaml 생성
cat <<EOF >./kustomization.yaml
namePrefix: dev-
commonLabels:
  app: my-nginx
resources:
- deployment.yaml
EOF
```

디플로이먼트 오브젝트 `dev-my-nginx`를 적용하려면 다음 명령어를 실행한다.

```shell
> kubectl apply -k ./
deployment.apps/dev-my-nginx created
```

디플로이먼트 오브젝트 `dev-my-nginx`를 보려면 다음 명령어들 중에 하나를 실행한다
.

```shell
kubectl get -k ./
```

```shell
kubectl describe -k ./
```

다음 명령을 실행해서 디플로이먼트 오브젝트 `dev-my-nginx` 를 매니페스트가 적용된
경우의 클러스터 상태와 비교한다.

```shell
kubectl diff -k ./
```

디플로이먼트 오브젝트 `dev-my-nginx`를 삭제하려면 다음 명령어를 실행한다.

```shell
> kubectl delete -k ./
deployment.apps "dev-my-nginx" deleted
```

## Kustomize 기능 리스트

| 필드                  | 유형                                                                                                                         | 설명                                                                                                                                                                       |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| namespace             | string                                                                                                                       | 모든 리소스에 네임스페이스 추가                                                                                                                                            |
| namePrefix            | string                                                                                                                       | 모든 리소스 네임에 이 필드의 값이 접두사로 추가된다                                                                                                                        |
| nameSuffix            | string                                                                                                                       | 모든 리소스 네임에 이 필드의 값이 접미사로 추가된다                                                                                                                        |
| commonLabels          | map[string]string                                                                                                            | 모든 리소스와 셀렉터에 추가될 레이블                                                                                                                                       |
| commonAnnotations     | map[string]string                                                                                                            | 모든 리소스에 추가될 어노테이션                                                                                                                                            |
| resources             | []string                                                                                                                     | 이 리스트 내 각각의 항목은 반드시 존재하는 리소스 구성 파일로 해석되어져야 한다                                                                                            |
| configmapGenerator    | [][configmapargs](https://github.com/kubernetes-sigs/kustomize/blob/release-kustomize-v4.0/api/types/kustomization.go#L99)   | 이 리스트 내 각각의 항목은 컨피그 맵을 생성한다                                                                                                                            |
| secretGenerator       | [][secretargs](https://github.com/kubernetes-sigs/kustomize/blob/release-kustomize-v4.0/api/types/kustomization.go#L106)     | 이 리스트 내 각각의 항목은 시크릿을 생성한다                                                                                                                               |
| generatorOptions      | [GeneratorOptions](https://github.com/kubernetes-sigs/kustomize/blob/release-kustomize-v4.0/api/types/kustomization.go#L109) | 모든 configMapGenerator와 secretGenerator의 동작을 변경                                                                                                                    |
| bases                 | []string                                                                                                                     | 이 리스트 내 각각의 항목은 kustomization.yaml 파일을 가지는 디렉터리로 해석되어져야 한다                                                                                   |
| patchesStrategicMerge | []string                                                                                                                     | 이 리스트 내 각각의 항목은 쿠버네티스 오브젝트의 전략적 병합 패치로 해석되어져야 한다                                                                                      |
| patchesJson6902       | [][json6902](https://github.com/kubernetes-sigs/kustomize/blob/release-kustomize-v4.0/api/types/patchjson6902.go#L8)         | 이 리스트 내 각각의 항목은 쿠버네티스 오브젝트와 Json 패치로 해석되어져야 한다                                                                                             |
| vars                  | [][var](https://github.com/kubernetes-sigs/kustomize/blob/master/api/types/var.go#L31)                                       | 각각의 항목은 한 리소스의 필드에서 텍스트를 캡쳐한다                                                                                                                       |
| images                | [][image](https://github.com/kubernetes-sigs/kustomize/tree/master/api/types/image.go#L23)                                   | 각각의 항목은 패치를 생성하지 않고 한 이미지의 name, tags 그리고/또는 digest를 수정한다                                                                                    |
| configurations        | []string                                                                                                                     | 이 리스트 내 각각의 항목은 [Kustomize 변환 설정](https://github.com/kubernetes-sigs/kustomize/tree/master/examples/transformerconfigs)을 포함하는 파일로 해석되어져야 한다 |
| crds                  | []string                                                                                                                     | 이 리스트 내 각각의 항목은 쿠버네티스 타입에 대한 OpenAPI 정의 파일로 해석되어져야 한다                                                                                    |

{{% /capture %}}

{{% capture whatsnext %}}

- [Kustomize](https://github.com/kubernetes-sigs/kustomize)
- [Kubectl Book](https://kubectl.docs.kubernetes.io)
- [Kubectl Command Reference](/docs/reference/generated/kubectl/kubectl/)
- [Kubernetes API
  Reference](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/)

{{% /capture %}}
