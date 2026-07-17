# OpenEBS (Mayastor) 存储部署记录

OpenEBS 提供基于 Mayastor 引擎的 replicated PV（分布式块存储），作为集群的持久化存储后端。
当前部署在 2 节点的 K8s 集群（`node1` + `worker-1`），均为物理机。

## 当前状态（2026-07-15）

- OpenEBS Helm 安装完成，全部控制面 pod Running
- etcd / nats / loki 副本数已改为 2（适配 2 节点集群）
- NVMe 内核模块已加载
- io-engine 通过 `envcontext=iova-mode=pa` 解决 DPDK DMA mask 问题，两个节点均 Running
- 两个节点已创建 100G file-backed disk，路径 `/var/local/openebs/disks/disk1.img`
- DiskPool 已创建，POOL-STATUS 为 Online（使用 `aio:///` URI）
- 单副本和多副本 PVC 均验证通过

## Helm 安装

OpenEBS 通过 Helm chart 统一安装，底层依赖 Mayastor 子 chart。

```bash
helm repo add openebs https://openebs.github.io/openebs
helm repo update
```

### 安装命令

```bash
helm install openebs openebs/openebs --version 4.4.0 -n openebs --create-namespace \
  --set localpv-provisioner.enabled=false \
  --set lvm-localpv.enabled=false \
  --set ndm.enabled=false \
  --set zfs-localpv.enabled=false \
  --set engines.replicated.mayastor.enabled=true \
  --set mayastor.etcd.replicaCount=2 \
  --set mayastor.nats.cluster.replicas=2 \
  --set mayastor.loki.singleBinary.replicas=2 \
  --set mayastor.io_engine.envcontext=iova-mode=pa \
  --timeout 15m
```

关键参数说明：
- 只启用 Mayastor（replicated PV 引擎），禁用 localpv、LVM、ZFS、NDM
- etcd / nats / loki 副本数设为 **2**（匹配 2 节点集群）
- `mayastor.io_engine.envcontext=iova-mode=pa` — 物理机开启了 `intel_iommu=on`，DPDK EAL 需要用 IOVA=PA 模式才能初始化大页内存

## 副本数与反亲和性

Mayastor 依赖的 etcd、NATS、Loki 默认配置 **3 副本 + required podAntiAffinity**
（`topologyKey: kubernetes.io/hostname`），要求每个副本跑在不同节点上。
2 节点集群下第 3 个副本永远调度不了，导致集群无法初始化。
解决方式：安装时通过 `--set` 将副本数改为 2。

## 节点前置条件：NVMe 内核模块

Mayastor 使用 NVMe-over-TCP 协议，需要在**每个节点**上加载以下内核模块：

```bash
modprobe nvme-core
modprobe nvme-tcp
```

验证：

```bash
lsmod | grep nvme
```

预期输出：
```
nvme_tcp      86016   0
nvme_core    212992   2 nvme_tcp,nvme_fabrics
```

### 开机自动加载

写入 `/etc/modules-load.d/openebs-mayastor.conf`：

```
nvme-core
nvme-tcp
```

## 节点前置条件：DPDK IOVA 模式（针对 intel_iommu=on）

如果物理机的内核参数启用了 `intel_iommu=on`，DPDK 初始化大页内存时会遇到 DMA mask 限制：

```
EAL: alloc_pages_on_heap(): couldn't allocate memory due to IOVA exceeding limits of current DMA mask
EAL: Please try initializing EAL with --iova-mode=pa parameter
```

解决方式是通过 `mayastor.io_engine.envcontext=iova-mode=pa` 将 IOVA 模式改为
Physical Address，绕过 IOMMU DMA mask 限制。io-engine CLI 通过 `--env-context`
参数透传给 DPDK EAL。

检查当前内核参数：

```bash
cat /proc/cmdline | grep intel_iommu
```

## 创建 file-backed 磁盘文件

本集群没有空闲裸磁盘（`sda` 已全部分区给根文件系统），使用 Mayastor 支持的
file-backed disks。

> 官方文档链接：https://openebs.io/docs/4.2.x/user-guides/replicated-storage-user-guide/replicated-pv-mayastor/additional-information/tips
>
> Mayastor 支持 file-backed disks，使用 `aio:///` URI scheme，不需要 loop 设备。

### 创建稀疏文件（每个节点）

```bash
mkdir -p /var/local/openebs/disks
truncate -s 100G /var/local/openebs/disks/disk1.img
```

