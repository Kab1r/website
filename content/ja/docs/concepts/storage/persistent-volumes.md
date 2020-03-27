---
title: 永続ボリュー�
feature:
  title: ストレージオーケストレーション
  description: >
    ローカルストレージや<a href="https://cloud.google.com/storage/">GCP</a>、<a
    href="https://aws.amazon.com/products/storage/">AWS</a>などのパブリッククラウドプロバイダー、もしくはNFS、iSCSI、Gluster、Ceph、Cinder、Flockerのようなネットワークストレージシステムの中から選択されたものを自動的にマウントします。

content_template: templates/concept
weight: 20
---

{{% capture overview %}}

このドキュメントでは Kubernetes の`PersistentVolume`について説明します
。[ボリューム](/docs/concepts/storage/volumes/)を一読することをおすすめします。

{{% /capture %}}

{{% capture body %}}

## 概要

ストレージを管理することはインスタンスを管理することとは全くの別物です
。`PersistentVolume`サブシステムは、ストレージが何から提供されているか、どのよう
に消費されているかをユーザーと管理者から抽象化する API を提供します。これを実現
するための`PersistentVolume`と`PersistentVolumeClaim`という 2 つの新しい API リ
ソースを紹介します。

`PersistentVolume`(PV)
は[ストレージクラス](/docs/concepts/storage/storage-classes/)を使って管理者もし
くは動的にプロビジョニングされるクラスターのストレージの一部です。これは Node と
同じようにクラスターリソースの一部です。PV は Volume のようなボリュームプラグイ
ンですが、PV を使う個別の Pod とは独立したライフサイクルを持っています。この API
オブジェクトは NFS、iSCSI やクラウドプロバイダー固有のストレージシステムの実装の
詳細を捕捉します。

`PersistentVolumeClaim`(PVC)はユーザーによって要求されるストレージです。これは
Pod と似ています。Pod は Node リソースを消費し、PVC は PV リソースを消費します
。Pod は特定レベルの CPU とメモリーリソースを要求することができます。クレームは
特定のサイズやアクセスモード(例えば、1 ノードからのみ読み書きマウントができるモ
ードや、複数ノードから読み込み専用マウントができるモードなどです)を要求すること
ができます。

`PersistentVolumeClaim`はユーザーに抽象化されたストレージリソースの消費を許可す
る一方、ユーザーは色々な問題に対処するためにパフォーマンスといった様々なプロパテ
ィを持った`PersistentVolume`を必要とすることは一般的なことです。クラスター管理者
はユーザーに様々なボリュームがどのように実装されているかを表に出すことなく、サイ
ズやアクセスモードだけではない色々な点で異なった、様々な`PersistentVolume`を提供
できる必要があります。これらのニーズに応えるために`StorageClass`リソースがありま
す。

[実例を含む詳細なチュートリアル](/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)を
参照して下さい。

## ボリュームと要求のライフサイクル

PV はクラスター内のリソースです。PVC はこれらのリソースの要求でありまた、クレー
ムのチェックとしても機能します。PV と PVC の相互作用はこのライフサイクルに従いま
す。

### プロビジョニング

PV は静的か動的どちらかでプロビジョニングされます。

#### 静的

クラスター管理者は多数の PV を作成します。それらはクラスターのユーザーが使うこと
のできる実際のストレージの詳細を保持します。それらは Kubernetes API に存在し、利
用できます。

#### 動的

ユーザーの`PersistentVolumeClaim`が管理者の作成したいずれの静的 PV にも一致しな
い場合、クラスターは PVC 用にボリュームを動的にプロビジョニングしようとする場合
があります。このプロビジョニングは`StorageClass`に基づいています。PVC
は[ストレージクラス](/docs/concepts/storage/storage-classes/)の要求が必要であり
、管理者は動的プロビジョニングを行うためにストレージクラスの作成・設定が必要です
。ストレージクラスを""にしたストレージ要求は、自身の動的プロビジョニングを事実上
無効にします。

