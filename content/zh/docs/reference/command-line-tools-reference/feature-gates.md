---
weight: 10
title: 特性门控
content_template: templates/concept
---

## <!--

weight: 10 title: Feature Gates content_template: templates/concept

---

-->

{{% capture overview %}}

<!--
This page contains an overview of the various feature gates an administrator
can specify on different Kubernetes components.

See [feature stages](#feature-stages) for an explanation of the stages for a feature.
-->

本页详述了管理员可以在不同的 Kubernetes 组件上指定的各种特性门控。

关于特性各个阶段的说明，请参见[特性阶段](#feature-stages)。

{{% /capture %}}

{{% capture body %}}

<!--
## Overview
-->

## 概述

<!--
Feature gates are a set of key=value pairs that describe Kubernetes features.
You can turn these features on or off using the `--feature-gates` command line flag
on each Kubernetes component.
-->

特性门控是描述 Kubernetes 特性的一组键值对。您可以在 Kubernetes 的每一个组件中使
用 `--feature-gates` flag 来启用或禁用这些特性。

<!--
Each Kubernetes component lets you enable or disable a set of feature gates that
are relevant to that component.
Use `-h` flag to see a full set of feature gates for all components.
To set feature gates for a component, such as kubelet, use the `--feature-gates` flag assigned to a list of feature pairs:
-->

每个 Kubernetes 组件都支持启用或禁用与该组件相关的一组特性门控。使用 `-h` 参数来
查看所有组件支持的完整特性门控。要为诸如 kubelet 之类的组件设置特性门控，请使用
`--feature-gates` 参数，并向其传递一组特性：

```shell
--feature-gates="...,DynamicKubeletConfig=true"
```

<!--
The following tables are a summary of the feature gates that you can set on
different Kubernetes components.
-->

下表总结了在不同的 Kubernetes 组件上可以设置的特性门控。

<!--
- The "Since" column contains the Kubernetes release when a feature is introduced
  or its release stage is changed.
- The "Until" column, if not empty, contains the last Kubernetes release in which
  you can still use a feature gate.
- If a feature is in the Alpha or Beta state, you can find the feature listed
  in the [Alpha/Beta feature gate table](#feature-gates-for-alpha-or-beta-features).
- If a feature is stable you can find all stages for that feature listed in the
  [Graduated/Deprecated feature gate table](#feature-gates-for-graduated-or-deprecated-features).
- The [Graduated/Deprecated feature gate table](#feature-gates-for-graduated-or-deprecated-features)
  also lists deprecated and withdrawn features.
-->

- 引入特性或更改其发布阶段后，"Since" 列将包含 Kubernetes 版本。
- "Until" 列（如果不为空）包含最后一个 Kubernetes 版本，您仍可以在其中使用特性门
  控。
- 如果某个特性处于 Alpha 或 Beta 状态，您可以在
  [Alpha 和 Beta 特性门控表](#feature-gates-for-alpha-or-beta-features)中找到该
  特性。
- 如果某个特性处于稳定状态，您可以
  在[毕业和废弃特性门控表](#feature-gates-for-graduated-or-deprecated-features).
  中找到该特性的所有阶段。
- [毕业和废弃特性门控表](#feature-gates-for-graduated-or-deprecated-features) 还
  列出了废弃的和已被移除的特性。

<!--
### Feature gates for Alpha or Beta features

{{< table caption="Feature gates for features in Alpha or Beta states" >}}

| Feature | Default | Stage | Since | Until |

{{< /table >}}
-->

### Alpha 和 Beta 的特性门控

{{< table caption="处于 Alpha 或 Beta 状态的特性门控" >}}

| 特性                                             | 默认值  | 状态  | 开始(Since) | 结束(Until) |
| ------------------------------------------------ | ------- | ----- | ----------- | ----------- |
| `APIListChunking`                                | `false` | Alpha | 1.8         | 1.8         |
| `APIListChunking`                                | `true`  | Beta  | 1.9         |             |
| `APIPriorityAndFairness`                         | `false` | Alpha | 1.17        |             |
| `APIResponseCompression`                         | `false` | Alpha | 1.7         |             |
| `AppArmor`                                       | `true`  | Beta  | 1.4         |             |
| `BalanceAttachedNodeVolumes`                     | `false` | Alpha | 1.11        |             |
| `BlockVolume`                                    | `false` | Alpha | 1.9         | 1.12        |
| `BlockVolume`                                    | `true`  | Beta  | 1.13        | -           |
| `BoundServiceAccountTokenVolume`                 | `false` | Alpha | 1.13        |             |
| `CPUManager`                                     | `false` | Alpha | 1.8         | 1.9         |
| `CPUManager`                                     | `true`  | Beta  | 1.10        |             |
| `CRIContainerLogRotation`                        | `false` | Alpha | 1.10        | 1.10        |
| `CRIContainerLogRotation`                        | `true`  | Beta  | 1.11        |             |
| `CSIBlockVolume`                                 | `false` | Alpha | 1.11        | 1.13        |
| `CSIBlockVolume`                                 | `true`  | Beta  | 1.14        |             |
| `CSIDriverRegistry`                              | `false` | Alpha | 1.12        | 1.13        |
| `CSIDriverRegistry`                              | `true`  | Beta  | 1.14        |             |
| `CSIInlineVolume`                                | `false` | Alpha | 1.15        | 1.15        |
| `CSIInlineVolume`                                | `true`  | Beta  | 1.16        | -           |
| `CSIMigration`                                   | `false` | Alpha | 1.14        | 1.16        |
| `CSIMigration`                                   | `true`  | Beta  | 1.17        |             |
| `CSIMigrationAWS`                                | `false` | Alpha | 1.14        |             |
| `CSIMigrationAWS`                                | `false` | Beta  | 1.17        |             |
| `CSIMigrationAWSComplete`                        | `false` | Alpha | 1.17        |             |
| `CSIMigrationAzureDisk`                          | `false` | Alpha | 1.15        |             |
| `CSIMigrationAzureDiskComplete`                  | `false` | Alpha | 1.17        |             |
| `CSIMigrationAzureFile`                          | `false` | Alpha | 1.15        |             |
| `CSIMigrationAzureFileComplete`                  | `false` | Alpha | 1.17        |             |
| `CSIMigrationGCE`                                | `false` | Alpha | 1.14        | 1.16        |
| `CSIMigrationGCE`                                | `false` | Beta  | 1.17        |             |
| `CSIMigrationGCEComplete`                        | `false` | Alpha | 1.17        |             |
| `CSIMigrationOpenStack`                          | `false` | Alpha | 1.14        |             |
| `CSIMigrationOpenStackComplete`                  | `false` | Alpha | 1.17        |             |
| `CustomCPUCFSQuotaPeriod`                        | `false` | Alpha | 1.12        |             |
| `CustomResourceDefaulting`                       | `false` | Alpha | 1.15        | 1.15        |
| `CustomResourceDefaulting`                       | `true`  | Beta  | 1.16        |             |
| `DevicePlugins`                                  | `false` | Alpha | 1.8         | 1.9         |
| `DevicePlugins`                                  | `true`  | Beta  | 1.10        |             |
| `DryRun`                                         | `false` | Alpha | 1.12        | 1.12        |
| `DryRun`                                         | `true`  | Beta  | 1.13        |             |
| `DynamicAuditing`                                | `false` | Alpha | 1.13        |             |
| `DynamicKubeletConfig`                           | `false` | Alpha | 1.4         | 1.10        |
| `DynamicKubeletConfig`                           | `true`  | Beta  | 1.11        |             |
| `EndpointSlice`                                  | `false` | Alpha | 1.16        | 1.16        |
| `EndpointSlice`                                  | `false` | Beta  | 1.17        |             |
| `EphemeralContainers`                            | `false` | Alpha | 1.16        |             |
| `ExpandCSIVolumes`                               | `false` | Alpha | 1.14        | 1.15        |
| `ExpandCSIVolumes`                               | `true`  | Beta  | 1.16        |             |
| `ExpandInUsePersistentVolumes`                   | `false` | Alpha | 1.11        | 1.14        |
| `ExpandInUsePersistentVolumes`                   | `true`  | Beta  | 1.15        |             |
| `ExpandPersistentVolumes`                        | `false` | Alpha | 1.8         | 1.10        |
| `ExpandPersistentVolumes`                        | `true`  | Beta  | 1.11        |             |
| `ExperimentalHostUserNamespaceDefaulting`        | `false` | Beta  | 1.5         |             |
| `EvenPodsSpread`                                 | `false` | Alpha | 1.16        |             |
| `HPAScaleToZero`                                 | `false` | Alpha | 1.16        |             |
| `HyperVContainer`                                | `false` | Alpha | 1.10        |             |
| `KubeletPodResources`                            | `false` | Alpha | 1.13        | 1.14        |
| `KubeletPodResources`                            | `true`  | Beta  | 1.15        |             |
| `LegacyNodeRoleBehavior`                         | `true`  | Alpha | 1.16        |             |
| `LocalStorageCapacityIsolation`                  | `false` | Alpha | 1.7         | 1.9         |
| `LocalStorageCapacityIsolation`                  | `true`  | Beta  | 1.10        |             |
| `LocalStorageCapacityIsolationFSQuotaMonitoring` | `false` | Alpha | 1.15        |             |
| `MountContainers`                                | `false` | Alpha | 1.9         |             |
| `NodeDisruptionExclusion`                        | `false` | Alpha | 1.16        |             |
| `NonPreemptingPriority`                          | `false` | Alpha | 1.15        |             |
| `PodOverhead`                                    | `false` | Alpha | 1.16        | -           |
| `ProcMountType`                                  | `false` | Alpha | 1.12        |             |
| `QOSReserved`                                    | `false` | Alpha | 1.11        |             |
| `RemainingItemCount`                             | `false` | Alpha | 1.15        |             |
| `ResourceLimitsPriorityFunction`                 | `false` | Alpha | 1.9         |             |
| `RotateKubeletClientCertificate`                 | `true`  | Beta  | 1.8         |             |
| `RotateKubeletServerCertificate`                 | `false` | Alpha | 1.7         | 1.11        |
| `RotateKubeletServerCertificate`                 | `true`  | Beta  | 1.12        |             |
| `RunAsGroup`                                     | `true`  | Beta  | 1.14        |             |
| `RuntimeClass`                                   | `false` | Alpha | 1.12        | 1.13        |
| `RuntimeClass`                                   | `true`  | Beta  | 1.14        |             |
| `SCTPSupport`                                    | `false` | Alpha | 1.12        |             |
| `ServerSideApply`                                | `false` | Alpha | 1.14        | 1.15        |
| `ServerSideApply`                                | `true`  | Beta  | 1.16        |             |
| `ServiceNodeExclusion`                           | `false` | Alpha | 1.8         |             |
| `ServiceTopology`                                | `false` | Alpha | 1.17        |             |
| `StartupProbe`                                   | `false` | Alpha | 1.16        |             |
| `StorageVersionHash`                             | `false` | Alpha | 1.14        | 1.14        |
| `StorageVersionHash`                             | `true`  | Beta  | 1.15        |             |
| `StreamingProxyRedirects`                        | `false` | Beta  | 1.5         | 1.5         |
| `StreamingProxyRedirects`                        | `true`  | Beta  | 1.6         |             |
| `SupportNodePidsLimit`                           | `false` | Alpha | 1.14        | 1.14        |
| `SupportNodePidsLimit`                           | `true`  | Beta  | 1.15        |             |
| `SupportPodPidsLimit`                            | `false` | Alpha | 1.10        | 1.13        |
| `SupportPodPidsLimit`                            | `true`  | Beta  | 1.14        |             |
| `Sysctls`                                        | `true`  | Beta  | 1.11        |             |
| `TaintBasedEvictions`                            | `false` | Alpha | 1.6         | 1.12        |
| `TaintBasedEvictions`                            | `true`  | Beta  | 1.13        |             |
| `TokenRequest`                                   | `false` | Alpha | 1.10        | 1.11        |
| `TokenRequest`                                   | `true`  | Beta  | 1.12        |             |
| `TokenRequestProjection`                         | `false` | Alpha | 1.11        | 1.11        |
| `TokenRequestProjection`                         | `true`  | Beta  | 1.12        |             |
| `TTLAfterFinished`                               | `false` | Alpha | 1.12        |             |
| `TopologyManager`                                | `false` | Alpha | 1.16        |             |
| `ValidateProxyRedirects`                         | `false` | Alpha | 1.12        | 1.13        |
| `ValidateProxyRedirects`                         | `true`  | Beta  | 1.14        |             |
| `VolumePVCDataSource`                            | `false` | Alpha | 1.15        | 1.15        |
| `VolumePVCDataSource`                            | `true`  | Beta  | 1.16        |             |
| `VolumeSnapshotDataSource`                       | `false` | Alpha | 1.12        | 1.16        |
| `VolumeSnapshotDataSource`                       | `true`  | Beta  | 1.17        | -           |
| `WindowsGMSA`                                    | `false` | Alpha | 1.14        |             |
| `WindowsGMSA`                                    | `true`  | Beta  | 1.16        |             |
| `WinDSR`                                         | `false` | Alpha | 1.14        |             |
| `WinOverlay`                                     | `false` | Alpha | 1.14        |             |

{{< /table >}}

<!--
### Feature gates for graduated or deprecated features

{{< table caption="Feature Gates for Graduated or Deprecated Features" >}}

| Feature | Default | Stage | Since | Until |

{{< /table >}}
-->

### 已毕业和不推荐使用的特性门控

{{< table caption="已毕业或不推荐使用的特性门控" >}}

| 特性                                | 默认值  | 状态       | 开始(Since) | 结束(Until) |
| ----------------------------------- | ------- | ---------- | ----------- | ----------- |
| `Accelerators`                      | `false` | Alpha      | 1.6         | 1.10        |
| `Accelerators`                      | -       | Deprecated | 1.11        | -           |
| `AdvancedAuditing`                  | `false` | Alpha      | 1.7         | 1.7         |
| `AdvancedAuditing`                  | `true`  | Beta       | 1.8         | 1.11        |
| `AdvancedAuditing`                  | `true`  | GA         | 1.12        | -           |
| `AffinityInAnnotations`             | `false` | Alpha      | 1.6         | 1.7         |
| `AffinityInAnnotations`             | -       | Deprecated | 1.8         | -           |
| `AllowExtTrafficLocalEndpoints`     | `false` | Beta       | 1.4         | 1.6         |
| `AllowExtTrafficLocalEndpoints`     | `true`  | GA         | 1.7         | -           |
| `CSINodeInfo`                       | `false` | Alpha      | 1.12        | 1.13        |
| `CSINodeInfo`                       | `true`  | Beta       | 1.14        | 1.16        |
| `CSINodeInfo`                       | `true`  | GA         | 1.17        |             |
| `AttachVolumeLimit`                 | `false` | Alpha      | 1.11        | 1.11        |
| `AttachVolumeLimit`                 | `true`  | Beta       | 1.12        | 1.16        |
| `AttachVolumeLimit`                 | `true`  | GA         | 1.17        | -           |
| `CSIPersistentVolume`               | `false` | Alpha      | 1.9         | 1.9         |
| `CSIPersistentVolume`               | `true`  | Beta       | 1.10        | 1.12        |
| `CSIPersistentVolume`               | `true`  | GA         | 1.13        | -           |
| `CustomPodDNS`                      | `false` | Alpha      | 1.9         | 1.9         |
| `CustomPodDNS`                      | `true`  | Beta       | 1.10        | 1.13        |
| `CustomPodDNS`                      | `true`  | GA         | 1.14        | -           |
| `CustomResourcePublishOpenAPI`      | `false` | Alpha      | 1.14        | 1.14        |
| `CustomResourcePublishOpenAPI`      | `true`  | Beta       | 1.15        | 1.15        |
| `CustomResourcePublishOpenAPI`      | `true`  | GA         | 1.16        | -           |
| `CustomResourceSubresources`        | `false` | Alpha      | 1.10        | 1.10        |
| `CustomResourceSubresources`        | `true`  | Beta       | 1.11        | 1.15        |
| `CustomResourceSubresources`        | `true`  | GA         | 1.16        | -           |
| `CustomResourceValidation`          | `false` | Alpha      | 1.8         | 1.8         |
| `CustomResourceValidation`          | `true`  | Beta       | 1.9         | 1.15        |
| `CustomResourceValidation`          | `true`  | GA         | 1.16        | -           |
| `CustomResourceWebhookConversion`   | `false` | Alpha      | 1.13        | 1.14        |
| `CustomResourceWebhookConversion`   | `true`  | Beta       | 1.15        | 1.15        |
| `CustomResourceWebhookConversion`   | `true`  | GA         | 1.16        | -           |
| `DynamicProvisioningScheduling`     | `false` | Alpha      | 1.11        | 1.11        |
| `DynamicProvisioningScheduling`     | -       | Deprecated | 1.12        | -           |
| `DynamicVolumeProvisioning`         | `true`  | Alpha      | 1.3         | 1.7         |
| `DynamicVolumeProvisioning`         | `true`  | GA         | 1.8         | -           |
| `EnableEquivalenceClassCache`       | `false` | Alpha      | 1.8         | 1.14        |
| `EnableEquivalenceClassCache`       | -       | Deprecated | 1.15        | -           |
| `ExperimentalCriticalPodAnnotation` | `false` | Alpha      | 1.5         | 1.12        |
| `ExperimentalCriticalPodAnnotation` | `false` | Deprecated | 1.13        | -           |
| `GCERegionalPersistentDisk`         | `true`  | Beta       | 1.10        | 1.12        |
| `GCERegionalPersistentDisk`         | `true`  | GA         | 1.13        | -           |
| `HugePages`                         | `false` | Alpha      | 1.8         | 1.9         |
| `HugePages`                         | `true`  | Beta       | 1.10        | 1.13        |
| `HugePages`                         | `true`  | GA         | 1.14        | -           |
| `Initializers`                      | `false` | Alpha      | 1.7         | 1.13        |
| `Initializers`                      | -       | Deprecated | 1.14        | -           |
| `KubeletConfigFile`                 | `false` | Alpha      | 1.8         | 1.9         |
| `KubeletConfigFile`                 | -       | Deprecated | 1.10        | -           |
| `KubeletPluginsWatcher`             | `false` | Alpha      | 1.11        | 1.11        |
| `KubeletPluginsWatcher`             | `true`  | Beta       | 1.12        | 1.12        |
| `KubeletPluginsWatcher`             | `true`  | GA         | 1.13        | -           |
| `MountPropagation`                  | `false` | Alpha      | 1.8         | 1.9         |
| `MountPropagation`                  | `true`  | Beta       | 1.10        | 1.11        |
| `MountPropagation`                  | `true`  | GA         | 1.12        | -           |
| `NodeLease`                         | `false` | Alpha      | 1.12        | 1.13        |
| `NodeLease`                         | `true`  | Beta       | 1.14        | 1.16        |
| `NodeLease`                         | `true`  | GA         | 1.17        | -           |
| `PersistentLocalVolumes`            | `false` | Alpha      | 1.7         | 1.9         |
| `PersistentLocalVolumes`            | `true`  | Beta       | 1.10        | 1.13        |
| `PersistentLocalVolumes`            | `true`  | GA         | 1.14        | -           |
| `PodPriority`                       | `false` | Alpha      | 1.8         | 1.10        |
| `PodPriority`                       | `true`  | Beta       | 1.11        | 1.13        |
| `PodPriority`                       | `true`  | GA         | 1.14        | -           |
| `PodReadinessGates`                 | `false` | Alpha      | 1.11        | 1.11        |
| `PodReadinessGates`                 | `true`  | Beta       | 1.12        | 1.13        |
| `PodReadinessGates`                 | `true`  | GA         | 1.14        | -           |
| `PodShareProcessNamespace`          | `false` | Alpha      | 1.10        | 1.11        |
| `PodShareProcessNamespace`          | `true`  | Beta       | 1.12        | 1.16        |
| `PodShareProcessNamespace`          | `true`  | GA         | 1.17        | -           |
| `PVCProtection`                     | `false` | Alpha      | 1.9         | 1.9         |
| `PVCProtection`                     | -       | Deprecated | 1.10        | -           |
| `RequestManagement`                 | `false` | Alpha      | 1.15        | 1.16        |
| `ResourceQuotaScopeSelectors`       | `false` | Alpha      | 1.11        | 1.11        |
| `ResourceQuotaScopeSelectors`       | `true`  | Beta       | 1.12        | 1.16        |
| `ResourceQuotaScopeSelectors`       | `true`  | GA         | 1.17        | -           |
| `ScheduleDaemonSetPods`             | `false` | Alpha      | 1.11        | 1.11        |
| `ScheduleDaemonSetPods`             | `true`  | Beta       | 1.12        | 1.16        |
| `ScheduleDaemonSetPods`             | `true`  | GA         | 1.17        | -           |
| `ServiceLoadBalancerFinalizer`      | `false` | Alpha      | 1.15        | 1.15        |
| `ServiceLoadBalancerFinalizer`      | `true`  | Beta       | 1.16        | 1.16        |
| `ServiceLoadBalancerFinalizer`      | `true`  | GA         | 1.17        | -           |
| `StorageObjectInUseProtection`      | `true`  | Beta       | 1.10        | 1.10        |
| `StorageObjectInUseProtection`      | `true`  | GA         | 1.11        | -           |
| `SupportIPVSProxyMode`              | `false` | Alpha      | 1.8         | 1.8         |
| `SupportIPVSProxyMode`              | `false` | Beta       | 1.9         | 1.9         |
| `SupportIPVSProxyMode`              | `true`  | Beta       | 1.10        | 1.10        |
| `SupportIPVSProxyMode`              | `true`  | GA         | 1.11        | -           |
| `TaintNodesByCondition`             | `false` | Alpha      | 1.8         | 1.11        |
| `TaintNodesByCondition`             | `true`  | Beta       | 1.12        | 1.16        |
| `TaintNodesByCondition`             | `true`  | GA         | 1.17        | -           |
| `VolumeScheduling`                  | `false` | Alpha      | 1.9         | 1.9         |
| `VolumeScheduling`                  | `true`  | Beta       | 1.10        | 1.12        |
| `VolumeScheduling`                  | `true`  | GA         | 1.13        | -           |
| `VolumeSubpath`                     | `true`  | GA         | 1.13        | -           |
| `VolumeSubpathEnvExpansion`         | `false` | Alpha      | 1.14        | 1.14        |
| `VolumeSubpathEnvExpansion`         | `true`  | Beta       | 1.15        | 1.16        |
| `VolumeSubpathEnvExpansion`         | `true`  | GA         | 1.17        | -           |
| `WatchBookmark`                     | `false` | Alpha      | 1.15        | 1.15        |
| `WatchBookmark`                     | `true`  | Beta       | 1.16        | 1.16        |
| `WatchBookmark`                     | `true`  | GA         | 1.17        | -           |

{{< /table >}}

<!--
## Using a feature

### Feature stages
-->

## 使用特性

### 特性阶段

<!--
A feature can be in *Alpha*, *Beta* or *GA* stage.
An *Alpha* feature means:
-->

处于 _Alpha_ 、_Beta_ 、 _GA_ 阶段的特性。

_Alpha_ 特性代表：

<!--
* Disabled by default.
* Might be buggy. Enabling the feature may expose bugs.
* Support for feature may be dropped at any time without notice.
* The API may change in incompatible ways in a later software release without notice.
* Recommended for use only in short-lived testing clusters, due to increased
  risk of bugs and lack of long-term support.
-->

- 默认禁用。
- 可能有错误，启用此特性可能会导致错误。
- 随时可能删除对此特性的支持，恕不另行通知。
- 在以后的软件版本中，API 可能会以不兼容的方式更改，恕不另行通知。
- 建议将其仅用于短期测试中，因为开启特性会增加错误的风险，并且缺乏长期支持。

<!--
A *Beta* feature means:
-->

_Beta_ 特性代表：

<!--
* Enabled by default.
* The feature is well tested. Enabling the feature is considered safe.
* Support for the overall feature will not be dropped, though details may change.
* The schema and/or semantics of objects may change in incompatible ways in a
  subsequent beta or stable release. When this happens, we will provide instructions
  for migrating to the next version. This may require deleting, editing, and
  re-creating API objects. The editing process may require some thought.
  This may require downtime for applications that rely on the feature.
* Recommended for only non-business-critical uses because of potential for
  incompatible changes in subsequent releases. If you have multiple clusters
  that can be upgraded independently, you may be able to relax this restriction.
-->

- 默认禁用。
- 该特性已经经过良好测试。启用该特性是安全的。
- 尽管详细信息可能会更改，但不会放弃对整体特性的支持。
- 对象的架构或语义可能会在随后的 Beta 或稳定版本中以不兼容的方式更改。当发生这种
  情况时，我们将提供迁移到下一版本的说明。此特性可能需要删除、编辑和重新创建 API
  对象。编辑过程可能需要慎重操作，因为这可能会导致依赖该特性的应用程序停机。
- 推荐仅用于非关键业务用途，因为在后续版本中可能会发生不兼容的更改。如果您具有多
  个可以独立升级的，则可以放宽此限制。

{{< note >}}

<!--
Please do try *Beta* features and give feedback on them!
After they exit beta, it may not be practical for us to make more changes.
-->

请试用 _Beta_ 特性并提供相关反馈！一旦特性结束 Beta 状态，我们就不太可能再对特性
进行大幅修改。 {{< /note >}}

<!--
A *General Availability* (GA) feature is also referred to as a *stable* feature. It means:
-->

_General Availability_ (GA) 特性也称为 _稳定_ 特性，_GA_ 特性代表着：

<!--
* The feature is always enabled; you cannot disable it.
* The corresponding feature gate is no longer needed.
* Stable versions of features will appear in released software for many subsequent versions.
-->

- 此特性会一直启用；你不能禁用它。
- 不再需要相应的特性门控。
- 对于许多后续版本，特性的稳定版本将出现在发行的软件中。

<!--
## List of feature gates {#feature-gates}

Each feature gate is designed for enabling/disabling a specific feature:
-->

### 特性门控列表

每个特性门控均用于启用或禁用某个特定的特性：

<!--
- `Accelerators`: Enable Nvidia GPU support when using Docker
- `AdvancedAuditing`: Enable [advanced auditing](/docs/tasks/debug-application-cluster/audit/#advanced-audit)
- `AffinityInAnnotations`(*deprecated*): Enable setting [Pod affinity or anti-affinity](/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity).
- `AllowExtTrafficLocalEndpoints`: Enable a service to route external requests to node local endpoints.
- `APIListChunking`: Enable the API clients to retrieve (`LIST` or `GET`) resources from API server in chunks.
- `APIPriorityAndFairness`: Enable managing request concurrency with prioritization and fairness at each server. (Renamed from `RequestManagement`)
- `APIResponseCompression`: Compress the API responses for `LIST` or `GET` requests.
- `AppArmor`: Enable AppArmor based mandatory access control on Linux nodes when using Docker.
   See [AppArmor Tutorial](/docs/tutorials/clusters/apparmor/) for more details.
-->

- `Accelerators`：使用 Docker 时启用 Nvidia GPU 支持。
- `AdvancedAuditing`：启
  用[高级审查功能](/docs/tasks/debug-application-cluster/audit/#advanced-audit)。
- `AffinityInAnnotations`（ _已弃用_ ）：启用
  [Pod 亲和力或反亲和力](/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity)。
- `AllowExtTrafficLocalEndpoints`：启用服务用于将外部请求路由到节点本地终端。
- `APIListChunking`：启用 API 客户端以块的形式从 API 服务器检索（“LIST” 或
  “GET”）资源。
- `APIPriorityAndFairness`: Enable managing request concurrency with
  prioritization and fairness at each server. (Renamed from `RequestManagement`)
- `APIPriorityAndFairness`: 在每个服务器上启用优先级和公平性来管理请求并发。（由
  `RequestManagement` 重命名而来）
- `APIResponseCompression`：压缩 “LIST” 或 “GET” 请求的 API 响应。
- `AppArmor`：使用 Docker 时，在 Linux 节点上启用基于 AppArmor 机制的强制访问控
  制。请参见 [AppArmor 教程](/docs/tutorials/clusters/apparmor/) 获取详细信息。

<!--
- `AttachVolumeLimit`: Enable volume plugins to report limits on number of volumes
  that can be attached to a node.
   See [dynamic volume limits](/docs/concepts/storage/storage-limits/#dynamic-volume-limits) for more details.
- `BalanceAttachedNodeVolumes`: Include volume count on node to be considered for balanced resource allocation
  while scheduling. A node which has closer CPU, memory utilization, and volume count is favored by the scheduler
  while making decisions.
- `BlockVolume`: Enable the definition and consumption of raw block devices in Pods.
   See [Raw Block Volume Support](/docs/concepts/storage/persistent-volumes/#raw-block-volume-support)
   for more details.
- `BoundServiceAccountTokenVolume`: Migrate ServiceAccount volumes to use a projected volume consisting of a
   ServiceAccountTokenVolumeProjection.
   Check [Service Account Token Volumes](https://git.k8s.io/community/contributors/design-proposals/storage/svcacct-token-volume-source.md)
   for more details.
- `CPUManager`: Enable container level CPU affinity support, see [CPU Management Policies](/docs/tasks/administer-cluster/cpu-management-policies/).
-->

- `AttachVolumeLimit`：启用卷插件用于报告可连接到节点的卷数限制。有关更多详细信
  息，请参
  见[动态卷限制](/docs/concepts/storage/storage-limits/#dynamic-volume-limits)。
- `BalanceAttachedNodeVolumes`：包括要在调度时进行平衡资源分配的节点上的卷数
  。scheduler 在决策时会优先考虑 CPU、内存利用率和卷数更近的节点。
- `BlockVolume`：在 Pod 中启用原始块设备的定义和使用。有关更多详细信息，请参
  见[原始块卷支持](/docs/concepts/storage/persistent-volumes/#raw-block-volume-support)。
- `BoundServiceAccountTokenVolume`：迁移 ServiceAccount 卷以使用由
  ServiceAccountTokenVolumeProjection 组成的预计卷。有关更多详细信息，请参见
  [Service Account Token 卷](https://git.k8s.io/community/contributors/design-proposals/storage/svcacct-token-volume-source.md)。
- `CPUManager`：启用容器级别的 CPU 亲和力支持，有关更多详细信息，请参见
  [CPU 管理策略](/docs/tasks/administer-cluster/cpu-management-policies/)。

<!--
- `CRIContainerLogRotation`: Enable container log rotation for cri container runtime.
- `CSIBlockVolume`: Enable external CSI volume drivers to support block storage. See the [`csi` raw block volume support](/docs/concepts/storage/volumes/#csi-raw-block-volume-support) documentation for more details.
- `CSIDriverRegistry`: Enable all logic related to the CSIDriver API object in csi.storage.k8s.io.
- `CSIInlineVolume`: Enable CSI Inline volumes support for pods.
- `CSIMigration`: Enables shims and translation logic to route volume operations from in-tree plugins to corresponding pre-installed CSI plugins
- `CSIMigrationAWS`: Enables shims and translation logic to route volume operations from the AWS-EBS in-tree plugin to EBS CSI plugin. Supports falling back to in-tree EBS plugin if a node does not have EBS CSI plugin installed and configured. Requires CSIMigration feature flag enabled.
- `CSIMigrationAWSComplete`: Stops registering the EBS in-tree plugin in kubelet and volume controllers and enables shims and translation logic to route volume operations from the AWS-EBS in-tree plugin to EBS CSI plugin. Requires CSIMigration and CSIMigrationAWS feature flags enabled and EBS CSI plugin installed and configured on all nodes in the cluster.
- `CSIMigrationAzureDisk`: Enables shims and translation logic to route volume operations from the Azure-Disk in-tree plugin to AzureDisk CSI plugin. Supports falling back to in-tree AzureDisk plugin if a node does not have AzureDisk CSI plugin installed and configured. Requires CSIMigration feature flag enabled.
- `CSIMigrationAzureDiskComplete`: Stops registering the Azure-Disk in-tree plugin in kubelet and volume controllers and enables shims and translation logic to route volume operations from the Azure-Disk in-tree plugin to AzureDisk CSI plugin. Requires CSIMigration and CSIMigrationAzureDisk feature flags enabled and AzureDisk CSI plugin installed and configured on all nodes in the cluster.
- `CSIMigrationAzureFile`: Enables shims and translation logic to route volume operations from the Azure-File in-tree plugin to AzureFile CSI plugin. Supports falling back to in-tree AzureFile plugin if a node does not have AzureFile CSI plugin installed and configured. Requires CSIMigration feature flag enabled.
- `CSIMigrationAzureFileComplete`: Stops registering the Azure-File in-tree plugin in kubelet and volume controllers and enables shims and translation logic to route volume operations from the Azure-File in-tree plugin to AzureFile CSI plugin. Requires CSIMigration and CSIMigrationAzureFile feature flags  enabled and AzureFile CSI plugin installed and configured on all nodes in the cluster.
-->

- `CRIContainerLogRotation`：为 cri 容器运行时启用容器日志轮换。
- `CSIBlockVolume`：启用外部 CSI 卷驱动程序用于支持块存储。有关更多详细信息，请
  参见
  [`csi` 原始块卷支持](/docs/concepts/storage/volumes/#csi-raw-block-volume-support)。
- `CSIDriverRegistry`：在 csi.storage.k8s.io 中启用与 CSIDriver API 对象有关的所
  有逻辑。
- `CSIInlineVolume`：为 Pod 启用 CSI 内联卷支持。
- `CSIMigration`：确保填充和转换逻辑能够将卷操作从内嵌插件路由到相应的预安装 CSI
  插件。
- `CSIMigrationAWS`：确保填充和转换逻辑能够将卷操作从 AWS-EBS 内嵌插件路由到 EBS
  CSI 插件。如果节点未安装和配置 EBS CSI 插件，则支持回退到内嵌 EBS 插件。这需要
  启用 CSIMigration 特性标志。
- `CSIMigrationAWSComplete`：停止在 kubelet 和卷控制器中注册 EBS 内嵌插件，并启
  用 shims 和转换逻辑将卷操作从 AWS-EBS 内嵌插件路由到 EBS CSI 插件。这需要启用
  CSIMigration 和 CSIMigrationAWS 特性标志，并在群集中的所有节点上安装和配置 EBS
  CSI 插件。
- `CSIMigrationAzureDisk`：确保填充和转换逻辑能够将卷操作从 Azure 磁盘内嵌插件路
  由到 Azure 磁盘 CSI 插件。如果节点未安装和配置 AzureDisk CSI 插件，支持回退到
  内建 AzureDisk 插件。这需要启用 CSIMigration 特性标志。
- `CSIMigrationAzureDiskComplete`：停止在 kubelet 和卷控制器中注册 Azure 磁盘内
  嵌插件，并启用 shims 和转换逻辑以将卷操作从 Azure 磁盘内嵌插件路由到 AzureDisk
  CSI 插件。这需要启用 CSIMigration 和 CSIMigrationAzureDisk 特性标志，并在群集
  中的所有节点上安装和配置 AzureDisk CSI 插件。
- `CSIMigrationAzureFile`：确保填充和转换逻辑能够将卷操作从 Azure 文件内嵌插件路
  由到 Azure 文件 CSI 插件。如果节点未安装和配置 AzureFile CSI 插件，支持回退到
  内嵌 AzureFile 插件。这需要启用 CSIMigration 特性标志。
- `CSIMigrationAzureFileComplete`：停止在 kubelet 和卷控制器中注册 Azure-File 内
  嵌插件，并启用 shims 和转换逻辑以将卷操作从 Azure-File 内嵌插件路由到
  AzureFile CSI 插件。这需要启用 CSIMigration 和 CSIMigrationAzureFile 特性标志
  ，并在群集中的所有节点上安装和配置 AzureFile CSI 插件。

<!--
- `CSIMigrationGCE`: Enables shims and translation logic to route volume operations from the GCE-PD in-tree plugin to PD CSI plugin. Supports falling back to in-tree GCE plugin if a node does not have PD CSI plugin installed and configured. Requires CSIMigration feature flag enabled.
- `CSIMigrationGCEComplete`: Stops registering the GCE-PD in-tree plugin in kubelet and volume controllers and enables shims and translation logic to route volume operations from the GCE-PD in-tree plugin to PD CSI plugin. Requires CSIMigration and CSIMigrationGCE feature flags enabled and PD CSI plugin installed and configured on all nodes in the cluster.
- `CSIMigrationOpenStack`: Enables shims and translation logic to route volume operations from the Cinder in-tree plugin to Cinder CSI plugin. Supports falling back to in-tree Cinder plugin if a node does not have Cinder CSI plugin installed and configured. Requires CSIMigration feature flag enabled.
- `CSIMigrationOpenStackComplete`: Stops registering the Cinder in-tree plugin in kubelet and volume controllers and enables shims and translation logic to route volume operations from the Cinder in-tree plugin to Cinder CSI plugin. Requires CSIMigration and CSIMigrationOpenStack feature flags enabled and Cinder CSI plugin installed and configured on all nodes in the cluster.
- `CSINodeInfo`: Enable all logic related to the CSINodeInfo API object in csi.storage.k8s.io.
- `CSIPersistentVolume`: Enable discovering and mounting volumes provisioned through a
  [CSI (Container Storage Interface)](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/storage/container-storage-interface.md)
  compatible volume plugin.
  Check the [`csi` volume type](/docs/concepts/storage/volumes/#csi) documentation for more details.
-->

- `CSIMigrationGCE`：使 shims 和转换逻辑能够将卷操作从 GCE-PD 内嵌插件路由到 PD
  CSI 插件。如果节点未安装和配置 PD CSI 插件，支持回退到内嵌 GCE 插件。这需要启
  用 CSIMigration 特性标志。
- `CSIMigrationGCEComplete`：停止在 kubelet 和卷控制器中注册 GCE-PD 内嵌插件，并
  启用 shims 和转换逻辑以将卷操作从 GCE-PD 内嵌插件路由到 PD CSI 插件。这需要启
  用 CSIMigration 和 CSIMigrationGCE 特性标志，并在群集中的所有节点上安装和配置
  PD CSI 插件。
- `CSIMigrationOpenStack`：确保填充和转换逻辑能够将卷操作从 Cinder 内嵌插件路由
  到 Cinder CSI 插件。如果节点未安装和配置 Cinder CSI 插件，支持回退到内嵌
  Cinder 插件。这需要启用 CSIMigration 特性标志。
- `CSIMigrationOpenStackComplete`：停止在 kubelet 和卷控制器中注册 Cinder 内嵌插
  件，并启用 shims 和转换逻辑将卷操作从 Cinder 内嵌插件路由到 Cinder CSI 插件。
  这需要启用 CSIMigration 和 CSIMigrationOpenStack 特性标志，并在群集中的所有节
  点上安装和配置 Cinder CSI 插件。
- `CSINodeInfo`：在 csi.storage.k8s.io 中启用与 CSINodeInfo API 对象有关的所有逻
  辑。
- `CSIPersistentVolume`：启用发现并挂载通过
  [CSI（容器存储接口）](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/storage/container-storage-interface.md)兼
  容卷插件配置的卷。有关更多详细信息，请参见
  [`csi` 卷类型](/docs/concepts/storage/volumes/#csi)。

<!--
- `CustomCPUCFSQuotaPeriod`: Enable nodes to change CPUCFSQuotaPeriod.
- `CustomPodDNS`: Enable customizing the DNS settings for a Pod using its `dnsConfig` property.
   Check [Pod's DNS Config](/docs/concepts/services-networking/dns-pod-service/#pods-dns-config)
   for more details.
- `CustomResourceDefaulting`: Enable CRD support for default values in OpenAPI v3 validation schemas.
- `CustomResourcePublishOpenAPI`: Enables publishing of CRD OpenAPI specs.
- `CustomResourceSubresources`: Enable `/status` and `/scale` subresources
  on resources created from [CustomResourceDefinition](/docs/concepts/api-extension/custom-resources/).
- `CustomResourceValidation`: Enable schema based validation on resources created from
  [CustomResourceDefinition](/docs/concepts/api-extension/custom-resources/).
- `CustomResourceWebhookConversion`: Enable webhook-based conversion
  on resources created from [CustomResourceDefinition](/docs/concepts/api-extension/custom-resources/).
  troubleshoot a running Pod.
-->

- `CustomCPUCFSQuotaPeriod`：使节点能够更改 CPUCFSQuotaPeriod。
- `CustomPodDNS`：使用其 `dnsConfig` 属性启用 Pod 的自定义 DNS 设置。有关更多详
  细信息，请参见
  [Pod 的 DNS 配置](/docs/concepts/services-networking/dns-pod-service/#pods-dns-config)。
- `CustomResourceDefaulting`：为 OpenAPI v3 验证架构中的默认值启用 CRD 支持。
- `CustomResourcePublishOpenAPI`：启用 CRD OpenAPI 规范的发布。
- `CustomResourceSubresources`：对于从
  [CustomResourceDefinition](/docs/concepts/api-extension/custom-resources/) 中
  创建的资源启用 `/status` 和 `/scale` 子资源。
- `CustomResourceValidation`：对于从
  [CustomResourceDefinition](/docs/concepts/api-extension/custom-resources/) 中
  创建的资源启用基于架构的验证。
- `CustomResourceWebhookConversion`：对于从
  [CustomResourceDefinition](/docs/concepts/api-extension/custom-resources/) 中
  创建的资源启用基于 Webhook 的转换。对正在运行的 Pod 进行故障排除。

<!--
- `DevicePlugins`: Enable the [device-plugins](/docs/concepts/cluster-administration/device-plugins/)
  based resource provisioning on nodes.
- `DryRun`: Enable server-side [dry run](/docs/reference/using-api/api-concepts/#dry-run) requests
  so that validation, merging, and mutation can be tested without committing.
- `DynamicAuditing`: Enable [dynamic auditing](/docs/tasks/debug-application-cluster/audit/#dynamic-backend)
- `DynamicKubeletConfig`: Enable the dynamic configuration of kubelet. See [Reconfigure kubelet](/docs/tasks/administer-cluster/reconfigure-kubelet/).
- `DynamicProvisioningScheduling`: Extend the default scheduler to be aware of volume topology and handle PV provisioning.
  This feature is superseded by the `VolumeScheduling` feature completely in v1.12.
- `DynamicVolumeProvisioning`(*deprecated*): Enable the [dynamic provisioning](/docs/concepts/storage/dynamic-provisioning/) of persistent volumes to Pods.
-->

- `DevicePlugins`：在节点上启用基于
  [device-plugins](/docs/concepts/cluster-administration/device-plugins/) 的资源
  供应。
- `DryRun`：启用服务器端
  [dry run](/docs/reference/using-api/api-concepts/#dry-run) 请求，以便无需提交
  即可测试验证、合并和差异化。
- `DynamicAuditing`：确
  保[动态审查](/docs/tasks/debug-application-cluster/audit/#dynamic-backend)。
- `DynamicKubeletConfig`：启用 kubelet 的动态配置。请参
  阅[重新配置 kubelet](/docs/tasks/administer-cluster/reconfigure-kubelet/)。
- `DynamicProvisioningScheduling`：扩展默认 scheduler 以了解卷拓扑并处理 PV 配置
  。此特性已在 v1.12 中完全被 `VolumeScheduling` 特性取代。
- `DynamicVolumeProvisioning`（ _已弃用_ ）：启用持久化卷到 Pod
  的[动态预配置](/docs/concepts/storage/dynamic-provisioning/)。

<!--
- `EnableAggregatedDiscoveryTimeout` (*deprecated*): Enable the five second timeout on aggregated discovery calls.
- `EnableEquivalenceClassCache`: Enable the scheduler to cache equivalence of nodes when scheduling Pods.
- `EphemeralContainers`: Enable the ability to add {{< glossary_tooltip text="ephemeral containers"
  term_id="ephemeral-container" >}} to running pods.
- `EvenPodsSpread`: Enable pods to be scheduled evenly across topology domains. See [Pod Topology Spread Constraints](/docs/concepts/workloads/pods/pod-topology-spread-constraints/).
- `ExpandInUsePersistentVolumes`: Enable expanding in-use PVCs. See [Resizing an in-use PersistentVolumeClaim](/docs/concepts/storage/persistent-volumes/#resizing-an-in-use-persistentvolumeclaim).
- `ExpandPersistentVolumes`: Enable the expanding of persistent volumes. See [Expanding Persistent Volumes Claims](/docs/concepts/storage/persistent-volumes/#expanding-persistent-volumes-claims).
- `ExperimentalCriticalPodAnnotation`: Enable annotating specific pods as *critical* so that their [scheduling is guaranteed](/docs/tasks/administer-cluster/guaranteed-scheduling-critical-addon-pods/).
  This feature is deprecated by Pod Priority and Preemption as of v1.13.
-->

- `EnableAggregatedDiscoveryTimeout` （ _已弃用_ ）：对聚集的发现调用启用五秒钟
  超时设置。
- `EnableEquivalenceClassCache`：调度 Pod 时，使 scheduler 缓存节点的等效项。
- `EphemeralContainers`：启用添�
  {{< glossary_tooltip text="临时容器" term_id="ephemeral-container" >}} 到正在
  运行的 Pod 的特性。
- `EvenPodsSpread`：使 Pod 能够在拓扑域之间平衡调度。请参阅
  [Pod 拓扑扩展约束](/docs/concepts/workloads/pods/pod-topology-spread-constraints/)。
- `ExpandInUsePersistentVolumes`：启用扩展使用中的 PVC。请查阅
  [调整使用中的 PersistentVolumeClaim 的大小](/docs/concepts/storage/persistent-volumes/#resizing-an-in-use-persistentvolumeclaim)。
- `ExpandPersistentVolumes`：启用持久卷的扩展。请查
  阅[扩展永久卷声明](/docs/concepts/storage/persistent-volumes/#expanding-persistent-volumes-claims)。
- `ExperimentalCriticalPodAnnotation`：启用将特定 Pod 注解为 _critical_ 的方式，
  用
  于[确保其调度](/docs/tasks/administer-cluster/guaranteed-scheduling-critical-addon-pods/)。
  从 v1.13 开始，Pod 优先级和抢占功能已弃用此特性。

<!--
- `ExperimentalHostUserNamespaceDefaultingGate`: Enabling the defaulting user
   namespace to host. This is for containers that are using other host namespaces,
   host mounts, or containers that are privileged or using specific non-namespaced
   capabilities (e.g. `MKNODE`, `SYS_MODULE` etc.). This should only be enabled
   if user namespace remapping is enabled in the Docker daemon.
- `EndpointSlice`: Enables Endpoint Slices for more scalable and extensible
   network endpoints. Requires corresponding API and Controller to be enabled.
   See [Enabling Endpoint Slices](/docs/tasks/administer-cluster/enabling-endpointslices/).
- `GCERegionalPersistentDisk`: Enable the regional PD feature on GCE.
- `HugePages`: Enable the allocation and consumption of pre-allocated [huge pages](/docs/tasks/manage-hugepages/scheduling-hugepages/).
-->

- `ExperimentalHostUserNamespaceDefaultingGate`：启用主机默认的用户命名空间。这
  适用于使用其他主机命名空间、主机安装的容器，或具有特权或使用特定的非命名空间功
  能（例如 MKNODE、SYS_MODULE 等）的容器。如果在 Docker 守护程序中启用了用户命名
  空间重新映射，则启用此选项。
- `EndpointSlice`：启用端点切片以实现更多可扩展的网络端点。需要启用相应的 API 和
  控制器，请参
  阅[启用端点切片](/docs/tasks/administer-cluster/enabling-endpointslices/)。
- `GCERegionalPersistentDisk`：在 GCE 上启用区域 PD 特性。
- `HugePages`: 启用分配和使用预分配的
  [huge pages](/docs/tasks/manage-hugepages/scheduling-hugepages/)。

<!--
- `HyperVContainer`: Enable [Hyper-V isolation](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/hyperv-container) for Windows containers.
- `HPAScaleToZero`: Enables setting `minReplicas` to 0 for `HorizontalPodAutoscaler` resources when using custom or external metrics.
- `KubeletConfigFile`: Enable loading kubelet configuration from a file specified using a config file.
  See [setting kubelet parameters via a config file](/docs/tasks/administer-cluster/kubelet-config-file/) for more details.
- `KubeletPluginsWatcher`: Enable probe-based plugin watcher utility to enable kubelet
  to discover plugins such as [CSI volume drivers](/docs/concepts/storage/volumes/#csi).
- `KubeletPodResources`: Enable the kubelet's pod resources grpc endpoint.
   See [Support Device Monitoring](https://git.k8s.io/community/keps/sig-node/compute-device-assignment.md) for more details.
- `LegacyNodeRoleBehavior`: When disabled, legacy behavior in service load balancers and node disruption will ignore the `node-role.kubernetes.io/master` label in favor of the feature-specific labels.
-->

- `HyperVContainer`：为 Windows 容器启
  用[Hyper-V 隔离](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/hyperv-container)。
- `HPAScaleToZero`：使用自定义指标或外部指标时，可将 `HorizontalPodAutoscaler`
  资源的 `minReplicas` 设置为 0。
- `KubeletConfigFile`：启用从使用配置文件指定的文件中加载 kubelet 配置。有关更多
  详细信息，请参
  见[通过配置文件设置 kubelet 参数](/docs/tasks/administer-cluster/kubelet-config-file/)。
- `KubeletPluginsWatcher`：启用基于探针的插件监视应用程序，使 kubelet 能够发现插
  件，例如 [CSI 卷驱动程序](/docs/concepts/storage/volumes/#csi)。
- `KubeletPodResources`：启用 kubelet 的 pod 资源 grpc 端点。有关更多详细信息，
  请参
  见[支持设备监控](https://git.k8s.io/community/keps/sig-node/compute-device-assignment.md)。
- `LegacyNodeRoleBehavior`：禁用此选项后，服务负载均衡中的旧版操作和节点中断将忽
  略 `node-role.kubernetes.io/master` 标签，而使用特性指定的标签。

<!--
- `LocalStorageCapacityIsolation`: Enable the consumption of [local ephemeral storage](/docs/concepts/configuration/manage-compute-resources-container/) and also the `sizeLimit` property of an [emptyDir volume](/docs/concepts/storage/volumes/#emptydir).
- `LocalStorageCapacityIsolationFSQuotaMonitoring`: When `LocalStorageCapacityIsolation` is enabled for [local ephemeral storage](/docs/concepts/configuration/manage-compute-resources-container/) and the backing filesystem for [emptyDir volumes](/docs/concepts/storage/volumes/#emptydir) supports project quotas and they are enabled, use project quotas to monitor [emptyDir volume](/docs/concepts/storage/volumes/#emptydir) storage consumption rather than filesystem walk for better performance and accuracy.
- `MountContainers`: Enable using utility containers on host as the volume mounter.
- `MountPropagation`: Enable sharing volume mounted by one container to other containers or pods.
  For more details, please see [mount propagation](/docs/concepts/storage/volumes/#mount-propagation).
- `NodeDisruptionExclusion`: Enable use of the node label `node.kubernetes.io/exclude-disruption` which prevents nodes from being evacuated during zone failures.
-->

- `LocalStorageCapacityIsolation`：启
  用[本地临时存储](/docs/concepts/configuration/manage-compute-resources-container/)的
  消耗，以及 [emptyDir 卷](/docs/concepts/storage/volumes/#emptydir) 的
  `sizeLimit` 属性。
- `LocalStorageCapacityIsolationFSQuotaMonitoring`：如果
  为[本地临时存储](/docs/concepts/configuration/manage-compute-resources-container/)启
  用了 `LocalStorageCapacityIsolation`，并且
  [emptyDir 卷](/docs/concepts/storage/volumes/#emptydir) 的后备文件系统支持项目
  配额，并且启用了这些配额，请使用项目配额来监视
  [emptyDir 卷](/docs/concepts/storage/volumes/#emptydir)的存储消耗而不是遍历文
  件系统，以此获得更好的性能和准确性。
- `MountContainers`：在主机上启用将应用程序容器用作卷安装程序。
- `MountPropagation`：启用将一个容器安装的共享卷共享到其他容器或 Pod。有关更多详
  细信息，请参见
  [mount propagation](/docs/concepts/storage/volumes/#mount-propagation)。
- `NodeDisruptionExclusion`：启用节点标签
  `node.kubernetes.io/exclude-disruption`，以防止在区域故障期间驱逐节点。

<!--
- `NodeLease`: Enable the new Lease API to report node heartbeats, which could be used as a node health signal.
- `NonPreemptingPriority`: Enable NonPreempting option for PriorityClass and Pod.
- `PersistentLocalVolumes`: Enable the usage of `local` volume type in Pods.
  Pod affinity has to be specified if requesting a `local` volume.
- `PodOverhead`: Enable the [PodOverhead](/docs/concepts/configuration/pod-overhead/) feature to account for pod overheads.
- `PodPriority`: Enable the descheduling and preemption of Pods based on their [priorities](/docs/concepts/configuration/pod-priority-preemption/).
- `PodReadinessGates`: Enable the setting of `PodReadinessGate` field for extending
  Pod readiness evaluation.  See [Pod readiness gate](/docs/concepts/workloads/pods/pod-lifecycle/#pod-readiness-gate)
  for more details.
-->

- `NodeLease`：启用新的租赁 API 以报告节点心跳，可用作节点运行状况信号。
- `NonPreemptingPriority`：为 PriorityClass 和 Pod 启用 NonPreempting 选项。
- `PersistentLocalVolumes`：在 Pod 中启用 “本地” 卷类型的使用。如果请求 “本地”
  卷，则必须指定 Pod 亲和力。
- `PodOverhead`：启用 [PodOverhead](/docs/concepts/configuration/pod-overhead/)
  特性以解决 Pod 开销。
- `PodPriority`：根
  据[优先级](/docs/concepts/configuration/pod-priority-preemption/)启用 Pod 的调
  度和抢占。
- `PodReadinessGates`：启用 `PodReadinessGate` 字段的设置以扩展 Pod 准备状态评估
  。有关更多详细信息，请参见
  [Pod readiness 特性门控](/docs/concepts/workloads/pods/pod-lifecycle/#pod-readiness-gate)。

<!--
- `PodShareProcessNamespace`: Enable the setting of `shareProcessNamespace` in a Pod for sharing
  a single process namespace between containers running in a pod.  More details can be found in
  [Share Process Namespace between Containers in a Pod](/docs/tasks/configure-pod-container/share-process-namespace/).
- `ProcMountType`: Enables control over ProcMountType for containers.
- `PVCProtection`: Enable the prevention of a PersistentVolumeClaim (PVC) from
  being deleted when it is still used by any Pod.
- `QOSReserved`: Allows resource reservations at the QoS level preventing pods at lower QoS levels from
  bursting into resources requested at higher QoS levels (memory only for now).
- `ResourceLimitsPriorityFunction`: Enable a scheduler priority function that
  assigns a lowest possible score of 1 to a node that satisfies at least one of
  the input Pod's cpu and memory limits. The intent is to break ties between
  nodes with same scores.
-->

- `PodShareProcessNamespace`：在 Pod 中启用 `shareProcessNamespace` 的设置，以便
  在 Pod 中运行的容器之间共享单个进程命名空间。更多详细信息，请参
  见[在 Pod 中的容器之间共享进程命名空间](/docs/tasks/configure-pod-container/share-process-namespace/)。
- `ProcMountType`：启用对容器的 ProcMountType 的控制。
- `PVCProtection`：启用防止任何 Pod 仍使用 PersistentVolumeClaim(PVC) 删除的特性
  。可以
  在[此处](/docs/tasks/administer-cluster/storage-object-in-use-protection/)中找
  到更多详细信息。
- `QOSReserved`：允许在 QoS 级别进行资源预留，以防止处于较低 QoS 级别的 Pod 突发
  进入处于较高 QoS 级别的请求资源（仅适用于内存）。
- `ResourceLimitsPriorityFunction`：启用 scheduler 优先级特性，该特性将最低可能
  得 1 分配给至少满足输入 Pod 的 cpu 和内存限制之一的节点，目的是打破得分相同的
  节点之间的联系。

<!--
- `RequestManagement`: Enable managing request concurrency with prioritization and fairness at each server.
- `ResourceQuotaScopeSelectors`: Enable resource quota scope selectors.
- `RotateKubeletClientCertificate`: Enable the rotation of the client TLS certificate on the kubelet.
  See [kubelet configuration](/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/#kubelet-configuration) for more details.
- `RotateKubeletServerCertificate`: Enable the rotation of the server TLS certificate on the kubelet.
  See [kubelet configuration](/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/#kubelet-configuration) for more details.
- `RunAsGroup`: Enable control over the primary group ID set on the init processes of containers.
- `RuntimeClass`: Enable the [RuntimeClass](/docs/concepts/containers/runtime-class/) feature for selecting container runtime configurations.
- `ScheduleDaemonSetPods`: Enable DaemonSet Pods to be scheduled by the default scheduler instead of the DaemonSet controller.
-->

- `RequestManagement`：在每个服务器上启用具有优先级和公平性的管理请求并发性。
- `ResourceQuotaScopeSelectors`：启用资源配额范围选择器。
- `RotateKubeletClientCertificate`：在 kubelet 上启用客户端 TLS 证书的轮换。有关
  更多详细信息，请参见
  [kubelet 配置](/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/#kubelet-configuration)。
- `RotateKubeletServerCertificate`：在 kubelet 上启用服务器 TLS 证书的轮换。有关
  更多详细信息，请参见
  [kubelet 配置](/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/#kubelet-configuration)。
- `RunAsGroup`：启用对容器初始化过程中设置的主要组 ID 的控制。
- `RuntimeClass`：启用 [RuntimeClass](/docs/concepts/containers/runtime-class/)
  特性用于选择容器运行时配置。
- `ScheduleDaemonSetPods`：启用 DaemonSet Pods 由默认调度程序而不是 DaemonSet 控
  制器进行调度。

<!--
- `SCTPSupport`: Enables the usage of SCTP as `protocol` value in `Service`, `Endpoint`, `NetworkPolicy` and `Pod` definitions
- `ServerSideApply`: Enables the [Sever Side Apply (SSA)](/docs/reference/using-api/api-concepts/#server-side-apply) path at the API Server.
- `ServiceLoadBalancerFinalizer`: Enable finalizer protection for Service load balancers.
- `ServiceNodeExclusion`: Enable the exclusion of nodes from load balancers created by a cloud provider.
  A node is eligible for exclusion if labelled with "`alpha.service-controller.kubernetes.io/exclude-balancer`" key or `node.kubernetes.io/exclude-from-external-load-balancers`.
- `ServiceTopology`: Enable service to route traffic based upon the Node topology of the cluster. See [ServiceTopology](https://kubernetes.io/docs/concepts/services-networking/service-topology/) for more details.
- `StartupProbe`: Enable the [startup](/docs/concepts/workloads/pods/pod-lifecycle/#when-should-you-use-a-startup-probe) probe in the kubelet.
- `StorageObjectInUseProtection`: Postpone the deletion of PersistentVolume or
  PersistentVolumeClaim objects if they are still being used.
-->

- `SCTPSupport`：在 “服务”、“端点”、“NetworkPolicy” 和 “Pod” 定义中，将 SCTP 用
  作 “协议” 值。
- `ServerSideApply`：在 API 服务器上启
  用[服务器端应用（SSA）](/docs/reference/using-api/api-concepts/#server-side-apply)
  路径。
- `ServiceLoadBalancerFinalizer`：为服务负载均衡启用终结器保护。
- `ServiceNodeExclusion`：启用从云提供商创建的负载均衡中排除节点。如果节点标记有
  `alpha.service-controller.kubernetes.io/exclude-balancer` 键或
  `node.kubernetes.io/exclude-from-external-load-balancers`，则可以排除节点。
- `ServiceTopology`: 启用服务拓扑可以让一个服务基于集群的节点拓扑进行流量路由。
  有关更多详细信息，请参
  见[Service 拓扑](https://kubernetes.io/zh/docs/concepts/services-networking/service-topology/)
- `StartupProbe`：在 kubelet 中启用
  [startup](/docs/concepts/workloads/pods/pod-lifecycle/#when-should-you-use-a-startup-probe)
  探针。
- `StorageObjectInUseProtection`：如果仍在使用 PersistentVolume 或
  PersistentVolumeClaim 对象，则将其推迟。

<!--
- `StorageVersionHash`: Allow apiservers to expose the storage version hash in the discovery.
- `StreamingProxyRedirects`: Instructs the API server to intercept (and follow)
   redirects from the backend (kubelet) for streaming requests.
  Examples of streaming requests include the `exec`, `attach` and `port-forward` requests.
- `SupportIPVSProxyMode`: Enable providing in-cluster service load balancing using IPVS.
  See [service proxies](/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies) for more details.
- `SupportPodPidsLimit`: Enable the support to limiting PIDs in Pods.
- `Sysctls`: Enable support for namespaced kernel parameters (sysctls) that can be set for each pod.
  See [sysctls](/docs/tasks/administer-cluster/sysctl-cluster/) for more details.
-->

- `StorageVersionHash`：允许 apiserver 在发现中公开存储版本的哈希值。
- `StreamingProxyRedirects`：指示 API 服务器拦截（并遵循）从后端（kubelet）进行
  重定向以处理流请求。流请求的例子包括 `exec`、`attach` 和 `port-forward` 请求。
- `SupportIPVSProxyMode`：启用使用 IPVS 提供内服务负载平衡。有关更多详细信息，请
  参
  见[服务代理](/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies)。
- `SupportPodPidsLimit`：启用支持限制 Pod 中的进程 PID。
- `Sysctls`：启用对可以为每个 Pod 设置的命名空间内核参数（sysctls）的支持。有关
  更多详细信息，请参见
  [sysctls](/docs/tasks/administer-cluster/sysctl-cluster/)。

<!--
- `TaintBasedEvictions`: Enable evicting pods from nodes based on taints on nodes and tolerations on Pods.
  See [taints and tolerations](/docs/concepts/configuration/taint-and-toleration/) for more details.
- `TaintNodesByCondition`: Enable automatic tainting nodes based on [node conditions](/docs/concepts/architecture/nodes/#condition).
- `TokenRequest`: Enable the `TokenRequest` endpoint on service account resources.
- `TokenRequestProjection`: Enable the injection of service account tokens into
  a Pod through the [`projected` volume](/docs/concepts/storage/volumes/#projected).
- `TopologyManager`: Enable a mechanism to coordinate fine-grained hardware resource assignments for different components in Kubernetes. See [Control Topology Management Policies on a node](/docs/tasks/administer-cluster/topology-manager/).
- `TTLAfterFinished`: Allow a [TTL controller](/docs/concepts/workloads/controllers/ttlafterfinished/) to clean up resources after they finish execution.
-->

- `TaintBasedEvictions`：根据节点上的污点和 Pod 上的容忍度启用从节点驱逐 Pod 的
  特性。有关更多详细信息，请参
  见[污点和容忍度](/docs/concepts/configuration/taint-and-toleration/)。
- `TaintNodesByCondition`：根
  据[节点条件](/docs/concepts/configuration/taint-and-toleration/)启用自动在节点
  标记污点。
- `TokenRequest`：在服务帐户资源上启用 `TokenRequest` 端点。
- `TokenRequestProjection`：启用通过
  [`projected` 卷](/docs/concepts/storage/volumes/#projected) 将服务帐户令牌注入
  到 Pod 中的特性。
- `TopologyManager`：启用一种机制来协调 Kubernetes 不同组件的细粒度硬件资源分配
  。详见
  [控制节点上的拓扑管理策略](/docs/tasks/administer-cluster/topology-manager/)。
- `TTLAfterFinished`：完成执行后，允许
  [TTL 控制器](/docs/concepts/workloads/controllers/ttlafterfinished/)清理资源。

<!--
- `VolumePVCDataSource`: Enable support for specifying an existing PVC as a DataSource.
- `VolumeScheduling`: Enable volume topology aware scheduling and make the
  PersistentVolumeClaim (PVC) binding aware of scheduling decisions. It also
  enables the usage of [`local`](/docs/concepts/storage/volumes/#local) volume
  type when used together with the `PersistentLocalVolumes` feature gate.
- `VolumeSnapshotDataSource`: Enable volume snapshot data source support.
-->

- `VolumePVCDataSource`：启用对将现有 PVC 指定数据源的支持。
- `VolumeScheduling`：启用卷拓扑感知调度，并使 PersistentVolumeClaim（PVC）绑定
  调度决策；当与 PersistentLocalVolumes 特性门控一起使用时，还可以使用
  `PersistentLocalVolumes` 卷类型。
- `VolumeSnapshotDataSource`：启用卷快照数据源支持。

<!--
- `VolumeSubpathEnvExpansion`: Enable `subPathExpr` field for expanding environment variables into a `subPath`.
- `WatchBookmark`: Enable support for watch bookmark events.
- `WindowsGMSA`: Enables passing of GMSA credential specs from pods to container runtimes.
- `WinDSR`: Allows kube-proxy to create DSR loadbalancers for Windows.
- `WinOverlay`: Allows kube-proxy to run in overlay mode for Windows.
-->

- `VolumeSubpathEnvExpansion`：启用 `subPathExpr` 字段用于将环境变量扩展为
  `subPath`。
- `WatchBookmark`：启用对监测 bookmark 事件的支持。
- `WindowsGMSA`：允许将 GMSA 凭据规范从 Pod 传递到容器运行时。
- `WinDSR`：允许 kube-proxy 为 Windows 创建 DSR 负载均衡。
- `WinOverlay`：允许 kube-proxy 在 Windows 的 overlay 模式下运行。

{{% /capture %}} {{% capture whatsnext %}}

<!--
* The [deprecation policy](/docs/reference/using-api/deprecation-policy/) for Kubernetes explains
  the project's approach to removing features and components.
-->

- Kubernetes 的 [弃用策略](/docs/reference/using-api/deprecation-policy/) 介绍了
  项目已移除的特性部件和组件的方法。

{{% /capture %}}