### 将目录挂载进 io-engine 容器

io-engine pod 默认没有映射此路径，需要补一个 hostPath volume：

```bash
kubectl patch daemonset -n openebs openebs-io-engine --type='json' -p='[
  {"op": "add", "path": "/spec/template/spec/volumes/-",
   "value": {"name": "disks", "hostPath": {"path": "/var/local/openebs/disks", "type": "DirectoryOrCreate"}}},
  {"op": "add", "path": "/spec/template/spec/containers/0/volumeMounts/-",
   "value": {"name": "disks", "mountPath": "/var/local/openebs/disks"}}
]'
```

重启 io-engine pods（DaemonSet 策略为 OnDelete，需手动触发）：

```bash
kubectl delete pod -n openebs -l app=io-engine
```

### 创建 DiskPool（使用 aio:///）

```yaml
apiVersion: openebs.io/v1beta3
kind: DiskPool
metadata:
  name: pool-node1
  namespace: openebs
spec:
  node: node1
  disks: ["aio:///var/local/openebs/disks/disk1.img"]
---
apiVersion: openebs.io/v1beta3
kind: DiskPool
metadata:
  name: pool-worker1
  namespace: openebs
spec:
  node: worker-1
  disks: ["aio:///var/local/openebs/disks/disk1.img"]
```

```bash
kubectl apply -f diskpool.yaml
```

验证：

```bash
kubectl get diskpool -n openebs
```

预期输出（POOL-STATUS 应为 Online）：

```
NAME           NODE       STATE     POOL-STATUS   CAPACITY   USED   AVAILABLE
pool-node1     node1      Created   Online         99.9 GiB   0 B    99.9 GiB
pool-worker1   worker-1   Created   Online         99.9 GiB   0 B    99.9 GiB
```

### 磁盘文件开机自创建（optional）

```bash
cat > /etc/systemd/system/openebs-file-disk.service << 'EOF'
[Unit]
Description=Setup OpenEBS Mayastor file-backed disk
Before=containerd.service kubelet.service
DefaultDependencies=no

[Service]
Type=oneshot
ExecStart=/bin/bash -c 'mkdir -p /var/local/openebs/disks && [ -f /var/local/openebs/disks/disk1.img ] || truncate -s 100G /var/local/openebs/disks/disk1.img'
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
systemctl enable --now openebs-file-disk.service
```

## 使用 Mayastor PVC

### 安装后自动创建一些存储类

安装完成后：
| StorageClass | Provisioner | BindingMode | 用途 | 来源 |
|---|---|---|---|---|
| openebs-single-replica | io.openebs.csi-mayastor | Immediate | 单副本 replicated PV | Helm 自动创建 |
| mayastor-etcd-localpv | openebs.io/local | WaitForFirstConsumer | etcd 内部存储 | Helm 自动创建 |
| openebs-hostpath | openebs.io/local | WaitForFirstConsumer | HostPath 本地卷 | Helm 自动创建 |
| openebs-loki-localpv | openebs.io/local | WaitForFirstConsumer | Loki 内部存储 | Helm 自动创建 |
| openebs-minio-localpv | openebs.io/local | WaitForFirstConsumer | MinIO 内部存储 | Helm 自动创建 |


