# Longhorn — 分布式块存储

## 概述

Longhorn 是 CNCF 毕业项目，轻量级 K8s 原生分布式块存储。使用节点磁盘组成存储池，通过 CSI 驱动提供 PV。
相比 Mayastor + 自建 NFS Server，一个 Longhorn 就能同时提供 replicated 块设备 + 原生 RWX，且支持在线扩容。

**核心特性**

- 支持裸盘、分区、目录三种磁盘类型，新增磁盘后自动加入存储池
- 内置 RWX 支持：自动为每个 RWX PVC 创建 Share Manager（NFSv4 导出），无需手动管理 NFS Server
- Volume 在线扩容：扩 PVC 即可，底层自动处理
- 快照、备份、恢复原生支持
- Web UI 管理

---

## 组件详解

### Pod / Deployment

```
NAME                                   类型        节点分布        说明
longhorn-manager-xxxxx                 DaemonSet   每节点 1 个     管理大脑，监听 CR 变化、注册磁盘、调度卷、响应 WebUI
longhorn-csi-plugin-xxxxx              DaemonSet   每节点 1 个     CSI 驱动，处理 kubelet 的挂载/创建请求
engine-image-ei-xxxxx-xxxxx            DaemonSet   每节点 1 个     引擎镜像预热，确保节点上有 longhorn-engine 镜像
instance-manager-xxxxxxxx-xxxxxxxx     Pod         每节点 1 个     实际处理卷 I/O，管理 replica 文件读写
share-manager-pvc-xxxxxxxx-xxxx        Pod         按 RWX 卷创建   NFS 导出端，把 Longhorn 卷通过 NFSv4 暴露给多个 Pod
longhorn-ui-xxxxx                      Deployment  2 副本          Web 管理界面
longhorn-driver-deployer-xxxxx         Deployment  1 副本          部署 CSI 组件
csi-attacher-xxxxx                     Deployment  3 副本          K8s 内置 sidecar，处理 VolumeAttachment
csi-provisioner-xxxxx                  Deployment  3 副本          K8s 内置 sidecar，调用 CSI 驱动创建/删除卷
csi-resizer-xxxxx                      Deployment  3 副本          K8s 内置 sidecar，处理卷扩容
csi-snapshotter-xxxxx                  Deployment  3 副本          K8s 内置 sidecar，处理卷快照
```

**DaemonSet vs Deployment 的选择逻辑**

判断一个组件用 DaemonSet 还是 Deployment，核心看它是否需要访问节点本地资源：

| 需要访问节点本地资源的 -> DaemonSet | 不需要的 -> Deployment |
|---|---|

**DaemonSet 的原因**：
- **longhorn-manager**：操作本机磁盘（读写下 /var/lib/longhorn/、检测裸盘），管理本机的 instance-manager 进程，无法跨节点工作
- **longhorn-csi-plugin**：kubelet 只在本地节点调用 CSI 驱动挂载卷，每节点必须有一个 CSI Pod 等在那里
- **engine-image-ei-xxx**：预热引擎镜像到每个节点的容器运行时，确保 instance-manager 启动时有镜像可用

**Deployment 的原因**：
- **longhorn-ui**：纯前端，不接触节点资源，只通过 API 与 longhorn-manager 通信。做成 Deployment 可以任意调度、按需扩缩容、滚动更新
- **csi-provisioner / csi-attacher / csi-resizer / csi-snapshotter**：处理集群级别的操作（创建/删除/扩容/快照），与具体节点无关，多副本只是为了高可用
- **longhorn-driver-deployer**：一次性部署任务，负责把 CSI 组件部署到集群中，部署完就稳定运行

### Service

```
NAMESPACE        NAME                    类型        说明
longhorn-system  longhorn-frontend       ClusterIP   Longhorn Web UI 入口，暴露 80 端口
longhorn-system  longhorn-engine-manager ClusterIP   管理 instance-manager 的 gRPC 端点
longhorn-system  longhorn-csi            ClusterIP   CSI 驱动的 gRPC 端点（内部使用）
```

### StorageClass

| SC 名称 | Provisioner | 默认 | 访问模式 | 说明 |
|---|---|---|---|---|
| `longhorn` | `driver.longhorn.io` | 是 | RWO | 自动创建的默认 SC，2 副本 |
| `longhorn-rwx` | `driver.longhorn.io` | 否 | RWX | 手动创建，2 副本，支持多 Pod 共享读写 |
| `longhorn-static` | `driver.longhorn.io` | 否 | RWO | 用于已存在的 Longhorn 卷导入 |

