## Smart-City-Orch

边缘云平台 Edge-Integrated Microgrid Cloud | 核心全栈开发

Edge-Integrated Cloud Platform 是一套基于 Kubernetes 自研的边缘云操作系统，将多台裸机服务器抽象为统一的资源池，提供节点自动发现、应用编排部署、物理拓扑展示、平台资源监控，多层权限管理等核心能力。平台核心包含自研的编排管理器（Orchestrator），实现对计算节点生命周期、容器化应用调度、持久化存储分配、多租户权限隔离的端到端管控，并集成配置 VPP， DHCP， DNS，openEBS, NFS 等开源组件，支撑编排器对物理节点进行管理。

核心贡献：

1. 框架架构升级（OSGI → Spring Boot）：主导了从遗留 OSGI-Spring 框架到 Spring Boot 4.0.1 + Java 21 的全量架构升级，重新设计模块化工程结构，统一了组件扫描、依赖注入、配置管理和事务控制体系，消除了 OSGI 运行时类加载复杂性，显著降低系统启动延迟并提升团队开发效率。
2. 高可用分布式存储（openEBS + NFS）：引入 openEBS 与 NFS 作为有状态应用的持久化存储方案，为集群提供高可用的分布式块存储能力，以及跨节点访问数据&多节点同时读写能力，为APP的高可用部署提供了基础设施保障。
3. 核心业务开发：负责"集群纳管 → Site 划分 → 镜像上传 → 镜像配置 → 应用部署"全链路的产品化实现。基于 IKubctlClient 抽象实现多 Kubernetes 集群统一管理；站点创建时自动同步 Namespace和节点标签；集成 TUS 断点续传协议对接 Harbor 镜像仓库实现大镜像可靠上传；应用通过参数化模板经 Dispatch 调度引擎翻译为 K8s Deployment / StatefulSet 等资源实现批量编排下发；应用部署时可分配域名，调度K8S创建Ingress资源并同步修改DNS解析记录。