### 单副本 PVC（只需 1 个 DiskPool）

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-mayastor-vol
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: openebs-single-replica
```

```bash
kubectl apply -f test-pvc.yaml
```

### 多副本同步复制（需要 2 个 DiskPool，不同节点）

创建自定义 StorageClass：

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: mayastor-replicated
parameters:
  protocol: nvmf
  ioTimeout: "30"
  replicaCount: "2"
provisioner: io.openebs.csi-mayastor
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

```bash
kubectl apply -f mayastor-replicated.yaml
```

> `replicaCount` 不能超过 DiskPool 数量。2 个节点最多 2 副本。

## 排障参考

### Pod 卡在 Init 容器

```bash
kubectl describe pod -n openebs <pod-name> | grep -A 10 'Init Containers'
```

常见等待链：
1. etcd 未就绪 → 所有带 `etcd-probe` init 容器的 pod 卡住
2. agent-core 未就绪 → 带 `agent-core-grpc-probe` init 容器的 pod 卡住
3. api-rest 未就绪 → csi-controller 卡住
4. nvme-tcp 模块未加载 → csi-node 卡在 `nvme-tcp-probe`

### io-engine DPDK 初始化失败

日志：

```
EAL: alloc_pages_on_heap(): couldn't allocate memory due to IOVA exceeding limits of current DMA mask
EAL: Please try initializing EAL with --iova-mode=pa parameter
FATAL: rte_service_init() failed
```

原因：内核参数 `intel_iommu=on` 导致 DPDK EAL 默认 IOVA 模式不兼容 DMA mask。

解决：helm install 时加 `--set mayastor.io_engine.envcontext=iova-mode=pa`。
如果已安装，修改 DaemonSet args 追加 `--env-context=--iova-mode=pa`。

### PVC Pending

```bash
kubectl describe pvc <pvc-name>
```

若看到 `waiting for pod to be scheduled`，则需排查 pod 调度失败的原因。
若看到 `Failed to create volume`，检查 DiskPool 状态和 io-engine 日志。

# OpenEBS 理论补充

## 组件说明

执行helm 安装后，集群中会部署以下组件：

### 控制面（Control Plane）

| Pod | 类型 | 职责 |
|---|---|---|
| `openebs-agent-core` | Deployment (1) | OpenEBS 控制面核心。处理 PV 的生命周期管理（创建/删除/扩容），维护卷与副本的映射关系，管理副本在 DiskPool 上的调度。所有控制操作最终的协调者。 |
| `openebs-api-rest` | Deployment (1) | REST API 网关。提供 HTTP API 供 kubectl-mayastor 插件和内部组件调用，将 REST 请求转给 agent-core 的 gRPC 接口。 |
| `openebs-operator-diskpool` | Deployment (1) | DiskPool Operator。监听 DiskPool CR，执行池的创建、导入、状态上报，将 K8s 资源状态同步到 mayastor 内部。 |

### 数据面（Data Plane）

| Pod | 类型 | 职责 |
|---|---|---|
| `openebs-io-engine` | DaemonSet (每节点 1) | **核心数据引擎**。管理本地 DiskPool（磁盘分配、空间管理），通过 NVMe-over-TCP 协议接收远程读写请求，执行实际的 I/O 操作。同时负责副本间同步（replica mirroring）。使用 DPDK 驱动网卡和磁盘以获得高性能。当前 2 节点各跑一个。 |

### CSI 驱动

| Pod | 类型 | 容器数 | 职责 |
|---|---|---|---|
| `openebs-csi-controller` | Deployment (1) | 6 | CSI 控制面容器组，包含：`csi-provisioner`（监听 PVC 创建，调用 agent-core 建卷）、`csi-attacher`（处理 VolumeAttachment）、`csi-resizer`（处理扩容）、`csi-snapshotter`（处理快照）、`livenessprobe` 和 `csi-driver`。 |
| `openebs-csi-node` | DaemonSet (每节点 1) | 2 | CSI 节点驱动。运行在集群每个节点上，负责在本节点执行卷挂载/卸载操作，将 K8s 的 CSI 调用转换为 nvme-tcp 连接操作。 |

### 基础设施组件

| Pod | 类型 | 职责 |
|---|---|---|
| `openebs-etcd-*` | StatefulSet (2 副本) | 存储 mayastor 内部状态：卷拓扑、副本位置、节点状态等。控制面组件依赖 etcd 做分布式协调。**etcd 未就绪时，所有依赖它的组件会卡在 init 容器等待。** |
| `openebs-nats-*` | StatefulSet (2 副本) | 控制面消息总线。agent-core 与 agent-ha-node 之间通过 NATS 通信。 |
| `openebs-loki-*` | StatefulSet (2/3 副本) | 日志聚合。收集 all openebs 组件的日志，供排障查询。 |
| `openebs-alloy-*` | DaemonSet (每节点 1) | 日志采集器。从节点拉取容器日志发送到 Loki。 |
| `openebs-minio-*` | StatefulSet (3 副本) | Loki 的日志存储后端。Loki 将日志数据持久化到 MinIO。 |

### 其他引擎组件

以下 Pod 是 OpenEBS 统一安装时附带的其他引擎组件，本集群未使用对应功能，可忽略：

| Pod | 说明 |
|---|---|
| `openebs-localpv-provisioner` | HostPath LocalPV 动态 provisioner（本集群未使用） |
| `openebs-lvm-localpv-*` | LVM LocalPV 引擎（本集群未使用） |
| `openebs-zfs-localpv-*` | ZFS LocalPV 引擎（本集群未使用） |

### 集群间高可用组件

| Pod | 类型 | 职责 |
|---|---|---|
| `openebs-agent-ha-node-*` | DaemonSet (每节点 1) | 运行在每个节点上，负责监控本节点 io-engine 的健康状态，通过 NATS 向 agent-core 上报。当节点故障时参与故障切换决策。 |

## 应用使用 Mayastor 的完整交互流程

以创建一个 2 副本 PV 并启动 Pod 为例，分两阶段：

### 阶段一：PVC 创建 → PV 就绪

```
User → kubectl apply -f pvc.yaml
  │
  ▼