### 关键 CRD

Longhorn 通过自定义资源（CRD）管理所有状态，核心的有：

| CRD | 作用 | 查看命令 |
|---|---|---|
| `nodes.longhorn.io` | 每个 Longhorn 节点。记录节点上的磁盘列表、调度状态、资源使用 | `kubectl -n longhorn-system get nodes.longhorn.io` |
| `volumes.longhorn.io` | 每个 Longhorn 卷。记录卷大小、副本数、健康状态、所在节点 | `kubectl -n longhorn-system get volumes.longhorn.io` |
| `replicas.longhorn.io` | 每个副本。记录副本在哪个节点的哪个磁盘上、是否健康 | `kubectl -n longhorn-system get replicas.longhorn.io` |
| `engineimages.longhorn.io` | 卷引擎镜像版本管理 | `kubectl -n longhorn-system get engineimages.longhorn.io` |
| `instancemanagers.longhorn.io` | instance-manager 实例 | `kubectl -n longhorn-system get instancemanagers.longhorn.io` |
| `sharemanagers.longhorn.io` | Share Manager（RWX 导出实例） | `kubectl -n longhorn-system get sharemanagers.longhorn.io` |
| `backups.longhorn.io` / `backupvolumes.longhorn.io` | 备份相关 | `kubectl -n longhorn-system get backups.longhorn.io` |

修改 `nodes.longhorn.io` 的 `spec.disks` 就是添加/移除磁盘的入口。

---

## 核心流程

### 体系架构：三层解耦

Longhorn 的 RWX 由三个独立层组成，**每层可以独立调度到不同节点上**：

```
用户层           App Pod A (node2)         App Pod B (node3)
                    \                        /
                     \______ NFSv4 _________/
                            (ClusterIP)
                            |
导出层            Share Manager Pod (可在任意节点)
                    | 内含 NFSd，挂载 Longhorn block volume
                    | 宕机后自动在其他节点重建
                    |
                    | iSCSI（跨节点网络访问）
                    |
存储层         replica 1 (node1 磁盘)    replica 2 (worker-1 磁盘)
```


**关键性质：三层不绑定。**
- Share Manager 不需要和 replica 在同一节点，它通过 iSCSI 跨节点访问
- App Pod 不需要和 Share Manager 在同一节点，它通过 NFS ClusterIP 跨节点访问
- 即使 Share Manager 所在节点宕机，Longhorn 会在另一个节点重建它，挂载同一个 volume 继续导出
- **2 节点限制**：replica 必须 ≤ 节点数（我们设 2），Share Manager 和 App Pod 可以跑在任一节点上

**注意**，Share Manager只是挂载了Longhorn卷，并通过NFSv4导出数据，它不是利用Longhorn来提供自己的存储和SC，这和Mayastor + NFS方案不同。

```
  用户创建 PVC (RWX, longhorn-rwx)
          │
          ▼
  Longhorn 卷 (block device, 2 replicas)
          │
          ▼
  Share Manager Pod ─── 内部挂载这个卷 (RWO 方式)
     (内含 NFSv4 Server)    │
                            │  通过 NFSv4 导出同一份数据
                            ▼
                    用户 Pod A (node1)
                    用户 Pod B (worker-1)
                    用户 Pod C (node1)

```

### 创建 RWX PVC 的完整流程

```
1. kubectl apply -f pvc.yaml
        │
        ▼
2. K8s API Server 收到 PVC 创建请求
        │  PVC 声明了 storageClassName: longhorn-rwx
        │  K8s 找到 Provisioner: driver.longhorn.io
        ▼
3. csi-provisioner（K8s 内置组件，跑在 Pod 里）
        │  通过 CSI gRPC 调用 Longhorn CSI 驱动的 CreateVolume()
        ▼
4. Longhorn CSI 驱动
        │  调用 longhorn-manager 的 API 请求创建卷
        ▼
5. longhorn-manager
        │  a. 检查当前磁盘池（nodes.longhorn.io），找出有足够空间的磁盘
        │  b. 根据 numberOfReplicas=2，挑选 2 个节点
        │  c. 创建 replicas.longhorn.io 资源（2 个 replica 对象）
        │  d. 通知对应节点上的 instance-manager 开始构建 .img 文件
        ▼
6. instance-manager（在目标节点上）
        │  在对应磁盘上创建 volume-head-000.img 稀疏文件（大小 = PVC容量）
        │  初始化 replica metadata，准备接收 I/O
        ▼
7. K8s 标记 PVC 为 Bound
        │  PV 创建完成，状态 = Available → Bound
        ▼
8. Pod 调度并挂载时：
        │  kubelet → csi-node（longhorn-csi-plugin 内）→ NodeStageVolume / NodePublishVolume
        │  卷通过 iSCSI 协议挂载到节点，再 bind mount 到 Pod
        ▼
9. 对于 RWX 卷（步骤 5 之后多一步）：
        longhorn-manager 创建 sharemanagers.longhorn.io CR
        → 在节点上拉起 share-manager-pvc-xxx Pod
        → Share Manager 把 Longhorn 卷通过 NFSv4 导出
        → 所有使用该 PVC 的 Pod 挂载 NFS 共享
```

