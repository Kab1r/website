---
title: 분산 시스템 코디네이터 ZooKeeper 실행하기
content_template: templates/tutorial
weight: 40
---

{{% capture overview %}} 이 튜토리얼은
[아파치 ZooKeeper](https://zookeeper.apache.org) 쿠버네티스에서
[스테이트풀셋](/ko/docs/concepts/workloads/controllers/statefulset/)과
[파드디스룹선버짓(PodDisruptionBudget)](/ko/docs/concepts/workloads/pods/disruptions/#specifying-a-poddisruptionbudget)과
[파드안티어피니티(PodAntiAffinity)](/ko/docs/user-guide/node-selection/#파드간-어피니티와-안티-어피니티)를
이용한 [Apache Zookeeper](https://zookeeper.apache.org) 실행을 설명한다.
{{% /capture %}}

{{% capture prerequisites %}}

이 튜토리얼을 시작하기 전에다음 쿠버네티스 개념에 친숙해야 한다.

- [파드](/docs/user-guide/pods/single-container/)
- [클러스터 DNS](/ko/docs/concepts/services-networking/dns-pod-service/)
- [헤드리스 서비스](/ko/docs/concepts/services-networking/service/#헤드리스-headless-서비스)
- [퍼시스턴트볼륨](/ko/docs/concepts/storage/volumes/)
- [퍼시스턴트볼륨 프로비저닝](https://github.com/kubernetes/examples/tree/{{<
  param "githubbranch" >}}/staging/persistent-volume-provisioning/)
- [스테이트풀셋](/ko/docs/concepts/workloads/controllers/statefulset/)
- [파드디스룹션버짓](/ko/docs/concepts/workloads/pods/disruptions/#specifying-a-poddisruptionbudget)
- [파드안티어피니티](/ko/docs/user-guide/node-selection/#파드간-어피니티와-안티-어피니티)
- [kubectl CLI](/docs/user-guide/kubectl/)

최소한 4개의 노드가 있는 클러스터가 필요하며, 각 노드는 적어도 2 개의 CPU와 4
GiB 메모리가 필요하다. 이 튜토리얼에서 클러스터 노드를 통제(cordon)하고 비우게
(drain) 할 것이다. **이것은 클러스터를 종료하여 노드의 모든 파드를 퇴출(evict)하
는 것으로, 모든 파드는 임시로 언스케줄된다는 의미이다.** 이 튜토리얼을 위해 전용
클러스터를 이용하거나, 다른 테넌트에 간섭을 하는 혼란이 발생하지 않도록 해야 합
니다.

이 튜토리얼은 클러스터가 동적으로 퍼시스턴트볼륨을 프로비저닝하도록 구성한다�
가정한다. 그렇게 설정되어 있지 않다면튜토리얼을 시작하기 전에 수동으로 3개의 20
GiB 볼륨을프로비저닝해야 한다. {{% /capture %}}

{{% capture objectives %}} 이 튜토리얼을 마치면 다음에 대해 알게 된다.

- 어떻게 스테이트풀셋을 이용하여 ZooKeeper 앙상블을 배포하는가.
- 어떻게 지속적해서 컨피그맵을 이용해서 앙상블을 설정하는가.
- 어떻게 ZooKeeper 서버 디플로이먼트를 앙상블 안에서 퍼뜨리는가.
- 어떻게 파드디스룹션버짓을 이용하여 계획된 점검 기간 동안 서비스 가용성을 보장
  하는가. {{% /capture %}}

{{% capture lessoncontent %}}

### ZooKeeper 기본 {#zookeeper-basics}

[아파치 ZooKeeper](https://zookeeper.apache.org/doc/current/)는분산 애플리케이션
을 위한 분산 오픈 소스 코디네이션 서비스이다. ZooKeeper는 데이터를 읽고 쓰고 갱
신을 지켜보도록 한다. 데이터는파일시스템처럼 계층적으로 관리되고 앙상블
(ZooKeeper 서버의 집합) 내에 모든 ZooKeeper서버에 복제된다. 데이터에 모든 연산은
원자적이고 순처적으로 일관된다. ZooKeeper는
[Zab](https://pdfs.semanticscholar.org/b02c/6b00bd5dbdbd951fddb00b906c82fa80f0b3.pdf)
합의 프로토콜을이용하여 앙상블 내에 모든 서버에 걸쳐 상태 머신을 복제하여 이를
보장한다.

앙상블은 리더 선출을 위해 Zab 프로토콜을 사용하고, 리더 선출과 선거가 완료되기
전까지 앙상블은 데이터를 쓸 수 없다. 완료되면 앙상블은 Zab을 이용하여 확인하�
클라이언트에 보이도록 모든 쓰기를 쿼럼(quorum)에 복제한다. 가중치있는 쿼럼과 관
련 없이, 쿼럼은 현재 리더를 포함하는 앙상블의 대다수 컴포넌트이다. 예를 들어 앙
상블이 3개 서버인 경우, 리더와 다른 서버로 쿼럼을 구성한다. 앙상블이 쿼럼을 달성
할 수 없다면, 앙상블은 데이터를 쓸 수 없다.

ZooKeeper는 전체 상태 머신을 메모리에 보존하고 모든 돌연변이를 저장 미디어의 내
구성 있는 WAL(Write Ahead Log)에 기록한다. 서버 장애시 WAL을 재생하여 이전 상태
를 복원할 수 있다. WAL이 무제한으로 커지는 것을 방지하기 위해 ZooKeeper는 주기적
으로 저장 미디어에 메모리 상태의 스냅샷을 저장한다. 이 스냅샷은 메모리에 직접 적
재할 수 있고 스냅샷 이전의 모든 WAL 항목은 삭제될 수 있다.

## ZooKeeper 앙상블 생성하기

아래 메니페스트에는
[헤드리스 서비스](/ko/docs/concepts/services-networking/service/#헤드리스-headless-서비스),
[서비스](/ko/docs/concepts/services-networking/service/),
[파드디스룹션버짓](/ko/docs/concepts/workloads/pods/disruptions//#specifying-a-poddisruptionbudget),
[스테이트풀셋](/ko/docs/concepts/workloads/controllers/statefulset/)을 포함한다.

{{< codenew file="application/zookeeper/zookeeper.yaml" >}}

터미널을 열�
[`kubectl apply`](/docs/reference/generated/kubectl/kubectl-commands/#apply) 명
령어로메니페스트를 생성하자.

```shell
kubectl apply -f https://k8s.io/examples/application/zookeeper/zookeeper.yaml
```

이는 `zk-hs` 헤드리스 서비스, `zk-cs` 서비스, `zk-pdb` PodDisruptionBudget과
`zk` 스테이트풀셋을 생성한다.

```shell
service/zk-hs created
service/zk-cs created
poddisruptionbudget.policy/zk-pdb created
statefulset.apps/zk created
```

[`kubectl get`](/docs/reference/generated/kubectl/kubectl-commands/#get)을 사용
하여스테이트풀셋 컨트롤러가 스테이트풀셋 파드를 생성하는지 확인한다.

```shell
kubectl get pods -w -l app=zk
```

`zk-2` 파드가 Running and Ready 상태가 되면, `CTRL-C`를 눌러 kubectl을 종료하자.

```shell
NAME      READY     STATUS    RESTARTS   AGE
zk-0      0/1       Pending   0          0s
zk-0      0/1       Pending   0         0s
zk-0      0/1       ContainerCreating   0         0s
zk-0      0/1       Running   0         19s
zk-0      1/1       Running   0         40s
zk-1      0/1       Pending   0         0s
zk-1      0/1       Pending   0         0s
zk-1      0/1       ContainerCreating   0         0s
zk-1      0/1       Running   0         18s
zk-1      1/1       Running   0         40s
zk-2      0/1       Pending   0         0s
zk-2      0/1       Pending   0         0s
zk-2      0/1       ContainerCreating   0         0s
zk-2      0/1       Running   0         19s
zk-2      1/1       Running   0         40s
```

스테이트풀셋 컨트롤러는 3개의 파드를 생성하고, 각 파드는
[ZooKeeper](http://www-us.apache.org/dist/zookeeper/stable/) 서버를 포함한 컨테
이너를 가진다.

### 리더 선출 촉진

익명 네트워크에서 리더 선출을 위한 종료 알고리즘이 없기에, Zab은 리더 선출을 위
해 명시적인 멤버 구성을 해야 한다. 앙상블의 각 서버는 고유 식별자를 가져야 하고,
모든 서버는 식별자 전역 집합을 알아야 하며, 각 식별자는 네트워크 주소에 연관되어
야 한다.

[`kubectl exec`](/docs/reference/generated/kubectl/kubectl-commands/#exec)를 이
용하여 `zk` 스테이트풀셋의 파드의호스트네임을 알아내자.

```shell
for i in 0 1 2; do kubectl exec zk-$i -- hostname; done
```

스테이트풀셋 컨트롤러는 각 순번 인덱스에 기초하여 각 파드에 고유한 호스트네임을
부여한다. 각 호스트네임은 `<스테이트풀셋 이름>-<순번 인덱스>` 형식을 취한다.
`zk` 스테이트풀셋의 `replicas` 필드는 `3`으로 설정되었기 때문에, 그 스테이트풀셋
컨트롤러는 3개 파드의 호스트네임을 `zk-0`, `zk-1`, `zk-2`로 정한다.

```shell
zk-0
zk-1
zk-2
```

ZooKeeper 앙상블에 서버들은 고유 식별자로서 자연수를 이용하고 서버 데이터 디렉터
리에 `my` 라는 파일로 서버 식별자를 저장한다.

각 서버에서 다음 명령어를 이용하여 `myid` 파일의 내용을 확인하자.

```shell
for i in 0 1 2; do echo "myid zk-$i";kubectl exec zk-$i -- cat /var/lib/zookeeper/data/myid; done
```

식별자는 자연수이고, 순번 인덱스들도 음수가 아니므로, 순번에 1을 더하여 순번을
만들 수 있다.

```shell
myid zk-0
1
myid zk-1
2
myid zk-2
3
```

`zk` 스테이트풀셋의 각 파드 Fully Qualified Domain Name (FQDN)을 얻기 위해 다음
명령어를 이용하자.

```shell
for i in 0 1 2; do kubectl exec zk-$i -- hostname -f; done
```

`zk-hs` 서비스는 모든 파드를 위한 도메인인 `zk-hs.default.svc.cluster.local`을
만든다.

```shell
zk-0.zk-hs.default.svc.cluster.local
zk-1.zk-hs.default.svc.cluster.local
zk-2.zk-hs.default.svc.cluster.local
```

[쿠버네티스 DNS](/ko/docs/concepts/services-networking/dns-pod-service/)의 A 레
코드는 FQDN을 파드의 IP 주소로 풀어낸다. 쿠버네티스가 파드를 리스케줄하면, 파드
의 새 IP 주소로 A 레코드를 갱신하지만, A 레코드의 이름은 바뀌지 않는다.

ZooKeeper는 그것의 애플리케이션 환경설정을 `zoo.cfg` 파일에 저장한다.
`kubectl exec`를 이용하여 `zk-0` 파드의 `zoo.cfg` 내용을 보자.

```shell
kubectl exec zk-0 -- cat /opt/zookeeper/conf/zoo.cfg
```

아래 파일의 `server.1`, `server.2`, `server.3` 속성에서 `1`, `2`, `3`은
ZooKeeper 서버의 `myid` 파일에 구분자와 연관된다. 이들은 `zk` 스테이트풀셋의 파
드의 FQDNS을 설정한다.

```shell
clientPort=2181
dataDir=/var/lib/zookeeper/data
dataLogDir=/var/lib/zookeeper/log
tickTime=2000
initLimit=10
syncLimit=2000
maxClientCnxns=60
minSessionTimeout= 4000
maxSessionTimeout= 40000
autopurge.snapRetainCount=3
autopurge.purgeInterval=0
server.1=zk-0.zk-hs.default.svc.cluster.local:2888:3888
server.2=zk-1.zk-hs.default.svc.cluster.local:2888:3888
server.3=zk-2.zk-hs.default.svc.cluster.local:2888:3888
```

### 합의 달성

합의 프로토콜에서 각 참가자의 식별자는 유일해야 한다. Zab 프로토콜에서 동일한 �
유 식별자를 요청하는 참가자는 없다. 이는 시스템 프로세스가 어떤 프로세스가 어떤
데이터를 커밋했는지 동의하게 하는데 필요하다. 2개 파드를 동일 순번으로 시작하였
다면 두 대의 ZooKeeper 서버는 둘 다 스스로를 동일 서버로 식별한다.

합의 프로토콜에서 각 참여자의 식별자는 고유해야 한다. Zab 프로토콜에 두 참여자가
동일한 고유 식별자로 요청해서는 안된다. 이는 시스템 프로세스가 어떤 프로세스가
어떤 데이터를 커밋했는지 동의하도록 하기 위해 필수적이다. 동일 순번으로 두 개의
파드가 실행했다면 두 ZooKeeper 서버는 모두 동일한 서버로 식별된다.

```shell
kubectl get pods -w -l app=zk

NAME      READY     STATUS    RESTARTS   AGE
zk-0      0/1       Pending   0          0s
zk-0      0/1       Pending   0         0s
zk-0      0/1       ContainerCreating   0         0s
zk-0      0/1       Running   0         19s
zk-0      1/1       Running   0         40s
zk-1      0/1       Pending   0         0s
zk-1      0/1       Pending   0         0s
zk-1      0/1       ContainerCreating   0         0s
zk-1      0/1       Running   0         18s
zk-1      1/1       Running   0         40s
zk-2      0/1       Pending   0         0s
zk-2      0/1       Pending   0         0s
zk-2      0/1       ContainerCreating   0         0s
zk-2      0/1       Running   0         19s
zk-2      1/1       Running   0         40s
```

각 파드의 A 레코드는 파드가 Ready 상태가 되면 입력된다. 따라서 ZooKeeper 서버의
FQDN은 단일 엔드포인트로 확인되고해당 엔드포인트는 `myid` 파일에 구성된 식별자를
가진고유한 ZooKeeper 서버가 된다.

```shell
zk-0.zk-hs.default.svc.cluster.local
zk-1.zk-hs.default.svc.cluster.local
zk-2.zk-hs.default.svc.cluster.local
```

이것은 ZooKeeper의 `zoo.cfg` 파일에 `servers` 속성이 정확히 구성된 앙상블로 나타
나는 것을 보증한다.

```shell
server.1=zk-0.zk-hs.default.svc.cluster.local:2888:3888
server.2=zk-1.zk-hs.default.svc.cluster.local:2888:3888
server.3=zk-2.zk-hs.default.svc.cluster.local:2888:3888
```

서버가 Zab 프로토콜로 값을 커밋 시도하면, 합의를 이루어 값을 커밋하거나(리더 �
출에 성공했고 나머지 두 개 파드도 Running과 Ready 상태라면) 실패한다(조건 중 하
나라도 충족하지 않으면). 다른 서버를 대신하여 쓰기를 승인하는 상태는 발생하지 않
는다.

### 앙상블 무결성 테스트

가장 기본적인 테스트는 한 ZooKeeper 서버에 데이터를 쓰고 다른 ZooKeeper 서버에서
데이터를 읽는 것이다.

아래 명령어는 앙상블 내에 `zk-0` 파드에서 `/hello` 경로로 `world`를 쓰는 스크립
트인 `zkCli.sh`를 실행한다.

```shell
kubectl exec zk-0 zkCli.sh create /hello world

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
Created /hello
```

`zk-1` 파드에서 데이터를 읽기 위해 다음 명령어를 이용하자.

```shell
kubectl exec zk-1 zkCli.sh get /hello
```

`zk-0`에서 생성한 그 데이터는 앙상블 내에 모든 서버에서사용할 수 있다.

```shell
WATCHER::

WatchedEvent state:SyncConnected type:None path:null
world
cZxid = 0x100000002
ctime = Thu Dec 08 15:13:30 UTC 2016
mZxid = 0x100000002
mtime = Thu Dec 08 15:13:30 UTC 2016
pZxid = 0x100000002
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 5
numChildren = 0
```

### 내구성있는 저장소 제공

[ZooKeeper 기본](#zookeeper-basics) 섹션에서 언급했듯이 ZooKeeper는 모든 항목을
내구성있는 WAL에 커밋하고 메모리 상태의 스냅샷을 저장 미디에에 주기적으로 저장한
다. 내구성을 제공하기 위해 WAL을 이용하는 것은복제된 상태 머신을 이루는 합의 프
로토콜에서이용하는 일반적인 기법이다.

[`kubectl delete`](/docs/reference/generated/kubectl/kubectl-commands/#delete)
명령을 이용하여 `zk` 스테이트풀셋을 삭제하자.

```shell
kubectl delete statefulset zk
statefulset.apps "zk" deleted
```

스테이트풀셋의 파드가 종료되는 것을 지켜보자.

```shell
kubectl get pods -w -l app=zk
```

`zk-0`이 완전히 종료되면 `CTRL-C`를 이용해 kubectl을 종료하자.

```shell
zk-2      1/1       Terminating   0         9m
zk-0      1/1       Terminating   0         11m
zk-1      1/1       Terminating   0         10m
zk-2      0/1       Terminating   0         9m
zk-2      0/1       Terminating   0         9m
zk-2      0/1       Terminating   0         9m
zk-1      0/1       Terminating   0         10m
zk-1      0/1       Terminating   0         10m
zk-1      0/1       Terminating   0         10m
zk-0      0/1       Terminating   0         11m
zk-0      0/1       Terminating   0         11m
zk-0      0/1       Terminating   0         11m
```

`zookeeper.yaml` 메니페스트를 다시 적용한다.

```shell
kubectl apply -f https://k8s.io/examples/application/zookeeper/zookeeper.yaml
```

`zk` 스테이트풀셋 오브젝트를 생성하지만, 매니페스트에 다른 API 오브젝트는 이미
존재하므로 수정되지 않는다.

스테이트풀셋 컨트롤러가 스테트풀셋의 파드를 재생성하는 것을 확인한다.

```shell
kubectl get pods -w -l app=zk
```

`zk-2` 파드가 Running과 Ready가 되면 `CTRL-C`를 이용하여 kubectl을 종료한다.

```shell
NAME      READY     STATUS    RESTARTS   AGE
zk-0      0/1       Pending   0          0s
zk-0      0/1       Pending   0         0s
zk-0      0/1       ContainerCreating   0         0s
zk-0      0/1       Running   0         19s
zk-0      1/1       Running   0         40s
zk-1      0/1       Pending   0         0s
zk-1      0/1       Pending   0         0s
zk-1      0/1       ContainerCreating   0         0s
zk-1      0/1       Running   0         18s
zk-1      1/1       Running   0         40s
zk-2      0/1       Pending   0         0s
zk-2      0/1       Pending   0         0s
zk-2      0/1       ContainerCreating   0         0s
zk-2      0/1       Running   0         19s
zk-2      1/1       Running   0         40s
```

아래 명령어로 [무결성 테스트](#앙상블-무결성-테스트)에서 입력한 값을 `zk-2` 파드
에서 얻어온다.

```shell
kubectl exec zk-2 zkCli.sh get /hello
```

`zk` 스테이트풀셋의 모든 파드를 종료하고 재생성했음에도, 앙상블은 여전히 원래 값
을 돌려준다.

```shell
WATCHER::

WatchedEvent state:SyncConnected type:None path:null
world
cZxid = 0x100000002
ctime = Thu Dec 08 15:13:30 UTC 2016
mZxid = 0x100000002
mtime = Thu Dec 08 15:13:30 UTC 2016
pZxid = 0x100000002
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 5
numChildren = 0
```

`zk` 스테이트풀셋의 `spec`에 `volumeClaimTemplates` 필드는 각 파드에 프로비전�
퍼시스턴트볼륨을 지정한다.

```yaml
volumeClaimTemplates:
  - metadata:
      name: datadir
      annotations:
        volume.alpha.kubernetes.io/storage-class: anything
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 20Gi
```

`스테이트풀셋` 컨트롤러는 `스테이트풀셋`의 각 파드에 대한
`퍼시스턴트볼륨클레임`을 생성한다.

다음 명령어를 이용하여 `스테이트풀셋`의 `퍼시스턴트볼륨클레임`을 살펴보자.

```shell
kubectl get pvc -l app=zk
```

`스테이트풀셋`의 파드를 재생성할 때에 파드의 퍼시스턴트볼륨도 다시 마운트한다.

```shell
NAME           STATUS    VOLUME                                     CAPACITY   ACCESSMODES   AGE
datadir-zk-0   Bound     pvc-bed742cd-bcb1-11e6-994f-42010a800002   20Gi       RWO           1h
datadir-zk-1   Bound     pvc-bedd27d2-bcb1-11e6-994f-42010a800002   20Gi       RWO           1h
datadir-zk-2   Bound     pvc-bee0817e-bcb1-11e6-994f-42010a800002   20Gi       RWO           1h
```

`스테이트풀셋`의 컨테이너 `template`의 `volumeMounts` 부분이 ZooKeeper 서버의 데
이터 디렉터리에 퍼시스턴트볼륨 마운트하는 내용이다.

```shell
volumeMounts:
        - name: datadir
          mountPath: /var/lib/zookeeper
```

`zk` 스테이트풀셋이 (재)스케줄링될 때 항상 동일한 `퍼시스턴트볼륨`을 ZooKeeper의
서버 디렉터리에 마운트한다. 파드를 재스케쥴할 때에도 ZooKeeper의 WAL을 통해 이뤄
진 모든 쓰기와모든 그 스냅샷도 내구성을 유지한다.

## 일관된 구성 보장하기

[리더 선출 촉진](#리더-선출-촉진)과 [합의 달성](#합의-달성) 섹션에서 알렸듯이,
ZooKeeper 앙상블에 서버는 리더 선출과 쿼럼을 구성하기 위한 일관된 설정이 필요하
다. 또한 Zab 프로토콜의 일관된 설정도 네트워크에 걸쳐 올바르게 동작하기 위해서필
요하다. 이 예시에서는 메니페스트에 구성을 직접 포함시켜서 일관된 구성을달성한다.

`zk` 스테이트풀셋을 살펴보자.

```shell
kubectl get sts zk -o yaml
…
command:
      - sh
      - -c
      - "start-zookeeper \
        --servers=3 \
        --data_dir=/var/lib/zookeeper/data \
        --data_log_dir=/var/lib/zookeeper/data/log \
        --conf_dir=/opt/zookeeper/conf \
        --client_port=2181 \
        --election_port=3888 \
        --server_port=2888 \
        --tick_time=2000 \
        --init_limit=10 \
        --sync_limit=5 \
        --heap=512M \
        --max_client_cnxns=60 \
        --snap_retain_count=3 \
        --purge_interval=12 \
        --max_session_timeout=40000 \
        --min_session_timeout=4000 \
        --log_level=INFO"
…
```

ZooKeeper 서버를 시작하는데 사용한 명령어는 커맨드라인 파라미터로 환경 구성을 전
달했다. 환경 변수를 이용하여서도 앙상블에 환경 구성을 전달할 수 있다.

### 로깅 설정하기

`zkGenConfig.sh` 스크립트로 생성된 파일 중 하나는 ZooKeeper의 로깅을 제어한다.
ZooKeeper는 [Log4j](http://logging.apache.org/log4j/2.x/)를 이용하며기본 로깅 구
성으로는 시간과 파일 크기 기준의 롤링 파일 어펜더를 사용한다.

`zk` `스테이트풀셋`의 한 파드에서 로깅 설정을 살펴보는 아래 명령어를 이용하자.

```shell
kubectl exec zk-0 cat /usr/etc/zookeeper/log4j.properties
```

아래 로깅 구성은 ZooKeeper가 모든 로그를 표준 출력 스트림으로 처리하게 한다.

```shell
zookeeper.root.logger=CONSOLE
zookeeper.console.threshold=INFO
log4j.rootLogger=${zookeeper.root.logger}
log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
log4j.appender.CONSOLE.Threshold=${zookeeper.console.threshold}
log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
log4j.appender.CONSOLE.layout.ConversionPattern=%d{ISO8601} [myid:%X{myid}] - %-5p [%t:%C{1}@%L] - %m%n
```

이는 컨테이너 내에서 안전하게 로깅하는 가장 단순한 방법이다. 표준 출력으로 애플
리케이션 로그를 작성하면, 쿠버네티스는 로그 로테이션을 처리한다. 또한 쿠버네티스
는 애플리케이션이 표준 출력과 표준 오류에 쓰인 로그로 인하여 로컬 저장 미디어가
고갈되지 않도록 보장하는 정상적인 보존 정책을 구현한다.

파드의 마지막 20줄의 로그를 가져오는
[`kubectl logs`](/docs/reference/generated/kubectl/kubectl-commands/#logs) 명령
을 이용하자.

```shell
kubectl logs zk-0 --tail 20
```

`kubectl logs`를 이용하거나 쿠버네티스 대시보드에서 표준 출력과 표준 오류로 쓰인
애플리케이션 로그를 볼 수 있다.

```shell
2016-12-06 19:34:16,236 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@827] - Processing ruok command from /127.0.0.1:52740
2016-12-06 19:34:16,237 [myid:1] - INFO  [Thread-1136:NIOServerCnxn@1008] - Closed socket connection for client /127.0.0.1:52740 (no session established for client)
2016-12-06 19:34:26,155 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxnFactory@192] - Accepted socket connection from /127.0.0.1:52749
2016-12-06 19:34:26,155 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@827] - Processing ruok command from /127.0.0.1:52749
2016-12-06 19:34:26,156 [myid:1] - INFO  [Thread-1137:NIOServerCnxn@1008] - Closed socket connection for client /127.0.0.1:52749 (no session established for client)
2016-12-06 19:34:26,222 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxnFactory@192] - Accepted socket connection from /127.0.0.1:52750
2016-12-06 19:34:26,222 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@827] - Processing ruok command from /127.0.0.1:52750
2016-12-06 19:34:26,226 [myid:1] - INFO  [Thread-1138:NIOServerCnxn@1008] - Closed socket connection for client /127.0.0.1:52750 (no session established for client)
2016-12-06 19:34:36,151 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxnFactory@192] - Accepted socket connection from /127.0.0.1:52760
2016-12-06 19:34:36,152 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@827] - Processing ruok command from /127.0.0.1:52760
2016-12-06 19:34:36,152 [myid:1] - INFO  [Thread-1139:NIOServerCnxn@1008] - Closed socket connection for client /127.0.0.1:52760 (no session established for client)
2016-12-06 19:34:36,230 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxnFactory@192] - Accepted socket connection from /127.0.0.1:52761
2016-12-06 19:34:36,231 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@827] - Processing ruok command from /127.0.0.1:52761
2016-12-06 19:34:36,231 [myid:1] - INFO  [Thread-1140:NIOServerCnxn@1008] - Closed socket connection for client /127.0.0.1:52761 (no session established for client)
2016-12-06 19:34:46,149 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxnFactory@192] - Accepted socket connection from /127.0.0.1:52767
2016-12-06 19:34:46,149 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@827] - Processing ruok command from /127.0.0.1:52767
2016-12-06 19:34:46,149 [myid:1] - INFO  [Thread-1141:NIOServerCnxn@1008] - Closed socket connection for client /127.0.0.1:52767 (no session established for client)
2016-12-06 19:34:46,230 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxnFactory@192] - Accepted socket connection from /127.0.0.1:52768
2016-12-06 19:34:46,230 [myid:1] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@827] - Processing ruok command from /127.0.0.1:52768
2016-12-06 19:34:46,230 [myid:1] - INFO  [Thread-1142:NIOServerCnxn@1008] - Closed socket connection for client /127.0.0.1:52768 (no session established for client)
```

쿠버네티스는 더 강력하지만 조금 복잡한 로그 통합을
[스택드라이버](/docs/tasks/debug-application-cluster/logging-stackdriver/)와
[Elasticsearch와 Kibana](/docs/tasks/debug-application-cluster/logging-elasticsearch-kibana/)를
지원한다. 클러스터 수준의 로그 적재(ship)와 통합을 위해서는 로그 순환과 적재를
위해
[사이드카](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns)
컨테이너를 배포하는 것을 고려한다.

### 권한 없는 사용자를 위해 구성하기

컨테이너 내부의 권한있는 유저로 애플리케이션을 실행할 수 있도록 하는최상의 방법
은 논쟁거리이다. 조직에서 애플리케이션을 권한 없는 사용자가 실행한다면, 진입점을
실행할 사용자를 제어하기 위해
[시큐리티컨텍스트](/docs/tasks/configure-pod-container/security-context/)를 이용
할 수 있다.

`zk` `스테이트풀셋`의 파드 `template`은 `SecurityContext`를 포함한다.

```yaml
securityContext:
  runAsUser: 1000
  fsGroup: 1000
```

파드 컨테이너에서 UID 1000은 ZooKeeper 사용자이며, GID 1000은 ZooKeeper의 그룹에
해당한다.

`zk-0` 파드에서 프로세스 정보를 얻어오자.

```shell
kubectl exec zk-0 -- ps -elf
```

`securityContext` 오브젝트의 `runAsUser` 필드 값이 1000 이므로루트 사용자로 실행
하는 대신 ZooKeeper 프로세스는 ZooKeeper 사용자로 실행된다.

```shell
F S UID        PID  PPID  C PRI  NI ADDR SZ WCHAN  STIME TTY          TIME CMD
4 S zookeep+     1     0  0  80   0 -  1127 -      20:46 ?        00:00:00 sh -c zkGenConfig.sh && zkServer.sh start-foreground
0 S zookeep+    27     1  0  80   0 - 1155556 -    20:46 ?        00:00:19 /usr/lib/jvm/java-8-openjdk-amd64/bin/java -Dzookeeper.log.dir=/var/log/zookeeper -Dzookeeper.root.logger=INFO,CONSOLE -cp /usr/bin/../build/classes:/usr/bin/../build/lib/*.jar:/usr/bin/../share/zookeeper/zookeeper-3.4.9.jar:/usr/bin/../share/zookeeper/slf4j-log4j12-1.6.1.jar:/usr/bin/../share/zookeeper/slf4j-api-1.6.1.jar:/usr/bin/../share/zookeeper/netty-3.10.5.Final.jar:/usr/bin/../share/zookeeper/log4j-1.2.16.jar:/usr/bin/../share/zookeeper/jline-0.9.94.jar:/usr/bin/../src/java/lib/*.jar:/usr/bin/../etc/zookeeper: -Xmx2G -Xms2G -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false org.apache.zookeeper.server.quorum.QuorumPeerMain /usr/bin/../etc/zookeeper/zoo.cfg
```

기본적으로 파드의 퍼시스턴트볼륨은 ZooKeeper 서버의 데이터 디렉터리에 마운트되�
, 루트 사용자만이 접근 가능하다. 이 구성은 ZooKeeper 프로세스가 WAL에 기록하�
스냅샷을 저장하는 것을 방지한다.

`zk-0` 파드의 ZooKeeper 데이터 디렉터리의 권한을 얻어오는 아래 명령어를 이용하자
.

```shell
kubectl exec -ti zk-0 -- ls -ld /var/lib/zookeeper/data
```

`securityContext` 오브젝트의 `fsGroup` 필드 값이 1000 이므로, 파드의 퍼시스턴트
볼륨의 소유권은 ZooKeeper 그룹으로 지정되어 ZooKeeper 프로세스에서 읽고 쓸 수 있
다.

```shell
drwxr-sr-x 3 zookeeper zookeeper 4096 Dec  5 20:45 /var/lib/zookeeper/data
```

## ZooKeeper 프로세스 관리하기

[ZooKeeper 문서](https://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_supervision)에
서는 "ZooKeeper의 서버 프로세스(JVM)을 관리할감독 프로세스를 필요할 것이다."라�
말한다. 와치독(감독 프로세스)를 활용하여 실패한 프로세스를 재시작하는 것은 분산
시스템에서일반적인 방식이다. 쿠버네티스에서 애플리케이션을 배포할 때에는감독 프
로세스로 외부 유틸리티를 사용하기보다 쿠버네티스를 애플리케이션의와치독으로서 사
용해야 한다.

### 앙상블 관리하기

`zk` `스테이트풀셋`은 `RollingUpdate` 업데이트 전략을 이용하도록 구성되었다.

`kubectl patch`로 서버에 할당된 `cpu` 수를 갱신할 수 있다.

```shell
kubectl patch sts zk --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/resources/requests/cpu", "value":"0.3"}]'

statefulset.apps/zk patched
```

업데이트 상황을 지켜보기 위해 `kubectl rollout status` 이용하자.

```shell
kubectl rollout status sts/zk

waiting for statefulset rolling update to complete 0 pods at revision zk-5db4499664...
Waiting for 1 pods to be ready...
Waiting for 1 pods to be ready...
waiting for statefulset rolling update to complete 1 pods at revision zk-5db4499664...
Waiting for 1 pods to be ready...
Waiting for 1 pods to be ready...
waiting for statefulset rolling update to complete 2 pods at revision zk-5db4499664...
Waiting for 1 pods to be ready...
Waiting for 1 pods to be ready...
statefulset rolling update complete 3 pods at revision zk-5db4499664...
```

이것은 파드를 역순으로 한 번에 하나씩 종료하고, 새로운 구성으로 재생성한다. 이는
롤링업데이트 동안에 쿼럼을 유지하도록 보장한다.

이력과 이전 구성을 보기 위해 `kubectl rollout history` 명령을 이용하자.

```shell
kubectl rollout history sts/zk

statefulsets "zk"
REVISION
1
2
```

수정사항을 롤백하기 위해 `kubectl rollout undo` 명령을 이용하자.

```shell
kubectl rollout undo sts/zk

statefulset.apps/zk rolled back
```

### 프로세스 장애 관리하기

[재시작 정책](/ko/docs/concepts/workloads/pods/pod-lifecycle/#재시작-정책)은쿠버
네티스가 파드 내에 컨테이너의 진입점에서 프로세스 실패를 어떻게 다루는지 제어한
다. `스테이트풀셋`의 파드에서 오직 적절한 `재시작 정책`는 Always이며이것이 기본
값이다. 상태가 유지되는 애플리케이션을 위해기본 정책을 **절대로** 변경하지 말자.

`zk-0` 파드에서 실행 중인 ZooKeeper 서버에서 프로세스 트리를 살펴보기 위해 다음
명령어를 이용하자.

```shell
kubectl exec zk-0 -- ps -ef
```

컨테이너의 엔트리 포인트로 PID 1 인 명령이 사용되었으며 ZooKeeper 프로세스는 엔
트리 포인트의 자식 프로세스로 PID 27 이다.

```shell
UID        PID  PPID  C STIME TTY          TIME CMD
zookeep+     1     0  0 15:03 ?        00:00:00 sh -c zkGenConfig.sh && zkServer.sh start-foreground
zookeep+    27     1  0 15:03 ?        00:00:03 /usr/lib/jvm/java-8-openjdk-amd64/bin/java -Dzookeeper.log.dir=/var/log/zookeeper -Dzookeeper.root.logger=INFO,CONSOLE -cp /usr/bin/../build/classes:/usr/bin/../build/lib/*.jar:/usr/bin/../share/zookeeper/zookeeper-3.4.9.jar:/usr/bin/../share/zookeeper/slf4j-log4j12-1.6.1.jar:/usr/bin/../share/zookeeper/slf4j-api-1.6.1.jar:/usr/bin/../share/zookeeper/netty-3.10.5.Final.jar:/usr/bin/../share/zookeeper/log4j-1.2.16.jar:/usr/bin/../share/zookeeper/jline-0.9.94.jar:/usr/bin/../src/java/lib/*.jar:/usr/bin/../etc/zookeeper: -Xmx2G -Xms2G -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false org.apache.zookeeper.server.quorum.QuorumPeerMain /usr/bin/../etc/zookeeper/zoo.cfg
```

다른 터미널에서 다음 명령어로 `zk` `스테이트풀셋`의 파드를 확인한다.

```shell
kubectl get pod -w -l app=zk
```

또 다른 터미널에서 다음 명령어로 `zk-0` 파드의 ZooKeeper 프로세스를 종료시킨다.

```shell
kubectl exec zk-0 -- pkill java
```

ZooKeeper 프로세스의 종료는 부모 프로세스의 종료를 일으킨다. 컨테이너
`재시작정책`이 Always이기 때문에 부모 프로세스를 재시작했다.

```shell
NAME      READY     STATUS    RESTARTS   AGE
zk-0      1/1       Running   0          21m
zk-1      1/1       Running   0          20m
zk-2      1/1       Running   0          19m
NAME      READY     STATUS    RESTARTS   AGE
zk-0      0/1       Error     0          29m
zk-0      0/1       Running   1         29m
zk-0      1/1       Running   1         29m
```

애플리케이션이 스크립트(`zkServer.sh` 같은)를 애플리케이션의 비지니스 로직을 구
현한 프로세스를 시작하기 위해 이용한다면, 그 스크립트는 자식 프로세스와 함께 반
드시 종료되어야 한다. 이는 쿠버네티스가 애플리케이션의 비지니스 로직을 구현한 프
로세스가 실패할 때에애플리케이션 컨테이너를 재시작하는 것을 보증한다.

### 활성도(Liveness) 테스트하기

실패한 애플리케이션을 재시작하도록 구성하는 것은 분산 시스템을건강하게 유지하는
데 충분하지 않다. 시스템의 프로세스는 살아있지만응답이 없을 수 있고, 혹은 다른
건강하지 않은 경우의 시나리오가 있다. 애플리케이션 프로세스가 건강하지 않고 재시
작해야만 한다는 것을쿠버네티스에게 알리도록 활성도 검사를 이용해야 한다.

`zk` `스테이트풀셋`에 파드 `template`에 활성도 검사를 명시한다. ``

```yaml
livenessProbe:
  exec:
    command:
      - sh
      - -c
      - "zookeeper-ready 2181"
  initialDelaySeconds: 15
  timeoutSeconds: 5
```

검사는 ZooKeeper의 `ruok` 4 글자 단어를 이용해서 서버의 건강을 테스트하는배쉬 스
크립트를 호출한다.

```bash
OK=$(echo ruok | nc 127.0.0.1 $1)
if [ "$OK" == "imok" ]; then
    exit 0
else
    exit 1
fi
```

한 터미널에서 `zk` 스테이트풀셋의 파드를 지켜보기 위해 다음 명령어를 이용하자.

```shell
kubectl get pod -w -l app=zk
```

다른 창에서 `zk-0` 파드의 파일시스템에서 `zkOk.sh` 스크립트를 삭제하기 위해 다음
명령어를 이용하자.

```shell
kubectl exec zk-0 -- rm /usr/bin/zookeeper-ready
```

ZooKeeper의 활성도 검사에 실패하면, 쿠버네티스는 자동으로 프로세스를 재시작하여
앙상블에 건강하지 않은 프로세스를재시작하는 것을 보증한다.

```shell
kubectl get pod -w -l app=zk

NAME      READY     STATUS    RESTARTS   AGE
zk-0      1/1       Running   0          1h
zk-1      1/1       Running   0          1h
zk-2      1/1       Running   0          1h
NAME      READY     STATUS    RESTARTS   AGE
zk-0      0/1       Running   0          1h
zk-0      0/1       Running   1         1h
zk-0      1/1       Running   1         1h
```

### 준비도 테스트

준비도는 활성도와 동일하지 않다. 프로세스가 살아 있다면, 스케쥴링되고 건강하다.
프로세스가 준비되면 입력을 처리할 수 있다. 활성도는 필수적이나 준비도의 조건으로
는충분하지 않다. 몇몇 경우특별히 초기화와 종료 시에 프로세스는 살아있지만준비되
지 않을 수 있다.

준비도 검사를 지정하면, 쿠버네티스는 준비도가 통과할 때까지애플리케이션 프로세스
가 네트워크 트래픽을 수신하지 않게 한다.

ZooKeeper 서버에서는 준비도가 활성도를 내포한다. 그러므로 `zookeeper.yaml` 메니
페스트에서준비도 검사는 활성도 검사와 동일하다.

```yaml
readinessProbe:
  exec:
    command:
      - sh
      - -c
      - "zookeeper-ready 2181"
  initialDelaySeconds: 15
  timeoutSeconds: 5
```

활성도와 준비도 검사가 동일함에도 둘 다 지정하는 것은 중요하다. 이는 ZooKeeper
앙상블에 건강한 서버만 아니라네트워크 트래픽을 수신하는 것을 보장한다.

## 노드 실패 방지

ZooKeeper는 변조된 데이터를 성공적으로 커밋하기 위한 서버의 쿼럼이 필요하다. 3개
의 서버 앙상블에서 성공적으로 저장하려면 2개 서버는 반드시 건강해야 한다. 쿼럼
기반 시스템에서, 멤버는 가용성을 보장하는실패 영역에 걸쳐 배포된다. 중단을 방지
하기 위해 개별 시스템의 손실로 인해 모범 사례에서는 동일한 시스템에여러 인스턴스
의 응용 프로그램을 함께 배치하는 것을 배제한다.

기본적으로 쿠버네티스는 동일 노드상에 `스테이트풀셋`의 파드를 위치시킬 수 있다.
생성한 3개의 서버 앙상블에서 2개의 서버가 같은 노드에 있다면, 그 노드는 실패하�
ZooKeeper 서비스 클라이언트는 그 파드들의 최소 하나가 재스케쥴링될 때까지 작동
중단을 경험할 것이다.

노드 실패하는 사건 중에도 중요 시스템의 프로세스가 재스케줄될 수 있게항상 추가적
인 용량을 프로비전해야 한다. 그렇게 하면 쿠버네티스 스케줄러가 ZooKeeper 서버 하
나를 다시 스케줄하는 동안까지만 작동 중단될 것이다. 그러나 서비스에서 노드 실패
로 인한 다운타임을 방지하려 한다면, `파드안티어피니티`를 설정해야 한다.

`zk` `스테이트풀셋`의 파드의 노드를 알아보기 위해 다음 명령어를 이용하자.

```shell
for i in 0 1 2; do kubectl get pod zk-$i --template {{.spec.nodeName}}; echo ""; done
```

`zk` `스테이트풀셋`에 모든 파드는 다른 노드에 배포된다.

```shell
kubernetes-node-cxpk
kubernetes-node-a5aq
kubernetes-node-2g2d
```

이는 `zk` `스테이트풀셋`의 파드에 `파드안티어피니티(PodAntiAffinity)`를 지정했기
때문이다.

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - key: "app"
              operator: In
              values:
                - zk
        topologyKey: "kubernetes.io/hostname"
```

`requiredDuringSchedulingIgnoredDuringExecution` 필드는쿠버네티스 스케줄러에
`topologyKey`로 정의된 도메인에서 `app`이 `zk`라고 레이블링된두 개 파드가 위치하
지 않도록 한다. `topologyKey` `kubernetes.io/hostname`은 도메인이 개별 노드임을
나타낸다. 다른 규칙과 레이블, 셀렉터를 사용하여앙상블을 물리적인, 네트워크, 전원
장애 분야에 걸쳐 확산하도록 이 기법을 확장할 수 있다.

## 생존 유지

**이 섹션에서는 노드를 통제(cordon)하고 비운다(drain). 공유된 클러스터에서 이 튜
토리얼을 진행한다면, 다른 테넌트에 부정적인 영향을 비치지 않음을 보증해야 한다
.**

이전 섹션은 계획되지 않은 노드 실패에서 살아남도록어떻게 파드를 확산할 것인가에
대해 알아보았다. 그러나 계획된 점검으로 인해 발생하는 일시적인 노드 실패에 대한
계획도 필요하다.

클러스터에서 다음 명령으로 노드를 살펴보자.

```shell
kubectl get nodes
```

[`kubectl cordon`](/docs/reference/generated/kubectl/kubectl-commands/#cordon)을
이용하여 클러스터 내에 4개 노드를 제외하고 다른 모든 노드를 통제해보자.

```shell
kubectl cordon <노드-이름>
```

`zk-pdb` `PodDisruptionBudget`을 살펴보고자 이 명령어를 이용하자.

```shell
kubectl get pdb zk-pdb
```

`max-unavailable` 필드는 쿠버네티스가 `zk` `스테이트풀셋`에서 최대 1개의 파드는
언제든지 가용하지 않을 수 있음을 나타낸다.

```shell
NAME      MIN-AVAILABLE   MAX-UNAVAILABLE   ALLOWED-DISRUPTIONS   AGE
zk-pdb    N/A             1                 1
```

한 터미널에서 `zk` `스테이트풀셋`의 파드를 지켜보는 이 명령어를 이용하자.

```shell
kubectl get pods -w -l app=zk
```

다른 터미널에서 현재 스케쥴되는 파드의 노드를 살펴보자.

```shell
for i in 0 1 2; do kubectl get pod zk-$i --template {{.spec.nodeName}}; echo ""; done

kubernetes-node-pb41
kubernetes-node-ixsl
kubernetes-node-i4c4
```

`zk-0`파드가 스케쥴되는 노드를 통제하기 위해
[`kubectl drain`](/docs/reference/generated/kubectl/kubectl-commands/#drain)를
이용하자.

```shell
kubectl drain $(kubectl get pod zk-0 --template {{.spec.nodeName}}) --ignore-daemonsets --force --delete-local-data
node "kubernetes-node-group-pb41" cordoned

WARNING: Deleting pods not managed by ReplicationController, ReplicaSet, Job, or DaemonSet: fluentd-cloud-logging-kubernetes-node-group-pb41, kube-proxy-kubernetes-node-group-pb41; Ignoring DaemonSet-managed pods: node-problem-detector-v0.1-o5elz
pod "zk-0" deleted
node "kubernetes-node-group-pb41" drained
```

클러스터에 4개 노드가 있기 때문에 `kubectl drain`이 성공하여 `zk-0`을 다른 노드
로 재스케쥴링 된다.

```shell
NAME      READY     STATUS    RESTARTS   AGE
zk-0      1/1       Running   2          1h
zk-1      1/1       Running   0          1h
zk-2      1/1       Running   0          1h
NAME      READY     STATUS        RESTARTS   AGE
zk-0      1/1       Terminating   2          2h
zk-0      0/1       Terminating   2         2h
zk-0      0/1       Terminating   2         2h
zk-0      0/1       Terminating   2         2h
zk-0      0/1       Pending   0         0s
zk-0      0/1       Pending   0         0s
zk-0      0/1       ContainerCreating   0         0s
zk-0      0/1       Running   0         51s
zk-0      1/1       Running   0         1m
```

계속해서 `스테이트풀셋`의 파드를 첫 터미널에서 지켜보고 `zk-1` 이 스케쥴된 노드
를 비워보자.

```shell
kubectl drain $(kubectl get pod zk-1 --template {{.spec.nodeName}}) --ignore-daemonsets --force --delete-local-data "kubernetes-node-ixsl" cordoned

WARNING: Deleting pods not managed by ReplicationController, ReplicaSet, Job, or DaemonSet: fluentd-cloud-logging-kubernetes-node-ixsl, kube-proxy-kubernetes-node-ixsl; Ignoring DaemonSet-managed pods: node-problem-detector-v0.1-voc74
pod "zk-1" deleted
node "kubernetes-node-ixsl" drained
```

`zk-1` 파드는 스케쥴되지 않는데 이는 `zk` `스테이트풀셋`이 오직 2개 노드가 스케
쥴되도록 파드를 위치시키는 것을 금하는 `파드안티어피니티` 규칙을 포함하였기 때문
이고 그 파드는 Pending 상태로 남을 것이다.

```shell
kubectl get pods -w -l app=zk

NAME      READY     STATUS    RESTARTS   AGE
zk-0      1/1       Running   2          1h
zk-1      1/1       Running   0          1h
zk-2      1/1       Running   0          1h
NAME      READY     STATUS        RESTARTS   AGE
zk-0      1/1       Terminating   2          2h
zk-0      0/1       Terminating   2         2h
zk-0      0/1       Terminating   2         2h
zk-0      0/1       Terminating   2         2h
zk-0      0/1       Pending   0         0s
zk-0      0/1       Pending   0         0s
zk-0      0/1       ContainerCreating   0         0s
zk-0      0/1       Running   0         51s
zk-0      1/1       Running   0         1m
zk-1      1/1       Terminating   0         2h
zk-1      0/1       Terminating   0         2h
zk-1      0/1       Terminating   0         2h
zk-1      0/1       Terminating   0         2h
zk-1      0/1       Pending   0         0s
zk-1      0/1       Pending   0         0s
```

계속해서 스테이트풀셋의 파드를 지켜보고 `zk-2`가 스케줄된 노드를 비워보자.

```shell
kubectl drain $(kubectl get pod zk-2 --template {{.spec.nodeName}}) --ignore-daemonsets --force --delete-local-data
node "kubernetes-node-i4c4" cordoned

WARNING: Deleting pods not managed by ReplicationController, ReplicaSet, Job, or DaemonSet: fluentd-cloud-logging-kubernetes-node-i4c4, kube-proxy-kubernetes-node-i4c4; Ignoring DaemonSet-managed pods: node-problem-detector-v0.1-dyrog
WARNING: Ignoring DaemonSet-managed pods: node-problem-detector-v0.1-dyrog; Deleting pods not managed by ReplicationController, ReplicaSet, Job, or DaemonSet: fluentd-cloud-logging-kubernetes-node-i4c4, kube-proxy-kubernetes-node-i4c4
There are pending pods when an error occurred: Cannot evict pod as it would violate the pod's disruption budget.
pod/zk-2
```

kubectl 을 종료하기 위해 `CTRL-C`를 이용하자.

`zk-2`를 추출하는 것은 `zk-budget`을 위반하기 때문에 셋째 노드를 비울 수 없다.
그러나 그 노드는 통제 상태로 남는다.

`zk-0`에서 온전성 테스트 때에 입력한 값을 가져오는 `zkCli.sh`를 이용하자.

```shell
kubectl exec zk-0 zkCli.sh get /hello
```

`PodDisruptionBudget`이 존중되기 때문에 서비스는 여전히 가용하다.

```shell
WatchedEvent state:SyncConnected type:None path:null
world
cZxid = 0x200000002
ctime = Wed Dec 07 00:08:59 UTC 2016
mZxid = 0x200000002
mtime = Wed Dec 07 00:08:59 UTC 2016
pZxid = 0x200000002
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 5
numChildren = 0
```

[`kubectl uncordon`](/docs/reference/generated/kubectl/kubectl-commands/#uncordon)
이용하여 첫 노드의 통제를 풀자.

```shell
kubectl uncordon kubernetes-node-pb41

node "kubernetes-node-pb41" uncordoned
```

`zk-1`은 이 노드에서 재스케쥴된다. `zk-1`이 Running과 Ready가 될 때까지 기다리자
.

```shell
kubectl get pods -w -l app=zk

NAME      READY     STATUS    RESTARTS   AGE
zk-0      1/1       Running   2          1h
zk-1      1/1       Running   0          1h
zk-2      1/1       Running   0          1h
NAME      READY     STATUS        RESTARTS   AGE
zk-0      1/1       Terminating   2          2h
zk-0      0/1       Terminating   2         2h
zk-0      0/1       Terminating   2         2h
zk-0      0/1       Terminating   2         2h
zk-0      0/1       Pending   0         0s
zk-0      0/1       Pending   0         0s
zk-0      0/1       ContainerCreating   0         0s
zk-0      0/1       Running   0         51s
zk-0      1/1       Running   0         1m
zk-1      1/1       Terminating   0         2h
zk-1      0/1       Terminating   0         2h
zk-1      0/1       Terminating   0         2h
zk-1      0/1       Terminating   0         2h
zk-1      0/1       Pending   0         0s
zk-1      0/1       Pending   0         0s
zk-1      0/1       Pending   0         12m
zk-1      0/1       ContainerCreating   0         12m
zk-1      0/1       Running   0         13m
zk-1      1/1       Running   0         13m
```

`zk-2`가 스케쥴된 노드를 비워보자.

```shell
kubectl drain $(kubectl get pod zk-2 --template {{.spec.nodeName}}) --ignore-daemonsets --force --delete-local-data
```

출력은

```
node "kubernetes-node-i4c4" already cordoned
WARNING: Deleting pods not managed by ReplicationController, ReplicaSet, Job, or DaemonSet: fluentd-cloud-logging-kubernetes-node-i4c4, kube-proxy-kubernetes-node-i4c4; Ignoring DaemonSet-managed pods: node-problem-detector-v0.1-dyrog
pod "heapster-v1.2.0-2604621511-wht1r" deleted
pod "zk-2" deleted
node "kubernetes-node-i4c4" drained
```

이번엔 `kubectl drain` 이 성공한다.

`zk-2`가 재스케줄되도록 두 번째 노드의 통제를 풀어보자.

```shell
kubectl uncordon kubernetes-node-ixsl
```

```
node "kubernetes-node-ixsl" uncordoned
```

`kubectl drain`을 `PodDisruptionBudget`과 결합하면 유지보수 중에도 서비스를 가용
하게 할 수 있다. drain으로 노드를 통제하고 유지보수를 위해 노드를 오프라인하기
전에 파드를 추출하기 위해 사용한다면 서비스는 혼란 예산을 표기한 서비스는 그 예
산이 존중은 존중될 것이다. 파드가 즉각적으로 재스케줄 할 수 있도록 항상 중요 서
비스를 위한 추가 용량을 할당해야 한다. {{% /capture %}}

{{% capture cleanup %}}

- `kubectl uncordon`은 클러스터 내에 모든 노드를 통제 해제한다.
- 이 튜토리얼에서 사용한 퍼시스턴트 볼륨을 위한 퍼시스턴트 스토리지 미디어를 삭
  제하자. 귀하의 환경과 스토리지 구성과 프로비저닝 방법에서 필요한 절차를 따라서
  모든 스토리지가 재확보되도록 하자. {{% /capture %}}