1. K8s 控制面
   └─ 发现 PVC 的 StorageClass(自定义的replicated)
   └─ StorageClass 用了 io.openebs.csi-mayastor 作为rpovisioner
   └─ 调用 CSI Provisioner（在 csi-controller 中）
      │
      ▼
2. csi-provisioner
   └─ 调用 agent-core 的 gRPC 接口
      │
      ▼
3. agent-core
   ├─ 查询 etcd — 当前有哪些健康 DiskPool
   ├─ 选择 2 个不同节点的 DiskPool（pool-node1 + pool-worker1）
   ├─ 调度 2 个副本：各分配一块空间
   ├─ 通知 io-engine 创建 Nexus（NVMe 控制器）
   └─ 返回 Nexus 的 NVMe-over-TCP 连接信息
      │
      ▼
4. csi-provisioner
   └─ 创建 PV 对象，存储 Nexus 连接信息（NQN + 目标 IP）
      │
      ▼
PV 状态变为 Available / Bound
```

### 阶段二：Pod 调度 → 应用 IO

```
User → kubectl run pod
  │
  ▼
1. Kubelet 调度 Pod 到某节点（假设 node3）
   └─ 发现 Pod 需要挂载 Mayastor PV
   └─ 调用本节点的 CSI Node Driver（csi-node）
      │
      ▼
2. csi-node（在 node3 上）
   ├─ StageVolume：
   │   └─ 解析 PV 中的 Nexus NQN 地址
   │   └─ 调用内核 nvme-tcp 模块
   │   └─ nvme connect -t tcp -n <NQN> -a <io-engine IP> -s 8420
   │   └─ 内核创建 NVMe 设备（如 /dev/nvme0n1）
   │
   ├─ PublishVolume：
   │   └─ 格式化（首次），如 ext4/xfs
   │   └─ mount 到 Pod 的挂载点
   │
   ▼
3. App Pod 开始读写
   │
   ├─ 读请求 → ext4/xfs → /dev/nvme0n1
   │           → nvme-tcp 驱动 → 网络 → io-engine on node1
   │           → 读取副本1 → 返回
   │
   └─ 写请求 → ext4/xfs → /dev/nvme0n1
               → nvme-tcp 驱动 → 网络 → io-engine on node1
               → 写入副本1 → 同步到副本2（worker-1 → io-engine）
               → 两副本都确认 → 返回写完成
```

### 架构总结

```
┌─────────────────────────────────────────────────────────────┐
│                    控制面（agent-core / api-rest）              │
│                    元数据存 etcd                              │
│                    消息总线 NATS                              │
└────────┬───────────────────────────┬─────────────────────────┘
         │                           │
         ▼                           ▼
┌─────────────────┐     ┌──────────────────────┐
│ CSI Controller    │     │   K8s API Server       │
│ (csi-controller)  │     │                        │
│ 创建 PV / 扩容/快照│     │  pvc / pv / va 等资源  │
└─────────────────┘     └──────────────────────┘
         │
         ▼
┌──────────────────────────────────────────────┐
│            每节点 CSI Node（csi-node）           │
│              ┌─────────────────┐              │
│              │ nvme-tcp 客户端   │              │
│              │ mount + 格式化    │              │
│              └────────┬────────┘              │
└───────────────────────┼──────────────────────┘
                        │ NVMe-over-TCP (网络)
                        ▼