### 修改 CR 时，longhorn-manager 如何响应

```
kubectl edit node.longhorn.io <node> -n longhorn-system
        │  修改 spec.disks，例如新增 /dev/sdb
        ▼
longhorn-manager（节点上的 Pod）
        │  Informer 机制监听到 nodes.longhorn.io 变化
        ▼
      ┌─ 验证：路径存在？读写权限？是否已被占用？
      │  如果是裸盘：检查是否有分区表/文件系统
      │  如果是目录：检查是否可写
      ▼
      验证通过后：
      │  在磁盘上初始化 Longhorn metadata
      │  （目录盘：创建 longhorn-disk.cfg / metadata 目录）
      │  （裸盘：写入 Longhorn 标识到设备头部）
      ▼
      更新 nodes.longhorn.io status
      │  status.diskStatus.磁盘路径.conditions → Ready
      │  storageAvailable / storageMaximum 更新
      ▼
      Longhorn 调度器现在可以在这个磁盘上安排 replica
```

这个过程**完全在线**，已有卷的 replica 不受影响。新增的磁盘只参与后续新卷的调度，或者当你扩展现有卷容量时才会被使用。

---

## 磁盘管理

### 支持的磁盘类型

| 类型 | diskType 值 | 路径示例 | 说明 |
|---|---|---|---|
| 目录（文件系统） | `filesystem` | `/var/lib/longhorn/` | Longhorn 在目录里创建稀疏文件（`.img`）模拟块设备，适合利用空闲空间 |
| 裸块设备 | `block` | `/dev/sdb` 或 `/dev/nvme1n1` | Longhorn 直接接管整块磁盘，不依赖文件系统，性能更好 |

`filesystem` 和 `block` 可以混用，同一个卷的 replica 可以一个在目录盘上、一个在裸盘上，互相不受影响。

### 磁盘管理底层原理

Longhorn 在每个节点上跑一个 `longhorn-manager`，它做的事情就像一个轻量级的 LVM + iSCSI target：

```
裸盘 /dev/sdb ──┐
                ├──→ Longhorn 磁盘池 ──→ 副本 (replica)
目录 /var/lib/longhorn/ ──┘                     │
                                            ┌────┴────┐
                                            │  卷 A   │
                                            │ replica │
                                            │ (1G .img)│
                                            └─────────┘
```

- 对 **裸盘**：Longhorn 直接在上面创建稀疏文件作为 volume 数据
- 对 **目录**：Longhorn 在目录里创建稀疏文件（`volume-head-000.img`），这个文件会按需增长，最大到 PVC 声明的容量
- 每个卷有 N 个 replica（我们设的 2），每个 replica 是一个独立的 `.img` 文件，分布在不同的节点上
- 每个 replica `.img` 文件只能落在一张磁盘上，**不能跨磁盘拆分**。申请 600G 的 3 副本卷，需要每张磁盘都有 ≥ 600G 的可用空间
- 稀疏文件有硬上限，**就是 PVC 声明的容量**。写到上限时报 `No space left on device`，不能超
- 用户读写穿透到所有 replica，写入时数据同时写到所有健康的 replica，只要有任一 replica 存活数据就不丢

### 添加新磁盘

**新增磁盘三种方法，都是动态的，不影响已有卷的运行：**

#### 方法一：Longhorn Web UI（推荐）

1. 浏览器访问 Longhorn UI（`kubectl port-forward` 或 NodePort）
2. Node → 选择节点 → 点 ⋮ → Edit Disks
3. 点 Add Disk，填入路径和类型
4. Save

