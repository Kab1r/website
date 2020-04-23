---
reviewers:
title: 스테이트풀셋 기본
content_template: templates/tutorial
weight: 10
---

{{% capture overview %}} 이 튜토리얼은 스테이트풀셋
([StatefulSets](/ko/docs/concepts/workloads/controllers/statefulset/))을 이용하
여애플리케이션을 관리하는 방법을 소개한다. 어떻게 스테이트풀셋의 파드(Pod)을 생
성하고 삭제하며스케일링하고 업데이트하는지 시연한다. {{% /capture %}}

{{% capture prerequisites %}} 튜토리얼을 시작하기 전에 다음의 쿠버네티스 컨셉에
대해익숙해야 한다.

- [파드](/docs/user-guide/pods/single-container/)
- [클러스터 DNS(Cluster DNS)](/ko/docs/concepts/services-networking/dns-pod-service/)
- [헤드리스 서비스(Headless Services)](/ko/docs/concepts/services-networking/service/#헤드리스-headless-서비스)
- [퍼시스턴트볼륨(PersistentVolumes)](/ko/docs/concepts/storage/persistent-volumes/)
- [퍼시턴트볼륨 프로비저닝](https://github.com/kubernetes/examples/tree/{{<
  param "githubbranch" >}}/staging/persistent-volume-provisioning/)
- [스테이트풀셋](/ko/docs/concepts/workloads/controllers/statefulset/)
- [kubectl CLI](/docs/user-guide/kubectl/)

이 튜토리얼은 클러스터가 퍼시스턴스볼륨을 동적으로 프로비저닝 하도록설정되었다�
가정한다. 만약 클러스터가 이렇게 설정되어 있지 않다면, 튜토리얼 시작 전에 수동으
로 2개의 1 GiB 볼륨을프로비저닝해야 한다. {{% /capture %}}

{{% capture objectives %}} 스테이트풀셋은 상태 유지가 필요한(stateful) 애플리케
이션과 분산시스템에서이용하도록 의도했다. 그러나 쿠버네티스 상에 스테이트풀 애플
리케이션과분산시스템을 관리하는 것은 광범위하고 복잡한 주제이다. 스테이트풀셋의
기본 기능을 보여주기 위해이 둘을 결합하지 않고, 스테이트풀셋을 사용한단순 웹 �
플리케이션을 배포할 것이다.

이 튜토리얼을 마치면 다음 항목에 대해 익숙해질 것이다.

- 스테이트풀셋을 어떻게 생성하는지
- 스테이트풀셋이 어떻게 파드를 관리하는지
- 스테이트풀셋을 어떻게 삭제하는지
- 스테이트풀셋은 어떻게 스케일링하는지
- 스테이트풀셋의 파드는 어떻게 업데이트하는지 {{% /capture %}}

{{% capture lessoncontent %}}

## 스테이트풀셋 생성하기