ストレージクラスに基づいたストレージの動的プロビジョニングを有効化するには、クラ
スター管理者
が`DefaultStorageClass`[アドミッションコントローラー](/docs/reference/access-authn-authz/admission-controllers/#defaultstorageclass)を
API サーバーで有効化する必要があります。これは例えば、`DefaultStorageClass`が
API サーバーコンポーネントの`--enable-admission-plugins`フラグのコンマ区切りの順
序付きリストの中に含まれているかで確認できます。 API サーバーのコマンドラインフ
ラグの詳細については[kube-apiserver](/docs/admin/kube-apiserver/)のドキュメント
を参照してください。

### バインディング

ユーザは、特定のサイズのストレージとアクセスモードを指定した上
で`PersistentVolumeClaim`を作成します（動的プロビジョニングの場合は、すでに作ら
れています）。マスター内のコントロールループは、新しく作られる PVC をウォッチし
て、それにマッチする PV が見つかったときに、それらを紐付けます。PV が新しい PVC
用に動的プロビジョニングされた場合、コントロールループは常に PV をその PVC に紐
付けます。そうでない場合、ユーザーは常に少なくとも要求したサイズ以上のボリュー�
を取得しますが、ボリュームは要求されたサイズを超えている可能性があります。一度紐
付けされると、どのように紐付けられたかに関係なく`PersistentVolumeClaim`の紐付け
は排他的（決められた特定の PV としか結びつかない状態）になります。PVC から PV へ
の紐付けは 1 対 1 です。

一致するボリュームが存在しない場合、クレームはいつまでも紐付けされないままになり
ます。一致するボリュームが利用可能になると、クレームがバインドされます。たとえば
、50Gi の PV がいくつもプロビジョニングされているクラスターだとしても、100Gi を
要求する PVC とは一致しません。100Gi の PV がクラスターに追加されると、PVC を紐
付けできます。

### 使用

Pod は要求をボリュームとして使用します。クラスターは、要求を検査して紐付けられた
ボリュームを見つけそのボリュームを Pod にマウントします。複数のアクセスモードを
サポートするボリュームの場合、ユーザーは Pod のボリュームとしてクレームを使う時
にどのモードを希望するかを指定します。

ユーザーがクレームを取得し、そのクレームがバインドされると、バインドされた PV は
必要な限りそのユーザーに属します。ユーザーは Pod をスケジュールし、Pod の
volumes ブロックに`persistentVolumeClaim`を含めることで、バインドされたクレー�
の PV にアクセスします。
[書式の詳細はこちらを確認して下さい。](#claims-as-volumes)

### 使用中のストレージオブジェクトの保護

使用中のストレージオブジェクト保護機能の目的はデータ損失を防ぐために、Pod によっ
て実際に使用されている永続ボリュームクレーム(PVC)と、PVC にバインドされている永
続ボリューム(PV)がシステムから削除されないようにすることです。

{{< note >}} PVC を使用している Pod オブジェクトが存在する場合、PVC は Pod によ
って実際に使用されています。 {{< /note >}}

ユーザーが Pod によって実際に使用されている PVC を削除しても、その PVC はすぐに
は削除されません。PVC の削除は、PVC が Pod で使用されなくなるまで延期されます。
また、管理者が PVC に紐付けられている PV を削除しても、PV はすぐには削除されませ
ん。PV が PVC に紐付けられなくなるまで、PV の削除は延期されます。

PVC の削除が保護されているかは、PVC のステータスが`Terminating`になっていて、そ
して`Finalizers`のリストに`kubernetes.io/pvc-protection`が含まれているかで確認で
きます。

```shell
kubectl describe pvc hostpath
Name:          hostpath
Namespace:     default
StorageClass:  example-hostpath
Status:        Terminating
Volume:
Labels:        <none>
Annotations:   volume.beta.kubernetes.io/storage-class=example-hostpath
               volume.beta.kubernetes.io/storage-provisioner=example.com/hostpath
Finalizers:    [kubernetes.io/pvc-protection]
```

同様に PV の削除が保護されているかは、PV のステータスが`Terminating`になっていて
、そして`Finalizers`のリストに`kubernetes.io/pv-protection`が含まれているかで確
認できます。

```shell
kubectl describe pv task-pv-volume
Name:            task-pv-volume
Labels:          type=local
Annotations:     <none>
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    standard
Status:          Available
Claim:
Reclaim Policy:  Delete
Access Modes:    RWO
Capacity:        1Gi
Message:
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /tmp/data
    HostPathType:
Events:            <none>
```

### 再クレー�

ユーザーは、ボリュームの使用が完了したら、リソースの再クレームを許可する API か
ら PVC オブジェクトを削除できます。`PersistentVolume`の再クレームポリシーはその
クレームが解放された後のボリュームの処理をクラスターに指示します。現在、ボリュー
ムは保持、リサイクル、または削除できます。

#### 保持

`Retain`という再クレームポリシーはリソースを手動で再クレームすることができます
。`PersistentVolumeClaim`が削除される時、`PersistentVolume`は依然として存在はし
ますが、ボリュームは解放済みです。ただし、以前のクレームデータはボリューム上に残
っているため、別のクレームにはまだ使用できません。管理者は次の手順でボリュームを
手動で再クレームできます。

1. `PersistentVolume`を削除します。PV が削除された後も、外部インフラストラクチャ
   ー(AWS EBS、GCE PD、Azure Disk、Cinder ボリュームなど)に関連付けられたストレ
   ージアセットは依然として残ります。
1. ストレージアセットに関連するのデータを手動で適切にクリーンアップします。
1. 関連するストレージアセットを手動で削除するか、同じストレージアセットを再利用
   したい場合、新しいストレージアセット定義と共に`PersistentVolume`を作成します
   。

#### 削除

`Delete`再クレームポリシーをサポートするボリュームプラグインの場合、削除する
と`PersistentVolume`オブジェクトが Kubernetes から削除されるだけでなく、AWS
EBS、GCE PD、Azure Disk、Cinder ボリュームなどの外部インフラストラクチャーの関連
ストレージアセットも削除されます。動的にプロビジョニングされたボリュームは
、[`StorageClass`の再クレームポリシー](#reclaim-policy)を継承します。これはデフ
ォルトで削除です。管理者は、ユーザーの需要に応じて`StorageClass`を構成する必要が
あります。そうでない場合、PV は作成後に編集またはパッチを適用する必要があります
。[PersistentVolume の再クレームポリシーの変更](/docs/tasks/administer-cluster/change-pv-reclaim-policy/)を
参照してください。

#### リサイクル

{{< warning >}} `Recycle`再クレームポリシーは廃止されました。代わりに、動的プロ
ビジョニングを使用することをおすすめします。 {{< /warning >}}

基盤となるボリュームプラグインでサポートされている場合、`Recycle`再クレームポリ
シーはボリュームに対して基本的な削除(`rm -rf /thevolume/*`)を実行し、新しいクレ
ームに対して再び利用できるようにします。

管理者は[こちら](/docs/admin/kube-controller-manager/)で説明するように
、Kubernetes コントローラーマネージャーのコマンドライン引数を使用して、カスタ�
リサイクラー Pod テンプレートを構成できます。カスタムリサイクラー Pod テンプレー
トには、次の例に示すように、`volumes`仕様が含まれている必要があります。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pv-recycler
  namespace: default
spec:
  restartPolicy: Never
  volumes:
    - name: vol
      hostPath:
        path: /any/path/it/will/be/replaced
  containers:
    - name: pv-recycler
      image: "k8s.gcr.io/busybox"
      command:
        [
          "/bin/sh",
          "-c",
          'test -e /scrub && rm -rf /scrub/..?* /scrub/.[!.]* /scrub/*  && test
          -z "$(ls -A /scrub)" || exit 1',
        ]
      volumeMounts:
        - name: vol
          mountPath: /scrub
```

ただし、カスタムリサイクラー Pod テンプレートの`volumes`パート内で指定された特定
のパスは、リサイクルされるボリュームの特定のパスに置き換えられます。

### 永続ボリュームクレームの拡大

{{< feature-state for_k8s_version="v1.11" state="beta" >}}

PersistentVolumeClaim(PVC)の拡大はデフォルトで有効です。次のボリュームの種類で拡
大できます。

- gcePersistentDisk
- awsElasticBlockStore
- Cinder
- glusterfs
- rbd
- Azure File
- Azure Disk
- Portworx
- FlexVolumes
- CSI

そのストレージクラスの`allowVolumeExpansion`フィールドが true となっている場合の
み、PVC を拡大できます。

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gluster-vol-default
provisioner: kubernetes.io/glusterfs
parameters:
  resturl: "http://192.168.10.100:8080"
  restuser: ""
  secretNamespace: ""
  secretName: ""
allowVolumeExpansion: true
```

PVC に対してさらに大きなボリュームを要求するには、PVC オブジェクトを編集してより
大きなサイズを指定します。これにより`PersistentVolume`を受け持つ基盤にボリュー�
の拡大がトリガーされます。クレームを満たすため新しく`PersistentVolume`が作成され
ることはありません。代わりに既存のボリュームがリサイズされます。

#### CSI ボリュームの拡張

{{< feature-state for_k8s_version="v1.16" state="beta" >}}

CSI ボリュームの拡張のサポートはデフォルトで有効になっていますが、ボリューム拡張
をサポートするにはボリューム拡張を利用できる CSI ドライバーも必要です。詳細につ
いては、それぞれの CSI ドライバーのドキュメントを参照してください。

#### ファイルシステムを含むボリュームのリサイズ

ファイルシステムが XFS、Ext3、または Ext4 の場合にのみ、ファイルシステムを含むボ
リュームのサイズを変更できます。

ボリュームにファイルシステムが含まれる場合、新しい Pod
が`PersistentVolumeClaim`で ReadWrite モードを使用している場合にのみ、ファイルシ
ステムのサイズが変更されます。ファイルシステムの拡張は、Pod の起動時、もしくは
Pod の実行時で基盤となるファイルシステムがオンラインの拡張をサポートする場合に行
われます。

FlexVolumes では、ドライバの`RequiresFSResize`機能が true に設定されている場合、
サイズを変更できます。 FlexVolume は、Pod の再起動時にサイズ変更できます。

#### 使用中の永続ボリュームクレームのリサイズ

{{< feature-state for_k8s_version="v1.15" state="beta" >}}

{{< note >}} 使用中の PVC の拡張は、Kubernetes 1.15 以降のベータ版と、1.11 以降
のアルファ版として利用可能です。`ExpandInUsePersistentVolume`機能を有効化する必
要があります。これはベータ機能のため多くのクラスターで自動的に行われます。詳細に
ついては
、[フィーチャーゲート](/docs/reference/command-line-tools-reference/feature-gates/)の
ドキュメントを参照してください。 {{< /note >}}

この場合、既存の PVC を使用している Pod または Deployment を削除して再作成する必
要はありません。使用中の PVC は、ファイルシステムが拡張されるとすぐに Pod で自動
的に使用可能になります。この機能は、Pod または Deployment で使用されていない PVC
には影響しません。拡張を完了する前に、PVC を使用する Pod を作成する必要がありま
す。

他のボリュームタイプと同様、FlexVolume ボリュームは、Pod によって使用されている
最中でも拡張できます。

{{< note >}} FlexVolume のリサイズは、基盤となるドライバーがリサイズをサポートし
ている場合のみ可能です。 {{< /note >}}

{{< note >}} EBS の拡張は時間がかかる操作です。また変更は、ボリュームごとに 6 時
間に 1 回までというクォータもあります。 {{< /note >}}

## 永続ボリュームの種類

`PersistentVolume`の種類はプラグインとして実装されます。Kubernetes は現在次のプ
ラグインに対応しています。

- GCEPersistentDisk
- AWSElasticBlockStore
- AzureFile
- AzureDisk
- CSI
- FC (Fibre Channel)
- FlexVolume
- Flocker
- NFS
- iSCSI
- RBD (Ceph Block Device)
- CephFS
- Cinder (OpenStack block storage)
- Glusterfs
- VsphereVolume
- Quobyte Volumes
- HostPath (テスト用の単一ノードのみ。ローカルストレージはどのような方法でもサポ
  ートされておらず、またマルチノードクラスターでは動作しません)
- Portworx Volumes
- ScaleIO Volumes
- StorageOS

## 永続ボリュー�

各 PV には、仕様とボリュームのステータスが含まれている spec と status が含まれて
います。

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
```

### 容量

通常、PV には特定のストレージ容量があります。これは PV の`capacity`属性を使用し
て設定されます。容量によって期待される単位を理解するためには、Kubernetes
の[リソースモデル](https://git.k8s.io/community/contributors/design-proposals/scheduling/resources.md)を
参照してください。

現在、設定または要求できるのはストレージサイズのみです。将来の属性には、IOPS、ス
ループットなどが含まれます。

### ボリュームモード

{{< feature-state for_k8s_version="v1.13" state="beta" >}}

Kubernetes 1.9 より前は、すべてのボリュームプラグインが永続ボリュームにファイル
システムを作成していました。現在は Raw ブロックデバイスを使うため
に`volumeMode`の値を`block`に設定するか、ファイルシステムを使うため
に`filesystem`を設定できます。値が省略された場合のデフォルトは`filesystem`です。
これはオプションの API パラメーターです。

### アクセスモード

`PersistentVolume`は、リソースプロバイダーがサポートする方法でホストにマウントで
きます。次の表に示すように、プロバイダーにはさまざまな機能があり、各 PV のアクセ
スモードは、その特定のボリュームでサポートされる特定のモードに設定されます。たと
えば、NFS は複数の読み取り/書き込みクライアントをサポートできますが、特定の NFS
の PV はサーバー上で読み取り専用としてエクスポートされる場合があります。各 PV は
、その特定の PV の機能を記述する独自のアクセスモードのセットを取得します。

アクセスモードは次の通りです。

- ReadWriteOnce –ボリュームは単一の Node で読み取り/書き込みとしてマウントできま
  す
- ReadOnlyMany –ボリュームは多数の Node で読み取り専用としてマウントできます
- ReadWriteMany –ボリュームは多数の Node で読み取り/書き込みとしてマウントできま
  す

CLI ではアクセスモードは次のように略されます。

- RWO - ReadWriteOnce
- ROX - ReadOnlyMany
- RWX - ReadWriteMany

> **Important!** ボリュームは、多数のモードをサポートしていても、一度に 1 つのア
> クセスモードでしかマウントできません。たとえば、GCEPersistentDisk は、単一
> Node では ReadWriteOnce として、または多数の Node では ReadOnlyMany としてマウ
> ントできますが、同時にマウントすることはできません。

| ボリュームプラグイン |  ReadWriteOnce   |   ReadOnlyMany   |          ReadWriteMany           |
| :------------------- | :--------------: | :--------------: | :------------------------------: |
| AWSElasticBlockStore |     &#x2713;     |        -         |                -                 |
| AzureFile            |     &#x2713;     |     &#x2713;     |             &#x2713;             |
| AzureDisk            |     &#x2713;     |        -         |                -                 |
| CephFS               |     &#x2713;     |     &#x2713;     |             &#x2713;             |
| Cinder               |     &#x2713;     |        -         |                -                 |
| CSI                  | ドライバーに依存 | ドライバーに依存 |         ドライバーに依存         |
| FC                   |     &#x2713;     |     &#x2713;     |                -                 |
| FlexVolume           |     &#x2713;     |     &#x2713;     |         ドライバーに依存         |
| Flocker              |     &#x2713;     |        -         |                -                 |
| GCEPersistentDisk    |     &#x2713;     |     &#x2713;     |                -                 |
| Glusterfs            |     &#x2713;     |     &#x2713;     |             &#x2713;             |
| HostPath             |     &#x2713;     |        -         |                -                 |
| iSCSI                |     &#x2713;     |     &#x2713;     |                -                 |
| Quobyte              |     &#x2713;     |     &#x2713;     |             &#x2713;             |
| NFS                  |     &#x2713;     |     &#x2713;     |             &#x2713;             |
| RBD                  |     &#x2713;     |     &#x2713;     |                -                 |
| VsphereVolume        |     &#x2713;     |        -         | - (Pod が連結されている場合のみ) |
| PortworxVolume       |     &#x2713;     |        -         |             &#x2713;             |
| ScaleIO              |     &#x2713;     |     &#x2713;     |                -                 |
| StorageOS            |     &#x2713;     |        -         |                -                 |

### Class

PV はクラスを持つことができます。これは`storageClassName`属性
を[ストレージクラス](/docs/concepts/storage/storage-classes/)の名前に設定するこ
とで指定されます。特定のクラスの PV は、そのクラスを要求する PVC にのみバインド
できます。`storageClassName`にクラスがない PV は、特定のクラスを要求しない PVC
にのみバインドできます。

以前`volume.beta.kubernetes.io/storage-class`アノテーションは
、`storageClassName`属性の代わりに使用されていました。このアノテーションはまだ機
能しています。ただし、将来の Kubernetes リリースでは完全に非推奨です。

### 再クレームポリシー {#reclaim-policy}

現在の再クレームポリシーは次のとおりです。

- 保持 -- 手動再クレー�
- リサイクル -- 基本的な削除 (`rm -rf /thevolume/*`)
- 削除 -- AWS EBS、GCE PD、Azure Disk、もしくは OpenStack Cinder ボリュームに関
  連するストレージアセットを削除

現在、NFS と HostPath のみがリサイクルをサポートしています。AWS EBS、GCE
PD、Azure Disk、および Cinder volume は削除をサポートしています。

### マウントオプション

Kubernets 管理者は永続ボリュームが Node にマウントされるときの追加マウントオプシ
ョンを指定できます。

{{< note >}} すべての永続ボリュームタイプがすべてのマウントオプションをサポート
するわけではありません。 {{< /note >}}

次のボリュームタイプがマウントオプションをサポートしています。

- AWSElasticBlockStore
- AzureDisk
- AzureFile
- CephFS
- Cinder (OpenStack ブロックストレージ)
- GCEPersistentDisk
- Glusterfs
- NFS
- Quobyte Volumes
- RBD (Ceph Block Device)
- StorageOS
- VsphereVolume
- iSCSI

マウントオプションは検証されないため、不正だった場合マウントは失敗します。

以前`volume.beta.kubernetes.io/mount-options`アノテーションが`mountOptions`属性
の代わりに使われていました。このアノテーションはまだ機能しています。ただし、将来
の Kubernetes リリースでは完全に非推奨です。

### ノードアフィニティ

{{< note >}} ほとんどのボリュームタイプはこのフィールドを設定する必要がありませ
ん
。[AWS EBS](/docs/concepts/storage/volumes/#awselasticblockstore)、[GCE PD](/docs/concepts/storage/volumes/#gcepersistentdisk)、
もしくは[Azure Disk](/docs/concepts/storage/volumes/#azuredisk)ボリュームブロッ
クタイプの場合自動的に設定されます
。[local](/docs/concepts/storage/volumes/#local)ボリュームは明示的に設定する必要
があります。 {{< /note >}}

PV
は[ノードアフィニティ](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#volumenodeaffinity-v1-core)
を指定して、このボリュームにアクセスできる Node を制限する制約を定義できます。PV
を使用する Pod は、ノードアフィニティによって選択された Node にのみスケジュール
されます。

### フェーズ

ボリュームは次のフェーズのいずれかです。

- 利用可能 -- まだクレームに紐付いていない自由なリソース
- バウンド -- クレームに紐付いている
- リリース済み -- クレームが削除されたが、クラスターにまだクレームされている
- 失敗 -- 自動再クレームに失敗

CLI には PV に紐付いている PVC の名前が表示されます。

## 永続ボリューム要求

各 PVC には spec とステータスが含まれます。これは、仕様とクレームのステータスで
す。

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow
  selector:
    matchLabels:
      release: "stable"
    matchExpressions:
      - { key: environment, operator: In, values: [dev] }
```

### アクセスモード

クレームは、特定のアクセスモードでストレージを要求するときにボリュームと同じ規則
を使用します。

### ボリュームモード

クレームは、ボリュームと同じ規則を使用して、ファイルシステムまたはブロックデバイ
スとしてのボリュームの消費を示します。

### リソース

Pod と同様に、クレームは特定の量のリソースを要求できます。この場合、要求はストレ
ージ用です。同
じ[リソースモデル](https://git.k8s.io/community/contributors/design-proposals/scheduling/resources.md)が
ボリュームとクレームの両方に適用されます。

### セレクター

クレームでは
、[ラベルセレクター](/docs/concepts/overview/working-with-objects/labels/#label-selectors)を
指定して、ボリュームセットをさらにフィルター処理できます。ラベルがセレクターに一
致するボリュームのみがクレームにバインドできます。セレクターは 2 つのフィールド
で構成できます。

- `matchLabels` - ボリュームはこの値のラベルが必要です
- `matchExpressions` - キー、値のリスト、およびキーと値を関連付ける演算子を指定
  することによって作成された要件のリスト。有効な演算子は、In、NotIn、Exists およ
  び DoesNotExist です。

`matchLabels`と`matchExpressions`の両方からのすべての要件は AND で結合されます。
一致するには、すべてが一致する必要があります。

### クラス

クレームは、`storageClassName`属性を使用し
て[ストレージクラス](/docs/concepts/storage/storage-classes/)の名前を指定するこ
とにより、特定のクラスを要求できます。PVC にバインドできるのは、PVC と同
じ`storageClassName`を持つ、要求されたクラスの PV のみです。

PVC は必ずしもクラスをリクエストする必要はありません。`storageClassName`が`""`に
設定されている PVC は、クラスのない PV を要求していると常に解釈されるため、クラ
スのない PV にのみバインドできます（アノテーションがないか、`""`に等しい 1 つの
セット）。`storageClassName`のない PVC はまったく同じではなく
、[`DefaultStorageClass`アドミッションプラグイン](/docs/reference/access-authn-authz/admission-controllers/#defaultstorageclass)が
オンになっているかどうかによって、クラスターによって異なる方法で処理されます。

- アドミッションプラグインがオンになっている場合、管理者はデフォルト
  の`StorageClass`を指定できます。`storageClassName`を持たないすべての PVC は、
  そのデフォルトの PV にのみバインドできます。デフォルトの`StorageClass`の指定は
  、`StorageClass`オブジェクトで`storageclass.kubernetes.io/is-default-class`ア
  ノテーションを`true`に設定することにより行われます。管理者がデフォルトを指定し
  ない場合、クラスターは、アドミッションプラグインがオフになっているかのように
  PVC 作成をレスポンスします。複数のデフォルトが指定されている場合、アドミッショ
  ンプラグインはすべての PVC の作成を禁止します。
- アドミッションプラグインがオフになっている場合、デフォルトの`StorageClass`の概
  念はありません。`storageClassName`を持たないすべての PVC は、クラスを持たない
  PV にのみバインドできます。この場合、storageClassName を持たない PVC は
  、`storageClassName`が`""`に設定されている PVC と同じように扱われます。

インストール方法によっては、インストール時にアドオンマネージャーによってデフォル
トのストレージクラスが Kubernetes クラスターにデプロイされる場合があります。

PVC が`selector`を要求することに加えて`StorageClass`を指定する場合、要件は AND
で一緒に結合されます。要求されたクラスの PV と要求されたラベルのみが PVC にバイ
ンドされます。

{{< note >}} 現在、`selector`が空ではない PVC は、PV を動的にプロビジョニングで
きません。 {{< /note >}}

以前は、`storageClassName`属性の代わり
に`volume.beta.kubernetes.io/storage-class`アノテーションが使用されていました。
このアノテーションはまだ機能しています。ただし、今後の Kubernetes リリースではサ
ポートされません。

## ボリュームとしてのクレー�

Pod は、クレームをボリュームとして使用してストレージにアクセスします。クレームは
、そのクレームを使用する Pod と同じ名前空間に存在する必要があります。クラスター
は、Pod の名前空間でクレームを見つけ、それを使用してクレームを支援してい
る`PersistentVolume`を取得します。次に、ボリュームがホストと Pod にマウントされ
ます。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
        - mountPath: "/var/www/html"
          name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

### 名前空間に関する注意

`PersistentVolume`バインドは排他的であり、`PersistentVolumeClaim`は名前空間オブ
ジェクトであるため、"多"モード(`ROX`、`RWX`)でクレームをマウントすることは 1 つ
の名前空間内でのみ可能です。

## Raw ブロックボリュームのサポート

{{< feature-state for_k8s_version="v1.13" state="beta" >}}

次のボリュームプラグインは、必要に応じて動的プロビジョニングを含む raw ブロック
ボリュームをサポートします。

- AWSElasticBlockStore
- AzureDisk
- FC (Fibre Channel)
- GCEPersistentDisk
- iSCSI
- Local volume
- RBD (Ceph Block Device)
- VsphereVolume (alpha)

{{< note >}} Kubernetes 1.9 では、FC および iSCSI ボリュームのみが raw ブロック
ボリュームをサポートしていました。追加のプラグインのサポートは 1.10 で追加されま
した。 {{< /note >}}

### Raw ブロックボリュームを使用した永続ボリュー�

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: block-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  volumeMode: Block
  persistentVolumeReclaimPolicy: Retain
  fc:
    targetWWNs: ["50060e801049cfd1"]
    lun: 0
    readOnly: false
```

### Raw ブロックボリュームを要求する永続ボリュームクレー�

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: block-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Block
  resources:
    requests:
      storage: 10Gi
```

### コンテナに Raw ブロックデバイスパスを追加する Pod 仕様

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-block-volume
spec:
  containers:
    - name: fc-container
      image: fedora:26
      command: ["/bin/sh", "-c"]
      args: ["tail -f /dev/null"]
      volumeDevices:
        - name: data
          devicePath: /dev/xvda
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: block-pvc
```

{{< note >}} Pod に raw ブロックデバイスを追加する場合は、マウントパスの代わりに
コンテナーでデバイスパスを指定します。 {{< /note >}}

### ブロックボリュームのバインド

ユーザーが`PersistentVolumeClaim`spec の`volumeMode`フィールドを使用して raw ブ
ロックボリュームの要求を示す場合、バインディングルールは、このモードを spec の一
部として考慮しなかった以前のリリースとわずかに異なります。表にリストされているの
は、ユーザーと管理者が raw ブロックデバイスを要求するために指定可能な組み合わせ
の表です。この表は、ボリュームがバインドされるか、組み合わせが与えられないかを示
します。静的にプロビジョニングされたボリュームのボリュームバインディングマトリク
スはこちらです。

| PV ボリュームモード | PVC ボリュームモード |         結果 |
| ------------------- | :------------------: | -----------: |
| 未定義              |        未定義        |     バインド |
| 未定義              |       ブロック       | バインドなし |
| 未定義              |   ファイルシステム   |     バインド |
| ブロック            |        未定義        | バインドなし |
| ブロック            |       ブロック       |     バインド |
| ブロック            |   ファイルシステム   | バインドなし |
| ファイルシステム    |   ファイルシステム   |     バインド |
| ファイルシステム    |       ブロック       | バインドなし |
| ファイルシステム    |        未定義        |     バインド |

{{< note >}} アルファリリースでは、静的にプロビジョニングされたボリュームのみが
サポートされます。管理者は、raw ブロックデバイスを使用する場合、これらの値を考慮
するように注意する必要があります。 {{< /note >}}

## ボリュームのスナップショットとスナップショットからのボリュームの復元のサポート

{{< feature-state for_k8s_version="v1.12" state="alpha" >}}

ボリュームスナップショット機能は、CSI ボリュームプラグインのみをサポートするため
に追加されました。詳細については
、[ボリュームのスナップショット](/docs/concepts/storage/volume-snapshots/)を参照
してください。

ボリュームスナップショットのデータソースからボリュームを復元する機能を有効にする
には、apiserver および controller-manager で`VolumeSnapshotDataSource`フィーチャ
ーゲートを有効にします。

### ボリュームスナップショットから永続ボリュームクレームを作成する

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: restore-pvc
spec:
  storageClassName: csi-hostpath-sc
  dataSource:
    name: new-snapshot-test
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

## ボリュームの複製

{{< feature-state for_k8s_version="v1.16" state="beta" >}}

ボリュームの複製機能は、CSI ボリュームプラグインのみをサポートするために追加され
ました。詳細については
、[ボリュームの複製](/docs/concepts/storage/volume-pvc-datasource/)を参照してく
ださい。

PVC データソースからのボリューム複製機能を有効にするには、apiserver および
controller-manager で`VolumeSnapshotDataSource`フィーチャーゲートを有効にします
。

### 既存の PVC からの永続ボリュームクレーム作成

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cloned-pvc
spec:
  storageClassName: my-csi-plugin
  dataSource:
    name: existing-src-pvc-name
    kind: PersistentVolumeClaim
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

## 可搬性の高い設定の作成

もし幅広いクラスターで実行され、永続ボリュームが必要となる構成テンプレートやサン
プルを作成している場合は、次のパターンを使用することをお勧めします。

- 構成に PersistentVolumeClaim オブジェクトを含める(Deployment や ConfigMap と共
  に)
- ユーザーが設定をインスタンス化する際に PersistentVolume を作成する権限がない場
  合があるため、設定に PersistentVolume オブジェクトを含めない。
- テンプレートをインスタンス化する時にストレージクラス名を指定する選択肢をユーザ
  ーに与える
  - ユーザーがストレージクラス名を指定する場合
    、`persistentVolumeClaim.storageClassName`フィールドにその値を入力する。これ
    により、クラスターが管理者によって有効にされたストレージクラスを持っている場
    合、PVC は正しいストレージクラスと一致する。
  - ユーザーがストレージクラス名を指定しない場合
    、`persistentVolumeClaim.storageClassName`フィールドは nil のままにする。こ
    れにより、PV はユーザーにクラスターのデフォルトストレージクラスで自動的にプ
    ロビジョニングされる。多くのクラスター環境ではデフォルトのストレージクラスが
    インストールされているが、管理者は独自のデフォルトストレージクラスを作成する
    ことができる。
- ツールが PVC を監視し、しばらくしてもバインドされないことをユーザーに表示する
  。これはクラスターが動的ストレージをサポートしない(この場合ユーザーは対応する
  PV を作成するべき)、もしくはクラスターがストレージシステムを持っていない(この
  場合ユーザーは PVC を必要とする設定をデプロイできない)可能性があることを示す。

{{% /capture %}}