#### 方法二：kubectl

```bash
kubectl edit node.longhorn.io <node-name> -n longhorn-system
```

在 `spec.disks` 中新增一个 disk entry：

```yaml
spec:
  disks:
    /var/lib/longhorn/:              # 已有的目录盘
      allowScheduling: true
      diskType: filesystem
      path: /var/lib/longhorn/
      storageReserved: 0
      tags: []
    /dev/sdb:                         # 新增的裸盘
      allowScheduling: true
      diskType: block
      path: /dev/sdb
      storageReserved: 0
      tags: []
```

#### 方法三：新节点加入，自动创建默认 disk

新节点加入 K8s 集群后，只要满足条件就会自动被 Longhorn 纳入管理：

```bash
# 1. 安装依赖
apt install -y open-iscsi nfs-common
systemctl enable --now iscsid

# 2. 创建目录（默认路径 /var/lib/longhorn）
mkdir -p /var/lib/longhorn

# 3. 打标签触发自动创建默认 disk
kubectl label node <new-node> node.longhorn.io/create-default-disk=true

# 4. 验证
kubectl -n longhorn-system get nodes.longhorn.io -o wide
```

### 迁移/移除磁盘

如果某块盘要退役，Longhorn 支持 **eviction**，先把盘上所有 replica 迁到其他盘，迁移过程中卷不中断：

```bash
# 在 Longhorn UI 中操作
# Node → 选择节点 → Edit Disks → 在目标磁盘上勾选 Eviction Requested → Save
# Longhorn 会自动把这个盘上的 replica 逐个迁移到其他可用磁盘
# 迁移完成后，再物理移除磁盘
```

### 裸盘注意事项

- 裸盘不能有已有分区表或文件系统，加入前需清空：
  ```bash
  wipefs -a /dev/sdb
  ```
- 裸盘被 Longhorn 接管后，不能同时用于其他用途（如普通挂载）
- 裸盘没有文件系统层开销（无 ext4 journal、inode 等），IO 延迟略低于目录模式

---

## 前置条件

每个节点需要安装以下依赖：

```bash
# Longhorn CSI 依赖：iscsiadm
apt install -y open-iscsi

# 启动 iscsid 服务
systemctl enable --now iscsid

# NFS 客户端（nfs-common 已安装）
# 用于挂载 Longhorn 的 RWX Share Manager 导出的 NFS 卷
```

内核模块验证（一般默认已有）：

```bash
lsmod | grep -E 'iscsi_tcp|nfsv4'
```

## Helm 安装

### 1. 添加 Helm 仓库

```bash
helm repo add longhorn https://charts.longhorn.io
helm repo update
```

### 2. 创建专用数据目录

建议在每个节点上创建专用目录（比如利用根分区的空闲空间）：

```bash
mkdir -p /var/lib/longhorn
```

### 3. 安装 Longhorn

2 节点集群需要将默认副本数改为 2：

```bash
helm upgrade --install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace \
  --set defaultSettings.replicaCount=2 \
  --set defaultSettings.createDefaultDiskLabeledNodes=true \
  --set persistence.defaultClassReplicaCount=2 \
  --set csi.kubeletRootDir=/var/lib/kubelet \
  --timeout 10m
```

说明：
- `defaultSettings.replicaCount=2` — 全局默认副本数，2 节点集群必须设 ≤ 节点数
- `persistence.defaultClassReplicaCount=2` — 默认 StorageClass 的副本数
- `createDefaultDiskLabeledNodes=true` — 只在使用 `node.longhorn.io/create-default-disk=true` label 的节点上自动创建默认 disk

### 4. 给节点打标签

只有打了标签的节点才会被 Longhorn 纳入默认 disk 管理：

```bash
kubectl label node node1 node.longhorn.io/create-default-disk=true
kubectl label node worker-1 node.longhorn.io/create-default-disk=true
```

### 5. 验证

```bash
kubectl -n longhorn-system get pods -w
```

所有 pod Running 后，确认 Longhorn UI 可访问：

```bash
kubectl -n longhorn-system port-forward svc/longhorn-frontend 8080:80
# 浏览器打开 http://localhost:8080
```

---

## 创建 RWX StorageClass

安装后 Longhorn 会自动创建默认 StorageClass `longhorn`，但它是 RWO 的。
需要为 RWX 创建独立的 StorageClass。**注意参数值必须用引号包裹，否则 API server 会报错**：