┌──────────────────────────────────────────────┐
│       数据面 io-engine（每个 DiskPool 节点）     │
│                                              │
│  node1.io-engine            worker-1.io-engine│
│  ┌────────────────┐        ┌────────────────┐ │
│  │ pool-node1      │        │ pool-worker1    │ │
│  │ replica A       │  ◄──── ▶ replica B       │ │
│  │ /var/local/...   │        │ /var/local/...   │ │
│  │   /disk1.img    │        │   /disk1.img    │ │
│  └────────────────┘        └────────────────┘ │
└──────────────────────────────────────────────┘
```

### 关键特性

- **NVMe-over-TCP 直连** — App 不经过 CSI Node 代理 I/O，csi-node 只负责挂载阶段（connect + mount），之后 App 进程直接通过内核 nvme-tcp 驱动与 io-engine 通信，数据路径无额外跳转
- **副本同步发生在 io-engine 之间** — 写入时 io-engine 写入本地副本后直接同步到另一个 io-engine，不经过 Nexus 代理
- **故障切换** — 副本 io-engine 宕机时，agent-core 检测到并切换 Nexus 到存活副本，App 端 nvme-tcp 重连后继续 IO（受 ioTimeout 控制）


## DiskPool

### 什么是 DiskPool

DiskPool 是 Mayastor 副本存储的最底层资源单元。每个 DiskPool 绑定到一个物理块设备（或 AIO 文件），在该设备上初始化 SPDK blobstore，为上层 volume replica 提供持久化空间。

**核心事实：目前（Mayastor 2.10）每个 DiskPool 只能有一块盘。**

官方文档明确说明：*"The current version of Replicated PV Mayastor supports only one disk device per pool."*

`spec.disks` 虽然是数组，但只能填一个磁盘 URI。

### 单盘架构的含义

```
DiskPool pool-node1 (node1)
  └── 磁盘: /dev/disk/by-id/<某个设备> 或 aio:///path/to/file
       └── SPDK blobstore
            ├── replica-A (volume-xxx)
            ├── replica-B (volume-yyy)
            └── replica-C ... (共享同一块盘的空间)
```

- 一个池的磁盘坏了，这个池上的所有 replica 都丢了
- 数据冗余靠跨池副本（volume 配置 `replicaCount: 2`，两个 replica 分别放在不同节点的不同池上）
- 不存在把两个池合并成一个大逻辑池的机制（不像 LVM VG 或 Ceph OSD Pool）
- volume 的总容量受限于单个池的空闲空间（一个 replica 在哪个池上就占那个池的空间）

### 多个 DiskPool 的关系

同一个 openEBS 实例下的多个 DiskPool **被同一个控制面（agent-core）统一管理**，但不汇聚存储空间：

```
agent-core（控制面）
  ├── 知道 pool-node1 (node1, 99.9 GiB)
  └── 知道 pool-worker1 (worker-1, 99.9 GiB)

创建 volume 时（replicaCount: 2）：
  └── 调度 replica-0 → pool-node1
  └── 调度 replica-1 → pool-worker1
```

控制面负责在创建 volume 时选择合适的池（基于拓扑、剩余空间等），但池与池之间是独立的故障域。

### 支持的磁盘 URI 格式

| URI 格式 | 说明 | 典型场景 |
|---|---|---|
| `aio:///dev/disk/by-id/xxx` | 物理块设备，Linux AIO 方式 | 裸盘、分区 |
| `aio:///var/local/openebs/disks/disk1.img` | 文件模拟的块设备，AIO 方式 | 无空闲裸盘时使用文件替代 |
| `uring:///dev/disk/by-id/xxx` | 物理块设备，io_uring 方式 | 需要更低延迟的高性能场景 |

本集群的 DiskPool 使用 `aio:///` 文件后端。

### DiskPool 不支持自动发现

Mayastor 架构中没有 NDM（Node Disk Manager）或类似的自动扫描组件。新插入磁盘的流程是**手动声明式的**：

```
插新盘 → 节点上 lsblk 确认盘符 → 获取 /dev/disk/by-id/ 稳定路径
→ 手动创建 DiskPool CR（kubectl apply）→ operator 通知 io-engine 接管
```

openEBS 4.x 的 Helm values 中 `ndm.enabled: false`（本集群也是如此）。旧版（openEBS 3.x 及之前）cStor/Jiva 时代依赖 NDM 做磁盘发现，Mayastor 弃用了这个模式——因为 SPDK 需要从用户态直接接管块设备，不允许操作系统内核先看到它。

要查看一个节点上哪些块设备对 io-engine 可见，需在 io-engine 内部查询：
```
kubectl mayastor get block-devices <node>
```

### DiskPool 扩容

若底层磁盘扩容（如文件变大、LVM 卷扩容、虚机盘扩容），DiskPool 可以跟随扩容，但有前提条件：

- **必须在创建时设置 `spec.maxExpansion`**，例如 `"10x"` 表示允许扩容到原始大小的 10 倍
- 创建时未设 `maxExpansion`（值为 `null`），默认等于 `1x`，即不允许扩容
- 扩容是**一次性操作**：通过 annotation `openebs.io/expand=true` 触发，完成后不可回退

