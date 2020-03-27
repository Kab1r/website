---
title: フィーチャーゲート
weight: 10
content_template: templates/concept
---

{{% capture overview %}} このページでは管理者がそれぞれの Kubernetes コンポーネ
ントで指定できるさまざまなフィーチャーゲートの概要について説明しています。

各機能におけるステージの説明については、[機能のステージ](#feature-stages)を参照
してください。 {{% /capture %}}

{{% capture body %}}

## 概要

フィーチャーゲートはアルファ機能または実験的機能を記述する key=value のペアのセ
ットです。管理者は各コンポーネントで`--feature-gates`コマンドラインフラグを使用
することで機能をオンまたはオフにできます。

各コンポーネントはそれぞれのコンポーネント固有のフィーチャーゲートの設定をサポー
トします。すべてのコンポーネントのフィーチャーゲートの全リストを表示するに
は`-h`フラグを使用します。kubelet などのコンポーネントにフィーチャーゲートを設定
するには以下のようにリストの機能ペアを`--feature-gates`フラグを使用して割り当て
ます。

```shell
--feature-gates="...,DynamicKubeletConfig=true"
```

次の表は各 Kubernetes コンポーネントに設定できるフィーチャーゲートの概要です。

- 「導入開始バージョン」列は機能が導入されたとき、またはリリース段階が変更された
  ときの Kubernetes リリースバージョンとなります。
- 「最終利用可能バージョン」列は空ではない場合はフィーチャーゲートを使用できる最
  後の Kubernetes リリースバージョンとなります。
- アルファまたはベータ状態の機能
  は[Alpha または Beta のフィーチャーゲート](#feature-gates-for-alpha-or-beta-features)に
  載っています。
- 安定している機能は
  、[graduated または deprecated のフィーチャーゲート](#feature-gates-for-graduated-or-deprecated-features)に
  載っています。
- graduated または deprecated のフィーチャーゲートには、非推奨および廃止された機
  能もリストされています。

### Alpha または Beta のフィーチャーゲート {#feature-gates-for-alpha-or-beta-features}

{{< table caption="AlphaまたはBetaのフィーチャーゲート" >}}

| 機能名                                           | デフォルト値 | ステージ | 導入開始バージョン | 最終利用可能バージョン |
| ------------------------------------------------ | ------------ | -------- | ------------------ | ---------------------- |
| `APIListChunking`                                | `false`      | Alpha    | 1.8                | 1.8                    |
| `APIListChunking`                                | `true`       | Beta     | 1.9                |                        |
| `APIPriorityAndFairness`                         | `false`      | Alpha    | 1.17               |                        |
| `APIResponseCompression`                         | `false`      | Alpha    | 1.7                |                        |
| `AppArmor`                                       | `true`       | Beta     | 1.4                |                        |
| `BalanceAttachedNodeVolumes`                     | `false`      | Alpha    | 1.11               |                        |
| `BlockVolume`                                    | `false`      | Alpha    | 1.9                | 1.12                   |
| `BlockVolume`                                    | `true`       | Beta     | 1.13               | -                      |
| `BoundServiceAccountTokenVolume`                 | `false`      | Alpha    | 1.13               |                        |
| `CPUManager`                                     | `false`      | Alpha    | 1.8                | 1.9                    |
| `CPUManager`                                     | `true`       | Beta     | 1.10               |                        |
| `CRIContainerLogRotation`                        | `false`      | Alpha    | 1.10               | 1.10                   |
| `CRIContainerLogRotation`                        | `true`       | Beta     | 1.11               |                        |
| `CSIBlockVolume`                                 | `false`      | Alpha    | 1.11               | 1.13                   |
| `CSIBlockVolume`                                 | `true`       | Beta     | 1.14               |                        |
| `CSIDriverRegistry`                              | `false`      | Alpha    | 1.12               | 1.13                   |
| `CSIDriverRegistry`                              | `true`       | Beta     | 1.14               |                        |
| `CSIInlineVolume`                                | `false`      | Alpha    | 1.15               | 1.15                   |
| `CSIInlineVolume`                                | `true`       | Beta     | 1.16               | -                      |
| `CSIMigration`                                   | `false`      | Alpha    | 1.14               | 1.16                   |
| `CSIMigration`                                   | `true`       | Beta     | 1.17               |                        |
| `CSIMigrationAWS`                                | `false`      | Alpha    | 1.14               |                        |
| `CSIMigrationAWS`                                | `false`      | Beta     | 1.17               |                        |
| `CSIMigrationAWSComplete`                        | `false`      | Alpha    | 1.17               |                        |
| `CSIMigrationAzureDisk`                          | `false`      | Alpha    | 1.15               |                        |
| `CSIMigrationAzureDiskComplete`                  | `false`      | Alpha    | 1.17               |                        |
| `CSIMigrationAzureFile`                          | `false`      | Alpha    | 1.15               |                        |
| `CSIMigrationAzureFileComplete`                  | `false`      | Alpha    | 1.17               |                        |
| `CSIMigrationGCE`                                | `false`      | Alpha    | 1.14               | 1.16                   |
| `CSIMigrationGCE`                                | `false`      | Beta     | 1.17               |                        |
| `CSIMigrationGCEComplete`                        | `false`      | Alpha    | 1.17               |                        |
| `CSIMigrationOpenStack`                          | `false`      | Alpha    | 1.14               |                        |
| `CSIMigrationOpenStackComplete`                  | `false`      | Alpha    | 1.17               |                        |
| `CustomCPUCFSQuotaPeriod`                        | `false`      | Alpha    | 1.12               |                        |
| `CustomResourceDefaulting`                       | `false`      | Alpha    | 1.15               | 1.15                   |
| `CustomResourceDefaulting`                       | `true`       | Beta     | 1.16               |                        |
| `DevicePlugins`                                  | `false`      | Alpha    | 1.8                | 1.9                    |
| `DevicePlugins`                                  | `true`       | Beta     | 1.10               |                        |
| `DryRun`                                         | `false`      | Alpha    | 1.12               | 1.12                   |
| `DryRun`                                         | `true`       | Beta     | 1.13               |                        |
| `DynamicAuditing`                                | `false`      | Alpha    | 1.13               |                        |
| `DynamicKubeletConfig`                           | `false`      | Alpha    | 1.4                | 1.10                   |
| `DynamicKubeletConfig`                           | `true`       | Beta     | 1.11               |                        |
| `EndpointSlice`                                  | `false`      | Alpha    | 1.16               | 1.16                   |
| `EndpointSlice`                                  | `false`      | Beta     | 1.17               |                        |
| `EphemeralContainers`                            | `false`      | Alpha    | 1.16               |                        |
| `ExpandCSIVolumes`                               | `false`      | Alpha    | 1.14               | 1.15                   |
| `ExpandCSIVolumes`                               | `true`       | Beta     | 1.16               |                        |
| `ExpandInUsePersistentVolumes`                   | `false`      | Alpha    | 1.11               | 1.14                   |
| `ExpandInUsePersistentVolumes`                   | `true`       | Beta     | 1.15               |                        |
| `ExpandPersistentVolumes`                        | `false`      | Alpha    | 1.8                | 1.10                   |
| `ExpandPersistentVolumes`                        | `true`       | Beta     | 1.11               |                        |
| `ExperimentalHostUserNamespaceDefaulting`        | `false`      | Beta     | 1.5                |                        |
| `EvenPodsSpread`                                 | `false`      | Alpha    | 1.16               |                        |
| `HPAScaleToZero`                                 | `false`      | Alpha    | 1.16               |                        |
| `HyperVContainer`                                | `false`      | Alpha    | 1.10               |                        |
| `KubeletPodResources`                            | `false`      | Alpha    | 1.13               | 1.14                   |
| `KubeletPodResources`                            | `true`       | Beta     | 1.15               |                        |
| `LegacyNodeRoleBehavior`                         | `true`       | Alpha    | 1.16               |                        |
| `LocalStorageCapacityIsolation`                  | `false`      | Alpha    | 1.7                | 1.9                    |
| `LocalStorageCapacityIsolation`                  | `true`       | Beta     | 1.10               |                        |
| `LocalStorageCapacityIsolationFSQuotaMonitoring` | `false`      | Alpha    | 1.15               |                        |
| `MountContainers`                                | `false`      | Alpha    | 1.9                |                        |
| `NodeDisruptionExclusion`                        | `false`      | Alpha    | 1.16               |                        |
| `NonPreemptingPriority`                          | `false`      | Alpha    | 1.15               |                        |
| `PodOverhead`                                    | `false`      | Alpha    | 1.16               | -                      |
| `ProcMountType`                                  | `false`      | Alpha    | 1.12               |                        |
| `QOSReserved`                                    | `false`      | Alpha    | 1.11               |                        |
| `RemainingItemCount`                             | `false`      | Alpha    | 1.15               |                        |
| `ResourceLimitsPriorityFunction`                 | `false`      | Alpha    | 1.9                |                        |
| `RotateKubeletClientCertificate`                 | `true`       | Beta     | 1.8                |                        |
| `RotateKubeletServerCertificate`                 | `false`      | Alpha    | 1.7                | 1.11                   |
| `RotateKubeletServerCertificate`                 | `true`       | Beta     | 1.12               |                        |
| `RunAsGroup`                                     | `true`       | Beta     | 1.14               |                        |
| `RuntimeClass`                                   | `false`      | Alpha    | 1.12               | 1.13                   |
| `RuntimeClass`                                   | `true`       | Beta     | 1.14               |                        |
| `SCTPSupport`                                    | `false`      | Alpha    | 1.12               |                        |
| `ServerSideApply`                                | `false`      | Alpha    | 1.14               | 1.15                   |
| `ServerSideApply`                                | `true`       | Beta     | 1.16               |                        |
| `ServiceNodeExclusion`                           | `false`      | Alpha    | 1.8                |                        |
| `StartupProbe`                                   | `true`       | Beta     | 1.17               |                        |
| `StorageVersionHash`                             | `false`      | Alpha    | 1.14               | 1.14                   |
| `StorageVersionHash`                             | `true`       | Beta     | 1.15               |                        |
| `StreamingProxyRedirects`                        | `false`      | Beta     | 1.5                | 1.5                    |
| `StreamingProxyRedirects`                        | `true`       | Beta     | 1.6                |                        |
| `SupportNodePidsLimit`                           | `false`      | Alpha    | 1.14               | 1.14                   |
| `SupportNodePidsLimit`                           | `true`       | Beta     | 1.15               |                        |
| `SupportPodPidsLimit`                            | `false`      | Alpha    | 1.10               | 1.13                   |
| `SupportPodPidsLimit`                            | `true`       | Beta     | 1.14               |                        |
| `Sysctls`                                        | `true`       | Beta     | 1.11               |                        |
| `TaintBasedEvictions`                            | `false`      | Alpha    | 1.6                | 1.12                   |
| `TaintBasedEvictions`                            | `true`       | Beta     | 1.13               |                        |
| `TokenRequest`                                   | `false`      | Alpha    | 1.10               | 1.11                   |
| `TokenRequest`                                   | `true`       | Beta     | 1.12               |                        |
| `TokenRequestProjection`                         | `false`      | Alpha    | 1.11               | 1.11                   |
| `TokenRequestProjection`                         | `true`       | Beta     | 1.12               |                        |
| `TTLAfterFinished`                               | `false`      | Alpha    | 1.12               |                        |
| `TopologyManager`                                | `false`      | Alpha    | 1.16               |                        |
| `ValidateProxyRedirects`                         | `false`      | Alpha    | 1.12               | 1.13                   |
| `ValidateProxyRedirects`                         | `true`       | Beta     | 1.14               |                        |
| `VolumePVCDataSource`                            | `false`      | Alpha    | 1.15               | 1.15                   |
| `VolumePVCDataSource`                            | `true`       | Beta     | 1.16               |                        |
| `VolumeSnapshotDataSource`                       | `false`      | Alpha    | 1.12               | 1.16                   |
| `VolumeSnapshotDataSource`                       | `true`       | Beta     | 1.17               | -                      |
| `WindowsGMSA`                                    | `false`      | Alpha    | 1.14               |                        |
| `WindowsGMSA`                                    | `true`       | Beta     | 1.16               |                        |
| `WinDSR`                                         | `false`      | Alpha    | 1.14               |                        |
| `WinOverlay`                                     | `false`      | Alpha    | 1.14               |                        |

{{< /table >}}

### Graduated または Deprecated のフィーチャーゲート {#feature-gates-for-graduated-or-deprecated-features}

{{< table caption="GraduatedまたはDeprecatedのフィーチャーゲート" >}}

| 機能名                              | デフォルト値 | ステージ   | 導入開始バージョン | 最終利用可能バージョン |
| ----------------------------------- | ------------ | ---------- | ------------------ | ---------------------- |
| `Accelerators`                      | `false`      | Alpha      | 1.6                | 1.10                   |
| `Accelerators`                      | -            | Deprecated | 1.11               | -                      |
| `AdvancedAuditing`                  | `false`      | Alpha      | 1.7                | 1.7                    |
| `AdvancedAuditing`                  | `true`       | Beta       | 1.8                | 1.11                   |
| `AdvancedAuditing`                  | `true`       | GA         | 1.12               | -                      |
| `AffinityInAnnotations`             | `false`      | Alpha      | 1.6                | 1.7                    |
| `AffinityInAnnotations`             | -            | Deprecated | 1.8                | -                      |
| `AllowExtTrafficLocalEndpoints`     | `false`      | Beta       | 1.4                | 1.6                    |
| `AllowExtTrafficLocalEndpoints`     | `true`       | GA         | 1.7                | -                      |
| `CSINodeInfo`                       | `false`      | Alpha      | 1.12               | 1.13                   |
| `CSINodeInfo`                       | `true`       | Beta       | 1.14               | 1.16                   |
| `CSINodeInfo`                       | `true`       | GA         | 1.17               |                        |
| `AttachVolumeLimit`                 | `false`      | Alpha      | 1.11               | 1.11                   |
| `AttachVolumeLimit`                 | `true`       | Beta       | 1.12               | 1.16                   |
| `AttachVolumeLimit`                 | `true`       | GA         | 1.17               | -                      |
| `CSIPersistentVolume`               | `false`      | Alpha      | 1.9                | 1.9                    |
| `CSIPersistentVolume`               | `true`       | Beta       | 1.10               | 1.12                   |
| `CSIPersistentVolume`               | `true`       | GA         | 1.13               | -                      |
| `CustomPodDNS`                      | `false`      | Alpha      | 1.9                | 1.9                    |
| `CustomPodDNS`                      | `true`       | Beta       | 1.10               | 1.13                   |
| `CustomPodDNS`                      | `true`       | GA         | 1.14               | -                      |
| `CustomResourcePublishOpenAPI`      | `false`      | Alpha      | 1.14               | 1.14                   |
| `CustomResourcePublishOpenAPI`      | `true`       | Beta       | 1.15               | 1.15                   |
| `CustomResourcePublishOpenAPI`      | `true`       | GA         | 1.16               | -                      |
| `CustomResourceSubresources`        | `false`      | Alpha      | 1.10               | 1.10                   |
| `CustomResourceSubresources`        | `true`       | Beta       | 1.11               | 1.15                   |
| `CustomResourceSubresources`        | `true`       | GA         | 1.16               | -                      |
| `CustomResourceValidation`          | `false`      | Alpha      | 1.8                | 1.8                    |
| `CustomResourceValidation`          | `true`       | Beta       | 1.9                | 1.15                   |
| `CustomResourceValidation`          | `true`       | GA         | 1.16               | -                      |
| `CustomResourceWebhookConversion`   | `false`      | Alpha      | 1.13               | 1.14                   |
| `CustomResourceWebhookConversion`   | `true`       | Beta       | 1.15               | 1.15                   |
| `CustomResourceWebhookConversion`   | `true`       | GA         | 1.16               | -                      |
| `DynamicProvisioningScheduling`     | `false`      | Alpha      | 1.11               | 1.11                   |
| `DynamicProvisioningScheduling`     | -            | Deprecated | 1.12               | -                      |
| `DynamicVolumeProvisioning`         | `true`       | Alpha      | 1.3                | 1.7                    |
| `DynamicVolumeProvisioning`         | `true`       | GA         | 1.8                | -                      |
| `EnableEquivalenceClassCache`       | `false`      | Alpha      | 1.8                | 1.14                   |
| `EnableEquivalenceClassCache`       | -            | Deprecated | 1.15               | -                      |
| `ExperimentalCriticalPodAnnotation` | `false`      | Alpha      | 1.5                | 1.12                   |
| `ExperimentalCriticalPodAnnotation` | `false`      | Deprecated | 1.13               | -                      |
| `GCERegionalPersistentDisk`         | `true`       | Beta       | 1.10               | 1.12                   |
| `GCERegionalPersistentDisk`         | `true`       | GA         | 1.13               | -                      |
| `HugePages`                         | `false`      | Alpha      | 1.8                | 1.9                    |
| `HugePages`                         | `true`       | Beta       | 1.10               | 1.13                   |
| `HugePages`                         | `true`       | GA         | 1.14               | -                      |
| `Initializers`                      | `false`      | Alpha      | 1.7                | 1.13                   |
| `Initializers`                      | -            | Deprecated | 1.14               | -                      |
| `KubeletConfigFile`                 | `false`      | Alpha      | 1.8                | 1.9                    |
| `KubeletConfigFile`                 | -            | Deprecated | 1.10               | -                      |
| `KubeletPluginsWatcher`             | `false`      | Alpha      | 1.11               | 1.11                   |
| `KubeletPluginsWatcher`             | `true`       | Beta       | 1.12               | 1.12                   |
| `KubeletPluginsWatcher`             | `true`       | GA         | 1.13               | -                      |
| `MountPropagation`                  | `false`      | Alpha      | 1.8                | 1.9                    |
| `MountPropagation`                  | `true`       | Beta       | 1.10               | 1.11                   |
| `MountPropagation`                  | `true`       | GA         | 1.12               | -                      |
| `NodeLease`                         | `false`      | Alpha      | 1.12               | 1.13                   |
| `NodeLease`                         | `true`       | Beta       | 1.14               | 1.16                   |
| `NodeLease`                         | `true`       | GA         | 1.17               | -                      |
| `PersistentLocalVolumes`            | `false`      | Alpha      | 1.7                | 1.9                    |
| `PersistentLocalVolumes`            | `true`       | Beta       | 1.10               | 1.13                   |
| `PersistentLocalVolumes`            | `true`       | GA         | 1.14               | -                      |
| `PodPriority`                       | `false`      | Alpha      | 1.8                | 1.10                   |
| `PodPriority`                       | `true`       | Beta       | 1.11               | 1.13                   |
| `PodPriority`                       | `true`       | GA         | 1.14               | -                      |
| `PodReadinessGates`                 | `false`      | Alpha      | 1.11               | 1.11                   |
| `PodReadinessGates`                 | `true`       | Beta       | 1.12               | 1.13                   |
| `PodReadinessGates`                 | `true`       | GA         | 1.14               | -                      |
| `PodShareProcessNamespace`          | `false`      | Alpha      | 1.10               | 1.11                   |
| `PodShareProcessNamespace`          | `true`       | Beta       | 1.12               | 1.16                   |
| `PodShareProcessNamespace`          | `true`       | GA         | 1.17               | -                      |
| `PVCProtection`                     | `false`      | Alpha      | 1.9                | 1.9                    |
| `PVCProtection`                     | -            | Deprecated | 1.10               | -                      |
| `RequestManagement`                 | `false`      | Alpha      | 1.15               | 1.16                   |
| `ResourceQuotaScopeSelectors`       | `false`      | Alpha      | 1.11               | 1.11                   |
| `ResourceQuotaScopeSelectors`       | `true`       | Beta       | 1.12               | 1.16                   |
| `ResourceQuotaScopeSelectors`       | `true`       | GA         | 1.17               | -                      |
| `ScheduleDaemonSetPods`             | `false`      | Alpha      | 1.11               | 1.11                   |
| `ScheduleDaemonSetPods`             | `true`       | Beta       | 1.12               | 1.16                   |
| `ScheduleDaemonSetPods`             | `true`       | GA         | 1.17               | -                      |
| `ServiceLoadBalancerFinalizer`      | `false`      | Alpha      | 1.15               | 1.15                   |
| `ServiceLoadBalancerFinalizer`      | `true`       | Beta       | 1.16               | 1.16                   |
| `ServiceLoadBalancerFinalizer`      | `true`       | GA         | 1.17               | -                      |
| `StorageObjectInUseProtection`      | `true`       | Beta       | 1.10               | 1.10                   |
| `StorageObjectInUseProtection`      | `true`       | GA         | 1.11               | -                      |
| `SupportIPVSProxyMode`              | `false`      | Alpha      | 1.8                | 1.8                    |
| `SupportIPVSProxyMode`              | `false`      | Beta       | 1.9                | 1.9                    |
| `SupportIPVSProxyMode`              | `true`       | Beta       | 1.10               | 1.10                   |
| `SupportIPVSProxyMode`              | `true`       | GA         | 1.11               | -                      |
| `TaintNodesByCondition`             | `false`      | Alpha      | 1.8                | 1.11                   |
| `TaintNodesByCondition`             | `true`       | Beta       | 1.12               | 1.16                   |
| `TaintNodesByCondition`             | `true`       | GA         | 1.17               | -                      |
| `VolumeScheduling`                  | `false`      | Alpha      | 1.9                | 1.9                    |
| `VolumeScheduling`                  | `true`       | Beta       | 1.10               | 1.12                   |
| `VolumeScheduling`                  | `true`       | GA         | 1.13               | -                      |
| `VolumeSubpath`                     | `true`       | GA         | 1.13               | -                      |
| `VolumeSubpathEnvExpansion`         | `false`      | Alpha      | 1.14               | 1.14                   |
| `VolumeSubpathEnvExpansion`         | `true`       | Beta       | 1.15               | 1.16                   |
| `VolumeSubpathEnvExpansion`         | `true`       | GA         | 1.17               | -                      |
| `WatchBookmark`                     | `false`      | Alpha      | 1.15               | 1.15                   |
| `WatchBookmark`                     | `true`       | Beta       | 1.16               | 1.16                   |
| `WatchBookmark`                     | `true`       | GA         | 1.17               | -                      |

{{< /table >}}

## 機能を使用する

### 機能のステージ {#feature-stages}

機能には*Alpha* 、_Beta_ 、_GA_ の段階があります。_Alpha_ 機能とは：

- デフォルトでは無効になっています。
- バグがあるかもしれません。機能を有効にするとバグが発生する可能性があります。
- 機能のサポートは予告無しにいつでも削除される場合があります。
- API は今後のソフトウェアリリースで予告なく互換性の無い変更が行われる場合があり
  ます。
- バグが発生するリスクが高く長期的なサポートはないため、短期間のテストクラスター
  でのみ使用することをお勧めします。

_Beta_ 機能とは：

- デフォルトで有効になっています。
- この機能は十分にテストされていて、有効にすることは安全と考えられます。
- 詳細は変更される可能性がありますが、機能全体のサポートは削除されません。
- オブジェクトのスキーマやセマンティックは、その後のベータ版または安定版リリース
  で互換性の無い変更が行われる場合があります。互換性の無い変更が行われた場合には
  次のバージョンへの移行手順を提供します。これには API オブジェクトの削除、編集
  、および再作成が必要になる場合があります。バージョンアップにはいくつかの対応が
  必要な場合があります。これには機能に依存するアプリケーションのダウンタイムが発
  生する場合があります。
- 今後のリリースで互換性の無い変更が行われる可能性があるため、ビジネスクリティカ
  ルでない使用のみが推奨されます。個別にアップグレードできる複数のクラスターがあ
  る場合はこの制限を緩和できる場合があります。

{{< note >}} _ベータ版_ の機能を試してフィードバックをお寄せください！ GA になっ
てからさらなる変更を加えることは現実的ではない場合があります。 {{< /note >}}

_GA_ 機能とは(_GA_ 機能は*安定版* 機能とも呼ばれます):

- 機能は常に有効となり、無効にすることはできません。
- フィーチャーゲートの設定は不要になります。
- 機能の安定版は後続バージョンでリリースされたソフトウェアで使用されます。

### フィーチャーゲート

各フィーチャーゲートは特定の機能を有効/無効にするように設計されています。

- `Accelerators`: Docker での Nvidia GPU のサポートを有効にします。
- `AdvancedAuditing`:
  [高度な監査機能](/docs/tasks/debug-application-cluster/audit/#advanced-audit)を
  有効にします。
- `AffinityInAnnotations`(_非推奨_):
  [Pod のアフィニティまたはアンチアフィニティ](/ja/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity)を
  有効にします。
- `AllowExtTrafficLocalEndpoints`: サービスが外部へのリクエストをノードのローカ
  ルエンドポイントにルーティングできるようにします。
- `APIListChunking`: API クライアントが API サーバーからチャンク単位で
  （`LIST`や`GET`の）リソースを取得できるようにします。
  `APIPriorityAndFairness`: 各サーバーで優先順位付けと公平性を備えた要求の並行性
  を管理できるようにします(`RequestManagement`から名前が変更されました)。
- `APIResponseCompression`:`LIST`や`GET`リクエストの API レスポンスを圧縮します
  。
- `AppArmor`: Docker を使用する場合に Linux ノードで AppArmor による強制アクセス
  コントロールを有効にします。詳細
  は[AppArmor チュートリアル](/docs/tutorials/clusters/apparmor/)で確認できます
  。
- `AttachVolumeLimit`: ボリュームプラグインを有効にすることでノードにアタッチで
  きるボリューム数の制限を設定できます。
- `BalanceAttachedNodeVolumes`: スケジューリング中にバランスのとれたリソース割り
  当てを考慮するノードのボリュームカウントを含めます。判断を行う際に、CPU、メモ
  リー使用率、およびボリュームカウントが近いノードがスケジューラーによって優先さ
  れます。
- `BlockVolume`: Pod で Raw ブロックデバイスの定義と使用を有効にします。詳細
  は[Raw ブロックボリュームのサポート](/docs/concepts/storage/persistent-volumes/#raw-block-volume-support)で
  確認できます。
- `BoundServiceAccountTokenVolume`: ServiceAccountTokenVolumeProjection によって
  構成される計画ボリュームを使用するには ServiceAccount ボリュームを移行します。
  詳細
  は[Service Account Token Volumes](https://git.k8s.io/community/contributors/design-proposals/storage/svcacct-token-volume-source.md)で
  確認できます。
- `CPUManager`: コンテナレベルの CPU アフィニティサポートを有効します
  。[CPU マネジメントポリシー](/docs/tasks/administer-cluster/cpu-management-policies/)を
  見てください。
- `CRIContainerLogRotation`: cri コンテナランタイムのコンテナログローテーション
  を有効にします。
- `CSIBlockVolume`: 外部 CSI ボリュームドライバーを有効にしてブロックストレージ
  をサポートします。詳細
  は[`csi`Raw ブロックボリュームのサポート](/docs/concepts/storage/volumes/#csi-raw-block-volume-support)で
  確認できます。
- `CSIDriverRegistry`: csi.storage.k8s.io の CSIDriver API オブジェクトに関連す
  るすべてのロジックを有効にします。
- `CSIInlineVolume`: Pod の CSI インラインボリュームサポートを有効にします。
- `CSIMigration`: シムと変換ロジックを有効にしてボリューム操作を Kubernetes リポ
  ジトリー内のプラグインから対応した事前インストール済みの CSI プラグインにルー
  ティングします。
- `CSIMigrationAWS`: シムと変換ロジックを有効にしてボリューム操作を Kubernetes
  リポジトリー内の AWS-EBS プラグインから EBS CSI プラグインにルーティングします
  。ノードに EBS CSI プラグインがインストールおよび設定されていない場合、ツリー
  内の EBS プラグインへのフォールバックをサポートします。CSIMigration 機能フラグ
  を有効にする必要があります。
- `CSIMigrationAWSComplete`: EBS ツリー内プラグインの kubelet およびボリュームコ
  ントローラーへの登録を停止し、シムと変換ロジックを有効にして、AWS-EBS ツリー内
  プラグインから EBS CSI プラグインにボリューム操作をルーティングします。
  CSIMigration および CSIMigrationAWS 機能フラグを有効にし、クラスター内のすべて
  のノードに EBS CSI プラグインをインストールおよび設定する必要があります。
- `CSIMigrationAzureDisk`: シムと変換ロジックを有効にしてボリューム操作を
  Kubernetes リポジトリー内の Azure-Disk プラグインから Azure Disk CSI プラグイ
  ンにルーティングします。ノードに AzureDisk CSI プラグインがインストールおよび
  設定されていない場合、ツリー内の AzureDisk プラグインへのフォールバックをサポ
  ートします。CSIMigration 機能フラグを有効にする必要があります。
- `CSIMigrationAzureDiskComplete`: Azure-Disk ツリー内プラグインの kubelet およ
  びボリュームコントローラーへの登録を停止し、シムと変換ロジックを有効にして
  、Azure-Disk ツリー内プラグインから AzureDisk CSI プラグインにボリューム操作を
  ルーティングします。CSIMigration および CSIMigrationAzureDisk 機能フラグを有効
  にし、クラスター内のすべてのノードに AzureDisk CSI プラグインをインストールお
  よび設定する必要があります。
- `CSIMigrationAzureFile`: シムと変換ロジックを有効にしてボリューム操作を
  Kubernetes リポジトリー内の Azure-File プラグインから Azure File CSI プラグイ
  ンにルーティングします。ノードに AzureFile CSI プラグインがインストールおよび
  設定されていない場合、ツリー内の AzureFile プラグインへのフォールバックをサポ
  ートします。CSIMigration 機能フラグを有効にする必要があります。
- `CSIMigrationAzureFileComplete`: Azure-File ツリー内プラグインの kubelet およ
  びボリュームコントローラーへの登録を停止し、シムと変換ロジックを有効にして
  、Azure-File ツリー内プラグインから AzureFile CSI プラグインにボリューム操作を
  ルーティングします。CSIMigration および CSIMigrationAzureFile 機能フラグを有効
  にし、クラスター内のすべてのノードに AzureFile CSI プラグインをインストールお
  よび設定する必要があります。
- `CSIMigrationGCE`: シムと変換ロジックを有効にしてボリューム操作を Kubernetes
  リポジトリー内の GCE-PD プラグインから PD CSI プラグインにルーティングします。
  ノードに PD CSI プラグインがインストールおよび設定されていない場合、ツリー内の
  GCE プラグインへのフォールバックをサポートします。CSIMigration 機能フラグを有
  効にする必要があります。
- `CSIMigrationGCEComplete`: GCE-PD のツリー内プラグインの kubelet およびボリュ
  ームコントローラーへの登録を停止し、シムと変換ロジックが GCE-PD のツリー内プラ
  グインから PD CSI プラグインにボリューム操作をルーティングできるようにします
  。CSIMigration および CSIMigrationGCE 機能フラグを有効にし、クラスター内のすべ
  てのノードに PD CSI プラグインをインストールおよび設定する必要があります。
- `CSIMigrationOpenStack`: シムと変換ロジックを有効にしてボリューム操作を
  Kubernetes リポジトリー内の Cinder プラグインから Cinder CSI プラグインにルー
  ティングします。ノードに Cinder CSI プラグインがインストールおよび設定されてい
  ない場合、ツリー内の Cinder プラグインへのフォールバックをサポートします
  。CSIMigration 機能フラグを有効にする必要があります。
- `CSIMigrationOpenStackComplete`: Cinder のツリー内プラグインの kubelet および
  ボリュームコントローラーへの登録を停止し、シムと変換ロジックが Cinder のツリー
  内プラグインから Cinder CSI プラグインにボリューム操作をルーティングできるよう
  にします。CSIMigration および CSIMigrationOpenStack 機能フラグを有効にし、クラ
  スター内のすべてのノードに Cinder CSI プラグインをインストールおよび設定する必
  要があります。
- `CSINodeInfo`: csi.storage.k8s.io の CSINodeInfo API オブジェクトに関連するす
  べてのロジックを有効にします。
- `CSIPersistentVolume`:
  [CSI(Container Storage Interface)](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/storage/container-storage-interface.md)互
  換のボリュームプラグインを通してプロビジョニングされたボリュームの検出とマウン
  トを有効にします。詳細について
  は[`csi`ボリュームタイプ](/docs/concepts/storage/volumes/#csi)ドキュメントを確
  認してください。
- `CustomCPUCFSQuotaPeriod`: ノードが CPUCFSQuotaPeriod を変更できるようにします
  。
- `CustomPodDNS`: `dnsConfig`プロパティを使用した Pod の DNS 設定のカスタマイズ
  を有効にします。詳細
  は[Pod の DNS 構成](/docs/concepts/services-networking/dns-pod-service/#pods-dns-config)で
  確認できます。
- `CustomResourceDefaulting`: OpenAPI v3 バリデーションスキーマにおいて、デフォ
  ルト値の CRD サポートを有効にします。
- `CustomResourcePublishOpenAPI`: CRD の OpenAPI 仕様での公開を有効にします。
- `CustomResourceSubresources`:
  [CustomResourceDefinition](/docs/concepts/api-extension/custom-resources/)から
  作成されたリソースの`/status`および`/scale`サブリソースを有効にします。
- `CustomResourceValidation`:
  [CustomResourceDefinition](/docs/concepts/api-extension/custom-resources/)から
  作成されたリソースのスキーマによる検証を有効にします。
- `CustomResourceWebhookConversion`:
  [CustomResourceDefinition](/docs/concepts/api-extension/custom-resources/)から
  作成されたリソースの Webhook ベースの変換を有効にします。
- `DevicePlugins`:
  [device-plugins](/docs/concepts/cluster-administration/device-plugins/)による
  ノードでのリソースプロビジョニングを有効にします。
- `DryRun`: サーバーサイドで
  の[dry run](/docs/reference/using-api/api-concepts/#dry-run)リクエストを有効に
  します。
- `DynamicAuditing`:
  [動的監査](/docs/tasks/debug-application-cluster/audit/#dynamic-backend)を有効
  にします。
- `DynamicKubeletConfig`: kubelet の動的構成を有効にします
  。[kubelet の再設定](/docs/tasks/administer-cluster/reconfigure-kubelet/)を参
  照してください。
- `DynamicProvisioningScheduling`: デフォルトのスケジューラーを拡張してボリュー
  ムトポロジーを認識し PV プロビジョニングを処理します。この機能は、v1.12
  の`VolumeScheduling`機能に完全に置き換えられました。
- `DynamicVolumeProvisioning`(_非推奨_): Pod への永続ボリュー�
  の[動的プロビジョニング](/docs/concepts/storage/dynamic-provisioning/)を有効に
  します。
- `EnableAggregatedDiscoveryTimeout` (_非推奨_): 集約されたディスカバリーコール
  で 5 秒のタイムアウトを有効にします。
- `EnableEquivalenceClassCache`: Pod をスケジュールするときにスケジューラーがノ
  ードの同等をキャッシュできるようにします。
- `EphemeralContainers`: 稼働する Pod
  に{{< glossary_tooltip text="ephemeral containers" term_id="ephemeral-container" >}}を
  追加する機能を有効にします。
- `EvenPodsSpread`: Pod をトポロジードメイン全体で均等にスケジュールできるように
  します。[Even Pods Spread](/docs/concepts/configuration/even-pods-spread)をご
  覧ください。
- `ExpandInUsePersistentVolumes`: 使用中の PVC のボリューム拡張を有効にします
  。[使用中の PersistentVolumeClaim のサイズ変更](/docs/concepts/storage/persistent-volumes/#resizing-an-in-use-persistentvolumeclaim)を
  参照してください。
- `ExpandPersistentVolumes`: 永続ボリュームの拡張を有効にします
  。[永続ボリューム要求の拡張](/docs/concepts/storage/persistent-volumes/#expanding-persistent-volumes-claims)を
  参照してください。
- `ExperimentalCriticalPodAnnotation`:
  [スケジューリングが保証されるよう](/docs/tasks/administer-cluster/guaranteed-scheduling-critical-addon-pods/)に
  特定の pod への _クリティカル_ の注釈を加える設定を有効にします。
- `ExperimentalHostUserNamespaceDefaultingGate`: ホストするデフォルトのユーザー
  名前空間を有効にします。これは他のホストの名前空間やホストのマウントを使用して
  いるコンテナ、特権を持つコンテナ、または名前空間のない特定の機能（たとえ
  ば`MKNODE`、`SYS_MODULE`など）を使用しているコンテナ用です。これは Docker デー
  モンでユーザー名前空間の再マッピングが有効になっている場合にのみ有効にすべきで
  す。
- `EndpointSlice`: よりスケーラブルで拡張可能なネットワークエンドポイントのエン
  ドポイントスライスを有効にします。対応する API とコントローラーを有効にする必
  要があります
  。[Enabling Endpoint Slices](/docs/tasks/administer-cluster/enabling-endpointslices/)を
  ご覧ください。
- `GCERegionalPersistentDisk`: GCE でリージョナル PD 機能を有効にします。
- `HugePages`: 事前に割り当てられ
  た[huge pages](/docs/tasks/manage-hugepages/scheduling-hugepages/)の割り当てと
  消費を有効にします。
- `HyperVContainer`: Windows コンテナ
  の[Hyper-V による分離](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/hyperv-container)を
  有効にします。
- `HPAScaleToZero`: カスタムメトリクスまたは外部メトリクスを使用するときに
  、`HorizontalPodAutoscaler`リソースの`minReplicas`を 0 に設定できるようにしま
  す。
- `KubeletConfigFile`: 設定ファイルを使用して指定されたファイルからの kubelet 設
  定の読み込みを有効にします。詳細
  は[設定ファイルによる kubelet パラメーターの設定](/docs/tasks/administer-cluster/kubelet-config-file/)で
  確認できます。
- `KubeletPluginsWatcher`: 調査ベースのプラグイン監視ユーティリティを有効にして
  kubelet が[CSI ボリュームドライバー](/docs/concepts/storage/volumes/#csi)など
  のプラグインを検出できるようにします。
- `KubeletPodResources`: kubelet の pod のリソース grpc エンドポイントを有効にし
  ます。詳細
  は[デバイスモニタリングのサポート](https://git.k8s.io/community/keps/sig-node/compute-device-assignment.md)で
  確認できます。
- `LegacyNodeRoleBehavior`: 無効にすると、サービスロードバランサーの従来の動作と
  ノードの中断により機能固有のラベルが優先され
  、`node-role.kubernetes.io/master`ラベルが無視されます。
- `LocalStorageCapacityIsolation`:
  [ローカルの一時ストレージ](/docs/concepts/configuration/manage-compute-resources-container/)の
  消費を有効にして
  、[emptyDir ボリューム](/docs/concepts/storage/volumes/#emptydir)の`sizeLimit`プ
  ロパティも有効にします。
- `LocalStorageCapacityIsolationFSQuotaMonitoring`:
  `LocalStorageCapacityIsolation`が[ローカルの一時ストレージ](/docs/concepts/configuration/manage-compute-resources-container/)で
  有効になっていて
  、[emptyDir ボリューム](/docs/concepts/storage/volumes/#emptydir)の backing
  filesystem がプロジェクトクォータをサポートし有効になっている場合、プロジェク
  トクォータを使用して、パフォーマンスと精度を向上させるために、ファイルシステ�
  へのアクセスではな
  く[emptyDir ボリューム](/docs/concepts/storage/volumes/#emptydir)ストレージ消
  費を監視します。
- `MountContainers`: ホスト上のユーティリティコンテナをボリュームマウンターとし
  て使用できるようにします。
- `MountPropagation`: あるコンテナによってマウントされたボリュームを他のコンテナ
  または pod に共有できるようにします。詳細
  は[マウントの伝播](/docs/concepts/storage/volumes/#mount-propagation)で確認で
  きます。
- `NodeDisruptionExclusion`: ノードラベ
  ル`node.kubernetes.io/exclude-disruption`の使用を有効にします。これにより、ゾ
  ーン障害時にノードが退避するのを防ぎます。
- `NodeLease`: 新しい Lease API を有効にしてノードヘルスシグナルとして使用できる
  ノードのハートビートをレポートします。
- `NonPreemptingPriority`: PriorityClass と Pod の NonPreempting オプションを有
  効にします。
- `PersistentLocalVolumes`: Pod で`local`ボリュームタイプの使用を有効にします
  。`local`ボリュームを要求する場合、pod アフィニティを指定する必要があります。
- `PodOverhead`: [PodOverhead](/docs/concepts/configuration/pod-overhead/)機能を
  有効にして、Pod のオーバーヘッドを考慮するようにします。
- `PodPriority`:
  [優先度](/docs/concepts/configuration/pod-priority-preemption/)に基づいて Pod
  の再スケジューリングとプリエンプションを有効にします。
- `PodReadinessGates`: Pod の readiness の評価を拡張するため
  に`PodReadinessGate`フィールドの設定を有効にします。詳細
  は[Pod readiness gate](/docs/concepts/workloads/pods/pod-lifecycle/#pod-readiness-gate)で
  確認できます。
- `PodShareProcessNamespace`: Pod で実行されているコンテナ間で単一のプロセス名前
  空間を共有するには、Pod で`shareProcessNamespace`の設定を有効にします。 詳細に
  ついては
  、[Pod 内のコンテナ間でプロセス名前空間を共有する](/docs/tasks/configure-pod-container/share-process-namespace/)を
  ご覧ください。
- `ProcMountType`: コンテナの ProcMountType の制御を有効にします。
- `PVCProtection`: 永続ボリューム要求（PVC）が Pod でまだ使用されているときに削
  除されないようにします。詳細
  は[ここ](/docs/tasks/administer-cluster/storage-object-in-use-protection/)で確
  認できます。
- `QOSReserved`: QoS レベルでのリソース予約を許可して、低い QoS レベルのポッドが
  高い QoS レベルで要求されたリソースにバーストするのを防ぎます（現時点ではメモ
  リのみ）。
- `ResourceLimitsPriorityFunction`: 入力した Pod の CPU 制限とメモリ制限の少なく
  とも 1 つを満たすノードに対して最低スコアを 1 に割り当てるスケジューラー優先機
  能を有効にします。その目的は同じスコアを持つノード間の関係を断つことです。
- `ResourceQuotaScopeSelectors`: リソース割当のスコープセレクターを有効にします
  。
- `RotateKubeletClientCertificate`: kubelet でクライアント TLS 証明書のローテー
  ションを有効にします。詳細
  は[kubelet の設定](/docs/tasks/administer-cluster/storage-object-in-use-protection/)で
  確認できます。
- `RotateKubeletServerCertificate`: kubelet でサーバー TLS 証明書のローテーショ
  ンを有効にします。詳細
  は[kubelet の設定](/docs/tasks/administer-cluster/storage-object-in-use-protection/)で
  確認できます。
- `RunAsGroup`: コンテナの初期化プロセスで設定されたプライマリグループ ID の制御
  を有効にします。
- `RuntimeClass`: コンテナのランタイム構成を選択するに
  は[RuntimeClass](/docs/concepts/containers/runtime-class/)機能を有効にします。
- `ScheduleDaemonSetPods`: DaemonSet の Pod を DaemonSet コントローラーではなく
  、デフォルトのスケジューラーによってスケジュールされるようにします。
- `SCTPSupport`: `Service`、`Endpoints`、`NetworkPolicy`、`Pod`の定義
  で`protocol`の値として SCTP を使用できるようにします
- `ServerSideApply`: API サーバー
  で[サーバーサイド Apply(SSA)](/docs/reference/using-api/api-concepts/#server-side-apply)の
  パスを有効にします。
- `ServiceLoadBalancerFinalizer`: サービスロードバランサーのファイナライザー保護
  を有効にします。
- `ServiceNodeExclusion`: クラウドプロバイダーによって作成されたロードバランサー
  からのノードの除外を有効にします
  。"`alpha.service-controller.kubernetes.io/exclude-balancer`"キーまた
  は`node.kubernetes.io/exclude-from-external-load-balancers`でラベル付けされて
  いる場合ノードは除外の対象となります。
- `StartupProbe`: kubelet
  で[startup](/docs/concepts/workloads/pods/pod-lifecycle/#when-should-you-use-a-startup-probe)プ
  ローブを有効にします。
- `StorageObjectInUseProtection`: PersistentVolume または PersistentVolumeClaim
  オブジェクトがまだ使用されている場合、それらの削除を延期します。
- `StorageVersionHash`: apiservers がディスカバリーでストレージのバージョンハッ
  シュを公開できるようにします。
- `StreamingProxyRedirects`: ストリーミングリクエストのバックエンド(kubelet)から
  のリダイレクトをインターセプト（およびフォロー）するよう API サーバーに指示し
  ます。ストリーミングリクエストの例には`exec`、`attach`、`port-forward`リクエス
  トが含まれます。
- `SupportIPVSProxyMode`: IPVS を使用したクラスター内サービスの負荷分散の提供を
  有効にします。詳細
  は[サービスプロキシー](/ja/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies)で
  確認できます。
- `SupportPodPidsLimit`: Pod の PID 制限のサポートを有効にします。
- `Sysctls`: 各 pod に設定できる名前空間付きのカーネルパラメーター(sysctl)のサポ
  ートを有効にします。詳細
  は[sysctls](/docs/tasks/administer-cluster/sysctl-cluster/)で確認できます。
- `TaintBasedEvictions`: ノードの汚染と pod の許容に基づいてノードから pod を排
  除できるようにします。。詳細
  は[汚染と許容](/docs/concepts/configuration/taint-and-toleration/)で確認できま
  す。
- `TaintNodesByCondition`:
  [ノードの条件](/ja/docs/concepts/architecture/nodes/#condition)に基づいてノー
  ドの自動汚染を有効にします。
- `TokenRequest`: サービスアカウントリソースで`TokenRequest`エンドポイントを有効
  にします。
- `TokenRequestProjection`:
  [Projected ボリューム](/docs/concepts/storage/volumes/#projected)を使用した
  pod へのサービスアカウントのトークンの注入を有効にします。
- `TTLAfterFinished`:
  [TTL コントローラー](/docs/concepts/workloads/controllers/ttlafterfinished/)が
  実行終了後にリソースをクリーンアップできるようにします。
- `VolumePVCDataSource`: 既存の PVC をデータソースとして指定するサポートを有効に
  します。
- `VolumeScheduling`: ボリュームトポロジー対応のスケジューリングを有効にし
  、PersistentVolumeClaim（PVC）バインディングにスケジューリングの決定を認識させ
  ます。また`PersistentLocalVolumes`フィーチャーゲートと一緒に使用する
  と[`local`](/docs/concepts/storage/volumes/#local)ボリュームタイプの使用が可能
  になります。
- `VolumeSnapshotDataSource`: ボリュームスナップショットのデータソースサポートを
  有効にします。
- `VolumeSubpathEnvExpansion`: 環境変数を`subPath`に展開するため
  の`subPathExpr`フィールドを有効にします。
- `WatchBookmark`: ブックマークイベントの監視サポートを有効にします。
- `WindowsGMSA`: GMSA 資格仕様を pod からコンテナランタイムに渡せるようにします
  。
- `WinDSR`: kube-proxy が Windows 用の DSR ロードバランサーを作成できるようにし
  ます。
- `WinOverlay`: kube-proxy を Windows のオーバーレイモードで実行できるようにしま
  す。

{{% /capture %}} {{% capture whatsnext %}}

- Kubernetes の[非推奨ポリシー](/docs/reference/using-api/deprecation-policy/)で
  は、機能とコンポーネントを削除するためのプロジェクトのアプローチを説明していま
  す。 {{% /capture %}}