```bash
kubectl create -f - << 'JSONEOF'
{
  "apiVersion": "storage.k8s.io/v1",
  "kind": "StorageClass",
  "metadata": { "name": "longhorn-rwx" },
  "provisioner": "driver.longhorn.io",
  "allowVolumeExpansion": true,
  "parameters": {
    "numberOfReplicas": "2",
    "staleReplicaTimeout": "2880",
    "fsType": "ext4",
    "dataLocality": "best-effort"
  },
  "reclaimPolicy": "Delete",
  "volumeBindingMode": "Immediate"
}
JSONEOF
```

---

## 用户使用 Longhorn

用户只需要关心三个步骤：创建 SC（一次性的）、创建 PVC、创建 Deployment。

### 1. 创建 RWX StorageClass（仅首次）

Helm 安装后会自动生成 longhorn SC（RWO），但 RWX 的 SC 需要手动创建一次：

```bash
kubectl create -f - << 'JSONEOF'
{
  "apiVersion": "storage.k8s.io/v1",
  "kind": "StorageClass",
  "metadata": { "name": "longhorn-rwx" },
  "provisioner": "driver.longhorn.io",
  "allowVolumeExpansion": true,
  "parameters": {
    "numberOfReplicas": "2",
    "fsType": "ext4",
    "dataLocality": "best-effort"
  },
  "reclaimPolicy": "Delete",
  "volumeBindingMode": "Immediate"
}
JSONEOF
```

### 2. 创建 PVC

```yaml
# my-data-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-app-data
  namespace: production
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: longhorn-rwx
  resources:
    requests:
      storage: 10Gi
```

```bash
kubectl apply -f my-data-pvc.yaml
```

PVC 创建后，Longhorn 会自动在后台完成：
- 选择 2 个节点，在每个节点的磁盘上创建 replica（.img 文件）
- 如果是 RWX 卷，额外拉起一个 Share Manager Pod 做 NFSv4 导出
- PVC 状态变为 Bound

### 3. 创建 Deployment 使用 PVC

```yaml
# my-app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: nginx:latest
        volumeMounts:
        - name: data
          mountPath: /data
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: my-app-data
```

```bash
kubectl apply -f my-app.yaml
```

多个副本的 Pod 会被调度到不同节点（K8s 默认调度），所有 Pod 通过 NFSv4 共享同一份数据。

### 用户不需要关心的

- Share Manager 的创建和销毁
- replica 在哪些节点、哪些磁盘上
- 数据的同步复制
- 卷的挂载格式和协议

用户只需要声明"我要多大、什么模式、用哪个 SC"，剩下的 Longhorn 处理。

## 在线扩容

1. 在存储池中增加新磁盘（通过 Longhorn UI 或 kubectl）
2. 扩 PVC：

```bash
kubectl edit pvc <pvc-name> -n <namespace>
# 修改 spec.resources.requests.storage 为目标值
```

或直接用 kubectl patch：

```bash
kubectl patch pvc <pvc-name> -n <namespace> -p '{"spec":{"resources":{"requests":{"storage":"20Gi"}}}}'
```

---

## 常用命令

### 集群状态

```bash
# 查看所有 Longhorn 组件的 Pod 状态
kubectl -n longhorn-system get pods -o wide

# 查看 StorageClass
kubectl get sc | grep longhorn

# 查看节点磁盘池
kubectl -n longhorn-system get nodes.longhorn.io

# 查看节点磁盘容量和使用量（详细）
kubectl -n longhorn-system get nodes.longhorn.io -o yaml | grep -E 'name:|path:|storage(Available|Maximum|Scheduled):'

# 查看卷列表
kubectl -n longhorn-system get volumes.longhorn.io -o wide

# 查看副本分布
kubectl -n longhorn-system get replicas.longhorn.io -o wide

# 查看 Share Manager（RWX 导出实例）
kubectl -n longhorn-system get sharemanagers.longhorn.io
```

### 磁盘管理

```bash
# 添加磁盘（编辑节点 CR）
kubectl edit node.longhorn.io <node-name> -n longhorn-system

# 触发磁盘 eviction（迁移 replica）
# 在 UI 中勾选 Eviction Requested，或通过 kubectl patch
kubectl patch node.longhorn.io <node-name> -n longhorn-system -p '{"spec":{"disks":{"/path/to/disk":{"evictionRequested":true}}}}' --type=merge
```

