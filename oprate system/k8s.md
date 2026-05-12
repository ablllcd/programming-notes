# 基础

## 什么是 Kubernetes

Kubernetes（简称 K8s）是一个开源的容器编排平台，用于自动化容器化应用程序的部署、扩展和管理。它最初由 Google 设计并开源，现在由 Cloud Native Computing Foundation (CNCF) 维护。

注意，kuberenetes 是一个容器编排平台，它是一系列的工具和组件的集合，而不是一个单一的软件。而 minikube/kind/kubeadm 是用来`搭建/部署` Kubernetes 集群的工具。

通俗来说，把kubernetes看作一个操作系统：

* kubeadm 是一个安装这个操作系统的工具
* minikube 是一个在本地虚拟机运行这个操作系统的工具
* kind 是一个在 Docker 容器中运行这个操作系统的工具

## 主要特点

### 1. 容器编排
- **自动化部署**：自动将容器部署到集群中的合适节点
- **自动扩缩容**：根据负载自动增减容器实例
- **滚动更新**：无停机时间的应用更新
- **回滚机制**：快速回滚到之前的版本

### 2. 服务发现和负载均衡
- **DNS 服务发现**：自动为服务分配 DNS 名称
- **负载均衡**：在多个容器实例间分配流量
- **健康检查**：自动检测和替换不健康的容器

### 3. 存储编排
- **持久化存储**：支持各种存储后端（本地存储、云存储等）
- **动态存储供应**：根据需求自动分配存储
- **存储类**：定义不同类型的存储配置

### 4. 自我修复
- **故障检测**：监控容器和节点健康状态
- **自动重启**：重启失败的容器
- **节点故障处理**：将故障节点上的工作负载迁移到健康节点

### 5. 配置和密钥管理
- **ConfigMap**：管理应用配置
- **Secret**：安全存储敏感信息
- **环境变量注入**：动态配置应用参数

## 整体架构

Kubernetes 集群由一个控制平面和一组用于运行容器化应用的工作机器组成， 这些工作机器称作节点（Node）。每个集群至少需要一个工作节点来运行 Pod。

![alt text](imgs/k8s-1.png)

### 控制平面
控制平面负责管理集群的整体状态，包括调度、监控和维护。它由以下组件组成：
- **API Server**：集群的前端接口，处理 REST 请求
- **etcd**：分布式键值存储，保存集群状态
- **Scheduler**：负责将 Pod 调度到合适的节点
- **Controller Manager**：运行控制器进程，负责维护集群的期望状态

控制平面组件可以在集群中的任何节点上运行。 在多节点集群中，控制平面通常不运行在工作节点上，以确保其高可用性和安全性。而在单节点集群中，控制平面和工作负载可以在同一节点上运行。

### 工作节点
工作节点负责运行容器化应用程序。每个工作节点包含以下组件：
- **kubelet**：节点代理，管理 Pod 生命周期
- **kube-proxy**：网络代理，实现服务发现和负载均衡
- **Container Runtime**：容器运行时（Docker、containerd 等）

### 客户端工具
- **kubectl**：命令行工具，用于与 Kubernetes API Server 交互。我们可以使用 kubectl 来部署应用、查看和管理集群资源、以及调试应用程序。（其不属于 Kubernetes 集群的一部分，但它是与 Kubernetes 交互的主要工具。）

## 核心概念

### Pod
- 最小部署单位
- 包含一个或多个紧密相关的容器
- 共享网络和存储

### Service
- 为 Pod 提供稳定的网络访问入口
- 支持负载均衡
- 类型：ClusterIP、NodePort、LoadBalancer、ExternalName

### Deployment
- 管理 Pod 的副本集
- 支持滚动更新和回滚
- 声明式配置

### Namespace
- 逻辑隔离资源
- 多租户支持
- 资源配额管理

### Service 和 Pod 的关系

Service 和 Pod 是 Kubernetes 中两个核心且紧密关联的概念，它们的关系可以用一句话概括：**Service 为一组动态变化的 Pod 提供稳定的网络访问入口**。

#### 1. 为什么需要 Service？

Pod 具有以下特性，导致直接访问 Pod 不可靠：

- **IP 不固定**：Pod 每次创建/重启都会被分配新的 IP 地址
- **动态扩缩容**：Pod 的数量会随副本数变化而增减
- **节点迁移**：Pod 可能因节点故障被调度到其他节点

Service 解决了这些问题，为客户端提供一个**固定的虚拟 IP（ClusterIP）和 DNS 名称**。

