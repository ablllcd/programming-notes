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

#### Service 类型讲解
| 类型 | 说明 |
|------|------|
| ClusterIP | 默认类型，提供集群内部访问 |
| NodePort | 在每个节点上开放一个端口，允许外部访问 |
| LoadBalancer | 在云环境中创建负载均衡器，提供外部访问 |
| ExternalName | 将 Service 映射到外部 DNS 名称 |

注意：类型是向下兼容的，NodePort 包含 ClusterIP 的功能，LoadBalancer 包含 NodePort 的功能。

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

#### 5. 哪些pod应该放入相同的Service中？
在 Kubernetes 中，判断 Pod 是否应该放入同一个 Service，核心看一点：`这些 Pod 是否对外提供“无差别”的服务`。

Service 的本质是一个负载均衡器，它将流量随机或按策略分发到后端的 Pod。因此，如果请求发送到 A Pod 和 B Pod，对客户端来说必须是可以互换的，它们才应该在一个 Service 里。

简单来说，同一个应用的多个副本（ReplicaSet 管理的 Pod）通常应该放在同一个 Service 中，因为它们提供相同的功能和接口。而不同应用或不同版本的 Pod 则应该分开到不同的 Service 中，以避免混淆和错误路由。

#### 6. Endpoint 与 EndpointSlice

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

#### 7. 无 Selector 的 Service

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




### Namespace的作用

Namespace 位于 Kubernetes 的逻辑组织层，具体位置如下：
```
┌─────────────────────────────────────────────┐
│                集群 (Cluster)               │
│  ┌─────────────┐  ┌─────────────┐           │
│  │ Namespace A │  │ Namespace B │  ...      │
│  │  ┌───────┐  │  │  ┌───────┐  │           │
│  │  │ Pod   │  │  │  │ Pod   │  │           │
│  │  │Service│  │  │  │Service│  │           │
│  │  │ConfigMap││  │  │ConfigMap││           │
│  │  └───────┘  │  │  └───────┘  │           │
│  └─────────────┘  └─────────────┘           │
│          ↓ 物理层面 ↓                        │
│  ┌─────────────────────────────────┐        │
│  │    Node 1    │    Node 2    │   │        │
│  │  Pod(A) Pod(B) Pod(A) Pod(B) │  │        │
│  └─────────────────────────────────┘        │
└─────────────────────────────────────────────┘
```

Namespace 不涉及物理隔离，Pod 依然可以在不同 Node 上混合调度。



### Ingress

#### 1. 为什么需要 Ingress？

Service 的 NodePort 和 LoadBalancer 类型虽然能将服务暴露到集群外，但存在一些局限：

- **端口管理混乱**：每个 Service 一个端口，难以记住和统一管理
- **无法通过 HTTP域名进行访问**：NodePort/LoadBalancer 工作在传输层(TCP/UDP)，无法基于 HTTP 路径、域名等 L7 信息做路由，

**Ingress** 解决了这些问题，它工作在应用层(L7)，为集群内的 Service 提供**统一的 HTTP/HTTPS 入站访问入口**。

#### 2. 什么是 Ingress？

Ingress 是 Kubernetes 的一种 API 资源，用于管理集群外部访问集群内部服务的 HTTP/HTTPS 路由规则。它相当于集群的 **"智能网关"** 或 **"反向代理"**。

```
       ┌──────────────────────────────────────────┐
       │              Internet                     │
       │             http://myapp.com              │
       └────────────────┬─────────────────────────┘
                        │
                 ┌──────▼──────┐
                 │   Ingress    │  ← 统一入口，根据规则分发
                 │  (Nginx/HAProxy/Traefik/...)
                 └──┬───────┬──┘
                    │       │
             ┌──────▼─┐ ┌──▼──────┐
             │Service A│ │Service B│
             │:80      │ │:80      │
             └────┬────┘ └────┬────┘
                  │           │
             ┌────▼────┐ ┌───▼────┐
             │ Pod A1  │ │ Pod B1 │
             │ Pod A2  │ │ Pod B2 │
             └─────────┘ └────────┘
```