### 卷操作

```bash
# 创建 RWO PVC
kubectl apply -f - << 'EOF'
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-rwo
spec:
  accessModes: [ReadWriteOnce]
  storageClassName: longhorn
  resources: { requests: { storage: 1Gi } }
EOF

# 创建 RWX PVC
kubectl apply -f - << 'EOF'
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-rwx
spec:
  accessModes: [ReadWriteMany]
  storageClassName: longhorn-rwx
  resources: { requests: { storage: 1Gi } }
EOF

# 查看 PVC 状态
kubectl get pvc -A

# 在线扩容
kubectl patch pvc <pvc-name> -n <namespace> -p '{"spec":{"resources":{"requests":{"storage":"10Gi"}}}}'

# 删除 PVC（会自动清理卷和 replica）
kubectl delete pvc <pvc-name> -n <namespace>
```

### 调试

```bash
# 查看 longhorn-manager 日志
kubectl -n longhorn-system logs -l app=longhorn-manager

# 查看 CSI 插件日志
kubectl -n longhorn-system logs -l app=longhorn-csi-plugin

# 查看卷详细信息
kubectl -n longhorn-system describe volumes.longhorn.io <volume-name>

# 查看节点磁盘详情
kubectl -n longhorn-system describe nodes.longhorn.io <node-name>

# 端口转发访问 Longhorn UI
kubectl -n longhorn-system port-forward svc/longhorn-frontend 8080:80
# 浏览器打开 http://localhost:8080
```

### 验证 RWX

```bash
# 部署 2 副本测试应用，强制跨节点调度
kubectl apply -f - << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rwx-test
spec:
  replicas: 2
  selector:
    matchLabels: { app: rwx-test }
  template:
    metadata:
      labels: { app: rwx-test }
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels: { app: rwx-test }
            topologyKey: kubernetes.io/hostname
      containers:
      - name: alpine
        image: alpine:3.19
        command: ["sleep", "infinity"]
        volumeMounts:
        - name: data
          mountPath: /data
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: demo-rwx
EOF

# A 写，B 读
POD1=$(kubectl get pods -l app=rwx-test -o jsonpath='{.items[0].metadata.name}')
POD2=$(kubectl get pods -l app=rwx-test -o jsonpath='{.items[1].metadata.name}')
kubectl exec $POD1 -- sh -c 'echo "hello" > /data/test.txt'
kubectl exec $POD2 -- cat /data/test.txt
```

---

## 实际安装记录

**环境**

- 集群：2 节点（node1 + worker-1）
- 磁盘方案：根分区（/dev/sda3 ext4）有空闲空间，使用目录 `/var/lib/longhorn/`
- 安装时间：2026-07-17
- 版本：Longhorn v1.12.0

**安装步骤**

1. 每个节点安装 open-iscsi 并启动 iscsid：

```bash
apt install -y open-iscsi
systemctl enable --now iscsid
```

2. 每个节点创建数据目录：

```bash
mkdir -p /var/lib/longhorn
```

3. Helm 安装：

```bash
helm repo add longhorn https://charts.longhorn.io
helm repo update
helm upgrade --install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace \
  --set defaultSettings.replicaCount=2 \
  --set defaultSettings.createDefaultDiskLabeledNodes=true \
  --set persistence.defaultClassReplicaCount=2 \
  --set csi.kubeletRootDir=/var/lib/kubelet \
  --timeout 10m
```

4. 节点打标签触发默认 disk 创建：

```bash
kubectl label node node1 node.longhorn.io/create-default-disk=true
kubectl label node worker-1 node.longhorn.io/create-default-disk=true
```

**安装过程遇到的问题和解决**

- Worker-1 节点 Longhorn manager CrashLoopBackOff：原因是未安装 open-iscsi。安装后重启 Pod 恢复。
- `staleReplicaTimeout` 参数在 YAML 中必须用引号包裹 `"2880"`，否则 Kubernetes API server 报 `cannot unmarshal number into Go struct field StorageClass.parameters of type string`。

**验证结果**

- `longhorn` StorageClass 自动创建，设为默认（`storageclass.kubernetes.io/is-default-class: true`）
- `longhorn-rwx` RWX SC 手动创建，验证通过
- RWX PVC 创建后自动 Bound，Share Manager Pod 自动拉起
- 两个 Pod 分别跑在 node1 和 worker-1，同时读写同一卷，数据一致
- PVC 删除后 Longhorn 自动清理 Share Manager

