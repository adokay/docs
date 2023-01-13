---
title: "高级选项和配置"
weight: 45
aliases:
  - /k3s/latest/en/running/
  - /k3s/latest/en/configuration/
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

本文描述了用于运行和管理 K3s 的高级设置，以及为 K3s 准备主机操作系统所需的步骤。

## 证书轮换

### 自动轮换
默认情况下，K3s 中的证书在 12 个月后过期。

如果证书已经过期或剩余的时间不足 90 天，则在 K3s 重启时轮换证书。

### 手动轮换

要手动轮换证书，请使用 `k3s certificate rotate` 子命令：

```bash
# Stop K3s
systemctl stop k3s
# Rotate certificates
k3s certificate rotate
# Start K3s
systemctl start k3s
```

你可以通过指定证书名称来轮换单个或多个证书：

```bash
k3s certificate rotate --service <SERVICE>,<SERVICE>
```

可以轮换的证书：`admin`、`api-server`、`controller-manager`、`scheduler`、`k3s-controller`, `k3s-server`, `cloud-controller`, `etcd`, `auth-proxy`, `kubelet`，`kube-proxy`。


## 自动部署清单

在 `/var/lib/rancher/k3s/server/manifests` 中找到的任何文件都会以类似 `kubectl apply` 的方式自动部署到 Kubernetes，在启动和在磁盘上更改文件时也是一样。删除该目录的文件不会同时删除集群中相应的资源。

有关部署 Helm Chart 的更多信息，请参阅 [Helm](../helm/helm.md) 部分。

## 使用 Docker 作为容器运行时

