---
title: 对 kubeadm 进行故障排查
content_template: templates/concept
weight: 20
---

## <!--

title: Troubleshooting kubeadm content_template: templates/concept weight: 20

---

--> {{% capture overview %}}

<!--
As with any program, you might run into an error installing or running kubeadm.
This page lists some common failure scenarios and have provided steps that can help you understand and fix the problem.

If your problem is not listed below, please follow the following steps:

- If you think your problem is a bug with kubeadm:
  - Go to [github.com/kubernetes/kubeadm](https://github.com/kubernetes/kubeadm/issues) and search for existing issues.
  - If no issue exists, please [open one](https://github.com/kubernetes/kubeadm/issues/new) and follow the issue template.

- If you are unsure about how kubeadm works, you can ask on [Slack](http://slack.k8s.io/) in #kubeadm, or open a question on [StackOverflow](https://stackoverflow.com/questions/tagged/kubernetes). Please include
  relevant tags like `#kubernetes` and `#kubeadm` so folks can help you.
-->

与任何程序一样，您可能会在安装或者运行 kubeadm 时遇到错误。本文列举了一些常见的
故障场景，并提供可帮助您理解和解决这些问题的步骤。

如果您的问题未在下面列出，请执行以下步骤：

- 如果您认为问题是 kubeadm 的错误：

  - 转到
    [github.com/kubernetes/kubeadm](https://github.com/kubernetes/kubeadm/issues)
    并搜索存在的问题。
  - 如果没有问题，请 [打开](https://github.com/kubernetes/kubeadm/issues/new) 并
    遵循问题模板。

- 如果您对 kubeadm 的工作方式有疑问，可以在 [Slack](http://slack.k8s.io/) 上的
  #kubeadm 频道提问，或者在
  [StackOverflow](https://stackoverflow.com/questions/tagged/kubernetes) 上提问
  。请加入相关标签，例如 `#kubernetes` 和 `#kubeadm`，这样其他人可以帮助您。

{{% /capture %}}

{{% capture body %}}

<!--
## `ebtables` or some similar executable not found during installation

If you see the following warnings while running `kubeadm init`

```sh
[preflight] WARNING: ebtables not found in system path
[preflight] WARNING: ethtool not found in system path
```

Then you may be missing `ebtables`, `ethtool` or a similar executable on your node. You can install them with the following commands:

- For Ubuntu/Debian users, run `apt install ebtables ethtool`.
- For CentOS/Fedora users, run `yum install ebtables ethtool`.
-->

## 在安装过程中没有找到 `ebtables` 或者其他类似的可执行文件

如果在运行 `kubeadm init` 命令时，遇到以下的警告

```sh
[preflight] WARNING: ebtables not found in system path
[preflight] WARNING: ethtool not found in system path
```

那么或许在您的节点上缺失 `ebtables`、`ethtool` 或者类似的可执行文件。您可以使用
以下命令安装它们：

- 对于 Ubuntu/Debian 用户，运行 `apt install ebtables ethtool` 命令。
- 对于 CentOS/Fedora 用户，运行 `yum install ebtables ethtool` 命令。

<!--
## kubeadm blocks waiting for control plane during installation

If you notice that `kubeadm init` hangs after printing out the following line:

```sh
[apiclient] Created API client, waiting for the control plane to become ready
```
-->

## 在安装过程中，kubeadm 一直等待控制平面就绪

如果您注意到 `kubeadm init` 在打印以下行后挂起：

```sh
[apiclient] Created API client, waiting for the control plane to become ready
```

<!--
This may be caused by a number of problems. The most common are:

- network connection problems. Check that your machine has full network connectivity before continuing.
- the default cgroup driver configuration for the kubelet differs from that used by Docker.
  Check the system log file (e.g. `/var/log/message`) or examine the output from `journalctl -u kubelet`. If you see something like the following:

  ```shell
  error: failed to run Kubelet: failed to create kubelet:
  misconfiguration: kubelet cgroup driver: "systemd" is different from docker cgroup driver: "cgroupfs"
  ```

  There are two common ways to fix the cgroup driver problem:

 1. Install Docker again following instructions
  [here](/docs/setup/production-environment/container-runtimes/#docker).

 1. Change the kubelet config to match the Docker cgroup driver manually, you can refer to
    [Configure cgroup driver used by kubelet on Master Node](/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#configure-cgroup-driver-used-by-kubelet-on-master-node)

- control plane Docker containers are crashlooping or hanging. You can check this by running `docker ps` and investigating each container by running `docker logs`.
-->

这可能是由许多问题引起的。最常见的是：

- 网络连接问题。在继续之前，请检查您的计算机是否具有全部联通的网络连接。
- kubelet 的默认 cgroup 驱动程序配置不同于 Docker 使用的配置。检查系统日志文件 (
  例如 `/var/log/message`) 或检查 `journalctl -u kubelet` 的输出。 如果您看见以
  下内容：

  ```shell
  error: failed to run Kubelet: failed to create kubelet:
  misconfiguration: kubelet cgroup driver: "systemd" is different from docker cgroup driver: "cgroupfs"
  ```

  有两种常见方法可解决 cgroup 驱动程序问题：

1.  按照 [此处](/docs/setup/production-environment/container-runtimes/#docker)
    的说明再次安装 Docker。

1.  更改 kubelet 配置以手动匹配 Docker cgroup 驱动程序，您可以参考
    [在主节点上配置 kubelet 要使用的 cgroup 驱动程序](/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#configure-cgroup-driver-used-by-kubelet-on-master-node)

- 控制平面上的 Docker 容器持续进入崩溃状态或（因其他原因）挂起。您可以运行
  `docker ps` 命令来检查以及 `docker logs` 命令来检视每个容器的运行日志。

<!--
## kubeadm blocks when removing managed containers

The following could happen if Docker halts and does not remove any Kubernetes-managed containers:

```bash
sudo kubeadm reset
[preflight] Running pre-flight checks
[reset] Stopping the kubelet service
[reset] Unmounting mounted directories in "/var/lib/kubelet"
[reset] Removing kubernetes-managed containers
(block)
```

A possible solution is to restart the Docker service and then re-run `kubeadm reset`:

```bash
sudo systemctl restart docker.service
sudo kubeadm reset
```

Inspecting the logs for docker may also be useful:

```sh
journalctl -ul docker
```
-->

## 当删除托管容器时 kubeadm 阻塞

如果 Docker 停止并且不删除 Kubernetes 所管理的所有容器，可能发生以下情况：

```bash
sudo kubeadm reset
[preflight] Running pre-flight checks
[reset] Stopping the kubelet service
[reset] Unmounting mounted directories in "/var/lib/kubelet"
[reset] Removing kubernetes-managed containers
(block)
```

一个可行的解决方案是重新启动 Docker 服务，然后重新运行 `kubeadm reset`：

```bash
sudo systemctl restart docker.service
sudo kubeadm reset
```

检查 docker 的日志也可能有用：

```sh
journalctl -ul docker
```

<!--
## Pods in `RunContainerError`, `CrashLoopBackOff` or `Error` state

Right after `kubeadm init` there should not be any pods in these states.

- If there are pods in one of these states _right after_ `kubeadm init`, please open an
  issue in the kubeadm repo. `coredns` (or `kube-dns`) should be in the `Pending` state
  until you have deployed the network solution.
- If you see Pods in the `RunContainerError`, `CrashLoopBackOff` or `Error` state
  after deploying the network solution and nothing happens to `coredns` (or `kube-dns`),
  it's very likely that the Pod Network solution that you installed is somehow broken.
  You might have to grant it more RBAC privileges or use a newer version. Please file
  an issue in the Pod Network providers' issue tracker and get the issue triaged there.
- If you install a version of Docker older than 1.12.1, remove the `MountFlags=slave` option
  when booting `dockerd` with `systemd` and restart `docker`. You can see the MountFlags in `/usr/lib/systemd/system/docker.service`.
  MountFlags can interfere with volumes mounted by Kubernetes, and put the Pods in `CrashLoopBackOff` state.
  The error happens when Kubernetes does not find `var/run/secrets/kubernetes.io/serviceaccount` files.
-->

## Pods 处于 `RunContainerError`、`CrashLoopBackOff` 或者 `Error` 状态

在 `kubeadm init` 命令运行后，系统中不应该有 pods 处于这类状态。

- 在 `kubeadm init` 命令执行完后，如果有 pods 处于这些状态之一，请在 kubeadm 仓
  库提起一个 issue。`coredns` (或者 `kube-dns`) 应该处于 `Pending` 状态，直到您
  部署了网络解决方案为止。
- 如果在部署完网络解决方案之后，有 Pods 处于
  `RunContainerError`、`CrashLoopBackOff` 或 `Error` 状态之一，并且`coredns` （
  或者 `kube-dns`）仍处于 `Pending` 状态，那很可能是您安装的网络解决方案由于某种
  原因无法工作。您或许需要授予它更多的 RBAC 特权或使用较新的版本。请在 Pod
  Network 提供商的问题跟踪器中提交问题，然后在此处分类问题。
- 如果您安装的 Docker 版本早于 1.12.1，请在使用 `systemd` 来启动 `dockerd` 和重
  启 `docker` 时，删除 `MountFlags=slave` 选项。您可以在
  `/usr/lib/systemd/system/docker.service` 中看到 MountFlags。 MountFlags 可能会
  干扰 Kubernetes 挂载的卷， 并使 Pods 处于 `CrashLoopBackOff` 状态。当
  Kubernetes 不能找到 `var/run/secrets/kubernetes.io/serviceaccount` 文件时会发
  生错误。

<!--
## `coredns` (or `kube-dns`) is stuck in the `Pending` state

This is **expected** and part of the design. kubeadm is network provider-agnostic, so the admin
should [install the pod network solution](/docs/concepts/cluster-administration/addons/)
of choice. You have to install a Pod Network
before CoreDNS may be deployed fully. Hence the `Pending` state before the network is set up.
-->

## `coredns` （或 `kube-dns`）停滞在 `Pending` 状态

这一行为是 **预期之中** 的，因为系统就是这么设计的。 kubeadm 的网络供应商是中立
的，因此管理员应该选择
[安装 pod 的网络解决方案](/docs/concepts/cluster-administration/addons/)。您必须
完成 Pod 的网络配置，然后才能完全部署 CoreDNS。在网络被配置好之前，DNS 组件会一
直处于 `Pending` 状态。

<!--
## `HostPort` services do not work

The `HostPort` and `HostIP` functionality is available depending on your Pod Network
provider. Please contact the author of the Pod Network solution to find out whether
`HostPort` and `HostIP` functionality are available.

Calico, Canal, and Flannel CNI providers are verified to support HostPort.

For more information, see the [CNI portmap documentation](https://github.com/containernetworking/plugins/blob/master/plugins/meta/portmap/README.md).

If your network provider does not support the portmap CNI plugin, you may need to use the [NodePort feature of
services](/docs/concepts/services-networking/service/#nodeport) or use `HostNetwork=true`.
-->

## `HostPort` 服务无法工作

此 `HostPort` 和 `HostIP` 功能是否可用取决于您的 Pod 网络配置。请联系 Pod 解决方
案的作者，以确认 `HostPort` 和 `HostIP` 功能是否可用。

已验证 Calico、Canal 和 Flannel CNI 驱动程序支持 HostPort。

有关更多信息，请参考
[CNI portmap 文档](https://github.com/containernetworking/plugins/blob/master/plugins/meta/portmap/README.md).

如果您的网络提供商不支持 portmap CNI 插件，您或许需要使用
[NodePort 服务的功能](/docs/concepts/services-networking/service/#nodeport) 或者
使用 `HostNetwork=true`。

<!--
## Pods are not accessible via their Service IP

- Many network add-ons do not yet enable [hairpin mode](/docs/tasks/debug-application-cluster/debug-service/#a-pod-cannot-reach-itself-via-service-ip)
  which allows pods to access themselves via their Service IP. This is an issue related to
  [CNI](https://github.com/containernetworking/cni/issues/476). Please contact the network
  add-on provider to get the latest status of their support for hairpin mode.

- If you are using VirtualBox (directly or via Vagrant), you will need to
  ensure that `hostname -i` returns a routable IP address. By default the first
  interface is connected to a non-routable host-only network. A work around
  is to modify `/etc/hosts`, see this [Vagrantfile](https://github.com/errordeveloper/k8s-playground/blob/22dd39dfc06111235620e6c4404a96ae146f26fd/Vagrantfile#L11)
  for an example.
-->

## 无法通过其服务 IP 访问 Pod

- 许多网络附加组件尚未启用
  [hairpin 模式](/docs/tasks/debug-application-cluster/debug-service/#a-pod-cannot-reach-itself-via-service-ip)
  该模式允许 Pod 通过其服务 IP 进行访问。这是与
  [CNI](https://github.com/containernetworking/cni/issues/476) 有关的问题。请与
  网络附加组件提供商联系，以获取他们所提供的 hairpin 模式的最新状态。

- 如果您正在使用 VirtualBox (直接使用或者通过 Vagrant 使用)，您需要确保
  `hostname -i` 返回一个可路由的 IP 地址。默认情况下，第一个接口连接不能路由的仅
  主机网络。解决方法是修改 `/etc/hosts`，请参考示例
  [Vagrantfile](https://github.com/errordeveloper/k8s-playground/blob/22dd39dfc06111235620e6c4404a96ae146f26fd/Vagrantfile#L11)。

<!--
## TLS certificate errors

The following error indicates a possible certificate mismatch.

```none
# kubectl get pods
Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "kubernetes")
```

- Verify that the `$HOME/.kube/config` file contains a valid certificate, and
  regenerate a certificate if necessary. The certificates in a kubeconfig file
  are base64 encoded. The `base64 -d` command can be used to decode the certificate
  and `openssl x509 -text -noout` can be used for viewing the certificate information.
- Unset the `KUBECONFIG` environment variable using:

  ```sh
  unset KUBECONFIG
  ```

  Or set it to the default `KUBECONFIG` location:

  ```sh
  export KUBECONFIG=/etc/kubernetes/admin.conf
  ```

- Another workaround is to overwrite the existing `kubeconfig` for the "admin" user:

  ```sh
  mv  $HOME/.kube $HOME/.kube.bak
  mkdir $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ```
-->

## TLS 证书错误

以下错误指出证书可能不匹配。

```none
# kubectl get pods
Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "kubernetes")
```

- 验证 `$HOME/.kube/config` 文件是否包含有效证书，并在必要时重新生成证书。在
  kubeconfig 文件中的证书是 base64 编码的。该 `base64 -d` 命令可以用来解码证书
  ，`openssl x509 -text -noout` 命令可以用于查看证书信息。
- 使用如下方法取消设置 `KUBECONFIG` 环境变量的值：

  ```sh
  unset KUBECONFIG
  ```

  或者将其设置为默认的 `KUBECONFIG` 位置：

  ```sh
  export KUBECONFIG=/etc/kubernetes/admin.conf
  ```

- 另一个方法是覆盖 `kubeconfig` 的现有用户 "管理员" ：

  ```sh
  mv  $HOME/.kube $HOME/.kube.bak
  mkdir $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ```

<!--
## Default NIC When using flannel as the pod network in Vagrant

The following error might indicate that something was wrong in the pod network:

```sh
Error from server (NotFound): the server could not find the requested resource
```

- If you're using flannel as the pod network inside Vagrant, then you will have to specify the default interface name for flannel.

  Vagrant typically assigns two interfaces to all VMs. The first, for which all hosts are assigned the IP address `10.0.2.15`, is for external traffic that gets NATed.

  This may lead to problems with flannel, which defaults to the first interface on a host. This leads to all hosts thinking they have the same public IP address. To prevent this, pass the `--iface eth1` flag to flannel so that the second interface is chosen.
-->

## 在 Vagrant 中使用 flannel 作为 pod 网络时的默认 NIC

以下错误可能表明 Pod 网络中出现问题：

```sh
Error from server (NotFound): the server could not find the requested resource
```

- 如果你正在 Vagrant 中使用 flannel 作为 pod 网络，则必须指定 flannel 的默认接口
  名称。

  Vagrant 通常为所有 VM 分配两个接口。第一个为所有主机分配了 IP 地址
  `10.0.2.15`，用于获得 NATed 的外部流量。

  这可能会导致 flannel 出现问题，它默认为主机上的第一个接口。这导致所有主机认为
  它们具有相同的公共 IP 地址。为防止这种情况，传递 `--iface eth1` 标志给 flannel
  以便选择第二个接口。

<!--
## Non-public IP used for containers

In some situations `kubectl logs` and `kubectl run` commands may return with the following errors in an otherwise functional cluster:

```sh
Error from server: Get https://10.19.0.41:10250/containerLogs/default/mysql-ddc65b868-glc5m/mysql: dial tcp 10.19.0.41:10250: getsockopt: no route to host
```

- This may be due to Kubernetes using an IP that can not communicate with other IPs on the seemingly same subnet, possibly by policy of the machine provider.
- Digital Ocean assigns a public IP to `eth0` as well as a private one to be used internally as anchor for their floating IP feature, yet `kubelet` will pick the latter as the node's `InternalIP` instead of the public one.

  Use `ip addr show` to check for this scenario instead of `ifconfig` because `ifconfig` will not display the offending alias IP address. Alternatively an API endpoint specific to Digital Ocean allows to query for the anchor IP from the droplet:

  ```sh
  curl http://169.254.169.254/metadata/v1/interfaces/public/0/anchor_ipv4/address
  ```

  The workaround is to tell `kubelet` which IP to use using `--node-ip`. When using Digital Ocean, it can be the public one (assigned to `eth0`) or the private one (assigned to `eth1`) should you want to use the optional private network. The [`KubeletExtraArgs` section of the kubeadm `NodeRegistrationOptions` structure](https://github.com/kubernetes/kubernetes/blob/release-1.13/cmd/kubeadm/app/apis/kubeadm/v1beta1/types.go) can be used for this.

  Then restart `kubelet`:

  ```sh
  systemctl daemon-reload
  systemctl restart kubelet
  ```
-->

## 容器使用的非公共 IP

在某些情况下 `kubectl logs` 和 `kubectl run` 命令或许会返回以下错误，即便除此之
外集群一切功能正常：

```sh
Error from server: Get https://10.19.0.41:10250/containerLogs/default/mysql-ddc65b868-glc5m/mysql: dial tcp 10.19.0.41:10250: getsockopt: no route to host
```

- 这或许是由于 Kubernetes 使用的 IP 无法与看似相同的子网上的其他 IP 进行通信的缘
  故，可能是由机器提供商的政策所导致的。
- Digital Ocean 既分配一个共有 IP 给 `eth0`，也分配一个私有 IP 在内部用作其浮动
  IP 功能的锚点，然而 `kubelet` 将选择后者作为节点的 `InternalIP` 而不是公共 IP

  使用 `ip addr show` 命令代替 `ifconfig` 命令去检查这种情况，因为 `ifconfig` 命
  令 不会显示有问题的别名 IP 地址。或者指定的 Digital Ocean 的 API 端口允许从
  droplet 中 查询 anchor IP：

  ```sh
  curl http://169.254.169.254/metadata/v1/interfaces/public/0/anchor_ipv4/address
  ```

  解决方法是通知 `kubelet` 使用哪个 `--node-ip`。当使用 Digital Ocean 时，可以是
  公网 IP（分配给 `eth0`的）， 或者是私网 IP（分配给 `eth1` 的）。私网 IP 是可选
  的。 这个
  [`KubeletExtraArgs` section of the kubeadm `NodeRegistrationOptions` structure](https://github.com/kubernetes/kubernetes/blob/release-1.13/cmd/kubeadm/app/apis/kubeadm/v1beta1/types.go)
  被用来处理这种情况。

  然后重启 `kubelet`：

  ```sh
  systemctl daemon-reload
  systemctl restart kubelet
  ```

  <!--

## `coredns` pods have `CrashLoopBackOff` or `Error` state

If you have nodes that are running SELinux with an older version of Docker you
might experience a scenario where the `coredns` pods are not starting. To solve
that you can try one of the following options:

- Upgrade to a
  [newer version of Docker](/docs/setup/production-environment/container-runtimes/#docker).

- [Disable SELinux](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/security-enhanced_linux/sect-security-enhanced_linux-enabling_and_disabling_selinux-disabling_selinux).
- Modify the `coredns` deployment to set `allowPrivilegeEscalation` to `true`:

```bash
kubectl -n kube-system get deployment coredns -o yaml | \
  sed 's/allowPrivilegeEscalation: false/allowPrivilegeEscalation: true/g' | \
  kubectl apply -f -
```

Another cause for CoreDNS to have `CrashLoopBackOff` is when a CoreDNS Pod
deployed in Kubernetes detects a loop.
[A number of workarounds](https://github.com/coredns/coredns/tree/master/plugin/loop#troubleshooting-loops-in-kubernetes-clusters)
are available to avoid Kubernetes trying to restart the CoreDNS Pod every time
CoreDNS detects the loop and exits. -->

## `coredns` pods 有 `CrashLoopBackOff` 或者 `Error` 状态

如果有些节点运行的是旧版本的 Docker，同时启用了 SELinux，您或许会遇到 `coredns`
pods 无法启动的情况。要解决此问题，您可以尝试以下选项之一：

- 升级到
  [Docker 的较新版本](/docs/setup/production-environment/container-runtimes/#docker)。

- [禁用 SELinux](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/security-enhanced_linux/sect-security-enhanced_linux-enabling_and_disabling_selinux-disabling_selinux).
- 修改 `coredns` 部署以设置 `allowPrivilegeEscalation` 为 `true`：

```bash
kubectl -n kube-system get deployment coredns -o yaml | \
  sed 's/allowPrivilegeEscalation: false/allowPrivilegeEscalation: true/g' | \
  kubectl apply -f -
```

CoreDNS 处于 `CrashLoopBackOff` 时的另一个原因是当 Kubernetes 中部署的 CoreDNS
Pod 检测到环路时
。[有许多解决方法](https://github.com/coredns/coredns/tree/master/plugin/loop#troubleshooting-loops-in-kubernetes-clusters)
可以避免在每次 CoreDNS 监测到循环并退出时，Kubernetes 尝试重启 CoreDNS Pod 的情
况。

{{< warning >}}

<!--
Disabling SELinux or setting `allowPrivilegeEscalation` to `true` can compromise
the security of your cluster.
-->

**警告**：禁用 SELinux 或设置 `allowPrivilegeEscalation` 为 `true` 可能会损害集
群的安全性。 {{< /warning >}}

<!--
## etcd pods restart continually

If you encounter the following error:

```
rpc error: code = 2 desc = oci runtime error: exec failed: container_linux.go:247: starting container process caused "process_linux.go:110: decoding init error from pipe caused \"read parent: connection reset by peer\""
```

this issue appears if you run CentOS 7 with Docker 1.13.1.84.
This version of Docker can prevent the kubelet from executing into the etcd container.

To work around the issue, choose one of these options:

- Roll back to an earlier version of Docker, such as 1.13.1-75
```
yum downgrade docker-1.13.1-75.git8633870.el7.centos.x86_64 docker-client-1.13.1-75.git8633870.el7.centos.x86_64 docker-common-1.13.1-75.git8633870.el7.centos.x86_64
```

- Install one of the more recent recommended versions, such as 18.06:
```bash
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce-18.06.1.ce-3.el7.x86_64
```
-->

## etcd pods 持续重启

如果您遇到以下错误：

```
rpc error: code = 2 desc = oci runtime error: exec failed: container_linux.go:247: starting container process caused "process_linux.go:110: decoding init error from pipe caused \"read parent: connection reset by peer\""
```

如果您使用 Docker 1.13.1.84 运行 CentOS 7 就会出现这种问题。此版本的 Docker 会阻
止 kubelet 在 etcd 容器中执行。

为解决此问题，请选择以下选项之一：

- 回滚到早期版本的 Docker，例如 1.13.1-75

```
yum downgrade docker-1.13.1-75.git8633870.el7.centos.x86_64 docker-client-1.13.1-75.git8633870.el7.centos.x86_64 docker-common-1.13.1-75.git8633870.el7.centos.x86_64
```

- 安装较新的推荐版本之一，例如 18.06:

```bash
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce-18.06.1.ce-3.el7.x86_64
```

<!--
## Not possible to pass a comma separated list of values to arguments inside a `--component-extra-args` flag

`kubeadm init` flags such as `--component-extra-args` allow you to pass custom arguments to a control-plane
component like the kube-apiserver. However, this mechanism is limited due to the underlying type used for parsing
the values (`mapStringString`).

If you decide to pass an argument that supports multiple, comma-separated values such as
`--apiserver-extra-args "enable-admission-plugins=LimitRanger,NamespaceExists"` this flag will fail with
`flag: malformed pair, expect string=string`. This happens because the list of arguments for
`--apiserver-extra-args` expects `key=value` pairs and in this case `NamespacesExists` is considered
as a key that is missing a value.

Alternatively, you can try separating the `key=value` pairs like so:
`--apiserver-extra-args "enable-admission-plugins=LimitRanger,enable-admission-plugins=NamespaceExists"`
but this will result in the key `enable-admission-plugins` only having the value of `NamespaceExists`.

A known workaround is to use the kubeadm [configuration file](/docs/setup/production-environment/tools/kubeadm/control-plane-flags/#apiserver-flags).
-->

## 无法将以逗号分隔的值列表传递给 `--component-extra-args` 标志内的参数

`kubeadm init` 标志例如 `--component-extra-args` 允许您将自定义参数传递给像
kube-apiserver 这样的控制平面组件。然而，由于解析 (`mapStringString`) 的基础类型
值，此机制将受到限制。

如果您决定传递一个支持多个逗号分隔值（例如
`--apiserver-extra-args "enable-admission-plugins=LimitRanger,NamespaceExists"`）
参数，将出现 `flag: malformed pair, expect string=string` 错误。发生这种问题是�
为参数列表 `--apiserver-extra-args` 预期的是 `key=value` 形式，而这里的
`NamespacesExists` 被误认为是缺少取值的键名。

一种解决方法是尝试分离 `key=value` 对，像这样：
`--apiserver-extra-args "enable-admission-plugins=LimitRanger,enable-admission-plugins=NamespaceExists"`
但这将导致键 `enable-admission-plugins` 仅有值 `NamespaceExists`。

已知的解决方法是使用 kubeadm
[配置文件](/docs/setup/production-environment/tools/kubeadm/control-plane-flags/#apiserver-flags)。

<!--
## kube-proxy scheduled before node is initialized by cloud-controller-manager

In cloud provider scenarios, kube-proxy can end up being scheduled on new worker nodes before
the cloud-controller-manager has initialized the node addresses. This causes kube-proxy to fail
to pick up the node's IP address properly and has knock-on effects to the proxy function managing
load balancers.

The following error can be seen in kube-proxy Pods:
```
server.go:610] Failed to retrieve node IP: host IP unknown; known addresses: []
proxier.go:340] invalid nodeIP, initializing kube-proxy with 127.0.0.1 as nodeIP
```

A known solution is to patch the kube-proxy DaemonSet to allow scheduling it on control-plane
nodes regardless of their conditions, keeping it off of other nodes until their initial guarding
conditions abate:
```
kubectl -n kube-system patch ds kube-proxy -p='{ "spec": { "template": { "spec": { "tolerations": [ { "key": "CriticalAddonsOnly", "operator": "Exists" }, { "effect": "NoSchedule", "key": "node-role.kubernetes.io/master" } ] } } } }'
```

The tracking issue for this problem is [here](https://github.com/kubernetes/kubeadm/issues/1027).
-->

## 在节点被云控制管理器初始化之前，kube-proxy 就被调度了

在云环境场景中，可能出现在云控制管理器完成节点地址初始化之前，kube-proxy 就被调
度到新节点了。这会导致 kube-proxy 无法正确获取节点的 IP 地址，并对管理负载平衡器
的代理功能产生连锁反应。

在 kube-proxy Pod 中可以看到以下错误：

```
server.go:610] Failed to retrieve node IP: host IP unknown; known addresses: []
proxier.go:340] invalid nodeIP, initializing kube-proxy with 127.0.0.1 as nodeIP
```

一种已知的解决方案是修补 kube-proxy DaemonSet，以允许在控制平面节点上调度它，而
不管它们的条件如何，将其与其他节点保持隔离，直到它们的初始保护条件消除：

```
kubectl -n kube-system patch ds kube-proxy -p='{ "spec": { "template": { "spec": { "tolerations": [ { "key": "CriticalAddonsOnly", "operator": "Exists" }, { "effect": "NoSchedule", "key": "node-role.kubernetes.io/master" } ] } } } }'
```

此问题的跟踪 [在这里](https://github.com/kubernetes/kubeadm/issues/1027)。

<!--
## The NodeRegistration.Taints field is omitted when marshalling kubeadm configuration

*Note: This [issue](https://github.com/kubernetes/kubeadm/issues/1358) only applies to tools that marshal kubeadm types (e.g. to a YAML configuration file). It will be fixed in kubeadm API v1beta2.*

By default, kubeadm applies the `node-role.kubernetes.io/master:NoSchedule` taint to control-plane nodes.
If you prefer kubeadm to not taint the control-plane node, and set `InitConfiguration.NodeRegistration.Taints` to an empty slice,
the field will be omitted when marshalling. When the field is omitted, kubeadm applies the default taint.

There are at least two workarounds:

1. Use the `node-role.kubernetes.io/master:PreferNoSchedule` taint instead of an empty slice. [Pods will get scheduled on masters](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/), unless other nodes have capacity.

2. Remove the taint after kubeadm init exits:
```bash
kubectl taint nodes NODE_NAME node-role.kubernetes.io/master:NoSchedule-
```
-->

## NodeRegistration.Taints 字段在编组 kubeadm 配置时丢失

_注意：这个 [问题](https://github.com/kubernetes/kubeadm/issues/1358) 仅适用于操
控 kubeadm 数据类型的工具（例如，YAML 配置文件）。它将在 kubeadm API v1beta2 修
复。_

默认情况下，kubeadm 将 `node-role.kubernetes.io/master:NoSchedule` 污点应用于控
制平面节点。如果您希望 kubeadm 不污染控制平面节点，并将
`InitConfiguration.NodeRegistration.Taints` 设置成空切片，则应在编组时省略该字段
。如果省略该字段，则 kubeadm 将应用默认污点。

至少有两种解决方法：

1. 使用 `node-role.kubernetes.io/master:PreferNoSchedule` 污点代替空切片。除非其
   他节点具有容量
   ，[否则将在主节点上调度 Pods](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)。

2. 在 kubeadm init 退出后删除污点：

```bash
kubectl taint nodes NODE_NAME node-role.kubernetes.io/master:NoSchedule-
```

{{% /capture %}}