#### 2. Service 如何关联 Pod —— Label Selector

Service 通过 **Label Selector（标签选择器）** 来匹配合适的 Pod：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
spec:
  selector:         # ← 标签选择器
    app: my-app     #   选择所有带 app=my-app 标签的 Pod
  ports:
    - protocol: TCP
      port: 80          # Service 端口
      targetPort: 8080  # 转发到 Pod 的端口
```

对应的 Pod 需要带有匹配的标签：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
  labels:
    app: my-app     # ← 与 Service selector 匹配
spec:
  containers:
    - name: my-app
      image: my-app:latest
      ports:
        - containerPort: 8080
```

#### 3. 关系示意图

```
┌──────────────────────────────────────────────────┐
│                   Service                         │
│              ClusterIP: 10.0.0.1                  │
│              Port: 80                             │
│         selector: app=my-app                      │
├──────────┬──────────┬──────────┬──────────────────┤
│          │          │          │                   │
│    ┌─────▼────┐ ┌──▼─────┐ ┌──▼──────┐            │
│    │  Pod A   │ │ Pod B  │ │ Pod C   │   ...      │
│    │10.0.1.2  │ │10.0.1.5│ │10.0.1.9 │            │
│    │app=my-app│ │app=my-a│ │app=my-ap│            │
│    └──────────┘ └────────┘ └─────────┘            │
│                                                    │
│  客户端 → Service(10.0.0.1:80) → 随机转发到某个Pod   │
└──────────────────────────────────────────────────┘
```

#### 4. 核心特性

| 特性 | 说明 |
|------|------|
| **稳定访问入口** | Service 的 ClusterIP 和 DNS 名称在生命周期内保持不变 |
| **自动负载均衡** | 请求被随机分发到符合条件的 Pod（默认基于 iptables/IPVS） |
| **动态感知** | Service 持续监听 Pod 变化，自动加入/移除 Endpoint |
| **解耦** | 客户端只需知道 Service 名称，无需关心后端 Pod 的具体 IP |
| **多种暴露方式** | ClusterIP（集群内访问）、NodePort（节点端口）、LoadBalancer（负载均衡器）、ExternalName（外部服务映射） |

#### 5. Endpoint 与 EndpointSlice

当 Service 通过 Selector 匹配到 Pod 后，Kubernetes 会自动创建对应的 **Endpoint**（或 **EndpointSlice**，新版默认）资源，记录所有后端 Pod 的 IP 和端口：

```bash
# 查看 Service 对应的 Endpoint
kubectl get endpoints my-app-svc

# 输出示例
NAME         ENDPOINTS                          AGE
my-app-svc   10.0.1.2:8080,10.0.1.5:8080,10.0.1.9:8080   5m
```

这里Service 的ClusterIP 是虚拟IP,只供Cluster内访问; 而Endpoints中的IP是实际物理Pod的IP。当客户端 (Pod) 访问Service的CLuster IP时，kube-proxy会根据Endpoint列表将流量转发到对应的Pod上。

**流量转发流程**：

```
客户端请求 → Service(ClusterIP:Port) → Endpoint(记录Pod列表) → 具体某个Pod
```

注意这里指的是Cluster内部的访问；如果需要从集群外部访问Service，则需要使用<任意NodIP>:<service-port>来访问。

#### 6. 无 Selector 的 Service

Service 也可以不指定 Selector，用于手动管理 Endpoint，常见场景：

- 访问**集群外部的服务**（如外部数据库）
- 跨命名空间访问服务
- 迁移过程中的过渡阶段

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  ports:
    - port: 3306
---
apiVersion: v1
kind: Endpoints
metadata:
  name: external-db
subsets:
  - addresses:
      - ip: 192.168.1.100    # 外部数据库 IP
    ports:
      - port: 3306
```

## 常见操作

### 查看K8S状态
```
kubectl cluster-info    # 查看集群信息
kubectl get nodes       # 查看节点状态
kubectl get componentstatuses  # 查看控制平面组件状态
kubectl get pods        # 查看Pod状态
kubectl get svc         # 查看服务状态
```

### 操作Pod
```
kubectl get pods -o wide  # 查看Pod的详细信息
kubectl describe pod <pod-name>  # 查看Pod的详细描述
kubectl logs <pod-name>  # 查看Pod的日志
kubectl exec -it <pod-name> -- /bin/bash  # 进入Pod的交互式终端
kubectl delete pod <pod-name>  # 删除Pod(会被Deployment等控制器自动重建)
```