本集群的两个池的 `maxExpansion` 均为 `null`，当前不支持扩容。如需扩容能力，需重新创建带 `maxExpansion` 的新池并将数据迁移过去。

### DiskPool 运维操作一览

| 操作 | 命令/方式 | 影响 |
|---|---|---|
| 创建新池 | `kubectl apply -f diskpool.yaml` | 已有池无影响 |
| 查看池列表 | `kubectl get dsp -n openebs` | - |
| 查看池详情 | `kubectl get dsp <name> -n openebs -o yaml` | - |
| 删除池（非空） | 被 `openebs.io/diskpool-protection` finalizer 阻止 | 需先迁移/删除 replica |
| 删除池（空池） | `kubectl delete dsp <name> -n openebs` | 数据面释放设备 |
| 扩容池 | annotation `openebs.io/expand=true` | 无中断，需创建时设过 `maxExpansion` |
| cordon/uncordon | kubectl-mayastor 插件或 REST API | 阻止/允许新 replica 调度 |

### DiskPool 故障行为

DiskPool 损坏时，Mayastor **不会**自动在另一个池上重建 replica 来补回副本数。这是有意为之的设计：故障域取决于用户对拓扑的选择，全量重建对 I/O 和带宽有明显影响，不适合自动触发。

实际行为按故障性质区分：

| 故障类型 | 表现 | 恢复方式 |
|---|---|---|
| 临时故障（重启/网络闪断/磁盘短暂离线） | replica 标记为 Faulted；Nexus 切换至其他健康 replica 继续服务；volume 进入 Degraded 状态 | 磁盘/节点恢复后，replica 自动重新同步至 Nexus，恢复完整副本数 |
| 永久故障（磁盘物理损坏/节点彻底失联） | replica 永久丢失；volume 停留在 Degraded 状态，剩余 replica 继续提供 I/O | 手动操作：新建 DiskPool → 删除旧 volume 重建，或等官方未来支持 replica 迁移后做替换 |

与 Ceph 的自动回填不同，Mayastor 不维护分布式哈希层的 PG 分布拓扑，因此重建 replica 无法由系统自主决策。多数生产场景中，2 副本配置下任一池损坏后虽然数据仍在（剩余副本），但冗余降为零——再坏一个池即数据丢失。

### DiskPool 注意事项

- 池创建后不能更换磁盘（会破坏 blobstore 元数据）
- 本集群的 disk1.img 是 file-backed，删除池后文件还在，可以重新 import
- 3 副本需要至少 3 个节点各有一个 DiskPool（你的集群 2 个节点，最多 2 副本）

# NFS(Network File System) 增强 openEBS Mayastor 的 RWX 能力

## 为什么要 NFS？

**Mayastor 的 PV 本身是 RWO（ReadWriteOnce）**，同一时刻只能被一个 Pod 挂载。这是因为 NVMe-over-TCP 协议是一对一的连接，不支持多个客户端同时挂载同一块卷。

但有些场景需要多个 Pod 同时读写同一份数据：

```
Deployment replicas: 3
  ├─ Pod-A → 同时读写同一份数据
  ├─ Pod-B → 同时读写同一份数据
  └─ Pod-C → 同时读写同一份数据
```

NFS 的引入就是为了解决这个问题：一个 NFS Server Pod 挂载 Mayastor 的 RWO PV，然后将它通过 NFSv4 协议共享出去。多个 App Pod 通过 NFS 协议连接这个 Server 来实现并发访问。

**不需要 NFS 的场景**（直接使用 `mayastor-replicated`）：
- 每个 Pod 自己独立的数据（StatefulSet + volumeClaimTemplates）
- 数据库多实例通过自身复制协议同步（MySQL Group Replication、ES 等）
- 不需要持久化的纯无状态服务

## 部署NFS

部署顺序：**先部署 NFS Server Pod，再安装 NFS CSI Driver（创建 StorageClass）**。

因为 StorageClass 的 `server` 参数填的是 NFS Server 的 Service DNS 名（`nfs-server.nfs-server.svc.cluster.local`），虽然 SC 创建时不会立即检查连通性，但后续 PVC 挂载时必须能解析到这个地址。NFS Server 不存在时挂载会失败。

### 第一步：节点前置条件：nfs-common