아래 예제를 이용해서 스테이트풀셋을 생성하자. 이는
[스테이트풀셋](/ko/docs/concepts/workloads/controllers/statefulset/) 개념에서 보
인예제와 유사하다. 이것은 `web`과 이 스테이트풀셋 파드의 IP 주소를 게시하는
[헤드리스 서비스](/ko/docs/concepts/services-networking/service/#헤드리스-headless-서비스)인
`nginx` 를 생성한다.

{{< codenew file="application/web/web.yaml" >}}

위에 예제를 다운로드 받아서 파일이름을 `web.yaml`으로 저장하자.

2개의 터미널창을 사용한다. 첫째 터미널에서
[`kubectl get`](/docs/reference/generated/kubectl/kubectl-commands/#get)을 이용
해서스테이트풀셋의 파드가 생성되는지 감시하자.

```shell
kubectl get pods -w -l app=nginx
```

두 번째 터미널에서
[`kubectl apply`](/docs/reference/generated/kubectl/kubectl-commands/#apply)로
`web.yaml`에 정의된 헤드리스 서비스와 스테이트풀셋을 생성한다.

```shell
kubectl apply -f web.yaml
service/nginx created
statefulset.apps/web created
```

상기 명령어는 [NGINX](https://www.nginx.com) 웹 서버를실행하는 2개의 파드를 생성
한다. `nginx` 서비스와 `web` 스테이트풀셋이 성공적으로 생성되었는지 알아보자.

```shell
kubectl get service nginx
NAME      TYPE         CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
nginx     ClusterIP    None         <none>        80/TCP    12s

kubectl get statefulset web
NAME      DESIRED   CURRENT   AGE
web       2         1         20s
```

### 차례대로 파드 생성하기

N개의 레플리카를 가진 스테이트풀셋은 배포 시에순차적으로 {0..N-1} 순으로 생성된
다. 첫째 터미널에서 `kubectl get` 명령의 출력 내용을 살펴보자. 결국 그 내용은 아
래 예와 비슷할 것이다.

```shell
kubectl get pods -w -l app=nginx
NAME      READY     STATUS    RESTARTS   AGE
web-0     0/1       Pending   0          0s
web-0     0/1       Pending   0         0s
web-0     0/1       ContainerCreating   0         0s
web-0     1/1       Running   0         19s
web-1     0/1       Pending   0         0s
web-1     0/1       Pending   0         0s
web-1     0/1       ContainerCreating   0         0s
web-1     1/1       Running   0         18s
```

`web-1` 파드는 `web-0` 파드가
[Running과 Ready](/ko/docs/concepts/workloads/pods/pod-lifecycle/) 상태가 되기
전에시작하지 않음을 주의하자.

## 스테이트풀셋 안에 파드

스테이트풀셋 안에 파드는 고유한 순번과 동일한 네트워크 신원을 가진다.

### 파드 순번 살펴보기

스테이트풀셋의 파드를 가져오자.

```shell
kubectl get pods -l app=nginx
NAME      READY     STATUS    RESTARTS   AGE
web-0     1/1       Running   0          1m
web-1     1/1       Running   0          1m

```

[스테이트풀셋](/ko/docs/concepts/workloads/controllers/statefulset/) 개념에서언
급했듯 스테이트풀셋의 파드는 끈끈하고 고유한 정체성을 가진다. 이 정체성은 스테이
트풀 컨트롤러에서 각 파드에 주어지는고유한 순번에 기인한다. 파드의 이름의 형식은
`<스테이트풀셋 이름>-<순번>` 이다. 앞서 `web` 스테이트풀셋은 2개의 레플리카를 가
졌으므로 `web-0` 과 `web-1` 2개 파드를 생성한다.

### 안정적인 네트워크 신원 사용하기

각 파드는 각 순번에 따른 안정적인 호스트네임을 갖는다. 각 파드에서 `hostname` 명
령어를 실행하도록
[`kubectl exec`](/docs/reference/generated/kubectl/kubectl-commands/#exec)를 이
용하자.

```shell
for i in 0 1; do kubectl exec web-$i -- sh -c 'hostname'; done
web-0
web-1
```

`dnsutils` 패키지에서 `nslookup` 명령을 제공하는 컨테이너를실행하도록
[`kubectl run`](/docs/reference/generated/kubectl/kubectl-commands/#run)을 이용
하자. 파드의 호스트네임에 `nslookup`을 이용하면 클러스터 내부 DNS 주소를확인�
수 있다.

```shell
kubectl run -i --tty --image busybox:1.28 dns-test --restart=Never --rm
nslookup web-0.nginx
Server:    10.0.0.10
Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-0.nginx
Address 1: 10.244.1.6

nslookup web-1.nginx
Server:    10.0.0.10
Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-1.nginx
Address 1: 10.244.2.6
```

헤드리스 서비스의 CNAME은 SRV 레코드를 지칭한다 (Running과 Ready 상태의 각 파드
마다 1개). SRV 레코드는 파드의 IP 주소를 포함한 A 레코드 엔트리를 지칭한다.

첫째 터미널에서 스테이트풀셋의 파드를 가져오자.

```shell
kubectl get pod -w -l app=nginx
```

두 번째 터미널에서 스테이트풀셋 내에 파드를 모두 삭제하기 위해
[`kubectl delete`](/docs/reference/generated/kubectl/kubectl-commands/#delete)를
이용하자.

```shell
kubectl delete pod -l app=nginx
pod "web-0" deleted
pod "web-1" deleted
```

스테이트풀셋이 재시작되고 두 파드가 Running과 Ready 상태로전환되도록 기다리자.

```shell
kubectl get pod -w -l app=nginx
NAME      READY     STATUS              RESTARTS   AGE
web-0     0/1       ContainerCreating   0          0s
NAME      READY     STATUS    RESTARTS   AGE
web-0     1/1       Running   0          2s
web-1     0/1       Pending   0         0s
web-1     0/1       Pending   0         0s
web-1     0/1       ContainerCreating   0         0s
web-1     1/1       Running   0         34s
```

파드의 호스트네임과 클러스터 내부 DNS 엔트리를 보기 위해 `kubectl exec`과
`kubectl run`을 이용하자.

```shell
for i in 0 1; do kubectl exec web-$i -- sh -c 'hostname'; done
web-0
web-1

kubectl run -i --tty --image busybox:1.28 dns-test --restart=Never --rm /bin/sh
nslookup web-0.nginx
Server:    10.0.0.10
Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-0.nginx
Address 1: 10.244.1.7

nslookup web-1.nginx
Server:    10.0.0.10
Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-1.nginx
Address 1: 10.244.2.8
```

파드의 순번, 호스트네임, SRV 레코드와 A 레코드이름은 변경되지 않지만파드의 IP 주
소는 변경될 수 있다. 이는 튜토리얼에서 사용하는 클러스터나다른 클러스터에도 동일
하다. 따라서 다른 애플리케이션이 IP 주소로스테이트풀셋의 파드에 접속하지 않도록
하는 것이 중요하다.

스테이트풀셋의 활성 멤버를 찾아 연결할 경우헤드리스 서비스
(`nginx.default.svc.cluster.local`)의 CNAME을 쿼리해야 한다. CNAME과 연관된 SRV
레코드는 스테이트풀셋의 Running과 Ready 상태의 모든 파드들을담고 있다.

애플리케이션에서 이미 활성상태(liveness)와 준비성(readiness) 테스트하는연결 로직
을 구현되어 있다면파드`web-0.nginx.default.svc.cluster.local`,
`web-1.nginx.default.svc.cluster.local`)의 SRV레코드를 안정적으로 사용할 수 있어
애플리케이션은 파드가 Running과 Ready 상태로 전환할 때파드의 주소를 검색할 수 있
다.

### 안정적인 스토리지에 쓰기 {#writing-to-stable-storage}

`web-0`과 `web-1`에 대해 퍼시스턴트볼륨클레임(PersistentVolumeClaim)을 가져오자.

```shell
kubectl get pvc -l app=nginx
NAME        STATUS    VOLUME                                     CAPACITY   ACCESSMODES   AGE
www-web-0   Bound     pvc-15c268c7-b507-11e6-932f-42010a800002   1Gi        RWO           48s
www-web-1   Bound     pvc-15c79307-b507-11e6-932f-42010a800002   1Gi        RWO           48s
```

스테이트풀셋 컨트롤러는 2개의
[퍼시스턴트볼륨](/ko/docs/concepts/storage/persistent-volumes/)에묶인 2개의 퍼시
스턴트볼륨클레임을 생성했다. 본 튜토리얼에서 사용되는 클러스터는 퍼시스턴트볼륨
을 동적으로프로비저닝하도록 설정되었으므로 생성된 퍼시스턴트볼륨도 자동으로 묶인
다.

NGINX 웹서버는 기본 색인 파일로 `/usr/share/nginx/html/index.html`을 이용합니다.
스테이트풀셋 `spec`내의 `volumeMounts` 필드는 `/usr/share/nginx/html` 디렉터리가
퍼시스턴트볼륨으로 제공되는지 보증합니다.

파드의 호스트네임을 `index.html` 파일에 작성하고 NGINX 웹서버가 해당 호스트네임
을 제공하는지 확인해보자.

```shell
for i in 0 1; do kubectl exec web-$i -- sh -c 'echo $(hostname) > /usr/share/nginx/html/index.html'; done

for i in 0 1; do kubectl exec -it web-$i -- curl localhost; done
web-0
web-1
```

{{< note >}} 위에 curl 명령어로 403 Forbidden 아닌 응답을 보려면
`volumeMounts`로 마운트된 디렉터리의 퍼미션을 수정해야 한다
([hostPath 볼륨을 사용할 때에 버그](https://github.com/kubernetes/kubernetes/issues/2630)로
인함).

```shell
for i in 0 1; do kubectl exec web-$i -- chmod 755 /usr/share/nginx/html; done
```

위에 curl 명령을 재시도하기 전에 {{< /note >}}

첫째 터미널에서 스테이트풀셋의 파드를 감시하자.

```shell
kubectl get pod -w -l app=nginx
```

두 번째 터미널에서 스테이트풀셋의 모든 파드를 삭제하자.

```shell
kubectl delete pod -l app=nginx
pod "web-0" deleted
pod "web-1" deleted
```

첫 번째 터미널에서 실행 중인 `kubectl get`명령어의 출력을 확인하고, 모든 파드가
Running과 Ready 상태로 전환될 때까지 기다리자.

```shell
kubectl get pod -w -l app=nginx
NAME      READY     STATUS              RESTARTS   AGE
web-0     0/1       ContainerCreating   0          0s
NAME      READY     STATUS    RESTARTS   AGE
web-0     1/1       Running   0          2s
web-1     0/1       Pending   0         0s
web-1     0/1       Pending   0         0s
web-1     0/1       ContainerCreating   0         0s
web-1     1/1       Running   0         34s
```

웹서버에서 자신의 호스트네임을 계속 제공하는지 확인하자.

```
for i in 0 1; do kubectl exec -it web-$i -- curl localhost; done
web-0
web-1
```

비록 `web-0`과 `web-1`이 재스케줄링되어도 계속해서자신의 호스트네임을 제공하는데
이는 각 퍼시스턴트볼륨클레임에연관된 퍼시스턴트볼륨이 해당 `volumeMounts`로 재마
운트되기 때문이다. `web-0`과 `web-1`의 스케줄링에 관계없이각각의 퍼시스턴트볼륨
은 적절하게 마운트된다.

## 스테이트풀셋 스케일링

스테이트풀셋을 스케일링하는 것은 레플리카 개수를 늘리거나 줄이는 것을 의미한다.
이것은 `replicas` 필드를 갱신하여 이뤄진다.
[`kubectl scale`](/docs/reference/generated/kubectl/kubectl-commands/#scale)이나
[`kubectl patch`](/docs/reference/generated/kubectl/kubectl-commands/#patch)을이
용해서 스테이트풀셋을 스케일링할 수 있다.

### 스케일 업

터미널창에서 스테이트풀셋의 파드를 감시하자.

```shell
kubectl get pods -w -l app=nginx
```

다른 터미널창에서 `kubectl scale`을 이용하여 레플리카 개수를 5로 스케일링하자.

```shell
kubectl scale sts web --replicas=5
statefulset.apps/web scaled
```

첫 번째 터미널에서 실행 중인 `kubectl get`명령어의 출력을 확인하고, 3개의 추가
파드가 Running과 Ready 상태로 전환될 때까지 기다리자.

```shell
kubectl get pods -w -l app=nginx
NAME      READY     STATUS    RESTARTS   AGE
web-0     1/1       Running   0          2h
web-1     1/1       Running   0          2h
NAME      READY     STATUS    RESTARTS   AGE
web-2     0/1       Pending   0          0s
web-2     0/1       Pending   0         0s
web-2     0/1       ContainerCreating   0         0s
web-2     1/1       Running   0         19s
web-3     0/1       Pending   0         0s
web-3     0/1       Pending   0         0s
web-3     0/1       ContainerCreating   0         0s
web-3     1/1       Running   0         18s
web-4     0/1       Pending   0         0s
web-4     0/1       Pending   0         0s
web-4     0/1       ContainerCreating   0         0s
web-4     1/1       Running   0         19s
```

스테이트풀셋 컨트롤러는 레플리카개수를 스케일링한다.
[스테이트풀셋 생성](#차례대로-파드-생성하기)으로 스테이트풀셋 컨트롤러는각 파드
을 순차적으로 각 순번에 따라 생성하고 후속 파드 시작 전에이전 파드가 Running과
Ready 상태가 될 때까지기다린다.

### 스케일 다운 {#scaling-down}

터미널에서 스테이트풀셋의 파드를 감시하자.

```shell
kubectl get pods -w -l app=nginx
```

다른 터미널에서 `kubectl patch`으로 스테이트풀셋을 뒤로 3개의 레플리카로 스케일
링하자.

```shell
kubectl patch sts web -p '{"spec":{"replicas":3}}'
statefulset.apps/web patched
```

`web-4`와 `web-3`이 Terminating으로 전환되기까지 기다리자.

```
kubectl get pods -w -l app=nginx
NAME      READY     STATUS              RESTARTS   AGE
web-0     1/1       Running             0          3h
web-1     1/1       Running             0          3h
web-2     1/1       Running             0          55s
web-3     1/1       Running             0          36s
web-4     0/1       ContainerCreating   0          18s
NAME      READY     STATUS    RESTARTS   AGE
web-4     1/1       Running   0          19s
web-4     1/1       Terminating   0         24s
web-4     1/1       Terminating   0         24s
web-3     1/1       Terminating   0         42s
web-3     1/1       Terminating   0         42s
```

### 순차 파드 종료

컨트롤러는 순번의 역순으로 한 번에 1개 파드를 삭제하고다음 파드를 삭제하기 전에
각각이 완전하게 종료되기까지 기다린다.

스테이트풀셋의 퍼시스턴트볼륨클레임을 가져오자.

```shell
kubectl get pvc -l app=nginx
NAME        STATUS    VOLUME                                     CAPACITY   ACCESSMODES   AGE
www-web-0   Bound     pvc-15c268c7-b507-11e6-932f-42010a800002   1Gi        RWO           13h
www-web-1   Bound     pvc-15c79307-b507-11e6-932f-42010a800002   1Gi        RWO           13h
www-web-2   Bound     pvc-e1125b27-b508-11e6-932f-42010a800002   1Gi        RWO           13h
www-web-3   Bound     pvc-e1176df6-b508-11e6-932f-42010a800002   1Gi        RWO           13h
www-web-4   Bound     pvc-e11bb5f8-b508-11e6-932f-42010a800002   1Gi        RWO           13h

```

여전히 5개의 퍼시스턴트볼륨클레임과 5개의 퍼시스턴트볼륨이 있다. 파드의
[안전한 스토리지](#writing-to-stable-storage)를 탐색하면서 스테이트풀셋의 파드가
삭제될 때에 파드에 마운트된 스테이트풀셋의 퍼시스턴트볼륨이 삭제되지 않은 것을
보았다. 스테이트풀셋 스케일 다운으로 파드 삭제할 때에도 여전히 사실이다.

## 스테이트풀셋 업데이트하기

쿠버네티스 1.7 이상에서 스테이트풀셋 컨트롤러는 자동 업데이트를 지원한다. 전략은
스테이트풀셋 API 오브젝트의 `spec.updateStrategy` 필드로 결정된다. 이 기능은 컨
테이너 이미지, 스테이트풀셋의 리소스 요청이나혹은 한계와 레이블과 파드의 어노테
이션을 업그레이드하기 위해 사용될 수 있다. `RollingUpdate`과 `OnDelete`의 2개의
유효한 업데이트 전략이 있다.

`RollingUpdate` 업데이트 전략은 스테이트풀셋에서 기본 값이다.

### 롤링 업데이트

`RollingUpdate` 업데이트 전략은 스테이트풀셋을 보장하면서 스테이트풀셋 내에 파드
를 역순으로 업데이트합니다.

스테이트풀셋 `web`의 업데이트 전략을 `RollingUpdate`으로 패치하자.

```shell
kubectl patch statefulset web -p '{"spec":{"updateStrategy":{"type":"RollingUpdate"}}}'
statefulset.apps/web patched
```

터미널 창에서 스테이트풀셋 `web`의 컨테이너 이미지를 바꾸도록또 패치하자.

```shell
kubectl patch statefulset web --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/image", "value":"gcr.io/google_containers/nginx-slim:0.8"}]'
statefulset.apps/web patched
```

다른 터미널창에서 스테이트풀셋의 파드를 감시하자.

```shell
kubectl get po -l app=nginx -w
NAME      READY     STATUS    RESTARTS   AGE
web-0     1/1       Running   0          7m
web-1     1/1       Running   0          7m
web-2     1/1       Running   0          8m
web-2     1/1       Terminating   0         8m
web-2     1/1       Terminating   0         8m
web-2     0/1       Terminating   0         8m
web-2     0/1       Terminating   0         8m
web-2     0/1       Terminating   0         8m
web-2     0/1       Terminating   0         8m
web-2     0/1       Pending   0         0s
web-2     0/1       Pending   0         0s
web-2     0/1       ContainerCreating   0         0s
web-2     1/1       Running   0         19s
web-1     1/1       Terminating   0         8m
web-1     0/1       Terminating   0         8m
web-1     0/1       Terminating   0         8m
web-1     0/1       Terminating   0         8m
web-1     0/1       Pending   0         0s
web-1     0/1       Pending   0         0s
web-1     0/1       ContainerCreating   0         0s
web-1     1/1       Running   0         6s
web-0     1/1       Terminating   0         7m
web-0     1/1       Terminating   0         7m
web-0     0/1       Terminating   0         7m
web-0     0/1       Terminating   0         7m
web-0     0/1       Terminating   0         7m
web-0     0/1       Terminating   0         7m
web-0     0/1       Pending   0         0s
web-0     0/1       Pending   0         0s
web-0     0/1       ContainerCreating   0         0s
web-0     1/1       Running   0         10s
```

스테이트풀셋 내에 파드는 순번의 역순으로 업데이트된다. 이 스테이트풀셋 컨트롤러
는 각 파드를 종료시키고 다음 파드를 업데이트하기 전에그것이 Running과 Ready 상태
로 전환될 때까지 기다린다. 알아둘 것은 비록 스테이트풀셋 컨트롤러에서 이전 파드
가 Running과 Ready 상태가 되기까지다음 파드를 업데이트하지 않아도 현재 버전으로
파드를 업데이트하다 실패하면 복원한다는 것이다. 업데이트를 이미 받은 파드는 업데
이트된 버전으로 복원되고 아직 업데이트를 받지 못한 파드는이전 버전으로 복원한다.
이런 식으로 컨트롤러는 간헐적인 오류가 발생해도애플리케이션을 계속 건강하게 유지
하고업데이트도 일관되게 유지하려 한다.

컨테이너 이미지를 살펴보기 위해 파드를 가져오자.

```shell
for p in 0 1 2; do kubectl get po web-$p --template '{{range $i, $c := .spec.containers}}{{$c.image}}{{end}}'; echo; done
k8s.gcr.io/nginx-slim:0.8
k8s.gcr.io/nginx-slim:0.8
k8s.gcr.io/nginx-slim:0.8

```

스테이트풀셋의 모든 파드가 지금은 이전 컨테이너 이미지를 실행 중이이다.

**팁** 롤링 업데이트 상황을 살펴보기 위해 `kubectl rollout status sts/<name>` 명
령어도 사용할 수 있다.

#### 단계적으로 업데이트 하기 {#staging-an-update}

`RollingUpdate` 업데이트 전략의 파라미터인 `partition`를 이용하여스테이트풀셋의
단계적으로 업데이트할 수 있다. 단계적 업데이트는 스테이트풀셋의 모든 파드를 현재
버전으로 유지하면서스테이트풀셋의 `.spec.template`에 변경을 허용한다.

스테이트풀셋 `web`의 `updateStrategy` 필드에 partition을 추가하자.

```shell
kubectl patch statefulset web -p '{"spec":{"updateStrategy":{"type":"RollingUpdate","rollingUpdate":{"partition":3}}}}'
statefulset.apps/web patched
```

컨테이너의 이미지를 바꾸도록 스테이트풀셋을 또 패치하자.

```shell
kubectl patch statefulset web --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/image", "value":"k8s.gcr.io/nginx-slim:0.7"}]'
statefulset.apps/web patched
```

스테이트풀셋의 파드를 삭제하자.

```shell
kubectl delete po web-2
pod "web-2" deleted
```

파드가 Running과 Ready 상태가 되기까지 기다리자.

```shell
kubectl get po -l app=nginx -w
NAME      READY     STATUS              RESTARTS   AGE
web-0     1/1       Running             0          4m
web-1     1/1       Running             0          4m
web-2     0/1       ContainerCreating   0          11s
web-2     1/1       Running   0         18s
```

파드의 컨테이너를 가져오자.

```shell
kubectl get po web-2 --template '{{range $i, $c := .spec.containers}}{{$c.image}}{{end}}'
k8s.gcr.io/nginx-slim:0.8

```

비록 업데이트 전략이 `RollingUpdate`이지만 스테이트풀셋은파드를 그것의 원래 컨테
이너로 복원한다. 파드의 순번이 `updateStrategy`에서 지정된 `파티션`보다 작기 때
문이다.

#### 카나리(Canary) 롤링 아웃

[위에서](#staging-an-update) 지정한 `partition`값을 차감시키면변경사항을 테스트
하기 위해 카나리 롤아웃을 할 수 있다.

스테이트풀셋에 partition을 차감하도록 패치하자.

```shell
kubectl patch statefulset web -p '{"spec":{"updateStrategy":{"type":"RollingUpdate","rollingUpdate":{"partition":2}}}}'
statefulset.apps/web patched
```

`web-2` 파드가 Running과 Ready 상태가 되기까지 기다리자.

```shell
kubectl get po -l app=nginx -w
NAME      READY     STATUS              RESTARTS   AGE
web-0     1/1       Running             0          4m
web-1     1/1       Running             0          4m
web-2     0/1       ContainerCreating   0          11s
web-2     1/1       Running   0         18s
```

파드의 컨테이너를 가져오자.

```shell
kubectl get po web-2 --template '{{range $i, $c := .spec.containers}}{{$c.image}}{{end}}'
k8s.gcr.io/nginx-slim:0.7

```

`partition`을 바꾸면 스테이트풀셋 컨트롤러는 자동으로 `web-2` 파드를 업데이트하
는데이는 해당 파드의 순번이 `partition` 이상이기 때문이다.

`web-1` 파드를 삭제하자.

```shell
kubectl delete po web-1
pod "web-1" deleted
```

`web-1` 파드가 Running과 Ready 상태가 되기까지 기다리자.

```shell
kubectl get po -l app=nginx -w
NAME      READY     STATUS        RESTARTS   AGE
web-0     1/1       Running       0          6m
web-1     0/1       Terminating   0          6m
web-2     1/1       Running       0          2m
web-1     0/1       Terminating   0         6m
web-1     0/1       Terminating   0         6m
web-1     0/1       Terminating   0         6m
web-1     0/1       Pending   0         0s
web-1     0/1       Pending   0         0s
web-1     0/1       ContainerCreating   0         0s
web-1     1/1       Running   0         18s
```

`web-1` 파드의 컨테이너를 가져오자.

```shell
kubectl get po web-1 --template '{{range $i, $c := .spec.containers}}{{$c.image}}{{end}}'
k8s.gcr.io/nginx-slim:0.8

```

`web-1` 는 원래 환경설정으로 복원되었는데이는 파드의 순번이 partition보다 작기
때문이다. 스테이트풀셋의 `.spec.template`이 갱신되면, 지정된 partition 이상의 순
번을가진 모든 파드는 업데이트된다. 미만의 순번을 가진 파드라면 삭제되거나종료되
어 원래 환경설정으로 복원된다.

#### 단계적 롤아웃

[카나리 롤아웃](#카나리-canary-롤링-아웃)에서 했던 방법과 비슷하게분할된 롤링 업
데이트를 이용하여 단계적 롤아웃(e.g. 선형, 기하 또는 지수적 롤아웃)을수행할 수
있다. 단계적 롤아웃을 수행하려면컨트롤러가 업데이트를 일시 중지할 순번으로
`partition`를 정하자.

partition은 현재 `2`이다. partition을 `0`으로 바꾸자.

```shell
kubectl patch statefulset web -p '{"spec":{"updateStrategy":{"type":"RollingUpdate","rollingUpdate":{"partition":0}}}}'
statefulset.apps/web patched
```

스테이트풀셋의 모든 파드가 Running과 Ready 상태가 되기까지 기다리자.

```shell
kubectl get po -l app=nginx -w
NAME      READY     STATUS              RESTARTS   AGE
web-0     1/1       Running             0          3m
web-1     0/1       ContainerCreating   0          11s
web-2     1/1       Running             0          2m
web-1     1/1       Running   0         18s
web-0     1/1       Terminating   0         3m
web-0     1/1       Terminating   0         3m
web-0     0/1       Terminating   0         3m
web-0     0/1       Terminating   0         3m
web-0     0/1       Terminating   0         3m
web-0     0/1       Terminating   0         3m
web-0     0/1       Pending   0         0s
web-0     0/1       Pending   0         0s
web-0     0/1       ContainerCreating   0         0s
web-0     1/1       Running   0         3s
```

파드의 컨테이너를 가져오자.

```shell
for p in 0 1 2; do kubectl get po web-$p --template '{{range $i, $c := .spec.containers}}{{$c.image}}{{end}}'; echo; done
k8s.gcr.io/nginx-slim:0.7
k8s.gcr.io/nginx-slim:0.7
k8s.gcr.io/nginx-slim:0.7

```

`partition`을 `0`으로 이동하여 스테이트풀셋 컨트롤러에서 계속해서업데이트 처리를
하도록 허용하였다.

### 삭제 시 동작

`OnDelete` 업데이트 전략은 예전 동작(1.6 이하)으로, 이 업데이트 전략을 선택하면
스테이트풀셋 컨트롤러는 스테이트풀셋의 `.spec.template` 필드에 수정 사항이 발생
해도 자동으로 파드를 업데이트하지 않는다. 이 전략은
`.spec.template.updateStrategy.type`을 `OnDelete`로 설정하여 선택할 수 있다.

## 스테이트풀셋 삭제하기

스테이트풀셋은 비종속적(non-cascading), 종속적(cascading) 삭제를 둘 다 지원한다.
비종속적 삭제에서는 스테이트풀셋이 지워질 때에 스테이트풀셋의 파드는 지워지지 않
는다. 종속적 삭제에서는 스테이트풀셋과 그에 속한 파드가 모두 지워진다.

### 비종속적 삭제

터미널창에서 스테이트풀셋의 파드를 감시하자.

```
kubectl get pods -w -l app=nginx
```

다른 터미널에서는 스테이트풀셋을 지우기 위해
[`kubectl delete`](/docs/reference/generated/kubectl/kubectl-commands/#delete)
명령어를 이용하자. 이 명령어에 `--cascade=false` 파라미터가 추가되었다. 이 파라
미터는 쿠버네티스에 스테이트풀셋만 삭제하고 그에 속한 파드는 지우지 않도록 요청
한다.

```shell
kubectl delete statefulset web --cascade=false
statefulset.apps "web" deleted
```

상태를 확인하기 위해 파드를 가져오자.

```shell
kubectl get pods -l app=nginx
NAME      READY     STATUS    RESTARTS   AGE
web-0     1/1       Running   0          6m
web-1     1/1       Running   0          7m
web-2     1/1       Running   0          5m
```

비록 `web`이 삭제되고 있어도, 모든 파드는 여전히 Running과 Ready 상태이다.
`web-0`을 삭제하자.

```shell
kubectl delete pod web-0
pod "web-0" deleted
```

스테이트풀셋의 파드를 가져오자.

```shell
kubectl get pods -l app=nginx
NAME      READY     STATUS    RESTARTS   AGE
web-1     1/1       Running   0          10m
web-2     1/1       Running   0          7m
```

스테이트풀셋 `web`이 삭제되는 동안 `web-0`은 재시작하지 않았다.

첫째 터미널에서 스테이트풀셋의 파드를 감시하자.

```shell
kubectl get pods -w -l app=nginx
```

두 번째 터미널에서 스테이트풀셋을 다시 생성하자. `nginx` 서비스(가지지 말았어야
하는)를 삭제하기 전까지는 그 서비스가 이미 존재한다는 에러를볼 것이라는 것을 명
심하자.

```shell
kubectl apply -f web.yaml
statefulset.apps/web created
service/nginx unchanged
```

이 에러는 무시하자. 이것은 다만 해당 서비스가 있더라도 nginx 헤드리스 서비스를
생성하려고 했음을 뜻한다.

첫째 터미널에서 실행 중인 `kubectl get` 명령어의 출력을 살펴보자.

```shell
kubectl get pods -w -l app=nginx
NAME      READY     STATUS    RESTARTS   AGE
web-1     1/1       Running   0          16m
web-2     1/1       Running   0          2m
NAME      READY     STATUS    RESTARTS   AGE
web-0     0/1       Pending   0          0s
web-0     0/1       Pending   0         0s
web-0     0/1       ContainerCreating   0         0s
web-0     1/1       Running   0         18s
web-2     1/1       Terminating   0         3m
web-2     0/1       Terminating   0         3m
web-2     0/1       Terminating   0         3m
web-2     0/1       Terminating   0         3m
```

`web` 스테이트풀셋이 다시 생성될 때 먼저 `web-0` 시작한다. `web-1`은 이미
Running과 Ready 상태이므로 `web-0`이 Running과 Ready 상태로전환될 때는 단순히 이
파드에 적용됬다. 스테이트풀셋에`replicas`를 2로 하고 `web-0`을 재생성했다면
`web-1`이이미 Running과 Ready 상태이고, `web-2`은 종료되었을 것이다.

파드의 웹서버에서 제공한 `index.html` 파일 내용을다른 관점으로 살펴보자.

```shell
for i in 0 1; do kubectl exec -it web-$i -- curl localhost; done
web-0
web-1
```

스테이트풀셋과 `web-0` 파드를 둘 다 삭제했으나 여전히 `index.html` 파일에 입력했
던원래 호스트네임을 제공한다. 스테이트풀셋은파드에 할당된 퍼시스턴트볼륨을 결코
삭제하지 않기때문이다. 다시 스테이트풀셋을 생성하면 `web-0`을 시작하며원래 퍼시
스턴트볼륨을 다시 마운트한다.

### 단계식 삭제

터미널창에서 스테이트풀셋의 파드를 감시하자.

```shell
kubectl get pods -w -l app=nginx
```

다른 터미널창에서 스테이트풀셋을 다시 지우자. 이번에는 `--cascade=false` 파라미
터를 생략하자.

```shell
kubectl delete statefulset web
statefulset.apps "web" deleted
```

첫째 터미널에서 실행 중인 `kubectl get` 명령어의 출력을 살펴보고모든 파드가
Terminating 상태로 전환될 때까지 기다리자.

```shell
kubectl get pods -w -l app=nginx
NAME      READY     STATUS    RESTARTS   AGE
web-0     1/1       Running   0          11m
web-1     1/1       Running   0          27m
NAME      READY     STATUS        RESTARTS   AGE
web-0     1/1       Terminating   0          12m
web-1     1/1       Terminating   0         29m
web-0     0/1       Terminating   0         12m
web-0     0/1       Terminating   0         12m
web-0     0/1       Terminating   0         12m
web-1     0/1       Terminating   0         29m
web-1     0/1       Terminating   0         29m
web-1     0/1       Terminating   0         29m

```

[스케일 다운](#scaling-down) 섹션에서 보았듯 파드는각 순번의 역순으로 하나씩 종
료된다. 파드가 종료될 때스테이트풀 컨트롤러는 이전 파드가완전히 종료되기까지 기
다린다.

스테이트풀셋과 그 파드를 종속적으로 삭제하는 중에 연관된 헤드리스 서비스를삭제하
지 않음을 주의하자. 꼭 `nginx` 서비스를 수동으로 삭제해라.

```shell
kubectl delete service nginx
service "nginx" deleted
```

스테이트풀셋과 헤드리스 서비스를 한번 더 다시 생성하자.

```shell
kubectl apply -f web.yaml
service/nginx created
statefulset.apps/web created
```

스테이트풀셋의 모든 파드가 Running과 Ready 상태로 전환될 때 `index.html` 파일 내
용을 검색하자.

```shell
for i in 0 1; do kubectl exec -it web-$i -- curl localhost; done
web-0
web-1
```

스테이트풀셋과 그 내부의 모든 파드를 삭제했지만 퍼시스턴트볼륨이 마운트된 채로다
시 생성되고 `web-0`과 `web-1`은 여전히각 호스트네임을 제공한다.

최종적으로 `web` 스테이트풀셋과`nginx` 서비스를 삭제한다.

```shell
kubectl delete service nginx
service "nginx" deleted

kubectl delete statefulset web
statefulset "web" deleted
```

## 파드 관리 정책

일부 분산 시스템의 경우 스테이트풀셋의 순서 보증은불필요하거나 바람직하지 않다.
이러한 시스템은 고유성과 신원만 필요하다. 이를 해결하기 위해 쿠버네티스 1.7에서
`.spec.podManagementPolicy`를스테이트풀셋 API 오브젝트에 도입했다.

### OrderedReady 파드 관리

`OrderedReady` 파드 관리는 스테이트풀셋에서는 기본이다. 이는 스테이트풀셋 컨트롤
러가 지금까지 위에서 설명했던 순서를보증함을 뜻한다.

### Parallel 파드 관리

`Parallel` 파드 관리는 스테이트풀셋 컨트롤러가 모든 파드를병렬로 시작하고 종료하
는 것으로 다른 파드를 시작/종료하기 전에파드가 Running과 Ready 상태로 전환되거나
완전히 종료되기까지기다리지 않음을 뜻한다.

{{< codenew file="application/web/web-parallel.yaml" >}}

상기 예제를 다운로드받아 파일 이름을 `web-parallel.yaml`로 저장하자.

이 매니페스트는 `web` 스테이트풀셋의 `.spec.podManagementPolicy`이 `Parallel`인
것 말고는 이전에 다운로드 받았던 것과 동일하다.

터미널에서 스테이트풀셋의 파드를 감시하자.

```shell
kubectl get po -l app=nginx -w
```

다른 터미널에서 매니페스트 안에 스테이트풀셋과 서비스를 생성하자.

```shell
kubectl apply -f web-parallel.yaml
service/nginx created
statefulset.apps/web created
```

첫째 터미널에서 실행했던 `kubectl get` 명령어의 출력을 살펴보자.

```shell
kubectl get po -l app=nginx -w
NAME      READY     STATUS    RESTARTS   AGE
web-0     0/1       Pending   0          0s
web-0     0/1       Pending   0         0s
web-1     0/1       Pending   0         0s
web-1     0/1       Pending   0         0s
web-0     0/1       ContainerCreating   0         0s
web-1     0/1       ContainerCreating   0         0s
web-0     1/1       Running   0         10s
web-1     1/1       Running   0         10s
```

스테이트풀셋 컨트롤러는 `web-0`와 `web-1`를 둘 다 동시에 시작했다.

두 번째 터미널을 열어 놓고 다른 터미널창에서 스테이트풀셋을스케일링 하자.

```shell
kubectl scale statefulset/web --replicas=4
statefulset.apps/web scaled
```

`kubectl get` 명령어를 실행 중인 터미널의 출력을 살펴보자.

```shell
web-3     0/1       Pending   0         0s
web-3     0/1       Pending   0         0s
web-3     0/1       Pending   0         7s
web-3     0/1       ContainerCreating   0         7s
web-2     1/1       Running   0         10s
web-3     1/1       Running   0         26s
```

스테이트풀 컨트롤러는 두 개의 새 파드를 시작하였다. 두 번째 것을 런칭하기 위해
먼저 런칭한 것이 Running과 Ready 상태가 될 때까지 기다리지 않는다.

이 터미널을 열어 놓고 다른 터미널에서 `web` 스테이트풀셋을 삭제하자.

```shell
kubectl delete sts web
```

다시 한번 다른 터미널에서 실행 중인 `kubectl get`명령의 출력을 확인해보자.

```shell
web-3     1/1       Terminating   0         9m
web-2     1/1       Terminating   0         9m
web-3     1/1       Terminating   0         9m
web-2     1/1       Terminating   0         9m
web-1     1/1       Terminating   0         44m
web-0     1/1       Terminating   0         44m
web-0     0/1       Terminating   0         44m
web-3     0/1       Terminating   0         9m
web-2     0/1       Terminating   0         9m
web-1     0/1       Terminating   0         44m
web-0     0/1       Terminating   0         44m
web-2     0/1       Terminating   0         9m
web-2     0/1       Terminating   0         9m
web-2     0/1       Terminating   0         9m
web-1     0/1       Terminating   0         44m
web-1     0/1       Terminating   0         44m
web-1     0/1       Terminating   0         44m
web-0     0/1       Terminating   0         44m
web-0     0/1       Terminating   0         44m
web-0     0/1       Terminating   0         44m
web-3     0/1       Terminating   0         9m
web-3     0/1       Terminating   0         9m
web-3     0/1       Terminating   0         9m
```

스테이트풀 컨트롤러는 모든 파드를 동시에 삭제한다. 파드를 삭제하기 전에그 파드의
순서상 후계자를 기다리지 않는다.

`kubectl get` 명령어가 실행된 터미널을 닫고 `nginx` 서비스를 삭제하자.

```shell
kubectl delete svc nginx
```

{{% /capture %}}

{{% capture cleanup %}} 이 튜토리얼에서 사용된 퍼시턴트볼륨을 위한퍼시스턴트 스
토리지 미디어를 삭제해야 한다. 모든 스토리지를 반환하도록 환경, 스토리지 설정과
프로비저닝 방법에 따른 단계를 따르자. {{% /capture %}}
