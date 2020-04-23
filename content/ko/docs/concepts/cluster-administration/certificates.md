---
title: 인증서
content_template: templates/concept
weight: 20
---

{{% capture overview %}}

클라이언트 인증서로 인증을 사용하는 경우 `easyrsa`, `openssl` 또는 `cfssl` 을 통
해 인증서를 수동으로 생성할 수 있다.

{{% /capture %}}

{{% capture body %}}

### easyrsa

**easyrsa** 는 클러스터 인증서를 수동으로 생성할 수 있다.

1.  easyrsa3의 패치 버전을 다운로드하여 압축을 풀고, 초기화한다.

        curl -LO https://storage.googleapis.com/kubernetes-release/easy-rsa/easy-rsa.tar.gz
        tar xzf easy-rsa.tar.gz
        cd easy-rsa-master/easyrsa3
        ./easyrsa init-pki

1.  새로운 인증 기관(CA)을 생성한다. `--batch` 는 자동 모드를 설정한다.
    `--req-cn` 는 CA의 새 루트 인증서에 대한 일반 이름(Common Name (CN))을 지정
    한다.

        ./easyrsa --batch "--req-cn=${MASTER_IP}@`date +%s`" build-ca nopass

1.  서버 인증서와 키를 생성한다. `--subject-alt-name` 인수는 API 서버에 접근이
    가능한 IP와 DNS 이름을 설정한다. `MASTER_CLUSTER_IP` 는 일반적으로 API 서버
    와컨트롤러 관리자 컴포넌트에 대해 `--service-cluster-ip-range` 인수로지정된
    서비스 CIDR의 첫 번째 IP이다. `--days` 인수는 인증서가 만료되는일 수를 설정
    하는데 사용된다. 또한, 아래 샘플은 기본 DNS 이름으로 `cluster.local` 을사용
    한다고 가정한다.

        ./easyrsa --subject-alt-name="IP:${MASTER_IP},"\
        "IP:${MASTER_CLUSTER_IP},"\
        "DNS:kubernetes,"\
        "DNS:kubernetes.default,"\
        "DNS:kubernetes.default.svc,"\
        "DNS:kubernetes.default.svc.cluster,"\
        "DNS:kubernetes.default.svc.cluster.local" \
        --days=10000 \
        build-server-full server nopass

1.  `pki/ca.crt`, `pki/issued/server.crt` 그리고 `pki/private/server.key` 를 디
    렉터리에 복사한다.
1.  API 서버 시작 파라미터에 다음 파라미터를 채우고 추가한다.

        --client-ca-file=/yourdirectory/ca.crt
        --tls-cert-file=/yourdirectory/server.crt
        --tls-private-key-file=/yourdirectory/server.key

### openssl

**openssl** 은 클러스터 인증서를 수동으로 생성할 수 있다.

1.  ca.key를 2048bit로 생성한다.

        openssl genrsa -out ca.key 2048

1.  ca.key에 따라 ca.crt를 생성한다(인증서 유효 기간을 사용하려면 -days를 사용한
    다).

        openssl req -x509 -new -nodes -key ca.key -subj "/CN=${MASTER_IP}" -days 10000 -out ca.crt

1.  server.key를 2048bit로 생성한다.

        openssl genrsa -out server.key 2048

1.  인증서 서명 요청(Certificate Signing Request (CSR))을 생성하기 위한 설정 파
    일을 생성한다. 파일에 저장하기 전에 꺾쇠 괄호(예: `<MASTER_IP>`)로표시된 값
    을 실제 값으로 대체한다(예: `csr.conf`). `MASTER_CLUSTER_IP` 의 값은 이전 하
    위 섹션에서설명한 대로 API 서버의 서비스 클러스터 IP이다. 또한, 아래 샘플에
    서는 `cluster.local` 을 기본 DNS 도메인이름으로 사용하고 있다고 가정한다.

        [ req ]
        default_bits = 2048
        prompt = no
        default_md = sha256
        req_extensions = req_ext
        distinguished_name = dn

        [ dn ]
        C = <국가(country)>
        ST = <도(state)>
        L = <시(city)>
        O = <조직(organization)>
        OU = <조직 단위(organization unit)>
        CN = <MASTER_IP>

        [ req_ext ]
        subjectAltName = @alt_names

        [ alt_names ]
        DNS.1 = kubernetes
        DNS.2 = kubernetes.default
        DNS.3 = kubernetes.default.svc
        DNS.4 = kubernetes.default.svc.cluster
        DNS.5 = kubernetes.default.svc.cluster.local
        IP.1 = <MASTER_IP>
        IP.2 = <MASTER_CLUSTER_IP>

        [ v3_ext ]
        authorityKeyIdentifier=keyid,issuer:always
        basicConstraints=CA:FALSE
        keyUsage=keyEncipherment,dataEncipherment
        extendedKeyUsage=serverAuth,clientAuth
        subjectAltName=@alt_names

1.  설정 파일을 기반으로 인증서 서명 요청을 생성한다.

        openssl req -new -key server.key -out server.csr -config csr.conf