#### 3. Ingress 的核心组成

| 组件 | 说明 |
|------|------|
| **Ingress Controller** | 实际的流量转发组件（如 Nginx Ingress Controller、Traefik、HAProxy、AWS ALB Ingress Controller），负责解析 Ingress 规则并实现反向代理 |
| **Ingress 资源** | 定义的 YAML 规则，描述"什么域名/路径 → 转发到哪个 Service" |
| **TLS 证书** | 可选，配置 HTTPS 证书实现 SSL 终止 |

> **注意**：Ingress 资源本身只是一组规则定义，真正的流量转发由 **Ingress Controller** 实现。集群默认不会安装 Ingress Controller，需要手动部署。

#### 4. 一个完整的 Ingress 示例

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /   # 路径重写
spec:
  ingressClassName: nginx          # 指定 Ingress Controller
  tls:
    - hosts:
        - myapp.example.com
      secretName: myapp-tls-secret # TLS 证书 Secret
  rules:
    - host: myapp.example.com      # 域名路由
      http:
        paths:
          - path: /api
            pathType: Prefix       # 前缀匹配
            backend:
              service:
                name: api-service
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-service
                port:
                  number: 80
    - host: admin.example.com      # 另一个域名
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: admin-service
                port:
                  number: 80
```

#### 5. Ingress 路由规则详解

Ingress 支持多种路由方式：

| 路由方式 | 示例 | 说明 |
|----------|------|------|
| **域名路由** | `host: app.example.com` | 根据不同域名转发到不同 Service |
| **路径路由** | `path: /api` | 根据不同 URL 路径转发到不同 Service |
| **域名 + 路径** | 两者组合 | 同时匹配域名和路径 |
| **默认后端** | 不指定 host | 处理所有未匹配规则的请求 |

**pathType 说明**：

| pathType | 行为 |
|----------|------|
| `Prefix` | 前缀匹配，如 `/api` 匹配 `/api`、`/api/v1`、`/api/v2/users` |
| `Exact` | 精确匹配，如 `/api` 只匹配 `/api` |
| `ImplementationSpecific` | 由 Ingress Controller 自行决定匹配规则 |

#### 6. 流量流转完整路径

```
用户请求 (http://myapp.example.com/api/users)
  │
  ▼
① DNS 解析 → 指向集群入口（NodeIP 或 LB 地址）
  │
  ▼
② Ingress Controller（如 Nginx）接收请求
  │  检查 Ingress 规则：host=myapp.example.com, path=/api → api-service:80
  ▼
③ Service (api-service) 接收请求
  │  kube-proxy 根据 Endpoint 列表转发
  ▼
④ Pod (api-service-xxx) 处理请求并返回响应
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
kubectl exec <pod-name> -- command  # 在Pod中执行命令
kubectl delete pod <pod-name>  # 删除Pod(会被Deployment等控制器自动重建)
```

### 操作Namespace
```
kubectl -n <namespace-name> <command> # 在指定命名空间下执行命令
kubectl get namespaces  # 查看所有命名空间
kubectl create namespace <namespace-name>  # 创建命名空间
kubectl delete namespace <namespace-name>  # 删除命名空间
kubectl config set-context --current --namespace=<namespace-name> # 切换默认命名空间
kubectl config view --minify | grep namespace:  # 查看当前默认命名空间
```

# Kubectl

## 配置

### 配置文件位置
```bash
# Linux/MacOS
~/.kube/config
# Windows
%USERPROFILE%\.kube\config
```

注意，kubectl是跟用户绑定的，例如ubuntu用户和root用户去查找的配置文件位置是不同的。

### 配置文件作用

- **集群信息**：包含集群的 API Server 地址、证书等连接信息；kubectl就是根据这里的信息来连接 Kubernetes 集群的。