所有可能运行使用 `nfs-csi` PVC 的 Pod 的节点需安装 NFS 客户端（NFS CSI Driver 的节点 DaemonSet 需要 `mount.nfs` 命令）：

```bash
apt-get install -y nfs-common
```

### 第二步：部署 NFS Server

NFS Server 是一个普通的 Deployment，挂载 Mayastor 的 RWO PV 后通过 NFS 协议对外共享。

1. 为 NFS Server 创建后端 PVC

    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: nfs-server-claim
      namespace: nfs-server
    spec:
      storageClassName: mayastor-replicated
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 10Gi
    ```

2. 创建 NFS Server Deployment 和 Service

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nfs-server
      namespace: nfs-server
    spec:
      replicas: 1
      selector:
        matchLabels:
          role: nfs-server
      template:
        metadata:
          labels:
            role: nfs-server
        spec:
          volumes:
            - name: nfs-vol
              persistentVolumeClaim:
                claimName: nfs-server-claim
          containers:
            - name: nfs-server
              image: itsthenetwork/nfs-server-alpine
              env:
                - name: SHARED_DIRECTORY
                  value: /nfsshare
              ports:
                - name: nfs
                  containerPort: 2049
              securityContext:
                privileged: true
              volumeMounts:
                - mountPath: /nfsshare
                  name: nfs-vol
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: nfs-server
      namespace: nfs-server
    spec:
      ports:
        - name: nfs
          port: 2049
      selector:
        role: nfs-server
    ```

    NFS Server 在集群内通过 `nfs-server.nfs-server.svc.cluster.local` 访问。

### 第三步：安装 NFS CSI Driver（创建 StorageClass）

通过 Helm 安装 `csi-driver-nfs`，同时创建 `nfs-csi` StorageClass：

```bash
helm repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
helm repo update
helm install csi-nfs csi-driver-nfs/csi-driver-nfs --namespace csi-nfs --create-namespace \
  --set storageClass.create=true \
  --set storageClass.name=nfs-csi \
  --set storageClass.parameters.server=nfs-server.nfs-server.svc.cluster.local \
  --set 'storageClass.parameters.share=/' \
  --set 'storageClass.mountOptions[0]=nfsvers=4.1'
```

参数说明：

| 参数 | 值 | 含义 |
|---|---|---|
| `storageClass.create=true` | true | 安装时自动创建一个 StorageClass |
| `storageClass.name=nfs-csi` | nfs-csi | StorageClass 的名称，应用 PVC 通过 `storageClassName: nfs-csi` 引用 |
| `storageClass.parameters.server` | `nfs-server.nfs-server.svc.cluster.local` | NFS Server 的 Service 地址。**必须先部署 NFS Server，这个 DNS 才能解析** |
| `storageClass.parameters.share=/` | `/` | NFS Server 上的共享目录路径。对应 NFS Server Pod 的 `SHARED_DIRECTORY=/nfsshare` |
| `storageClass.mountOptions[0]=nfsvers=4.1` | nfsvers=4.1 | NFS 挂载协议版本。4.1 支持 pNFS 等特性 |

安装后集群中会增加一个 `nfs-csi` StorageClass。之后应用创建 RWX PVC 时指定 `storageClassName: nfs-csi`，CSI Driver 会自动在 NFS Server 的 `/nfsshare` 下为每个 PVC 创建独立子目录。

### 第四步：使用 RWX PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-shared-volume
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: nfs-csi
```

多个 Pod 可同时挂载此 PVC 进行读写。 这里通过 nfs-csi 创建的PVC声明的容量，是请求 NFS SERVER POD 为其创建一个子目录，声明该子目录的容量。

#### 特别注意：NFS PVC 声明的容量没有约束力

`nfs-csi` 属于**协议型存储**（类似 NFS/CIFS），和块存储不同。

使用 `nfs-csi` 声明 PVC 时填写的 `storage: 10Gi` **不会给子目录施加任何磁盘配额**。
K8s 和 NFS 都不会阻止 APP 写入超过声明大小的数据。

对比一下就清楚了：

| 存储类型 | 声明 10Gi 的实际行为 |
|----------|---------------------|
| `mayastor-replicated`（RWO 块设备） | 在 DiskPool 中分配 10Gi，最多只能写 10Gi |
| **`nfs-csi`**（RWX 协议挂载） | 只是在 NFS Server 上 `mkdir` 一个子目录，**可以写超，无任何阻拦** |

声明的这个值只对 K8s 本身有意义：

- `kubectl get pv` 能显示容量
- `ResourceQuota` 可以用它限制 namespace 总声明量
- 调度器用它做基础的容量估算

**但不管后端实际还剩多少空间。**

#### 真正受限的是 NFS Server 后端 PV

```
NFS Server 后端 PV: 10Gi (mayastor-replicated, 2 副本)
         │
         ├── app-A-pvc    声明 1Gi     ← 无配额，可以写到超过 1Gi
         ├── app-B-pvc    声明 10Gi    ← 无配额
         └── app-C-pvc    声明 5Gi     ← 无配额
         ─────────────────────────────────
