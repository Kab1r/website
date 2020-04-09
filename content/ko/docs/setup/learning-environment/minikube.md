---
title: Minikube로 쿠버네티스 설치
weight: 30
content_template: templates/concept
---

{{% capture overview %}}

Minikube는 쿠버네티스를 로컬에서 쉽게 실행하는 도구이다. Minikube는 매일 쿠버네
티스를 사용하거나 개발하려는 사용자들을 위해 가상 머신(VM) 이나 노트북에서 단일
노드 쿠버네티스 클러스터를 실행한다.

{{% /capture %}}

{{% capture body %}}

## Minikube 특징

Minikube는 다음과 같은 쿠버네티스의 기능을 제공한다.

- DNS
- 노드 포트
- 컨피그 맵과 시크릿
- 대시보드
- 컨테이너 런타임: [Docker](https://www.docker.com/), [CRI-O](https://cri-o.io/)
  와 [containerd](https://github.com/containerd/containerd)
- CNI(Container Network Interface) 사용
- 인그레스

## 설치

[Minikube 설치](/ko/docs/tasks/tools/install-minikube/) 항목을 보자.

## 빠른 시작

여기서 기술하는 간단한 데모는 어떻게 로컬에서 Minikube를 시작하고, 사용하고 삭제
하는지를 안내한다. 다음의 주어진 단계를 따라서 Minikube를 시작하고 탐구한다.

1. Minikube를 시작하고 클러스터를 생성

   ```shell
   minikube start
   ```

   결과는 다음과 비슷하다.

   ```
   Starting local Kubernetes cluster...
   Running pre-create checks...
   Creating machine...
   Starting local Kubernetes cluster...
   ```

   특정 쿠버네티스 버전, VM, 컨테이너 런타임 상에서 클러스터를 시작하기 위한 보
   다 상세한 정보는 [클러스터 시작하기](#클러스터-시작하기)를 참조한다.

2. 이제, kubectl을 통해서 클러스터와 상호작용할 수 있다. 보다 상세한 정보는
   [클러스터와 상호 작용하기](#클러스터와-상호-작용하기)를 참조한다.

   단순한 HTTP 서버인 `echoserver` 이미지를 사용해서 쿠버네티스 디플로이먼트를
   만들고 `--port`를 이용해서 8080 포트로 노출해보자.

   ```shell
   kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.10
   ```

   결과는 다음과 비슷하다.

   ```
   deployment.apps/hello-minikube created
   ```

3. `hello-minikube` 디플로이먼트에 액세스하기 위해, 서비스로 노출시킨다.

   ```shell
   kubectl expose deployment hello-minikube --type=NodePort --port=8080
   ```

   `--type=NodePort` 옵션은 서비스 타입을 지정한다.

   결과는 다음과 비슷하다.

   ```
   service/hello-minikube exposed
   ```

4. `hello-minikube` 파드가 이제 시작되었지만 노출된 서비스를 통해서 접근하기 전
   에 파드가 뜨기를 기다려야한다.

   파드가 떠서 구동되고 있는지 확인한다.

   ```shell
   kubectl get pod
   ```

   출력에서 `STATUS`가 `ContainerCreating`으로 나타나는 경우, 파드는 아직 생성
   중이다.

   ```
   NAME                              READY     STATUS              RESTARTS   AGE
   hello-minikube-3383150820-vctvh   0/1       ContainerCreating   0          3s
   ```

   출력에서 `STATUS`가 `Running`으로 나타나는 경우, 파드는 이제 떠서 기동 중이다
   .

   ```
   NAME                              READY     STATUS    RESTARTS   AGE
   hello-minikube-3383150820-vctvh   1/1       Running   0          13s
   ```

5. 서비스 상세를 보기 위해서 노출된 서비스의 URL을 얻는다.
   ```shell
   minikube service hello-minikube --url
   ```
6. 로컬 클러스터의 상세를 보기위해서, 출력에서 얻은 URL을 브라우저에 복사해서 붙
   여 넣는다.

   출력은 다음과 비슷하다.

   ```
   Hostname: hello-minikube-7c77b68cff-8wdzq

   Pod Information:
       -no pod information available-

   Server values:
       server_version=nginx: 1.13.3 - lua: 10008

   Request Information:
       client_address=172.17.0.1
       method=GET
       real path=/
       query=
       request_version=1.1
       request_scheme=http
       request_uri=http://192.168.99.100:8080/

   Request Headers:
       accept=*/*
       host=192.168.99.100:30674
       user-agent=curl/7.47.0

   Request Body:
       -no body in request-
   ```

   서비스나 클러스터가 더 이상 구동되지 않도록 하려면, 삭제한다.

7. `hello-minikube` 서비스 삭제
   ```shell
   kubectl delete services hello-minikube
   ```
   출력은 다음과 비슷하다.
   ```
   service "hello-minikube" deleted
   ```
8. `hello-minikube` 디플로이먼트 삭제
   ```shell
   kubectl delete deployment hello-minikube
   ```
   출력은 다음과 비슷하다.
   ```
   deployment.extensions "hello-minikube" deleted
   ```
9. 로컬 Minikube 클러스터 중지
   ```shell
   minikube stop
   ```
   출력은 다음과 비슷하다.
   ```
   Stopping "minikube"...
   "minikube" stopped.
   ```
   보다 상세한 정보는 [클러스터 중지하기](#클러스터-중지하기)를 참조한다.
10. 로컬 Minikube 클러스터 삭제
    ```shell
    minikube delete
    ```
    출력은 다음과 비슷하다.
    ```
    Deleting "minikube" ...
    The "minikube" cluster has been deleted.
    ```
    보다 상세한 정보는 [Deleting a cluster](#클러스터-삭제하기)를 참조한다.

## 클러스터 관리하기

### 클러스터 시작하기

클러스터를 시작하기 위해서 `minikube start` 커멘드를 사용할 수 있다. 이 커멘드는
단일 노드 쿠버네티스 클러스터를 구동하는 가상 머신을 생성하고 구성한다. 이 커멘
드는 또한 [kubectl](/docs/user-guide/kubectl-overview/)도 설정해서 클러스터와 통
신할 수 있도록 한다.

{{< note >}} 웹 프록시 뒤에 있다면, `minikube start` 커맨드에 해당 정보를 전달해
야 한다.

```shell
https_proxy=<my proxy> minikube start --docker-env http_proxy=<my proxy> --docker-env https_proxy=<my proxy> --docker-env no_proxy=192.168.99.0/24
```

불행하게도, 환경 변수 설정만으로는 되지 않는다.

Minikube는 또한 "minikube" 컨텍스트를 생성하고 이를 kubectl의 기본값으로 설정한
다. 이 컨텍스트로 돌아오려면, 다음의 코멘드를 입력한다.
`kubectl config use-context minikube`. {{< /note >}}

#### 쿠버네티스 버전 지정하기

`minikube start` 코멘드에 `--kubernetes-version` 문자열을 추가해서 Minikube에서
사용할 쿠버네티스 버전을 지정할 수 있다. 예를 들어 버전
{{< param "fullversion" >}}를 구동하려면, 다음과 같이 실행한다.

```
minikube start --kubernetes-version {{< param "fullversion" >}}
```

#### VM 드라이버 지정하기

`minikube start` 코멘드에 `--driver=<enter_driver_name>` 플래그를 추가해서 VM 드
라이버를 변경할 수 있다. 코멘드를 예를 들면 다음과 같다.

```shell
minikube start --driver=<driver_name>
```

Minikube는 다음의 드라이버를 지원한다. {{< note >}} 지원되는 드라이버와 플러그인
설치 방법에 대한 보다 상세한 정보는
[드라이버](https://minikube.sigs.k8s.io/docs/reference/drivers/)를 참조한다.
{{< /note >}}

- virtualbox
- vmwarefusion
- docker (EXPERIMENTAL)
- kvm2
  ([드라이버 설치](https://minikube.sigs.k8s.io/docs/reference/drivers/kvm2/))
- hyperkit
  ([드라이버 설치](https://minikube.sigs.k8s.io/docs/reference/drivers/hyperkit/))
- hyperv
  ([드라이버 설치](https://minikube.sigs.k8s.io/docs/reference/drivers/hyperv/))
  다음 IP는 동적이며 변경할 수 있다. `minikube ip`로 알아낼 수 있다.
- vmware
  ([드라이버 설치](https://minikube.sigs.k8s.io/docs/reference/drivers/vmware/))
  (VMware unified driver)
- parallels
  ([드라이버 설치](https://minikube.sigs.k8s.io/docs/reference/drivers/parallels/))
- none (쿠버네티스 컴포넌트를 가상 머신이 아닌 호스트 상에서 구동한다. 리눅스를
  실행중이어야 하고, {{< glossary_tooltip term_id="docker" >}}가 설치되어야 한다
  .)

{{< caution >}} `none` 드라이버를 사용한다면 일부 쿠버네티스 컴포넌트는 Minikube
환경 외부에 있는 부작용이 있는 권한을 가진 컨테이너로 실행된다. 이런 부작용은 개
인용 워크스테이션에는 `none` 드라이버가 권장하지 않는 것을 의미 한다.
{{< /caution >}}

#### 대안적인 컨테이너 런타임 상에서 클러스터 시작하기

Minikube를 다음의 컨테이너 런타임에서 기동할 수 있다.
{{< tabs name="container_runtimes" >}} {{% tab name="containerd" %}}
[containerd](https://github.com/containerd/containerd)를 컨테이너 런타임으로 사
용하려면, 다음을 실행한다.

```bash
minikube start \
    --network-plugin=cni \
    --enable-default-cni \
    --container-runtime=containerd \
    --bootstrapper=kubeadm
```

혹은 확장 버전을 사용할 수 있다.

```bash
minikube start \
    --network-plugin=cni \
    --enable-default-cni \
    --extra-config=kubelet.container-runtime=remote \
    --extra-config=kubelet.container-runtime-endpoint=unix:///run/containerd/containerd.sock \
    --extra-config=kubelet.image-service-endpoint=unix:///run/containerd/containerd.sock \
    --bootstrapper=kubeadm
```

{{% /tab %}} {{% tab name="CRI-O" %}} [CRI-O](https://cri-o.io/)를 컨테이너 런타
임으로 사용하려면, 다음을 실행한다.

```bash
minikube start \
    --network-plugin=cni \
    --enable-default-cni \
    --container-runtime=cri-o \
    --bootstrapper=kubeadm
```

혹은 확장 버전을 사용할 수 있다.

```bash
minikube start \
    --network-plugin=cni \
    --enable-default-cni \
    --extra-config=kubelet.container-runtime=remote \
    --extra-config=kubelet.container-runtime-endpoint=/var/run/crio.sock \
    --extra-config=kubelet.image-service-endpoint=/var/run/crio.sock \
    --bootstrapper=kubeadm
```

{{% /tab %}} {{< /tabs >}}

#### Docker 데몬 재사용을 통한 로컬 이미지 사용하기

쿠버네티스 단일 VM을 사용하면 Minikube에 내장된 도커 데몬을 재사용하기에 매우 간
편하다. 이 경우는 호스트 장비에 도커 레지스트리를 설치하고 이미지를 푸시할 필요
가 없다. 또 로컬에서 빠르게 실행할 수 있는데 이는 Minikube와 동일한 도커 데몬 안
에서 이미지를 빌드하기 때문이다.

{{< note >}} Docker 이미지를 'latest'가 아닌 다른 태그로 태그했는지 확인하고 이
미지를 풀링할 때에는 그 태그를 이용한다. 혹시 이미지 태그 버전을 지정하지 않았다
면, 기본값은 `:latest`이고 이미지 풀링 정책은 `Always`가 가정하나, 만약 기본
Docker 레지스트리(보통 DockerHub)에 해당 Docker 이미지 버전이 없다면
`ErrImagePull`의 결과가 나타날 것이다. {{< /note >}}

맥이나 리눅스 호스트에서 해당 Docker 데몬을 사용하려면 `minikube docker-env` 에
서 마지막 줄을 실행한다.

이제 개인의 맥/리눅스 머신 내 커멘드 라인에서 도커를 사용해서 Minikube VM 안의
도커 데몬과 통신할 수 있다.

```shell
docker ps
```

{{< note >}} Centos 7 에서 Docker는 아래와 같은 오류를 발생한다.

```
Could not read CA certificate "/etc/docker/ca.pem": open /etc/docker/ca.pem: no such file or directory
```

/etc/sysconfig/docker를 업데이트하고 Minikube의 환경에 변경이 반영되었는지 확인
해서 고칠 수 있다.

```shell
< DOCKER_CERT_PATH=/etc/docker
---
> if [ -z "${DOCKER_CERT_PATH}" ]; then
>   DOCKER_CERT_PATH=/etc/docker
> fi
```

{{< /note >}}

### 쿠버네티스 구성하기

Minikube는 사용자가 쿠버네티스 컴포넌트를 다양한 값으로 설정할 수 있도록 하는 '
설정기' 기능이 있다. 이 기능을 사용하려면, `--extra-config` 플래그를
`minikube start` 명령어에 추가하여야 한다.

이 플래그는 여러번 쓸 수 있어 여러 옵션 설정을 전달 할 수 있다.

이 플래그는 `component.key=value`형식의 문자열로, 앞에 `component`는 아래 목록에
하나의 문자열이며 `key`는 configuration struct의 값이고 `value`는 설정할 값이다(
역주: key는 struct의 맴버명).

올바른 키들은 각 컴포넌트의 쿠버네티스 `componentconfigs` 문서에서 찾아 볼 수 있
다. 다음은 각각의 지원하는 설정에 대한 문서이다.

- [kubelet](https://godoc.org/k8s.io/kubernetes/pkg/kubelet/apis/kubeletconfig#KubeletConfiguration)
- [apiserver](https://godoc.org/k8s.io/kubernetes/cmd/kube-apiserver/app/options#ServerRunOptions)
- [proxy](https://godoc.org/k8s.io/kubernetes/pkg/proxy/apis/config#KubeProxyConfiguration)
- [controller-manager](https://godoc.org/k8s.io/kubernetes/pkg/controller/apis/config#KubeControllerManagerConfiguration)
- [etcd](https://godoc.org/github.com/coreos/etcd/etcdserver#ServerConfig)
- [scheduler](https://godoc.org/k8s.io/kubernetes/pkg/scheduler/apis/config#KubeSchedulerConfiguration)

#### 예제

쿠블렛에서 `MaxPods` 설정을 5로 바꾸려면 `--extra-config=kubelet.MaxPods=5` 플래
그를 전달하자.

이 기능은 또한 중첩 구조를 지원한다. 스케쥴러에서 `LeaderElection.LeaderElect`
설정을 `true`로 하려면,
`--extra-config=scheduler.LeaderElection.LeaderElect=true` 플래그를 전달하자.

`apiserver`에서 `AuthorizationMode`를 `RBAC`으로 바꾸려면,
`--extra-config=apiserver.authorization-mode=RBAC`를 사용할 수 있다.

### 클러스터 중지

`minikube stop` 명령어는 클러스터를 중지하는데 사용할 수 있다. 이 명령어는
Minikube 가상 머신을 종료하지만, 모든 클러스터 상태와 데이터를 보존한다. 클러스
터를 다시 시작하면 이전의 상태로 돌려줍니다.

### 클러스터 삭제

`minikube delete` 명령은 클러스터를 삭제하는데 사용할 수 있다. 이 명령어는
Minikube 가상 머신을 종료하고 삭제한다. 어떤 데이터나 상태도 보존되지 않다.

### minikube 업그레이드

[minikube 업그레이드](https://minikube.sigs.k8s.io/docs/start/macos/)를 본다.

## 클러스터와 상호 작용하기

### Kubectl

`minikube start` 명령어는 Minikube로 부르는
[kubectl 컨텍스트](/docs/reference/generated/kubectl/kubectl-commands/#-em-set-context-em-)를
생성한다. 이 컨텍스트는 Minikube 클러스터와 통신하는 설정을 포함한다.

Minikube는 이 컨텍스트를 자동적으로 기본으로 설정한다. 만약 미래에 이것을 바꾸�
싶다면

`kubectl config use-context minikube`을 실행하자.

혹은 `kubectl get pods --context=minikube`처럼 코멘드를 실행할때마다 매번 컨텍스
트를 전달한다.

### 대시보드

[쿠버네티스 대시보드](/ko/docs/tasks/access-application-cluster/web-ui-dashboard/)를
이용하려면, Minikube를 실행한 후 쉘에서 아래 명령어를 실행하여 주소를 확인한다.

```shell
minikube dashboard
```

### 서비스

노드 포트로 노출된 서비스를 접근하기 위해, Minikube를 시작한 이후 쉘에서 아래 명
령어를 실행하여 주소를 확인하자.

```shell
minikube service [-n NAMESPACE] [--url] NAME
```

## 네트워킹

Minikube VM은 host-only IP 주소를 통해 호스트 시스템에 노출되고, 이 IP 주소는
`minikube ip` 명령어로 확인할 수 있다. `NodePort` 서비스 타입은 IP 주소에 해당
노드 포트로 접근할 수 있다.

서비스의 NodePort를 확인하려면 `kubectl` 명령어로 아래와 같이 하면 된다.

`kubectl get service $SERVICE --output='jsonpath="{.spec.ports[0].nodePort}"'`

## 퍼시스턴트 볼륨

Minikube는 [퍼시스턴트 볼륨](/docs/concepts/storage/persistent-volumes/)을
`hostPath` 타입으로 지원한다. 이런 퍼시스턴트 볼륨은 Minikube VM 내에 디렉터리로
매핑됩니다.

Minikube VM은 tmpfs에서 부트하는데, 매우 많은 디렉터리가 재부트(`minikube stop`)
까지는 유지되지 않다. 그러나, Minikube는 다음의 호스트 디렉터리 아래 파일은 유지
하도록 설정되어 있다.

- `/data`
- `/var/lib/minikube`
- `/var/lib/docker`

이것은 `/data` 디렉터리에 데이터를 보존하도록 한 퍼시스턴트 볼륨 환경설정의 예이
다.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0001
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 5Gi
  hostPath:
    path: /data/pv0001/
```

## 호스트 폴더 마운트

어떤 드라이버는 VM 안에 호스트 폴더를 마운트하여 VM과 호스트 사이에 쉽게 파일을
공유할 수 있게 한다. 이들은 지금 설정할 수 없고 사용하는 드라이버나 운영체제에
따라 다르다.

{{< note >}} 호스트 폴더 공유는 KVM 드라이버에서 아직 구현되어 있지 않다.
{{< /note >}}

| Driver        | OS      | HostFolder | VM              |
| ------------- | ------- | ---------- | --------------- |
| VirtualBox    | Linux   | /home      | /hosthome       |
| VirtualBox    | macOS   | /Users     | /Users          |
| VirtualBox    | Windows | C://Users  | /c/Users        |
| VMware Fusion | macOS   | /Users     | /mnt/hgfs/Users |
| Xhyve         | macOS   | /Users     | /Users          |

## 프라이빗 컨테이너 레지스트리

프라이빗 컨테이너 레지스트리를 이용하려면,
[이 페이지](/ko/docs/concepts/containers/images/)의 단계를 따르자.

`ImagePullSecrets`를 이용하기를 권하지만, Minikube VM 상에서 설정하려 한다면
`/home/docker` 디렉터리에 `.dockercfg`를 두거나 `/home/docker/.docker` 디렉터리
에 `config.json`을 둘 수 있다.

## 애드온

Minikube에서 커스텀 애드온을 적절히 시작하고 재시작할 수 있으려면, Minikube와 함
께 시작하려는 애드온을 `~/.minikube/addons` 디렉터리에 두자. 폴더 내부의 애드온
은 Minikube VM으로 이동되어 Minikube가 시작하거나 재시작될 때에 함께 실행된다.

## HTTP 프록시 환경에서 Minikube 사용하기

Minikube는 쿠버네티스와 Docker 데몬을 포함한 가상 머신을 생성한다. 쿠버네티스가
Docker를 이용하여 컨테이너를 스케쥴링 시도할 때에, Docker 데몬은 컨테이너 이미지
를 풀링하기 위해 외부 네트워크를 이용해야 한다.

HTTP 프록시 내부라면, Docker에서 프록시 설정을 해야 한다. 이를 하기 위해서 요구
되는 환경 변수를 `minikube start` 중에 플래그로 전달한다.

예를 들어:

```shell
minikube start --docker-env http_proxy=http://$YOURPROXY:PORT \
                 --docker-env https_proxy=https://$YOURPROXY:PORT
```

만약 가상 머신 주소가 192.168.99.100 이라면 프록시 설정이 `kubectl`에 직접적으로
도달하지 못할 수도 있다. 이 IP 주소에 대해 프록시 설정을 지나치게 하려면
no_proxy 설정을 수정해야 한다. 다음과 같이 할 수 있다.

```shell
export no_proxy=$no_proxy,$(minikube ip)
```

## 알려진 이슈

다중 노드가 필요한 기능은 Minukube에서 동작하지 않는다.

## 설계

Minikube는 VM 프로비저닝을 위해서
[libmachine](https://github.com/docker/machine/tree/master/libmachine)를 사용하
고, 쿠버네티스 클러스터를 프로비저닝하기 위해
[kubeadm](https://github.com/kubernetes/kubeadm)을 사용한다.

Minikube에 대한 더 자세한 정보는,
[제안](https://git.k8s.io/community/contributors/design-proposals/cluster-lifecycle/local-cluster-ux.md)
부분을 읽어보자.

## 추가적인 링크:

- **목표와 비목표**: Minikube 프로젝트의 목표와 비목표에 대해서는
  [로드맵](https://git.k8s.io/minikube/docs/contributors/roadmap.md)을 살펴보자.
- **개발 가이드**: 풀 리퀘스트를 보내는 방법에 대한 개요는
  [참여 가이드](https://git.k8s.io/minikube/CONTRIBUTING.md)를 살펴보자.
- **Minikube 빌드**: Minikube를 소스에서 빌드/테스트하는 방법은
  [빌드 가이드](https://git.k8s.io/minikube/docs/contributors/build_guide.md)를
  살펴보자.
- **새 의존성 추가하기**: Minikube에 새 의존성을 추가하는 방법에 대해서는,
  [의존성 추가 가이드](https://git.k8s.io/minikube/docs/contributors/adding_a_dependency.md)를
  보자.
- **새 애드온 추가하기**: Minikube에 새 애드온을 추가하는 방법에 대해서는,
  [애드온 추가 가이드](https://git.k8s.io/minikube/docs/contributors/adding_an_addon.md)를
  보자.
- **MicroK8s**: 가상 머신을 사용하지 않으려는 Linux 사용자는 대안으로
  [MicroK8s](https://microk8s.io/)를 고려할 수 있다.

## 커뮤니티

컨트리뷰션, 질문과 의견은 모두 환영하며 격려한다! Minikube 개발자는
[슬랙](https://kubernetes.slack.com)에 #minikube 채널(초청받으려면
[여기](http://slack.kubernetes.io/))에 상주하고 있다. 또한
[kubernetes-dev 구글 그룹 메일링 리스트](https://groups.google.com/forum/#!forum/kubernetes-dev)도
있다. 메일링 리스트에 포스팅한다면 제목에 "minikube: "라는 접두어를 사용하자.

{{% /capture %}}