---

## 与现有 Mayastor + NFS 方案对比

| 项目 | Mayastor + NFS Server | Longhorn |
|---|---|---|
| 副本 | Mayastor replicated | 原生配置 |
| RWX | 手动部署 NFS Server + nfs-csi | 内置 Share Manager，自动创建 |
| 在线扩容 | 不支持 | 原生支持 |
| 新盘加给现有 RWX | 不行，必须起新 NFS Server | 直接 PVC resize |
| 管理界面 | Mayastor CLI + kubectl | Web UI + kubectl |
| 快照/备份 | 需额外工具 | 内置 |
| 运维复杂度 | 两层（Mayastor + NFS） | 一层 |

---


## 高可用与故障恢复

### Share Manager 的可用性方案

Longhorn 的 RWX 卷通过 Share Manager Pod 以 NFSv4 导出。当 Share Manager 所在节点宕机时，恢复时间取决于检测和迁移机制。

**两种机制对比：**

| 机制 | 依赖 | 恢复时间 | 误判风险 |
|---|---|---|---|
| K8s 默认节点驱逐 | node-monitor-grace-period(40s) + pod-eviction-timeout(5min) | ~6 分钟 | 低 |
| RWX Fast Failover (Lease 心跳) | Lease 健康检测，独立于 K8s 节点状态 | **~30-60 秒** | 略高（网络抖动可能误触发） |

### RWX Fast Failover 配置

```bash
# 开启 Lease 心跳检测（默认关闭）
kubectl -n longhorn-system patch settings.longhorn.io rwx-volume-fast-failover -p '{"value":"true"}' --type=merge

# 可选：缩短 K8s 节点检测时间（默认 40s）
# 在 /etc/kubernetes/manifests/kube-controller-manager.yaml 中修改：
# --node-monitor-grace-period=40s → 10s
# kubelet 自动重启 kube-controller-manager 静态 Pod
```

### 测试记录

**环境：** 2 节点集群（node1 + worker-1），Longhorn v1.12.0

**测试方法：** 在 node1 的 App Pod 中持续写入文件（每秒一次），拔掉 worker-1 网线模拟节点宕机，记录写入中断时长。

| 配置 | 恢复时间 | 说明 |
|---|---|---|
| 全部默认 | ~6 分钟 | 等 K8s 驱逐 + 重建 |
| 仅开启 rwx-volume-fast-failover | **~98 秒** | Lease 检测到迁移完成 |
| + node-monitor-grace-period=10s | **~74 秒** | 缩短节点状态检测时间 |

**74 秒的时间线（优化后）：**

```
00:00  写中断，节点断网
00:10  节点标记 Unknown（grace-period 10s）
00:30  Lease 检测到 Share Manager 失联（~20s 心跳超时）
00:40  Longhorn 开始迁移，旧 Pod 标记删除
00:50  新 Share Manager 调度到另一节点
00:55  容器启动（镜像已预拉，秒级）
01:00  NFSd 就绪
01:05  NFS 客户端自动重连
01:14  写入恢复，总中断约 74 秒
```

**结论：** 74 秒已接近当前架构的极限，主要瓶颈在 Lease 心跳超时周期（~20s）和 Pod 重建流程。对大多数业务场景（媒体共享、文件服务等）已经足够。

## 注意事项

1. **2 节点集群**：所有副本数参数必须 ≤ 2，否则卷创建后一直 Pending
2. **dataLocality**：默认是 `disabled`（写远程副本），可改为 `best-effort` 减少延迟
3. **磁盘调度**：建议用 tag 区分 SSD/HDD，通过 StorageClass 的 `diskSelector` 参数控制
4. **新节点加入后，已有卷的 replica 不会自动迁移过去**，需要手动在 UI 里调整 replica 分布
5. **Replica 不能跨磁盘**：一个 replica 的 `.img` 文件必须完整落在一块磁盘上，Longhorn 不会跨盘拆分单个 replica。卷大小必须 ≤ 单块磁盘的可用空间，同时 卷大小 × 副本数 ≤ 所有节点可用空间之和
6. **PVC 容量是硬限制**：虽然 `.img` 文件是稀疏的（按需增长），但最大不能超过 PVC 声明的容量，写满后报 `No space left on device`
7. **升级**：Helm 升级会自动处理，建议先读 changelog