K8s 记录总和: 16Gi  >  后端实际 10Gi    ← K8s 不会阻止超卖
```

所有子目录共享 NFS Server 后端 PV 的 10Gi 空间。**任何一个 APP 写满后端，所有 APP 一起报磁盘满。**

## APP-NFS-OPENEBS 架构

```
┌────────────────────────────────────────────────────────────────┐
│                     分层可靠性模型                                │
│                                                                │
│  上层：应用高可用                                                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ App Deployment replicas: 3                              │   │
│  │ ├─ Pod-A (node1)  ──┐                                  │   │
│  │ ├─ Pod-B (worker-1) ├─ 都挂载 nfs-csi PVC（RWX）        │   │
│  │ └─ Pod-C (node1)  ──┘                                  │   │
│  │ 特性：任一 Pod 宕机不影响服务，剩 2 个继续服务              │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  中间层：文件访问共享                                           │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ NFS CSI Driver (nfs.csi.k8s.io)                        │   │
│  │   └─ 在 NFS Server 上为每个 PVC 创建子目录               │   │
│  │                                                         │   │
│  │ NFS Server Pod (1 副本, 任意节点)                        │   │
│  │   └─ 通过 nvme-tcp 挂载后端 Mayastor PV                 │   │
│  │   └─ 不要求与数据同节点，通过网络连接 io-engine            │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  底层：数据可靠性                                               │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ 后端 PV: mayastor-replicated (2 副本)                   │   │
│  │   ├─ pool-node1 (node1)  ── 文件数据                     │   │
│  │   └─ pool-worker1 (worker-1) ── 同步镜像                  │   │
│  │  特性：任一节点磁盘坏了，另一份副本完好                     │   │
│  └─────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────┘
```

NFS Server 与数据的位置关系：

```
NFS Server Pod 可以跑在任意节点（比如 node1, worker-1, 或未来的 node3）
  └─ CSI Node 驱动在 Pod 所在节点执行 nvme-tcp connect
  └─ 通过内核 nvme-tcp 驱动 → 网络 → io-engine（持有副本的任何节点）
  └─ NFS Server 与数据不需要在同一节点
```

## NFS CSI Driver 的工作原理

NFS CSI Driver 和后端的 NFS Server 配合，为每个应用 PVC 分配独立的子目录，而不是每个应用创建独立的 NFS Server 实例：

```
NFS Server 共享目录 /nfsshare
  ├── pvc-xxxx-xxxx-xxxx-xxxx-a/  ← App-A 的 RWX 卷内容
  │     ├── file1
  │     └── ...
  ├── pvc-yyyy-yyyy-yyyy-yyyy-b/  ← App-B 的 RWX 卷内容
  └── pvc-zzzz-zzzz-zzzz-zzzz-c/  ← App-C 的 RWX 卷内容
```

所以多个应用共用同一个 NFS Server 后端 PV 的存储空间。当 RWX 需求增大时，要么扩容后端 PV，要么创建独立的 NFS Server 组来做隔离。


## NFS 扩容方式

`mayastor-replicated` 支持在线扩容，但 NFS Server 的文件系统需要额外处理：

```bash
# 1. 扩容 PVC
kubectl edit pvc -n nfs-server nfs-server-claim
# 把 storage: 10Gi 改成 20Gi

# 2. NFS Server Pod 重启后，内部文件系统需要扩展
kubectl rollout restart deployment -n nfs-server nfs-server
# 进入 Pod 执行 resize2fs / xfs_growfs（取决于文件系统类型）
```

### 无法通过增加磁盘为 NFS Server 扩容

因为 NSF SERVER POD使用的PVC是通过 mayastor-replicated创建的，其要求PV必须存放于单个磁盘上，不能跨多个磁盘聚合空间，所以无法通过增加磁盘来扩容 NFS Server 的 PV。