K3s 包含并默认为 [containerd](https://containerd.io/)，它是一个行业标准的容器运行时。
从 Kubernetes 1.24 开始，Kubelet 不再包含 dockershim，该组件允许 kubelet 与 dockerd 通信。
K3s 1.24 及更高版本包括了 [cri-dockerd](https://github.com/Mirantis/cri-dockerd)，它允许你无缝升级旧的 K3s 版本，同时继续使用 Docker 容器运行时。

要使用 Docker 而不是 containerd：

1. 在 K3s 节点上安装 Docker。你可以使用 Rancher 的一个 [Docker 安装脚本](https://github.com/rancher/install-docker)来安装 Docker：

   ```bash
   curl https://releases.rancher.com/install-docker/20.10.sh | sh
   ```

2. 使用 `--docker` 选项安装 K3s：

   ```bash
   curl -sfL https://get.k3s.io | sh -s - --docker
   ```

3. 确认集群可用：

   ```bash
   $ sudo k3s kubectl get pods --all-namespaces
   NAMESPACE     NAME                                     READY   STATUS      RESTARTS   AGE
   kube-system   local-path-provisioner-6d59f47c7-lncxn   1/1     Running     0          51s
   kube-system   metrics-server-7566d596c8-9tnck          1/1     Running     0          51s
   kube-system   helm-install-traefik-mbkn9               0/1     Completed   1          51s
   kube-system   coredns-8655855d6-rtbnb                  1/1     Running     0          51s
   kube-system   svclb-traefik-jbmvl                      2/2     Running     0          43s
   kube-system   traefik-758cd5fc85-2wz97                 1/1     Running     0          43s
   ```

4. 确认 Docker 容器正在运行：

   ```bash
   $ sudo docker ps
   CONTAINER ID        IMAGE                     COMMAND                  CREATED              STATUS              PORTS               NAMES
   3e4d34729602        897ce3c5fc8f              "entry"                  About a minute ago   Up About a minute                       k8s_lb-port-443_svclb-traefik-jbmvl_kube-system_d46f10c6-073f-4c7e-8d7a-8e7ac18f9cb0_0
   bffdc9d7a65f        rancher/klipper-lb        "entry"                  About a minute ago   Up About a minute                       k8s_lb-port-80_svclb-traefik-jbmvl_kube-system_d46f10c6-073f-4c7e-8d7a-8e7ac18f9cb0_0
   436b85c5e38d        rancher/library-traefik   "/traefik --configfi…"   About a minute ago   Up About a minute                       k8s_traefik_traefik-758cd5fc85-2wz97_kube-system_07abe831-ffd6-4206-bfa1-7c9ca4fb39e7_0
   de8fded06188        rancher/pause:3.1         "/pause"                 About a minute ago   Up About a minute                       k8s_POD_svclb-traefik-jbmvl_kube-system_d46f10c6-073f-4c7e-8d7a-8e7ac18f9cb0_0
   7c6a30aeeb2f        rancher/pause:3.1         "/pause"                 About a minute ago   Up About a minute                       k8s_POD_traefik-758cd5fc85-2wz97_kube-system_07abe831-ffd6-4206-bfa1-7c9ca4fb39e7_0
   ae6c58cab4a7        9d12f9848b99              "local-path-provisio…"   About a minute ago   Up About a minute                       k8s_local-path-provisioner_local-path-provisioner-6d59f47c7-lncxn_kube-system_2dbd22bf-6ad9-4bea-a73d-620c90a6c1c1_0
   be1450e1a11e        9dd718864ce6              "/metrics-server"        About a minute ago   Up About a minute                       k8s_metrics-server_metrics-server-7566d596c8-9tnck_kube-system_031e74b5-e9ef-47ef-a88d-fbf3f726cbc6_0
   4454d14e4d3f        c4d3d16fe508              "/coredns -conf /etc…"   About a minute ago   Up About a minute                       k8s_coredns_coredns-8655855d6-rtbnb_kube-system_d05725df-4fb1-410a-8e82-2b1c8278a6a1_0
   c3675b87f96c        rancher/pause:3.1         "/pause"                 About a minute ago   Up About a minute                       k8s_POD_coredns-8655855d6-rtbnb_kube-system_d05725df-4fb1-410a-8e82-2b1c8278a6a1_0
   4b1fddbe6ca6        rancher/pause:3.1         "/pause"                 About a minute ago   Up About a minute                       k8s_POD_local-path-provisioner-6d59f47c7-lncxn_kube-system_2dbd22bf-6ad9-4bea-a73d-620c90a6c1c1_0
   64d3517d4a95        rancher/pause:3.1         "/pause"
   ```

## 使用 etcdctl

etcdctl 提供了一个与 etcd 服务器交互的 CLI。K3s 附带 etcdctl。

如果你想使用 etcdctl 与 K3s 的嵌入式 etcd 进行交互，请参阅[官方文档](https://etcd.io/docs/latest/install/)安装 etcdctl。

```bash
ETCD_VERSION="v3.5.5"
ETCD_URL="https://github.com/etcd-io/etcd/releases/download/${ETCD_VERSION}/etcd-${ETCD_VERSION}-linux-amd64.tar.gz"
curl -sL ${ETCD_URL} | sudo tar -zxv --strip-components=1 -C /usr/local/bin
```

然后，你可以将 etcdctl 配置为使用 K3s 管理的证书和密钥来进行身份验证，从而使用 etcdctl：

```bash
sudo etcdctl version \
  --cacert=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt \
  --cert=/var/lib/rancher/k3s/server/tls/etcd/client.crt \
  --key=/var/lib/rancher/k3s/server/tls/etcd/client.key
```

## 配置 Containerd

K3s 会在 `/var/lib/rancher/k3s/agent/etc/containerd/config.toml` 中为 containerd 生成 config.toml。

如果要对这个文件进行高级定制，你可以在同一目录中创建另一个名为 `config.toml.tmpl` 的文件，此文件将会代替默认设置。

`config.toml.tmpl` 是一个 Go 模板文件，并且 `config.Node` 结构会被传递给模板。有关如何使用该结构自定义配置文件的 Linux 和 Windows 示例，请参阅[此文件夹](https://github.com/k3s-io/k3s/blob/master/pkg/agent/templates)。

## NVIDIA 容器运行时支持

如果 NVIDIA 容器运行时在 K3s 启动时存在，K3s 将自动检测并配置它。

1. 按照以下说明在节点上安装 nvidia-container 包仓库：
   https://nvidia.github.io/libnvidia-container/
1. 安装 nvidia 容器运行时包。例如：
   `apt install -y nvidia-container-runtime cuda-drivers-fabricmanager-515 nvidia-headless-515-server`
1. 安装 K3s，如果已经安装则重启它：
   `curl -ksL get.k3s.io | sh -`
1. 确认 K3s 已经找到 nvidia 容器运行时：
   `grep nvidia /var/lib/rancher/k3s/agent/etc/containerd/config.toml`

这将根据找到的运行时可执行文件自动将 `nvidia` 和/或 `nvidia-experimental` 运行时添加到 containerd 配置中。
你仍然必须向集群添加 RuntimeClass 定义，并通过在 Pod 规范中设置 `runtimeClassName: nvidia` 来部署显式请求运行时的 Pod：
```yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: nvidia
handler: nvidia
---
apiVersion: v1
kind: Pod
metadata:
  name: nbody-gpu-benchmark
  namespace: default
spec:
  restartPolicy: OnFailure
  runtimeClassName: nvidia
  containers:
  - name: cuda-container
    image: nvcr.io/nvidia/k8s/cuda-sample:nbody
    args: ["nbody", "-gpu", "-benchmark"]
    resources:
      limits:
        nvidia.com/gpu: 1
    env:
    - name: NVIDIA_VISIBLE_DEVICES
      value: all
    - name: NVIDIA_DRIVER_CAPABILITIES
      value: all
```

请注意，NVIDIA Container Runtime 也经常与 [NVIDIA Device Plugin](https://github.com/NVIDIA/k8s-device-plugin/) 和 [GPU Feature Discovery](https://github.com/NVIDIA/gpu-feature-discovery/) 一起使用，它们必须单独安装，而且需要修改以确保 Pod 规范能包括 `runtimeClassName: nvidia`，如前所述。

## 运行无 Agent 的 Server（实验性）
> **警告**：此功能是实验性的。

当使用 `--disable-agent` 标志启动时，Server 不运行 kubelet、容器运行时或 CNI。它们不会在集群中注册 Node 资源，也不会出现在 `kubectl get nodes` 输出中。
因为它们不托管 kubelet，所以它们不能运行 pod，也不能由依赖枚举集群节点的 Operator 管理，包括嵌入式 etcd controller 和 system-upgrade-controller。

如果你想让 control plane 节点不被 Agent 和工作负载发现，你可以运行无 Agent 的 Server，但是代价是由于缺乏集群 Operator 支持，管理开销会增加。

## 使用 Rootless 模式运行 Server（实验性）
> **警告**：此功能是实验性的。

Rootless 模式允许非特权用户运行 K3s Server，这样可以保护主机上真正的 root 免受潜在的容器攻击。

有关 Rootless 模式 Kubernetes 的更多信息，请参阅[此处](https://rootlesscontaine.rs/)。

### Rootless 模式的已知问题

* **端口**

   如果以 Rootless 模式运行，将创建一个新的网络命名空间。换言之，K3s 实例在网络与主机完全分离的情况下运行。
   要从主机访问在 K3s 中运行的 Service，唯一的方法是设置转发到 K3s 网络命名空间的端口。
   Rootless 模式下的 K3s 包含控制器，它会自动将 6443 和低于 1024 的 Service 端口绑定到偏移量为 10000 的主机。

   例如，端口 80 上的 Service 在主机上会变成 10080，但 8080 会变成 8080，没有任何偏移。目前只有 LoadBalancer Service 是自动绑定的。

* **Cgroups**

   不支持 Cgroup v1 和 Hybrid v1/v2，仅支持纯 Cgroup v2。如果 K3s 在 Rootless 模式下运行时由于缺少 cgroup 而无法启动，很可能你的节点处于 Hybrid 模式，而且“丢失”的 cgroup 仍然绑定了 v1 控制器。

* **多节点/多进程集群**

   目前，我们不支持多节点无根集群或同一节点上的多个无根 k3s 进程。有关详细信息，请参阅 [#6488](https://github.com/k3s-io/k3s/issues/6488#issuecomment-1314998091)。

### 启动 Rootless Server
* 启用 cgroup v2 授权，请参阅 https://rootlesscontaine.rs/getting-started/common/cgroup2/。
   此步骤是必需的。如果没有正确的 cgroups 授权，rootless kubelet 将无法启动。

* 从 [`https://github.com/k3s-io/k3s/blob/<VERSION>/k3s-rootless.service`](https://github.com/k3s-io/k3s/blob/master/k3s-rootless.service) 下载 `k3s-rootless.service`。
   确保使用了相同版本的 `k3s-rootless.service` 和 `k3s`。

* 将 `k3s-rootless.service` 安装到 `~/.config/systemd/user/k3s-rootless.service`。
   不支持将此文件安装为全系统服务 (`/etc/systemd/...`)。
   根据 `k3s` 二进制文件的路径，你可能需要修改文件的 `ExecStart=/usr/local/bin/k3s ...` 行。

* 运行 `systemctl --user daemon-reload`

* 运行 `systemctl --user enable --now k3s-rootless`

* 运行 `KUBECONFIG=~/.kube/k3s.yaml kubectl get pods -A`，并确保 Pod 正在运行。

> **注意**：由于终端会话不允许 cgroup v2 授权，因此不要尝试在终端上运行 `k3s server --rootless`。
> 如果你确实需要在终端上使用，请使用 `systemd-run --user -p Delegate=yes --tty k3s server --rooless` 将其包装在 systemd 范围内。

### 高级无根配置

Rootless K3s 使用 [rootlesskit](https://github.com/rootless-containers/rootlesskit) 和 [slirp4netns](https://github.com/rootless-containers/slirp4netns) 在主机和用户网络命名空间之间进行通信。
rootlesskit 和 slirp4nets 使用的一些配置可以通过环境变量来设置。设置它们的最佳方法是将它们添加到 k3s-rootless systemd 单元的 `Environment` 字段中。

| 变量 | 默认 | 描述 |
|--------------------------------------|--------------|------------
| `K3S_ROOTLESS_MTU` | 1500 | 为 slirp4netns 虚拟接口设置 MTU。 |
| `K3S_ROOTLESS_CIDR` | 10.41.0.0/16 | 设置 slirp4netns 虚拟接口使用的 CIDR。 |
| `K3S_ROOTLESS_ENABLE_IPV6` | autotedected | 启用 slirp4netns IPv6 支持。如果未指定，则在 K3s 配置为双栈时自动启用。 |
| `K3S_ROOTLESS_PORT_DRIVER` | builtin | 选择无根 port driver，可选值是 `builtin` 或 `slirp4netns`。`builtin` 速度更快，但会伪装入站数据包的原始源地址。 |
| `K3S_ROOTLESS_DISABLE_HOST_LOOPBACK` | true | 控制是否允许通过网关接口访问主机的环回地址。出于安全原因，建议不要更改此设置。 |

### Rootless 模式故障排除

* 运行 `systemctl --user status k3s-rootless` 来检查 daemon 状态
* 运行 `journalctl --user -f -u k3s-rootless` 来查看​​ daemon 日志
* 另见 https://rootlesscontaine.rs/

## 节点标签和污点

K3s Agent 可以通过 `--node-label` 和 `--node-taint` 选项来配置，它们会为 kubelet 添加标签和污点。这两个选项仅在[注册时](../reference/agent-config.md#agent-的节点标签和污点)添加标签和/或污点，因此只能在节点首次加入集群时设置。

当前所有的 Kubernetes 版本都限制节点注册到带有 `kubernetes.io` 和 `k8s.io` 前缀的大部分标签，特别是 `kubernetes.io/role` 标签。如果你尝试启动带有不允许的标签的节点，K3s 将无法启动。正如 Kubernetes 作者所说：

> 不允许节点断言自己的角色标签。节点角色通常用于识别节点的特权或 control plane 类型，如果允许节点将自己标记到该池，那么受感染的节点将能吸引可授予更高特权凭证访问权限的工作负载（如 control plane 守护进程）。

有关详细信息，请参阅 [SIG-Auth KEP 279](https://github.com/kubernetes/enhancements/blob/master/keps/sig-auth/279-limit-node-access/README.md#proposal)。

如果你想在节点注册后更改节点标签和污点，或者添加保留标签，请使用 `kubectl`。关于如何添加[污点](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)和[节点标签](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/#add-a-label-to-a-node)的详细信息，请参阅官方 Kubernetes 文档。

## 使用安装脚本启动服务

安装脚本将自动检测你的操作系统使用的是 systemd 还是 openrc，并在安装过程中启动该服务。
* 使用 openrc 运行时，将在 `/var/log/k3s.log` 中创建日志。
* 使用 systemd 运行时，将在 `/var/log/syslog` 中创建日志，你可以通过 `journalctl -u k3s`（Agent 上是 `journalctl -u k3s-agent`）查看日志。

使用安装脚本禁用自动启动和服务启用的示例：

```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_SKIP_START=true INSTALL_K3S_SKIP_ENABLE=true sh -
```

## 其他操作系统准备

### 旧的 iptables 版本

几个主流 Linux 发行版发布的 iptables 版本包含一个错误，该错误会导致重复规则的累积，从而对节点的性能和稳定性产生负面影响。有关如何确定你是否受此问题影响，请参阅 [issue #3117](https://github.com/k3s-io/k3s/issues/3117)。

K3s 具有一个可以正常运行的 iptables (v1.8.8) 版本。你可以通过使用 `--prefer-bundled-bin` 选项来启动 K3s，或从操作系统中卸载 iptables/nftables 包，从而让 K3s 使用捆绑的 iptables 版本。

:::info 版本

`--prefer-bundled-bin` 标志从 2022-12 版本开始可用（v1.26.0+k3s1、v1.25.5+k3s1、v1.24.9+k3s1、v1.23.15+k3s1）。

:::

### Red Hat Enterprise Linux / CentOS

建议关闭 firewalld：
```bash
systemctl disable firewalld --now
```

如果启用，则需要禁用 nm-cloud-setup 并重新启动节点：
```bash
systemctl disable nm-cloud-setup.service nm-cloud-setup.timer
reboot
```

### Raspberry Pi

Raspberry Pi OS 基于 Debian，可能会受到旧 iptables 版本的影响。请参阅[解决方法](#旧的-iptables-版本)。

标准 Raspberry Pi OS 不会在启用 `cgroups` 的情况下开始。**K3S** 需要 `cgroups` 来启动 systemd 服务。你可以通过将 `cgroup_memory=1 cgroup_enable=memory` 附加到 `/boot/cmdline.txt` 来启用 `cgroups` 。

示例 cmdline.txt：
```
console=serial0,115200 console=tty1 root=PARTUUID=58b06195-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait cgroup_memory=1 cgroup_enable=memory
```

从 Ubuntu 21.10 开始，对 Raspberry Pi 的 vxlan 支持已移至单独的内核模块中。
```bash
sudo apt install linux-modules-extra-raspi
```

## 在 Docker 中运行 K3s

在 Docker 中运行 K3s 有几种方法：

<Tabs>
<TabItem value="K3d" default>

[k3d](https://github.com/k3d-io/k3d) 是一个用于在 Docker 中轻松运行 K3s 的实用程序。

你可以使用 MacOS 上的 [brew](https://brew.sh/) 实用程序来安装它：

```bash
brew install k3d
```

</TabItem>
<TabItem value="Docker Compose">

K3s repo 中的 `docker-compose.yml` 是一个[示例](https://github.com/k3s-io/k3s/blob/master/docker-compose.yml)，介绍了如何从 Docker 运行 K3s。要在这个 repo 中运行 `docker-compose`，运行：

```bash
docker-compose up --scale agent=3
# kubeconfig is written to current dir

kubectl --kubeconfig kubeconfig.yaml get node

NAME           STATUS   ROLES    AGE   VERSION
497278a2d6a2   Ready    <none>   11s   v1.13.2-k3s2
d54c8b17c055   Ready    <none>   11s   v1.13.2-k3s2
db7a5a5a5bdd   Ready    <none>   12s   v1.13.2-k3s2
```

要仅在 Docker 中运行 Agent，请使用 `docker-compose up agent`。

</TabItem>
<TabItem value="Docker">

要使用 Docker，你还可以使用 `rancher/k3s` 镜像来运行 K3s Server 和 Agent。
使用 `docker run` 命令：

```bash
sudo docker run \
  -d --tmpfs /run \
  --tmpfs /var/run \
  -e K3S_URL=${SERVER_URL} \
  -e K3S_TOKEN=${NODE_TOKEN} \
  --privileged rancher/k3s:vX.Y.Z
```
</TabItem>
</Tabs>

## SELinux 支持

:::info 版本

从 v1.19.4+k3s1 起可用

:::

如果你在默认启用 SELinux 的系统（例如 CentOS）上安装 K3s，则必须确保已安装正确的 SELinux 策略。

<Tabs>
<TabItem value="自动安装" default>

如果系统兼容，而且没有进行离线安装，那么[安装脚本](../installation/configuration.md#使用安装脚本的选项)将自动从 Rancher RPM 仓库安装 SELinux RPM。你通过设置 `INSTALL_K3S_SKIP_SELINUX_RPM=true` 来跳过自动安装。

</TabItem>

<TabItem value="手动安装" default>

可以使用以下命令安装必要的策略：
```bash
yum install -y container-selinux selinux-policy-base
yum install -y https://rpm.rancher.io/k3s/latest/common/centos/7/noarch/k3s-selinux-0.2-1.el7_8.noarch.rpm
```

要让安装脚本报告 warning 而不是 fail，你可以设置环境变量 `INSTALL_K3S_SELINUX_WARN=true`。
</TabItem>
</Tabs>

### 启用 SELinux 强制执行

要利用 SELinux，请在启动 K3s Server 和 Agent 时指定 `--selinux` 标志。

你也可以在 K3s [配置文件](#)中指定此选项。

```
selinux: true
```

不支持在 SELinux 下使用自定义 `--data-dir`。要自定义它，你可能需要自行编写自定义策略。如需指导，你可以参考 [containers/container-selinux](https://github.com/containers/container-selinux) 仓库，仓库包含 Container Runtime 的 SELinux 策略文件，同时你可以参考 [rancher/k3s-selinux](https://github.com/rancher/k3s-selinux) 仓库，该仓库包含 K3s 的 SELinux 策略。

## 启用 eStargz 的 Lazy Pulling（实验性）

### 什么是 Lazy Pulling 和 eStargz？

拉取镜像是容器生命周期中比较耗时的步骤之一
（根据 [Harter 等人](https://www.usenix.org/conference/fast16/technical-sessions/presentation/harter)的说法）。

> 拉包占容器启动时间的 76%，但却只读取了 6.4% 的数据。

为了解决这个问题，K3s 的实验功能支持镜像内容的 *lazy pulling*。
这允许 K3s 在拉取整个镜像之前启动一个容器。
必要的内容块（例如单个文件）是按需获取的。
对于大镜像而言，这种技术可以缩短容器启动延迟。

要启用 lazy pulling，你需要将目标镜像格式化为 [*eStargz*](https://github.com/containerd/stargz-snapshotter/blob/main/docs/stargz-estargz.md)。
这是 OCI 的一个替代品，但它 100% OCI 兼容镜像格式，用于 Lazy Pulling。
由于兼容性，eStargz 可以推送到标准容器镜像仓库（例如 ghcr.io），并且即使在 eStargz-agnostic 运行时也*仍然可以运行*。

eStargz 是基于 [Google CRFS 项目提出的 stargz 格式](https://github.com/google/crfs)开发的，具有内容验证和性能优化等实用功能。
有关 Lazy Pulling 和 eStargz 的更多信息，请参阅 [Stargz Snapshotter 项目仓库](https://github.com/containerd/stargz-snapshotter)。

### 配置 K3s 进行 eStargz 的 Lazy Pulling

如下所示，K3s Server 和 Agent 需要 `--snapshotter=stargz` 选项。

```bash
k3s server --snapshotter=stargz
```

使用此配置，你可以对 eStargz 格式的镜像进行 Lazy Pulling。
以下 Pod 清单示例使用 eStargz 格式的 `node:13.13.0` 镜像 (`ghcr.io/stargz-containers/node:13.13.0-esgz`)。
当启用 stargz snapshotter 时，K3s 会对该镜像进行 lazy pulling。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nodejs
spec:
  containers:
  - name: nodejs-estargz
    image: ghcr.io/stargz-containers/node:13.13.0-esgz
    command: ["node"]
    args:
    - -e
    - var http = require('http');
      http.createServer(function(req, res) {
        res.writeHead(200);
        res.end('Hello World!\n');
      }).listen(80);
    ports:
    - containerPort: 80
```

## 其他日志来源

你可以在不使用 Rancher 的情况下为 K3s 安装 [Rancher Logging](https://rancher.com/docs/rancher/v2.6/en/logging/helm-chart-options/)。为此，你可以执行以下命令：

```bash
helm repo add rancher-charts https://charts.rancher.io
helm repo update
helm install --create-namespace -n cattle-logging-system rancher-logging-crd rancher-charts/rancher-logging-crd
helm install --create-namespace -n cattle-logging-system rancher-logging --set additionalLoggingSources.k3s.enabled=true rancher-charts/rancher-logging
```

## 其他网络策略日志

支持记录网络策略丢弃的数据包。数据包被发送到 iptables NFLOG 操作，它显示了数据包的详细信息，包括阻止它的网络策略。

要将 NFLOG 转换为日志条目，请安装 ulogd2 并将 `[log1]` 配置为在 `group=100` 上读取。然后，重启 ulogd2 服务以提交新配置。

也可以使用 tcpdump 读取命中 NFLOG 操作的数据包：
```bash
tcpdump -ni nflog:100
```
但是请注意，在这种情况下，不会显示阻止数据包的网络策略。


当数据包被网络策略规则阻止时，日志消息将出现在 `/var/log/ulog/syslogemu.log` 中。如果流量很大，日志文件可能会增长得非常快。为了控制它，你可以向相关网络策略添加以下注释，从而设置 `limit` 和 `limit-burst` iptables 参数：
```bash
* kube-router.io/netpol-nflog-limit=<LIMIT-VALUE>
* kube-router.io.io/netpol-nflog-limit-burst=<LIMIT-BURST-VALUE>
```

默认值为 `limit=10/minute` 和 `limit-burst=10`。你可以查看 iptables 手册以进一步了解这些字段的格式和可选值。