1.  ca.key, ca.crt 그리고 server.csr을 사용해서 서버 인증서를 생성한다.

        openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key \
        -CAcreateserial -out server.crt -days 10000 \
        -extensions v3_ext -extfile csr.conf

1.  인증서를 본다.

        openssl x509  -noout -text -in ./server.crt

마지막으로, API 서버 시작 파라미터에 동일한 파라미터를 추가한다.

### cfssl

**cfssl** 은 인증서 생성을 위한 또 다른 도구이다.

1.  아래에 표시된 대로 커맨드 라인 도구를 다운로드하여 압축을 풀고 준비한다. 사
    용 중인 하드웨어 아키텍처 및 cfssl 버전에 따라 샘플명령을 조정해야 할 수도
    있다.

        curl -L https://github.com/cloudflare/cfssl/releases/download/v1.4.1/cfssl_1.4.1_linux_amd64 -o cfssl
        chmod +x cfssl
        curl -L https://github.com/cloudflare/cfssl/releases/download/v1.4.1/cfssljson_1.4.1_linux_amd64 -o cfssljson
        chmod +x cfssljson
        curl -L https://github.com/cloudflare/cfssl/releases/download/v1.4.1/cfssl-certinfo_1.4.1_linux_amd64 -o cfssl-certinfo
        chmod +x cfssl-certinfo

1.  아티팩트(artifact)를 보유할 디렉터리를 생성하고 cfssl을 초기화한다.

        mkdir cert
        cd cert
        ../cfssl print-defaults config > config.json
        ../cfssl print-defaults csr > csr.json

1.  CA 파일을 생성하기 위한 JSON 설정 파일을 `ca-config.json` 예시와 같이 생성한
    다.

        {
          "signing": {
            "default": {
              "expiry": "8760h"
            },
            "profiles": {
              "kubernetes": {
                "usages": [
                  "signing",
                  "key encipherment",
                  "server auth",
                  "client auth"
                ],
                "expiry": "8760h"
              }
            }
          }
        }

1.  CA 인증서 서명 요청(CSR)을 위한 JSON 설정 파일을 `ca-csr.json` 예시와 같이
    생성한다. 꺾쇠 괄호로 표시된값을 사용하려는 실제 값으로 변경한다.

        {
          "CN": "kubernetes",
          "key": {
            "algo": "rsa",
            "size": 2048
          },
          "names":[{
            "C": "<국가(country)>",
            "ST": "<도(state)>",
            "L": "<시(city)>",
            "O": "<조직(organization)>",
            "OU": "<조직 단위(organization unit)>"
          }]
        }

1.  CA 키(`ca-key.pem`)와 인증서(`ca.pem`)을 생성한다.

        ../cfssl gencert -initca ca-csr.json | ../cfssljson -bare ca

1.  API 서버의 키와 인증서를 생성하기 위한 JSON 구성파일을 `server-csr.json` 예
    시와 같이 생성한다. 꺾쇠 괄호 안의 값을사용하려는 실제 값으로 변경한다.
    `MASTER_CLUSTER_IP` 는이전 하위 섹션에서 설명한 API 서버의 클러스터 IP이다.
    아래 샘플은 기본 DNS 도메인 이름으로 `cluster.local` 을사용한다고 가정한다.

        {
          "CN": "kubernetes",
          "hosts": [
            "127.0.0.1",
            "<MASTER_IP>",
            "<MASTER_CLUSTER_IP>",
            "kubernetes",
            "kubernetes.default",
            "kubernetes.default.svc",
            "kubernetes.default.svc.cluster",
            "kubernetes.default.svc.cluster.local"
          ],
          "key": {
            "algo": "rsa",
            "size": 2048
          },
          "names": [{
            "C": "<국가(country)>",
            "ST": "<도(state)>",
            "L": "<시(city)>",
            "O": "<조직(organization)>",
            "OU": "<조직 단위(organization unit)>"
          }]
        }

1.  API 서버 키와 인증서를 생성하면, 기본적으로 `server-key.pem` 과 `server.pem`
    파일에 각각 저장된다.

        ../cfssl gencert -ca=ca.pem -ca-key=ca-key.pem \
        --config=ca-config.json -profile=kubernetes \
        server-csr.json | ../cfssljson -bare server

## 자체 서명된 CA 인증서의 배포

클라이언트 노드는 자체 서명된 CA 인증서를 유효한 것으로 인식하지 않을 수 있다.
비-프로덕션 디플로이먼트 또는 회사 방화벽 뒤에서 실행되는디플로이먼트의 경우, 자
체 서명된 CA 인증서를 모든 클라이언트에배포하고 유효한 인증서의 로컬 목록을 새로
고칠 수 있다.

각 클라이언트에서, 다음 작업을 수행한다.

```bash
sudo cp ca.crt /usr/local/share/ca-certificates/kubernetes.crt
sudo update-ca-certificates
```

```
Updating certificates in /etc/ssl/certs...
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d....
done.
```

## 인증서 API

`certificates.k8s.io` API를 사용해서
[여기](/docs/tasks/tls/managing-tls-in-a-cluster)에설명된 대로 인증에 사용�
x509 인증서를 프로비전 할 수 있다.

{{% /capture %}}
