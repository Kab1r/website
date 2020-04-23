---
content_template: templates/concept
title: GPU 스케줄링
---

{{% capture overview %}}

{{< feature-state state="beta" for_k8s_version="1.10" >}}

쿠버네티스는 AMD 및 NVIDIA GPU(그래픽 프로세싱 유닛)를 노드들에 걸쳐 관리하기 위
한 **실험적인** 지원을 포함한다.

이 페이지는 다른 쿠버네티스 버전 간에 걸쳐 사용자가 GPU들을 소비할 수 있는 방법
과현재의 제약 사항을 설명한다.

{{% /capture %}}

{{% capture body %}}

## 디바이스 플러그인 사용하기

쿠버네티스는
{{< glossary_tooltip text="디바이스 플러그인" term_id="device-plugin" >}}을 구현
하여파드가 GPU와 같이 특별한 하드웨어 기능에 접근할 수 있게 한다.

관리자는 해당하는 하드웨어 벤더의 GPU 드라이버를 노드에설치해야 하며, GPU 벤더가
제공하는 디바이스 플러그인을실행해야 한다.

- [AMD](#amd-gpu-디바이스-플러그인-배치하기)
- [NVIDIA](#nvidia-gpu-디바이스-플러그인-배치하기)

위의 조건이 만족되면, 쿠버네티스는 `amd.com/gpu` 또는 `nvidia.com/gpu` 를 스케줄
가능한 리소스로써 노출시킨다.

사용자는 이 GPU들을 `cpu` 나 `memory` 를 요청하는 방식과 동일하게
`<vendor>.com/gpu` 를 요청함으로써 컨테이너를 통해 소비할 수 있다. 그러나 GPU를
사용할 때는 리소스 요구 사항을 명시하는 방식에 약간의제약이 있다.

- GPU는 `limits` 섹션에서만 명시되는 것을 가정한다. 그 의미는 다음과 같다.
  - 쿠버네티스는 limits를 requests의 기본 값으로 사용하게 되므로사용자는 GPU
    `limits` 를 명시할 때 `requests` 명시하지 않아도 된다.
  - 사용자는 `limits` 과 `requests` 를 모두 명시할 수 있지만, 두 값은동일해야 한
    다.
  - 사용자는 `limits` 명시 없이는 GPU `requests` 를 명시할 수 없다.
- 컨테이너들(그리고 파드들)은 GPU를 공유하지 않는다. GPU에 대한 초과 할당
  (overcommitting)은 제공되지 않는다.
- 각 컨테이너는 하나 이상의 GPU를 요청할 수 있다. GPU의 일부(fraction)를 요청하
  는 것은불가능하다.

다음은 한 예제를 보여준다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cuda-vector-add
spec:
  restartPolicy: OnFailure
  containers:
    - name: cuda-vector-add
      # https://github.com/kubernetes/kubernetes/blob/v1.7.11/test/images/nvidia-cuda/Dockerfile
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
      resources:
        limits:
          nvidia.com/gpu: 1 # GPU 1개 요청하기
```

### AMD GPU 디바이스 플러그인 배치하기

[공식 AMD GPU 디바이스 플러그인](https://github.com/RadeonOpenCompute/k8s-device-plugin)에
는다음의 요구 사항이 있다.

- 쿠버네티스 노드들에는 AMD GPU 리눅스 드라이버가 미리 설치되어 있어야 한다.

클러스터가 실행 중이고 위의 요구 사항이 만족된 후, AMD 디바이스 플러그인을 배치
하기 위해서는아래 명령어를 실행한다.

```shell
kubectl create -f https://raw.githubusercontent.com/RadeonOpenCompute/k8s-device-plugin/v1.10/k8s-ds-amdgpu-dp.yaml
```

[RadeonOpenCompute/k8s-device-plugin](https://github.com/RadeonOpenCompute/k8s-device-plugin)에
이슈를 로깅하여해당 서드 파티 디바이스 플러그인에 대한 이슈를 리포트할 수 있다.

### NVIDIA GPU 디바이스 플러그인 배치하기

현재는 NVIDIA GPU에 대한 두 개의 디바이스 플러그인 구현체가 있다.

#### 공식 NVIDIA GPU 디바이스 플러그인

[공식 NVIDIA GPU 디바이스 플러그인](https://github.com/NVIDIA/k8s-device-plugin)은
다음의 요구 사항을 가진다.

- 쿠버네티스 노드에는 NVIDIA 드라이버가 미리 설치되어 있어야 한다.
- 쿠버네티스 노드에는
  [nvidia-docker 2.0](https://github.com/NVIDIA/nvidia-docker)이 미리 설치되어
  있어야 한다.
- Kubelet은 자신의 컨테이너 런타임으로 도커를 사용해야 한다.
- 도커는 runc 대신 `nvidia-container-runtime` 이
  [기본 런타임](https://github.com/NVIDIA/k8s-device-plugin#preparing-your-gpu-nodes)으
  로설정되어야 한다.
- NVIDIA 드라이버의 버전은 조건 ~= 361.93 을 만족해야 한다.

클러스터가 실행 중이고 위의 요구 사항이 만족된 후, NVIDIA 디바이스 플러그인을 배
치하기 위해서는아래 명령어를 실행한다.

```shell
kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/1.0.0-beta4/nvidia-device-plugin.yml
```

[NVIDIA/k8s-device-plugin](https://github.com/NVIDIA/k8s-device-plugin)에 이슈를
로깅하여해당 서드 파티 디바이스 플러그인에 대한 이슈를 리포트할 수 있다.

#### GCE에서 사용되는 NVIDIA GPU 디바이스 플러그인

[GCE에서 사용되는 NVIDIA GPU 디바이스 플러그인](https://github.com/GoogleCloudPlatform/container-engine-accelerators/tree/master/cmd/nvidia_gpu)은
nvidia-docker의 사용이 필수가 아니며 컨테이너 런타임 인터페이스(CRI)에호환되는
다른 컨테이너 런타임을 사용할 수 있다. 해당 사항은
[컨테이너에 최적화된 OS](https://cloud.google.com/container-optimized-os/)에서
테스트되었고, 우분투 1.9 이후 버전에 대한 실험적인 코드를 가지고 있다.

사용자는 다음 커맨드를 사용하여 NVIDIA 드라이버와 디바이스 플러그인을 설치할 수
있다.

```shell
# 컨테이너에 최적회된 OS에 NVIDIA 드라이버 설치:
kubectl create -f https://raw.githubusercontent.com/GoogleCloudPlatform/container-engine-accelerators/stable/daemonset.yaml

# 우분투에 NVIDIA 드라이버 설치(실험적):
kubectl create -f https://raw.githubusercontent.com/GoogleCloudPlatform/container-engine-accelerators/stable/nvidia-driver-installer/ubuntu/daemonset.yaml

# 디바이스 플러그인 설치:
kubectl create -f https://raw.githubusercontent.com/kubernetes/kubernetes/release-1.14/cluster/addons/device-plugins/nvidia-gpu/daemonset.yaml
```

[GoogleCloudPlatform/container-engine-accelerators](https://github.com/GoogleCloudPlatform/container-engine-accelerators)에
이슈를 로깅하여해당 서드 파티 디바이스 플러그인에 대한 이슈를 리포트할 수 있다.

Google은 GKE에서 NVIDIA GPU 사용에 대한 자체
[설명서](https://cloud.google.com/kubernetes-engine/docs/how-to/gpus)를 게재하�
있다.

## 다른 타입의 GPU들을 포함하는 클러스터

만약 클러스터의 노드들이 서로 다른 타입의 GPU를 가지고 있다면, 사용자는파드를 적
합한 노드에 스케줄 하기 위해서
[노드 레이블과 노드 셀렉터](/docs/tasks/configure-pod-container/assign-pods-nodes/)를
사용할 수 있다.

예를 들면,

```shell
# 노드가 가진 가속기 타입에 따라 레이블을 단다.
kubectl label nodes <node-with-k80> accelerator=nvidia-tesla-k80
kubectl label nodes <node-with-p100> accelerator=nvidia-tesla-p100
```

## 노드 레이블링 자동화 {#node-labeller}

만약 AMD GPU 디바이스를 사용하고 있다면,
[노드 레이블러](https://github.com/RadeonOpenCompute/k8s-device-plugin/tree/master/cmd/k8s-node-labeller)를
배치할 수 있다. 노드 레이블러는 GPU 디바이스의 속성에 따라서 노드에 자동으로 레
이블을 달아 주는 {{< glossary_tooltip text="컨트롤러" term_id="controller" >}}이
다.

현재 이 컨트롤러는 다음의 속성에 대해 레이블을 추가할 수 있다.

- 디바이스 ID (-device-id)
- VRAM 크기 (-vram)
- SIMD 개수 (-simd-count)
- 계산 유닛 개수 (-cu-count)
- 펌웨어 및 기능 버전 (-firmware)
- GPU 계열, 두 개 문자 형태의 축약어 (-family)
  - SI - Southern Islands
  - CI - Sea Islands
  - KV - Kaveri
  - VI - Volcanic Islands
  - CZ - Carrizo
  - AI - Arctic Islands
  - RV - Raven

```shell
kubectl describe node cluster-node-23
```

```
    Name:               cluster-node-23
    Roles:              <none>
    Labels:             beta.amd.com/gpu.cu-count.64=1
                        beta.amd.com/gpu.device-id.6860=1
                        beta.amd.com/gpu.family.AI=1
                        beta.amd.com/gpu.simd-count.256=1
                        beta.amd.com/gpu.vram.16G=1
                        beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/os=linux
                        kubernetes.io/hostname=cluster-node-23
    Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                        node.alpha.kubernetes.io/ttl: 0
    …
```

노드 레이블러가 사용된 경우, GPU 타입을 파드 스펙에 명시할 수 있다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cuda-vector-add
spec:
  restartPolicy: OnFailure
  containers:
    - name: cuda-vector-add
      # https://github.com/kubernetes/kubernetes/blob/v1.7.11/test/images/nvidia-cuda/Dockerfile
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
      resources:
        limits:
          nvidia.com/gpu: 1
  nodeSelector:
    accelerator: nvidia-tesla-p100 # 또는 nvidia-tesla-k80 등.
```

이것은 파드가 사용자가 지정한 GPU 타입을 가진 노드에 스케줄 되도록만든다.

{{% /capture %}}
